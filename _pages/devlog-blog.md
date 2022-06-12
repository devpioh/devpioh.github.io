---
title: "Blog"
layout: archive
permalink: devlog/blog
author_profile: true
sidebar_nav: true
---

***
{% assign posts = site.categories.blog %}

{% for post in posts %}
    {% include archive-single2.html type=page.entries_layout %}
{% endfor %}