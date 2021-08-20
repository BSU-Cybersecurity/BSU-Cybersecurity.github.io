---
layout: post
title: "Tools - PfSense"
date: 2021-06-11 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, pfsense, soc, client]
---

## OpenVPN Tunnel Guide
  >
    This section details how to create a tunnel between networks. For that purpose, a server, client and vpn user will be configured on the PfSense GUI. This setup utilizes at least two PfSense routers, one as client, with the other as a server
  >
  ### OpenVPN Server Setup
  - Client Export Package _(Easily Generates User Config Files)_
    - On PfSense GUI Navigate to System/Package Manager/Available Packages and search for openvpn-client-export then install.
  - Server Creation Wizard
    - Navigate to VPN/OpenVPN/Wizards
    - Select <b>Local User Access</b>
    - Create a new CA ![Desktop View](https://www.ceos3c.com/wp-content/uploads/2021/05/openvpn-on-pfsense-000211.jpg)
    - Create a new Server Certificate
    ![Desktop View](https://www.ceos3c.com/wp-content/uploads/2021/05/openvpn-on-pfsense-000212.jpg?ezimgfmt=ng:webp/ngcb48)
    - General Server Information
     ![Desktop View](https://www.ceos3c.com/wp-content/uploads/2021/05/openvpn-on-pfsense-000213.jpg?ezimgfmt=ng:webp/ngcb48)
     _Make certain the server's listening port is not in use_
     - Cryptography Settings
    ![Desktop View](https://www.ceos3c.com/wp-content/uploads/2021/05/openvpn-on-pfsense-000214.jpg?ezimgfmt=ng:webp/ngcb48)
    - Tunnel Settings
    ![Desktop View](https://www.ceos3c.com/wp-content/uploads/2021/05/openvpn-on-pfsense-000215.jpg?ezimgfmt=ng:webp/ngcb48)
    _The tunnel network address must not be in use, the local network will specify the LAN communication for VPN clients on this server_
    - Client and Advanced Settings

## Adding a User to Pfsense


## Configuring Firewall for Admin/User Tunnels


