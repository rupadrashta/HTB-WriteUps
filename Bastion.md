# HTB - Bastion
## OS - Windows (IP address: 10.10.10.134)

I started with an nmap scan of Bastion host. A bastion host is a jump host for remote desktop, so this box might be related to something like that.

```
$nmap -sC -sV -oN Bastion.nmap 10.10.10.134
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-29 15:44 PST
Nmap scan report for 10.10.10.134
Host is up (0.092s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -16m12s, deviation: 34m37s, median: 3m46s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-12-30T00:48:15+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-12-29T23:48:17
|_  start_date: 2020-12-29T23:44:42

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.21 seconds
```

SSH and SMB-related ports are open. I ran NSE for port 445 too, but did not get any useful results. 

I tried smbclient after that. 

```
$smbclient -U guest -L 10.10.10.134
Enter WORKGROUP\guest's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available

```
I had no idea what to do with a Backups folder accessible over SMB. After watching ippsec's walkthrough, I learned **I could mount the Bakcups folder locally**.

```
mkdir /mnt/smb - Create a local folder
mount -t cifs //10.10.10.134/Backups /mnt/smb
```




