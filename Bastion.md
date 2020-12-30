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
mount -t cifs //10.10.10.134/Backups /mnt/smb - This did not work, had to install cifs-utils.

$sudo mount.cifs  //10.10.10.134/Backups /mnt/smb
Password for root@//10.10.10.134/Backups:                          
$ls /mnt/smb/
note.txt  SDT65CB.tmp  WindowsImageBackup

```
I went to /mnt/smb and started exploring the folders. Note.txt wasn't useful. WindowsImageBackup had lots of files.

/mnt/smb/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351 had some vhd files and xml files. The xml files were unreadable. I learned that **vhd files can be extracted using 7zip***.

```
7z l 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
```
l to list the contents of the archive

The output is 

```
-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,6 CPUs Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz (206D7),ASM,AES-NI)

Scanning the drive for archives:
1 file, 37761024 bytes (37 MiB)

Listing archive: 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd

--
Path = 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd
Type = VHD
Physical Size = 37761024
Offset = 0
Created = 2019-02-22 12:44:00
Cluster Size = 2097152
Method = Dynamic
Creator Application = vsim 1.1
Host OS = Windows
Saved State = +
ID = B32434BB9E36E9119876080027DAEC14
----
Size = 104970240
Packed Size = 37748736
Created = 2019-02-22 12:44:00
--
Path = 9b9cfbc3-369e-11e9-a17c-806e6f6e6963.mbr
Type = MBR
Physical Size = 104970240
----
Path = 0.ntfs
Size = 104857600
File System = NTFS
Offset = 65536
Primary = +
Begin CHS = 321-3-2
End CHS = 281-1-4
--
Path = 0.ntfs
Type = NTFS
Physical Size = 104857600
Label = System Reserved
File System = NTFS 3.1
Cluster Size = 4096
Sector Size = 512
Record Size = 1024
Created = 2019-02-22 13:34:52
ID = 18056182273110301591

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-02-22 13:34:52 ..HS.       262144       262144  [SYSTEM]/$MFT
2019-02-22 13:34:52 ..HS.         4096         4096  [SYSTEM]/$MFTMirr
2019-02-22 13:34:52 ..HS.      2097152      2097152  [SYSTEM]/$LogFile
2019-02-22 13:34:52 ..HS.            0            0  [SYSTEM]/$Volume
2019-02-22 13:34:52 ..HS.         2560         4096  [SYSTEM]/$AttrDef
2019-02-22 04:37:35 D.HS.                            [SYSTEM]/.
2019-02-22 13:34:52 ..HS.         3200         4096  [SYSTEM]/$Bitmap
2019-02-22 13:34:52 ..HS.         8192         8192  [SYSTEM]/$Boot
2019-02-22 13:34:52 ..HS.            0            0  [SYSTEM]/$BadClus
2019-02-22 13:34:52 ..HS.            0            0  [SYSTEM]/$Secure
2019-02-22 13:34:52 ..HS.       131072       131072  [SYSTEM]/$UpCase
2019-02-22 13:34:52 D.HS.                            [SYSTEM]/$Extend
2019-02-22 13:34:52 ..HSA            0            0  [SYSTEM]/$Extend/$Quota
2019-02-22 13:34:52 ..HSA            0            0  [SYSTEM]/$Extend/$ObjId
2019-02-22 13:34:52 ..HSA            0            0  [SYSTEM]/$Extend/$Reparse
2019-02-22 13:34:52 D.HS.                            [SYSTEM]/$Extend/$RmMetadata
2019-02-22 13:34:52 ..HSA            0            0  [SYSTEM]/$Extend/$RmMetadata/$Repair
2019-02-22 13:34:52 D.HS.                            [SYSTEM]/$Extend/$RmMetadata/$TxfLog
2019-02-22 13:34:52 D.HS.                            [SYSTEM]/$Extend/$RmMetadata/$Txf
2019-02-22 13:34:52 ..HSA          100          100  [SYSTEM]/$Extend/$RmMetadata/$TxfLog/$Tops
2019-02-22 04:43:54 ....A        65536        65536  [SYSTEM]/$Extend/$RmMetadata/$TxfLog/$TxfLog.blf
2019-02-22 04:43:54 ....A      3145728      3145728  [SYSTEM]/$Extend/$RmMetadata/$TxfLog/$TxfLogContainer00000000000000000001
2019-02-22 13:37:16 ....A      3145728      3145728  [SYSTEM]/$Extend/$RmMetadata/$TxfLog/$TxfLogContainer00000000000000000002
2019-02-22 13:37:05 D.HS.                            Boot
2019-02-22 13:37:04 ..HSA        65536        65536  Boot/BOOTSTAT.DAT
2019-02-22 13:37:04 D....                            Boot/cs-CZ
2009-07-13 17:17:52 ....A        89168        90112  Boot/cs-CZ/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/da-DK
2009-07-13 17:17:51 ....A        87616        90112  Boot/da-DK/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/de-DE
2009-07-13 17:17:51 ....A        91712        94208  Boot/de-DE/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/el-GR
2009-07-13 17:17:54 ....A        94800        98304  Boot/el-GR/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/en-US
2009-07-13 17:17:51 ....A        85056        86016  Boot/en-US/bootmgr.exe.mui
2011-04-11 18:15:48 ....A        43600        45056  Boot/en-US/memtest.exe.mui
2019-02-22 13:37:04 D....                            Boot/es-ES
2009-07-13 17:17:51 ....A        90192        94208  Boot/es-ES/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/fi-FI
2009-07-13 17:17:51 ....A        89152        90112  Boot/fi-FI/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/fr-FR
2009-07-13 17:17:51 ....A        93248        94208  Boot/fr-FR/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/hu-HU
2009-07-13 17:17:51 ....A        90688        94208  Boot/hu-HU/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/it-IT
2009-07-13 17:17:54 ....A        90704        94208  Boot/it-IT/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/ja-JP
2009-07-13 17:17:51 ....A        76352        77824  Boot/ja-JP/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/ko-KR
2009-07-13 17:17:51 ....A        75344        77824  Boot/ko-KR/bootmgr.exe.mui
2010-11-20 13:29:11 ....A       485760       487424  Boot/memtest.exe
2019-02-22 13:37:04 D....                            Boot/nb-NO
2009-07-13 17:17:54 ....A        88144        90112  Boot/nb-NO/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/nl-NL
2009-07-13 17:17:51 ....A        90704        94208  Boot/nl-NL/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/pl-PL
2009-07-13 17:17:54 ....A        90704        94208  Boot/pl-PL/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/pt-BR
2009-07-13 17:17:51 ....A        90176        94208  Boot/pt-BR/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/pt-PT
2009-07-13 17:17:51 ....A        89664        90112  Boot/pt-PT/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/ru-RU
2009-07-13 17:17:52 ....A        90192        94208  Boot/ru-RU/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/sv-SE
2009-07-13 17:17:51 ....A        87616        90112  Boot/sv-SE/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/tr-TR
2009-07-13 17:17:51 ....A        87104        90112  Boot/tr-TR/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/zh-CN
2009-07-13 17:17:51 ....A        70720        73728  Boot/zh-CN/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/zh-HK
2009-07-13 17:17:52 ....A        70224        73728  Boot/zh-HK/bootmgr.exe.mui
2019-02-22 13:37:04 D....                            Boot/zh-TW
2009-07-13 17:17:51 ....A        70208        73728  Boot/zh-TW/bootmgr.exe.mui
2010-11-20 13:29:06 .RHSA       383786       385024  bootmgr
2019-02-22 13:37:05 D....                            Boot/Fonts
2009-06-10 13:15:17 ....A      3694080      3694592  Boot/Fonts/chs_boot.ttf
2009-06-10 13:15:17 ....A      3876772      3878912  Boot/Fonts/cht_boot.ttf
2009-06-10 13:15:18 ....A      1984228      1986560  Boot/Fonts/jpn_boot.ttf
2009-06-10 13:15:18 ....A      2371360      2371584  Boot/Fonts/kor_boot.ttf
2009-06-10 13:15:18 ....A        47452        49152  Boot/Fonts/wgl4_boot.ttf
2019-02-22 04:43:52 ....A        24576        24576  Boot/BCD
2019-02-22 04:43:52 ..HSA        21504        24576  Boot/BCD.LOG
2019-02-22 13:37:05 ..HSA            0            0  Boot/BCD.LOG1
2019-02-22 13:37:05 ..HSA            0            0  Boot/BCD.LOG2
2019-02-22 13:37:05 .RHSA         8192         8192  BOOTSECT.BAK
2019-02-22 04:43:52 D.HS.                            System Volume Information
2019-02-22 04:37:35 ..HSA        20480        20480  System Volume Information/tracking.log
2019-02-22 04:43:52 ..HSA            0            0  [SYSTEM]/$Extend/$UsnJrnl
2019-02-22 04:43:52 D.HS.                            System Volume Information/SPP
2019-02-22 04:43:52 D.HS.                            System Volume Information/SPP/OnlineMetadataCache
2019-02-22 04:43:52 ..HSA         1192         4096  System Volume Information/SPP/snapshot-2
2019-02-22 04:43:52 ..HSA         1192         4096  System Volume Information/SPP/OnlineMetadataCache/{6353c538-7915-4028-bd65-16546650aba6}_OnDiskSnapshotProp
2019-02-22 04:43:52 ..HSA      5192360      5193728  System Volume Information/SPP/metadata-2
2019-02-22 04:43:52 ..HSA     33554432     33554432  System Volume Information/{bb3424b3-369e-11e9-9876-080027daec14}{3808876b-c176-4e48-b7ae-04046e6cc752}
2019-02-22 04:43:52 ..HSA        65536        65536  System Volume Information/{3808876b-c176-4e48-b7ae-04046e6cc752}
------------------- ----- ------------ ------------  ------------------------
2019-02-22 13:37:16           62687034     62771300  62 files, 33 folders
2019-02-22 13:34:52            1318548      1581096  5 alternate streams
2019-02-22 13:37:16           64005582     64352396  67 streams

```


There were two vhd files in that folder. One shows the Windows file system. 


I learned that **mounting vhd files is done using guestmount command. To install this, you need to run "apt install libguestfs-tools"**

Then make directory /mnt/vhd on the attacker box. Then use guestmount command to mount the vhd file.

```
$guestmount --add 9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro -v /mnt/vhd 
```

We can cd to /mnt/vhd and view the contents. I had to sudo -i as root on the attacker host.

```
──╼ $sudo -i
┌─[root@parrot]─[~]
└──╼ #cd /mnt/vhd
┌─[root@parrot]─[/mnt/vhd]
└──╼ #ls
'$Recycle.Bin'             pagefile.sys     Recovery
 autoexec.bat              PerfLogs        'System Volume Information'
 config.sys                ProgramData      Users
'Documents and Settings'  'Program Files'   Windows

```



