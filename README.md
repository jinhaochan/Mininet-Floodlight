# Mininet-Floodlight-Snort
A small networking Project

Setup an SDN network with 5 hosts, with host 5 sniffing traffic on host 4 using Snort.

This project will have 3 malicious actors (h1 h2, h3), a victim machine (h4) and an IDS using Snort sniffer (h5)

We will configure the network such that the 5 hosts are connected to the a switch, and the switch is connected to Floodlight SDN Controller. h1, h2 and h3 will attack h4 with a DoS attack, and h5 will be able to pick it up using Snort rules.


## Setting up floodlight

Follow instructions here:

https://floodlight.atlassian.net/wiki/spaces/floodlightcontroller/pages/1343544/Installation+Guide

Floodlight GUI will be running on http://localhost:8080/ui/pages/index.html


## Setting up mininet

Clone and install:

```
$ git clone git://github.com/mininet/mininet
$ cd mininet
$ sudo ./util/install.sh -a
```

Mininet is now installed.

Spawn your network with the command:

`$ sudo mn --topo single,5 --controller=remote,ip=127.0.0.1,port=6653`

Spawns a single layer network, with 5 hosts connected to a switch.

The switch is connected to a remote controller, which is the floodlight service you setup earlier.

*Note: your port specified in this command should be `6653` and not `8080`. `8080` is used for showing the UI, `6653` is used for communicating with your switch.*

If your floodlight service is running on another machine, configure the `ip` and `port` accordingly.

## Setting up Snort (In Ubuntu)

Before installing Snort, you have to first install DAQ

Updating your apt
```
$ apt-get update -y 
$ apt-get upgrade -y
```

Installing dependencies
`$ apt-get install openssh-server ethtool build-essential libpcap-dev libpcre3-dev libdumbnet-dev bison flex zlib1g-dev liblzma-dev openssl libssl-dev`

Grabbing DAQ source (Change the value of the version to the lastest one)
```
$ wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
$ tar xvf daq-2.0.6.tar.gz
$ cd daq-2.0.6
```

Configure and install DAQ
```
$ ./configure && make && make install
```

Now that you've installed DAQ, you can proceed to install Snort

Grabbing Snort source (Change the value of the version to the lastest one)
```
$ wget https://www.snort.org/downloads/snort/snort-2.9.8.3.tar.gz
$ tar vzf snort-2.9.8.3.tar.gz
$ cd snort-2.9.8.3
```
Configure and install Snort
```
$ ./configure --enable-sourcefire && make && make install
```

Link the libraries
```
$ ldconfig
```

Creating a symbolic link to Snort binary
```
$ ln -s /usr/local/bin/snort /usr/sbin/snort
```

Test it out!
```
$ snort -V

 ,,_ -*> Snort! <*- 
 o" )~ Version 2.9.8.3 GRE (Build 383) 
 '''' By Martin Roesch & The Snort Team: http://www.snort.org/contact#team 
     Copyright (C) 2014-2015 Cisco and/or its affiliates. All rights reserved. 
     Copyright (C) 1998-2013 Sourcefire, Inc., et al. 
     Using libpcap version 1.7.4 
     Using PCRE version: 8.38 2015-11-23 
     Using ZLIB version: 1.2.8
     
```

After Snort is up and running, you will need to create directory structures for it
```
$ mkdir /etc/snort
$ mkdir /etc/snort/preproc_rules
$ mkdir /etc/snort/rules
$ mkdir /var/log/snort
$ mkdir /usr/local/lib/snort_dynamicrules
$ touch /etc/snort/rules/white_list.rules
$ touch /etc/snort/rules/black_list.rules
$ touch /etc/snort/rules/local.rules
$ chmod -R 5775 /etc/snort/ 
$ chmod -R 5775 /var/log/snort/ 
$ chmod -R 5775 /usr/local/lib/snort
```

## Configuring Snort Rules

Download Snort rules here https://www.snort.org/downloads

Edit your `snort.conf` accoringly to remove any preprocessors you don't have

If you're having trouble, `$ sudo find / -type f -name snort.conf`

Adding a rule in `snort.conf` to catch DoS by ICMP packets

`alert icmp any any -> any any (threshold: type both, track by_dst, count 70, seconds 10; msg: "DoS by ICMP detected"; sid:1001;)`

## Mirroring port h4 to h5 and sniff using Snort

Command to mirror h4 traffic to h5
`mininet> s1 ovs-vsctl -- set Bridge "s1" mirrors=@m -- --id=@s1-eth4 get Port s1-eth4 -- --id=@s1-eth5 get Port s1-eth5 -- --id=@m create Mirror name=e4toe5 select-dst-port=@s1-eth4 output-port=@s1-eth5`

Now all traffic that is flowing into h4 will be mirrored onto h5, where Snort is running.

`mininet> xterm h5`

In the new terminal spawned for h5, run:

`h5> ifconfig` to get the adapter name

`h5> snort -i <adapter name> -v -c <snort.conf location> &`

h5 is now sniffing traffic on h4

## Starting the attack

`mininet> h1 ping -f h4`

You should see in your `/var/log/snort/alert` the message `"DoS by ICMP detected"`
