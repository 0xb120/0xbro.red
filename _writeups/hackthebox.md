---
layout: default
title:  HackTheBox
date: 2021-04-30
has_children: true
child_nav_order: reversed
nav_order: 99
has_toc: false
---

# HackTheBox (B2R) writeups
**Writeups** and tutorials for different HackTheBox **boot2root** machines.
I hope you will find them useful. In case of advice, feel free to contact me.

{% include HTB_Card.html %}


{% assign filtered_writeups = site.writeups | where: "parent", "HackTheBox" | sort: "date" | reverse %}
{% for writeup in filtered_writeups%}
- [{{ writeup.title}} \[{{writeup.difficulty}}\]]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% endfor %}