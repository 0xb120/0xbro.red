---
title: "Intercept HTTPS on non-rooted Android devices"
date: 2022-09-20
categories: [Articles & Writeups, Android]
tags: [hackthebox, android]
description: Learn how to intercept HTTPS on non-rooted Android devices in this Android HackTheBox challange called Anchored.
toc: true
media_subpath: /assets/img/Anchored/
image: Thumbnail.png
---

## Introduction
In this HackTheBox challenge, named Anchored, we **reverse engineer and patch** an APK to **bypass certificate pinning** to be able to intercept application requests on **non-rooted** Android devices.


### Improved skills
- **Disassemble** APK
- **Decompile** .dex file 
- **Reverse Engineering** Android applications
- **Bypass certificate pinning** security implementations (**network_security_config.xml**)

### Used tools
- **APKTool**
- **jadx**
- **jd-gui**
- **Burpsuite**

## Video
<iframe width="736" height="491" src="https://www.youtube.com/embed/KGdCvJs9w7w" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Notes
### AndroidManifest.xml
AndroidManifest.xml do not present the **minSdkVersion** field, allowing to install the application on any desired Android version:
```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?><manifest xmlns:android="http://schemas.android.com/apk/res/android" android:compileSdkVersion="31" android:compileSdkVersionCodename="12" package="com.example.anchored" platformBuildVersionCode="31" platformBuildVersionName="12">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application android:allowBackup="true" android:appComponentFactory="androidx.core.app.CoreComponentFactory" android:icon="@mipmap/ic_launcher" android:label="@string/app_name" android:networkSecurityConfig="@xml/network_security_config" android:roundIcon="@mipmap/ic_launcher_round" android:supportsRtl="true" android:theme="@style/Theme.Anchored">
        <activity android:exported="true" android:name="com.example.anchored.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

### Patched the network_security_config.xml
Added row 6 and 7 to the original file:
```xml
$ bat res/xml/network_security_config.xml 
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: res/xml/network_security_config.xml
       │ Size: 403 B
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ <?xml version="1.0" encoding="utf-8"?>
   2   │ <network-security-config>
   3   │     <domain-config cleartextTrafficPermitted="false">
   4   │         <domain includeSubdomains="true">anchored.com</domain>
   5   │         <trust-anchors>
   6   │         <certificates src="system" />
   7   │         <certificates src="user" overridePins="true" />
   8   │         <certificates src="@raw/certificate" />
   9   │         </trust-anchors>
  10   │     </domain-config>
  11   │ </network-security-config>
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

### Generated the key and signed the patched APK:

```jsx
$ keytool -genkey -v -keystore test.keystore -alias Test -keyalg RSA -keysize 1024 -sigalg SHA1withRSA -validity 10000
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter keystore password:
Re-enter new password: 
What is your first and last name?
  [Unknown]:  0xbro
What is the name of your organizational unit?
  [Unknown]:  maoutis
What is the name of your organization?
  [Unknown]:
What is the name of your City or Locality?
  [Unknown]:
What is the name of your State or Province?
  [Unknown]:
What is the two-letter country code for this unit?
  [Unknown]:
Is CN=0xbro, OU=maoutis, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Generating 1,024 bit RSA key pair and self-signed certificate (SHA1withRSA) with a validity of 10,000 days
        for: CN=0xbro, OU=maoutis, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing test.keystore]

Warning:
The generated certificate uses the SHA1withRSA signature algorithm which is considered a security risk. This algorithm will be disabled in a future update.
The generated certificate uses a 1024-bit RSA key which is considered a security risk. This key size will be disabled in a future update.

┌──(kali㉿kali)-[~/…/Anchored/apktool/Anchored/dist]
└─$ jarsigner -keystore test.keystore Anchored.apk -sigalg SHA1withRSA -digestalg SHA1 Test
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter Passphrase for keystore:
jar signed.

Warning:
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk. This algorithm will be disabled in a future update.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk. This algorithm will be disabled in a future update.
The RSA signing key has a keysize of 1024 which is considered a security risk. This key size will be disabled in a future update.

$ adb install Anchored.apk              
Performing Streamed Install
Success
```

### External resources
- [Intercepting HTTPS on Android](https://httptoolkit.tech/blog/intercepting-android-https/)
- [Intercepting Android HTTP](https://httptoolkit.tech/docs/guides/android/)
- [Network security configuration](https://developer.android.com/training/articles/security-config)
- [How to use frida on a non-rooted device](https://lief-project.github.io/doc/latest/tutorials/09_frida_lief.html)
- [Using Frida on Android without root](https://koz.io/using-frida-on-android-without-root/)
