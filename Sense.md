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

```
