---
title: "web"
permalink: /tags/web
layout: archive
---

{% assign posts = site.tags.['web'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
