# HTB - Arctic
## OS - Windows

I started with an nmap scan of the Arctic box (IP address: 10.10.10.11)

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-27 13:30 PST
Nmap scan report for 10.10.10.11
Host is up (0.096s latency).
Not shown: 997 filtered ports
PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 145.28 seconds

```

Port 8500/tcp looks interesting. FMTP protocol - Internet search showed that it was Flight Message Transfer Protocol, 
but port 8500/tcp is a port used for ColdFusion MX Server (edition 6) to allow remote access as a web server.

We can go to http://10.10.10.11:8500 and see two folders. Going to http://10.10.10.11:8500/CFIDE/ shows an Administrator folder. 

But it is a web page with a login. The username is prepopulated with "admin".

Online search showed this exploit - https://www.exploit-db.com/exploits/14641 using directory traversal.

When I tried the exploit, it worked! The password hash was 2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03. Tried it crack it online and the first result was https://hashtoolkit.com/decrypt-sha1-hash/2f635f6d20e3fde0c53075a84b68fb07dcec9b03

The decrypted password was happyday.

Logged in and browsed around, and generally searched online about ColdFusion.
