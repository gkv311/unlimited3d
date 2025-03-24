---
layout: post
title: Blog
categories: en
permalink: /blog/
---

{% for post in site.posts %}
  {% if    post.tags       contains 'draft' %}
  {% elsif post.categories contains 'blog' %}
  <h2 class='indent'><a href="{{site.baseurl}}{{post.url}}">{{post.title}}</a></h2>
  <summary>{{post.excerpt}}</summary>
  {% endif %}
{% endfor %}
