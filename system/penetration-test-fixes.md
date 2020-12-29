# SONiC Penetration Test Fixes

Information on fixes for some of the security vulnerabilities reported by the eBay Red team as part of their SONiC Network OS PenTest report

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
| Rev |   Date  |    Author    | Change Description        |
|:---:|:-----------:|:------------------:|-----------------------------------|
| 0.1 | 28/12/2020 |  Senthil    | Initial version          |

# About this Manual

This document provides information on fixes for some of the security vulnerabilities reported by the eBay Red team as part of their SONiC Network OS PenTest report.

# Scope

This document describes the high-level design on fixes for some of the security vulnerabilities reported by the eBay Red team as part of their SONiC Network OS PenTest report. This document focuses only the following three main categroies of issues reported. 

1. Docker running with privilage access.
2. Sudo entry with wildcard expression which allows privileage escalation.
3. Redis database accesss without authentication.

Other issues that are reported as part of penetration tests are not covered in this document.

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term**              | **Meaning**                         |
|-----------------------|-------------------------------------|
| ODM                   | Original Design Manufacturer        |
| ACL                   | Access Control List                 |
| ENV                   | Environment variable                |

# 1 Feature Overview

This document provides information on fixes for some of the security vulnerabilities reported by the eBay Red team as part of their SONiC Network OS PenTest report. The PenTest report provides detailed information about each vulnerability and possible solutions to counter the identified issues. This document can be used as a response from the Enterprise SONiC team to overcome the identified vulnerabilities. These changes are being proposed for the SONiC 3.2.0 release.


## 1.1 Requirements

### 1.1.1 Functional Requirements

Address the three major vulnerabilities reported in PenTest by ebay.

- It should allow only the authenticated application or user to access the redis database.
- Entries in the sudo file should have a explicit command specification that requires prvilege access and all the command that uses wildcard(*) should be replaced with specific instance of required command.
- All the container privilege access should be removed and should run isolated from the host environment, Any container requires specicial access from host should use sys capability feature instead of privilege access. 

### 1.1.2 Configuration and Management Requirements
- NA 

### 1.1.3 Scalability Requirements
- NA

### 1.1.4 Warm/fast/cold Boot Requirements
- NA
 
# 2 Design

### 2.1 Sudo entry with wildcard expression leading to privilege escalation.

##### 1. sudo Entry "/usr/bin/docker exec * ps aux" leading to privilege escalation

- The intention for the wildcard character ‘*’ specified in the sudo entry is to allow the “ps aux” command to be executed in all docker containers. However, the wildcard character ‘*’ can be used by a hacker to inject a crafted command to get access to the shell inside the container.

##### Command to check the vulnerability

- sudo docker exec -it mgmt-framework sh -c "/bin/bash;# ps aux"

##### Solution 

- Removed sudo entry of "docker exec * ps aux" in the sudoers file and replaced it with specific "docker exec {{container-name}} ps aux" commands. 

##### 2. sudo Entry "/usr/bin/vtysh -c show *" can be used to do a configuration write by a non sudoer

- The vtysh command allows a user to execute multiple commands using chained -c command arguments. The wildcard character “*” used in the sudo entry can be used by a hacker to substitute it with configuration commands while using a show command in the beginning to match the sudo entry. The vtysh is not validating user input from a security standpoint and it is expected that the sudo entry take care of it which is being bypassed due to the use of ‘*’.

##### Command to check the vulnerability

- sudo vtysh -c 'show version' -c 'configure terminal' -c 'interface Ethernet0' -c 'exit' -c 'exit' -E


##### Solution 
- vtysh script has been modified to validate user input and allow only show commands. The command exits with an error if any other vty command is used.

##### 3. sudo entry "/bin/cat /var/log/syslog" arbitrary file read*

- A non-sudo user is able to read the contents of /etc/shadow even though the file permissions of /etc/shadow file do not allow the unprivileged to view this file.

##### Command to check the vulnerability
- sudo /bin/cat /var/log/syslog.123 /etc/shadow

##### Solution 
- The sudo entry has been modified to specifically allow cat of only syslog and syslog.1

## 2.2 Local state/config Redis database accessible without authentication
- The local Redis database acts as a centralized location for storing device state and configuration. The Redis server runs inside the database container, and the user is required to execute redis-cli inside the database container to access the Redis database, which requires admin privilege. However, the Redis database also listens locally on TCP6379 without authentication, allowing any user to read/write to the Redis database.

##### Vulnerability check
- The Redis server listens on TCP6379 on localhost. The standard redis-cli client binary is uploaded to host OS for accessing Redis database via TCP6379 without authentication.

##### Solution 
- Enabled redis db authentication by default to prevent read/write access from local unprivileged users.
- Random passwords generated on every system boot up to be stored in a file shared across linux host and all containers with appropriate permissions set.
- All Clients(cpp,python,go variants) to pass in the password parameter(read from the file) while accessing the redis db.
- redis-cli client binary internally parses the password from an env variable (REDISCLI_AUTH) set.
- Non-sudo users direct read/write access to redis db is prevented but the basic fixed show commands which depend on the redis db are added to sudoers file.

## 2.3 Docker Containers Running in Privileged Mode

- Each service is running inside their own docker container in privileged mode. The privileged mode container doesn’t offer security benefit of isolating but have full read/write access to host OS resources, such as file system, allowing container escape or elevate privilege on the host OS. The docker socket on the host OS is also exposed inside mgmt-framework container, giving the container full control over all other docker containers and host OS.

##### Check the vulnerability
- for c in $(docker ps --format '{{.Names}}'); do printf "$c:"; docker inspect $c --format {{.HostConfig.Privileged}}'; done

##### Solution 

- All the container privilege access should be removed and should run isolated from the host environment, Any container requires specicial access from host should use sys capability feature instead of privilege access. 


# 3 Unit Test

|SNO|  Testcase                                                        | Result  |
|---|------------------------------------------------------------------| ------- |
| 1 | Verify the dockeer exec cmd from non sudo user                   |         |
| 1 | Verify the the vtysh show command from non sudo user             |         |
| 1 | Verify the syslog file asscess from non sudo user                |         |
| 2 | Verify the database access with the authentication               |         | 
| 2 | Verify the database access without the authentication            |         | 
| 3 | Verify the contianer isolation with capability support           |         |



