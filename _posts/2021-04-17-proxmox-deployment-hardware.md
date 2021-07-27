---
layout: post
title: "Proxmox - Hardware"
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

Scrolling down, there are settings for registering iDRAC on DNS, which we disabled. We enabled DHCP for IPv4 as DHCP makes enlisting nodes on MAAS much easier, and we also enabled using DHCP to obtain DNS addresses.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/iDrac%20on%20DNS%20disable.jpg?raw=true)


Lastly, and the whole point of setting up the iDRAC in the first place - we need to enable IPMI so that MAAS can power-cycle the machines. Below are the default settings - you can choose to assign a different privilege or encryption key but keep in mind that if you change the default, to make IPMI work you will need to specify both of these parameters in MAAS when setting up a new node.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/ipmi%20lan.jpg?raw=true)


Now press the ESC key to return to the Life Cycle Manager main page and you will be asked to reboot. At this point you are done setting up iDRAC + IPMI settings.


### 3x RAID Virtual Disks
The first virtual drive is set up using the Wizard on the “Home” tab of Life Cycle Manager. This first step is ideal if the drives have foreign raid configurations already (if the drives came from another server). The wizard will walk you through the RAID configuration. Note: This is just one of three virtual disks we will set up so allocate physical drives accordingly.


The subsequent two virtual disks will be set up using the device manager itself. To access this, go to the “System Setup” tab of Life Cycle Manager and select “Advanced Hardware Configuration”. From there select Device Settings and then select the RAID controller from the list of devices.
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/Device%20Settings.jpg?raw=true)


Now press the ESC key to return to the Life Cycle Manager main page and you will be asked to reboot. At this point you are done setting up your three RAID virtual drives.

## PXE Boot
When MAAS wakes a server using IPMI, the server needs to perform a PXE Network boot by default. To enable this, go to the “System Setup” tab of Life Cycle Manager and select “Advanced Hardware Configuration”.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


From here, select “System BIOS”, and once in System BIOS ensure the Boot Mode is set to UEFI. During our testing phase we also disabled Boot Sequence retry for debugging purposes. Next, click on “UEFI Boot Settings”.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Once in UEFI boot settings select “UEFI Boot Sequence '', and use the +/- keys to move the NIC that you will be PXE booting from to the top of the list. In our case, we PXE boot from the same NIC that we perform IPMI over.
  ![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Now press the ESC key to return to the Life Cycle Manager main page and you will be asked to reboot. At this point you are done setting up PXE boot settings.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Now press the ESC key to return to the Life Cycle Manager main page and you will be asked to reboot. At this point you are done setting up PXE boot settings.


# [Next Section: Network](https://bsu-cybersecurity.github.io/posts/openstack-deployment-network/)
