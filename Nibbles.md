# HTB - Nibbles
## OS - Linux

I started with an nmap scan as usual. IP address of Nibbles is 10.10.10.75.

```$nmap -sC -sV -oN nibbles.nmap 10.10.10.75
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-11 16:40 PST
Nmap scan report for 10.10.10.75
Host is up (0.092s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.87 seconds
```
 
Ports 22 and 80 are open. 

I browsed to the index page of the web site. There was only "Hello World" there. That seemed odd, so I viewed source. It had a comment.

```
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

Went to http://10.10.10.75/nibbleblog. There was a blog post and some links but nothing seemed interesting. It was "powered by nibbleblog".

I ran dirbuster to find any files, but got nothing other than the icons folder. WHile that is going, I searched about nibbleblog.

I haven't heard of nibbleblog, but the source code is at https://github.com/dignajar/nibbleblog. There is no default password.

I also see a link - https://wikihak.com/how-to-upload-a-shell-in-nibbleblog-4-0-3/. Once we login, we can inject our own php shell and run commands.


Tring a bunch of passwords 5 times makes us blacklisted. So we have to wait until it is cleared. After a few tries, I looked up online and found the credentials were admin/nibbles.



