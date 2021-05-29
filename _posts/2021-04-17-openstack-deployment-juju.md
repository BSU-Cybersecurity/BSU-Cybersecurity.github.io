---
layout: post
title: "OpenStack Deployment - JuJu"
date: 2021-02-05 09:00:00 -0500
categories: [Training-SOC, OpenStack]
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

[INSERT IMAGE]


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
host    $MAAS_DBNAME    $MAAS_DBUSER    0/0     md5
```

Now it is time to initialize the MAAS application using the command below. Replace the variables as appropriate, and since we are running the MAAS application on our MAAS  Controller, the `````$HOSTNAME````` should be set to ```localhost```.


```bash
$ sudo maas init region+rack --database-uri "postgres://$MAAS_DBUSER:$MAAS_DBPASS@$HOSTNAME/$MAAS_DBNAME"
```

Note that the IP of the web GUI is printed out on the terminal during this step. Press enter unless you want to make any changes. If you specified that the ```$HOSTNAME``` was ```localhost```, the IP of the web GUI is the same as that of the MAAS Controller machine.

Finally, a MAAS admin user needs to be created.

```bash
$ sudo maas createadmin
```

You will be prompted to input user credentials - these will be used to log into the web GUI so make sure to document these for future use.


Now check the status of the MAAS application using the following command


```bash
$ sudo maas status
```

The output should be similar to the following.

```bash
$ sudo maas status
bind9                        RUNNING   pid 22236, uptime 0:01:02
dhcpd                        STOPPED   Not started
dhcpd6                       STOPPED   Not started
http                         RUNNING   pid 22461, uptime 0:00:42
ntp                          RUNNING   pid 22372, uptime 0:00:45
proxy                        RUNNING   pid 22604, uptime 0:00:34
rackd                        RUNNING   pid 22241, uptime 0:01:02
regiond                      RUNNING   pid 22248, uptime 0:01:02
syslog                       RUNNING   pid 22366, uptime 0:00:45

```
If any  services are not running, follow this process to re-initialize MAAS.

At this point the MAAS web GUI should be accessible from a web browser on a machine sharing the same network as the MAAS Controller.

### Initial Set-Up

Access the MAAS web GUI from your local browser. You should be able to access it as long as you are on the same network as the MAAS Controller and MAAS is running.
[INSERT IMAGE]

Log in to the MAAS web GUI and follow the set-up process. The steps are mostly straight-forward.

[INSERT IMAGE]

#### Import SSH Keys
In the next step of the initial set-up, you will be asked to input an ssh-key so that you can ssh into the machines MAAS deploys using RSA key authentication instead of a password. Grab the machine you intend to use to SSH into the MAAS-deployed nodes and retrieve the RSA key pair of that machine. If you are on a Linux machine and you have already generated SSH keys, the key can usually be retrieved from the default directory by entering the following command.

```bash
$ sudo cat /home/$USERNAME/.ssh/id_rsa.pub
```

More than likely though, you will need to generate your “private/public key pair”. To generate the RSA key pair enter the following command. Follow the shell prompts and take note of the directory and passphrase (if any) used.

```bash
$ ssh-keygen -t rsa -b 4096
```
If you saved the RSA key pair to the default directory you will be able to retrieve your public key with the following command.

```bash
$ sudo cat /home/$USERNAME/.ssh/id_rsa.pub
```
The output of this will look something like this:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC/2sats4R9ino4qZfNiPFL+PYAVjY6vdRP9G9KJZPyxeOhbcpUPQ88gf2TpjEhy9CICweJ9Wt71uhkyh7iL4nW2ZN1oPyF9jzJTTfxPyAGjV6cOOPC1ispOgmFu3ZklGpRXAaNYsgIpp5FGGyioIAatzDuBjHffYEEXqmj6qkamRhCZZdMl7EBxa0rwsTlOfdY72ohpqwqUQr6QFMpvnSO1VFTjQJIXSvpDBigdhYsL0TxH+4S9UG+ywBQunhaOpX34Z6bIQT1rfZbDEfMh/C6qcQQSFghclhCjvqn2V7FNkiYcIuImf18xLQMnZU2NKRM9x2inhdjWUXvpihJbVzZuZ03AAm0NF1H2va3fBMJrTAXKLf4wTj1k+841+d59EVD6eUGW94p3sAzkC39FQMizv1PovFCSFFjNt5zFJWEFjMNYJWvLmx2DUnEnn7Qsmjw8U66YcI03FxWwW1sab+hbsS9y5rjOlUIDc1zjHlfRZmqf0obAghcQ852WvRYKv6TJ/WE2RQtUXg0/lJjtG70+mnrOxqAE3Pk2kT2gZumhk01VlvZCVBz8bLNdZ3r5jjEVYJ/MChQW7Uv828EtA1fayXqXLtIrYZxCPcvA99GKN3/xT0HNGWSYdrelVf0CZ207aVjJuP2mfDr5DWZ9PhXxiiz/hF6TP1abKOqNfsb8w== controller@maas
```

You will need to copy this output and paste it into the MAAS web GUI.
[INSERT IMAGE]

With the RSA public key imported, continue through the initial set-up. Once you land on the page below there are a couple things that still need to be configured.
[INSERT IMAGE]

#### Configuration
#### Enable DHCP
This step only applies if you’ve decided to let MAAS serve DHCP, as described in the Serving DHCP section of this guide.

To enable DHCP click on the “Controllers” tab at the top, and select the controller (called maas.maas in this case).



## OpenStack
We will be using JuJu to orchestrate the OpenStack deployment one individual “charm” container at a time. OpenStack can also be deployed via “bundles” - for instructions on a bundle deployment follow the official guide on the OpenStack website.

Per the official guide:
*“There are many moving parts involved in a charmed OpenStack install. During much of the process there will be components that have not yet been satisfied, which will cause error-like messages to be displayed in the output of the juju status command. Do not be alarmed. Indeed, these are opportunities to learn about the interdependencies of the various pieces of software. Messages such as Missing relation and blocked will vanish once the appropriate applications and relations have been added and processed.”*

### Installation
First, make sure you are interacting with the correct juju controller and model.
```bash
$ juju switch juju-controller:openstack
```

With that out of the way, it may be useful to monitor the deployment progress either through the JuJu dashboard or using the JuJu status command.

#### Monitoring the OpenStack Deployment
To access the JuJu dashboard, enter the following command.
```bash
$ juju dashboard
```
The output will look something like this.


```bash
$ juju dashboard
Dashboard 0.6.2 for controller "juju-controller" is enabled at:
  https://192.168.1.240:17070/dashboard
Your login credential is:
  username: admin
  password: 536d568fb04609daee40c61abc36844d

```
Alternatively, to monitor JuJu status use this command.


```bash
$ watch -n 1 -c juju status --color
```
### Ceph-OSD
The first charm we will deploy is the Ceph-osd charm or “object storage device”. This is done by first creating a YAML file called **ceph-osd.yaml** which specifies the charm’s configurations, where ```/dev/sdb``` is the path to the

```yaml
ceph-osd:
  osd-devices: /dev/sdb
  source: cloud:focal-wallaby
```
Next, we will deploy the Ceph-osd charm to 4 nodes. Since we only have compute nodes, the “compute” tag constraint is not necessary strictly speaking, but it is good practice for future deployments.


```bash
$ juju deploy -n 4 --config ceph-osd.yaml --constraints tags=compute ceph-osd
```

With this being the first charm deployed to the compute nodes, MAAS will first power on the servers and deploy Ubuntu 20.04 to each node - and once that is done, JuJu will proceed to install the charm software. You can monitor the progress using the JuJu status command output - it will look like this once the Ceph-osd charm is finished installing.


```bash
Model      Controller       Cloud/Region  Version  SLA          Timestamp
openstack  juju-controller  maas-cloud    2.8.10   unsupported  05:09:54Z

App       Version  Status   Scale  Charm     Store       Rev  OS      Notes
ceph-osd  16.2.0   blocked      4  ceph-osd  jujucharms  310  ubuntu

Unit         Workload  Agent  Machine  Public address  Ports  Message
ceph-osd/0  blocked  idle  0  192.168.1.241  Missing relation: monitor
ceph-osd/1* blocked  idle  1  192.168.1.235  Missing relation: monitor
ceph-osd/2  blocked  idle  2  192.168.1.236  Missing relation: monitor
ceph-osd/3  blocked  idle  3  192.168.1.237  Missing relation: monitor

Machine  State    DNS            Inst id  Series  AZ       Message
0        started  192.168.1.241  node0  focal   default  Deployed
1        started  192.168.1.235  node1  focal   default  Deployed
2        started  192.168.1.236  node2  focal   default  Deployed
3        started  192.168.1.237  node3  focal   default  Deployed
```
### Nova Compute
With the Ceph-osd installed, the Nova Compute charm is next on the list of deployment. The process is very similar across charms. First, create the file **nova-compute.yaml**.



```yaml
nova-compute:
  config-flags: default_ephemeral_format=ext4
  enable-live-migration: true
  enable-resize: true
  migration-auth-type: ssh
  openstack-origin: cloud:focal-wallaby
```
Then, deploy the Nova Compute charm using the file nova-compute.yaml configurations.

```bash
$ juju deploy -n 3 --to 1,2,3 --config nova-compute.yaml nova-compute
```
