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
/* Tag cloud styling - matches home page exactly with important overrides */
.tag-cloud {
  margin: 20px 0 !important;
  line-height: 2 !important;
}

.tag-item {
  display: inline-block !important;
  margin: 5px !important;
}

/* Use exact same styling as home page post-tags - override theme completely */
.tag-cloud .tag,
.tag-cloud a,
.tag-cloud a.tag {
  display: inline-block !important;
  background-color: #f1f3f4 !important;
  color: #5f6368 !important;
  padding: 0.2rem 0.5rem !important;
  margin: 0.1rem 0.2rem 0.1rem 0 !important;
  border-radius: 0.25rem !important;
  font-size: 0.8rem !important;
  text-decoration: none !important;
  border: none !important;
}

.tag-cloud .tag:hover,
.tag-cloud a:hover,
.tag-cloud a.tag:hover {
  background-color: #e8eaed !important;
  color: #202124 !important;
  text-decoration: none !important;
}

/* Remove any theme icons or decorations */
.tag-cloud .tag:before,
.tag-cloud a:before,
.tag-cloud a.tag:before {
  display: none !important;
  content: none !important;
}

.tag-posts {
  list-style: none;
  padding: 0;
  margin-bottom: 30px;
}

.tag-posts li {
  margin: 0.5rem 0;
  padding: 0.75rem;
  background-color: #f8f9fa;
  border-radius: 0.25rem;
  border-left: 3px solid #c94570;
}

.tag-posts li a {
  color: #c94570;
  text-decoration: none;
  font-weight: 500;
}

.tag-posts li a:hover {
  text-decoration: underline;
}

.post-date {
  color: #5f6368;
  font-size: 0.9rem;
  display: block;
  margin-top: 0.25rem;
}

@media (max-width: 600px) {
  .post-date {
    font-size: 0.8em;
  }
}
</style>
