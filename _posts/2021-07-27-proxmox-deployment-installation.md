---
layout: post
title: "Proxmox Deployment - Installation & Configuration"
date: 2021-07-27 09:00:00 -0500
categories: [Training-SOC, Proxmox]
tags: [proxmox, deployment, soc, guide]
---
# Initial Install
After booting from the flash drive you should be greeted with a splash screen displaying the proxmox logo. Press enter to select the 'Install Proxmox VE' option.

You may have to wait a bit while the system loads the initial installation configuration. Once this is complete you should see graphical environment reappear showing an end user license agreement which you should accept.

## Drive Selection
Select from the drop down menu at the bottom of the screen the drive you wish to install the Proxmox VE operating system onto. This should be whichever virtual disk you set up in a RAID 1 configuration in the previous section.

## Locality Settings
Set your country as the United States, your time zone as America/Boise, and your keyboard layout shoud default to U.S. English.

## Administration Settings
Set the root password for the node and add the management email.

## Network Settings
* __Management Interface__: This should default to the 'eno1' NIC on Dell machines. If not, choose whichever NIC you configured for the device that is plugged into the switch and has internet.
* __Hostname__: The naming convention we have established for our cluster is 'pve' (proxmox virtual environment) followed by the number denoting the order in which it was added. Check the current node names from the proxmox web interface in Datacenter -> Cluster to choose a 'pve#' hostname one above the current highest node. The full hostname should be 'pve#.trainingsoc.edu'.
* __IP Address__: The addressing convention is sequentially increasing from 192.168.0.50/16 upwards (except for two nodes which have 192.168.240.x addresses). Check the current node addresses from the proxmox web interface in Datacenter -> Cluster to choose an IP one above the current highest node.
* __Gateway__: Our gateway is 192.168.0.1 
* __DNS Server__: Our DNS Server is 192.168.0.1 

## Review and Install
After reviewing your configuration you should be good to install. This can take a while (~10 min. depending on hardware). After the install is complete the system should restart. Rember to remove the flash drive 

After the reboot, if you are greeted by a login terminal displaying your configured IP address, you have successfully installed Proxmox VE!

# Configuration
## Connect
The first step to configuring your new Proxmox node is to connect to its web interface. To do so enter `https://NODE_IP:8006` into your web browser where `NODE_IP` is the IP address that you configured in the previous section (make sure that you are on the same network as the node).

If done correcctly you will be prompted that the website may be unsafe due to a lack of an SSL cert. This is fine, just proceed anyways.

You should now be able to log in with the username `root` and the password that you configured in the previous section.

## Cluster
You will now want to set up clustering so that you can take advantage of all the datacenter features that proxmox has to offer. This includes the ability to monitor the resources of many nodes at once 

# [Next Section: Network](https://bsu-cybersecurity.github.io/posts/openstack-deployment-network/)
