---
layout: post
title: "Training Scenario One"
date: 2021-05-28 09:00:00 -0500
categories: [Training-SOC, Scenarios]
tags: [soc, training, scenario]
---


## Overview
THis is a guide about a scenario that we haven't written.
### Versions
The software versions we use in our environment are as follows:
* Ubuntu 20.04 LTS (Focal) for the MAAS server, Juju client, Juju controller, and all cloud nodes (including containers)
* MAAS 2.9.2
* Juju 2.7.6
* OpenStack Wallaby

### Hardware Layout (5 Nodes)

```bash
 sudo apt-get update
 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```bash
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```bash
 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

```bash
 sudo usermod -aG docker $USER
```
You'll need to log out then back in to apply this

### Install Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
