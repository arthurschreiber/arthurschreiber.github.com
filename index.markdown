---
layout: default
title: "Tom Ward's blog"
description: "Personal blog of Tom Ward, in which he writes about ruby, rails and web development, as well as other random ephemera"
---
<h1 class="title">Blog Posts</h1>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>