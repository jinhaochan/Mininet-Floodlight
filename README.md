# Mininet-Floodlight-Snort
Networking Project

Setup an SDN network with 5 hosts, with host 5 sniffing traffic on host 4 using Snort.

Floodlight is the SDN controller.

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

`sudo mn --topo single,5 --controller=remote,ip=127.0.0.1,port=6653`

Spawns a single layer network, with 5 hosts connected to a switch.

The switch is connected to a remote controller, which is the floodlight service you setup earlier.

*Note: your port specified in this command should be `6653` and not `8080`. `8080` is used for showing the UI, `6653` is used for communicating with your switch.*

If your floodlight service is running on another machine, configure the `ip` and `port` accordingly.

## Setting up Snort


In Progress

## Configuring Snort Rules

Download Snort rules here https://www.snort.org/downloads

Edit your `snort.conf` accoringly to remove any preprocessors you don't have

If you're having trouble, `sudo find / -type f -name snort.conf`

Adding a rule in `snort.conf` to catch DoS by ICMP packets

`alert icmp any any -> any any (threshold: type both, track by_dst, count 70, seconds 10; msg: "DoS by ICMP detected"; sid:1001;)`

## Mirroring Port h4 to h5

`s1 ovs-vsctl -- set Bridge "s1" mirrors=@m -- --id=@s1-eth4 get Port s1-eth4 -- --id=@s1-eth5 get Port s1-eth5 -- --id=@m create Mirror name=e4toe5 select-dst-port=@s1-eth4 output-port=@s1-eth5`

## Running Snort on h5

`mininet> xterm h5`

In the new terminal

`ifconfig` to get the adapter name

`snort -i <adapter name> -v -c <snort.conf location> &`

h5 is now sniffing traffic on h4

## Starting the attack

`mininet> h1 ping -f h4`

You should see in your `/var/log/snort/alert` the message `"DoS by ICMP detected"`
