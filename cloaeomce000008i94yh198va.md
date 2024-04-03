---
title: "Getting Started with Docker"
seoTitle: "Getting started with docker"
seoDescription: "This article summarizes docker's reason for being.  It explains how docker works and compares it to virtual machines."
datePublished: Sat Oct 28 2023 18:59:07 GMT+0000 (Coordinated Universal Time)
cuid: cloaeomce000008i94yh198va
slug: getting-started-with-docker
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1698517995990/a202cb6b-ee53-4340-97d1-bcb91b8c3577.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1698519277377/52255d9e-b22c-4364-b009-ddc8017132ef.png
tags: virtual-machine, docker

---

# Introduction

If you’ve been in the world of software for a while, you must have heard of `Docker`. But what exactly is it? Is it just another buzzword or some glorified esoteric concept? In this article, we will find out just what this “docker” is, why it is, and how it’s used.

# Why Docker?

First off, let’s talk about “why docker?”

When we develop software, it is typically intended that this software’s functionality should be replicable when ported or used in a different environment. By environment, I mean the machine running this software. Whether it’s a mobile device, a PC, or a bare-bones server, if the software is moved to a new machine, it is expected to work regardless of machine architecture, operating system (OS), and other system-specific intricacies. I mean your users shouldn’t have to be bothered about whether they can run your software on their OS. It should just work!

The process of getting the software to its intended audience or users is known as deployment or software delivery. This is a very crucial part of the software development life cycle. It is here that the software is ported to an environment where its stability can be assured. In this stage, the software dependencies are bundled. Tests are run and the application is deployed on a server. Usually, the server is a computer in some data center somewhere abstracted away from your user.

While creating the environment for your software, it is important that this environment is easily reproducible in a fast, light, and scalable manner.

Initially, applications were deployed on bare bones servers as services running in the background. The server is set up with application dependencies on a particular OS. The problem with this was,

1. **Reproducibility**: The time it takes to setup a server properly to serve an application is significant. One could argue that the process could be automated but the processes required to reproduce the application environment would still take a lot of time.
    
2. **Portability**: SInce the application is running on a bare bones server, it is not guaranteed that if there is need to change OS or server architecture the application environment can be reproduced in the same manner. Sometimes OSes have different commands and protocols for running the same jobs. This will require an extension of our scripts to reproduce our application environment in a different OS. Surely you wouldn’t want to do this every time you had the need to migrate a server.
    
3. **Scalability**: Running applications on a bare bones server is ok until you need to scale horizontally. You would have to buy new servers for this purpose and once again perform the required setup.
    

Seeing as this solution was not entirely sustainable, a new method was developed to create and isolate environments within our bare bones server. A server within a server if you will. This was the dawn of Virtual Machines (VMs). Using a virtualization technology called a Hypervisor, you could configure your server to split or share it’s system resources such as compute, memory, and storage. But here’s the catch, virtualization mimics an actual server in it’s entirety with the ability to host it’s own OS, have it’s own network structure, control it’s own process and much more. This is is not always a good thing as it causes portability and scalability issues as more system resources will be required to run different VMs. The fact that these VMs can do so much is a big blow on system resources. You could argue that you can define and allocate resources to VMs on creation to control resource usage but the problem still remains. The VMs comes pre-packed with a lot of bloatware and this increases demand for resources.

So, we have isolated environments now. We can package our applications into a VM to ensure it always runs as expected. But, we still have portability and scalability issues. Moving VMs is expensive. Remember we just want to be able to run our application on different devices or servers without worrying if it’ll function the same. I don’t know about you but having a dedicated VM to package my to-do list app seems unreasonable. Scalability is also hard for the same reasons. VMs just take up way too much space.

What if we could have the perks of a VM but require less resources? This my friends, is where containers come in. Containers solve our problems by isolating processes from a host machine and running them directly against the host’s hardware and OS.

![](https://i.pinimg.com/564x/c8/6f/c8/c86fc8917462ae5cac752ab33ad2ad42.jpg align="left")

Unlike VMs where you have to install an OS and treat it like it’s a dedicated server, the container just comes with what it needs to run a specific process say your todo list app. It does not require a hypervisor as virtualization is not needed (unless you’re on windows. We won't go down this rabbit hole though). Containers ship without bloatware and as such are very portable and scalable. You can run hundreds of these on your servers or VMs. Scale up or scale down quite easily because they are simply just light weight and do not have any OS footprint. They just have one job, packaging apps in isolated light weight environments. I should add that containers only work on Linux based OSes. If you’re not using one, Well, I'd like to ask why you hate life.

So why Docker? Well, Docker allows us to run containers in an exquisite way.

# What is Docker?

At its core, Docker is an array of tools that allows us to create and manage the interface that enables us to run containers on a host machine. It’s open-source and easy to use. It mainly consists of the docker client a CLI, and the docker daemon, `dockerd`. The docker client allows users to interact with the daemon and give it instructions. The daemon does all the heavy lifting. Creating and managing docker objects such as containers, volumes, networks, images, and plugins. It does this with the help of a different daemon, `containerd`. A container runtime responsible for ensuring that containers can be created and managed on a host device. It talks to the host systems Linux kernel so you don’t have to. Containerd manages all activities that involve containers and images.

If you’re wondering what images are, fret not. They are simply a list of instructions composed into layers for efficiency. They tell the container what it’s supposed to be like. Essentially defining their state. They are built using `Dockerfiles` another list of commands that tells the docker demon how to build the layers of an image. Dockerfiles are basically the base config for a docker image at least at the user level.

# How It’s Used

For the most part, what you’ll need to do with docker is create an image, run images in containers, and pull and push container images to container registries like docker hub or Amazon ECS (Elastic Container Storage). Lets go over how you can do these.

### Install docker

If you want to docker, guess what? You have to install it. So if you don't have it already [docker's documentation](https://docs.docker.com/get-docker) is your source of truth for this. Always refer to it for this purpose.

### Create an image

Creating a docker image is pretty straightforward. Just define your `Dockerfile`. An example of a Dockerfile is given below.

```dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

This dockerfile inherits from an already existing Alpine (a lightweight linux distro) image with node installed. It creates a working directory and sets up a node application to run once the image is run in a container. Each line in the Dockerfile creates a new layer for our image. This allows the docker engine to selectively build only layers that have changes when the Dockerfile is edited.

After creating your Dockerfile, you just have to run the build command and tag it for easy identification. Usually, you’d specify the full path to the image and append the tag-name to it.

```bash
docker image build -t <registry-username>/<image-name>:<tag-name> <path-to-dockerfile>
```

### Push & Pull Container Images

If you’re familiar with git, this is pretty much the same idea. We want to be able to share our images really conveniently so we store them in a registry on the cloud or in a personal local network accessible to those who would need the image. The choice is yours where to keep your images. This is where the portability of containers becomes apparent.

To share the image, you just need to run the push command and specify the image to push

```bash
docker push <registry-username>/<image-name>:<tag-name>
```

To retrieve images a similar command is run

```bash
docker pull <registry-username>/<image-name>:<tag-name>
```

There’s an authentication and authorization wall for docker images for security purposes that’s why the image name has a username that would be specific to the container registry you are using. Images can be labeled as public or private to prevent unwanted access.

### Run a container

This is where all the talk pays off. You finally get to use this beautifully isolated lightweight environment. To do this, you can pull the image first and then run it or just run it and docker looks for it on your system first then searches your specified container registry for the image.

```bash
docker run -- name new-container <registry-username>/<image-name>:<tag-name>
```

### Hands-On!

Now that we know what docker is and how to use it, lets run a simple nginx server with it!

```bash
docker run --name web-server -p 80:80 nginx
```

Yep, that’s it. This command will look for an image named nginx on your system and if it doesn’t find it, it’ll search your configured container registry. When it finds the image it creates the container space and names it `web-server` and also maps traffic to port 80 on the host to port 80 in the container. That way, HTTP traffic from the host can get to the container. All you have to do now is wait for the image to pull and run

![Run an nginx container](https://lh7-us.googleusercontent.com/l60XY8RPJMmdGwtGvjSXROfeYi88zke25YYOkYaAp-t8YbjC1NBeQ49gteoFO7FQcJpipQvAZ3c6jDxb47NcL0uV3zscfv6q_5yDDuU16GYa_Hgjw64LKMBNm0jDkrk-9R3FKImIGMIo6BTTlow-U3Y align="left")

When it gets to the stage in the picture above, you can open up your browser and check out [localhost](http://localhost)

![](https://lh7-us.googleusercontent.com/9DeaJ3WDIK0QTNgksOBiqaFyRAMPzXx26UMVnz8gwoFINowE7bNpBqX-b4uZrQEQ-Xj_iemIshKoMKMAydzRBo3SwXvkcFs4-fiM-4m8mRl42FHOcDTpVkh9STSX3hl1f7ymBFYgqplkLLlQBzm_t5A align="left")

Now press `ctrl + c` in your terminal to stop the container. If you run the command again you’ll notice it’s much faster because it doesn’t have to perform the pull step.

# Conclusion

Now, my friends, we have come to the end of the little docker exploration. We looked at what docker is and why it was created. Comparing it to virtual machines and found out how containers make it easier to deliver applications at scale. Finally, we talked about how you can use it, we deployed a simple nginx web-server container to demonstrate docker’s utility.

I do hope you found this insightful, don’t hesitate to share if it helps you!