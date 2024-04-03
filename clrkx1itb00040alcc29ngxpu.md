---
title: "Introduction to Computer Networks -  Subnetting & IP Addresses"
seoTitle: "Computer Networks, IP address & subnetting"
seoDescription: "A computer network is a system of devices connected by wire or radio signals to transmit data. IP address identifies devices on a network"
datePublished: Fri Jan 19 2024 17:29:50 GMT+0000 (Coordinated Universal Time)
cuid: clrkx1itb00040alcc29ngxpu
slug: introduction-to-computer-networks-subnetting-ip-addresses
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705684863633/9438ddab-e524-4534-a572-9e749bf31a26.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1705685374011/db9d02e2-02b6-4df2-9c9f-2d37a2aa3980.png
tags: networking, ip-address, subnetting

---

# **Introduction**

Computers need to be able to communicate with each other directly without having to perform actions such as copying to disks and transferring to different systems. Computer networks exist to make this communication seamless. A computer network is a group of computers that are linked or connected by physical means such as cables or non-physical means like radio frequency waves (think WiFi or Bluetooth) such that they can communicate and share resources. The Internet is the largest computer network that currently exists. It is a massive mesh network of machines sharing information. For these networks to function properly, certain fundamental questions need to be answered.

1. How do we create a network?
    
2. How do we identify the nodes (computers) on the network?
    
3. How do we define the boundaries of the network?
    

In this article, we will be answering these questions. We will learn about,

1. Subnetting
    
2. TCP/IP addresses
    
3. Network devices
    

# How to create a network

A network can be created in various ways ranging from wired methods to wireless methods. For a device to be able to join a network, it must possess the required hardware, a Network Interface Card (NIC). These cards have unique IDs called Media Access Control Addresses (MAC addresses). This address is used to uniquely identify the machines or computers on the network. Modern devices usually ship with at least one NIC either for wired (Ethernet) or wireless (WiFi) network connections.

## Network Devices

To manage the nodes of a network efficiently and route data packets effectively, dedicated devices are used to establish pathways for data packets. Data packets are an encapsulation of information being sent from one service on a network to another. It typically contains identity information such as the source port address (port number and IP address) and destination port address. The major network devices include

1. Hubs
    
2. Switches
    
3. Bridges
    
4. Routers
    

### Hubs

![](https://lh7-us.googleusercontent.com/hk3WAivQU7aVpfJ05MCmLoltT6LrQLQZ222pa72R21Hqs3ewsaYS9FIqBcQQ4gpK8h9yGgXPkVyc3fBziPlIoEQgqfxog_OpGEO9CusdM1jZODIcDXS9f7fpWoRoMWVPmMY74iF62OQFPZnvIL6VEws align="left")

Hubs broadcast data to all nodes on a network. They make use of ethernet ports. A computer network is created when devices connect to these ports using an ethernet cable. The hub is not very smart though. From the diagram above, if computer A sends a data packet to computer D. The result will be that the hub will intercept the packet and just distribute it to computers B, C, and D. It is a layer 1 networking device. It does not perform any kind of inspection on data packets to discover the destination node. Because of this, It cannot identify the nodes in the network uniquely and so has to broadcast all sent data packets even if it is meant for a single node.

### Switches

![](https://lh7-us.googleusercontent.com/lJXlrfqZ8oOqwr4V6MnVSt-0-9OlxEGzy4Cv4ULzruSt0OG4edXRIN24vxsEh9BKKlJL6a9QF0R91dSxaDvbHTjflcVT13F15VO5dwRa9nMOPHwy8AVP9eVVG5lCo__87S_KOOGSZsevOo-_YeUjUkc align="left")

These devices are similar to hubs but are smarter as they are capable of identifying nodes uniquely using their MAC addresses. They inspect data packets to find out destination MAC addresses and route data accordingly only to the target intended. In this case, when computer A sends a packet to computer D, only computer D receives it. This makes them a layer 2 network device. Switches keep track of MAC addresses using a neat table which is updated when devices are added to the network and requests are made to or from them. This helps to keep network traffic clean and efficient as opposed to Hubs which put a lot of stress on the network because unnecessary traffic is always occurring. Switches are also better for security as only the intended node receives the data being sent.

### Routers

A router is a device that exposes an internal or private network to the public internet facilitating data transfer between the private network and the public internet. It is a layer 3 device because it makes use of IP addresses to route data packets to their destination. It does this using a neat method known as Network Address Translation (NAT) to map public port addresses to private port addresses. It maintains a NAT table to fulfill this role. This table allows the router to allow only desired packets from known sources or deny access from untrusted sources.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705431780074/5537a0e5-557f-4437-aad0-0b9c6981d7d8.png align="center")

The above figure shows how a router exposes 2 private local area networks to the internet. Inbound rebound request must first hit the router using its public IP, the data packet is unwrapped in search of a destination port address and then a NAT is done to find out which node that address belongs to. It then routes the packet to its intended recipient.

### Bridges

![](https://lh7-us.googleusercontent.com/3WhXznaWhRTXrciOkC5q22qIGrQwAl_KhQhDSr9XB0JY_Ypc0unRqmlZPB8_J6zNvcc4pBALhwEk0DNklsQFnCzaQTTXQlOIkPSTBW6TRWpbl5QQR0D67_G68UzZmDw2RynognVz1QCvYEy_vLFo6SU align="left")

A Bridge allows a network to be extended by connecting two distinct networks via their hubs, switches, or routers. It fuses two distinct networks into one single network. The bridge is not a dedicated device on its own but rather a capability of most routers, switches, or hubs. That is to say, a router can be turned into a bridge. In the diagram above, computers A, B, C, and D effectively make up one network. The two hubs are connected via a bridge (wired or wireless) connection. One hub assumes the role of the bridge while the other retains the primary role of the hub. The same thing applies to the other network devices. In this case, the hub still broadcasts to all its ports but routes through the bridge for nodes not directly connected to it. Switches and routers will also perform the same but with a more refined routing pattern as already discussed.

# **IP Addresses**

The TCP/IP (Transmission Control Protocol/Internet Protocol) framework was created as a protocol that would define how communication over an IP network (network using IP addresses) should happen. It defines an IP address as the identity of a node on a network. An IP Address is a group of 32 bits (1s and 0s) used to identify any machine on a computer network whether it's the internet or a LAN (Local Area Network). To make it human-readable, the groups of 8 bits are converted from binary to decimal. So instead of this stuff, `00000000.00000000.00000000.00000000` you would see this, `0.0.0.0`. Before a device can access an IP network, it must have this address. These IP addresses are unique per network (think of it like an ID of sorts).

How do devices get these addresses though?

## IP address assignment

When a computer joins a network, the network device (usually a router) identifies the computer by its MAC (Media Access Control) address and typically an IP address from the router's subnet is also assigned via DHCP (Dynamic Host Configuration Protocol) and mapped to the MAC address. The MAC address is unique per NIC (Network Interface Card). The NIC allows the computer to have the physical capacity to join a network and machines can have multiple of them. For example, a computer could have an Ethernet NIC and a WiFi NIC. The method of connection to the network (ethernet or WiFI) would determine which MAC address gets read by the network device (router, switch, or hub).

You’re probably wondering why we need an IP address if the MAC addresses are already unique per device and the answer is subnetting which we will go into in later sections. IP addresses provide a better interface or basis to carry out subnetting.

## Types of IP networks

1. **Private or local networks**: These networks are not exposed to the public internet. The only way of accessing them is by physically connecting to the network locally via ethernet or WiFi. This makes this network more secure. They make use of private IP addresses which are reusable in other private networks.
    
2. **Public networks**: This is the internet. IP addresses are unique here as this is just one giant network. It is accessible to any device with a public IP address.
    

## Classification of IP addresses

IP addresses can be classified into the following:

* **Public IPs**: These IPs are distributed by organizations that maintain the Internet (ICANN - Internet Corporation for Assigned Numbers and Names and IANA - Internet Assigned Numbers Authority). The distribution is done in a way that allows say a company like AWS (Amazon Web Services) or an Internet Service Provider (ISP) to own particular blocks of addresses (subnets) which they can in turn distribute to users who wish to access the Internet through them. An example of an ISP would be MTN or Verizon. These IPs are the only way to communicate on the internet.
    
* **Private IPs**: These addresses are assigned to computers when they join a private network. They cannot be used to communicate over the internet and only have agency within a particular local network. The router typically uses a protocol such as DHCP (Dynamic Host Configuration Protocol) to assign an IP address to the computer when it joins its network. Settings can be made to tell your router to always assign you a particular IP from its available pool. The Internet Assigned Numbers Authority ([IANA](https://www.iana.org/)) reserves the following IP ranges as private IPs. They cannot be reached from the public internet.
    
    * `10.0.0.0` to `10.255.255.255`
        
    * `172.16.0.0` to `172.31.255.255`
        
    * `192.168.0.0` to `192.168.255.255`
        
    
    If you check the IP address of your local machine or remote server using the command `ip a` for Linux-based operating systems and `ipconfig` for Windows systems, you would find that the `inet` (IPv4) addresses fall within the above ranges. They are local to whatever network your device is in.
    

# **Subnetting**

We’ve mentioned this concept a couple of times already. Let’s find out what it means. A Subnet is a defined sub-range of an Internet Protocol network. To properly define the boundaries of a network, the concept of subnetting was created. It operates by first defining the range of possible IP addresses that could exist on the Internet from `0.0.0.0` to `255.255.255.255` (i.e., `00000000.00000000.00000000.00000000` -&gt; `11111111.11111111.11111111.11111111` in binary). Typically the first IP in a subnet marks the IP of the network itself while the last IP in the range marks the broadcast IP which can be used to stream data to all nodes in the network. These two IP addresses are reserved in any defined subnet and cannot be assigned to a node on the network. In the case of the Internet, `0.0.0.0` would represent the entire internet while `255.255.255.255` would be the broadcast IP for the entire internet.

## Methods of Subnetting

Two major ways subnetting has been achieved are outlined below.

* Class-based IP ranging and
    
* Classless IP ranging or CIDR (Classless Inter-Domain Routing)
    

To distribute IP addresses in an orderly manner, a series of calculations must be performed. These calculations are the basis for these subnetting methods.

### **Class-based IPs**

These types of IPs are no longer typically used. This is because of the sheer size of the internet and the incredible number of devices that are connected to it. Remember the definition of an IP address? It is a number consisting of 32 bits. This means that there are a finite number of IPs that can go around. These class-based IPs however encourage the wastage of these IPs and are no longer adopted.

The different classes include:

| **Class** | **Description** |
| --- | --- |
| Class A | These can accommodate over 16 million hosts |
| Class B | These can accommodate over 65 thousand hosts |
| Class C | These can accommodate over 254 hosts |

Those are some very big numbers and you can see why they encourage the wastage of IPs. It would be quite the feat to completely exhaust the available hosts any of these classes provide except the owners are an incredibly large organization like an ISP or a cloud services provider.

If you're wondering how these numbers were calculated or what they mean, relax, we are getting there.

### **Classless IPs (CIDR)**:

These IPs are distributed more frugally. This prevents wastage of IPs as organizations can now be given IPs for only the number of hosts they own. So if you needed just 10 or 20 hosts, you would not be gifted 254 as would be the case for Class C IPs. This is the method used for distributing IPs nowadays. It is also referred to as CIDR (Classless Inter-Domain Routing).

## **How to calculate an IP address**

As we found out earlier an IP address is of the format `*.*.*.*`. Four numbers between 0 and 255 inclusive; separated by three periods.

In case you're already wondering, the range 0-255 is not just some arbitrary range. This range signifies 256 possible entries which is also 2^8 which is the maximum number that can be contained in 8 bits or an octet when converted to binary. In other words,

| **Decimal** | **Binary** |
| --- | --- |
| 255 | 11111111 |

Alright, things seem to have started getting complicated but hang in there, it's really all quite simple.

An octet is a group of 8 bits. In most machines, it would also be equivalent to a byte.  
A Bit is a single binary digit.  
A binary digit is a number in the base 2 counting system which comprises 1s and 0s. The language of machines.

With that out of the way, we can see how the IP address 10.15.250.0 can be transformed to its true form, `00001010.00001111.11111010.00000000` by converting each decimal number in the IP address to its binary form. You can learn how to do this conversion (Decimal -&gt; Binary) by making simple searches on Google or by using any conversion tool you fancy.

We have successfully decoded the true form of an IP address.

### **Networks IPs**

We have already established that the internet is a giant wide area network (WAN). This is primarily because it consists of millions of other nested WANs which in turn consist of other nested networks and so on. But networks are not machines so how does the internet know how to identify them and route data packets accordingly?

The answer is quite simple, If you think of a network as a hierarchy tree of sorts, at the top node there will always be an IP that tells the internet that a certain number of IPs exist as its children. In this case, the top node becomes the network IP while its children become the host IP.

![](https://lh7-us.googleusercontent.com/kjLCbKKH9IOXqx0Ic_1wdZF69-M1k8q4Qg3ige8H0TYcwn1z5wjLk8OVZF65qWuNrGLI-BX3c12Cfj85Ij-UUU96V4YM30_nuicHIfqQ6ez7h0tTNHz6ol66iE4LBBb2jBKbiBS4bcAJPuMtsZe4q9g align="left")

At the top of this network hierarchy sits the entire internet with an IP of 0.0.0.0 while every other IP is a child of the internet's IP address. All `[(2^32) - 2]` of them. The subtraction excludes the internet's network IP (0.0.0.0) and its broadcast address (255.255.255.255).

As has already been stated, the broadcast address is the last available IP in a network. This is why the internet has a broadcast IP of 255.255.255.255. Remember, the numbers don't get higher than 255.

Hopefully, things are starting to add up now.

Remember when we were talking about distributing IPs and hosts? This is what it means to be given a group of IPs. In reality, The first IP in the block you were given becomes the top node of your network while the rest save for the last serve as possible host IPs.

The grouping of IPs is done and calculated via a routine called netmasking or subnet masking. A netmask simply indicates the number of host IPs that a given network has. It is used to define a range of consecutive IP addresses that could potentially form a network.

A netmask is typically defined using the format, `a.b.c.d/e` where `a` to `d` are integers in the range of \[0-255\] and `e` is the netmask or subnet mask which ranges from \[0-32\] (for the 32 bits available in an IP address). However, in practice, the netmask range would typically be within \[8-32\]

Using the class-based system of IP address distribution, only the following netmasks would be used:

* /8 for class A
    
* /16 for class B and
    
* /24 for class C
    

This is the reason for the outrageous number of hosts allocated to these class-based IPs. However, the classless model allows all integers between \[0-32\] to be assigned as netmasks. This allows IPs to be grouped in smaller amounts by increasing the netmask beyond 24.

## How to Calculate CIDR Ranges

Suppose we were given a particular netmask and IP address say 193.16.20.35/29. If we were asked to calculate the Network IP, number of hosts, range of IP addresses, and broadcast IP from this subnet, the following procedure should be followed.

1. Calculate the wildcard. The wildcard shows the available IP address slots. It is obtained by first converting the IP to its binary form, subtracting the netmask from 32, and then counting from the rightmost bit of the binary IP until you reach the difference calculated. At this point, Assign a value of 0 to all bits on the left-hand side and 1 to all bits on the right-hand side. The wildcard is then inverted to obtain the netmask.
    
    ```bash
    decimal IP ==> 193.16.20.35
    
    // convert to binary
    binary IP ==> 11000001.00010000.00010100.00100011
    
    // Calculate netmask difference
    difference = 32 - 29 = 3
    
    // Count 3 from right to left on the binary IP
    ==> 11000001.00010000.00010100.00100 ^ 011
    
    // the ^ symbol shows the 3rd position mark
    // Now assign the right and left bit values of 1 and 0 respectively
    
    ==> wildcard ==> 00000000.00000000.00000000.00000 ^ 111 ==> 0.0.0.7 
    
    // The netmask can also be represented as the inverse of the wildcard
    ==> netmask ==> 11111111.11111111.11111111.11111   ^      000          ==> 255.255.255.248
                   |---------network part-----------|    |---host part---|
    
    // 7 is obtained by converting the binary number 111 on the right hand side of the wildcard to decimal
    // this means we have only 7 available slots for this netmask
    // This also means that the number of hosts = 7 - 2 = 5
    // where 2 signifies the network IP and broadcast IP
    ```
    
2. Identify the network IP. To do this, first of all, get the binary form of the given IP. calculate the netmask difference and count from the right again up to the calculated difference. Then change all bits on the right to 0.
    
    ```bash
    binary IP ==> 11000001.00010000.00010100.00100011
    
    // Calculate netmask difference
    difference = 32 - 29 = 3
    
    // Count 3 from right to left on the binary IP
    ==> 11000001.00010000.00010100.00100 ^ 011
    
    // convert right side to 0 bits
    ==> Binary Network IP ==> 11000001.00010000.00010100.00100 ^ 000
    
    // removing the ^ and merging both sides
    ==> Binary Network IP ==> 11000001.00010000.00010100.00100000
    
    ==> Decimal Network IP ==> 193.16.20.32
    
    ==> Network IP ==> 193.16.20.32
    ```
    
3. Find the broadcast. To do this, count serially from the Network IP to the number of available slots.
    
    ```bash
    ==> Network IP ==> 193.16.20.32
    
    ==> No. of IPs in network = 7
    
    ==> Broadcast IP ==> 193.16.20.38 // The last IP on the netmask
    // where 38 = 32 + 7 - 1
    // remember the network IP is inclusive in the no. of IPs in the network
    ```
    
4. Find the range of IPs.
    
    ```bash
    // With the information we have so far we can deduce that we have a range of 5 addresses (hosts)
    ==> min host address ==> 193.16.20.33
    ==> max host address ==> 193.16.20.37
    ```
    
5. The answer to the question would therefore be:
    
    ```bash
    Given IP netmask ==> 193.16.20.35/29
    
    ==> Network IP ==> 193.16.20.32 // The first IP on the netmask
    
    ==> min host address ==> 193.16.20.33
    
    ==> max host address ==> 193.16.20.37
    
    ==> Broadcast IP ==> 193.16.20.38 // The last IP on the netmask
    
    ==> Number of hosts = 5
    ```
    

Congratulations! You can now make IP address calculations. It should please you to know that you can find numerous online tools to do these calculations for you but it’s always nice to know what’s going on behind the scenes.

## **Conclusion**

In this article,  we tried to answer three questions,

1. How do we create a network?
    
2. How do we identify the nodes (computers) on the network?
    
3. How do we define the boundaries of the network?
    

We showed how a network might be created and managed using network devices like hubs, switches, bridges, and routers. We explored MAC addresses and IP addresses as identification methods for nodes on a network. We then dived into subnetting as a means to define network boundaries in an IP network.

These concepts are important to Network, Cloud, and Operations Engineers who are typically tasked with setting up networks for infrastructure. Whether on-prem or in the cloud, it is important to be able to split up and secure your infrastructure by employing subnetting and usage of appropriate network devices.

Thanks for reading!