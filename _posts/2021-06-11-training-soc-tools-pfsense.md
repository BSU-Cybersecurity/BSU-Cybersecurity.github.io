---
layout: post
title: "Tools - PfSense"
date: 2021-06-11 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, pfsense, soc, client]
---

## OpenVPN Tunnel Guide
 
  > This section details how to create a tunnel between networks. For that purpose, a server, client and vpn user will be configured on the PfSense GUI. This setup utilizes at least two PfSense routers, one as client, with the other as a server
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
    ![Desktop View]( https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/Pfsense%20Client%20Settings.png?raw=true)
    ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/PfSense%20Advanced%20Config.png?raw=true)
    
### Adding a User to PfSense
>In order to access the PfSense tunnel, creating a user with authentication information is necissary.
- Create a VPN User
  - Navigate to System/User Manager and select <b>+Add</b> to begin the process of creating a new user
  - Fill out a Username and Password and check <b>Click to create a user certificate</b>. 
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/OpenVPN%20User.png?raw=true)
 ### Exporting a Client
 > The previously installed <b>client export package</b> will be used to generate a file containing the necissary information for a client to connect to the server.
 - Navigate to VPN/OpenVPN/Client Export
 - Select Server and Client Connection Settings
 ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/Client%20Connection.png?raw=true)
 ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/clientdownload.png?raw=true)
 _Choose the Inline Configurations/Most Clients option if connecting to a router or choose OS specific download for single user connections_
 ### PfSense Client Import
 > While PfSense has a package for exporting, the free version does not have an easy tool for importing client configurations.
 - Creating User CA
   - Navigate to /System/Certificate Manager/ CAs on the client PfSense router.
   ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/CACreation.png?raw=true)
   _The Certificate Authority Data is in the file generated from the client-export package used on the server. Copy from `<ca>` to `</ca>`_.
- Creating User Cert
  - Navigate to /System/Certificate Manager/Certificates
  ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/CERTCREATION.png?raw=true)
  _The Certificate Data and Private key is found in the file generated from the client-export package. Copy the certificate data from `<cert> to </cert>` and the private key from `<key> to </key>`._
- Creating Client
  - Navigate to VPN/OpenVPN/Clients and click on <b>+ Add</b>
    ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/ClientGeneralInformation%20-%20Copy.png?raw=true)
    _Change server address and server port - these are located in the client-export file if you don't remember._
     ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/authcrypt1.png?raw=true)
      _The username and password of the PfSense user created on the server_
      ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/authcrypt2.png?raw=true)

       ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/tunnelsettings.png?raw=true)
       _Let the server handle the tunnel settings_
       ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/advancedconfig.png?raw=true)
      
### Configuring Firewall for Admin/User Tunnels
- Add tunnel interface
  - Navigate to Interfaces/Interface Assignments
    - Add the interface with the network port
  - Navigate to Firewall/NAT/Outbound
    - Select manual outbound for the mode
    - Add a new mapping
    ![Desktop View](https://github.com/BSU-Cybersecurity/BSU-Cybersecurity.github.io/blob/main/images/NAT.png?raw=true)
   










