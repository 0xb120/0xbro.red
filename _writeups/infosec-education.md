---
layout: default
title:  "InfoSec Education"
date: 2021-04-30
has_children: true
child_nav_order: reversed
nav_order: 15
has_toc: false
---

# InfoSec Education
**Videos and articles** about **general InfoSec** and **Cyber Security**.
I hope you will find them useful. In case of advice, feel free to contact me.

{% assign filtered_posts = site.writeups | where: "parent", "InfoSec Education" | sort: "date" | reverse %}
{% for writeup in filtered_posts %}
- [{{ writeup.title }}]({{ writeup.url }})<br>
*{{writeup.date | date: "%Y, %-d %B" }}*
{% endfor %}


