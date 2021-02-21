# HTB - Irked
## OS - Linux

I started with an nmap scan of Irked (IP address: 10.10.10.117).

```
$nmap -sC -sV -oN irked.nmap 10.10.10.117
Nmap scan report for 10.10.10.117
Host is up (0.094s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp  open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          54652/tcp6  status
|   100024  1          55037/udp   status
|   100024  1          55516/udp6  status
|_  100024  1          58121/tcp   status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.71 seconds
```

When I browse to the web page, I see a jpg and "IRC is almost working!" message. nmap scan doesn't show anything about IRC, so doing another nmap scan. (judging by the name, Irked might be something to do with IRC.)

```
$nmap -vvv -p- 10.10.10.117
Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-20 17:25 PST
Initiating Ping Scan at 17:25
Scanning 10.10.10.117 [2 ports]
Completed Ping Scan at 17:25, 0.12s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 17:25
Completed Parallel DNS resolution of 1 host. at 17:25, 1.03s elapsed
DNS resolution of 1 IPs took 1.03s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 17:25
Scanning 10.10.10.117 [65535 ports]
Discovered open port 22/tcp on 10.10.10.117
Discovered open port 80/tcp on 10.10.10.117
Discovered open port 111/tcp on 10.10.10.117
Increasing send delay for 10.10.10.117 from 0 to 5 due to max_successful_tryno increase to 4
Connect Scan Timing: About 5.61% done; ETC: 17:34 (0:08:42 remaining)
Increasing send delay for 10.10.10.117 from 5 to 10 due to max_successful_tryno increase to 5
Connect Scan Timing: About 9.86% done; ETC: 17:36 (0:09:18 remaining)
Connect Scan Timing: About 27.44% done; ETC: 17:37 (0:08:46 remaining)
Connect Scan Timing: About 33.34% done; ETC: 17:37 (0:08:08 remaining)
Increasing send delay for 10.10.10.117 from 10 to 20 due to max_successful_tryno increase to 6
Connect Scan Timing: About 52.69% done; ETC: 17:41 (0:07:31 remaining)
Connect Scan Timing: About 60.17% done; ETC: 17:42 (0:06:42 remaining)
Discovered open port 58121/tcp on 10.10.10.117
Discovered open port 8067/tcp on 10.10.10.117
Increasing send delay for 10.10.10.117 from 20 to 40 due to max_successful_tryno increase to 7
Increasing send delay for 10.10.10.117 from 40 to 80 due to max_successful_tryno increase to 8
Discovered open port 6697/tcp on 10.10.10.117

```
Ran nmap scans on each port again. 58121 turned out to be RPC service. Port 8067 was running IRC.

```
Nmap scan report for 10.10.10.117
Host is up (0.096s latency).

PORT     STATE SERVICE VERSION
8067/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.28 seconds
```

Trying to nc resulted in a timeout.

```
$nc 10.10.10.117 8067
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
ERROR :Closing Link: [10.10.14.16] (Ping timeout)
```


After learning some IRC commands, we repeat this:

```
$nc 10.10.10.117 8067
:irked.htb NOTICE AUTH :*** Looking up your hostname...
**PASS test123**
**NICK test123**
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
**USER test123 testing testing :test123**
:irked.htb 001 test123 :Welcome to the ROXnet IRC Network test123!test123@10.10.14.16
:irked.htb 002 test123 :Your host is irked.htb, running version Unreal3.2.8.1
:irked.htb 003 test123 :This server was created Mon May 14 2018 at 13:12:50 EDT
:irked.htb 004 test123 irked.htb Unreal3.2.8.1 iowghraAsORTVSxNCWqBzvdHtGp lvhopsmntikrRcaqOALQbSeIKVfMCuzNTGj
:irked.htb 005 test123 UHNAMES NAMESX SAFELIST HCN MAXCHANNELS=10 CHANLIMIT=#:10 MAXLIST=b:60,e:60,I:60 NICKLEN=30 CHANNELLEN=32 TOPICLEN=307 KICKLEN=307 AWAYLEN=307 MAXTARGETS=20 :are supported by this server
:irked.htb 005 test123 WALLCHOPS WATCH=128 WATCHOPTS=A SILENCE=15 MODES=12 CHANTYPES=# PREFIX=(qaohv)~&@%+ CHANMODES=beI,kfL,lj,psmntirRcOAQKVCuzNSMTG NETWORK=ROXnet CASEMAPPING=ascii EXTBAN=~,cqnr ELIST=MNUCT STATUSMSG=~&@%+ :are supported by this server
:irked.htb 005 test123 EXCEPTS INVEX CMDS=KNOCK,MAP,DCCALLOW,USERIP :are supported by this server
:irked.htb 251 test123 :There are 1 users and 0 invisible on 1 servers
:irked.htb 255 test123 :I have 1 clients and 0 servers
:irked.htb 265 test123 :Current Local Users: 1  Max: 1
:irked.htb 266 test123 :Current Global Users: 1  Max: 1
:irked.htb 422 test123 :MOTD File is missing
:test123 MODE test123 :+iwx
```


