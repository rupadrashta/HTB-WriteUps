# HTB - Devel
## OS - Windows

Devel seems similar to Optimum. I started with an nmap scan of Devel (IP address 10.10.10.5)

```
$nmap -sC -sV -o Devel 10.10.10.5
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-26 16:01 PDT
Nmap scan report for 10.10.10.5
Host is up (0.11s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.71 seconds
```


The scan shows that FTP and HTTP ports are open. Anonymous FTP is on. 
Microsoft IIS server is running, and FTP result shows that iisstart.htm and welcome.png, the files that are served by IIS Server, are in the same file path for FTP.

We can test it by sending a file through FTP. I created a random .htm file (since the other file was a .htm), and FTP'd to Devel VM, and then browsed to http://10.10.10.5/<FILENAME> and it worked.
  
  Tried to use Metasploit for exploitation. Metaspoit has an auxiliary FTP scanner to detect anonymous login (auxiliary/scanner/ftp/anonymous), but I could not figure out how to use that for exploiting. 

  So whatever I send now, has to be through FTP and it should be good enough to have a reverse tcp shell. 
  
 I used Social ENgineering Toolkit to create a payload with reverse_tcp, saved it as .aspx file, and FTP'd to 10.10.10.5. When I brosed to the payload.aspx file though, I got a runtime error. So that did not work. 
