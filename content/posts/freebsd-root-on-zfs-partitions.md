---
title: "FreeBSD Root on ZFS - Partitions"
date: 2020-11-19T23:00:00Z
weight: 1
aliases: ["/freebsd-root-on-zfs"]
tags: ["FreeBSD", "ZFS"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1605883892/jjba-site/blog/freebsd/bsdinstall-zfs-partmenu_mfcis5.png"
    alt: "FreeBSD"
    caption: ""
    relative: false
    hidden: false
comments: false
description: "This guide assumes you want to install FreeBSD and manually create the ZFS pool, to change default settings, or to not use the entire disk."
disableHLJS: false
---

It is important to have a clean separation of concerns, both in your code, as in your machine's setup. If you are looking for information on how to get a ZFS FreeBSD setup on one hard-drive with multiple partitions, this is what you want.

This guide assumes you want to install FreeBSD and manually create the ZFS pool, to change default settings, or to not use the entire disk.

If you just want to setup ZFS on the entire disk, use the 'ZFS' option in the bsdinstall partitioning menu.

1. Boot FreeBSD install DVD or USB Memstick
2. Select Install, and answer questions such as keyboard layout and hostname
3. When prompted to partition the disk, choose the `shell` option. In this mode you are expected to create a root file system mounted under `/mnt` that the installer will install the operating system into.

Now to the installation steps, start by determining which disks you wish to use:

```sh
camcontrol devlist
```

That should return something like `TOSHIBA HDWN180 GX2M at scbus7 target 0 lun 0 (ada0,pass1)`

SATA disks usually start with ada, followed by a number. SAS and USB disks start with da, followed by a number. The rest of this page assumes the disk is `ada0`, replace it with your disk devices name in all commands below.

Proceed to create a fresh partition table, erasing ALL data on the disk!:

```sh
# The first command is not needed if you are working on an empty new disk
gpart destroy ada0
gpart create -s gpt ada0
```

Then create the boot code partition:
for UEFI Boot:

```sh
gpart add -a 4k -s 800K -t efi ada0
gpart bootcode -p /boot/boot1.efifat -i 1 ada0
```

for Legacy Boot:

```sh
gpart add -a 4k -s 512K -t freebsd-boot ada0
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada0
```

We will now be creating the needed partitions to setup the system as we wish. I will be working with a 20GB hard drive, which I want to split in 3 partitions. 2GB for Swap, 10 GB for the FreeBSD installation and the remaining (8GB) for the Jails storage, in a secondary pool which will later be created.

Note that while a ZFS Swap Volume can be used instead of the freebsd-swap partition, it is not recommended, and crash dumps can't be saved to the ZFS Swap Volume.

Some of the `gpart` options explained:
* `-a {number}`: controls alignment.
* `-s {size}`: sets the size of the partition, if not set, it will default to the remaining space on the disk.
* `-l {name}`: sets the label for the partition

```sh
gpart add -a 1m -s 4G -t freebsd-swap -l swap0 ada0
gpart add -a 1m -s 10G -t freebsd-zfs -l disk0 ada0
gpart add -a 1m -t freebsd-zfs -l disk1 ada0
```

Find the device for the freebsd-zfs partition created above with `gpart show -p ada0`
   
```sh
=>      40  41942960  ada0  GPT  (20G)
		40      1024    1  freebsd-boot  (512K)
		1064       984       - free -  (492K)
		2048   4194304    2  freebsd-swap  (2.0G)
		4196352  20971520    3  freebsd-zfs  (10G)
		25167872  16773120    4  freebsd-zfs  (8.0G)
		41940992      2008       - free -  (1.0M)
```

Our FreeBSD installation will be on `ada0p3`.

For proceeding with the install we will mount a `tmpfs` filesystem to `/mnt` with:

```sh
mount -t tmpfs tmpfs /mnt
```

We will assume zroot as the name of the ystem's `zpool`, it could be anything (i.e. tank, data, mypool), and to create it we do:

```sh
zpool create -o altroot=/mnt zroot ada0p3
```

We will use the lz4 compression algorithm, since it is fast enough to be used by default, even for uncompressible data. It can be enabled with:

```sh
zfs set compress=on zroot         
```

We will then create a Boot Environment hierarchy with:

```sh
zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/default
mount -t zfs zroot/ROOT/default /mnt
```

and then proceed to create the rest of filesystems required for FreeBSD to be installed to this ZFS Pool:

```sh
zfs create -o mountpoint=/tmp -o exec=on -o setuid=off zroot/tmp
zfs create -o canmount=off -o mountpoint=/usr zroot/usr
zfs create zroot/usr/home
zfs create -o exec=off -o setuid=off zroot/usr/src
zfs create zroot/usr/obj
zfs create -o mountpoint=/usr/ports -o setuid=off zroot/usr/ports
zfs create -o exec=off -o setuid=off zroot/usr/ports/distfiles
zfs create -o exec=off -o setuid=off zroot/usr/ports/packages
zfs create -o canmount=off -o mountpoint=/var zroot/var
zfs create -o exec=off -o setuid=off zroot/var/audit
zfs create -o exec=off -o setuid=off zroot/var/crash
zfs create -o exec=off -o setuid=off zroot/var/log
zfs create -o atime=on -o exec=off -o setuid=off zroot/var/mail
zfs create -o exec=on -o setuid=off zroot/var/tmp             
```

then we will set a symlink to home, and set the correct permissions with:

```sh
ln -s /usr/home /mnt/home
chmod 1777 /mnt/var/tmp
chmod 1777 /mnt/tmp
```

We can then finalize the zpool creation, by setting the boot environment configuration:

```sh
zpool set bootfs=zroot/ROOT/default zroot
```

To finish the installation, we will create a new entry in the `fstab` of the installer, which later will be stored in our system. Now is your opportunity to attach other devices if needed as well, but for this tutorial, simply adding the swap is enough:

```sh
cat << EOF > /tmp/bsdinstall_etc/fstab
# Device Mountpoint FStype Options Dump Pass#
/dev/gpt/swap0 none swap sw 0 0
EOF     
```


We can then `exit` the shell Partitioning mode, at which point bsdinstall will continue and complete the installation.

In the post installation step, when asked if you would like to drop into the installed system, choose yes, and execute the following:

```sh
sysrc zfs_enable="YES"
echo zfs_load="YES" >> /boot/loader.conf    
```

You are now ready to reboot into your newly installed FreeBSD system. Simply type `exit` at the current prompt, and eject the installation media, when the machine restarts.

## Welcome to the magical world of FreeBSD!