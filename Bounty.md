# HTB - Bounty
## OS - Windows

Started with an nmap scan as usual. (IP address of Bounty is 10.10.10.93)

```
$nmap -sC -sV -oN bounty.nmap 10.10.10.93
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-23 17:39 PST
Nmap scan report for 10.10.10.93
Host is up (0.093s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```
