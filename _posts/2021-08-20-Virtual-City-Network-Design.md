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

## Outer Layer
This first outer layer consists of the _192.168.0.0/16_ subnet that runs across a physical switch and gets routing from a physical PFSense router. This purpose of this network is to act as the internet for the Virtual City as well as provide management access of the Proxmox nodes to Virtual City admins. All Proxmox nodes have access to this network via their physical NICs and can pass access to the virtual machines that they host with their built-in vmbr0 bridges.

## Inner Layer
The inner layer consists of a multitude of virtual _10.X.X.X/X_ subnets for the client, provider, and attacker networks.

### Type 1 Nets (vnets)
Both provider and attacker networks utilize type 1 networks. These are vnets built using Proxmox's SDN functionality (which you can read about [here](https://bsu-cybersecurity.github.io/posts/proxmox-networking-SDN/)). These are the most versatile type of networks we can create as they can network VMs between different physical Proxmox nodes. For instance, you could have VMs hosted on three different proxmox nodes all on the same LAN. In our diagram these Type 2 nets are the vmbr5 & vmbr6 bridges.

> *note: Every network device in proxmox is named as a bridge (vmbr#). This can get confusing as not all of these "bridges" are the same type of device. Some are traditional network bridges (such as the built in vmbr0 bridges present on every proxmox node that give access to Layer 1) while others might be vnets.

### Type 2 Nets (Open vSwitches)
Due to our usage of Security Onion all clients must use Type 2 networks. These are built using Open vSwitches. The reason for this is that Security Onion sensor nodes must get network traffic from a tap port on the network's switch. Unfortunately, Type 1 netowrks do not support this functionality. Another quirk that needs to be considered is that these Type 2 networks cannot talk directly cross node like SDN vnets. If a VM in a Type 2 net needs to talk with a network in another proxmox node both networks will need access to Layer 1 and talk across it. The Open vSwitch in our diagram is vmbr7.

# Service Communication
Due to the nuances of our multiple network types and the way the services we use want to talk with each other we had to go through some serious trial and error before we found a setup that worked.

We're using Security Onion (SO) as our primary security service platform and to replicate how a real MSSP would set up its production environment we went with a distributed deployment. This means that there are some components of SO that are hosted in the provider network (manager, search) and one that is hosted on the client's network (sensor). The sensor must be able to shuttle client network and host data back to the manager on the provider network.

![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/vpnDiagram.png?raw=true)

To implement this functionality we tried a few different configurations but eventually got it to work by creating an OpenVPN server on the provider network's router and an OpenVPN client on the client network's router. The client router's LAN interface is connected to vmbr7 (Type 2, Open vSwitch) and its WAN interface to vmbr0 (Layer 1). The provider router's LAN interface is connected to vmbr5 (Type 1, vnet) and its WAN interface is connected to vmbr0 (Layer 1).