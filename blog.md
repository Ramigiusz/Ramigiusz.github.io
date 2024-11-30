---
layout: default
title: Blog
permalink: /blog/
---

# Blog Posts

{% assign categories = site.categories | sort %}

{% for category in categories %}
  <h2>{{ category | capitalize }}</h2>
  <ul>
    {% assign posts_in_category = site.posts | where: "categories", category %}
    {% for post in posts_in_category %}
      <li>
        - [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
      </li>
    {% endfor %}
  </ul>
{% endfor %}

