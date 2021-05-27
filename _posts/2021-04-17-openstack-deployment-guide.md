---
layout: post
title: "Open Stack Deploymet Guide"
date: 2021-04-24 09:00:00 -0500
categories: [OpenStack, Cloud]
tags: [openstack, cloud, deployment, soc, guide, charms, juju, maas]
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


### Deployment Process

The first step in deployment is to configure the hardware (servers) so they can play nicely with the MAAS Application later in the deployment process. It is important to ensure all minimal hardware requirements are met.

Next, there needs to be some thought put into network architecture, and the Network Set-Up section lists some of the considerations.

Ubuntu Server 20.04 LTS needs to be With hardware and network configuration done, installed on a server separate from the OpenStack cluster - this server is referred to as the “MAAS Controller”. The MAAS Application is then installed on the MAAS Controller using the Snap Store, and is then configured appropriately.

The servers that are part of the OpenStack cluster (which are referred to as JuJu Controller, Node0, Node1, Node2, and Node3) need to then be enlisted on MAAS, and set to a “ready” state to be “deployed” later on by JuJu. This process is covered in the MAAS Enlisting section.

Before any JuJu magic can happen, the JuJu Application needs to be installed on the MAAS Controller using the Snap Store - all JuJu commands will be run on the MAAS Controller command line, but will be orchestrated by the JuJu Controller.

To actually deploy the JuJu Controller hardware (and nodes later on), the JuJu Application needs to have API access to MAAS. This quick set-up process is outlined in the JuJu Configuration section.  And once JuJu can control MAAS-managed hardware through API, the actual JuJu controller can be bootstrapped and deployed - this is done through command line on the MAAS Controller as described in the JuJu Deployment section.

With the JuJu Controller deployed, the OpenStack installation can begin. This (lengthy and tedious) process is described step-by-step under “Open Stack Installation”. The OpenStack cloud then needs to be configured for non-admin access and use, and managed for maintenance, scalability and power events - all of which is outlined in detail in the OpenStack Management section.

## Hardware
### Minimum Requirements
| Name | CPUs (cores) | RAM (GB) | Storage | NIC's |
| ------ | ------ | ------ | ------ | ------ |
| MAAS Controller | 2| 8| 1x 40GB| 1 |
| JuJu  Controller | 2| 4| 1x 40GB| 1 |
| Node0 | 2| 8| 3x 80GB| 2 |
| Node1 | 2| 8| 3x 80GB| 2 |
| Node2 | 2| 8| 3x 80GB| 2 |
| Node3 | 2| 8| 3x 80GB| 2 |

### Configuration
#### Dell PowerEdge Servers
Our environment consists of Dell PowerEdge servers (no iDRAC license), so set-up steps are mostly identical across servers aside from varying hardware generations - and this process goes as follows for each server that will be managed by MAAS:

* Connect a display and keyboard (and mouse if preferred) to the server.
* Power on the server and wait until the boot menu on the top of the screen is displayed - at this point press F10 (Life Cycle Manager) on the keyboard and wait for the screen to load.
* Once in Life Cycle Manager we need to set up the following:

## Network
### Architecture

#### Serving DHCP
We have the option to let our router or MAAS serve DHCP, and both options have their implications.

#### Letting MAAS Serve DHCP
When MAAS is serving DHCP, MAAS will automatically discover a server that is PXE booting on the same network. When a server is booting over PXE, MAAS takes over by providing a boot image and automatically listing the node under “Machines” on the MAAS Application GUI - and if IPMI was set up correctly, MAAS will automatically configure the IPMI. Though this is very convenient, the downside however, is that the Firewall/Router won’t manage DHCP leases.

#### Letting the Firewall/Router Serve DHCP
Alternatively, we can let the router serve DHCP. The main consideration here (as far as MAAS is concerned) is that we will have to manually enlist new servers by MAC and IP address, which isn’t a super tedious process but it is something to consider.


## MAAS
### Controller
One server that isn’t part of the OpenStack cluster (but is on the same network) needs to be the designated “MAAS Controller”. The MAAS Controller will “manage” the hardware (servers), meaning it can power on/off + deploy operating system images to each of the servers networked in the OpenStack cluster.
#### Installing the Operating System
Though most Debian Linux distributions would work, our MAAS Controller runs Ubuntu Server Focal 20.04 LTS.
#### A Few Pointers
* If DHCP is being served by MAAS and there are no other DHCP servers, then Ubuntu won’t be able to automatically configure the network during the installation process - it will need to be configured manually (see below). Alternatively, you may decide to enable serving DHCP on the router during this process, and then disable it once the MAAS application is installed and configured.

[INSER IMAGE]


* It is infinitely easier to manage the MAAS Controller over SSH, however, Ubuntu Server comes out of the box with UFW, so SSH needs to be allowed through the firewall. To allow ssh, input the following command.
```bash
$ sudo ufw allow ssh
```

### Installation
With Ubuntu configured on the MAAS Controller, the MAAS application needs to be installed using the Snap Store by inputting this command:

```bash
$ sudo snap install --channel=2.9/stable maas
```
MAAS needs a database backend, which can be installed using the following commands.

```bash
$ sudo apt update -y
$ sudo apt install -y postgresql
```

A user for the database then needs to be created. Input the following command - the $VARIABLE’s need to be replaced appropriately and documented for future reference.
```bash
$ sudo -u postgres psql -c "CREATE USER \"$MAAS_DBUSER\" WITH ENCRYPTED PASSWORD '$MAAS_DBPASS'"
```
Then the database needs to be created with the user created above. Run the following command replacing the $VARIABLE’s.

```bash
$ sudo -u postgres createdb -O "$MAAS_DBUSER" "$MAAS_DBNAME"
```
A postgres config file now needs to be edited to reflect the new database user and database name. At the time of writing this guide, the latest stable version of postgres is version 12, so the following command will use vim to open the file in this directory  - `````/etc/postgresql/12/main/pg_hba.conf````` using root privilege. In the event this doesn’t work, manually navigate to this directory, you likely have a different version of postgres installed so the ‘12’ in the directory may be a different number.


```bash
$ sudo vim /etc/postgresql/12/main/pg_hba.conf
```

With the ```pg_hba.conf``` file open, add the following line at the end, and save the file.


```bash

```


```bash

```


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
