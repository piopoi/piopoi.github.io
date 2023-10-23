---
title: "Programming"
permalink: /categories/programming
layout: archive
---

{% assign posts = site.categories.['Programming'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}