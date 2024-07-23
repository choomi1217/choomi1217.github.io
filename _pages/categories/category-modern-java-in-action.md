---
title: "modern java in action"
layout: archive
permalink: categories/modern-java-in-action
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['modern-java-in-action'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
