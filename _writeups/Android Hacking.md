---
layout: default
title:  "Android Hacking"
date: 2021-04-30
has_children: true
child_nav_order: reversed
nav_order: 10
has_toc: false
---

# Android Hacking 
**Videos and writeups** about **Android hacking** and **Android application security**.
I hope you will find them useful. In case of advice, feel free to contact me.

{% include HTB_Card.html %}

{% assign filtered_writeups = site.writeups | where: "parent", "Android Hacking" | sort: "date" | reverse %}
{% for writeup in filtered_writeups %}
{% if writeup.challenge_title and writeup.platform %}
- [{{ writeup.title }} - {{ writeup.challenge_title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% elsif writeup.platform %}
- [{{ writeup.title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% else %}
- [{{ writeup.title }}]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% endif %}
{% endfor %}


