---
layout: page
title: blog
tagline: ""
---
{% include JB/setup %}

<ul class="posts">
    <li><span>08 Dec 2014</span> &raquo; <a href="/about.html">About</a></li>
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
