---
title: "FreeBSD Jails Loopback IP"
date: 2020-11-19T23:00:00Z
weight: 1
aliases: ["/freebsd-jails-loopback-ip"]
tags: ["FreeBSD", "Jails", "PF"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1605880424/jjba-site/blog/freebsd/VirtualBox_FreeBSD_12.1_07_05_2020_11_59_43_ilhmzr.png"
    alt: "FreeBSD"
    caption: ""
    relative: false
    hidden: false
comments: false
description: "Learn how to set up a loopback interface communication for your iocage jails with pf firewall."
disableHLJS: false
---

In order to keep all of the jails behind a single public IP address, you’ll need to set up a loopback interface and direct the incoming and outgoing traffic of your Jails with a proper firewall like `pf`. Learn how to get started on an amazing setup.

There are multiple ways to manage your jails, but iocage is one of the more popular ones. In terms of jail managers it’s a fairly new player in the game of jail management and is being very actively developed. It has a full rewrite in Python, and while the code in the background might be different, the actual user interface has stayed very similar.

Iocage makes use of ZFS clones in order to create base jails, which allow for sharing of one set of system packages between multiple jails, reducing the amount of resources necessary. Alternatively, jails can be completely independent of each other; however, using a base jail makes it easier to update multiple jails as well.

You can install it from a binary package:

`pkg install -y py37-iocage`

## Setting up the zpool

It is a great idea to separate the jails storage device from the host system's storage. Ideally in a separate physical device, but a separate partition will also do.

This means that you would be in a situation where your `zroot` is already setup, and you have a free device/partition where you can create a second `zpool` to be used specifically for the Jails.

Doing a `gpart show` in my machine reveals the following setup:

```sh
=>      40  41942960  da0  GPT  (20G)
            40      1024    1  freebsd-boot  (512K)
          1064       984       - free -  (492K)
          2048   4194304    2  freebsd-swap  (2.0G)
       4196352  20971520    3  freebsd-zfs  (10G)
      25167872  16773120    4  freebsd-zfs  (8.0G)
      41940992      2008       - free -  (1.0M)
```    

This means I want to create a new pool on `da0p4` with the name `iocage` since my system has been installed on `da0p3`.

This can be achieved very simply with the command:

```sh
zpool create iocage da0p4
```  
          

With the pool created, you can now activate `iocage` to use the wanted pool:

```sh  
iocage activate iocage
```
          

Then fetch the latest FreeBSD release (or the one you prefer) to be used to setup jails:

```sh
iocage fetch
```

Add the following new line to your `/etc/fstab` file to improve the jail performance:

```sh
fdescfs /dev/fd fdescfs rw 0 0
```
          

## Setting up the network

You could go ahead and create your jail right off, but the networking will not work out of the box. Take a minute and do some basic network configuration before creating your first jail.

### Virtual Interface

For our internal network, we create a cloned loopback device called `lo1`. Therefore we need to customize the `/etc/rc.conf` file, adding the following two lines:

In order to keep all of the jails behind a single public IP address, you’ll need to set up a new network interface. This new interface will be a clone of the loopback interface which will have an IP address assigned to it. You can use any RFC 1918 address space. In this tutorial, `192.168.0.1` will be used. Add the following to `/etc/rc.conf` to get the new interface set up:

```sh
# /etc/rc.conf
# Setup the interface that all jails will use
cloned_interfaces="lo1"
ipv4_addrs_lo1="192.168.0.1/24"

gateway_enable="YES"

# Enable port forwarding and packet filtering
pf_enable="YES"

# Enable iocage at startup
iocage_enable="YES"
```
    

This defines a `/24` network, offering IP addresses for a maximum of 254 jails:

```sh
joe@devjails > ipcalc 192.168.0.1/24

Address:   192.168.0.1          11000000.10101000.00000000. 00000001
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   192.168.0.0/24       11000000.10101000.00000000. 00000000
HostMin:   192.168.0.1          11000000.10101000.00000000. 00000001
HostMax:   192.168.0.254        11000000.10101000.00000000. 11111110
Broadcast: 192.168.0.255        11000000.10101000.00000000. 11111111
Hosts/Net: 254                   Class C, Private Internet
```
    

Then we need to restart the network. Please be aware of currently active SSH sessions as they will be dropped during restart. It’s a good moment to ensure you have KVM or physical access to that server ;-)

With those entries in `/etc/rc.conf` the interfaces will all be created and configured at boot time, so give the machine a nice reboot

After booting, an `ifconfig` should now show a new interface:

```sh
lo1: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
    options=680003<RXCSUM,TXCSUM,LINKSTATE,RXCSUM_IPV6,TXCSUM_IPV6>
    inet 192.168.0.1 netmask 0xffffff00
    groups: lo
    nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
```
    

### NAT for packet forwarding

Now that you have an interface to use with your jails, you’ll need to get some packet forwarding set up. Edit `/etc/pf.conf` (which will be empty by default) according to the following

```sh  
ext_if="em0"
jail_if="lo1"

# Public IP address
ip_pub="192.168.178.40"

# Packet normalization
scrub in all

# Skip packet filtering on loopback interfaces
set skip on lo0
set skip on lo1

# Allow outbound connections from within the jails
nat pass on $ext_if from 192.168.0.0/24 to any -> $ip_pub
```
    

The first 3 lines define a couple of useful variables – called macros in PF parlance. The nat line instructs PF to mask outbound traffic from the jails (all of them) behind the IP address of the external interface. In short, all of your outbound jail traffic will come from the IP address of your droplet.

With that in place, you can enable and start PF:

```sh
service pf enable
service pf start
```

Before you load the firewall ruleset, test the config file to ensure that all is well:

```sh
pfctl -nvf /etc/pf.conf
```
    

That command will cause PF to parse the rules in /etc/pf.conf, and print them back out to the console. If there are syntax errors, they will be called out.

Your output should be very similar to:

```sh
ext_if = "em0"
jail_if = "lo1"
IP_PUB = "192.168.178.40"
set skip on { lo0 }
set skip on { lo1 }
scrub in all fragment reassemble
nat pass on em0 inet from 192.168.0.0/24 to any -> 192.168.178.40
```
    

Then reload the firewall rules with:

```sh
pfctl -F all -f /etc/pf.conf
```
          

## Creating the first jail

It’s time to create & start a jail:

```sh
iocage create -n "mydb" -r 12.2-RELEASE ip4_addr="lo1|192.168.0.2"
iocage start mydb
```
    

Those commands will have a lot of output, and may end with a warning. You can safely ignore the warnings. The jail needs to be told how to do DNS lookups. The simple way to solve this is to copy the hosts /etc/resolv.conf into the jail. Assuming your zpool was setup to point to /iocage:

```sh
mkdir /iocage/iocage/jails/mydb/etc
cp /etc/resolv.conf /iocage/iocage/jails/mydb/etc/resolv.conf
```
    

Then start the machine and enter the console:

```sh
iocage start mydb
iocage console mydb
```
    

There will be a lot of stuff on the console – very similar to what you should have seen when you first connected to your Droplet.

Notice that your prompt has changed. Congratulations, you are inside your jail! It’s probably a good idea to test your connectivity to the outside world, but by default, and for security reasons, FreeBSD jails are not allowed to ping. To test connectivity, you can use telnet:

```sh
telnet www.digitalocean.com 80
```
    

That command will open up a very basic connection to a webserver at Digital Ocean. You can get out of it pressing Control-\] to close the connection, followed by quit to close telnet.

```sh 
Trying 104.16.24.4...
Connected to www.digitalocean.com.
Escape character is '^]'.
^]
telnet> quit
Connection closed.
```
    

Now you are ready to install your desired software and set up the wanted services in the jail. It is a good idea to get familiar with `iocage` since it has many handy commands to manage the jails.

In my case, I installed MariaDB on this jail, and for that purpose, opened up a port redirection in the firewall, so that traffic from the public-facing interface will be redirected to the internal jail's port. You can do this by adding this line to the end of `/etc/pf.conf`, with `192.168.0.4` being the internal IP I have assigned to this jail:

```sh
rdr on $ext_if proto tcp from any to $ip_pub port 3306 -> 192.168.0.4 port 3306
```
          

Then reload the firewall rules with:

```sh
pfctl -F all -f /etc/pf.conf
```
          

## Locking down the firewall

The way things stand right now, your jail is working, but the host system is pretty wide-open. If you edit your `/etc/pf.conf` once more, we can restrict all inbound traffic that is not destined for a jail except for SSH.

Edit your file to look like:

```sh
ext_if="em0"
jail_if="lo1"
jail_net= $jail_if:network

# Public IP address
ip_pub="192.168.178.40"

# Packet normalization
scrub in all

# Skip packet filtering on loopback interfaces
set skip on lo0
set skip on lo1

# Allow outbound connections from within the jails
nat pass on $ext_if from 192.168.0.0/24 to any -> $ip_pub

# Port redirect for MariaDB
rdr pass on $ext_if proto tcp to port 3306 -> 192.168.0.4

# Set the default: block everything
block all

# Allow the jail traffic to be translated
pass from { lo0, $jail_net } to any keep state

# Allow SSH in to the host
pass in inet proto tcp to $ext_if port ssh

# Allow OB traffic
pass out all keep state
```

Now you have a secure machine, that only takes traffic from the wanted ports. This starts feeling production ready!

## Jail tweaks

Jump back into your jail with `iocage console -f mydb` and add the following 2 lines to `/etc/rc.conf`:

```sh
# Disable the RPC daemon
rpcbind_enable="NO"
# Clear /tmp at startup
clear_tmp_enable="YES"
```
          

Go on now, create your universe of inter-connected systems, welcome to the world of **FreeBSD** jails!