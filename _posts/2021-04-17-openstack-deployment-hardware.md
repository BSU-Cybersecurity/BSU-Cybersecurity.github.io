---
layout: post
title: "OpenStack Deployment - Hardware"
date: 2021-02-29 09:00:00 -0500
categories: [Training-SOC, OpenStack]
tags: [openstack, cloud, deployment, soc, guide, charms, juju, maas]
---
# Minimum Requirements

| Name | CPUs (cores) | RAM (GB) | Storage | NIC's |
| ------ | ------ | ------ | ------ | ------ |
| MAAS Controller | 2| 8| 1x 40GB| 1 |
| JuJu  Controller | 2| 4| 1x 40GB| 1 |
| Node0 | 2| 8| 3x 80GB| 2 |
| Node1 | 2| 8| 3x 80GB| 2 |
| Node2 | 2| 8| 3x 80GB| 2 |
| Node3 | 2| 8| 3x 80GB| 2 |

# Configuration
## Dell PowerEdge Servers
BSU's SOC environment consists of Dell PowerEdge servers (no iDRAC license), so set-up steps are mostly identical across servers aside from varying hardware generations - and this process goes as follows for each server that will be managed by MAAS:

* Connect a display and keyboard (and mouse if preferred) to the server.
* Power on the server and wait until the boot menu on the top of the screen is displayed - at this point press F10 (Life Cycle Manager) on the keyboard and wait for the screen to load.
* Once in Life Cycle Manager we need to set up the following:

### iDRAC +IPMI
As mentioned above, we do not have an iDRAC license, so this step is especially unique for us.
From the “Home” tab of Life Cycle Manager select “Configure server for remote access (iDRAC)”. The Home tab will look something like this depending on server generation.

![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Here, select “Network”.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Since we do not have an iDRAC license we will use NIC1 for iDRAC instead of the dedicated iDRAC NIC. In our case we did not specify a “Fallover Network”.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Scrolling down, there are settings for registering iDRAC on DNS, which we disabled. We enabled DHCP for IPv4 as DHCP makes enlisting nodes on MAAS much easier, and we also enabled using DHCP to obtain DNS addresses.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Lastly, and the whole point of setting up the iDRAC in the first place - we need to enable IPMI so that MAAS can power-cycle the machines. Below are the default settings - you can choose to assign a different privilege or encryption key but keep in mind that if you change the default, to make IPMI work you will need to specify both of these parameters in MAAS when setting up a new node.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


Now press the ESC key to return to the Life Cycle Manager main page and you will be asked to reboot. At this point you are done setting up iDRAC + IPMI settings.
![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }

### 3x RAID Virtual Disks
The first virtual drive is set up using the Wizard on the “Home” tab of Life Cycle Manager. This first step is ideal if the drives have foreign raid configurations already (if the drives came from another server). The wizard will walk you through the RAID configuration. Note: This is just one of three virtual disks we will set up so allocate physical drives accordingly.


The subsequent two virtual disks will be set up using the device manager itself. To access this, go to the “System Setup” tab of Life Cycle Manager and select “Advanced Hardware Configuration”. From there select Device Settings and then select the RAID controller from the list of devices.
  ![Shadow Avatar](https://cdn.jsdelivr.net/gh/cotes2020/chirpy-images/posts/20190808/window.png){: .shadow width="90%" }


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


## [Next Section: Network](https://bsu-cybersecurity.github.io/posts/openstack-deployment-network/)
