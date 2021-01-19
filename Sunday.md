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

Metasploit has an auxilliary module that can identify valid users through finger service.

