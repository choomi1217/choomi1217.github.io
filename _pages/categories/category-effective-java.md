---
title: "이펙티브 자바"
layout: archive
permalink: categories/effective-java
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['Effective Java'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
