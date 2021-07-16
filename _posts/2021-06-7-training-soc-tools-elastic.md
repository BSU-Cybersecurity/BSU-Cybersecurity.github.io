---
layout: post
title: "Tools - Elk Stack"
date: 2021-06-18 09:00:00 -0500
categories: [Training-SOC, Tools]
tags: [tools, elk, soc, client]
---
# Installing and Configuring ELK

## Initial Installs

1. Install java with `sudo apt-get install openjdk-8-jdk`
   - Check java version with `java -version`
2. Install apt-transport package `sudo apt-get install apt-transport-https`
3. Add Elastic Repo
   `echo “deb https://artifacts.elastic.co/packages/7.x/apt stable main” | sudo tee –a /etc/apt/sources.list.d/elastic-7.x.list`

## Elasticsearch

Elasticsearch acts as a powerful search, analysis and storage tool.

### Install Elasticsearch

`sudo apt-get install elasticsearch`

### Configuration Elasticsearch

1. Edit configuration file with `sudo nano /etc/elasticsearch/elasticsearch.yml`

   - Uncomment lines
     >       #network.host: 192.168.0.1 (Replace with localhost)
     >       #http.port: 9200
   - Add `discovery.type: single-node` in Discovery section
2. Set the heap size 
   - `sudo nano /etc/elasticsearch/jvm.options`
   - Edit -Xms and -Xmx to desired heap size
      > No more than half of ram
      
### Starting and Testing Elasticsearch

1. Start Elasticsearch
   - Begin service `sudo systemctl start elasticsearch.service`
   - Enable on boot `sudo systemctl enable elasticsearch.service`

2. Test

- `curl –X GET “localhost:9200”`
![Desktop View](https://phoenixnap.com/kb/wp-content/uploads/2021/04/test-elasticsearch-service.png)
_Display of curl command_

## Kibana

Kibana is a graphical user interface for displaying data.

### Install Kibana

`sudo apt-get install kibana`

1. Open config file `sudo nano /etc/kibana/kibana.yaml`
2. Uncomment lines
   - `#server.port: 5601`
   - `#server.host: “localhost”`
   - `#elasticsearch.hosts: [“http://localhost:9200”]`

## Starting and Testing Kibana

1. Start Kibana
   - Begin service `sudo systemctl start kibana`
   - Enable on boot `sudo systemctl enable kibana`
> Allow past UFW if enabled `sudo ufw allow 5601/tcp`
2. Test Kibana
   - Browse to http://localhost:5601 
