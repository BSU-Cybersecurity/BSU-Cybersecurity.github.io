---
layout: post
title: "Tools - Ghosts Automation of NPCS "
date: 2021-06-18 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, ghosts, traffic, generation, soc, client]
---
![Desktop View](https://github.com/cmu-sei/GHOSTS/raw/master/assets/ghosts-logo.jpg)
_GitHub: <https://github.com/cmu-sei/GHOSTS>_

## Install

The install/config for ghosts requires a server and client. Easy installation uses docker. The client supports Windows and Linux machines. This guide uses Ubuntu for the server and client.

### Ghost Linux Server Install

#### Install Docker
- Dependencies  
  ```console
    $ sudo apt-get update
    $ sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
    ```
- GPG key
    ```console
    $ curl -fsSL https://download.docker.com/linux/ubuntu gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
- Setup stable repo 
     ```console
     $ echo \
      "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
#### Install Docker Engine
- Get and Install Packages  
    ```console
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
  ```
      
- Verify install
    ```console 
  $ sudo docker run hello-world
    ```
#### Install Docker Compose
   - Download stable release
      ```console
      $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
      ```
   - Allow execution for binary
      ```console
      $ sudo chmod +x /usr/local/bin/docker-compose
      ```
   - Test install
      ```console
      $ docker-compose --version
      ```
#### Install the Server
   - Download the docker yaml
     ```console
     $ wget https://raw.githubusercontent.com/cmu-sei/GHOSTS/master/src/Ghosts.Api/docker-compose.yml
     ```
   - Stand up ghost containers
     ```console
     $ docker-compose up -d
     ```
   - Test API
     > Navigate to http://localhost:5000/api/home
   - Set up Grafana
      1. Access Grafana at :3000 via web browser
      2. Default login <b>admin/admin</b>
      3. Set up a datasource named "ghosts" to the ghosts postgres database
      4. import https://github.com/cmu-sei/GHOSTS/blob/master/configuration/grafana/dashboards/GHOSTS-default%20Grafana%20dashboard.json
      > If restarts occur due to insufficient permissions, chown the host location of the docker-compose file.

### Ghost Linux Client Install
#### Ubuntu Install
  - .NET Install
    ```console
    $ wget https://packages.microsoft.com/config/debian/10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    $ sudo dpkg -i packages-microsoft-prod.deb
    $ rm packages-microsoft-prod.deb
    ```
    ```console 
    $ sudo apt-get update; \
      sudo apt-get install -y apt-transport-https && \
      sudo apt-get update && \
      sudo apt-get install -y aspnetcore-runtime-5.0
     ```
  - Download Client
    - Navigate to https://cmu.box.com/s/1696ptcmdsu36f0qicx100kkesf4t3gv and download zip
    - Unzip to user directory that ghosts should run in 
      ```console
      $ unzip ghosts-client-linux-v3.0.zip
      ```



