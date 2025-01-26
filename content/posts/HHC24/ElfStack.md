+++ 
draft = false
date = 2025-01-26T14:59:04+01:00
title = "HHC24 - ElfStack"
description = "Blue team operation by using ELK stack based SIEM system."
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "SIEM",
    "LogAnalytics",
    "BlueTeam"
]
externalLink = ""
series = [
    "HHC24",
]
+++

## Introduction

- At the beginning there is an information table informing players that the challenge can be solved by using SIEM solution based on ELK `(Elastic, Logdash, Kibana)` stack.
- Of course I used the easier way and jump for an option with SIEM solution. But, I for some challenges I was forced to parse them manually with good-old **Linux** tools.

## Easy mode - 🥈

- **Question** 1: How many unique values are there for the event_source field in all logs?
- ***Answer***: 5

- **Question** 2: Which event_source has the fewest number of events related to it?
- ***Answer***: AuthLog

- **Question** 3: Using the event_source from the previous question as a filter, what is the field name that contains the name of the system the log event originated from?
- ***Answer***: event.hostname

- **Question** 4: Which event_source has the second highest number of events related to it?
- AuthLog:269
- GreenCoat: 7496
- NetflowPmacct: 34679
- SnowGlowMailPxy: 1398
- WindowsEvent: 2299324
- ***Answer***: NetflowPmacct

- **Question** 5: Using the event_source from the previous question as a filter, what is the name of the field that defines the destination port of the Netflow logs?
- ***Answer***: event.port_dst

- **Question** 6: Which event_source is related to email traffic?
- ***Answer***: SnowGlowMailPxy

- **Question** 7: Looking at the event source from the last question, what is the name of the field that contains the actual email text?
- ***Answer***: event.Body

- **Question** 8: Using the 'GreenCoat' event_source, what is the only value in the hostname field?
- ***Answer***: SecureElfGwy

- **Question** 9: Using the 'GreenCoat' event_source, what is the name of the field that contains the site visited by a client in the network?
- ***Answer***: event.url

- **Question** 10: Using the 'GreenCoat' event_source, which unique URL and port (URL:port) did clients in the TinselStream network visit most?
- ***Answer***: pagead2.googlesyndication.com:443

- **Question** 11: Using the 'WindowsEvent' event_source, how many unique Channels is the SIEM receiving Windows event logs from?
- ***Answer***: 5

- **Question** 12: What is the name of the event.Channel (or Channel) with the second highest number of events?
- Microsoft-Windows-PowerShell/Operational: 11751
- Microsoft-Windows-Sysmon/Operational: 17713
- Security: 2268402
- Windows PowerShell: 50
- ***Answer***: Microsoft-Windows-Sysmon/Operational

- **Question** 13: Our environment is using Sysmon to track many different events on Windows systems. What is the Sysmon Event ID related to loading of a driver?
- ***Answer***: 6

- **Question** 14: What is the Windows event ID that is recorded when a new service is installed on a system?
- ***Answer***: 4697

- **Question** 15: Using the WindowsEvent event_source as your initial filter, how many user accounts were created?
- ***Answer***: 0

===

## Hard mode - Gold 🥇

- **Question** 1: What is the event.EventID number for Sysmon event logs relating to process creation?
- ***Answer***: 1

- **Question** 2: How many unique values are there for the 'event_source' field in all of the logs?
- ***Answer***: 5

- **Question** 3: What is the event_source name that contains the email logs?
- ***Answer***: SnowGlowMailPxy

- **Question** 4: The North Pole network was compromised recently through a sophisticated phishing attack sent to one of our elves. The attacker found a way to bypass the middleware that prevented phishing emails from getting to North Pole elves. As a result, one of the Received IPs will likely be different from what most email logs contain. Find the email log in question and submit the value in the event 'From:' field for this email log event.
- ***Answer***: kriskring1e@northpole.local

- **Question** 5: Our ElfSOC analysts need your help identifying the hostname of the domain computer that established a connection to the attacker after receiving the phishing email from the previous question. You can take a look at our GreenCoat proxy logs as an event source. Since it is a domain computer, we only need the hostname, not the fully qualified domain name (FQDN) of the system.
- ***Answer***:  SleighRider

- **Question** 6: What was the IP address of the system you found in the previous question?
- ***Answer***: 172.24.25.12

- **Question** 7: A process was launched when the user executed the program AFTER they downloaded it. What was that Process ID number (digits only please)?
- ***Answer***: 10014

- **Question** 8: Did the attacker's payload make an outbound network connection? Our ElfSOC analysts need your help identifying the destination TCP port of this connection.
- ***Answer***: 8443

```json
{
  "_index": ".ds-logs-generic-default-2024.12.30-000001",
  "_id": "8999f8e78e346f8b074f33cb9d0362b8ae1f8a15",
  "_version": 1,
  "_score": 0,
  "_source": {
    "hostname": "SleighRider.northpole.local",
    "@timestamp": "2024-09-15T14:37:51.000Z",
    "log": {
      "syslog": {
        "severity": {
          "code": 5,
          "name": "notice"
        },
        "facility": {
          "code": 1,
          "name": "user-level"
        }
      }
    },
    "data_stream": {
      "namespace": "default",
      "type": "logs",
      "dataset": "generic"
    },
    "@version": "1",
    "host": {
      "ip": "172.18.0.5"
    },
    "event_source": "WindowsEvent",
    "type": "syslog",
    "event": {
      "Task": 3,
      "Category": "Network connection detected (rule: NetworkConnect)",
      "Keywords": "-9223372036854775808",
      "User": "NORTHPOLE\\elf_user02",
      "EventType": "INFO",
      "SourcePort": 64543,
      "Image": "C:\\Users\\elf_user02\\Downloads\\howtosavexmas\\howtosavexmas.pdf.exe",
      "DestinationPort": 8443,
      "MoreDetails": "Network connection detected:",
      "ProcessGuid": "{face0b26-426e-660c-eb0f-000000000700}",
      "DestinationPortName": "-",
      "Initiated": true,
      "DestinationIp": "103.12.187.43",
      "Version": 5,
      "SeverityValue": 2,
      "UserID": "S-1-5-18",
      "DestinationIsIpv6": false,
      "ProcessID": 10014,
      "RuleName": "Usermode",
      "ProtocolText": "tcp",
      "OpcodeValue": 0,
      "SourceModuleType": "im_msvistalog",
      "Channel": "Microsoft-Windows-Sysmon/Operational",
      "Hostname": "SleighRider.northpole.local",
      "SourceName": "Microsoft-Windows-Sysmon",
      "Severity": "INFO",
      "SourceHostname": "SleighRider.northpole.local",
      "AccountType": "User",
      "DestinationHostname": "19.148.239.35.bc.googleusercontent.com",
      "SourcePortName": "-",
      "SourceModuleName": "inSysmon",
      "ProviderGuid": "{5770385F-C22A-43E0-BF4C-06F5698FFBD9}",
      "SourceIp": "172.24.25.12",
      "SourceIsIpv6": false,
      "OpcodeDisplayNameText": "Info",
      "ThreadID": 5844,
      "EventTime": "2024-09-15T14:37:51.000Z",
      "EventID": 3,
      "ProcessId": 8096,
      "Domain": "NT AUTHORITY",
      "RecordNumber": 729,
      "AccountName": "SYSTEM"
    },
    "tags": [
      "match"
    ]
  },
  "fields": {
    "event.DestinationIsIpv6": [
      false
    ],
    "event.ProcessID": [
      10014
    ],
    "event.SourcePortName": [
      "-"
    ],
    "event.ProviderGuid": [
      "{5770385F-C22A-43E0-BF4C-06F5698FFBD9}"
    ],
    "event_source": [
      "WindowsEvent"
    ],
    "event.DestinationHostname": [
      "19.148.239.35.bc.googleusercontent.com"
    ],
    "type": [
      "syslog"
    ],
    "event.RecordNumber": [
      729
    ],
    "event.Task": [
      3
    ],
    "hostname": [
      "SleighRider.northpole.local"
    ],
    "log.syslog.facility.name": [
      "user-level"
    ],
    "event.AccountName": [
      "SYSTEM"
    ],
    "event.SourcePort": [
      64543
    ],
    "event.Channel": [
      "Microsoft-Windows-Sysmon/Operational"
    ],
    "event.Domain": [
      "NT AUTHORITY"
    ],
    "event.SourceName": [
      "Microsoft-Windows-Sysmon"
    ],
    "log.syslog.severity.name": [
      "notice"
    ],
    "event.ThreadID": [
      5844
    ],
    "event.ProcessId": [
      8096
    ],
    "log.syslog.severity.code": [
      5
    ],
    "event.OpcodeValue": [
      0
    ],
    "data_stream.type": [
      "logs"
    ],
    "tags": [
      "match"
    ],
    "event.DestinationPort": [
      8443
    ],
    "event.DestinationIp": [
      "103.12.187.43"
    ],
    "event.Initiated": [
      true
    ],
    "event.RuleName": [
      "Usermode"
    ],
    "event.Image": [
      "C:\\Users\\elf_user02\\Downloads\\howtosavexmas\\howtosavexmas.pdf.exe"
    ],
    "event.Severity": [
      "INFO"
    ],
    "event.DestinationPortName": [
      "-"
    ],
    "event.Category": [
      "Network connection detected (rule: NetworkConnect)"
    ],
    "event.Keywords": [
      "-9223372036854775808"
    ],
    "event.EventID": [
      3
    ],
    "event.UserID": [
      "S-1-5-18"
    ],
    "event.ProtocolText": [
      "tcp"
    ],
    "event.SourceModuleType": [
      "im_msvistalog"
    ],
    "host.ip": [
      "172.18.0.5"
    ],
    "event.SourceIp": [
      "172.24.25.12"
    ],
    "event.User": [
      "NORTHPOLE\\elf_user02"
    ],
    "event.MoreDetails": [
      "Network connection detected:"
    ],
    "@version": [
      "1"
    ],
    "event.SourceIsIpv6": [
      false
    ],
    "event.AccountType": [
      "User"
    ],
    "log.syslog.severity.name.text": [
      "notice"
    ],
    "event.OpcodeDisplayNameText": [
      "Info"
    ],
    "event.EventTime": [
      "2024-09-15T14:37:51.000Z"
    ],
    "data_stream.namespace": [
      "default"
    ],
    "event.Version": [
      5
    ],
    "event.Hostname": [
      "SleighRider.northpole.local"
    ],
    "event.SeverityValue": [
      2
    ],
    "event.SourceModuleName": [
      "inSysmon"
    ],
    "@timestamp": [
      "2024-09-15T14:37:51.000Z"
    ],
    "event.SourceHostname": [
      "SleighRider.northpole.local"
    ],
    "log.syslog.facility.name.text": [
      "user-level"
    ],
    "data_stream.dataset": [
      "generic"
    ],
    "event.EventType": [
      "INFO"
    ],
    "event.ProcessGuid": [
      "{face0b26-426e-660c-eb0f-000000000700}"
    ],
    "log.syslog.facility.code": [
      1
    ]
  }
}
```

- **Question** 9: The attacker escalated their privileges to the SYSTEM account by creating an inter-process communication (IPC) channel. Submit the alpha-numeric name for the IPC channel used by the attacker.
- ***Answer***: ddpvccdbr

- **Question** 10: The attacker's process attempted to access a file. Submit the full and complete file path accessed by the attacker's process.
- ***Answer***: C:\Users\elf_user02\Desktop\kkringl315@10.12.25.24.pem

- **Question** 11: The attacker attempted to use a secure protocol to connect to a remote system. What is the hostname of the target server?
- ***Answer***: kringleSSleigH

- **Question** 12: The attacker created an account to establish their persistence on the Linux host. What is the name of the new account created by the attacker?
- ***Answer***:  ssdh

- **Question** 13: The attacker wanted to maintain persistence on the Linux host they gained access to and executed multiple binaries to achieve their goal. What was the full CLI syntax of the binary the attacker executed after they created the new user account?
- ***Answer***: /usr/sbin/usermod -a -G sudo ssdh

- **Question** 14: The attacker enumerated Active Directory using a well known tool to map our Active Directory domain over LDAP. Submit the full ISO8601 compliant timestamp when the first request of the data collection attack sequence was initially recorded against the domain controller.
- ***Answer***: 2024-09-16T11:10:12-04:00
- Note: <https://www.manageengine.com/products/active-directory-audit/kb/system-events/event-id-2889.html>

- **Question** 15: The attacker attempted to perform an ADCS ESC1 attack, but certificate services denied their certificate request. Submit the name of the software responsible for preventing this initial attack.
- ***Answer***: KringleGuard

- **Question** 16: On what behalf certificate was successfully issued?
- ***Answer***: nutcrakr

- **Question** 17: One of our file shares was accessed by the attacker using the elevated user account (from the AD-CS attack). Submit the folder name of the share they accessed.
- ***Answer***: WishLists

- **Question** 18: The naughty attacker continued to use their privileged account to execute a PowerShell script to gain domain administrative privileges. What is the password for the account the attacker used in their attack payload?
- ***Answer***: fR0s3nF1@k3_s
- Note: worked only with local grep of log file, ELK is not properly parsing these events (4103). 

```bash 
cat log_chunk_2.log | grep nutcrakr | grep "Domain Admins" -i
```

- **Question** 19: The attacker then used remote desktop to remotely access one of our domain computers. What is the full ISO8601 compliant UTC EventTime when they established this connection?
- ***Answer***: 2024-09-16T15:35:57.000Z
- Note: event.EventID : 4648

- **Question** 20: The attacker is trying to create their own naughty and nice list! What is the full file path they created using their remote desktop connection?
- ***Answer***: C:\WishLists\santadms_only\its_my_fakelst.txt

- **Question** 21: The Wombley faction has user accounts in our environment. How many unique Wombley faction users sent an email message within the domain?
- ***Answer***: 4

- **Question** 22: The Alabaster faction also has some user accounts in our environment. How many emails were sent by the Alabaster users to the Wombley faction users?
- ***Answer***: 22

- **Question** 23: Of all the reindeer, there are only nine. What's the full domain for the one whose nose does glow and shine? To help you narrow your search, search the events in the 'SnowGlowMailPxy' event source.
- ***Answer***: rud01ph.glow

- **Question** 24: With a fiery tail seen once in great years, what's the domain for the reindeer who flies without fears? To help you narrow your search, search the events in the 'SnowGlowMailPxy' event source.
- ***Answer***: c0m3t.halleys
