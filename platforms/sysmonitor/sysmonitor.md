

# System Resource Monitoring
System Resource Monitoring Enhancements.
# High Level Design Document
#### Rev 0.1

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)

# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev |     Date    |       Author       | Change Description                |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 04/12/2019  |   Kalimuthu        | Initial version                   |

# About this Manual
This document provides general information about the System Resource Monitoring Enhancements feature implementation in SONiC.
# Scope
This document describes the high level design of System Resource Monitoring Enhancements feature. 

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**                 | **Meaning**                         |
|--------------------------|-------------------------------------|
| XYZ                      | Term description                    |

# 1 Feature Overview


As SONiC runs on different ODM platform hardware configurations, it becomes necessary to have a monitoring framework that is native to SONiC to monitor the system resouces usage and software service state of the system and generate  the syslog alert when it reaches the certian threshold . However, it should be noted that the scope of this framework is limited to monitoring and generating the syslog alert, the corrective measures that can be done in the context of the SONiC OS. 



## 1.1 Requirements
  
### 1.1.1 Functional Requirements

 - The monitoring framework should monitor both hardware and software resouces in the  Sonic system
 - The hardware resource monitoring should include CPU, physical memory
   and disk usage. 
 - The software resource monitoring should include  process, docker and
   core systemd services in the system
 - It should monitor and report the resource usage  through system syslog message
 - The system monitoring framework should run as separate service and it should get lanched automatically during bootup.  By default  system monitoring service should get started by default. 
 - It should support three levels of threshold for each 

Resouce Monitoring
 - CPU Monitoring
	 - Generate the syslog alert with process stats when its usage crosses threshold limit.
	 - CPU threshold shall be  predefined as  80%, 90%, 100%.
 - Memory Monitoring
	 - Generate the syslog alert with proceess memory stats when its usages crosses the threshold limit.
	 - Memory threshold shall be predefined as 80%, 90%, 95%
	 - Per process memory monitoring
		 - Generate the syslog alert with proceess memory stats when its usages crosses the threshold limit.
		 - Memory threshold shall be predefined as 80%, 90%, 95%

 - Disk Monitoring
	 - Generate the syslog alert with root partition stats when its usages crosses the threshold limit.
	 - Disk threshold shall be predefined as 70%, 80%, 90%
 - Service Monitoring
	 - Monitor the System core services   which includes  'swss',  'bgp',  'teamd',  'pmon',  'syncd' and  'database'.
	 - Generate the system ready syslog message when the core services are up and port initailation is done.
	 - Generate system not ready message when only the service goes down. 
 
 -  Generate syslog message level as INFO, WARNING and CRITIAL for respective threshold levels.
 
### 1.1.2 Configuration and Management Requirements

 - Threshold levels are dervied from the system configurations.

### 1.1.3 Scalability Requirements
   NA
### 1.1.4 Warm Boot Requirements
  NA
# 2 Design
## 2.1 Overview

![](http://10.59.132.240:9009/projects/csg_sonic/documentation/graphics/templates/Sysmonitor.png)

Monitor the resource usage and generate the  message  when its usage crosses the threshold level.
One Syslog message is generated for each threshold level crossing. 

### Memory Monitoring :

#### System Service Monitoring:
   
   It monitor the system core services and port initializatin state and generate the system ready message. If one of the system core service goes down, it monitor and print the system not ready message. 

- #### Example
		   - Dec 10 08:35:48.817550 System is ready
   
#### System Memory Usage:
   
   Memory usage of overall system is being monitored.  When the usage crosses the threshold, syslog message is being generated.  Syslog message is generated with following information.
   

	 - Overall system memory usage information
	 - Memory usage information of all the runnings processes 
	 
 - #### Example :
	- Dec 11 13:06:19.397949 sonic INFO system#state: System memory usage is above 60%, Total: 15.6G, Free: 1.8G, Used: 2.8G, Buffers: 314.8M, Cached: 10.7G
	- Dec 11 13:06:19.477884 sonic INFO system#state: MEM :: Name: orchagent, Pid:6269, Rss:10.5M
	- Dec 11 13:06:19.477951 sonic INFO system#state: MEM :: Name: ospfd, Pid:11029, Rss:10.5M
	- Dec 11 13:06:19.478011 sonic INFO system#state: MEM :: Name: redis-server, Pid:1006, Rss:10.6M
	- Dec 11 13:06:19.478060 sonic INFO system#state: MEM :: Name: zebra, Pid:9625, Rss:11.3M
	   
#### System Memory Usage:
  Per process memory usage goes beyond threshold limit, should also generate the process memory usage info on the system log message

- #### Example
	 - Dec 11 13:03:19.209233 sonic INFO system#state: Per process memory threshold exceeded for process rest_server[3781], threshold 3% of system memory 478.6M, current usage 538.2M
	 - Dec 11 13:03:19.242928 sonic INFO system#state: Per process memory threshold exceeded for process syncd[14083], threshold 3% of system memory 478.6M, current usage 515.3M


## 2.2 Resource DB
Monitoring framework has self contained python database for maintaining the state of the resource.  The duration and number of entries for a resouce is automatically tuned based on the system resource configuration.

## 2.3 Resource Threshold Limit 
The resouce limits are automitcally dervied from the system configurations. 
Memory usage:

 -  Overall memory usage  thresholds are drived from the system memory
	- INFO -  70%  of system memory
	 - WARN -  80%  of system memory
	 - CRITICAL -  90%  of system memory
	
 - Per process usage Memory threshold is derived from 30% system memory
	 - INFO - 30%  of system memory
	 - WARN - 40%  of system memory
	 - CRITICAL - above 50% of system memory 

CPU threshold limits:
-  The CPU threshold limit is considered as duration of Sampling interval in which the process high CPU condition detected.
	- INFO -  70%  of High CPU
	 - WARN -  80%  of  High CPU
	 - CRITICAL -  90%  of High CPU

Disk Parition Usage:
 - Overall partion usage thresholds are dervied as 
	- INFO -  70%  of partition size
	 - WARN -  80%  of  partition size 
	 - CRITICAL -  90%  of partition size
	 
## 2.4 SYSLOG alert 
Whenever the resouce threshold limit is being reached,  alert message and related statistics are forwarded to syslog messaging system. The following three levels of syslog is being generated 

	Level1  - INFO 
	Level2 -  WARN
	Level3 - CRITICAL
 
 
## 2.5 Sampling Interval
   By the sampling interval is set as 3 minutes which indicates that every three minutes resource usage being monitored and checked against the threshold.

## 2.5 Tech-Support 
All the resource statististics and usage alert are forwarded to syslog.  The syslog is automatically monitored by the logrotate. During the techsupport data collection all the syslog is also collected as part of the tech-support data archive.


# 4 Unit Test

|SNO|  Testcase                                     |  Result |
|---|-----------------------------------------------| ------- |
| 1 | Simulate and verify the overall memory usage when it goes above threshold |   |
| 2 | Verify the per process memory usage and check for the sys log alert || 
| 3 | Verify the high CPU condition of a process and check for syslog alert | |
| 4 | Verify the disk partition usage and syslog alert |  |
| 5 | Verify the system service status and check for system ready message | |
| 6 | Simulate the system services failure and check for the system not read message ||




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY4NzU5MjIzMywtMTgzNzc3MTE0NCwxND
Y1MTIxMzg5LC00MzAyMzUyNDcsMjA4NjU2MDkzMSw0MDE4NjQ2
MDYsMTQ2MzgwMzU2OCwxNDExMjc3Mzc4LDEzNzk3MDMzNjYsLT
kxMjEwMjc3MiwxMTg1MTc5MTY0LDg0NDY2NTIzOSwtNjIyNDA1
ODQ4LC0zNDI0ODY2MzcsMzc0MTY1MjkxLDExMzE0NzEwMTYsOT
g4NDU0NDgwLC03NDI1NzAzOTIsMTMzMjQ4NDkwNSwtMjA2MzQz
NzkxMl19
-->