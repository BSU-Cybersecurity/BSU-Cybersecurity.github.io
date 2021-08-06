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
# VXLAN
>These zones are used to detail the nodes that can cross communicate. For our purposes, we will allow all nodes to cross-communicate in our created zone.
>
## VXLAN Setup
1. On the proxmox GUI, navigate to Datacenter->SDN->Zones
2. Select "add vxlan" from the drop-down add tab at the top.
3. Enter a name for the zone in the ID, the list of nodes that should be cross communicating in the "Peer Address List" an MTU of 4500, and the names of the nodes in "Nodes".
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/proxmoxVXLanZone.png?raw=true)
4. Navigate to Vnets tab, under Zones and select the create button above. Enter a name starting with "vmbr" ex: "vmbr5", choose a descriptive alias, select the zone previously created, and give the VNet a unique tag. Make sure to leave VLAN Aware unchecked.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/VNet.png?raw=true)
5. Navigate to the "SDN" tab and click on "apply changes".<br>
_If the message indicates an error, it could just be a graphical bug, still test for functionality._
## Attatching Instances to VXlan
1. Create or edit an existing VM.
2. Attatch a network device under the hardware of the VM.
3. Select the name of the bridge that corresponds to the VNet that was just created.
![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/vnetbridge.png?raw=true)
4. If on Ubuntu Server, edit 
      ```console
        source /etc/network/interfaces.d/*
      ```
     ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/networkbridgesettings.png?raw=true)
      

# [Next Section: Proxmox Install](https://bsu-cybersecurity.github.io/posts/proxmox-deployment-installation/)
