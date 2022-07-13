---
title: "Algorithm"
layout: archive
permalink: devlog/algorithm
author_profile: true
sidebar_nav: true
---

***
{% assign posts = site.categories.algorithm %}

{% for post in posts %}
    {% include archive-single2.html type=page.entries_layout %}
{% endfor %}