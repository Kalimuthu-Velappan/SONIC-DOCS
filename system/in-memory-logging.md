

# In-Memory Logging

System logging Enhancements.

# High Level Design Document

#### Rev 0.1

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)
  * [Feature Overview](#FeatureOverview)
  * [Requirements](#Requirements)
  * [Design](#Design)
  * [Unit Test](#UnitTest)


# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev |     Date    |       Author       | Change Description                |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 04/12/2020  |   Kalimuthu        | Initial version                   |

# About this Manual

This document provides general information about the In-Memory Logging feature implementation in SONiC.

# Scope

This document describes the high level design of In-Memory Logging Enhancement feature. 

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**                 | **Meaning**                         |
|--------------------------|-------------------------------------|
| ODM                      | Original Design Manufacturer        |
| SYSLOG                   | System log                          |
| KDUMP                    | Kernel Dump                         |

# 1 Feature Overview

SONiC is an open source network operating system based on Linux that integrates and runs various opensource applications. Each SONiC application generates log with differnt log level for every event happens on the system. Capturing and storing the logs from all the applicaiton into persistent storage is essential for debugging the system events. In order maintain the persistance, every logs needs to be written into the log file in the disk. The continus write of log into disk reduces the life time of disk and also affect the performace of the logger. 

In order to improve the performance of the logger and increase the life of disk is logs are divied into debug and non-debug logs.  The debug Logs are called in-memory logging which will be stored in a non-persistance storage called ram memory or in-mormory and periodically saved them into persistance storage.  All the non-debug logs are stored directly into persistance storage. The division of debug and non-debug log improves the life of disk as well as performance of the logger because the log generation rate of debug log is very high compare to non-debug logs. In-memory logging feature allows the application to log the debug and non-debug messages through unified syslog interface.

## 1.1 Requirements
  
### 1.1.1 Functional Requirements

 - It should provide a unified interface to all SONiC application to log the information so that minimal code change is required from the application. 
 - It should leverage the syslog as a unified interface for the application to log both debug and non-debugs informations.
 - The separation of debug and non-debug logs should be based on the Log Level. 
 - All the non-debug logs should be stored into presistance disk directly and stored logs should be rotated by lograotate periodically. 
 - All the debug logs should be stored into in-memory first and then saved into disk and rotated by logrotate periodically.
 - All the in-memory logs should be saved into disk when cold/fast/warm command is issued.
 - In case of kernel crash, the in-memory logs should be saved into disk as part of kdump collection. 
 - During the techsupport data collection, it should include both debug and non-debug logs. 
 - Klish/Click CLI is added to dump and filter the logs from both debug and non debugs logs. 
 - It should provide offline tools to show/filter the logs from both debug and non-debug logs.

### 1.1.2 Configuration and Management Requirements
- Klish/Click CLI is added to dump and filter the logs from both debug and non debugs logs. 

### 1.1.3 Scalability Requirements
- NA

### 1.1.4 Warm/fast/cold Boot Requirements
- All the In-Memory logs should be saved into the disk before the reboot.
  
# 2 Design
## 2.1 Overview

SONiC uses syslog as a logging infrastructure for application to log information. In order to minimize the application code changes, the In-Memory infrastructure leverage the same existing syslog infrastructure for application to log the debug information. The debug and non-debug information are classified through syslog interface Log lelvel as below. 

| **Log Level**         | **Log Value**     | **Classification**    |
|-----------------------|-------------------|-----------------------|
|  LOG_EMERG            |     0             | Non-debug             |
|  LOG_ALERT            |     1             | Non-debug             |
|  LOG_CRIT             |     2             | Non-debug             |
|  LOG_ERR              |     3             | Non-debug             |
|  LOG_WARNING          |     4             | Non-debug             |
|  LOG_NOTICE           |     5             | Non-debug             |
|  LOG_INFO             |     6             | Debug                 |
|  LOG_DEBUG            |     7             | Debug                 |

Syslog provides uniform interface to all the langages in the SONiC applications. The application uses these log level to differentiate the debug and non-debug information for logging. 
- It uses the standard syslog mechanism for generating the debug messages.
- Is uses standard system call syslog(LOG_DEBUG,….) for sending the logs messages.
- It provides the standard interface to all the application including C, C++, Python, Go, Shell script and Perl languages.
- It minimze the code changes on the application side because of unified interface.

## 2.2 Current Model
SONiC uses the rsyslog as centralized logger for receiving and storing the logs from various SONiC applications including logs from docker applications.  The Rsyslog receives the logs and process the logs if some action to be taken and then store the logs into log files into disk usually on the /var/log/ folder. Some of the SONiC application adds rules in the rsyslog to serapate the application specific logs into a seprate file, for example, all the audit related log messages are stored on the separate log /var/log/audit.log. 

## 2.3 In-Memory Logging
The In-memory Logging levarage the existing rsyslog infrastructure to process and store the logs into In-Memory. It doesn't require much change from application side as it uses same syslog API for generating the debug information by using log level as INFO or DEBUG. A simple rsyslog filter rule is added to syslog config for filtering and storing the debug logs.  

![](images/in-memory-logging.png)

### Kernel driver 
A block of memory is reserved from kernel physicall address space during bootup and mapped into userspace as ramblock device. The phyical memory reservation is done at the kernel level and the same memory can be mapped in to userspace for saving the in-memory contents into a file. The following kernel parameter enforces the fixed physical memory reservation during kernel bootup. 

    memmap=memory-size@address

A simple kernel driver will be loaded into kernel which emulates this memory area as RAM block device into userspace as '/dev/im-device'. 

### In-Memory
All the In-Memory logs are initially stored in the RAM memory and then it gets stored into persistant storage disk periodically. During the system startup, a portion of system RAM is reserved for in-memory storage and emulated as ram block device into a userspace. The reserved memory is formated as in ext4 filesystem and mounted into into as part of log file system as bellow.  All the file stored inside the 'ramfs' folder will be treated as in-memory files. The rsyslog uses this memory for storing the debug information into this file system. 

        # mkfs.etx4 /dev/ramdisk  
        # mount /dev/ramdisk /var/log/ramfs/

### In-memory with Kdump
During the kernel crash, all the data stored in the in-memory should be written into persistant disk. This is done using fixed kernel memory map. During primary kernel bootup, a fixed physical memory is reserved for inmemory storage.  When secondary kernel bootup, it uses hte same fixed physical memory area and maps into application space. Kdump collection utility is enhanced to dump the in-memory contents into persistant storage. 

### Rsyslog Policy
In order to separate out the debug information from syslog, the following Rsyslog rules being added into rsylog config. This will configure rsyslog to store all the debug logs into ramfs file system. 

        # Store all the DEBUG and INFO logs into ramfs file system.
        if  $syslogseverity >= 6 then {
            /var/log/ramfs/syslog-debug.log
            stop
        }

Log format is remain same as regular syslog format as bellow.
 
        # SONiC syslog default template
        $template SONiCFileFormat,"%timegenerated%.%timegenerated:::date-subseconds% %timegenerated:::date-year% %HOSTNAME% %syslogseverity-text:::uu
        ppercase% %syslogtag%%!msg1:::sp-if-no-1st-sp%%!msg1:::drop-last-lf%\n"
        $ActionFileDefaultTemplate SONiCFileFormat


## 2.4 Log Rotation Policy
The following log rotation policy is applied for the log stored in in-memory and also logs stored in persistant disk. The first policy enforce the logrotate to rotate the logs stored in in-memory and as part of the port rotate script, it forces the rotated logs into persist disk. The second policy instruct the logrotate to rotate the logs within peristent storage which is same as other syslog rotation.

        /var/log/ramfs/syslog-debug.log
        {
            size 1M
            rotate 2
            daily
            missingok
            notifempty
            postrotate
                cat /var/log/ramfs/syslog-debug.log.1  >> /var/log/syslog-debug.log
                rm -f /var/log/ramfs/syslog-debug.log.1
            endscript
        }
        /var/log/debug-syslog.log
        {
            size 1M
            rotate 100
            missingok
            notifempty
            compress
            delaycompress
            nosharedscripts
        }

When rsyslog is being restarted, all the in-memory contents should flushed to disk. The in-memory log rotation policy is added as part existing syslog rotation policy.

## 2.5 In-Memory Logging Policy
In order to simply the application interface and improve the logging performace, the following polices are enforce on the in-memory logging.
- Only In-Memory logging file is prefered as this improves the performance and simplifies the in-memory logging interface and its the implementation.
- Log Rotation Policy
    -   Cron job to rotate the in-memory log for every 2 minutes.
    -   Size of file is restricted to 1mb
- Size of of the In-Memory as bellow
    - 128mb on <=4GB
    - 256mb on 8GB
    - 512mb on >=16GB
- Disk write policy
    - Logs are written into disk by every 2 minutes.
    - Written into disk when user issued a reboot(cold/warm/fast) command.
    - During kernel panic, all the in-memory logs are written into disk as port kdump data collection.

## 2.6 Tech-Support 

The techsupport script is enhanced to support the in-memory log collection. During the tech-support collection, all the in-memory logs are included as part regular syslog collection.

- All the logs from in-memory.
- All the logs which are already stored in the persistant disk.


## 2.7 In-Memory Dump Tools
A utility is provided to dump and filter the logs from in-memory as well as syslog. The in-memory logs should be accessed in the following order to get the log timeing sequence.

1. /var/log/ramfs/debug-in-memory.log
2. /var/log/syslog-debug.log
3. /var/log/syslog-debug.log.1
4. /var/log/syslog-debug.log.2.gz, ...


## 2.8 KLISH/CLICK Commands
The following CLI is commands are supported to dump the logs from in-memory.

- Show all the logs from In-Memory contents.

        # show log in-memory 
    
- Show logs from both standard syslog and in-memory with the sequence of timestamp.     

        # show log all 

## 2.9 Cold/Warm/Fast Reboot
During the sytem reobot, All the In-Memory logs should be stored on the persistence disk. When user initiate the system reboot, the following sequence gets executed.
1. Append the contents of In-Memory into persistence storage file /var/log/in-memory-debug.log. 
2. Reset the In-Memory file pointer
3. Let the reboot script to continue. 
4. Just before reboot, repeate the step 1 and 2 again. 
    
## 3.0 Kernal Crash 
During the kernel crash, all the In-memory logs should be saved as part of kdump data collection. When kenrel crash happens the following action sequence gets executed.
1. During the kernel bootup, reserve the same physical memory used in the primary kernel.
2. Insert the ramdisk driver for emulating the reserved physical memory as ram block device and user space.
3. During the kdump serve startup, mount the ramblock device as ramfs into /var/log/ramfs foler.
4. As port of the kdump data collection, copy the contents of in-memory contents into /var/log/ folder.
5. reboot the system. 

# 3 Unit Test

|SNO|  Testcase                                     |  Result |
|---|-----------------------------------------------| ------- |
| 1 | Verify the In-memory logging memory reservation  |   |
| 2 | Verify the reserved memory block mounted as ramfs || 
| 3 | Verify the In-memory logging entry through rsyslog  | |
| 4 | Verify the In-memory contents are stored into disk for every 2 mintues |  |
| 5 | Verify the In-memory log files are rotated in the disk | |
| 6 | Verify the In-memory contents are saved during system reboot ||
| 7 | Verify the In-memory contents are save during kernel panic ||
| 8 | Verify the show commands for both Im-memory and regularsyslog ||
| 9 | Verify the techsupport that includes the In-Memory contents ||
| 10 | Verify the In-memory dump utils ||


