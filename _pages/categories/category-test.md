---
title: "Test"
layout: archive
permalink: categories/test
author_profile: true
sidebar_main: true
toc: true
toc_sticky: true
toc_label: "hello"
---


{% assign posts = site.categories.web1 %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}