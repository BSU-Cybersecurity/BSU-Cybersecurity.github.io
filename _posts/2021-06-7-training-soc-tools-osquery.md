---
layout: post
title: "Tools - OSQuery"
date: 2021-06-08 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, osquery, soc, client]
---

https://osquery.readthedocs.io/en/stable/
# Overview & Use Cases
https://www.rapid7.com/blog/post/2016/05/09/introduction-to-osquery-for-threat-detection-dfir/

# Install
- https://kifarunix.com/install-osquery-on-ubuntu/
- Can follow guide word for word (works out of the box), commands using osqueryctl require use of sudo
 - ```sudo osqueryctl start osqueryd```
 - Can use osquery as a service with systemctl (in the state of OS), instructions in document above

# Setup
- Works out of the box as an endpoint agent, but needs a solution for use on a bigger scale; The section titled, Building your own osquery solution, explains:
  - https://www.uptycs.com/blog/osquery-what-it-is-how-it-works-and-how-to-use-it
- Tools recommended for osquery solution:
  - https://www.uptycs.com/blog/deploying-osquery-at-scale-a-comprehensive-list-of-open-source-tools
- Basic solution: Endpoint Config, Inspection, Management TBD; Data transport & Data storage (ELK)

# Tables:
List of tables in osquery: https://osquery.io/schema/4.8.0/

# Possible Kolide/Fleet install:

- https://kifarunix.com/install-fleet-osquery-manager-on-ubuntu/
- https://www.youtube.com/watch?v=XC7zfQ-SmqQ

# Issues setting up fleet management
- ERROR 1045 (28000): Access denied for user ‘debian-sys-maint’@’localhost’ (using password: YES)
  - https://electrictoolbox.com/mysql-access-denied-debian-sys-maint/
  -My solution: reinstalled the entire VM and walked through the entire tutorial (great right?)
