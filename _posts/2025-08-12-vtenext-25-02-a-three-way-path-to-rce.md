---
title: "Vtenext 25.02 vulnerability research"
date: 2025-08-12
categories: [Disclosed Vulnerabilities]
tags: [PHP, ATO, xss, lfi, rce]
description: Multiple vulnerabilities in vtenext 25.02.1 and prior versions allow unauthenticated attackers to bypass authentication through three separate vectors, ultimately leading to remote code execution on the underlying server.
toc: true
media_subpath: /assets/img/vtenext-25-02-research/
image: vtenext2502-thumb.png
---

## Summary

<dl>
  <dt>Product</dt>
  <dd>VTENEXT CRM</dd>
  <dt>Vendor</dt>
  <dd>vtenext</dd>
  <dt>Severity</dt>
  <dd>Critical</dd>
  <dt>Impact</dt>
  <dd>Authentication Bypass and Remote Code Execution</dd>
  <dt>Affected Version(s)</dt>
  <dd>25.02* and below</dd>
  <dt>Tested Version(s)</dt>
  <dd>20.04, 24.02, 25.02, 25.02.1</dd>
  <dt>First Patched Version</dt>
  <dd>25.02.2</dd>
</dl>

### Abstract

{: .abstract }
>Multiple vulnerabilities in **vtenext 25.02.1** and prior versions allow *unauthenticated* attackers to **bypass authentication** through *three* separate vectors, ultimately leading to **remote code execution** on the underlying server.

You can read the full article on the [SicuraNext blog](https://blog.sicuranext.com/vtenext-25-02-a-three-way-path-to-rce/).

### Disclosure Timeline
> *Please refer to the <a href="{{site.url}}/disclosures/policy/">disclosure policy</a> page for further details about the disclosure policy adopted by 0xbro.*<br>

- **28/05/2025**: Contacted vtenext for the first time through various communication channels, but did not receive any response.
- **05/06/2025**: Contacted vtenext for the second time, but didn’t receive any response again.
- **09/06/2025**: Submitted CVE Request 1879483 to MITRE (still awaiting official CVEs).
- **13/07/2025**: Attempted to contact the developers of vtenext via a direct channel on LinkedIn, but without success.
- **24/07/2025**: Vendor released version 25.02.1 containing a silent patch for the Arbitrary Password Reset vulnerability.
- **12/08/2025**: Full disclosure, since a patch exists and the grace period has expired.
- **??/08/2025**: Vendor released version 25.02.2 containing a patch for the other vulnerabilities.
- **04/09/2025**: Following a call with the vtenext team, it was decided to reduce the technical details of the vulnerabilities to give users time to update to the latest version and the team time to patch the remaining vulnerabilities.
