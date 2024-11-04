---
layout: default
title:  "Web Hacking"
date: 2021-04-30
has_children: true
child_nav_order: reversed
nav_order: 5
has_toc: false
---

# Web Hacking 
**Videos and writeups** about **web security** and **web challenges**.
I hope you will find them useful. In case of advice, feel free to contact me.

{% assign filtered_writeups = site.writeups | where: "parent", "Web Hacking" | sort: "date" | reverse %}
{% for writeup in filtered_writeups %}
{% if writeup.challenge_title and writeup.platform %}
- [{{ writeup.title }} - {{ writeup.challenge_title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% elsif writeup.platform %}
- [{{ writeup.title }} \[{{writeup.platform}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% elsif writeup.subtitle %}
- [{{ writeup.title }} - {{writeup.subtitle}}]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% else %}
- [{{ writeup.title }}]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% endif %}
{% endfor %}

