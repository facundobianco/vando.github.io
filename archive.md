---
layout: page
title: Archive
permalink: /archive/
---

{% for post in site.posts %}
  * {{ post.date | date_to_string }} <i class="fa fa-angle-right fa-fw"></i>[ {{ post.title }} ]({{ post.url }})
{% endfor %}
