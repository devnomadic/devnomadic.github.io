---
layout: default
title: Staging Area
permalink: /staging/
---

# 🚧 Staging Area

This is where posts are staged for preview before going live.

<div class="staging-info">
  <span class="badge staging-badge">STAGING</span>
  <span>Preview posts appear here before publication</span>
</div>

## Staged Posts

<div class="post-links">
  {% assign staging_posts = site.staging | sort: 'date' | reverse %}
  {% if staging_posts.size > 0 %}
    {% for post in staging_posts %}
      <div class="post-link-wrapper">
        <a href="{{ post.url | relative_url }}" class="post-link">{{ post.title }}</a>
        <div class="post-meta">
          <span class="badge staging-badge">STAGING</span>
          {% if site.dash.date_format %}
            {{ post.date | date: site.dash.date_format }}
          {% else %}
            {{ post.date | date: "%b %-d, %Y" }}
          {% endif %}
          {% if post.tags and post.tags.size > 0 %}
            <div class="post-tags">
              {% for tag in post.tags %}
                <a class="tag" href="/tags#{{ tag | slugify }}">{{ tag }}</a>
              {% endfor %}
            </div>
          {% endif %}
        </div>
      </div>
    {% endfor %}
  {% else %}
    <p>No posts currently staged for preview.</p>
  {% endif %}
</div>

## Navigation

- [← Back to main site](/)
- [About](/about/)
- [Archive](/archive/)

<style>
.staging-badge {
  background: #ff6b6b;
  color: white;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 0.8em;
  margin-right: 8px;
  font-weight: bold;
}

.staging-info {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  border-radius: 5px;
  padding: 10px;
  margin: 20px 0;
}

.post-link-wrapper {
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.post-link-wrapper:last-child {
  border-bottom: none;
}

.post-tags {
  margin-top: 5px;
}

.tag {
  font-size: 0.8em;
  background: #f8f9fa;
  padding: 2px 6px;
  border-radius: 3px;
  text-decoration: none;
  margin-right: 5px;
  color: #666;
}

.tag:hover {
  background: #e9ecef;
}
</style>
