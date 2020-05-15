

# Third Party Container Management

## High Level Design Document
**Rev 0.1**

## Table of Contents

* [List of Tables](#list-of-tables)
* [Revision](#revision)
* [About This Manual](#about-this-manual)
* [Scope](#scope)
* [Definition/Abbreviation](#definitionabbreviation)
* [Requirements Overview](#requirements-overview)
  * [Functional Requirements](#functional-requirements)
  * [Scalability Requirements](#scalability-requirements)
  * [Warmboot Requirements](#warmboot-requirements)
  * [Configuration and Management Requirements](#configuration-and-management-requirements)
* [Functional Description](#functional-description)
  * [Design](#design)
  * [Overview](#overview)
* [CLI](#cli)
* [Serviceability and DEBUG](#serviceability-and-debug)
* [Warm Boot Support](#warm-boot-support)
* [Unit Test](#unit-test)

# List of Tables

[Table 1: Abbreviations](#table-1-abbreviations)

# Revision

Rev   |   Date   |  Author   | Change Description
:---: | :-----:  | :------:  | :---------
1.0   | 05/07/20 | Kalimuthu | Initial version
2.0   |          |           |  


# About this Manual

This document describes the thrid party container management framework in SONiC network operating system. This feature helps the user to install and manages the third party container mangement.

# Scope

This document describes the high level design details of third party container management framework that consists of  installation, configuration and upgrade of the docker containers  It also describes the the tools  and commands used for container mangement.

# Definition/Abbreviation

### Table 1: Abbreviations

| **Term**     |  **Meaning**                  |
|:-------------|:------------------------------|
| TPCM | Third party container management      |
| ONIE | Open network intallation environment  |
|  | |


## 1 Requirements Overview

This document describes mechanisms to manage the third party container management(TPCM) in  SONiC environment. The TPCM allows the user to install the custom container into the SONIC  environemnt.  

SONiC is a network operation system based on the ubuntu distribution 14.0.4 and its compnents are tigtly intergrated with installation image which got created during the build time itself. Any addition componets needs to be installed, requires rebuilding of SONIC binary image.  However,  it allows the user to install the addition tools from the standstard debian repo.  This feature allows user to install the third party componets as contianer and that can  be loaded and integrated as part of the SONIC service. Any change in the SONiC image by reinstallation and upgrade shall not distrub the TPC componets.  The TPC shall be added and integrated from the new partition.

### TPCM Framework Requirements
- Third party container shall be installed indepentent of SONiC network operating system.
- It should co-exists with SONiC operating system
- Removal of SONiC OS from the device,  it should distrub the  TPC on the system.
- It should be installed on the separate partition
- TPC should be upgraded independent of SONIC
- TPC should be removed independent of SONIC
- One or more TPC can be installed on the system
- SONiC image upgrade is independent of TPC image upgrade.
- The TPCM should have facility to allow the TPC to be managed from the SONiC CLI.
- The TPC container image shall be created by the user and CLI commands shall be provided to install it on the SONiC system.
- TPC should be installed on seperate partitionn
- SONiC installer should recognize the TPC partion and leave untouched
- Service file should be imported from the TPC partition
- Any chnage in the container should be written back to the TPC partition
- 



### 1.2 Configuration and Management Requirements

- In order to integrate the TPC with SONiC, set of config and show commands shall be supported.

### Config commands
1. Config commands to add new TPC container into SONiC
2. Config commands to upgrade/remove  the existing container.

### Show commands
1. Show commands to display the list of installed TPC containers 
 
### 1.3 Scalability Requirements
- TPC partition size is restricted to 2G and consumed from the SONiC partition.  The amount of data stored ont the SONiC partion is restricted during the scaled configuration.

### 1.4 Warmboot Requirements
- The system should auto start all the TPC containers if the TPC partition is present.

## 2 Functional Description

## TPC Installation and Configuration

- All the TPC contianers are stored on the ,

2.  Ulimit configuration might prevent generation of core due to size configurations. We need to ensure this is not the case.

3.  Service restart functions - will not generate the core dump as it handle the graceful stop and start. This includes docker service restart as well.

# systemd-coredump

The [systemd-coredump](https://www.freedesktop.org/software/systemd/man/systemd-coredump.html) is a native systemd tool that is available in Debian o/s version 9 (stretch) and above. This tool provides an array of features to manage application core files. When it is installed, as part of base configuration it provides following functionality:

1.  Configures kernel to dump a core when application performs unexpected exit. The process ID, UID, GID, signal received, time of termination, command name of the terminated process are collected.
    
2.  Maintains a record of all crashes that have occured. The list is searchable using various key patterns. More about the [coredumpctl](https://www.freedesktop.org/software/systemd/man/coredumpctl.html#) tool can be found [here](https://www.freedesktop.org/software/systemd/man/coredumpctl.html#).
    
3.  If core files are deleted, a record is still maintained with some minimal information about the crash that has happened and indicating that the core file is missing. This helps maintain a historic record of all crash events even when core files have been deleted.
    
4.  Core files generated are stored in /var/lib/systemd/coredump with unique file name. E.g (core.myprogram.1000.30b28dabbdae41098a58bedd99bb1a95.28153.1560967987000000000000.lz4)
    
5.  Core files are compressed using LZ4 which provides optimal compression and decompression speed
    
6.  Maximum size of the core file stored. Configured as 2GB. This value is when corefile is in uncompressed format. On disk when it is stored in compressed format, it will occupy less space.
    
7.  File permissions of core files generated are set appropriately so that only root user can access them
    
8.  Ensures a maximum of 10% of total disk space to be used for storing core files.
    
9.  Ensures to keep at least 15% of total disk space as free.
    
10.  When too many core files are generated and take up more space than allocated disk space, oldest core files are automatically deleted to make way for the newly created ones.
  
# Configuration commands:

For SONiC switches following CLI commands will be provided to manage core files

#### show core [ config | info | list ]

>######  **\<config>** Show coredump configuration
>######  **\<info>** Show information about one or more coredumps
>######	 **\<list>** List available coredumps

Display list of current core files available and their information. This is a wrapper command for the coredumpctl utility provided by systemd-coredump package.

>###### config core <enable|disable>

Enable or disable coredump functionality. This configuration entry will be part of Config DB and thus can be stored as part of startup-configuration.

When disabled, this command will set ProcessSizeMax=0 in the /etc/systemd/coredump.conf file. When enabled this command will set ProcessSizeMax to be the same value as ExternalSizeMax.
 
Current SONiC code has some basic support for generation and compression of core files. Once systemd-coredump package is included in SONiC image, current functionality is removed.

>-   Install “systemd-coredump” as part of build_debian.sh
>-   Remove coredump directory entry in “/usr/lib/tmpfiles.d/systemd.conf”    
>-   “build_debian.sh” sets “kerne.core_pattern”. This needs to be removed
>-   “show techsupport” command collects all the core files present in /var/core. It needs to be modified to capture the files from “/var/lib/systemd/coredump”. Few edits to consider lz4 compressed core files.
   
Report of available core files
		
	root@sonic:/home/admin# coredumpctl list
	TIME PID UID GID SIG COREFILE EXE
	Tue 2019-06-18 15:14:12 UTC 5038 0 0 11 present /home/admin/hello
	Tue 2019-06-18 15:17:30 UTC 4458 0 0 6 present /usr/bin/vlanmgrd
	Tue 2019-06-18 15:20:44 UTC 9275 0 0 11 present /home/admin/hello
	Tue 2019-06-18 15:20:45 UTC 9281 0 0 11 present /home/admin/hello

Filter list to view core file of vlanmgrd

	root@sonic:/home/admin# coredumpctl list vlanmgrd
	TIME PID UID GID SIG COREFILE EXE
	Tue 2019-06-18 15:17:30 UTC 4458 0 0 6 present /usr/bin/vlanmgrd

View detailed information along with backtrace of a particular core file using PID as search string.

	root@sonic:/home/admin# coredumpctl dump 9275
	PID: 9275 (hello)
	UID: 0 (root)
	GID: 0 (root)

	Signal: 11 (SEGV)
	Timestamp: Tue 2019-06-18 15:20:44 UTC (37min ago)
	Command Line: ./hello
	Executable: /home/admin/hello
	Control Group: /system.slice/system-serial\x2dgetty.slice/serial-getty@ttyS2.service
	Unit: serial-getty@ttyS2.service
	Slice: system-serial\x2dgetty.slice
	Boot ID: f687508200b948a58c20222631c5ebbc
	Machine ID: 7bfb28a79813456090ec1df35a2073f5
	Hostname: sonic

	Storage: /var/lib/systemd/coredump/core.hello.0.f687508200b948a58c20222631c5ebbc.9275.1560871244000000000000.lz4
	Message: Process 9275 (hello) of user 0 dumped core.

	Stack trace of thread 9275:
	#0 0x00000000004005e7 main (hello)
	#1 0x00007f6ece5712e1 __libc_start_main (libc.so.6)
	#2 0x0000000000400519 _start (hello)
	Refusing to dump core to tty (use shell redirection or specify --output).

  
# Core File Rotation and Archive

When core file is generated for the same process multiple times, the framework should keep the generated core file in compressed form for the last N-1 number of core files. The number and the maximum size of the core files can be configured. The latest core file shall be kept in uncompressed format.

The archived core file should be generated with following format.  

	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.10618.1479890855000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.11686.1479886973000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.1748.1479887528000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.18600.1479888638000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.21734.1479888083000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.23065.1479890300000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.26069.1479889746000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.31104.1479891410000000000000.lz4  
	core.orchagent.0.8bc64adf67544e9e8b897cc5c1c9fde7.5952.1479889193000000000000.lz4
  
# Periodic Export of Core files

Through CLI interface, external storage server can be configured which includes server IP, path and access information like user credentials. The core manager shall scan for newly generated core files periodically and export the new core files to the configured location.

When an external server is configured, it may use either scp/sftp protocol to export core files to the remote server. This framework is  extended to create a “show tech support” information on detecting a core file, and export it.

Show tech-support export
1.  On detection of a new core file, “show tech-support” data is captured, and exported.
2.  In addition, this data is also periodically captured and exported.
    
To enable export feature:
### CLI commands

>##### sudo config export server username destdir protocol <server_ip> <username> <destination-directory> <scp|sftp>
> where: 
>>  server    => Name of the remote server to upload the tech-support data
>> username => remote server username
>> destdir      => destination server upload directory  path
>> protocol => transfer protocol either SCP/SFTP

>##### sudo config export <enable|disable>

This will export show tech-support data, on detection of a new core file. In addition, the data is also exported periodically.

### CLI commands
>##### sudo config export interval <interval in minutes>
>where  interval can be 
>> 0   =>  disable the periodic export of tech-support data
>> <30 - 1440>  => any value between 30 minutes to 24 hours. by default, 30 minutes is the export interval. 

1.  To disable or change the periodic export interval, use the following command:
   
### CLI commands
>##### sudo config export interval 0 (To disable)
>##### sudo config export interval 60 (To change it to one hour)
  


# UT report

https://drive.google.com/drive/u/0/folders/1jzVr93Kf9lY-eYmxjmUO86ugQzFLVp0J?ths=true


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg0NDY1NTI0M119
-->