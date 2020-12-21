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
