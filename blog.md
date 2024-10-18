---
layout: default
title: "Blog"
---
{% if site.show_excerpts %}
{% else %}
  {% include archive.html title="Posts" %}
{% endif %}
