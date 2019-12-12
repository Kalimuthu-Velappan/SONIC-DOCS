

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

![](http://10.59.132.240:9009/projects/csg_sonic/documentation/graphics/templates/Sysmonitor.png)
  
### 1.1.1 Functional Requirements

The monitoring framework should monitor both hardware and software resouces in the  Sonic system

 - The hardware resource monitoring should include CPU, Physical Memory
   and Disk usage. 
 - The software resource monitoring should include     Process and
   Services in the system
 - It should monitor and report the usage through Syslog Message
 - Add new resource monitoring service  named 'sysmonitor.service'  to systemd service list. 
 - All the alert message should be sent to syslog 

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



# 3 Tech-Support 
All the resource statististics and usage alert are forwarded to syslog.  The syslog is automatically monitored by the logrotate. During the techsupport data collection all the syslog is also collected as part of the tech-support data archive.


# 4 Unit Test

| SNO |  Testcase                                     |  Result |
|-----|-----------------------------------------------| ------- |
| 1   |  Simulate and verify the overall memory usage when it goes above threshold |         |
| 2   |  




<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcyNTM5MzA0OSwtOTEyMTAyNzcyLDExOD
UxNzkxNjQsODQ0NjY1MjM5LC02MjI0MDU4NDgsLTM0MjQ4NjYz
NywzNzQxNjUyOTEsMTEzMTQ3MTAxNiw5ODg0NTQ0ODAsLTc0Mj
U3MDM5MiwxMzMyNDg0OTA1LC0yMDYzNDM3OTEyLDEzOTIwNTMz
NzQsLTE4MjUwMjMzMTMsODgxMTU4NywxNzc3NTYyNzU5LDM1Nz
IxODYyLC0yMDY0ODAwMjM5LDExOTc1OTM4NDIsMjA5MTY0OTMy
XX0=
-->