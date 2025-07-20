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
  padding: 1rem;
  background: linear-gradient(135deg, #c94570 0%, #b83d64 100%);
  border-radius: 0.5rem;
  border-left: 4px solid #fef7f9;
  box-shadow: 0 2px 4px rgba(201, 69, 112, 0.1);
  transition: all 0.3s ease;
}

.archive-item:hover {
  background: linear-gradient(135deg, #b83d64 0%, #a73459 100%);
  box-shadow: 0 4px 8px rgba(201, 69, 112, 0.15);
  transform: translateY(-1px);
  border-left: 4px solid #ffffff;
}

.archive-link {
  color: #ffffff;
  text-decoration: none;
  font-weight: 600;
  font-size: 1.1rem;
  transition: color 0.3s ease;
}

.archive-link:hover {
  color: #fef7f9;
  text-decoration: underline;
}

.archive-date {
  color: #fdf2f5;
  font-size: 0.9rem;
  display: block;
  margin-top: 0.25rem;
  font-weight: 500;
}

h2 {
  color: #c94570;
  border-bottom: 3px solid #c94570;
  padding-bottom: 0.75rem;
  margin: 2.5rem 0 1.5rem 0;
  font-weight: 700;
  position: relative;
}

h2::after {
  content: '';
  position: absolute;
  bottom: -3px;
  left: 0;
  width: 50px;
  height: 3px;
  background: linear-gradient(90deg, #c94570 0%, #f8b2cc 100%);
}

/* Add subtle pink accent to the page */
#markdown-toc {
  background: linear-gradient(135deg, #fef7f9 0%, #fdf2f5 100%);
  border: 1px solid #c94570;
  border-radius: 0.5rem;
  padding: 1rem;
}
</style>
