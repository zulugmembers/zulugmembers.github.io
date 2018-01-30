---
layout: post
title: "Linux Network Troubleshooting 101"
description: ""
category: [Networking]
tags: Troubleshooting,Bash,Introductory
---

In this post we attempt to give you pointers on where to start looking when troubleshooting your network especially if you are starting out with Linux.

We've broken the article into the following:
1. [IP Addressing](#ip-addressing)
2. [Devices and Drivers](#devices-and-drivers)
3. [Network Related Logs](#network-related-logs)
4. [Common Network Troubleshooting Commands](#common-network-troubleshooting-commands)

Let's dig right in

## IP Addressing ##

On joining a network you get or assign an IP Address to an interface. This is either done dynamically via a [DHCP server](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) or manually.

You'll want to get a few details about your IP Address:

This can be done from the cli or command line using either commands in the [net-tools](https://wiki.linuxfoundation.org/networking/net-tools) or [iproute2](https://en.wikipedia.org/wiki/Iproute2) packages.

We'll use `iproute2` tools since the net-tools commands are being obsoleted which includes things like `ifconfig`. 

{% highlight bash %}
# Getting Basic IP information
ip address
hostname -I

# Filtering IP information for a specific interface e.g eth0 or wlan0
ip address show dev eth0
# Filtering IP by label e.g. docker for your docker containers
ip address show label docker*
# Filtering by IP version
ip -6 address show

{% endhighlight %}


You can even get complicated and pass the ip information around to some other commands. For example the snippet below obtains the IP Address class, Number of Hosts, Netmask etc. from [ipcalc](https://linux.die.net/man/1/ipcalc)

{% highlight sh %}

ip addr show dev wlan0 | egrep -o 'inet ([0-9]{1,3}\.?)*/[[:alnum:]]{2}' | cut -d ' ' -f2 | xargs ipcalc

# Command Output
Address:   192.168.1.22         11000000.10101000.00000001. 00010110
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   192.168.1.0/24       11000000.10101000.00000001. 00000000
HostMin:   192.168.1.1          11000000.10101000.00000001. 00000001
HostMax:   192.168.1.254        11000000.10101000.00000001. 11111110
Broadcast: 192.168.1.255        11000000.10101000.00000001. 11111111
Hosts/Net: 254                   Class C, Private Internet

{% endhighlight %}

The above should give you a starting point on what IP is assigned, the network class etc. Ideally this data helps troubleshoot DHCP issues, and answer whether you're in the right subnet or have a IP address conflict with `ip address show dadfailed` for example.

Further details on the `ip` are available from the man pages. 

To alter the ip address you can use `ip address add` or `ip address change` e.g.

{% highlight bash %}
# To change interface wlan0 address to 192.168.1.22/24 
ip address change 192.168.1.22/24 dev wlan0
{% endhighlight %}

## Network Related Logs

If you have [systemd](https://en.wikipedia.org/wiki/Systemd) a good starting point is [journalctl](https://www.commandlinux.com/man-page/man1/journalctl.1.html) you can run a command like: 

```
journalctl -t NetworkManager
```

This gets all messages logged by the `NetworkManager`.
Alternatively whilst using `journalctl` you can just get all it's unfiltered output and search on your desired keyword.

In addition you may get network log entries from the `dmesg` command and by reading output from your `/var/log/syslog` file.

## Devices and Drivers

To get the network devices or interfaces we can use a number of commands listed below:

{% highlight bash %}
# Get network device specifications, drivers and basic IP data
lshw -c network
lspci 
iwconfig
# ..where wlan0 is an wireless interface on your system
iwconfig wlan0 
iwlist wlan0 rate

# Get network device, kernel drivers in verbose mode on bus 02:00.0 assuming you know the bus from lshw or lspci for example
lspci -kv -s 02:00.0

# Getting devices from Network Manager via nmcli
# man nmcli
nmcli d

# Retrieve network device and some stats (e.g. packets,errors,drops,bytes etc) from procfs or filesystem
cat /proc/net/dev

# Lookup virtual network devices or bridges from filesystem
ls /sys/devices/virtual/net/
cat /sys/devices/virtual/net/*/address

{% endhighlight %}

From this data we can troubleshoot and resolve driver issues (are drivers loaded?) for example, using tools like `modprobe`, `ethtool` etc. We can also get the hardware capabilites of our network interfaces like bandwidth capacity.

## Common Network Troubleshooting Commands:

The table below lists a few common commands you might want to also use when working on Linux:

| Command     | Description                                                                    |
|-------------|--------------------------------------------------------------------------------|
| [ping](https://linux.die.net/man/8/ping)    |Test reachability on an IP network
| [mtr](https://linux.die.net/man/8/mtr)         |combines the functionality of the traceroute and ping programs in a single network diagnostic tool
| [tracepath](https://linux.die.net/man/8/tracepath) |traces path to a network host discovering MTU along this path 
| [iperf](https://linux.die.net/man/1/iperf)       |performing network throughput measurements
| [nslookup](https://linux.die.net/man/1/nslookup)    | program to query Internet domain name servers
| [dig](https://linux.die.net/man/1/dig)         |a flexible tool for interrogating DNS name servers
 

The above list of commands for checking network issues is not exhaustive and meant to just point you in the right direction

Troubleshooting network issues is a broad topic and we have just scratched the surface.  