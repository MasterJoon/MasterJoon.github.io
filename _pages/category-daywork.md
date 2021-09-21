---
title: "Posts by Category"
layout: categories
permalink: /categories/daywork
author_profile: true
---
{% assign posts = site.categories.daywork %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
