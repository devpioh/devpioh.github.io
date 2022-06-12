---
title: "Version Control"
layout: archive
permalink: devlog/version-control
author_profile: true
sidebar_nav: true
---

***
{% assign posts = site.categories['Version Control'] %}

{% for post in posts %}
    {% include archive-single2.html type=page.entries_layout %}
{% endfor %}