# HTB - Valentine
## OS - Linux

I started with an nmap scan of Valentine (10.10.10.79).

```
$nmap -sC -sV -oN Valentine.nmap 10.10.10.79
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-31 11:53 PST
Nmap scan report for 10.10.10.79
Host is up (0.094s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-12-31T19:57:21+00:00; +3m47s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 3m46s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
Ports 22, 80, and 443 are open. Searched for Apache httpd 2.2.22 vulns - https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-66/version_id-142323/Apache-Http-Server-2.2.22.html


When I went to http://10.10.10.79/, I see heartbleed logo, so this box has something to do with heartbleed vuln.

So I ran nmap again with safe scripts option. 

```
$nmap --script safe 10.10.10.79
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-31 11:56 PST
Pre-scan script results:
|_broadcast-wpad-discover: Failed to retrieve wpad.dat (http://wpad.net/wpad.dat) from server
| targets-asn: 
|_  targets-asn.asn is a mandatory parameter
Nmap scan report for 10.10.10.79
Host is up (0.096s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
|_banner: SSH-2.0-OpenSSH_5.9p1 Debian-5ubuntu1.10
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
| ssh2-enum-algos: 
|   kex_algorithms: (7)
|   server_host_key_algorithms: (3)
|   encryption_algorithms: (13)
|   mac_algorithms: (11)
|_  compression_algorithms: (2)
80/tcp  open  http
|_http-apache-negotiation: mod_negotiation enabled.
|_http-comments-displayer: Couldn't find any comments.
|_http-date: Thu, 31 Dec 2020 20:01:10 GMT; +3m47s from local time.
|_http-fetch: Please enter the complete path of the directory to save data in.
| http-headers: 
|   Date: Thu, 31 Dec 2020 20:01:08 GMT
|   Server: Apache/2.2.22 (Ubuntu)
|   X-Powered-By: PHP/5.3.10-1ubuntu3.26
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|   
|_  (Request type: HEAD)
|_http-mobileversion-checker: No mobile version detected.
| http-php-version: Versions from logo query (less accurate): 5.3.0 - 5.3.29, 5.4.0 - 5.4.45
| Versions from credits query (more accurate): 5.3.9 - 5.3.29
|_Version from header x-powered-by: PHP/5.3.10-1ubuntu3.26
|_http-referer-checker: Couldn't find any cross-domain scripts.
|_http-security-headers: 
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
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-xssed: No previously reported XSS vuln.
443/tcp open  https
|_http-apache-negotiation: mod_negotiation enabled.
|_http-comments-displayer: Couldn't find any comments.
|_http-date: Thu, 31 Dec 2020 20:01:10 GMT; +3m46s from local time.
|_http-fetch: Please enter the complete path of the directory to save data in.
| http-headers: 
|   Date: Thu, 31 Dec 2020 20:01:06 GMT
|   Server: Apache/2.2.22 (Ubuntu)
|   X-Powered-By: PHP/5.3.10-1ubuntu3.26
|   Vary: Accept-Encoding
|   Connection: close
|   Content-Type: text/html
|   
|_  (Request type: HEAD)
|_http-mobileversion-checker: No mobile version detected.
| http-php-version: Versions from logo query (less accurate): 5.3.0 - 5.3.29, 5.4.0 - 5.4.45
| Versions from credits query (more accurate): 5.3.9 - 5.3.29
|_Version from header x-powered-by: PHP/5.3.10-1ubuntu3.26
|_http-referer-checker: Couldn't find any cross-domain scripts.
| http-security-headers: 
|   Strict_Transport_Security: 
|_    HSTS not configured in HTTPS Server
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
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-xssed: No previously reported XSS vuln.
| ssl-ccs-injection: 
|   VULNERABLE:
|   SSL/TLS MITM vulnerability (CCS Injection)
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h
|       does not properly restrict processing of ChangeCipherSpec messages,
|       which allows man-in-the-middle attackers to trigger use of a zero
|       length master key in certain OpenSSL-to-OpenSSL communications, and
|       consequently hijack sessions or obtain sensitive information, via
|       a crafted TLS handshake, aka the "CCS Injection" vulnerability.
|           
|     References:
|       http://www.cvedetails.com/cve/2014-0224
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224
|_      http://www.openssl.org/news/secadv_20140605.txt
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-12-31T20:01:05+00:00; +3m47s from scanner time.
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://cvedetails.com/cve/2014-0160/
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
| ssl-poodle: 
|   VULNERABLE:
|   SSL POODLE information leak
|     State: VULNERABLE
|     IDs:  CVE:CVE-2014-3566  BID:70574
|           The SSL protocol 3.0, as used in OpenSSL through 1.0.1i and other
|           products, uses nondeterministic CBC padding, which makes it easier
|           for man-in-the-middle attackers to obtain cleartext data via a
|           padding-oracle attack, aka the "POODLE" issue.
|     Disclosure date: 2014-10-14
|     Check results:
|       TLS_RSA_WITH_AES_128_CBC_SHA
|     References:
|       https://www.openssl.org/~bodo/ssl-poodle.pdf
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
|       https://www.imperialviolet.org/2014/10/14/poodle.html
|_      https://www.securityfocus.com/bid/70574

Host script results:
|_clock-skew: mean: 3m46s, deviation: 0s, median: 3m46s
|_fcrdns: FAIL (No PTR record)
| unusual-port: 
|_  WARNING: this script depends on Nmap's service/version detection (-sV)

Post-scan script results:
| reverse-index: 
|   22/tcp: 10.10.10.79
|   80/tcp: 10.10.10.79
|_  443/tcp: 10.10.10.79
Nmap done: 1 IP address (1 host up) scanned in 69.67 seconds
```
