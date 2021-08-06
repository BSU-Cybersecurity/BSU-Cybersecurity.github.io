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
# VXLAN Zone


 ![Desktop View](https://jaletzki.de/img/create-vm-w19-os.png)
![Desktop View](https://jaletzki.de/img/create-vm-w19-hdd.png)
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
