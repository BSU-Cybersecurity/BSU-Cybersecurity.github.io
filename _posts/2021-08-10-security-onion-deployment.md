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

> IMPORTANT:
> Somewhere in this process you will have been asked to create a password for a `soremote` user account. Make sure to save this password as it will be used when joining search and forward nodes to the manager.

* Log in with the admin user we just created
* Install
* Choose distributed architecture (*note: press space to choose an option in these type of list selection menus)
* Choose manager
* Set standard internet connection
* Set machine hostname (__THIS CANNOT BE CHANGED LATER. MAKE SURE TO WRITE IT DOWN AS IT WILL BE USED IN THE SENSOR AND SEARCH NODE CONFIGURATION__)
* Select your management NIC. There should be only one option, the one we set to our provider network bridge (_vmbr7_)
* Set a static IP
* Enter an unused IP in your provider network followed by the CIDR mask. In our case our provider network has 254 useable addresses so this would look like `#.#.#.#/24`
* Enter the IP address of the provider network router
* Add two DNS server addresses to the front of this list (`providerNetworkRouterIP, 1.1.1.1`)
* Leave default search domain
* Choose direct internet connection
* Choose a manual patch schedule
* Keep default home networks
* Install all components
* Keep default docker IP range
* Enter an email addres for an administrator account (*note: this does not have to be a real email, it will be used to log into the security onion web interface)
* Set your admin password (*note: The first time using this account after setup there is a chance that the account will not authenticate properly. If that is the case you will simply have to create a new account with `sudo so-user-add someone@example.com`)
* Choose IP for web interface access
* Choose default NTP servers
* Say yes to running so-allow
* Designate an IP range from which analysts can access the Security Onion web interface. We chose our entire provider network (`#.#.#.0/24`)
* Review all configurations and say yes to start the final install process

When complete, restart the machine and log in with the admin account you created on the initial boot. Use `sudo so-status` to check on the status of all the services as they start up (this can take a few minutes). Once all services are running you should be able to access the web interface from a machine sitting in whatever network range you designated access by browsing to the static ip you set for the manager node in your web browser.  

## Deploying the Search Node
The process for deploying the search node will be very similar to the manager. It will be sitting in the same network as your manager node and you will only have to make a few changes:

>IMPORTANT: Make sure to chose a host name that represents that this is the search node
* Once you select the distributed deployment model, instead of choosing manager choose search
* Join the search node to the manager using the manager's hostname, static ip, and when prompted enter the password for the soremote account

Once the install process is complete and the machine reboots log in with the admin account and check `sudo so-status`. Once all the services are running check the Security onion web interface. In the left bar there should be a page called _Grid_ that you can browse to. If you can see the new search node in the grid you have successfully deployed this piece of security onion! 
> Make sure to scroll to the right in the grid page to see if there are any fault alerts. If there are, wait a few minutes to see if they clear up. These can sometimes happen as the node is booting up.

## Deploying the Forward Node
The forward node (or sensor) will be slightly different than the others as it will not reside in your service provider network but rather in the clients network that the provider is providing security monitoring service too. In our case that network is a separate virtual network in our proxmox virtual environment.

If you're still in the planning phase of your architecture you will need to make sure that however you set up your client network you have the ability to mirror network traffic to the sensor node which will be deployed there. This is traditionally done by creating a mirror port (sometimes called a span port or tap) on a physical switch. In our case, since we're using a virtual network, we specifically chose to use an [Open vSwitch](https://www.openvswitch.org/) device as it allows for the creation of taps. 

You will install and configure the sensor mostly the same way as you did with the search node except for these changes:

* When setting up the machine make sure that it has two NICs (one for its normal management interface on the client's network and another that will be attached to the tap). To attach the tap to the second interface in proxmox enter `ovs-vsctl -- --id=@p get port tapXiY -- --id=@m create mirror name=span1 select-all=true output-port=@p -- set bridge vmbrZ mirrors=@m` where `X` is the number of the machine in proxmox, `Y` is the number of the interface you are connecting the tap to, and `Z` is the number of the Open vSwitch bridge.
>IMPORTANT: Make sure that the management NIC has a direct way to communicate with the ip address assigned to the manager node's management NIC. We were able to achieve this using a VPN from the client's router to the provider's router. You can also do with using another router on the client's LAN serving as a VPN appliance (*note that this will necessitate 3 NICs on the sensor instead of 2). You can read more about our network design [here]()
* After choosing the management interface make sure that you choose the second interface as the tap interface
* Once you select the distributed deployment model, instead of choosing manager choose sensor
* Join the sensor node to the manager using the manager's hostname, static ip, and when prompted enter the password for the soremote account

