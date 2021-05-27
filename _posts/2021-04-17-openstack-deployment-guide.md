---
layout: post
title: "Open Stack Deploymet Guide"
date: 2021-04-24 09:00:00 -0500
categories: OpenStack Cloud
tags: openstack cloud deployment soc guide charms juju maas
---


## Overview
The purpose of this guide is to aid in the replication and maintenance of our OpenStack cloud by documenting the deployment process.

### Versions
The software versions we use in our environment are as follows:
* Ubuntu 20.04 LTS (Focal) for the MAAS server, Juju client, Juju controller, and all cloud nodes (including containers)
* MAAS 2.9.2
* Juju 2.7.6
* OpenStack Wallaby

### Hardware Layout (5 Nodes)

| Name          | Description   |
| ------------- |---------------|
| Client Machine | Machine used to SSH into the MAAS Controller, or MAAS nodes. |
| MAAS Controller | This machine is not part of the OpenStack cloud - it simply manages the “metal” of the cloud by power-cycling machines and network booting and deploying OS images. |
| JuJu Controller | One of the nodes managed by MAAS. Not to be confused with the JuJu application that lives within the MAAS Controller. The JuJu Controller orchestrates the commands given to the JuJu application and hosts the JuJu dashboard. |
| node0-3 | These MAAS managed nodes (tagged “compute”) are where the OpenStack Charm containers get deployed to. These nodes form the actual OpenStack cloud. |


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

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

## Traefik

```bash
mkdir traefik
cd traefik
mkdir data
cd data
touch acme.json
chmod 600 acme.json
touch traefik.yml
```

`traefik.config` can be found [here](https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik)

create docker network

```bash
docker network create proxy
```

```bash
touch docker-compose.yml
```

`docker-compose.yml` can be found [here](https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik)

```bash
cd data
touch config.yml
```

```bash
docker-compose up -d
```

## Portainer

```bash
mkdir portainer
cd portainer
touch docker-compose.yml
mkdir data
```

`docker-compose.yml` can be found [here](https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/portainer)


### Generate Basic Auth Password


```bash
sudo apt update
sudo apt install apache2-utils
```

```bash
echo $(htpasswd -nb USER PASSWORD) | sed -e s/\\$/\\$\\$/g
```

use this in your `docker-compose.yml` (`USER:BASIC_AUTH_PASSWORD`)

```bash
docker-compose up -d
```

## Traefik Routes Config

```bash
cd traefik/data
nano config.yml
```

`config.yml` [here](https://github.com/techno-tim/techno-tim.github.io/tree/master/reference_files/traefik-portainer-ssl/traefik)

```bash
docker-compose up -d --force-recreate
```
