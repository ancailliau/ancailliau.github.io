---
layout: page
title: Tags
---

{% for tag in site.tags %}
  <h2>{{ tag[0] }}</h2>
  <ul class="none">
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}