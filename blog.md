---
layout: default
title: Blog
permalink: /blog/
---

# Categories Menu
{% for category in categories %}
- [{{ category | capitalize }}](#{{ category }})
{% endfor %}

# Blog Posts

{% assign categories = site.posts | map: "categories" | uniq %}
{% for category in categories %}
## {{ category | capitalize }}

{% for post in site.posts %}
  {% if post.categories == category %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
  {% endif %}
{% endfor %}

{% endfor %}

