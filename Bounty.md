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





