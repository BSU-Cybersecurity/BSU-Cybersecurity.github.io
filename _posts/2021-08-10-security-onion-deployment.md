---
layout: post
title: "Security Onion Deployment"
date: 2021-08-10 09:00:00 -0500
categories: [Training-SOC, SOC, Security-Onion]
tags: [soc, logs, guide, securityonion]
---

[Security Onion](https://securityonionsolutions.com/) is a a free and open platform for threat hunting, network security monitoring, and log management. It aggregates many popular cyber defense tools into one convenient package that can be used to quickly provide all the functionality needed for a security operations center.

# Installation

The following will guide you through how we were able to achieve a successful Security Onion deployment. Please note that depending on your specific needs you may need a different set up. The official documentation for security onion can be found [here](https://docs.securityonion.net/en/2.3/index.html) and is the source material for much of the rest of this guide.

## architecture
The main thing to consider before starting your deployment is what type of architecture you need. This will depend on what you plan on using Security Onion for and the assets you wish to monitor. We chose to go with a distributed model that is spread amongst two virtual networks in our Proxmox virtual environment.

![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/soArchitecture.png?raw=true)

A distributed deployment of Security Onion means that there will be three actual virtual machine instances of Security Onion: a manager node, a search node, and a forward node. The manager node is the core of the deployment and will be set up first. Next will be the search node which will allow us to load balane the elasticsearch and logstash services. These two nodes will be on the same network. Finally we will set up a forward node which acts as a sensor to detect network traffic. This forward node will be set up on whatever network you are wanting to monitor (this may be the same network that your manager and search nodes are on or a different network as is our case). When deciding this architecture the most critical part is determining how the forward node will communicate back to the manager node. We were able to do this successfully by setting up a site to site vpn between the router of the manager node's network and the router of the network the forward node is on.

## Deploying The Manager
The first step to deploying the manager is to obtain the Security Onion .iso which can be found [here](https://securityonionsolutions.com/software). Thankfully this iso will give you the ability to choose your deployment model and node type as part of the installation process so it can be reused for each node of the deployment. In order to facilitate this we stored the .iso to our NAS storage in proxmox by navigating to _netStore -> ISO Images -> upload_

Next you will want to create a new VM with the _Create VM_ button. Ensure that you are deploying the VM to the correct proxmox node based on your network constraints. In our instance the manager node will be in a SDN virtual netwrok which can route cross pve node traffic so it doesn't actually matter which node we deploy a manager on (however due to the constraints of tap interface on virtual switches this will become a concern for the forward node).

The rest of the VM creation is as follows:
* Name the VM something meaningful
* Select the Security Onion .iso from your chosen storage location
* Keep system options default
* Give it a disk of 1TB
* 8 core cpu
* 16GB of memory
* Choose the bridge that is appropriate for your chosen network architecture. We're using a bridge (_vmbr7_) that corresponds to our "provider" virtual network.
* Confirm the creation and boot the machine.

## Manager Configuration
On the initial boot you will be walked through a very basic install process. Make sure to choose the basic graphics mode for the installer (you can sometimes run into weird problems otherwise, this basic mode should always work). After that you will want to type yes to agree, create an admin user and password, and then let the system install.

Eventually the installation will be complete and you will be prompted to reboot the system (note, this can take a fair ammount of time, Security Onion installs a lot of services).

Once the system reboots you will be walked through the system configuration wizard. This process is the most vital part of the initial setup and will involve some choices that will be very impractical to change later on. Here are the steps that we took:

* Log in with the admin user we just created
* Say yes
* Install
* Choose distributed architecture (*note: press space to choose an option in these type of list selection menus)
* Choose manager
* Agree to Elastic's terms
* Set standard internet connection
* Set machine hostname (__THIS CANNOT BE CHANGED LATER. MAKE SURE TO WRITE IT DOWN AS IT WILL BE USED IN THE SENSOR AND SEARCH NODE CONFIGURATION__)
* Select your management NIC. There should be only one option, the one we set to our provider network bridge (_vmbr7_)
* Set a static IP
* Enter an unused IP in your provider network followed by the CIDR mask. In our case our provider network has 254 useable addresses so this would look like _#.#.#.#/24_
* 