---
layout: page
title: 技術
permalink: /tech/
---

{% assign tech_posts = site.posts | where_exp: "post", "post.categories contains 'tech'" %}
{% for post in tech_posts %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}