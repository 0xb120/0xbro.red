---
layout: page
icon: fas fa-handshake
order: 4
toc: true
---

## Vulnerability Disclosure Policy
**Current version**: v1.0, last changed on October 17, 2023, 14.40<br>
**PDF version**: <a href="{{site.url}}/assets/pdf/disclosure-policy-v1.0.pdf">disclosure-policy-v1.0.pdf</a>

This policy governs the process adopted by 0xbro for reporting and disclosing vulnerabilities to product or security vendors, customers, developers, and the general public.

0xbro will **<u>always</u>** attempt to coordinate all reported vulnerabilities with the affected parties following a coordinated vulnerability disclosure process (also known as responsible disclosure process).

Coordinated disclosure is the most effective approach to address vulnerabilities and ensure the protection of customers. Despite this, the affected parties frequently exhibit deliberate negligence, excessive delays in developing necessary fixes, or lack of transparency towards vulnerability reports submitted them, leaving customers exposed for an irresponsible amount of time.

For this reason, **0xbro may disclose vulnerabilities to the general public fifteen (15) days after the last contact attempt without response** (see process below). **Depending on the circumstances**, including active exploitation, significant or minor threats, or the need to modify an existing standard, **the timing of disclosure may be adjusted either earlier or later**.

***Disclosure process:***

1. The **first contact attempt** will be through any formal mechanisms listed on the affected party web site (eg. an explicit security contact, contacts inside `/.well-known/security.txt`, etc.), or by sending an e-mail to security@, support@, info@, and secure@company.com requesting information about a security referent and a secure channel over which disclosing the technical vulnerability details. 0xbro will **<u>never</u>** provide information or insights about the vulnerability during this stage.

2. **If the first attempt is not followed by a response within seven (7) days**, a message requesting the security contact e-mail address may initially be sent to certain public contacts associated with the affected party (eg. public forms, general e-mails, social media, etc.) in what is considered a **second contact attempt**. Also in this case, 0xbro will **<u>never</u>** provide information or insights about the vulnerability during this stage..

3. **If the second attempt is not followed by a response within seven (7) days, the disclosure dates is set fifteen (15) days later.**

4. **If the involved party response is received within the fourteen (7+7) days timeframe** outlined above, 0xbro will **send a report** with the vulnerability details and some potential mitigation or workarounds. From this moment, **the involved party has thirty (30) days to address the vulnerability** with a security patch or other corrective measure as appropriate. Extensions to the 30-day disclosure timeline ***may*** be granted depending on the complexity and the effort spent to fix the flaw.

5. **If no response has been received after the 30-day disclosure timeline, the vulnerability is published immediately without further coordination attempts.**

6. **During the vulnerability analysis and remediation period**, 0xbro expects **regular updates** about the ongoing activities and is willing to cooperate appropriately for solving the problem and conducting appropriate retests. If no updates are provided by default, the **affected party will be contacted about once a week** with a status update request.

7. **If no response has been received after two (2) consecutive status update requests, no extension of the disclosure date shall be granted and the same rule as in 5th point applies.**


Vulnerabilities are disclosed to the public under these three circumstances:
- The pre-set/agreed disclosure date is reached.
- The involved party issues a fix and/or other corrective measure.
- Information about the same vulnerability is published by a third party.

Under **<u>no circumstances</u>** will a vulnerability that has been discovered **be suppressed or kept undisclosed** due to the unwillingness of an involved party to address it. Delays (not exceeding 15 days) to the disclosure date can be agreed upon following a fix to give customers time to install the appropriate patches.

In respect of the researcher and the time invested, the affected party is strongly encouraged to appropriately **acknowledge** and **give credit** to the reporter who submitted the issue.

If you have any questions about this policy, please <a href="{{ site.url }}/about/#contact">contact me</a>.

## PDF Version

#TODO
Qui devo cambiare il pdf con quello delle disclosure

<div style="text-align: center; margin: 30px 0;">
  <a href="{{site.url}}/assets/pdf/disclosure-policy-v1.0.pdf" class="btn btn-primary" download style="display: inline-block; padding: 12px 30px; background-color: #0d6efd; color: white; text-decoration: none; border-radius: 5px; font-weight: 500; box-shadow: 0 2px 4px rgba(0,0,0,0.1); transition: all 0.3s ease;">
    📄 Download Full CV (PDF)
  </a>
  <p style="margin-top: 10px; color: #6c757d; font-size: 0.9em;">Last updated: {{ site.time | date: "%B %Y" }}</p>
</div>

<style>
  .resume-container {
    position: relative;
    width: 100%;
    margin-top: 20px;
  }
  
  .resume-container iframe {
    border: 1px solid #dee2e6;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    width: 100%;
    height: 80vh;
    min-height: 600px;
  }
  
  @media (max-width: 768px) {
    .resume-container iframe {
      height: 70vh;
      min-height: 500px;
    }
    
    .mobile-notice {
      display: block;
      padding: 15px;
      background-color: #fff3cd;
      border: 1px solid #ffc107;
      border-radius: 5px;
      margin-bottom: 15px;
      text-align: center;
    }
  }
  
  @media (min-width: 769px) {
    .mobile-notice {
      display: none;
    }
  }
</style>

<div class="mobile-notice">
  📱 For the best experience on mobile, please download the PDF above.
</div>

<div class="resume-container">
  <iframe src="https://mozilla.github.io/pdf.js/web/viewer.html?file={{site.url}}/assets/pdf/disclosure-policy-v1.0.pdf" title="0xbro - Vulnerability Disclosure Policy"></iframe>
</div>