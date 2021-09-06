# HTB - Teacher
## OS - Linux

Teacher's IP address is 10.10.10.153. Started with an nmap scan.
```
$nmap -sC -sV -oN nmap 10.10.10.153
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-06 07:59 PDT
Nmap scan report for 10.10.10.153
Host is up (0.088s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Blackhat highschool

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.11 seconds
```

Only port 80 is open.
The web interface shows a web page for school. There's an announcement that the web portal was new, that the students can submit their work. Only gallery.html seems to be working.

In the gallery page, the teacher photos are seen. One of the photos is not visible, the rest are not rendered properly. If we view the source of the page, we see that the photo that is not rendered is http://10.10.10.153/images/5.png.
