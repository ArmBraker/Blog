+++ 
draft = false
date = 2025-01-26T14:58:53+01:00
title = "HHC24 - SantaVision"
description = ""
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "Iot",
    "MQTT",
    "ReverseEngineering",
    "JFFS2",
    "ROT13"
]
externalLink = ""
series = [
    "HHC24",
]
+++

## Introduction

- There is a terminal which will spawn you a instance for this challenge.
- The challenge is divided into multiple phases: A,B,C,D (for silver).
- Once the challenge is started, the terminal will spawn you an endpoint.
- We enumerated this endpoint with `nmap`:

```bash
─$ nmap -T 4 -sC 104.154.172.3 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-29 14:50 CET
Nmap scan report for 3.172.154.104.bc.googleusercontent.com (104.154.172.3)
Host is up (0.13s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE    SERVICE
22/tcp   open     ssh
| ssh-hostkey: 
|   256 d5:53:0f:37:a0:e0:95:0e:fa:88:3f:40:09:ff:a0:a3 (ECDSA)
|_  256 34:de:3f:51:1d:5a:31:7f:8e:60:0f:04:f6:52:4e:16 (ED25519)
25/tcp   filtered smtp
8000/tcp open     http-alt
|_http-title: Santa Vision
9001/tcp open     tor-orport
```

## Silver 🥈

### A

- From NMAP we know that portal is running on port 8000.
- In the website there is copyright note: ```(topic 'sitestatus' available.)```
- probably we will need to subscribe MQTT for this topic.
- In HTML source code webpage there is a note:

```html
<!-- mqtt: elfanon:elfanon -->
```

- These credentials give us an access into this portal.

### B

- In the challenge there is description:
`Once logged on, authenticate further without using Wombley's or Alabaster's accounts to see the northpolefeeds on the monitors. What username worked here?`
- In the camera portal we discovered Clients/users:
`Available clients: 'elfmonitor', 'WomblyC', 'AlabasterS'`
- In the camera portal we discovered Roles:
`Available roles: 'SiteDefaultPasswordRole', 'SiteElfMonitorRole', 'SiteAlabsterSAdminRole', 'SiteWomblyCAdminRole'`
- Apparently the credentials fro the clients are their role, e.g,:
```elfmonitor:SiteElfMonitorRole```

### C

- MQTT messages in **frostbitfeed**:

```txt
Error msg: Unauthorized access attempt. /api/v1/frostbitadmin/bot/<botuuid>/deactivate, authHeader: X-API-Key, status: Invalid Key, alert: Warning, recipient: Wombley

Let's Encrypt cert for api.frostbit.app verified. at path /etc/nginx/certs/api.frostbit.app.key

Additional messages available in santafeed
```

- MQTT messages in **santafeed**:

```txt
Santa is checking his list

Sixteen elves launched operation: Idemcerybu
Santa is checking his list

Santa is on his way to the North Pole

superAdminMode=true

Santa role: superadmin


Santa is making his list

Santa is making his list

Santa role: superadmin

Santa is making his list

Santa is making his list

Santa is on his way to the North Pole

AlabasterS role: admin

Santa is on his way to the North Pole

Santa is checking his list

singleAdminMode=false

AlabasterS role: admin
```

Answer for C: ***Idemcerybu***

### D

- We need to enable only ***singleAdminMode*** mode:

```bash
└─$ mosquitto_pub -h 104.154.172.3 -t 'idemcerybu' -u elfmonitor -P SiteElfMonitorRole -m "singleAdminMode=true"
```

- Answer for D is: ***"pogo stick"***

## Gold 🥇

### A

- We need to subscribe for a topic mentioned in footer by MQTT terminal.
- This document can be useful: <https://github.com/kh4sh3i/MQTT-Pentesting>

- The topic is ```sitestatus```

```bash
mosquitto_sub -h 104.154.172.3 -t 'sitestatus' -u elfanon -P elfanon
```

- The MQTT will give you one important message where you can find an image of JFFS2 file system: ```/static/sv-application-2024-SuperTopSecret-9265193/applicationDefault.bin>```

- To open the image you need to follow this documentation:
<https://github.com/onekey-sec/jefferson/>

- After extracting of the JFFS2 image, you will get a source code for this project.
- In file ```views.py``` is a trace for a next step. Direct path where application is checking for credentials: 

```python
@accounts_bp.route("/sv2024DB-Santa/SantasTopSecretDB-2024-Z.sqlite", methods=["GET"])
```

- Examined the SQLlite database and get following credentials:
```santaSiteAdmin:S4n+4sr3411yC00Lp455wd```

### B

- According to HINTs from ELF, we need to look on headers after logon with new discovered user.
- Analyze all requests and response which happen immediately after login and you will find one which contain 3 headers ***brkruser & brkrpswd & brkrtopic***.
- Credentials for new discovered account are: ```santashelper2024:playerSantaHelperPass3991439085```
- Topic: ```northpolefeeds```

### C

- Again, the message for golden objective C is the same as was for Silver.
- But, this time we need to use ROT13 with rotation of 10 (ROT16 hint - `sixteen elves`) on:
***“Sixteen elves launched operation: Idemcerybu“***
- Answer is: `Snowmobile`
