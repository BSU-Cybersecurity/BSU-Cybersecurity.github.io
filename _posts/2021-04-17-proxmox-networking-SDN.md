---
layout: post
title: "Proxmox - SDN"
date: 2021-07-27 09:00:00 -0500
categories: [Training-SOC, Proxmox]
tags: [proxmox, cloud, deployment, soc, guide, LAN, SDN, proxmox-networking]
---
> In Proxmox cross node LAN networking can be accomplished through the use of software defined networking. In order to imitate seperate networks, while utilizing multiple nodes within these networks, software defined networking can be utilized. In this guide we will demonstrate cross-node vnet by creating a 10.0.2.0 network.
>

# Software Defined Network Installation
1. Install dependencies <b>on every node</b>
   ```console
     apt update
     apt install libpve-network-perl ifupdown2
   ```

2. Edit configuration file <b>on every node</b>
   
    add 
      ```console
        source /etc/network/interfaces.d/*
      ```
      to the end of <b>every node's</b> 
      ```console
        /etc/network/interfaces
      ```
 ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/proxmoxSDNInterface.jpg?raw=true)
# VXLAN Zone
>These zones are used to detail the nodes that can cross communicate. For our purposes, we will allow all nodes to cross-communicate in our created zone.
>
1. On the proxmox GUI, navigate to Datacenter->SDN->Zones
2. Select "add vxlan" from the drop-down add tab at the top.
3. Enter a name for the zone in the ID, the list of nodes that should be cross communicating in the "Peer Address List" an MTU of 4500, and the names of the nodes in "Nodes".
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/proxmoxVXLanZone.png?raw=true)
4. Navigate to Vnets tab, under Zones and select the create button above. Enter a name starting with "vmbr" ex: "vmbr5", choose a descriptive alias, select the zone previously created, and give the VNet a unique tag. Make sure to leave VLAN Aware unchecked.

![Desktop View](https://jaletzki.de/img/create-vm-w19-net.png)
![Desktop View](https://jaletzki.de/img/proxmox-enable-qemu-agent.png)
3. Download the latest stable [virtio driver](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md)
4. Add a CD/DVD drive to the virtual machine with the virtio ISO
 ![Desktop View](https://jaletzki.de/img/proxmox-add-drive-virtio.png)  
# Windows Configuration
1. Navigate through the Windows install guide until prompted for a driver to install. When prompted, browse to D:\viostar\2k16\amd64\viostar.inf
2. When the installation has finished, update drivers through the device manager. Also, update Windows by searching for Windows update in the Windows search bar.
3. Run `C:\Windows\System32\Sysprep\sysprep.exe`
 ![Desktop View](https://jaletzki.de/img/sysprep.png)
 4. Remove second CD Drive, reset first CD Drive to, "Do not use any media" and right click on VM and select, "Clone".
- If desired, you can rename the hostname of the windows machine
   - `Rename-Computer <hostname>`  

# [Next Section: Proxmox Install](https://bsu-cybersecurity.github.io/posts/proxmox-deployment-installation/)
