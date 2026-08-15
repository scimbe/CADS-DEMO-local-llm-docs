---
title: Explanation
description: Understand why this is built this way.
permalink: /explanation/
---

# Explanation

<div class="index-list">
{% for p in site.explanation %}
  <a class="index-item" href="{{ p.url | relative_url }}">
    <strong>{{ p.title }}</strong>
    {% if p.description %}<span>{{ p.description }}</span>{% endif %}
  </a>
{% endfor %}
</div>
