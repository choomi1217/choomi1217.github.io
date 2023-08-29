---
title: "real-world"
layout: archive
permalink: categories/real-world
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['real-world'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
