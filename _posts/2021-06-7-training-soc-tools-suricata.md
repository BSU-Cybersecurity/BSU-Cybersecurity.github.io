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
# Suricata Rules
- Format
  - https://suricata.readthedocs.io/en/suricata-6.0.0/rules/intro.html
    - Defines the basic signature of action/header/options
    - Action: alert, pass (stop inspection), drop(drop packet and alert), reject (send unreachable error to packet sender), etc…
    - Header
      - Protocol (tcp, udp, icmp, http, ssh)
      - Source and destination of traffic ($HOME_NET, $EXTERNAL_NET) IP or Ports
      - Direction ->, <>
    - Options
      - Enclosed by parenthesis, separated by semicolons
      - (msg:"Message with semicolon)”;
- Custom
  - https://suricata.readthedocs.io/en/suricata-6.0.0/rule-management/adding-your-own-rules.html
    - Documentation for creating a ruleset file and adding the file path to the suricata .yaml ruleset
    - Restart suricata for changes to take effect
- Management tool (add pre-built rules)
  - Recommended method for adding rules
  - https://suricata.readthedocs.io/en/suricata-6.0.0/rule-management/suricata-update.html
    - (fetch updated rulesets) $sudo suricata-update update-sources
    - (list updated rulesets) $sudo suricata-update list-sources
    - (show rulesets in effect) $sudo suricata-update list-enabled-sources

