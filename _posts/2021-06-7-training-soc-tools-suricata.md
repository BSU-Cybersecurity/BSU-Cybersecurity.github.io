---
layout: post
title: "Tools - Suricata"
date: 2021-06-07 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, suricata, soc, client]
---
# Install/Setup
- https://suricata.readthedocs.io/en/suricata-6.0.0/quickstart.html
  - This quickstart guide uses an ubuntu repo. Can also be installed from source. Documentation also contains a curl command for easy testing.
# Host Bridged Network
- https://medium.com/nycdev/how-to-ssh-from-a-host-to-a-guest-vm-on-your-local-machine-6cb4c91acc2e
# Client-Router-Server
- https://sandilands.info/sgordon/building-internal-network-virtualbox
- Netplan is the manager for recent versions of LTS Ubuntu
- VirtualBox can use NatNetwork with promiscuous mode to share packet traffic between virtual machines - Much simpler to set-up method, but also more VM dependent.

# Testing Environment

For installation testing purposes, a simple setup of 2 virtualbox machines running linux server works well. One machine will act as a client, while the other machine can have Suricata installed. Once setup is complete, test the environment by using the command `curl http://testmyids.com/`. A sucessful setup should trigger an alert that will be visible using `sudo tail -f /var/log/suricata/fast.log`.
>Be sure to create a nat network and enable promiscious mode for the virtual machines. This will enable traffic to be shared on the nic that is monitored by Suricata.


# Suricata Rules
#### Format

Rules for Suricata are defined using a basic signature of action/header/options

- <span style="color:red">Action</span>
  - Determines what occurs when the signature matches
  - alert (generates an alert)
  - pass (stops inspection)
  - reject (send unreachable error to sender)
- <span style="color:green">Header</span>
  - Protocol (tcp, udp, icmp, http, ssh)
  - Source and destination of traffic
  - ($HOME\_NET, $EXTENRAL_NET) IP or Ports
  - Direction ->, <>
- <span style="color:blue">Options</span>
  - Enclosed by parenthesis, seperated by semicolons

> <span style="color:red"> drop<span style="color:green"> tcp $HOME\_NET any -> $EXTERNAL_NET any <span style="color:blue">(msg:”ET TROJAN Likely Bot Nick in IRC (USA +..)”; flow:established,to_server; flowbits:isset,is_proto_irc; content:”NICK “; pcre:”/NICK .*USA.*[0-9]{3,}/i”; reference:url,doc.emergingthreats.net/2008124; classtype:trojan-activity; sid:2008124; rev:2;)</span>

> More Information: https://suricata.readthedocs.io/en/suricata-6.0.0/rules/intro.html

##### Custom
1. Create a file to store rules
`sudo nano local.rules`
2. Update the Suricata configuration file to include rule file
   - `sudo nano /etc/suricata/suricata.yaml`
      - `~  default-rule-path: /usr/local/etc/suricata/rules` 
      rule-files:
     \- suricata.rules
     \- /path/to/local.rules
> More information:
https://suricata.readthedocs.io/en/suricata-6.0.0/rule-management/adding-your-own-rules.html
##### Management Tool
This is the method reccomended for adding rules for Suricata.
- Check available rulesets
1. fetch index with `sudo suricata-update update-sources`
2. List available sources with `sudo suricata-update list-sources`
3. enable example `sudo suricata-update enable-source oisf/trafficid` 
`sudo suricata-update`