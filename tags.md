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
  <span class="tag-item">
    <a href="#{{ tagname | slugify }}" class="tag" data-count="{{ posts.size }}">
      {{ tagname }} ({{ posts.size }})
    </a>
  </span>
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

<style>
.tag-cloud {
  margin: 20px 0;
  line-height: 2;
}

.tag-item {
  display: inline-block;
  margin: 5px;
}

.tag-cloud .tag {
  display: inline-block;
  background-color: #f1f3f4;
  color: #5f6368;
  padding: 0.2rem 0.5rem;
  margin: 0.1rem 0.2rem 0.1rem 0;
  border-radius: 0.25rem;
  font-size: 0.8rem;
  text-decoration: none;
}

.tag-cloud .tag:hover {
  background-color: #e8eaed;
  color: #202124;
}

.tag-posts {
  list-style: none;
  padding: 0;
}

.tag-posts li {
  margin: 0.5rem 0;
  padding: 0.5rem;
  background-color: #f8f9fa;
  border-radius: 0.25rem;
}

.tag-posts li a {
  color: #de5684;
  text-decoration: none;
  font-weight: 500;
}

.tag-posts li a:hover {
  text-decoration: underline;
}

.post-date {
  color: #5f6368;
  font-size: 0.9rem;
  margin-left: 1rem;
}
</style>

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
