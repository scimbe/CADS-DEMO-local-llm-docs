---
title: Reference
description: Look up an exact fact.
permalink: /reference/
---

# Reference

<div class="index-list">
{% for p in site.reference %}
  <a class="index-item" href="{{ p.url | relative_url }}">
    <strong>{{ p.title }}</strong>
    {% if p.description %}<span>{{ p.description }}</span>{% endif %}
  </a>
{% endfor %}
</div>
