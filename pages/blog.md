---
layout: default
title: Blog
permalink: /blog/
lang: zh-cn
---

<h2>一些随笔</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <h3><a href="{{ post.url }}">{{ post.date | date: "%Y-%m-%d" }} - {{ post.title }}</a></h3>
    </li>
  {% endfor %}
</ul>
