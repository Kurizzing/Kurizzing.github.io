---
title: "Web1"
layout: archive
permalink: categories/web1
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.web1 %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}