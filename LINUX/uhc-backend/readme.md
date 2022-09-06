---
title: "HTB - Backend"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Backend"
categories:
  - HTB
---

The given box ```Backend``` is a Linux machine 

- [HackTheBox- Backend](#hackthebox---Backend)


## Recon

### Nmap Scan

`nmap` found 2 open ports on TCP scan,

```shell
┌──(0xblake㉿kali)-[~/HTB/LINUX/Backend]
└─$ cat backend.nmap 
# Nmap 7.92 scan initiated Wed Aug 31 02:16:28 2022 as: nmap -Pn -sC -sV -oN backend.nmap -v 10.10.11.161
Nmap scan report for 10.10.11.161
Host is up (0.50s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
80/tcp open  http    uvicorn
|_http-title: Site doesnt have a title (application/json).
|_http-server-header: uvicorn
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     content-type: text/plain; charset=utf-8
|     Connection: close
|     Invalid HTTP request received.
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     date: Tue, 30 Aug 2022 19:27:39 GMT
|     server: uvicorn
|     content-length: 22
|     content-type: application/json
|     Connection: close
|     {"detail":"Not Found"}
|   GetRequest: 
|     HTTP/1.1 200 OK
|     date: Tue, 30 Aug 2022 19:27:25 GMT
|     server: uvicorn
|     content-length: 29
|     content-type: application/json
|     Connection: close
|     {"msg":"UHC API Version 1.0"}
|   HTTPOptions: 
|     HTTP/1.1 405 Method Not Allowed
|     date: Tue, 30 Aug 2022 19:27:32 GMT
|     server: uvicorn
|     content-length: 31
|     content-type: application/json
|     Connection: close
|_    {"detail":"Method Not Allowed"}
| http-methods: 
|_  Supported Methods: GET
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=8/31%Time=630E772F%P=aarch64-unknown-linux-gnu%
SF:r(GetRequest,AD,"HTTP/1\.1\x20200\x20OK\r\ndate:\x20Tue,\x2030\x20Aug\x
SF:202022\x2019:27:25\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x202
SF:9\r\ncontent-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{
SF:\"msg\":\"UHC\x20API\x20Version\x201\.0\"}")%r(HTTPOptions,BF,"HTTP/1\.
SF:1\x20405\x20Method\x20Not\x20Allowed\r\ndate:\x20Tue,\x2030\x20Aug\x202
SF:022\x2019:27:32\x20GMT\r\nserver:\x20uvicorn\r\ncontent-length:\x2031\r
SF:\ncontent-type:\x20application/json\r\nConnection:\x20close\r\n\r\n{\"d
SF:etail\":\"Method\x20Not\x20Allowed\"}")%r(RTSPRequest,76,"HTTP/1\.1\x20
SF:400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\
SF:r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.
SF:")%r(FourOhFourRequest,AD,"HTTP/1\.1\x20404\x20Not\x20Found\r\ndate:\x2
SF:0Tue,\x2030\x20Aug\x202022\x2019:27:39\x20GMT\r\nserver:\x20uvicorn\r\n
SF:content-length:\x2022\r\ncontent-type:\x20application/json\r\nConnectio
SF:n:\x20close\r\n\r\n{\"detail\":\"Not\x20Found\"}")%r(GenericLines,76,"H
SF:TTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20ch
SF:arset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x
SF:20received\.")%r(DNSVersionBindReqTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x
SF:20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(DNSStatusRe
SF:questTCP,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type:\x20tex
SF:t/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvalid\x20HTT
SF:P\x20request\x20received\.")%r(SSLSessionReq,76,"HTTP/1\.1\x20400\x20Ba
SF:d\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r\nConnec
SF:tion:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\.")%r(Term
SF:inalServerCookie,76,"HTTP/1\.1\x20400\x20Bad\x20Request\r\ncontent-type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\nInvali
SF:d\x20HTTP\x20request\x20received\.")%r(TLSSessionReq,76,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\ncontent-type:\x20text/plain;\x20charset=utf-8\r
SF:\nConnection:\x20close\r\n\r\nInvalid\x20HTTP\x20request\x20received\."
SF:);
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 31 02:18:08 2022 -- 1 IP address (1 host up) scanned in 99.22 seconds
```

## Enumeration

### Port 80 

#### Site

![[Pasted image 20220830222907.png]]

#### API bruteforcing

