---
layout: post
title:  "Incorrect Authorization to Authenticated (Contributor+) Multiple Media Actions in Prevent Direct Access Wordpress Plugin (CVE-2025-3861)"
has_children: no
parent: "Disclosed vulnerabilities"
date: 2025-04-24
last_update: 2025-04-24
nav_order: 2025-04-24
reading_time: 2 min
tags: wordpress wordpress-plugin prevent-direct-access pda-lite
twitter_text: Authorization Bypass in Prevent Direct Access Wordpress Plugin (CVE-2025-3861)
description: The Prevent Direct Access – Protect WordPress Files plugin for WordPress is vulnerable to unauthorized access and modification of data| due to a misconfigured capability check on the 'pda_lite_custom_permission_check' function in versions 2.8.6 to 2.8.8.2. This makes it possible for authenticated attackers, with Contributor-level access and above, to access and change the protection status of media.
---

{% include header-disclosures.html %}

{: .abstract }
>The Prevent Direct Access – Protect WordPress Files plugin for WordPress is vulnerable to **unauthorized access and modification of data** due to a **misconfigured capability check** on the `pda_lite_custom_permission_check` function in versions 2.8.6 to 2.8.8.2. This makes it possible for *authenticated* attackers, with *Contributor-level* access and above, to access and change the protection status of media.

---

{% include TOC.md %}

## Summary

<dl>
  <dt>Product</dt>
  <dd><a href="https://wordpress.org/plugins/prevent-direct-access/">Prevent Direct Access – Protect WordPress Files</a></dd>
  <dt>Vendor</dt>
  <dd><a href="https://profiles.wordpress.org/buildwps/">WP Folio Team</a></dd>
  <dt>Active installations</dt>
  <dd>10,000+</dd>
  <dt>Severity</dt>
  <dd>Medium</dd>
  <dt>Affected Version(s)</dt>
  <dd>2.8.6 - 2.8.8.2</dd>
  <dt>Fixed version</dt>
  <dd>2.8.8.3</dd>
  <dt>CVE</dt>
  <dd><a href="https://www.cve.org/CVERecord?id=CVE-2025-3861">CVE-2025-3861</a></dd>
  <dt>CVE Description</dt>
  <dd>The Prevent Direct Access – Protect WordPress Files plugin for WordPress is vulnerable to unauthorized access and modification of data| due to a misconfigured capability check on the `pda_lite_custom_permission_check` function in versions 2.8.6 to 2.8.8.2. This makes it possible for authenticated attackers, with Contributor-level access and above, to access and change the protection status of media.</dd>
  <dt>CWE</dt>
  <dd><a href="https://cwe.mitre.org/data/definitions/863.html">CWE-863: Incorrect Authorization</a></dd>
</dl>

### CVSS 3.1 Score

**Base Score:** 5.4 (Medium)  
**Vector String:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:L/A:N`

|**Metric**|**Value**|
|---|---|
|**Attack Vector (AV)**|Network|
|**Attack Complexity (AC)**|Low|
|**Privileges Required (PR)**|Low|
|**User Interaction (UI)**|None|
|**Scope (S)**|Unchanged|
|**Confidentiality (C)**|Low|
|**Integrity (I)**|Low|
|**Availability (A)**|None|


## Vulnerability Details

Prevend Direct Access exposes 4 different REST api:

```json
[
  "/pda-lite/v1",
  "/pda-lite/v1/files/(?P<id>\\d+)",
  "/pda-lite/v1/private-urls/(?P<id>\\d+)",
  "/pda-lite/v1/un-protect-files/(?P<id>\\d+)"
]
```
These APIs are intended to manage and restrict Media access to administrators or media owners only.

Each of them validates the privileges of the user who is interacting with the API via a custom `callback` called `pda_lite_custom_permission_check`:

*/includes/pda_lite_api.php*
```php
...
register_rest_route(PDA_Lite_Constants::PREFIX_API_NAME, '/files/(?P<id>\d+)', array(
    'methods' => 'POST',
    'callback' => array($this, 'protect_files'),
    'permission_callback' => array( $this, 'pda_lite_custom_permission_check' ),
));
...
register_rest_route(PDA_Lite_Constants::PREFIX_API_NAME, '/files/(?P<id>\d+)', array(
    'methods' => 'GET',
    'callback' => array($this, 'is_protected'),
    'permission_callback' => array( $this, 'pda_lite_custom_permission_check' ),
));
...
public function pda_lite_custom_permission_check() {
    return current_user_can('edit_posts') || 
    current_user_can('manage_options');
}
```
In the `pda_lite_custom_permission_check()` function, authorization is granted if the user possesses either the `manage_options` or `edit_post` capability. However, according to the WordPress documentation [^doc], the `edit_post` capability is available to Contributor, Author, and Editor roles — not just Administrators. 

As a result, users with lower privileges can access the relevant APIs and perform various media-related actions, potentially bypassing media protection mechanisms.

[^doc]: [Roles and Capabilities](https://wordpress.org/documentation/article/roles-and-capabilities/)


### References

- [Root cause in *pda_lite_api.php*](https://plugins.trac.wordpress.org/browser/prevent-direct-access/tags/2.8.8.2/includes/pda_lite_api.php#L71)
- [Changeset for version 2.8.8.3](https://plugins.trac.wordpress.org/changeset/3279923/)

{% include share.html %}