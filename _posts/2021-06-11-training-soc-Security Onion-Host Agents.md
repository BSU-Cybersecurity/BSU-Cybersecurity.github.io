---
layout: post
title: "Security Onion - Host Agents"
date: 2021-08-11 09:00:00 -0500
categories: [Training-SOC, SecurityOnion]
tags: [SecurityOnion, Windows, Linux, Agent]
---


 
  > Wazuh is an EDR (endpoint detection and response) system used to monitor and respond to threats on a host machine. Wazuh has two core components - a server and an agent. In a Security Onion distributed deployment, the server for Wazuh exists on the sensor node, while the agent exists on the host. This guide will navigate establishing the Wazuh agent's communication from a Windows host to the sensor node.

## Windows Hosts

### Wazuh Installation and Configuration

- On the manager node, use 
  ```console
  sudo so-allow
  ```
  to enable contact through Security Onion's firewall to the Wazuh endpoint
- From the Security Onion Console, navigate to the 'Downloads' section from the sidebar.
- Select the ossec-agent, download and transfer the downloaded installer to your Window's host machine.
- Access the Wazuh manager on the Sensor Node through the docker containing using
  - ```console
    $ sudo so-wazuh-agent-manage
    ```
  - Type `a` in the selection interface to add the windows host via I.P address to the manager.
  - Type `e` and select the newly added host machine to extract a key for that host. Copy the displayed key for transfer to the host machine.
  - Open the Wazuh agent manager on the Windows host machine and enter the I.P adress of the sensor as well as the previously extracted key.
  - Restart the agent and refresh the window
    
### OSquery Installation and Configuration

> OSQuery allows for low-level querying of an operating system by representing the system as a relational database.

_Security Onion leverages fleet to manage OSQuery. Fleet can be found in the Security Onion Console's side-panel and will list host devices OSQuery has been installed on_
- On the manager node, run
  ```console
  $ sudo so-allow
  ```
  to enable the host agent to connect on port 8090
- Download the OSQuery agent from the Security Onion Console's 'Downloads' tab and transfer the executable to the Windows host machine and run the installer
- Check the installation by navigating to Fleet. The hostname of the added Windows machine should appear in the listed fleet hosts.

_To change the hostname on a Windows computer_
 1. Open <b> Settings > System > About </b>
 2. Select 'Rename PC'
 3. Choose the hostname
 4. Restart Windows

### Winlogbeat Installation and Configuration

> Winlogbeat is a tool utilized for shipping Windows event logs to Elasticsearch

- Download Winlogbeat zip file from Security Onion Console's 'Downloads' section and transfer the file to the Windows host.
- Extract the Winlogbeat zip file.
- Edit `winlogbeat.yml` to use Logstash.
  - Comment out the lines related to direct output for Elasticsearch
  - Uncomment the lines for Logstash output and specify the manager node's I.P address
- Open PowerShell with <b>Administrator access</b>
- Run 
  ```console
  winlogbeat.exe -c winlogbeat.yml
  ``` 

## Linux Host Agents







