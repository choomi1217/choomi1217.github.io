---
title: "JPA 혼자 공부해보기"
layout: archive
permalink: categories/jpa-self-study
author_profile: true
sidebar_main: true
---

***

{% assign posts = site.categories['JPA Self'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
