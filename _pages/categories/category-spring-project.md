---
title: "Toby's Spring"
layout: archive
permalink: categories/spring-project
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['Spring Project'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
