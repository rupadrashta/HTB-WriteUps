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

