---
title: "Algorithm"
layout: archive
permalink: devlog/graphics
author_profile: true
sidebar_nav: true
---

***
{% assign posts = site.categories.graphics %}

{% for post in posts %}
    {% include archive-single2.html type=page.entries_layout %}
{% endfor %}