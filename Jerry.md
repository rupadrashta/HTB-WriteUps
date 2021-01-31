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
