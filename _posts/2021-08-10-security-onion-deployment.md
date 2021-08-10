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

A distributed deployment of Security Onion means that there will be three actual virtual machine instances of Security Onion: a manager node, a search node, and a forward node. The manager node is the core of the deployment and will be set up first. Next will be the search node which will allow us to load balane the elasticsearch and logstash services. These two nodes will be on the same network. Finally we will set up a forward node which acts as a sensor to detect network traffic. This forward node will be set up on whatever network you are wanting to monitor (this may be the same network that your manager and search nodes are on or a different network as is our case). When deciding this architecture the most critical part is determining how the forward node will communicate back to the manager node. We were able to do this successfully by setting up a site to site vpn between the router of the manager node's network and the router of the network the forward node is on.

## Deploying The Manager
The first step to deploying the manager is to obtain the Security Onion .iso which can be found [here](https://securityonionsolutions.com/software). Thankfully this iso will give you the ability to choose your deployment model and node type as part of the installation process so it can be reused for each node of the deployment. In order to facilitate this we stored the .iso to our NAS storage in proxmox by navigating to _netStore -> ISO Images -> upload_

Next you will want to create a new VM with the _Create VM_ button. Ensure that you are deploying the VM to the correct proxmox node based on your network constraints. In our instance the manager node will be in a SDN virtual netwrok which can route cross pve node traffic so it doesn't actually matter which node we deploy a manager on (however due to the constraints of tap interface on virtual switches this will become a concern for the forward node).

The rest of the VM creation is as follows:
* Name the VM something meaningful
* Select the Security Onion .iso from your chosen storage location
* 