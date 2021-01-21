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
