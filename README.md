# Mininet-Floodlight
Networking Project

Setup an SDN network with 5 hosts, with host 5 sniffing traffic on host 4.

Floodlight is the SDN controller.

## Setting up floodlight


In Progress

## Setting up mininet


`sudo mn --topo single,5 --controller=remote,ip=127.0.0.1,port=6653`

Spawns a single layer network, with 5 hosts connected to a switch.

The switch is connected to a remote controller, which is the floodlight service you setup earlier.

If your floodlight service is running on another machine, configure the `ip` and `port` accordingly.

## Setting up Snort


In Progress

## Configuring Snort Rules

Download Snort rules here https://www.snort.org/downloads

Edit your `snort.conf` accoringly to remove any preprocessors you don't have

If you're having trouble, `sudo find / -type f -name snort.conf`

## Mirroring Port h4 to h5


`s1 ovs-vsctl -- set Bridge "s1" mirrors=@m -- --id=@s1-eth4 get Port s1-eth4 -- --id=@s1-eth5 get Port s1-eth5 -- --id=@m create Mirror name=e4toe5 select-dst-port=@s1-eth4 output-port=@s1-eth5`

## Running Snort on h5


`mininet> xterm h5`

In the new terminal

`ifconfig` to get the adapter name

`snort -i <adapter name> -v -c <snort.conf location> &`

h5 is now sniffing traffic on h4
