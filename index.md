---
layout: default
title: Blog about 3D and development
categories: en
---

<div class="topnavbar">
<header class="content-header">
  <table style="width:100%; height:90px"><tr>
  <td style="width:100%">
  <p>Unlimited3d - {{page.title}}</p>
  <p style='font-size: 0.6em;'>🏷️ Tag filter:&nbsp;
  <a href="{{site.baseurl}}/">&#42;</a>
  </p>
  </td>
  </tr></table>
</header>
</div>

<div class="page-content inset">
<link rel="stylesheet" href="{{site.baseurl}}/css/code-highlight-molokai.css" />

{% assign posts = site.posts | sort: 'date' | reverse %}
{% for post in posts %}
  {% include post_excerpt.html %}
{% endfor %}

</div> <!-- page-content-inset -->
