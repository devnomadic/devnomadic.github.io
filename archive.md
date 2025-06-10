---
layout: page
title: Archive
permalink: /archive/
---

# Post Archive

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year_group in posts_by_year %}
  <h2>{{ year_group.name }}</h2>
  <ul class="archive-list">
    {% for post in year_group.items %}
      <li class="archive-item">
        <a href="{{ post.url }}" class="archive-link">{{ post.title }}</a>
        <time class="archive-date">{{ post.date | date: site.dash.date_format }}</time>
      </li>
    {% endfor %}
  </ul>
{% endfor %}

<style>
.archive-list {
  list-style: none;
  padding: 0;
  margin: 0 0 2rem 0;
}

.archive-item {
  margin: 0.75rem 0;
  padding: 0.75rem;
  background-color: #f8f9fa;
  border-radius: 0.25rem;
  border-left: 3px solid #de5684;
}

.archive-link {
  color: #de5684;
  text-decoration: none;
  font-weight: 500;
  font-size: 1.1rem;
}

.archive-link:hover {
  text-decoration: underline;
}

.archive-date {
  color: #5f6368;
  font-size: 0.9rem;
  display: block;
  margin-top: 0.25rem;
}

h2 {
  color: #de5684;
  border-bottom: 2px solid #de5684;
  padding-bottom: 0.5rem;
  margin: 2rem 0 1rem 0;
}
</style>
