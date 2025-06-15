---
layout: page
title: Tags
permalink: /tags/
---

# All Tags

{% assign tags = site.tags | sort %}
<div class="tag-cloud">
{% for tag in tags %}
  {% assign tagname = tag[0] %}
  {% assign posts = tag[1] %}
  <a href="#{{ tagname | slugify }}" class="tag" data-count="{{ posts.size }}">
    {{ tagname }} ({{ posts.size }})
  </a>
{% endfor %}
</div>

---

## Posts by Tag

{% for tag in tags %}
  {% assign tagname = tag[0] %}
  {% assign posts = tag[1] %}
  
<h3 id="{{ tagname | slugify }}">{{ tagname }}</h3>
<ul class="tag-posts">
  {% for post in posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span class="post-date">{{ post.date | date: site.dash.date_format }}</span>
    </li>
  {% endfor %}
</ul>
{% endfor %}

<!-- Tag styling is now handled in assets/css/custom.css -->