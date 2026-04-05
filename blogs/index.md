---
layout: page
title: Blog
---

Posts live under `_posts/game-engineering/`, `_posts/game-projects/`, etc. URLs look like `/blogs/game-engineering/YYYY/MM/DD/slug/` (the first path segment matches that folder name).

{% for post in site.posts %}
<div class="card" style="margin: 12px 0;">
  <div style="color: var(--muted); font-size: 14px;">{{ post.date | date: "%b %d, %Y" }}</div>
  <div style="font-size: 18px; font-weight: 650;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></div>
  {%- if post.excerpt -%}
    <div style="margin-top: 6px; color: var(--muted);">{{ post.excerpt | strip_html | truncate: 180 }}</div>
  {%- endif -%}
</div>
{% endfor %}
