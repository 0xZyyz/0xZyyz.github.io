---
layout: default
title: Writeups
---

# Writeups

Research notes, CTF walkthroughs, and lessons from authorized security testing.

{% if site.posts.size > 0 %}
<ul>
  {% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small>— {{ post.date | date: "%Y-%m-%d" }}</small>
    {% if post.excerpt %}<br>{{ post.excerpt | strip_html | truncatewords: 30 }}{% endif %}
  </li>
  {% endfor %}
</ul>
{% else %}
_No posts yet — watch this space._
{% endif %}

[← Back to the home page]({{ '/' | relative_url }})
