# MilkyWayCore
This organization (will) contain core part of MilkyWayLinux. 
# MilkyWayLinux
MilkyWayLinux is inspired by Android security model, though adds possibility to centralize control over servers which potentially reduces workload on administrators of large server groups.
While centralizing control MilkyWayLinux tries to keep its secure by introducing more signing algorithms.
## Abstract
Looking at current state of modern Linux distributions, one may observe that there is actual problem with keeping all dependencies compataible. 
And in the same time we can notice that most apps have their own versions of software and libraries required and in case of poor library or
software update the whole application may stop working or work incorrectly. The solution suggested by MilkyWayLinux is to ship all app dependencies
within app. This would not fit a desktop operating systems as they usually require a lot of applications to be usable. In the same time most Linux 
servers run very few applications which allows to ship dependencies within this applications.
Another noticable problem of Linux distributions is lack of mandatory access controls. As we have a very few applications on servers we may introduce
mandatory access controls on whole system without need to adapt to change in each library. 
One more problem is lack of centralized control of Linux servers. Usually update or/and installation of software on bare-metal and virtual Linux servers requires
manual interaction. The implementation of centralized controller may help in both maintaining servers and deploying applications.
# General architecture
The MilkyWayLinux consists of kernel, CoreOS and milkyway-agent. 
## Kernel
Kernel includes dm-verity and SELinux modules to provide security and ensure that only trusted applications can run on server.
## CoreOS
Core part of operating system contains basic utilities and library(shell, libc, etc.)
CoreOS is stored in read-only signed partition
### MilkyWay agent
The agent is functions as init starting services both from CoreOS and from user. Also it is capable of receiving remote commands from MilkyWay control server and execute them or pass to applications.
The MilkyWay agent is part of CoreOS
## Layer
Layer is an image of application which contains:
* Application itself
* Configuration for milkyway-agent on how to run this application
* MAC configuration overlay
* Dependencies in form of overlay filesystem
* Other meta information

The layer may have only libraries without an application
## Data
Data is read-write partition separated on three parts:
* Common data: data which can be safely shared across layers
* Secure data: isolated data folders which are accessible only for particular layer
* Group data: data shared across group of layers

## Secure config
Secure configuration stores signed configuration partition which is read only. Also it is possible to add read-write partition named secure dynamic config which would store signed configuration layer or files.

## Dynamic config
Partition for storing configurations which should be directly edited locally in OS runtime

# Secure boot
The secure boot ensures that system components have loaded properly and does not contain malicious code. The according diagram in draw.io file shows it.
If verification fails on any step the system is not booted.
## Vendor certificates
It is unlikely that users would be able to support whole reposiitory of layers, so partially they would be provided by some trusted vendors. 
## User certificates
In the same time user should be able to access own machines and install custom layers on them, so the user certificates are stored in bootloader and passed to kernel.
## Stage 1 bootloader
Stage 1 bootloader is signed by user certificates and has embedded user and vendor certificates. The only aim of stage 1 bootloader is to reduce times when user would sign images with own keys to minimum. 
Stage 1 bootloader loads and verifies stage 2 bootloader.
Stage 1 bootloader is verified with help of UEFI secure boot.
## Stage 2 bootloader
Stage 2 bootloader is provided by vendor and signed by vendor keys. It contains all necessary files to boot the system. Also it receives user certificates from stagge 1 bootloader. 
The stage 2 bootloader loads and verifies the kernel, then boots it passing user certificates to it.
## Kernel
Kernel verifies CoreOS partition(s) and if they are signed by vendor keys mounts them starting a milkyway-agent.
Also it provides a read-only partitions with user certificates.
## milkyway-agent
milkyway-agent mounts layer storage and secure config(which signed either by user keys or by vendor keys). Then it loads layers and verifies that they are signed by vendor or user keys.

# MAC
Mandatory Access Controls prevents the compormisation of whole system in case if one of apps is compromised. Generally following ideas should be followed during implementation of MAC for MilkyWayOS:
* Each layer have its own private read-write storage, no other module or even milkyway-agent should be able to access it
* The common data can be shared by layer or group of layers(Yes, layers should have groups implemented)
* Layer have execution permission for system utilities but utilities MUST inherit security rules for layer which called the utility
* Interaction with milkyway-agent should be limited
* milkyway-agent have own data
* Each layer may configure additional developer-shipped security rules for e.g. allowing usage of layer by other layers


