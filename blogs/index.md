---
layout: page
title: Blog
---

Posts will appear here once we add them under `_posts/` (they’ll have URLs like `/blogs/YYYY/MM/DD/slug/`).

{% for post in site.posts %}
<div class="card" style="margin: 12px 0;">
  <div style="color: var(--muted); font-size: 14px;">{{ post.date | date: "%b %d, %Y" }}</div>
  <div style="font-size: 18px; font-weight: 650;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></div>
  {%- if post.excerpt -%}
    <div style="margin-top: 6px; color: var(--muted);">{{ post.excerpt | strip_html | truncate: 180 }}</div>
  {%- endif -%}
</div>
{% endfor %}
