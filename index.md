---
layout: post
title: Unlimited3d
categories: en
---

<h2 class='indent'><a href='{{site.baseurl}}/blog'>Blog</a></h2>
{% for post in site.posts %}
  {% if    post.tags       contains 'draft' %}
  {% elsif post.categories contains 'blog' %}
  <h3 class='indent'><a href="{{site.baseurl}}{{post.url}}">{{post.title}}</a></h3>
  <summary>{{post.excerpt}}</summary>
  {% endif %}
{% endfor %}
