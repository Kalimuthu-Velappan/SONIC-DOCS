

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
- All the In-Memory logs should be saved into the disk before the reboot.
  
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
The In-memory Logging levarage the existing rsyslog infrastructure to process and store the logs into In-Memory. It doesn't require much change from application side as it uses same syslog API for generating the debug information by using log level as INFO or DEBUG. A simple rsyslog rule is added to syslog config for filtering the debug log 

![](images/in-memory-logging.png)

## 2.4 Log Rotation Policy

## 2.5 In-Memory Logging Policy


## 2.6 Tech-Support 

All the resource statististics and usage alerts are forwarded to syslog.  The syslog is automatically monitored by the logrotate framework. During the techsupport data collection, all the syslog data added as part of the tech-support data archive.

## 2.7 Offline Tools

## 2.8 KLISH/CLICK Commands
The following CLI is commands are supported 

## 2.9 Cold/Warm/Fast Reboot
All the logs should stored on hte persistant disk.

## 3.0 Kernal Crash 
All the In-memory logs should be saved as part of kdump data collection. 

# 3 Unit Test

|SNO|  Testcase                                     |  Result |
|---|-----------------------------------------------| ------- |
| 1 | Simulate and verify the overall memory usage when it goes above threshold |   |
| 2 | Verify the per process memory usage and check the syslog alert || 
| 3 | Verify the high CPU condition of a process and check the syslog alert | |
| 4 | Verify the disk partition usage and check the syslog alert |  |
| 5 | Verify the system service status and check for system ready message | |
| 6 | Simulate the system services failure and check for the system not read message ||


