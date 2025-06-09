---
layout: page
title: Tags
permalink: /tags/
---

# All Tags

{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtags = tags | split:' ' | sort | reverse %}

<div class="tag-cloud">
{% for tag in sortedtags %}
  {% assign tagitems = tag | split: '#' %}
  {% capture tagname %}{{ tagitems[1] }}{% endcapture %}
  {% capture tagcount %}{{ tagitems[2] }}{% endcapture %}
  
  <span class="tag-item">
    <a href="#{{ tagname | slugify }}" class="tag-link" data-count="{{ tagcount }}">
      {{ tagname }} ({{ tagcount }})
    </a>
  </span>
{% endfor %}
</div>

---

## Posts by Tag

{% for tag in site.tags %}
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

<style>
.tag-cloud {
  margin: 20px 0;
  line-height: 2;
}

.tag-item {
  display: inline-block;
  margin: 5px;
}

.tag-link {
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 4px;
  text-decoration: none;
  font-size: 0.9em;
  color: #333;
  border: 1px solid #ddd;
  transition: all 0.2s ease;
}

.tag-link:hover {
  background: #e0e0e0;
  border-color: #ccc;
}

.tag-posts {
  list-style: none;
  padding: 0;
  margin-bottom: 30px;
}

.tag-posts li {
  padding: 5px 0;
  border-bottom: 1px solid #eee;
}

.tag-posts li:last-child {
  border-bottom: none;
}

.post-date {
  float: right;
  color: #666;
  font-size: 0.9em;
}

@media (max-width: 600px) {
  .post-date {
    float: none;
    display: block;
    font-size: 0.8em;
  }
}
</style>
