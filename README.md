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
**WIP**
