# HTB - Blocky
## OS - Linux

I started with an nmap scan as usual. IP Address of Blocky: 10.10.10.37

```
$nmap -sC -sV -oN Blocky.nmap 10.10.10.37
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-09 12:06 PST
Nmap scan report for 10.10.10.37
Host is up (0.094s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.00 seconds
```

Ports 21, 22, and 80 are open. Internet search showed there are some interesting exploits in WordPress 4.8 that we might be able to use.

Ran dirbuster after that. There are a lot of results like "http://10.10.10.37/index.php/2017/07/02/welcome-to-blockycraft/".

It shows a post by user notch. http://10.10.10.37/wp-includes/ shows the php scripts loaded on the web site.

