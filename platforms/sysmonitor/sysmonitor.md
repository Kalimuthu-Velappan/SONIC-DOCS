

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
The hardware resource monitoring should include CPU, Physical Memory and Disk usage.
The software resource monitoring should include the process and services in the system 
It should monitor 
Requirement guidance: -
- Sonic system conFirst Pass: Fill out with top-level requirements that put the big picture in place
- Second Pass: Fill out with detailed, immutably numbered requirements from which test cases can be generated

See example file in template folder

### 1.1.2 Configuration and Management Requirements
Which UI's will be provided for the feature?
How will those UI's be implemented. Options might be:
- "Old" SONiC CLI (as of 201904 release)
- New SONiC Management Framework (Broadcom-provided in 201908 and beyond). CLI commands should be implemented in the SONiC IS-CLI within the KLISH framework. Any exceptions to this should be highlighted for discussion.
- Commands should be added in the appropriate manner, including:
--  Command modes
-- Authorization
-- Following IS-CLI syntax (incl "no" form)
-- Usage / Help string
-- Command line completion options
-- Error messages
-- Proper show output formats

- [Use/extend] existing non-SONiC UI ([e.g.] FRR)

### 1.1.3 Scalability Requirements
key scaling factors
### 1.1.4 Warm Boot Requirements

## 1.2 Design Overview
### 1.2.1 Basic Approach
Where is the code coming from? Existing Open-source project? Internal Broadcom codebase? New development? Somewhere else? Mixture?

### 1.2.2 Container
Which container(s) will the code go into? Will there be new container(s)?

### 1.2.3 SAI Overview
Do we expect to need a new SAI API specification? If so, provide rough semantics (information flow across the API)?
If using existing SAI APIs, then which area?
For non-hardware features this section will be N/A

# 2 Functionality
## 2.1 Target Deployment Use Cases
Wordy description, with diagrams if possible
## 2.2 Functional Description
Wordy description

# 3 Design
## 3.1 Overview
big picture view of the actors involved.
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
eyJoaXN0b3J5IjpbLTE4MjUwMjMzMTMsODgxMTU4NywxNzc3NT
YyNzU5LDM1NzIxODYyLC0yMDY0ODAwMjM5LDExOTc1OTM4NDIs
MjA5MTY0OTMyXX0=
-->