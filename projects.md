---
layout: page
title: "Projects"
permalink: /projects/
---

# Projects

<ul>
  {% for project in site.projects %}
    <li>
      <a href="{{ project.url | relative_url }}">{{ project.title }}</a> - {{ project.description }}
      <br>
      <small>{{ project.date | date: "%Y.%m.%d" }}</small>
    </li>
  {% endfor %}
</ul>
