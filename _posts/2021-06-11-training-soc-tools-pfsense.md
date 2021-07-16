---
layout: post
title: "Tools - PfSense"
date: 2021-06-11 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, pfsense, soc, client]
---

## OpenVPN Initial Set-Up
https://www.ceos3c.com/pfsense/openvpn-on-pfsense/

- All changes made from the guide:
  - Skip step one and two when creating open vpn on pfsense
  - Step 4 changes:
    - Local port: 1194
    - Protocol: UDP on IPv4 only
    - Concurrent connections: 15
    - Didn’t set DNS server under client settings (maybe be something to go back to if desired)
  - Step 6 changes:
    - Since step one and two were skipped, change host name resolution:
      - Host Name Resolution: Interface IP Address
  - Step 6 Linux setup for Openvpn Client export:
    - Mac/Windows are in pfsense OpenVPN section specified in setup guide
    - Linux OpenVPN client setup:
      - https://openvpn.net/cloud-docs/openvpn-3-client-for-linux/
- OpenVPN Hardware Requirements
  - https://openvpn.net/vpn-server-resources/openvpn-access-server-system-requirements/

## Adding a User to Pfsense


## Configuring Firewall for Admin/User Tunnels


