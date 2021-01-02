# HTB - Shocker
## OS - Linux

As usual, I started with an nmap scan of Shocker (IP address 10.10.10.56).

```
$nmap -sC -sV -oN Shocker.nmap 10.10.10.56
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-01 18:05 PST
Nmap scan report for 10.10.10.56
Host is up (0.092s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.27 seconds
```
Ports 80 and 2222 (SSH) are open. 

The index page on port 80 shows an image that says "Don't bug me". Not a lot of info to go on. So I ran nmap with safe scripts option and dirb tool.



dirb output:
```
$dirb http://10.10.10.56

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri Jan  1 18:08:21 2021
URL_BASE: http://10.10.10.56/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.56/ ----
+ http://10.10.10.56/cgi-bin/ (CODE:403|SIZE:294)                              
+ http://10.10.10.56/index.html (CODE:200|SIZE:137)                            
+ http://10.10.10.56/server-status (CODE:403|SIZE:299)       
```


nmap NSE output:
```
$nmap --script=safe 10.10.10.56
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-01 18:07 PST
Pre-scan script results:
|_broadcast-wpad-discover: Failed to retrieve wpad.dat (http://wpad.net/wpad.dat) from server
| targets-asn: 
|_  targets-asn.asn is a mandatory parameter
Nmap scan report for 10.10.10.56
Host is up (0.095s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
80/tcp   open  http
|_http-comments-displayer: Couldn't find any comments.
|_http-date: Sat, 02 Jan 2021 02:12:29 GMT; +3m47s from local time.
|_http-fetch: Please enter the complete path of the directory to save data in.
| http-headers: 
|   Date: Sat, 02 Jan 2021 02:12:29 GMT
|   Server: Apache/2.4.18 (Ubuntu)
|   Last-Modified: Fri, 22 Sep 2017 20:01:19 GMT
|   ETag: "89-559ccac257884"
|   Accept-Ranges: bytes
|   Content-Length: 137
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|   
|_  (Request type: HEAD)
|_http-mobileversion-checker: No mobile version detected.
|_http-referer-checker: Couldn't find any cross-domain scripts.
|_http-security-headers: 
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_      http://ha.ckers.org/slowloris/
|_http-title: Site doesn't have a title (text/html).
| http-useragent-tester: 
|   Status for browser useragent: 200
|   Allowed User Agents: 
|     Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
|     libwww
|     lwp-trivial
|     libcurl-agent/1.0
|     PHP/
|     Python-urllib/2.5
|     GT::WWW
|     Snoopy
|     MFC_Tear_Sample
|     HTTP::Lite
|     PHPCrawl
|     URI::Fetch
|     Zend_Http_Client
|     http client
|     PECL::HTTP
|     Wget/1.13.4 (linux-gnu)
|_    WWW-Mechanize/1.34
|_http-xssed: No previously reported XSS vuln.
2222/tcp open  EtherNetIP-1
|_banner: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2

Host script results:
|_clock-skew: 3m46s
|_fcrdns: FAIL (No PTR record)
| unusual-port: 
|_  WARNING: this script depends on Nmap's service/version detection (-sV)

Post-scan script results:
| reverse-index: 
|   80/tcp: 10.10.10.56
|_  2222/tcp: 10.10.10.56
Nmap done: 1 IP address (1 host up) scanned in 356.53 seconds
```

