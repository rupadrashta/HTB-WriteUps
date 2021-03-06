# HTB - Mirai
## OS - Linux

I started with an nmap scan of Mirai (IP address: 10.10.10.48)

```
──╼ $nmap -sC -sV 10.10.10.48
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-29 08:51 PST
Nmap scan report for 10.10.10.48
Host is up (0.093s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76
| dns-nsid: 
|_  bind.version: dnsmasq-2.76
80/tcp open  http    lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.13 seconds
```

So SSH, DNS/TCP and HTTP ports are open.

Connecting to the web page have me a 404, so I reset the machine. Even after that, I got the same results.

I used curl in verbose mode to get more info.

```
──╼ $curl -vvv http://10.10.10.48
*   Trying 10.10.10.48:80...
* TCP_NODELAY set
* Connected to 10.10.10.48 (10.10.10.48) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.48
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< X-Pi-hole: A black hole for Internet advertisements.
< Content-type: text/html; charset=UTF-8
< Content-Length: 0
< Date: Sun, 29 Nov 2020 17:15:15 GMT
< Server: lighttpd/1.4.35
< 
* Connection #0 to host 10.10.10.48 left intact
```

This time I see "X-Pi-hole: A black hole for Internet advertisements.". Searching for this showed that this was a header for Raspberry Pi devices. Then I searched for Pi hole. It acts as a DNS server for a private network, has a modified dnsmasq, curl, lighthttpd, php, and admin dashboard. It can be used to block DNS requests for advertising domains. 

Further research showed how to configure Pi hole...And the default SSH credentials are pi/raspberry. 

I was able to SSH as pi.

I got the user flag after logging in.


Running sudo shows that the user pi can run any command.
```
pi@raspberrypi:/home $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```
So I "sudo su -" and became root.

The root.txt file did not have the flag but the content:
```
I lost my original root.txt! I think I may have a backup on my USB stick...

```
The mount command shows 
```
/dev/sdb on /media/usbstick type ext4 (ro,nosuid,nodev,noexec,relatime,data=ordered)
```

A read-only disk.

When I cd to /media/usbstick, there's a file that shows that the content were deleted.

```
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
```
I did not know what /dev/sdb was, so I searched for it. 
The command "sudo fdisk -l" shows the disk attached to the USB stick. 

Running that command showed /dev/sdb.

cat command showed some unprintable chars, along with the above file. So I tried strings command. This time, the output showed 

```
^Croot@raspberrypi:~# strings /dev/sdb
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
>r &
/media/usbstick
lost+found
root.txt
damnit.txt
>r &
/media/usbstick
2]8^
lost+found
root.txt
damnit.txt
>r &
3d3e483143ff12ec505d026fa13e020b
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?
-James
```

I tried that long string  as the root flag and it worked.
