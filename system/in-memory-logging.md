

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

This document describes the high level design of In-Memory Logging Enhancements feature. 

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**                 | **Meaning**                         |
|--------------------------|-------------------------------------|
| ODM                      | Original Design Manufacturer        |
| SYSLOG                   | System log

# 1 Feature Overview

SONiC is an open source network operating system based on Linux that integrates and runs various opensource application. SONiC application generates logs with differnt log level, capturing and storing logs into persistent storage from all the appplications is essential for debugging the system. In order maintain the persistance, every logs needs to be written into the log file in disk. The continus write of log into disk reduces the life time of disk and also performace of the logger. 

In order to improve the performance of the logger and increase the life of disk by dividing the logs into  debug and non-debug logs.  The debug Logs are called in-memory logging which will be stored in a non-persistance storage called ram memory or in-mormory  and periodically saved them into persistance storage.  All the non-debug logs are stored directly into persistance storage. The division of debug and non-debug log improves the life of disk as well as performance of the logger because the log generation rate of debug log is very high compare to non-debug logs. In-memory logging feature for application to log the debug and non-debug messages through unified syslog interface.

## 1.1 Requirements
  
### 1.1.1 Functional Requirements

 - It should provide unified interface to all SONiC application to log the information so that minimal code change is required from application. 
 - It should leverage the syslog as unified to interface for logging the application debug and non-debugs informations.
 - The division of debug and non-debug logs should be based on the Log Level. 
 - All the non-debug logs should be stored into presistance disk directly and stored logs should be rotated by lograotate periodically. 
 - All the debug logs should be stored into in-memory first and then saved them into disk and rotated by logrotate periodically.
 - All the in-memory logs should be saved into disk when cold/fast/warm command is issued.
 - In case of kernel crash, the in-memory logs should saved into disk as part of kdump collection. 
 - During the techsupport data collection, it should include both debug and non-debug logs. 
 - Klish/Click CLI is added to dump and filter the logs from both debug and non debugs logs. 
 - It should provide offline tools to show/filter the logs from both debug and non-debug logs.

### 1.1.2 Configuration and Management Requirements

- Klish/Click CLI is added to dump and filter the logs from both debug and non debugs logs. 

### 1.1.3 Scalability Requirements
- NA
### 1.1.4 Warm/fast/cold Boot Requirements
  - All the 
# 2 Design
## 2.1 Overview

SONiC uses syslog as logging infrastructure for logging the application log informations. In order to minimize the application code changes, the In-Memory infrastructure leverage the same existing syslog infrastructure for application sending the debug informations. The debug and non-debug information is classified through syslog interface Log lelvel as below. 

| **Log Level**         | **Log Value**     | **Classification**    |
|-----------------------|-------------------|-----------------------|
|  LOG_EMERG            |     0             | Non-debug             |
|  LOG_ALERT            |     1             | Non-debug             |
|  LOG_CRIT             |     2             | Non-debug             |
|  LOG_ERR              |     3             | Non-debug             |
|  LOG_WARNING          |     4             | Non-debug             |
|  LOG_NOTICE           |     5             | Non-debug             |
|  LOG_INFO             |     6             | debug                 |
|  LOG_DEBUG            |     7             | debug                 |

Syslog provides uniform interface to all the langages in the SONiC applications. The application uses these log level to differentiate the debug and non-debug information for logging. 
- It uses the standard syslog mechanism for generating the debug messages.
- Is uses standard system call syslog(LOG_DEBUG,….) for sending the logs messages.
- It provides the standard interface to all the application including C, C++, Python, Go, Shell script and Perl languages.
- It minimze the code changes on the application side because of unified interface.


## 2.2 Current Model
SONiC uses the rsyslog as centralized logger for receiving and storing the logs from various SONiC applications including logs from docker applications.  The Rsyslog receives the logs and process the logs if some action to be taken and store the logs into log files usually on the /var/log/ folder. Some of the SONiC application adds rules in the rsyslog to serapate the application specific logs into a seprate file, for example, all the audit related log messages are stored on the /var/log/audit.log. 

## 2.3 In-Memory Logging
The In-memory Logging levarage the existing rsyslog in 
![](images/in-memory-logging.png)



The framework supports both software and hardware resource types.  It includes the three major hardware resource types as CPU, Memory and Disk partitions and the software resource types as systemd core services.  

## 2.4 SYSLOG Levels 

The syslog alert message and related statistics are forwarded to syslog messaging system. The following three levels of syslog shall be supported 

	Level1  - INFO 
	Level2 -  WARN
	Level3 - CRITICAL
 
## 2.3 Threshold Limit

 The resouce limits are automitcally dervied from the system configurations.  By default, the threshold limit defined as three levels and each of the level maps to corresponding syslog levels.
  
	  Level1 - 70% - INFO
	  Level2 - 80% - WARN
	  Level3 - 90% - CRITICAL
 
 There are thresholds  that are specific to the particulate resource would be defined under the resouce type. 

## 2.4 Sampling Interval

   By default, the sampling interval is set to 3 minutes which indicates that every 3 minutes resource usage being monitored and checked against the threshold. The sampling interval is fixed by default and it gets adjusted based on the system resource configuration.
      
#### System Service Monitoring:
   
   In sonic, it is essential know the current state the system whether the system is ready to handle all the config commands or not.  If one of the core services are down, there should be way to identify the system state that it is not ready to take the config commands.  The sysmonitor framework monitors the system core services and port initialization state and generate the system ready message. If one of the system core service goes down,  it alerts that the 'system not ready' message because  the service is not ready to handle the  config commands.  The system ready state message is sent to both syslog as well as console session so that user would know the live state on the console.
  
  The system core services includes  'swss',  'bgp',  'teamd',  'pmon',  'syncd' and  'database'. Other service can be added to list when it expands its core list.
It also monitors the docker services. If any of the docker service goes up/down, it checks the state of system and report log accordingly.

- #### Example
		   - Dec 10 08:35:48.817550 System is ready
  
#### Memory Monitoring

Memory is a critical resource in the system. It is essential to monitor the memory usage  at the system wide as well as per process level and report the usage accross the system. This helps to indentify memory distribution accross system, the spike in the memory allocation and also if there is any memory leaks in the process. 

The framework  monitors the memory usage at system level as well as per process level. Threshold is defined for both per process level and system level.  

#### System Memory Usage: 

  Memory usage of overall system is being monitored with predifined threshold.  When the usage crosses the threshold limit, syslog message is being generated.  Syslog message is generated with following information.

 - Overall system memory usage information
 - Memory usage information of all runnings processes 
	 - Process name, 
	 - Process Id, 
	 - Used memory size.

##### System Memory Threshold Limit 
The resouce limits are automitcally dervied from the system configurations. 

 -  Overall memory usage  thresholds are drived from the system memory
	- INFO -  70%  of system memory
	 - WARN -  80%  of system memory
	 - CRITICAL -  90%  of system memory
	
Memory usage of resource is dumped on the console with the following format:

- Process name, Process ID, RSS( physical memory)
 
 #### Example :
	- Dec 11 13:06:19.397949 sonic INFO system#state: System memory usage is above 60%, Total: 15.6G, Free: 1.8G, Used: 2.8G, Buffers: 314.8M, Cached: 10.7G
	- Dec 11 13:06:19.477884 sonic INFO system#state: MEM :: Name: orchagent, Pid:6269, Rss:10.5M
	- Dec 11 13:06:19.477951 sonic INFO system#state: MEM :: Name: ospfd, Pid:11029, Rss:10.5M
	- Dec 11 13:06:19.478011 sonic INFO system#state: MEM :: Name: redis-server, Pid:1006, Rss:10.6M
	- Dec 11 13:06:19.478060 sonic INFO system#state: MEM :: Name: zebra, Pid:9625, Rss:11.3M
	   
#### Per Process Memory Usage:

  When  per process memory usage goes beyond threshold limit, should also generate the process memory usage info on the system log message. The message is generated with the following information.
  
 - Memory info of a process 
	 - Process name, 
	 - Process Id
	 - Rss - Used memory size.
	  
##### Per Process Memory Threshold Limit 
  
 - Per process usage Memory threshold is derived from the overall system memory
	 - INFO - 30%  of system memory
	 - WARN - 40%  of system memory
	 - CRITICAL - above 50% of system memory 

#### Example
	 - Dec 11 13:03:19.209233 sonic INFO system#state: Per process memory threshold exceeded for process rest_server[3781], threshold 3% of system memory 478.6M, current usage 538.2M
	 - Dec 11 13:03:19.242928 sonic INFO system#state: Per process memory threshold exceeded for process syncd[14083], threshold 3% of system memory 478.6M, current usage 515.3M


#### CPU Monitoring:

CPU usage of all the process in the system is being monitored.  When the usage crosses the threshold, syslog message is being generated.  Syslog message is generated with following information.

- Process info:
	- Process Name
	- Process Id
	- CPU usage Time
   

##### Per Process CPU threshold limits:

-  The CPU threshold limit is considered as duration of sampling interval in which the process high CPU condition detected.

	- INFO -  70%  of High CPU
	 - WARN -  80%  of  High CPU
	 - CRITICAL -  90%  of High CPU

#### Disk Partition Monitoring:

   In a long running network environment,  monitoring the disk parition usage is a crucial process. As a network operating system, disk partitions mainly used for storing the log files, core dumps,  debug infos, application files, config files and OS images. It is essential to keep track of the disk partition usage. When the usage crosses the threshold, syslog message is being generated.  Syslog message is generated with the following parition information.

 - Parition name 
 - Used space
 - Free space 
 - Total space

 - Overall partion usage thresholds are dervied as 
 
	- INFO -  70%  of total partition size
	 - WARN -  80%  of  total partition size 
	 - CRITICAL -  90%  of total partition size

#### Example:
Nov 27 07:16:50.878011 sonic INFO system#state: DISK usage of '/' is above 8%, Total: 31.4G, Free: 20.1G, Used: 9.7G
Nov 27 07:16:50.878849 sonic INFO system#state: DISK:: {'used': 38285312, 'free': 3890614272,  'mountpoint': '/var/log', 'total': 4160421888}

	
## 2.5 Resource DB

Monitoring framework has a self contained python database for maintaining the state of each resources.  The duration and number of entries for a resouce is automatically tuned based on the system resource configuration.


## 2.6 Tech-Support 

All the resource statististics and usage alerts are forwarded to syslog.  The syslog is automatically monitored by the logrotate framework. During the techsupport data collection, all the syslog data added as part of the tech-support data archive.


# 3 Unit Test

|SNO|  Testcase                                     |  Result |
|---|-----------------------------------------------| ------- |
| 1 | Simulate and verify the overall memory usage when it goes above threshold |   |
| 2 | Verify the per process memory usage and check the syslog alert || 
| 3 | Verify the high CPU condition of a process and check the syslog alert | |
| 4 | Verify the disk partition usage and check the syslog alert |  |
| 5 | Verify the system service status and check for system ready message | |
| 6 | Simulate the system services failure and check for the system not read message ||


