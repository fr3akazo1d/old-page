---
layout: post
title: Writeup - THM - Anthem
description: "Walkthrough of Anthem of TryHackme"
tags: [TryHackeMe, Anthem, Hacking, pwned]
author: Fr3akazo1d!
---

# General Information

| Room        | Date        | Difficulty | Tags                                   | Time     |
| ----------- | ----------- | ---------- | -------------------------------------- | -------- |
| Anthem      | 03.10.2020  | easy       | Windows, CMS, enumeration, Weekly Challange | -- |

# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial $ip -Pn|
| rdesktop | rdesktop -u s* -p U***************! -d anthem.com $ip |

# open ports

| Port | Service     | Version             |
| ---- | ----------- | ------------------- |
| 80   | Http-Server  | Microsoft HTTPAPI httpd 2.0        |
| 135   | msrpc         | Microsoft Windows RPC       |
| 139   | netbios-ssn | Microsoft Windows netbios-ssn |
| 445 |  microsoft-ds? | |
| 389 | ms-wbt-server | Microsoft Terminal Services |

This Room is about a Web-Server where we have to find different flags, usernames and possible Passwords on the Web-Server. 

It is a Windows-Machine where i don't have much experience for Priv-Escalation.  

So lets start with the Web-Analysis.

# Website Analysis

I started with an nmap scan to find the open Ports:
```sh
nmap -sC -sV -oN nmap/initial 10.10.151.141 -Pn
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-02 21:15 CEST
Nmap scan report for 10.10.151.141
Host is up (0.079s latency).
Not shown: 995 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2020-10-02T19:15:38+00:00
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2020-10-01T19:14:59
|_Not valid after:  2021-04-02T19:14:59
|_ssl-date: 2020-10-02T19:16:51+00:00; +2s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 1s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-10-02T19:15:41
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.04 seconds
```

Found 5 Open ports on the Target: 

1. 80 - Web-Server - Microsoft HTTPAPI httpd 2.0
2. 135 - msrpc- Microsoft Windows RPC
3. 139 - netbios-ssn - Microsoft Windows netbios-ssn
4. 445 - microsoft-ds?
5. 3389 - ms-wbt-server - Microsoft Terminal Services

The first thing i was looking was the Web-server. Was checking if the robots.txt file exists and if it hade some usefull information in it. 

```
U***************!

# Use for all search robots
User-agent: *

# Define the directories not to crawl
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/
```

As for the questions from THM, this clould be a possible Password for admin user.

So i decided to have a look further into the website. And i could answer all other question, only through looking at the source code of the websites. 

The username of the Admin i got via googeling the Poem witch i found in one Blog-Post on the Web-server. 

so we now need the email address of the administrator. When i was looking on the Blog-Post "We are hiring" i found one email address: JD@anthem.com. so we know the email address is build with the first letter of the Firstname and the first letter of the Lastname. 

# Spot the flags

For the flags, i was looking through the source-code and found all 4-Flags on the Web-server. Just keep looking through the Web-Site.

# Final stage

After we got the Username and the password for the Administrator we tried to loggin into the RDP-Connection. 

For the RDP-Login i used the program rdesktop on Kali with following command: 

```sh
rdesktop -u s* -p U***************! -d anthem.com 10.10.163.60
```

We have now access to the target and we can now read the user flag witch is lockated on ther Desktop. 

For the Root-Flag we have to look around on the Target. If we can find something we can use. 

I found a hidden file on the C-drive. When i was looking into that the folder there was a file located, but i don't have any permissions for the file. 

So i decided to gave the user the permissions of the file and then i have access to the file.

In this file, there is a word, witch could be the Admin-Password. 
Now we can go to the Administrator folder and read the root.txt file and then we have the root-flag.
