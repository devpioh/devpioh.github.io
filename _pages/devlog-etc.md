---
title: "기타 잡다하거나 잘 안써서 잊어버리는 것들"
layout: archive
permalink: devlog/etc
author_profile: true
sidebar_nav: true
---

***
{% assign posts = site.categories.etc %}

{% for post in posts %}
    {% include archive-single2.html type=page.entries_layout %}
{% endfor %}