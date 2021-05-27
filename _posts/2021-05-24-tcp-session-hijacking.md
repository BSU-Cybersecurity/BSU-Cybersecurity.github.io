---
layout: post
title: "TCP Session Hijacking Attack"
date: 2021-05-24 09:00:00 -0500
categories: Network-Security-&-Defense Attacks
tags: network ssh tcp attack hijacking
---
### Overview
In this demo, we will hijack a telnet session and inject a malicious command.

### Setup
For this attack you will need 3 Linux VMs.
* VM1 as the victim (SSH client).
* VM2 as the ssh server.
* VM3 as the attacker.

The 3 VMs reside in the same network.

### Attack
The attacker, mimicking the client, needs to use "netwox 40" to perform this attack.

The command should be in this format:

```bash
sudo netwox 40 --ip4-src source_ip --ip4-dst destination_ip --tcp-src source_port --tcp-dst destination_port --tcp-seqnum sequence_number --ip4-ttl ttl_value --tcp-window window_size --tcp-ack --tcp-acknum acknowledge_number --tcp-data "putyourdatahere,in hex format"
```

> **Note**: The attacker needs to use Wireshark to find out the parameter information for the above command: pay attention to the last packet sent from the client to the server, it will tell you what the next sequence number should be, and what the acknowledge number should be.

In the command above, ```--tcp-data ``` specifies the data you want to transfer, in this case, it should be the command you want to execute on the server. Make sure your command is sandwiched by two newline signs ```\r```, so that your command will not be concatenated with strings (typed by the victim). (i.e., ```\r command \r```, space is needed).

> **Note2**: This is how you can convert a string into hex numbers:
```bash
$ python
>>> "Hello Boise".encode("hex")
'48656c6c6f20426f697365'
```
