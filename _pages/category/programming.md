---
title: "Programming"
permalink: /programming
layout: archive
---

{% assign posts = site.categories.['Programming'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}