---
layout: post
title: "Proxmox - Template Creation"
date: 2021-07-27 09:00:00 -0500
categories: [Training-SOC, Proxmox]
tags: [proxmox, cloud, deployment, soc, guide, templates, windows]
---
> In Proxmox templates are an easy way to instantiate mutliple virtual machines without having to reinstall the OS from scratch. Templates also allow for unique MAC addresses and SIDs.
>

# Windows VM Initialization
1. Download an [ISO image](https://www.microsoft.com/en-au/software-download/windows10) from microsoft
2. Create a Virtual Machine
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
