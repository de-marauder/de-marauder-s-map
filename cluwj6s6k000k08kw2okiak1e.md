---
title: "Container Networking"
seoTitle: "Container networking from scratch"
seoDescription: "Container networking is built on core Linux networking primitives. A beautiful combination of Linux network interfaces, routing rules, and iptables."
datePublished: Fri Apr 12 2024 10:34:22 GMT+0000 (Coordinated Universal Time)
cuid: cluwj6s6k000k08kw2okiak1e
slug: container-networking
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1712916214112/6e2ac860-9405-4034-bd56-483dd2756bc8.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1712917816898/cc337ee6-c756-487e-aa73-237792ee4b30.png
tags: docker, networking, containers, iptables, network-interfaces

---

In a previous [article](https://de-marauder.hashnode.dev/whats-in-a-container), we discussed how containers are created and the requirements for a container. Among the requirements is network isolation which is provided by Linux network namespaces. But our solution was largely incomplete at least compared to contemporary containerization technologies. One of the reasons was the inability of a container to talk to another container, the host machine, or a different node or VM on the host.

In this article, we’ll be diving into how we will be diving into how we can fix these problems and how we can grant containers the ability to speak! LFG!

Pre-requisites

* Basic Linux knowledge
    
* A Linux host (preferably a clean VM)
    

# Introduction

To properly understand container networking we must first understand Linux networking as containers (at least Linux containers) are largely facilitated by Linux networking primitives.

To handle internal networking on a Linux host the kernel exposes virtual [network devices](https://de-marauder.hashnode.dev/introduction-to-computer-networks-subnetting-ip-addresses#heading-network-devices) and interfaces which we can take advantage of. Some of them include:

* **The Linux bridge**: This works similarly to a normal bridge; extending a network by providing an interface to allow other networks to plug into an existing network
    
* **Veth pairs**: This network interface is essentially a virtual ethernet cable. Unlike the real thing this one is made of software and its job is to accept data from one end and output it via another end.
    
* **TUN**: A tun interface is kind of like a virtual NIC, with a dangling ethernet cable. It can be assigned IP addresses which makes it a layer 3 device.
    
* **Ipvlan**: This interface is used to provide connectivity between multiple networks on a Linux machine without the use of a bridge. This is because the use of a bridge introduces extra hops and latency in the lifecycle of network traffic. The ipvlan interface attaches directly to the ethernet interface instead to achieve lower latency and higher security.
    

Observe a typical host,

![](https://lh7-us.googleusercontent.com/Gbjm2G_ajF-j239FoBWgYoLmOoVxZ9gAZOyM4Yw45_c0WD39GPQzyidO7UU5qWnPqTOgtJgQKozXFWY4miF5NUYzce5cwP3ipXHvKw9S8rXi9n_Qz6MRQBSva6wu-eOc5PlBeFae96Gw4fCIz3naNhA align="left")

It has a physical Network Interface Card (NIC) which serves as an entry and exit point (gateway) for network traffic. This NIC usually comes with a port for an ethernet cable or WiFI adapter which carries the serialized network packets as voltage signals through an electrical cable or radio waves through space. Without the NIC, it would be impossible for a host to communicate with an external device on an external network. The OSI model makes this a bit easier to think about as all traffic eventually gets serialized down to layer 1 (physical layer) traffic where they are transferred via cable or radio waves.

![](https://lh7-us.googleusercontent.com/10nXUIrBIo6OQaZpzNSS7id56ykFNf5jZFNd5Q2z6Aol9E-uxA5ujqxEt8uPL9Jr0TTPgHwTktuAAoEUy8Lj0doV7Oe107rVwwjpSIRzydh7PVxyg3GUQfDJf8ZkHFPAO6FBXbE7FhL_bzwy17xgOHI align="left")

This presents a problem when we want to create virtualized hosts as we need to find a way to emulate these NICs for a scalable solution. In the context of this article, we will be dealing mostly with containers when we speak of virtualized hosts.

# Container Network Models

The aspect of the container we are particularly interested in in this article is the network namespace. Throughout this article, when references are made to containers, it's most probably network namespaces that are being referred to.

A network namespace provides a separate instance of the network stack and isolates it so that only processes running within that namespace can see it. These processes cannot see or access any other network stack by default.

For this reason, when a container is created without any network configuration it has no access to the outside world beyond the namespace it exists in.

![](https://lh7-us.googleusercontent.com/uSJG71n9mL6k9gn_gHElfQ-unYyrZzIvp-hdSIZLMqegLirlfIafnWHhny7y6JN613pAR3hM2dot5W98dWLnI_RevPb_lts8c239nMP1e-JMSgSHT3eSxAesGhfTpvW1dvAqn38sKxrjIUY6oB85ERw align="left")

We can’t have this because we almost always want our containers to talk to other containers and/or hosts whether on the same or different nodes.

We shall be discussing 4 configurations to understand how we can make this happen

1. Single container single host
    
2. Multi-container and single-host
    
3. Multi-container Multi-host on a physical L2 network
    
4. Multi-container Multi-host over the internet.
    

## Single container single host

This is the base case, we will set up a new network namespace and set up a communication channel from it to the default network namespace.

To make this work we will be making use of the Linux networking primitive, `veth` pairs. We will be making major use of the `ip` tool which contains a lot of utilities for Linux network configuration.

![](https://lh7-us.googleusercontent.com/nZOkVEI2tmJd3yTUffwIkZEHXHdzueUHD2Z8b9d6vqAJOikHBff6TIUrwTgM1WwuqucSyNgLlYn9MHWbhmA9YeLnKsbGiKUgMYlSR4pRuJq55rgmWdLAeFcFJIGeUPIrPpBg_I7pmBEMDl4EJV_TwX8 align="left")

Let us begin by inspecting the current state and create a new container namespace

```bash
# Check current available network namespaces
ip netns list

# create a new network namespace
sudo ip netns add container

# Check current interfaces available
sudo ip -n container -br -c addr
```

<details data-node-type="hn-details-summary"><summary>ip netns exec</summary><div data-type="detailsContent">ip netns exec &lt;net-ns-name&gt; &lt;command&gt; allows us to run any command in a specified network namespace</div></details><details data-node-type="hn-details-summary"><summary>ip -n</summary><div data-type="detailsContent">ip -n &lt;net-ns-name&gt; &lt;command&gt; allows us to run ip commands in a given namespace</div></details>

You’ll find that there are no interfaces that exist in this namespace which makes sense as it is a plain copy of the network stack without extra configurations.

If you run `ifconfig` or `ip addr` in the default namespace, you might observe that there are a couple of interfaces depending on the host’s configuration:

* a loopback interface (`lo` aka `localhost`),
    
* a WLAN interface for WiFi (looks like `wlp0s20f3`)
    
* A bridge interface (`docker0` if you have docker installed on the host)
    
* `eth0` or something like `enp0s31f6` which indicates the ethernet interface available on the host NIC.
    

In my case, I only have an `eth0` and a `loopback`.

![](https://lh7-us.googleusercontent.com/0mEWbqpC-f2QRKXSRpK56S95-1Mp3CFbg5AksuYahkeLfu5PKBxJYZenEvFiwC-gicidAEh_71vgfuaghKyul_1PhzhpGPJitjkX6Y2q6G0Orxdg2D47FQoikcH9XUtf54vxirgzECor_zb0sJb84HM align="left")

These interfaces in the default namespace are the reason we can perform a bunch of network operations from the host but we can’t do anything from the container namespace.

Let’s fix that!

```bash
sudo ip link add name veth0 type veth peer veth_0
sudo ip -br -c addr list
```

The first command instructs the kernel to create a new veth interface with one end named `veth0` and the other end named `veth_0`. Checking available interfaces in the default namespace shows that the veth pair has been added (both of them).

![](https://lh7-us.googleusercontent.com/BRcSUjQnFB8tSEbxRBV4OdUrhOiQOw9HhyW8G1ziWCN3yCEN7dXKtkmoiPyRYTXvWXHLGgXsjevNbmoSODJYzl366mUceO9Ay7KTn9RyKs4-_YZ1UIgpof3Z2pz5cjhNS3Sqa098tSqYFxX5Ao6X0SI align="left")

The veth interface allows us to send network information from one end and receive it from another end. Since these things are just processes mimicking network devices we can move one end into our container namespace and leave the other end in the default namespace. This way traffic to/from the container can get in/out of it via the veth pair.

```bash
# move the veth_0 end of the veth pair into our container network namespace
sudo ip link set dev veth_0 netns container

# verify that it has been moved
sudo ip -br -c addr ls # no longer present in default namespace
sudo ip -n container -br -c addr ls # Now has two interfaces the veth and a loopback interface
```

![](https://lh7-us.googleusercontent.com/gMwJWaTlfzkSip-YZBkSWeBkPO1Rv5746jgVlEJp931APzeh3VgqsCcDvY27h03NTJwoDJfxFdDRzNzSReXnV9srDxoZ8p0OotZy2sP2jPtiCu6DwZj4BBjcfu07iLHfgRI-RQd2ypkKRig_pV_bYyA align="left")

You’ll notice how these new interfaces don’t have IP addresses attached to them. We need to assign IP addresses to the veth interfaces so we can reference them by that and set up a routing rule so traffic can be sent to the appropriate interface.

In reality, what we’d be assigning to the interface is a subnet. The interface device will then get the IP address specified in the subnet mask while the gateway and broadcast IPs are set.

For this setup we will make the gateway address for both ends of the veth device to be the same.

```bash
# add CIDR range for veth0 device in the default namespace
sudo ip address add 10.0.0.1/24 dev veth0

# add CIDR range for veth_0 device in the container namespace
sudo ip -n container address add 10.0.0.2/24 dev veth_0
```

Now that the interfaces have IPs, we can bring them into an `UP` state

```bash
# default namespace
sudo ip link set dev veth0 up

# container namespace
sudo ip -n container link set dev veth_0 up
```

Now when we show all interfaces we get the result below.

![](https://lh7-us.googleusercontent.com/1BWa3imwDSIw5Egm784DBWm166F3yzlp3juokKXzlsvVf8Oe-lHZugBO5IkQ42rFa2BttWbIhsw09EUQuzMXwmksGARPmzqqN0W9pV-aIs_EBA9CGCa-N1G_ne2OJ8bFux7V6-Pn1QsR7IZwyvhX91k align="left")

You can see the `veth` interfaces in an `UP` state and with their IP address. You can also bring up the loopback interface but it’s not required for this demonstration.

We can now ping the container namespace from the host namespace and vice versa

![](https://lh7-us.googleusercontent.com/EYeagw4p-1pi4VNzo0v61hkrZyeJ8qlkT4vh0EJyK7vsL0-4ZqAJc7ysHq-UREfdHrujx6NoF3vFNvZlg9cuSFXs8dwiFJuH6q2OCPtHdsxC7XUeJMT_-5PYQkLVaUSiGtvGlT_2vAmgMUmniaVDYM4 align="left")

However, from the container namespace, we cannot access the internet or other interfaces outside the container outside the 10.0.0.0/24 subnet.

To fix this we need to set up routing rules and `iptables` rules

```bash
# Allow all traffic to leave via 10.0.0.2 on the veth_0 peer by default
sudo ip -n container route add default via 10.0.0.1 dev veth_0
```

![](https://lh7-us.googleusercontent.com/jsCDRWvTAErPYCcwwa0YFoRkXFN5zSBdCHuzxlS1berEhuBjntNK4gW2xUe_Ko4w3URBYqPOkgzFI3fIrRhbFQ5sAd3X7x9fVXdTFALapAWrULNS8kdTxtGT4nOv8msUyOLgPjJZ9TKODSkLUaljxoM align="left")

Now we can ping other interfaces in the default namespace outside the 10.0.0.0/24 subnet. But we still can’t get to the internet.

```bash
sudo iptables -t nat -I POSTROUTING -s 10.0.0.0/24 ! -o veth0 -j MASQUERADE 
sudo iptables -I FORWARD -i veth0 -o eth0 -j ACCEPT
sudo iptables -I FORWARD -o veth0 -i eth0 -j ACCEPT
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Run these commands preferably on a disposable VM. To avoid messing up previously existing rules.</div>
</div>

The first command sets a NAT (Network Address Translation) rule to route network packets from the source `10.0.0.0/24` to any interface that isn’t `veth0` and mask the traffic behind an internet gateway address as it exits the host. We also need to set forwarding rules to tell the veth0 interface to forward outbound traffic to the eth0 interface for egress and vice versa for ingress.

With that, we can get to the internet.

![](https://lh7-us.googleusercontent.com/HzfRiNlRfVxkYNAnz2eqAwxVKK4Z-fZkoRvUUGF0WXTaFp4g4NttRIqWH2llT0XTd1FLK1IBPE4jZSC_23fIUbs6kNx4YRyCoAe1pyEzZy4Tt_PdrYxIfyEIKatFwlL8p7r5w9yYHk5UNMJ7ZuvCBe4 align="left")

That’s it for a single container single machine configuration.

## Multi-container single host

Let’s take this a step further. We might want to run a 3-tier application with each service having its own container and all of them should be able to interact freely with each other and the internet.

Let’s tackle an example demo of how we would achieve this. We would be doing basically the same thing as in the last example but we would be making two network namespaces and connecting them via a Linux bridge that will expose them to the host from where we can then route outbound traffic to the internet.

![](https://lh7-us.googleusercontent.com/73GpLCcqmrWNzOGegPLCrns2iB6KlEhFKLxpgSiMEzpH2It-g1KQutelK7uvr0OzHE37ssEQusLuxymPK-Y5-_IzxHcGyEdYD-OMrPBeqKTKzRv73Do4sICGEJbDFagFVzQgXznzF-tADq3Phkyb0vI align="left")

**Step 1**: Create two container network namespaces

```bash
sudo ip netns add ns1
sudo ip netns add ns2
```

**Step 2**: Create the required interfaces

```bash
sudo ip link add name br0 type bridge
sudo ip link add name veth1 type veth peer veth_1
sudo ip link add name veth2 type veth peer veth_2
```

**Step 3**: Move the veth peers into the created namespaces

```bash
sudo ip link set dev veth_1 netns ns1
sudo ip link set dev veth_2 netns ns2
```

**Step 4**: Assign IP addresses to the interface

It’s advisable to make use of an IP address from a particular subnet say `10.0.0.0/24` just to keep things simple and avoid complicated routing rules. This way when we assign IP addresses to the interfaces, the gateway address for the interfaces remains the same while the assigned IP becomes the IP address without the subnet mask. In this case, for the bridge `br0`, we have 10.0.0.1 as the gateway address for the bridge while the subnet remains the same `10.0.0.0/24`. Since the gateway is the same, we won’t run into issues with our route tables and iptables configuration. If you are interested in understanding how subnetting works you can find more information [here](mailto:ecoonlineglobal@gmail.com)

```bash
sudo ip addr add 10.0.0.1/24 dev br0

# The veth1 and veth2 pairs don’t need IP addresses since they will be plugged into bridge br0 in the next step
sudo ip -n ns1 addr add 10.0.0.2/24 dev veth_1
sudo ip -n ns2 addr add 10.0.0.3/24 dev veth_2
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Only dangling (unconnected) interfaces require IP addresses.</div>
</div>

**Step 5**: Connect the veth peers in the default namespace to the bridge by setting the bridge as their master interface (think of it as plugging an ethernet cable into a normal bridge’s port)

```bash
sudo ip link set dev veth1 master br0
sudo ip link set dev veth2 master br0
```

**Step 6**: Bring all created interfaces up

```bash
sudo ip link set dev br0 up
sudo ip link set dev veth1 up
sudo ip link set dev veth2 up
sudo ip -n ns1 link set dev veth_1 up
sudo ip -n ns2 link set dev veth_2 up
```

**Step 7**: Setup routing rules

```bash
# route all traffic in the containers through the veth peer interface 
# in them and through the br0 bridge gateway 10.0.0.1
sudo ip -n ns1 route add default via 10.0.0.1 dev veth_1
sudo ip -n ns2 route add default via 10.0.0.1 dev veth_2

# Setup iptable rules to route internet-bound traffic
sudo iptables -t nat -I POSTROUTING -s 10.0.0.0/24 ! -o br0 -j MASQUERADE 
sudo iptables -I FORWARD -i br0 -o eth0 -j ACCEPT
sudo iptables -I FORWARD -o br0 -i eth0 -j ACCEPT
```

We should now be able to ping from one container namespace to another as well as reach the internet from a container namespace.

![](https://lh7-us.googleusercontent.com/pZ7Xc6BUFHMCYTPQrg1mnw0rYNA87E_sCelQS9eowE1ELiVsVTvqJWWOsZYLjHeka1-5AopcC0QW0IbK8x5WdzUS8lMgqrq1Amopl26zLzEFpIfpMVByMcNT59nKB9Uz-bKbtQpMuyitm4JU30TPKpw align="left")

The Linux bridges are used in this case to extend container networks just like a normal physical bridge can be used to extend a LAN. All you have to do is plug the other end of the container veth pair into the bridge by labeling it as a master. The bridge allows us to build a cluster of containers with a shared network isolated from the host. It is the default network device used by Docker to manage container networks.

When you install docker, it creates a `docker0` bridge by default which is used to connect the host’s network to the container’s network and route internet-bound traffic properly. You also create new sets of these bridges to group your containers by network. You can check this out by installing docker on a VM and running `ip link`. When you run containers, you’ll also see veth pairs being created by Docker.

The results of this configuration show how we can manually make multiple containers on the same host communicate with each other over a network.

## Multi-container multi-host configuration

In this scenario, we might be concerned with redundancy and failover for our container nodes. We want to ensure that we can replicate containers over multiple nodes. This would allow our services to remain operational in the event a node fails.

We might also just be interested in making a call from one service to another containerized service on a different node.

This type of behavior is usually achieved by using tools such as docker swarm, ECS, or Kubernetes that share containers in clusters of multiple nodes.

### Multi-container multi-host over a physical layer 2 network

![](https://lh7-us.googleusercontent.com/beaxh0jRMo6NpDmSVnyWzpA87_rhZpyIO4wvS4g7ojlsrE8EM-yxuRkbTfpXP4YGgk77ei86x8fYa-D-Fp5UAzafRfzHG4cIIP21BfVWKSLIbb2MR8KAhAdm3R1FRvQEKj2u0RF23iEMjOvXKXoUgJc align="left")

For this setup, we could create a duplicate copy of the example in a multi-container single-host. Of course, the copy would be on a different host or VM. The difference here would be that both hosts or VMs have to be connected by a layer 2 network device, a switch. This will allow the sending of packets from one host directly to the other. You’d just have to set up some extra routing rules to let the kernel know how to process calls to container IP networks in the different connected hosts.

So for example, if you had host A and host B each with the same configuration as in the previously discussed example save for the subnet range of the container networks. You can route all requests to the container networks in host B coming from host A to the ethernet gateway interface (usually `eth0`) which will send it to the switch which releases it to host B and to its destination container network.

This could work for an on-prem setup with physical servers or using VMs that share the same NIC.

### Multi-container multi-host over the internet (cloud setup)

![](https://lh7-us.googleusercontent.com/_tgn6FiL_iWeUn23B3eMTuq7rGGNJaGaixdERi-2cAqgxtOXMYZdtoiDCxOBoSGStuKuI5dy5LtbLWwmnky7_JzApkj2dzJSM3Kyxndps-ephEZA09SD9RoLTi4RvTSOLF-56QlFr4weLkmLh2W44aA align="left")

This is quite similar to that involving a physical layer 2 network. The only difference now is that the network traffic is routed from one node to another using a layer 3 device (a router) through the internet as opposed to the switch (a layer 2 device).

The networking implementation is usually done using an overlay network which is fancy speak for an external network injected into the system to encapsulate and move network traffic from one node to another. To do this, these tools usually store some state data about routing information in an external service like etcd, consul, or a database.

Tools like Kubernetes allow you the liberty to choose an overlay network provider that works best for you. Options include calico, flannel, weave, etc. These tools make it easy for containers in a cluster to find each other on different nodes in the cluster.

# Conclusions

This article aimed to demystify the “magic” seen in container networking and understand how it works. We can see quite clearly that it’s all just regular Linux networking. A pleasant combination of Linux bridges, veth pairs, route tables, and iptables. These Linux networking primitives allow us to securely expose our containers for communication over a network.

Besides the configurations discussed in this article, container networking can still be enhanced by using some other configurations as can be provided by the ipvlan and macvlan interfaces. These interfaces provided a higher level of security and lower latency for network traffic than their sibling, the Linux bridge.

If you’d like to learn more about this topic, check out these materials:

* [Docker networking docs](https://docs.docker.com/network/)
    
* [Software Networking and Interfaces on Linux by Matt Turner](https://www.youtube.com/watch?v=EnAZB8GI97c&list=PL6US9lsaI2DPzPxfqIjllnlB5wNjKHcV3&index=4&ab_channel=MattTurner)
    
* [Container Networking from Scratch by Kristen Jacobs](https://www.youtube.com/watch?v=6v_BDHIgOY8&list=PL6US9lsaI2DPzPxfqIjllnlB5wNjKHcV3&index=9&ab_channel=CNCF%5BCloudNativeComputingFoundation%5D)
    

Thanks for reading. Cheers!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1712943056600/ce56c80a-a5a2-4237-b9b7-e8e5a69b497a.png align="center")