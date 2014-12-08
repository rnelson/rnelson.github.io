---
layout: page
title: blog
tagline: ""
---
{% include JB/setup %}

## About

I keep starting blogs, but instead of adding new technical posts when I come up 
with one, I inevitably start filling them with personal posts to keep the site
active. A year or so later, I get tired of having so few technical posts and 
kill the blog.

Instead of doing that, I am going to keep this one technical. If no new posts
are added in a couple of years, so be it.

### About Me

I'm a developer living in Omaha, NE. [My day job](http://hobbytown.com) 
involves working with both Visual Basic 6 and the modern .NET stack, but for 
personal projects I work with whatever looks fun at the time. [Most of my 
projects](https://github.com/rnelson) are in Python or Ruby these days.
    
## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
