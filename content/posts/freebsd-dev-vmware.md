---
title: "FreeBSD Development GNOME VMWare"
date: 2021-03-28T09:00:00Z
weight: 1
aliases: ["/freebsd-dev-vmware"]
tags: ["FreeBSD", "VMWare", "Dev", "GNOME"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1603433377/jjba-site/blog/freebsd/p8jyvxodxkows5fgsnp4_ri0yvg.png"
    alt: "FreeBSD"
    caption: ""
    relative: false
    hidden: false
comments: true
description: "Setup a full FreeBSD development environment with GNOME in a VMWare virtual machine"
disableHLJS: false
---

In this tutorial I will proceed to explain how to setup a full developer environment in a VMWare virtual machine. This will help you test more closely to the production environments, and will allow you to develop directly in Jails, and run databases / other services in a containerised and controlled manner, without compromising you base system.

We will install a GUI on FreeBSD so you can run the IDE of your choice and have a full blown capable system.

Firstly you should perform a base installation of FreeBSD with Root on ZFS, following [my tutorial hosted on this very website](https://jjba.dev/posts/freebsd-root-on-zfs-partitions/).
You may prefer a different partitioning or combination of hard disks, that will be up to you. I will use 1 Virtual HDD with 2 partitions, one for the system zpool and one for the Jails zpool, as stated in the tutorial.

Once you have a properly partitioned and clean FreeBSD install with a user of your choice you can follow the rest of this tutorial.

We will begin by updating the system with:

```
freebsd-update fetch
freebsd-update install
```

Then the package manager:

```
pkg update
pkg upgrade
```

We will then install the Open VM tools and graphics driver which will enable several functionalities in the machine.

```
pkg install open-vm-tools
pkg install xf86-video-vmware
pkg install xf86-input-vmmouse
```

Then with that installed, add the following to `/etc/rc.conf` to enable services for GUI support and VMWare extensions.

```
# Your keyboard and mouse won't work without this.
hald_enable="YES"
dbus_enable="YES"

# Mouse
moused_enable="YES"

# VMWare Tools
vmware_guest_vmblock_enable="YES"
vmware_guest_vmhgfs_enable="YES"
vmware_guest_vmmemctl_enable="YES"
vmware_guest_vmxnet_enable="YES"
vmware_guestd_enable="YES"
```

You should then perform a reboot to ensure all still works as expected.

Then we will proceed to installing Gnome 3. You could opt for other environments, but Gnome is my preferred one, and it is very well-supported and runs on the trusty GTK3 libraries.

In order to do that run:

```
pkg install gnome-desktop gdm xorg gnome3
```

To enable GNOME at boot and login graphically add to the `/etc/rc.conf`

```
gdm_enable="YES"
gnome_enable="YES"
```

We will need to add a virtual device for the GNOME processes in our `/etc/fstab`:

```
proc /proc procfs rw 0 0
```

After all these changes you can reboot and you should be able to login graphically to a clean and un-bloated GNOME desktop. You can then proceed to install the IDE and software of you choice to make your development environment work nicely.

Myself, I then activated the zpool in the separate HDD partition for my jails, and followed my tutorial for setting that up, which can be read [here](https://jjba.dev/posts/freebsd-jails-loopback-ip/).

You can then proceed to rice your desktop, install your favourite programs, shell and etc.

## Welcome to the magical world of FreeBSD!