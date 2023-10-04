---
title: "좋은 코드, 나쁜 코드"
layout: archive
permalink: categories/goodcode-badcode
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['goodcode-badcode'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}


