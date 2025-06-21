---
title: "project"
permalink: /tags/project
layout: archive
---

{% assign posts = site.tags.['project'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
