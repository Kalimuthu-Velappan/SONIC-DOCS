

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
# 3 Design
## 3.1 Overview

![](http://10.59.132.240:9009/projects/csg_sonic/documentation/graphics/templates/Sysmonitor.png)

Monitor the resource usage and generate the  message  when its usage crosses the threshold level.
One Syslog message is generated for each threshold level crossing. 

### Memory Monitoring :

#### System Memory Usage:
   Memory usage of overall system is being monitored.  When the usage crosses the threshold, syslog message is being generated,
   

## 3.2 DB Changes
Describe changes to existing DBs or any new DB being added.
### 3.2.1 CONFIG DB
### 3.2.2 APP DB
### 3.2.3 STATE DB
### 3.2.4 ASIC DB
### 3.2.5 COUNTER DB

## 3.3 Switch State Service Design
### 3.3.1 Orchestration Agent
### 3.3.2 Other Process
Describe changes to other process within SwSS if applicable.

## 3.4 SyncD
Describe changes to syncd if applicable.

## 3.5 SAI
Describe [new/existing] SAI APIs used by this feature.

## 3.6 User Interface
### 3.6.1 Data Models
Can be reference to YANG if applicable. Also cover gNMI here.

### 3.6.2 CLI
#### 3.6.2.1 Configuration Commands
#### 3.6.2.2 Show Commands
#### 3.6.2.3 Debug Commands
#### 3.6.2.4 IS-CLI Compliance
The following table maps SONIC CLI commands to corresponding IS-CLI commands. The compliance column identifies how the command comply to the IS-CLI syntax:

- **IS-CLI drop-in replace**  – meaning that it follows exactly the format of a pre-existing IS-CLI command.
- **IS-CLI-like**  – meaning that the exact format of the IS-CLI command could not be followed, but the command is similar to other commands for IS-CLI (e.g. IS-CLI may not offer the exact option, but the command can be positioned is a similar manner as others for the related feature).
- **SONIC** - meaning that no IS-CLI-like command could be found, so the command is derived specifically for SONIC.

|CLI Command|Compliance|IS-CLI Command (if applicable)| Link to the web site identifying the IS-CLI command (if applicable)|
|:---:|:-----------:|:------------------:|-----------------------------------|
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |

**Deviations from IS-CLI:** If there is a deviation from IS-CLI, Please state the reason(s).

### 3.6.3 REST API Support

# 4 Flow Diagrams
Provide flow diagrams for inter-container and intra-container interactions.

# 5 Error Handling
Provide details about incorporating error handling feature into the design and functionality of this feature.

# 6 Serviceability and Debug
Logging, counters, stats, trace considerations. Please make sure you have incorporated the debugging framework feature. e.g., ensure your code registers with the debugging framework and add your dump routines for any debug info you want to be collected.

# 7 Warm Boot Support
Describe expected behavior and any limitation.

# 8 Scalability
Describe key scaling factor and considerations.

# 9 Unit Test
List unit test cases added for this feature including warm boot.

# 10 Internal Design Information
Internal BRCM information to be removed before sharing with the community.



<!--stackedit_data:
eyJoaXN0b3J5IjpbMzc0MTY1MjkxLDExMzE0NzEwMTYsOTg4ND
U0NDgwLC03NDI1NzAzOTIsMTMzMjQ4NDkwNSwtMjA2MzQzNzkx
MiwxMzkyMDUzMzc0LC0xODI1MDIzMzEzLDg4MTE1ODcsMTc3Nz
U2Mjc1OSwzNTcyMTg2MiwtMjA2NDgwMDIzOSwxMTk3NTkzODQy
LDIwOTE2NDkzMl19
-->