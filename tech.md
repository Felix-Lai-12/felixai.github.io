---
layout: page
title: 技術
permalink: /tech/
---

{% assign tech_posts = site.posts | where_exp: "post", "post.categories contains 'tech'" %}
{% for post in tech_posts %}
<article>
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <time>{{ post.date | date: "%Y-%m-%d" }}</time>
  <p>{{ post.description }}</p>
</article>
{% endfor %}