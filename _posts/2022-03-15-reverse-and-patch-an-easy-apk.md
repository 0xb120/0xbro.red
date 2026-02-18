---
title: "Reverse and patch an easy APK"
date: 2022-03-15
categories: [Articles & Writeups, Android]
tags: [hackthebox, android]
description: Learn how to reverse and patch an easy APK in this Android HackTheBox challange called APKrypt. 
toc: true
media_subpath: /assets/img/APKrypt/
image: thumbnail.png
---

## Introduction
Learn how to **disassemble, decompile, reverse, analyse** and **patch** an easy APK in this **Android HackTheBox challange** called **APKrypt**.

### Improved skills
- **Disassemble** APK
- **Decompile** .dex file 
- **Reverse Engineering** Android applications
- **Patch and rebuild** modified .smali code

### Used tools
- **APKTool**
- **dex2jar**
- **jadx-gui**
- **bytecode-viewer**
- **JD-GUI**
- **keytool** & **jarsigner**


## Video
<iframe width="736" height="491" src="https://www.youtube.com/embed/gPPi3v8iRog" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Notes
### Decompile APK
```bash
$ apktool d APKrypt.apk
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.5.0-dirty on APKrypt.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/kali/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...

$ ls APKrypt
AndroidManifest.xml  apktool.yml  original  res  smali
```

### Patch, rebuild, and sign the modified APK
```bash
$ echo -n maoutis | md5sum
6f2ae4978075eae54f9491744818d28d  -

$ grep -ri "735c3628699822c4c1c09219f317a8e9"
smali/com/example/apkrypt/MainActivity$1.smali:    const-string v0, "735c3628699822c4c1c09219f317a8e9"

$ sed -i 's/735c3628699822c4c1c09219f317a8e9/6f2ae4978075eae54f9491744818d28d/' smali/com/example/apkrypt/MainActivity\$1.smali

$ java -jar /opt/Android/apktool_2.6.1.jar b ./APKrypt
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
I: Using Apktool 2.6.1
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building resources...
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk...

$ keytool -genkey -v -keystore key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias HTB-alias
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:
What is the name of your organizational unit?
  [Unknown]:
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:
Is CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 10,000 days
        for: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing key.jks]

$ jarsigner -keystore key.jks APKrypt.apk HTB-alias
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter Passphrase for keystore:
jar signed.

Warning:
The signer certificate is self-signed.

$ sudo cp APKrypt.apk /mnt/hgfs/VM-Shared/HTB/APKrypt-patched.apk
```
