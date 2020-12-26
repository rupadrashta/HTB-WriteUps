# HTB - Granny
## OS - Windows 

I want to do this box without using Metasploit, so let's see how it goes!

I started with an nmap scan of Granny whose IP address is 10.10.10.15.

```
$nmap -sC -sV -oN granny.nmap 10.10.10.15
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-26 15:27 PST
Nmap scan report for 10.10.10.15
Host is up (0.097s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Date: Sat, 26 Dec 2020 23:31:09 GMT
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.45 seconds

```

This is very similar to Grandpa box, so I ran davtest scan like Grandpa.

```
$davtest -url 10.10.10.15
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		10.10.10.15
********************************************************
NOTE	Random string for this session: FTVFUmrC1gdYm
********************************************************
 Creating directory
MKCOL		FAIL
********************************************************
 Sending test files
PUT	pl	FAIL
PUT	jsp	FAIL
PUT	aspx	FAIL
PUT	cgi	FAIL
PUT	html	FAIL
PUT	txt	FAIL
PUT	jhtml	FAIL
PUT	php	FAIL
PUT	cfm	FAIL
PUT	shtml	FAIL
PUT	asp	FAIL

********************************************************
/usr/bin/davtest Summary:

```
