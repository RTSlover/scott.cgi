---
layout: default
title: Blog
permalink: /blog/
lang: zh-cn
---

<h1>一些随笔</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    </li>
  {% endfor %}
</ul>
