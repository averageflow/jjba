---
title: "FreeBSD over Linux"
date: 2020-10-22T22:00:00Z
weight: 1
aliases: ["/freebsd-over-linux"]
tags: ["FreeBSD", "Linux"]
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
description: "There are important technical reasons to consider migrating from GNU/Linux to FreeBSD"
disableHLJS: false
---

While I believe some of the "political" problems that have been going on with GNU/Linux are important and reasons to consider migrating from GNU/Linux to BSD, there also exist technical reasons to consider.

### Clean separation

Because of the way FreeBSD has been designed, and how the different components have been put together, and how you deal with configuration and tuning, and all the tools that has been developed and improved over the cause of many years, working with FreeBSD is something special.

With most GNU/Linux distributions I have worked with since about 1998 you get a feeling of "mismatch".

As an example, Debian GNU/Linux has the Debian way of doing things. The Debian way is represented by the usage of a specific set of configuration management tools and patches that makes third party software conform to "the Debian way" of setting things up. And while this in some sense can unify how you do things on the Debian distribution, it is breaking with upstream configuration which makes it very annoying to deal with.

This is especially a problem when something isn't working right, or when the way things are described in the upstream documentation doesn't match the setup on Debian. Another problem with this approach is that some third party software, and even core elements of the distribution such as systemd, cannot be forcefully shaped into "the Debian way". The result is a system where some parts are running "the Debian way" while other parts are not.

Debian GNU/Linux has adopted systemd yet at the same time the default networking part is Debian specific. Sometimes you have to disable and remove Debian specific things to get systemd specific things to work.

On a distribution like Arch Linux this problem doesn't exist as there isn't such as thing as "the Arch way". The Arch Linux distribution want third party software to be as upstream has made them, so they do not change anything unless absolutely necessary. This is great because this means that the upstream documentation matches the software. A problem with this approach however is that because third party software does handle things differently, you can end up with a system where things aren't running unified.

Ubuntu is even worse. Because it is based upon Debian it runs with a lot of Debian tooling and setup, yet at the same time there is also the "Ubuntu way" in which things have been changed from Debian, and then there is added a layer on top of all that, a so-called user improved tooling layer, which sometimes makes Ubuntu break in incomprehensible ways.

On FreeBSD one of the things you'll notice right away is that you're dealing with a system that has been put together very well.

The kernel and base system is completely separated from the third party applications and base system configuration goes into /etc while all third party configuration goes into `/usr/local/etc`. Everything you can configure and everything you can tune or setup is very well documented in the man pages.

You have everything from the `rc` utility, which is the command script that controls the automatic boot process after being called by `init`, to the command scripts, to the `sysctl` kernel management tool, to all the different system configuration, and everything else put together very well and well documented.

I do not know exactly how to express this, but because FreeBSD is managed the way it is, as a complete operating system and project and not as a bunch of different projects that has been glued together in a distribution, everything is very well-thought-out, it is based upon many years of experience, and when things change, they change for the better for the entire community and with a lot of feedback from real use cases and problems in the industry.

One of the best ways to really understand this is to read Michael W. Lucas book [Absolute FreeBSD](https://nostarch.com/absfreebsd3). He manages greatly to not only explain and describe all the technical issues the book deals with, but he also captures the important historical context to why things are exactly the way they are.

### Documentation

Some people don't consider documentation to be a part of a technical reason to use something, but documentation belongs as an integrate part of the technology it describes. Bad documentation, outdated documentation and missing documentation should be considered a bug.

FreeBSD documentation is shipped with the system so you don't have to search the web. The man pages for the base system are of excellent quality and specifically written for FreeBSD. Most of what you need is right there with the system.

FreeBSD also has the [FreeBSD Handbook](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/), which covers the installation and day to day use of FreeBSD. The FreeBSD Handbook can be installed locally during installation. The handbook do occasionally have some outdated sections as the book is the result of ongoing work by many individuals, but it is generally updated and it is well written.

### Security

Normally it's not the operating system that get compromised, but a program running on the operating system. In some situations a compromised program can interact with the operating system in a way that also compromise the operating system. Securing your operating system means that you try to ensure that your computer's resources are used only by authorized people for authorized purposes.

FreeBSD has undergone thorough auditing in order to eliminate buffer overflows as well as myriad other security issues, furthermore FreeBSD provides many tools and options to help you secure your system against attackers.

It's impossible for me to provide an exhaustive list of options and available features in this article as the subject of security with FreeBSD could easily fill out a book. I highly recommend Michael W. Lucas book Absolute FreeBSD if you want a more in-depth study of some of the security features FreeBSD provides.

However, I can mention a couple of things.

### Security install-time options

* Hiding other UIDs processes
* Hiding other GIDs processes
* Hiding jailed processes
* Hide the message buffer
* Disable process debugging
* Randomize process IDs
* Disable syslogd networking
* Disable Sendmail
* Secure console
* Non executable stack and stack guard

Most of FreeBSD's other kernel-level security settings are available in the `security.bsd sysctl` tree and more get added every few months. You can run `sysctl -d security.bsd` to display the available options on your FreeBSD installation.

```sh
# sysctl -d security.bsd
security.bsd: BSD security policy
security.bsd.stack_guard_page: Specifies the number of guard pages for a stack that grows
security.bsd.unprivileged_get_quota: Unprivileged processes may retrieve quotas for other uids and gids
security.bsd.hardlink_check_gid: Unprivileged processes cannot create hard links to files owned by other groups
security.bsd.hardlink_check_uid: Unprivileged processes cannot create hard links to files owned by other users
security.bsd.unprivileged_idprio: Allow non-root users to set an idle priority
security.bsd.unprivileged_proc_debug: Unprivileged processes may use process debugging facilities
security.bsd.conservative_signals: Unprivileged processes prevented from sending certain signals to processes whose credentials have changed
security.bsd.see_jail_proc: Unprivileged processes may see subjects/objects with different jail ids
security.bsd.see_other_gids: Unprivileged processes may see subjects/objects with different real gid
security.bsd.see_other_uids: Unprivileged processes may see subjects/objects with different real uid
security.bsd.unprivileged_read_msgbuf: Unprivileged processes may read the kernel message buffer
security.bsd.unprivileged_mlock: Allow non-root users to call mlock(2)
security.bsd.suser_enabled: processes with uid 0 have privilege
security.bsd.map_at_zero: Permit processes to map an object at virtual address 0.
```

### Vulnerability Statistics

This is a list of vulnerability statistics for FreeBSD and Linux. The generally lower amount of security issues on FreeBSD doesn't necessarily mean that FreeBSD is more secure than Linux, even though I do believe it is, but it can also be because there is a lot more eyes on Linux.

```sh
+---------+---------+-------+
| Year    | FreeBSD | Linux |
+---------|---------|-------+
| 1999    | 18      | 19    |
| 2000    | 27      | 5     |
| 2001    | 36      | 22    |
| 2002    | 31      | 15    |
| 2003    | 14      | 19    |
| 2004    | 15      | 51    |
| 2005    | 17      | 133   |
| 2006    | 27      | 90    |
| 2007    | 9       | 62    |
| 2008    | 15      | 71    |
| 2009    | 11      | 102   |
| 2010    | 8       | 123   |
| 2011    | 10      | 83    |
| 2012    | 10      | 115   |
| 2013    | 13      | 189   |
| 2014    | 18      | 130   |
| 2015    | 6       | 86    |
| 2016    | 6       | 217   |
| 2017    | 23      | 454   |
| 2018    | 29      | 177   |
| 2019    | 18      | 170   |
|---------|---------|-------|
| Total   | 361     | 2333  |
+---------+---------+-------+
```

For further information about the specific vulnerabilities you can take a look at the CVE Details website for [FreeBSD](https://www.cvedetails.com/vendor/6/Freebsd.html) and [Linux](https://www.cvedetails.com/product/47/Linux-Linux-Kernel.html?vendor_id=33).

### Stability

FreeBSD has great engineering and release management practices. FreeBSD goes through multiple steps from idea inception to public release.

When someone gets an idea and develops something new it first gets peer technical reviews. Then it enters into the "current" branch for integrated testing and depending on complexity or potential impact the migration window for going into stable will be adjusted. Then it enters into the "stable" branch for wider user base testing. This is usually where all beta testing occurs also with a wider community engagement. Then it enters into release candidate testing which usually lasts for 3 rounds before it becomes a normal release.

This means that you can be pretty confident that things are going to continue to work, provided that you understand the release and upgrading notes.

Patches for software are released to fix any vulnerabilities and bugs.

This typically makes FreeBSD a very solid operating system.

### The Ports Collection

The FreeBSD Ports collection is an amazing feat of engineering. Both NetBSD's pkgsrc (package source) and OpenBSD's ports collection trace their origins to the FreeBSD ports system.

Normally when you install software on a Unix operating system you find and download the software. Then you unpack the software which is typically tarball compressed. Then you locate the documentation in INSTALL, README of some other text based document and read the instructions on how to install the software.

If the software is distributed in source format you need to compile it which normally involves editing a Makefile or running a configure script. Then if the compilation was successful you test and install the software. If the software had dependencies you would need to download and install those first.

The FreeBSD ports collection uses Makefiles to automate the process of compilation, installing and uninstalling software with the `make` command. The files that comprise a port contain all the necessary information to automatically download, extract, patch, compile, and install the application and very little (if any) user intervention is required after issuing a beginning command such as `make install` or `make install clean` in the ports directory of the desired application. If the port has needed dependencies on other applications or libraries, these are installed beforehand automatically.

Most ports are configured with a default set of options which have been deemed generally appropriate for most users. However, and this is one of the great things about the ports system, these configuration options can be changed before installation using the make config command. The command brings up a text-based interface that allows the user to select the desired options.

At the time of writing more than 38,487 ports are available in the collection.

In most cases the ports applications are made available for download as pre-compiled packages setup with the default options. These packages can be installed using the FreeBSD `pkg` - Binary Package Management application. Pre-compiled ports are called "packages".

The FreeBSD project has a package build farm in which all packages for all supported architectures and major releases are built. The build logs and known errors for all ports built into packages are available in a database and weekly builds logs are also available through mailing list archives.

### Rolling release packages

You have two different branches to choose from regarding packages. One is called "quarterly" and the other is called "latest".

Quarterly is the name for Ports branches cut from the HEAD branch in the revision system at the beginning of every yearly quarter in January, April, July, and October, and the name for the binary package sets that are produced from these branches.

The quarterly branch provides users with a more predictable and stable experience for port and package installation and upgrades. This is done essentially by only allowing non-feature updates. Quarterly branches aim to receive security fixes, but there may also be version updates, or backports of commits, bug fixes and ports compliance or framework changes.

If you choose the "latest" branch FreeBSD becomes a rolling release with regard to third party packages and much like Arch Linux it gets bleeding edge software.

### ZFS

The ZFS filesystem is a first class citizen on FreeBSD. This not only means that it is possible to have root installed on ZFS, and the installer supports this out of the box, but it also means that a lot of the base system tools have been tightly integrated or build with support for ZFS. Running ZFS on FreeBSD is different from running ZFS on Linux. On FreeBSD you get more tools that can be used to investigate performance issues or other relevant issues with ZFS.

Some of the features of ZFS are (taken from Wikipedia):

* Designed for long-term storage of data, and indefinitely scaled datastore sizes with zero data loss, and high configurability.
* Hierarchical checksumming of all data and metadata, ensuring that the entire storage system can be verified on use, and confirmed to be correctly stored, or remedied if corrupt. Checksums are stored with a block's parent block, rather than with the block itself. This contrasts with many file systems where checksums (if held) are stored with the data so that if the data is lost or corrupt, the checksum is also likely to be lost or incorrect.
* Can store a user-specified number of copies of data or metadata, or selected types of data, to improve the ability to recover from data corruption of important files and structures.
* Automatic rollback of recent changes to the file system and data, in some circumstances, in the event of an error or inconsistency.
* Automated and (usually) silent self-healing of data inconsistencies and write failure when detected, for all errors where the data is capable of reconstruction. Data can be reconstructed using all of the following: error detection and correction checksums stored in each block's parent block; multiple copies of data (including checksums) held on the disk; write intentions logged on the SLOG (ZIL) for writes that should have occurred but did not occur (after a power failure); parity data from RAID/RAIDZ disks and volumes; copies of data from mirrored disks and volumes.
* Native handling of standard RAID levels and additional ZFS RAID layouts ("RAIDZ"). The RAIDZ levels stripe data across only the disks required, for efficiency (many RAID systems stripe indiscriminately across all devices), and checksumming allows rebuilding of inconsistent or corrupted data to be minimised to those blocks with defects.
* Native handling of tiered storage and caching devices, which is usually a volume related task. Because ZFS also understands the file system, it can use file-related knowledge to inform, integrate and optimize its tiered storage handling which a separate device cannot.
* Native handling of snapshots and backup/replication which can be made efficient by integrating the volume and file handling. Relevant tools are provided at a low level and require external scripts and software for utilization.
* Native data compression and deduplication, although the latter is largely handled in RAM and is memory hungry.
* Efficient rebuilding of RAID arrays—a RAID controller often has to rebuild an entire disk, but ZFS can combine disk and file knowledge to limit any rebuilding to data which is actually missing or corrupt, greatly speeding up rebuilding.
* Unaffected by RAID hardware changes which affect many other systems. On many systems, if self-contained RAID hardware such as a RAID card fails, or the data is moved to another RAID system, the file system will lack information that was on the original RAID hardware, which is needed to manage data on the RAID array. This can lead to a total loss of data unless near-identical hardware can be acquired and used as a "stepping stone". Since ZFS manages RAID itself, a ZFS pool can be migrated to other hardware, or the operating system can be reinstalled, and the RAIDZ structures and data will be recognized and immediately accessible by ZFS again.
* Ability to identify data that would have been found in a cache but has been discarded recently instead, this allows ZFS to reassess its caching decisions in light of later use and facilitates very high cache-hit levels (ZFS cache hit rates are typically over 80%).
* Alternative caching strategies can be used for data that would otherwise cause delays in data handling. For example, synchronous writes which are capable of slowing down the storage system can be converted to asynchronous writes by being written to a fast separate caching device, known as the SLOG (sometimes called the ZIL – ZFS Intent Log).
* Highly tunable—many internal parameters can be configured for optimal functionality.
* Can be used for high availability clusters and computing, although not fully designed for this use.

Of course, you also get all of these features when you run with ZFS on Linux. However, there is a big difference as no single Linux distribution comes even close to the level of integration with ZFS that FreeBSD has.

### Boot environments

Due to the tight integration with ZFS FreeBSD also supports boot environments. With boot environments you can install multiple versions of the core operating systems and choose which one to boot into. As such a boot environment is a bootable clone or snapshot of the working system. With boot environments you can perform bulletproof upgrades or changes to the system, you don't have to worry about breaking anything because you can always roll back.

This also mean that you can update a FreeBSD system inside a new ZFS boot environment without touching the running system. You can also perform upgrades and test the results inside a FreeBSD Jail. You can even copy or move a ZFS Boot Environment into another machine.

FreeBSD has the [bectl](https://www.freebsd.org/cgi/man.cgi?query=bectl&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html) utility that makes it easy to manage boot environments.

### BSD init

FreeBSD uses the traditional BSD-style .

In the BSD-style init, there are no run-levels and `/etc/inittab` does not exist. Instead, startup is controlled by `rc` scripts.

The scripts found in `/etc/rc.d/` are for applications that are part of the base system, such as `cron`, `sshd`, `syslog`, etc. The scripts in `/usr/local/etc/rc.d/` are for user-installed third party applications such as NGINX or Postfix.

As previously mentioned, because FreeBSD is developed as a complete operating system, user-installed third party applications are not a part of the base system. Third party applications are installed using Packages or Ports. In order to keep them separate from the base system, user-installed applications are installed under `/usr/local/`. As such, user-installed binaries reside in `/usr/local/bin/`, whereas configuration files are located in `/usr/local/etc/`.

In the BSD init system services are enabled by adding an entry for the service in `/etc/rc.conf`. Defaults settings are located in `/etc/defaults/rc.conf` and these default settings are overridden by settings in `/etc/rc.conf`.

The following entry in /etc/rc.conf enable sshd:

```sh
sshd_enable="YES"
```

You can manually add the entry, or you can run:

```sh
service sshd enable
```

Which will automatically edit `/etc/rc.conf` and add the entry. You can manually start a service with:

```sh
service sshd start
```

If a service has not been enabled, but you still want to start it, it can be started from the command line using:

```sh
service sshd onestart
```

You can read more about init systems on [Wikipedia](https://en.wikipedia.org/wiki/Init).

### Jails

The FreeBSD Jails system is another amazing feat of engineering.

Jails were introduced in FreeBSD in version 4.0 on March 14, 2000.

The FreeBSD jail is an OS-level virtualisation that allows you to install a FreeBSD-derived system into several independent mini-systems called jails. The systems running in jails all share the same kernel and system resources and as a result there is very little overhead.

The need for the FreeBSD jails came from a small shared-environment hosting provider's (R&D Associates, Inc.'s owner, Derrick T. Woolworth) desire to establish a clean, clear-cut separation between their own services and those of their customers, mainly for security and ease of administration. Instead of adding a new layer of fine-grained configuration options, the solution (developed by Poul-Henning Kamp) was to compartmentalize the system, both its files and its resources, in such a way that only the right people are given access to the right compartments.

With jail, it is possible to create various virtual machines, each having its own set of utilities installed and its own configuration. This makes it a safe way to try out software. For example, it is possible to run different versions or try different configurations of a web server package in different jails. And since the jail is limited to a narrow scope, the effects of a misconfiguration or mistake (even if done by the in-jail superuser) does not jeopardize the rest of the system's integrity. Since nothing has actually been modified outside the jail, "changes" can be discarded by deleting the jail's copy of the directory tree.

The FreeBSD jail does not however achieve true virtualisation; it does not allow the virtual machines to run different kernel versions than that of the base system.

FreeBSD jails are an effective way to increase the security of a server because of the separation between the jailed environment and the rest of the system (the other jails and the base system).

If you want to better understand the difference between FreeBSD jails and Linux containers, read the blog post [Setting the Record Straight: containers vs. Zones vs. Jails vs. VMs](https://blog.jessfraz.com/post/containers-zones-jails-vms/).

FreeBSD has the `iocage` tool that has been designed to simplify jail management tasks. It abstracts out the management of ZFS-backed jails running VNET or shared IP networking.

### Bastille

[Bastille](https://bastillebsd.org/) is an open-source system for automating deployment and management of containerized applications on FreeBSD.

Bastille uses FreeBSD jails as a container platform and adds template automation to create a Docker-like collection of containerized software.

Templates take care of installing, configuring, enabling, and starting the software, providing an automated way of building containerized stacks.

### Capsicum

Capsicum is a sandbox framework developed at the University of Cambridge Computer Laboratory, supported by grants from Google, the FreeBSD Foundation, and DARPA. Capsicum extends the POSIX API, providing several new OS primitives to support object-capability security on UNIX-like operating systems:

* Capabilities - refined file descriptors with fine-grained rights
* Capability mode - process sandboxes that deny access to global namespaces
* Process descriptors - capability-centric process ID replacement
* Anonymous shared memory objects - an extension to the POSIX shared memory API to support anonymous swap objects associated with file descriptors (capabilities)
* `rtld-elf-cap` - modified ELF run-time linker to construct sandboxed applications
* `libcapsicum` - library to create and use capabilities and sandboxed components
* `libuserangel` - library allowing sandboxed applications or components to interact with user angels, such as Power Boxes.
* `chromium-capsicum` - a version of Google's Chromium web browser that uses capability mode and capabilities to provide effective sandboxing of high-risk web page rendering.

The FreeBSD implementation of Capsicum, developed by Robert Watson and Jonathan Anderson, ships out of the box in FreeBSD 10.0. Capsicum for FreeBSD is the reference implementation, and serves not only as a reference for Capsicum APIs and semantics, but also provides starting-point source code for ports to other platforms (e.g., Capsicum for Linux and Capsicum for DragonFlyBSD).

For more information about Capsicum on FreeBSD please see:

* [Capsicum for FreeBSD](https://www.cl.cam.ac.uk/research/security/capsicum/freebsd.html)
* [Capsicum: practical capabilities for UNIX (PDF)](https://papers.freebsd.org/2010/rwatson-capsicum.files/rwatson-capsicum-paper.pdf)
* [Capsicum and Casper: a fairy tale about solving security problems (YouTube)](https://www.youtube.com/watch?v=Kwce64dqY1w)
* [Capsicum technologies](https://wiki.freebsd.org/Capsicum)

### DTrace

[DTrace](https://www.freebsd.org/cgi/man.cgi?query=dtrace&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html) is a comprehensive dynamic tracing framework ported from Solaris. DTrace provides a powerful infrastructure that permits administrators, developers, and service personnel to concisely answer arbitrary questions about the behavior of the operating system and user programs.

### bhyve

bhyve is a native FreeBSD hypervisor that runs guest operating systems inside a virtual machine. Parameters such as the number of virtual CPUs, amount of guest memory, and I/O connectivity can be specified with command-line parameters.

bhyve supports the virtualisation of several guest operating systems, including FreeBSD 9+, OpenBSD, NetBSD, Linux, illumos, DragonFly, Windows Vista and later and Windows Server 2008 and later. Current development efforts aim at widening support for other operating systems for the x86-64 architecture.

The bhyve hypervisor became part of the base system with FreeBSD 10.0-RELEASE.

bhyve requires a processor that supports Intel Extended Page Tables (EPT) or AMD Rapid Virtualization Indexing (RVI) or Nested Page Tables (NPT). Hosting Linux guests or FreeBSD guests with more than one vCPU requires VMX unrestricted mode support (UG). Newer processors, specifically the Intel Core i3/i5/i7 and Intel Xeon E3/E5/E7, support these features. UG support was introduced with Intel's Westmere micro-architecture.

### Firewall (#firewall)

FreeBSD has three different firewalls built into the base system: PF, IPFW, and IPFILTER, also known as IPF.

Since FreeBSD 5.3, a ported version of OpenBSD's PF firewall has been included as an integrated part of the base system. PF is a complete, full-featured firewall that has optional support for ALTQ (Alternate Queuing), which provides Quality of Service (QoS). The filtering syntax of PF is similar to IPF, with some modifications to make it clearer. Network Address Translation (NAT) and Quality of Service (QoS) have been integrated into PF, QoS by importing the ALTQ queuing software and linking it with PF's configuration. Features such as pfsync and CARP for failover and redundancy, authpf for session authentication, and ftp-proxy to ease firewalling the difficult FTP protocol, have also extended PF. Also, PF supports SMP (Symmetric multiprocessing) & STO (Stateful Tracking Options).

One of the many innovative features is PF's logging. PF's logging is configurable per rule within the pf.conf and logs are provided from PF by a pseudo-network interface called pflog, which is the only way to lift data from kernel-level mode for user-level programs. Logs may be monitored using standard utilities such as tcpdump.

IPFW is a stateful firewall written for FreeBSD which supports both IPv4 and IPv6. It consists of several components: the kernel firewall filter rule processor and its integrated packet accounting facility, the logging facility, NAT, the dummynet traffic shaper, a forward facility, a bridge facility, and an ipstealth facility.

FreeBSD provides a sample ruleset in /etc/rc.firewall which defines several firewall types for common scenarios to assist novice users in generating an appropriate ruleset. IPFW provides a powerful syntax which advanced users can use to craft customized rulesets that meet the security requirements of a given environment.

IPF is a cross-platform open source firewall which has been ported to several operating systems, including FreeBSD, NetBSD, OpenBSD, and Solaris. IPF is a kernel-side firewall and NAT mechanism that can be controlled and monitored by userland programs. Firewall rules can be set or deleted using ipf, NAT rules can be set or deleted using ipnat, run-time statistics for the kernel parts of IPF can be printed using ipfstat, and ipmon can be used to log IPF actions to the system log files.

IPF was originally written using a rule processing logic of "the last matching rule wins" and only used stateless rules. Since then, IPF has been enhanced to include the quick and keep state options.

### Fine-tuning

FreeBSD has over five hundred system variables that can be read and set using the [sysctl](https://www.freebsd.org/cgi/man.cgi?query=sysctl&apropos=0&sektion=0&manpath=FreeBSD+12.1-RELEASE+and+Ports&arch=default&format=html) utility. These system variables can be used to make changes to a running FreeBSD system. This includes many advanced options of the TCP/IP stack and virtual memory system that can dramatically improve performance for an experienced system administrator.

### GEOM

FreeBSD GEOM is the main storage framework for the FreeBSD operating system. It provides a standardized way to access storage layers. GEOM is modular and allows for GEOM modules to connect to the framework. For example, the geom_mirror module provides RAID1 or mirroring functionality to the system. A number of modules are already available, and new ones are always in active development by various FreeBSD developers.

Because of GEOM's modular design, modules can be stacked together to form a chain of GEOM layers. For example, on top of the geom_mirror module an encryption module can be added, such as geom_eli to provide a mirrored and encrypted volume. Each module has both consumers and providers. A provider is the source of the GEOM module, often a physical hard drive but sometimes a virtualized disk such as a memory disk. The geom module in turn provides an output device. Other GEOM modules, called consumers, can use this provider to create a chain of modules connected to each other.

### Linux Binary Compatibility

FreeBSD provides binary compatibility with Linux. This allows users to install and run many Linux binaries on a FreeBSD system without having to first modify the binary. In some specific situations Linux binaries can even perform better on FreeBSD than they do on Linux.

Not all Linux-specific operating system features are supported under FreeBSD. For example, Linux binaries will not work on FreeBSD if they overly use i386 specific calls, such as enabling virtual 8086 mode.

### Security Event Auditing

FreeBSD includes support for security event auditing. Event auditing supports reliable, fine-grained, and configurable logging of a variety of security-relevant system events, including logins, configuration changes, and file and network access. These log records can be invaluable for live system monitoring, intrusion detection, and postmortem analysis. FreeBSD implements Sun's published Basic Security Module (BSM) Application Programming Interface (API) and file format, and is interoperable with the Solaris and Mac OS X audit implementations.

### Final notes

This article is by no means an exhaustive list of technical reasons to use FreeBSD over GNU/Linux. Many other reasons exist that I haven't addressed. However, these are some of the features that most stand out in my humble opinion.

Unless you have a very specific need to use GNU/Linux, such as specific support for hardware, you will usually experience a greater sense of overview, control and harmony when you run and manage FreeBSD.

Situations where you might experience problems with FreeBSD are situations where there is a lack of hardware support, or where specific third party applications are very Linux centric. The later is mostly related to desktop applications, not server applications.

For example, Mozilla develops Firefox with their primary focus on Linux, OSX and Windows. These are called Tier-1 platforms. FreeBSD, OpenBSD, NetBSD and Solaris are located at the bottom of the support list as Tier-3 platforms. Mozilla developers do not reliably have access to non-Tier-1 platforms or build environments. At any given time a Firefox built from mozilla-central for non-Tier-1 platforms may or may not work correctly or build at all. Tier-3 platforms have a maintainer or community which attempt to keep the platform working. These platforms are not supported by Mozilla's continuous integration process, and Mozilla does not routinely test on these platforms.

This means that maintainers on FreeBSD has to spend an extra amount of time to make sure that applications such as Firefox compile and work on FreeBSD. And often when problems arise the Mozilla developers doesn't pay as much attention to the issues as they do when problems arise on Linux, Windows or OSX. The same goes for other Linux centric third party applications.

This doesn't mean that these applications doesn't work on FreeBSD, it just means that occasionally you might experience a problem of some kind.

I am running FreeBSD on both servers and desktop workstations, where I recently have migrated systems running ZFS on both Debian GNU/Linux and Arch Linux to FreeBSD. I have experienced an increase not only in performance, but also in reliability because of the better integration that FreeBSD provides for ZFS.

It is a real shame that FreeBSD hasn't received the same amount of attention as GNU/Linux. In many many cases, especially on production servers and in commercial usage, a company can gain much by running FreeBSD rather than Linux and often the only reason why they do run Linux is because of habit and a lack of knowledge about FreeBSD.

The next time you need to deploy a new system I recommend that you investigate and test out FreeBSD too. It is well worth your time.