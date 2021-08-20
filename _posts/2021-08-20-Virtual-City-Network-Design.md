---
layout: post
title: "Virtual City Network Design"
date: 2021-08-20 09:00:00 -0500
categories: [Training-SOC, SOC, Virtual-City, VPN, Networking]
tags: [soc, guide, securityonion, virtual-city, networking]
---

# Overview
One of the core projects of the Institute for Pervasive Cybersecurity is the Vritual City. This is a collection of fully virtualized networks that aim to model real world businesses/governments/utilities, their managed security service providers, and the attackers that are constantly attempting to steal data, extort, and otherwise wreak havoc on them. 

![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/vcityNetwork.png?raw=true)

Above you can see a simplified diagram that shows the core structure and functionality of the Virtual City. 

* Multiple clients can be constructed to model many different real world scenarios ranging from small businesses all the way to enterprise networks. 
* The provider network is set up to provide SIEM & SOAR capabilities along with log storage, vulnerability scanning, and a myriad of other defensive security services.
* Attackers can be set up to represent threat actors targeting the various clients. These can range from modeling a cyber criminal on a single Kali machine all the way to APTs with advanced command and control structures.

# Platform
In order to build this virtual city we needed a platform that would be able to host many virtual machines efficiently as well as allow for complex and varried virtual networking implementations. For this we decided to use a ProxmoxVE cluster. Compared to other self hosted solutions, like Openstack, a Proxmox cluster proved relatively easy to build, use, and maintain according to our use case. 

Here are some perks of proxmox we found:
* very simple to install and join new nodes to the cluster 
* great resource monitoring at the datacenter, node, and VM levels
* the ability to use traditional virtual bridges, software defined virtual nets to network VMs cross-node, and vSwitches to provide tap ports
* great integration with a multitude of network storage options so every node can have access to a large pool of storage for both images and VM disks

# Network Structure
The easiest way to conceptualize the network of the virtual city is in 2 layers with the second inner layer containing 2 distinct types of networks. 