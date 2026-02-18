---
title: "My OSCP Journey"
date: 2021-06-04
categories: [Articles & Writeups, InfoSec Education]
toc: true
media_subpath: /assets/img/OSCP-journey/
tags: [offsec, hacking-courses, OSCP]
description: A brief description of my experience with OSCP certification and the respective PEN-200 course.
image: offsec.png
---

Italian article available [here](https://www.inforge.net/forum/threads/la-mia-esperienza-con-la-certificazione-oscp.604245/)

## Pre-course
### Backgrund and preparation
When I enrolled in the PWK course, **I had worked as a software developer for 2 years** and had been working as a **Penetration Tester for another 2 years**. <br>
**Prior to OSCP I had not yet obtained any certification**, the only course I had taken was the eLearnSecurity's PTSv4, but without attempting the related eJPT certification.

Having been **self-taught for about 4 years**, **playing HackTheBox from a while** and now **working in the industry** from a couple of years, I already mastered the pre-requisites suggested by Offensive Security, so I decided to enroll.

***
## PEN-200 Course and Lab
I **enrolled** in the PEN-200 (PWK 2020) on the **15th of Jennuary 2021**. The course then **started** on the **24th of Jenuary** with **90 days of lab access**.


### Day 01-30
I spent the first month **looking at the course material** (850+ pdf pages and 17+ hours of video) and **doing the exercises** required to get the 5 bonus points during the exam. Although they were not difficult, the **exercises were many and required a lot of time to be done correctly** (to get the 5 bonus points the exercises must all be complete and correct), so I decided to start them from the first day. 

While reading the pdf I also began **structuring notes and cheatsheets** to use during the exam so that I would have all the useful references and commands ready at hand.


### Day 31-60
Finally after 30 days of exercises and notes writing I was able to start hacking some boxes. I decided to **start the journey following the [PWK Labs Learning Path](https://help.offensive-security.com/hc/en-us/articles/360050473812-PWK-Labs-Learning-Path)** and then proceed autonomously with the "*low hanging fruits*". 

In the meantime **I took notes for each one of the machines penetrated** and **I wrote short writeups** so that I would have an executive summary and a brief overview for each of them.

### Day 61-90
During the last month of access to the lab I did the **extra mile exercises** and I **hacked as many machines as possible**, trying to maintain an average of **1 machine per day**.

At the end of the lab time **I finished with 64 hacked machines** (out of a total of about 70), one domain controller completely compromised and access to all 3 internal networks.


### Up to the day before the exam
Between the end of the course and the final exam **I decided to let a month pass** in order to have the necessary **time to review or extend any doubts or knowledge** in addition to **writing the report of the lab** (a complete writeup of 10 machines with different attack vector + all pdf exercises) to totalize the 5 additional points.

In this month I have followed **two Udemy courses focused on Windows Privilege Escalation** ("*Windows Privilege Escalation for OSCP & Beyond!*" and "*Windows Privilege Escalation for Beginners*") and I kept **rooting as many machines as possible on HackTheBox** (I already had a VIP subscription) inspired by the lists of *OSCP like* machines that can be found on the web. 

![HTB_OSCP-like-machines.jpg]({{page.screenshots}}HTB_OSCP-like-machines.jpg)


During each penetration test I **always kept taking notes and writing writeups**, in order to keep my second brain always updated and organized.

I ended up with a total of 51 rooted machines on HTB:
- Lame - 10.10.10.3 <br>
- Legacy - 10.10.10.4 <br>
- Devel - 10.10.10.5 <br>
- Beep - 10.10.10.7 <br>
- Optimum - 10.10.10.8 <br>
- Bastard - 10.10.10.9 <br>
- Arctic - 10.10.10.11 <br>
- Grandpa - 10.10.10.14 <br>
- Granny - 10.10.10.15 <br>
- Blue - 10.10.10.40 <br>
- Shocker - 10.10.10.56 <br>
- Jeeves - 10.10.10.63 <br>
- Bashed - 10.10.10.68 <br>
- Chatterbox - 10.10.10.74 <br>
- DevOops - 10.10.10.91 <br>
- Bounty - 10.10.10.93 <br>
- Jerry - 10.10.10.95 <br>
- Active - 10.10.10.100 <br>
- SecNotes - 10.10.10.97 <br>
- Access - 10.10.10.98 <br>
- Querier - 10.10.10.125 <br>
- Netmon - 10.10.10.152 <br>
- Bastion - 10.10.10.134 <br>
- SwagShop - 10.10.10.140 <br>
- Writeup - 10.10.10.138 <br>
- Jarvis - 10.10.10.143 <br>
- Networked - 10.10.10.146 <br>
- Postman - 10.10.10.160 <br>
- Traverxec - 10.10.10.165 <br>
- Obscurity - 10.10.10.168 <br>
- OpenAdmin - 10.10.10.171 <br>
- Traceback - 10.10.10.181 <br>
- Magic - 10.10.10.185 <br>
- Admirer - 10.10.10.187 <br>
- Cache - 10.10.10.188 <br>
- Blunder - 10.10.10.191 <br>
- Tabby - 10.10.10.194 <br>
- Buff - 10.10.10.198 <br>
- Ready - 10.10.10.220 <br>
- Delivery - 10.10.10.222 <br>
- Tenet - 10.10.10.223 <br>
- ScriptKiddie - 10.10.10.226 <br>
- Ophiuchi - 10.10.10.227 <br>
- Spectra - 10.10.10.229 <br>
- TheNotebook - 10.10.10.230 <br>
- Armageddon - 10.10.10.233 <br>
- Schooled - 10.10.10.234 <br>
- Atom - 10.10.10.237 <br>
- Love - 10.10.10.239 <br>
- Pit - 10.10.10.241 <br>
- Knife - 10.10.10.242 <br>

**To retrain my Buffer Overflow skills** I also executed some exercises from the TryHackMe module **[Buffer Overflow Prep](https://tryhackme.com/room/bufferoverflowprep)** - *Practice stack based buffer overflows!*. 

***
## Day before the exam
The day before the exam **I stayed completely away from** the hacking world and **everything related to the certification**. I rested, I had company, I had fun and I tried to sleep as much as possible in preparation for the big day (sleep that actually lacked because of the pressure).

***
## Exam
### Day one
The exam started at 9am, the proctors were on point and I didn't have any kind of problem with the VPN or accessing the platforms. 

I **started right away with the BOF machine** (while the scans were running on the other machines) and after about **1 hour and 45 minutes** (yes, it took me longer than it should have according to my schedule) I got the first 25 points without any great difficulty. 

Afterwards, I decided to take a look at all the scans and tackle **one of the two 20 points machines** to take the pressure off. After about **3 hours** I got the user and after other **45 minutes** I got the full 20 points.

At that point **I decided to go ahead with the second 20 points machine**. After 2 hours and 30 minutes I got the user flag but from there **I got stuck on the privilege escalation** process for about 1 hour and 15 minutes so that **I decided to try** to approach **an easy win 10 points machine** to restore a mental balance.

The **10 points machine was really a peace of cake** and it took me less than 30 minutes to root it.

After the 10 points machine **I decided to try and tackle the last missing box** before going back to try the privilege escalation on the second Medium machine. After about **2 hours and 30 minutes** of intense enumeration I got the user and after other **45 minutes** I got the root flag.

Sure to have totaled a good margin of points to pass the exam (≈90pt) I have decided to still try an hour of privilege escalation, however without success. In the remaining time of VPN access **I double-checked all the notes, procedures, exploits and I took any missing screenshots**.

Around 1:00 AM **I went to rest for about 4 hours** and at 5:00 AM I started writing the final report.


### Day two
The second day was all downhill. **The report** (which I had already structured in the previous days) **was just a sequence of copying and pasting from the notes** and a bit of research for vulnerability remediation, but nothing more than that. At **12:27** I had already finished checking the report and **I had already submitted it**.

### Exam history
**08:30** - Proctoring stuff<br>
**09:00** - Exam started<br>
**10:43** - 25 points Buffer Overflow box rooted<br>
**12:35** - Lunch break (30 mins)<br>
**13:34** - 20 points box (1) user shell obtained<br>
**14:14** - 20 points box (1) rooted<br>
**16:43** - 20 points box (2) user shell obtained<br>
**18:26** - 10 points box rooted<br>
**19:30** - Dinner break (30 mins)<br>
**21:06** - 25 points box user shell obtained<br>
**22:00** - 25 points box rooted<br>
**01:05** - Rest<br>
**05:00** - Breakfast<br>
**05:30** - Started writing the final report<br>
**12:27** - Submitted the final report<br>

***
## Certificate
**The day after** the delivery of the Lab and Exam reports (01/06/2021 @ 13:12) **I received the email confirming the achievement of the certification**.
<div data-iframe-width="150" data-iframe-height="270" data-share-badge-id="4d593371-2011-42f2-a299-f75cf614d881" data-share-badge-host="https://www.credly.com"></div><script type="text/javascript" async src="//cdn.credly.com/assets/utilities/embed.js"></script>
***
## Overview
My experience with this course and certification has been overall positive. Although I feel that the work required to obtain the 5 bonus points is excessively time-consuming (considering that access to the lab is activated from day 1 and that it is not possible to stop it) the experience in the lab was exceptional: 70 and more machines almost all different from each other, different Active Directories, different internal networks to reach, interdependencies between different machines. It was really fun!

PDF and videos are very well structured and clear, perhaps a little too redundant in the two versions (many times the videos merely repeated things seen in the pdf without further elaboration). Not a big deal anyway.

The exam was challenging but fun. Reporting was not an issue.

The key concepts for obtaining the certification are primarily three:
- **Try harder** (or rather "enuemreate harder")!
- **Organize your notes** and cheatsheet **properly** 
- **Google is your best friend**

If you can master these key concepts getting certified will be a breeze.

***
## Useful resources
### OSCP general resources
- [Offensive Security](https://www.offensive-security.com/) official web site
- [awesome-oscp](https://github.com/0x4D31/awesome-oscp): A curated list of awesome OSCP resources 
- [OSCP/Pen Testing Resources](https://medium.com/@sdgeek/oscp-pen-testing-resources-271e9e570d45)
- [OSCP Repo](https://github.com/rewardone/OSCPRepo): A list of commands, scripts, resources, and more that I have gathered and attempted to consolidate for use as OSCP (and more) study material.

### Working with Shells
- [Pimp My Shell](https://medium.com/bugbountywriteup/pimp-my-shell-5-ways-to-upgrade-a-netcat-shell-ecd551a180d2): 5 Ways to Upgrade a Netcat Shell
- [Upgrading Simple Shells to Fully Interactive TTYs](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)
- [Reverse Shell Cheat Sheet](https://highon.coffee/blog/reverse-shell-cheat-sheet/) (by HighOn.Coffee)
- [Reverse Shell Cheat Sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) (by pentestmonkey)
- [Shell Upgrade Cheat Sheet](https://mlcsec.com/shell-upgrade-cheat-sheet/) (by mlcsec)
- [Offensive Msfvenom](https://medium.com/@PenTest_duck/offensive-msfvenom-from-generating-shellcode-to-creating-trojans-4be10179bb86): from Generating Shellcode to Creating Trojans
- [Msfvenom payload generation](https://vuln.be/post/msfvenom-payload-generation/)

### General purpose hacking resource
- [HackTricks](https://book.hacktricks.xyz/): the place where to find every hacking trick/technique/researches and news.
- [ippsec](https://ippsec.rocks/?#): ippsec video search
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings): A list of useful payloads and bypass for Web Application Security and Pentest/CTF
- [All About OSCP](https://oscp.infosecsanyam.in/): find everything related to OSCP
- [OSCP preparation notes](https://oscpnotes.infosecsanyam.in/My_OSCP_Preparation_Notes.html) (by infosecsanyam)
- [Penetration Testing Tools Cheat Sheet](https://highon.coffee/blog/penetration-testing-tools-cheat-sheet/) (by HighOn.Coffee)
- [Penetration testing tools](https://root4loot.com/pentools/): A neat list of penetration testing (some red) tools with usage commands and examples for quick reference. Originally intended for OSCP students.
- [RTFM](https://github.com/leostat/rtfm): A database of common, interesting or useful commands, in one handy referable form
- [Pentest-Cheatsheets](https://github.com/Tib3rius/Pentest-Cheatsheets) (by Tib3rius)

### Linux specific resources
- [pwk-cheatsheet/linux-template](https://github.com/ibr2/pwk-cheatsheet/blob/master/linux-template.md)
- [Linux Snippets](https://mlcsec.com/posts/linux-snippets/) (by mlcsec)
- [Linux Commands Cheat Sheet](https://highon.coffee/blog/linux-commands-cheat-sheet/) (by HighOn.Coffee)


#### Post Exploitation & PrivEsc
- [GTFOBins](https://gtfobins.github.io/): a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems.
- [9 Ways to Backdoor a Linux Box](https://medium.com/@airman604/9-ways-to-backdoor-a-linux-box-f5f83bae5a3c)

##### Methodology
- [Privilege Escalation Checklist](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md#checklists) (by swisskyrepo)
- [Privilege Escalation Checklist](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist) (by HackTricks)
- [Privilege Escalation Checklist](https://oscp.infosecsanyam.in/priv-escalation/linux-priv-escalation/checklist-linux-privilege-escalation) (by infosecsanyam)
- [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) (by g0tmi1k)

##### Cheatsheet
- [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) (by g0tmi1k)
- [PayloadsAllTheThings - Linux Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
- [HackTricks - Linux Privilege Escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation)
- [Privilege Escalation Cheatsheet](https://0xsp.com/offensive/privilege-escalation-cheatsheet#toc-12) (by 0xsp)
- [Awesome Privilege Escalation](https://awesomeopensource.com/project/m0nad/awesome-privilege-escalation): A curated list of awesome privilege escalation
- [Ignitetechnologies/Privilege-Escalation ](https://github.com/Ignitetechnologies/Privilege-Escalation): aimed at the CTF Players and Beginners to help them understand the fundamentals of Privilege Escalation with examples.

##### Tools
- [Linux Local Enumeration Script](https://github.com/Arr0way/linux-local-enumeration-script): performs basic linux local enumeration, a first step in the local privilege escalation process.
- [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS): a script that search for possible paths to escalate privileges on Linux/Unix* hosts. The checks are explained on [book.hacktricks.xyz](https://book.hacktricks.xyz)
- [linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration): Linux enumeration tool for pentesting and CTFs with verbosity levels 
- [unix-privesc-check](https://github.com/pentestmonkey/unix-privesc-check): Shell script to check for simple privilege escalation vectors on Unix systems
- [LinEnum](https://github.com/rebootuser/LinEnum):  Scripted Local Linux Enumeration & Privilege Escalation Checks 
- [LES (Linux Exploit Suggester)](https://github.com/mzet-/linux-exploit-suggester): designed to assist in detecting security deficiencies for given Linux kernel/Linux-based machine.
- [crontab guru](https://crontab.guru/): The quick and simple editor for cron schedule expressions by Cronitor


### Windows specific resources
 - [pwk-cheatsheet/windows-template](https://github.com/ibr2/pwk-cheatsheet/blob/master/windows-template.md)
 - [Windows Snippets](https://mlcsec.com/posts/windows-snippets/) (by mlcsec)
 - [Basics of windows](https://sushant747.gitbooks.io/total-oscp-guide/content/basics_of_windows.html) from [total-oscp-guide](https://sushant747.gitbooks.io/total-oscp-guide/content/)
 - [ yeyintminthuhtut/Awesome-Red-Teaming](https://github.com/yeyintminthuhtut/Awesome-Red-Teaming): List of Awesome Red Teaming Resources

#### Post Exploitation & PrivEsc
- [LOLBAS](https://lolbas-project.github.io/#): every binary, script, and library that can be used for Living Off The Land techniques.

##### Methodology
- [Local Windows Privilege Escalation Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation) (by HackTricks)
- [Windows Privilege Escalation Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/) (by absolomb)
- [Windows Privilege Escalation Fundamentals](https://www.fuzzysecurity.com/tutorials/16.html) (by FuzzySecurity)
- [Windows Privilege Escalation Checklists](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md) (by netbiosX)
- [Local Windows Privilege Escalation Checklist](https://oscp.infosecsanyam.in/priv-escalation/windows-priv-escalation/checklist-local-windows-privilege-escalation) (by infosecsanyam)

##### Cheatsheet
- [PayloadsAllTheThings - Windows Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [HackTricks - Windows Local Privilege Escalation](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation)
- [Windows Privilege Escalation Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/) (by absolomb)
- [ emilyanncr/Windows-Post-Exploitation ](https://github.com/emilyanncr/Windows-Post-Exploitation): Windows post-exploitation tools, resources, techniques and commands to use during post-exploitation phase of penetration test.
- [red-team-cheatsheet](https://0xsp.com/offensive/red-ops-techniques/red-team-cheatsheet) (by 0xsp)
- [Privilege Escalation Cheatsheet](https://0xsp.com/offensive/privilege-escalation-cheatsheet#toc-0) (by 0xsp)
- [Windows Privilege Escalation Fundamentals](https://www.fuzzysecurity.com/tutorials/16.html) (by FuzzySecurity)
- [Awesome Privilege Escalation](https://awesomeopensource.com/project/m0nad/awesome-privilege-escalation): A curated list of awesome privilege escalation

##### Tools
- [Seatbelt](https://github.com/GhostPack/Seatbelt): a C# project that performs a number of security oriented host-survey "safety checks" relevant from both offensive and defensive security perspectives.
- [WindowsEnum](https://github.com/absolomb/WindowsEnum):  A Powershell Privilege Escalation Enumeration Script.
- [winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS): WinPEAS is a script that search for possible paths to escalate privileges on Windows hosts. The checks are explained on [book.hacktricks.xyz](https://book.hacktricks.xyz)
- [PowerUp](https://www.powershellempire.com/?page_id=378): a PowerShell tool to assist with local privilege escalation on Windows systems. It contains several methods to identify and abuse vulnerable services, as well as DLL hijacking opportunities, vulnerable registry settings, and escalation opportunities.
- [windows-privesc-check](https://github.com/pentestmonkey/windows-privesc-check): Standalone Executable to Check for Simple Privilege Escalation Vectors on Windows Systems
- [Watson](https://github.com/rasta-mouse/Watson): Enumerate missing KBs and suggest exploits for useful Privilege Escalation vulnerabilities
- [Sherlock](https://github.com/rasta-mouse/Sherlock): PowerShell script to quickly find missing software patches for local privilege escalation vulnerabilities.
- [Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng): a tool based on the output of Windows' systeminfo utility which provides the list of vulnerabilities the OS is vulnerable to, including any exploits for these vulnerabilities.
- [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester): compares a targets patch levels against the Microsoft vulnerability database in order to detect potential missing patches on the target.
- [GhostPack-Compiled Binaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries): Compiled Binaries for Ghostpack (.NET v4.0)
- [Nishang](https://github.com/samratashok/nishang): Nishang - Offensive PowerShell for red team, penetration testing and offensive security. 
- [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG): New version of RottenPotato as a C++ DLL and standalone C++ binary - no need for meterpreter or other tools.
- [Juicy Potato](https://github.com/ohpe/juicy-potato): A sugared version of RottenPotatoNG, with a bit of juice, i.e. another Local Privilege Escalation tool, from a Windows Service Accounts to NT AUTHORITY\SYSTEM.
- [UACME](https://github.com/hfiref0x/UACME): Defeating Windows User Account Control
