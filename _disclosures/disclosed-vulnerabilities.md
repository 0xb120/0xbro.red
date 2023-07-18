---
layout: default
title:  "Disclosed vulnerabilities"
has_children: true
child_nav_order: reversed
nav_order: 2
has_toc: false
---

# Disclosed vulnerabilities
The following is a list of **publicly disclosed vulnerabilities** discovered and released according to the documented <a href="{{ site.url }}/disclosures/policy/">disclosure policy</a>.


{% assign filtered_articles = site.disclosures | where: "parent", "Disclosed vulnerabilities" | sort: "date" | reverse %}
{% for art in filtered_articles %}
{% if art.subtitle %}
- [{{ art.title }} - {{ art.subtitle}}]({{ art.url }})<br>
*{{art.date | date: "%Y, %-d %B" }}*
{% else %}
- [{{ art.title }}]({{ art.url }})<br>
*{{art.date | date: "%Y, %-d %B" }}*
{% endif %}
{% endfor %}