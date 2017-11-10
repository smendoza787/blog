---
published: true
layout: post
title: "Fiddling with Docker"
author: "Sergio"
permalink: /2017/11/06/fiddling-with-docker
date: '2017-11-06 16:16:01 -0600'
---

Ever since I've started programming, I'm always eager to jump on the hot-new-technology bandwagon whenever I see something mentioned a lot. Sometimes it can get a little overwhelming having to jump from tech to tech to fast, and other times I actually welcome the change.

Docker is one of those technologies that is bringing about a welcome change and taking the world by storm. When I first started deploying my apps to cloud services like Heroku, I noticed that things would rarely ever go smoothly. I would spend all of my time developing an application on my own machine, with all of its installed programs and dependencies that it took to get running.

When deploying applications, the server that you’re deploying to usually has a completely different configuration. So the app that ran perfectly on your machine, might not work so well on an array of other completely different machines with different configurations.

Enter Docker!

# What is Docker?

Docker is a containerization platform that packages your application and all its dependencies together in the form of a docker container to ensure that your application works seamlessly in any environment. The keywords of Docker are develop, ship and run anywhere. The whole idea of Docker is for developers to easily develop applications, ship them into containers which can then be deployed anywhere.

# What are Containers?

Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package. By doing so, thanks to the container, the developer can rest assured that the application will run on any other Linux machine regardless of any customized settings that machine might have that could differ from the machine used for writing and testing the code. Containers are not a new technology by any means; however, Docker has made the process of using containers a lot more simple.

# Why Should I Use Docker?

Bringing it down to a fundamental level, Docker allows applications to be isolated into containers with instructions for exactly what they need to survive that can be easily ported from machine to machine. Virtual Machines provide the exact same thing, with numerous tools like Puppet and Chef already in existence.

Docker has a more simplified structure compared to both of these, however, the real area where Docker trumps VMs is resource efficiency.

Lets so you have an application in charge of running 20 different microservices. With Docker, you can simply deploy 20 Docker containers on a single virtual machine in the cloud. Without Docker, you would have to run 20 different virtual machines with 20 different operating systems with at least the minimum resource requirements available for each VM.

Assuming you're going to need the minimum 256MB VM, thats 5GB of RAM with 20 different OS kernels managing resources. With Docker, you could run all 20 instances of a Docker container on a single VM with a single operating system at significantly lower performance costs. Once you push that Docker image to DockerHub (Sort of like Github for Docker images), anyone can download that image with a single command, and run it on their own machine with another command. Amazing!

In my next post I plan on building a simple Docker container for a static website and deploying it to DockerHub so you can observe how simple shipping Docker containers can be.