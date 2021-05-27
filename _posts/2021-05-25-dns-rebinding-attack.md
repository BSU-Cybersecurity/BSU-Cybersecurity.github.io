---
layout: post
title: "DNS Rebinding Attack"
date: 2021-05-25 09:00:00 -0500
categories: [Network-Security-&-Defense, Attacks]
tags: [network, security, dns, rebinding, attack]
---
## Overview
In this demo, you will demonstrate the DNS rebinding attack.

The goal of the attacker is, whenever the victim visits www.attacker32.com, the victim's smart thermostat's temperature will be changed to 88 Celsius degree.

## Setup
For this attack you will need 3 Linux VMs.
* VM1 as Victim Client - acting as 2 roles, a web client, and an IoT server (www.seediot32.com), a Local DNS Server
* VM2 as Local DNS Server
* VM3 as Attacker - acting as 2 roles, a malicious web server (www.attacker32.com), and malicious DNS serve (which is responsible for the attacker32.com domain).

The 3 VMs reside in the same network.

## Attack
### Setting up the client
1. Reduce Firefox’s DNS caching time: Type about:config in the URL field, and then change network.dnsCacheExpiration from 60 to 10.
2. Change /etc/hosts: add the following entry in the file:
```shell
CLIENT_VM_IP	www.seediot32.com (note: change CLIENT_VM_IP to the Client VM's IP address)
```
3. Configure DNS server information, i.e., let the client know the IP address of the DNS server.
 ..* Add this line to the end of file ```/etc/resolvconf/resolv.conf.d/head```

``` nameserver DNS_SERVER_IP (change DNS_SERVER_IP to the local DNS server's IP address)```
5.
