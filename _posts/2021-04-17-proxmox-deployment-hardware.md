---
layout: post
title: "Proxmox Deployment - Hardware"
date: 2021-07-27 09:00:00 -0500
categories: [Training-SOC, Proxmox]
tags: [proxmox, cloud, deployment, soc, guide, RAID, local, network]
---

# Configuration
## Dell PowerEdge Servers RAID
BSU's SOC environment consists of Dell PowerEdge servers (no iDRAC license), so set-up steps are mostly identical across servers aside from varying hardware generations and Local or Network (NAS) 

_The pictures used in this guide are specific to the PERC H730P Raid Controller (Other servers will be very similar)_ :

* Connect a display and keyboard (and mouse if preferred) to the server.
* Power on the server and wait until the boot menu on the top of the screen is displayed - at this point press F10 (Life Cycle Manager) on the keyboard and wait for the screen to load.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/lifecycle%20controller.jpg?raw=true)
* Once in Lifecycle Manager we need to navigate to the RAID configuration
  1. Select System Setup on the sidebar and then choose "Advanced Hardware Configuration"
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4420.jpeg?raw=true)
  2. The next screen will have an option for "Device Settings" to select. 
  3. In "Device Settings" you will be presented with a list, ensure you select the raid controller
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4422.jpeg?raw=true)

### Local Storage Configuration
- Overview
  - For local storage we will be creating raid 1 virtual disk storage out of two drives for the Proxmox boot. A second virtual disk will also be created with the remaining drives _raid 5 if greater than one drive, else raid 0_. This virtual disk will be utilized for local virtual machine storage.
- Creating boot storage
  1. From the main menu of the raid controller, navigate to "Configuration Management", then select, "Create Virtual Disk" 
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4425.jpeg?raw=true)
  2. In this menu, select the raid level in accordance with desired usage <b> for boot storage, choose raid 1</b>. Next, choose "Select Physical Disks" and select two drives for the storage and apply changes.
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4427.jpeg?raw=true)
  3. Finally, set the Virtual Disk Name appropriately and click on "Create Virtual Disk"
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4428.jpeg?raw=true)
- Creating Virtual Machine Storage
 1. From the main menu of the raid controller, navigate to "Configuration Management", then select, "Create Virtual Disk" 
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4425.jpeg?raw=true)
 2. In this menu, select the raid level in accordance with desired usage <b> for virtual machine storage, choose raid 0 (or raid 5 if multiple drives remaining)</b>. Next, choose "Select Physical Disks" and select the remaining drives for the storage and apply changes.
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4427.jpeg?raw=true)
3. Finally, set the Virtual Disk Name appropriately and click on "Create Virtual Disk"
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4428.jpeg?raw=true)
### Network Storage Configuration (NAS)
- Overview
  - RAID configuration for network storage is very similar to local raid configuration. The primary difference being that the SSDs were used for the boot storage virtual disk for linux server [raid 1], while the rest of the drives on the machines were selected for raid 5 storage.

## Creating and Booting Proxmox
This guide will use balenaEtcher and a usb drive to create a bootable drive for proxmox. It will then cover booting from the drive using the dell boot manager.
### Creating a bootable Proxmox
1. Navigate to Proxmox's list of [ISO Images](https://proxmox.com/en/downloads/category/iso-images-pve) and download Proxmox VE 6.4 ISO Installer
2. Acquire balenaEtcher by browsing to [balenaEtcher's homepage](https://www.balena.io/etcher/) and downloading the installer that corresponds to your OS.
3. Follow the installation wizard and open the installed application.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/balenaEtcher.jpg?raw=true)
4. Choose the proxmox ISO that was downloaded in step 1, then select the usb inside your machine and finally choose flash.

### Booting a Dell Server with Proxmox
* Connect a display and keyboard (and mouse if preferred) to the server.
* Power on the server and wait until the boot menu on the top of the screen is displayed - at this point press F11 (Life Cycle Manager) on the keyboard and wait for the screen to load.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/IMG_4430.jpeg?raw=true)
* Choose the One-shot UEFI Boot Menu and then select the usb device as the boot option.
# [Next Section: Proxmox Install](https://bsu-cybersecurity.github.io/posts/proxmox-deployment-installation/)
