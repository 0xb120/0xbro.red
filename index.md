---
layout: home
title: Home
nav_order: 1
---

# Welcome!
Hey! I am Mattia, aka **0xbro**. <br>This is my personal blog related to everything that surrounds ethical hacking, penetration testing, AppSec, CTFs, and other various cybersecurity stuff.<br>
If you want to know more about me or want to get in touch, please visit the <a href="{{ site.url }}/about/">About Me</a> page. 

## Recent writeups
<ul>
  {% assign filtered_writeups = site.writeups | where: "has_children", false | sort: "date" | reverse %}
  {% for writeup in filtered_writeups limit:5%}
    <li>
      <a href="{{ writeup.url }}">{{writeup.parent}} - {{ writeup.title }}</a>
    </li>
  {% endfor %}
</ul>

## Recent disclosures
<ul>
  {% assign filtered_disclosures = site.disclosures | where: "has_children", false | where: "parent", "Disclosed vulnerabilities" | sort: "date" | reverse %}
  {% for art in filtered_disclosures limit:5%}
    <li>
    {% if art.subtitle %}
      <a href="{{ art.url }}">{{art.title}} - {{ art.subtitle }}</a>
    {% else %}
      <a href="{{ art.url }}">{{art.title}}</a>
    {% endif %}
    </li>
  {% endfor %}
</ul>




