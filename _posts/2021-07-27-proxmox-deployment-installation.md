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

> _*note_: It is advantageous to set up your drive RAID configuration in such a way that the virtual disks are of different sizes. If, for instance, you had three drives which were all 600gb in size and you create one virtual disk in a RAID 1 configuration and the remaining virtual disk in a RAID 0 configuration you will see the disks listed as `/dev/sda` and `/dev/sdb`. Both will be the same size and they will be difficult to differentiate.

## Locality Settings
Set your country as the United States, your time zone as America/Boise, and your keyboard layout shoud default to U.S. English.

## Administration Settings
Set the root password for the node and add the management email.

## Network Settings
* __Management Interface__: This should default to the `eno1` NIC on Dell machines. If not, choose whichever NIC you configured for the device that is plugged into the switch and has internet.
* __Hostname__: The naming convention we have established for our cluster is `pve` (proxmox virtual environment) followed by the number denoting the order in which it was added. Check the current node names from the proxmox web interface in _Datacenter -> Cluster_ to choose a 'pve#' hostname one above the current highest node. The full hostname should be `pve#.trainingsoc.edu`.
* __IP Address__: The addressing convention is sequentially increasing from `192.168.0.50/16` upwards (except for two nodes which have 192.168.240.x addresses). Check the current node addresses from the proxmox web interface in _Datacenter -> Cluster_ to choose an IP one above the current highest node.
* __Gateway__: Our gateway is `192.168.0.1` 
* __DNS Server__: Our DNS Server is `192.168.0.1`

## Review and Install
After reviewing your configuration you should be good to install. This can take a while (~10 min. depending on hardware). After the install is complete the system should restart. Rember to remove the flash drive 

After the reboot, if you are greeted by a login terminal displaying your configured IP address, you have successfully installed Proxmox VE!

# Configuration
## Connect
The first step to configuring your new Proxmox node is to connect to its web interface. To do so enter `https://NODE_IP:8006` into your web browser where `NODE_IP` is the IP address that you configured in the previous section (make sure that you are on the same network as the node).

If done correcctly you will be prompted that the website may be unsafe due to a lack of an SSL cert. This is fine, just proceed anyways.

You should now be able to log in with the username `root` and the password that you configured in the previous section.

## Cluster
You will now want to set up a datacenter cluster so that you can take advantage of all the datacenter features that proxmox has to offer. This includes the ability to monitor the resources of many nodes at once, migrate VM's to different nodes, and add networked storage that all nodes in the datacenter can access.

### Create New Cluster
If this is a fresh environment and you do not already have a cluster to join you will have to create one. Navigate to _Datacenter -> Cluster_ in the web interface and click `Create Cluster`. Enter a name for the cluster and click create.

### Join
To join a cluster you will need to navigate to _Datacenter -> Cluster_ in the web interface of a node already in the cluster. Click `Join Information` and then `Copy Information`. Navigate to _Datacenter -> Cluster_ in the web interface of the node that will be joining the cluster and click `Join Cluster'. Paste the join information and then enter the root password for the machine that started the cluster.

## Storage
The next step is to configure storage so that you have space for virtual machines to store their virtual hard drives and space to store `.iso` images, containers and other resources.

There are many ways in which you can configure storage. Each have different use cases and should be cosidered carefully to suit your needs.

### Local

> _*note: Local storage is only necessary if you do not have networked storage._

There are two main kinds of local storage that we use: __LVM volumes__ and __directories__. LVM volumes will allow us to run images in a raw format while directories will run them in a qcow2 format. It is important to take into consideration that only directories can store `.iso` images. If you are only using local storage and not networked storage then you will want at least one local disk to be a directory so that you can store `.iso`'s.

To set up a local directory navigate to _pve# -> Disks -> Directory_ in the web interface and click `Create: Directory`. Select the disk you wish to use, select `ext4` for the filesystem, and name the directory.

To set up a local LVM volume navigate to _pve# -> Disks -> LVM_ in the web interface and click `Create: Volume Group`. Select the disk you wish to use and name the volume.

> "No Disks unused" error: If you have the correct number of drives installed to implement your desired local storage configuration and you get this error you may have to remove partitions from the disks you want to use. To do this navigate to _pve# -> Shell_ in the web interface. Enter `fdisk -l` to list disks and partitions on the machine. Find your target disk of the format `/dev/sdX` (X being some letter) and run `fdisk /dev/sdX`. __BE CAREFUL, FDISK IS POWERFUL__. Enter `d` to delete partitions on the selected disk. Enter the number of the partition you wish to delete based on their numbering format from the fdisk list (`/dev/sdX1` is partition 1). Once all partitions are deleted enter `w` to write the changes.

### Networked
Networked storage is very convenient and is ideally where you want to have the bulk of your storge capacity. It has the same functionality as local directory storage except it can be accessed by every node in the cluster.

If you have a NAS with an NFS server (Like we set up [here](https://bsu-cybersecurity.github.io/posts/nas-setup/)) you can add it like this:

First, navigate to _Datacenter -> Storage_ in the web interface. Then, click _add -> NFS_.
* ID: a name for the storage
* Server: the server IP address
* Export: the NFS filesystem location on the server
* Content: select all the types of content you plan to store

Then click add and you should have networked storage available to all your nodes!

Now that you have a datacenter set up with some clustered nodes, learn more about what you can do when them in our [proxmox-operations](https://bsu-cybersecurity.github.io/tags/proxmox-operations/) posts.