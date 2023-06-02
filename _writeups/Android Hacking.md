---
layout: default
title:  "Android Hacking"
has_children: true
child_nav_order: reversed
nav_order: 10
has_toc: false
---

# Web Hacking 
**Videos and writeups** about **Android hacking** and **Android application security**.
I hope you will find them useful. In case of advice, feel free to contact me.

{% include HTB_Card.html %}

{% assign filtered_writeups = site.writeups | where: "parent", "Android Hacking" | sort: "date" | reverse %}
{% for writeup in filtered_writeups %}
{% if writeup.challenge_title != "" %}
- [{{ writeup.title }} - {{ writeup.challenge_title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% else %}
- [{{ writeup.title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% endif %}
{% endfor %}


