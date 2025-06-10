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
  background: linear-gradient(135deg, #fef7f9 0%, #fdf2f5 100%);
  border-radius: 0.5rem;
  border-left: 4px solid #de5684;
  box-shadow: 0 2px 4px rgba(222, 86, 132, 0.1);
  transition: all 0.3s ease;
}

.archive-item:hover {
  background: linear-gradient(135deg, #fdf2f5 0%, #fcecf1 100%);
  box-shadow: 0 4px 8px rgba(222, 86, 132, 0.15);
  transform: translateY(-1px);
}

.archive-link {
  color: #de5684;
  text-decoration: none;
  font-weight: 600;
  font-size: 1.1rem;
  transition: color 0.3s ease;
}

.archive-link:hover {
  color: #c94570;
  text-decoration: underline;
}

.archive-date {
  color: #8a4f60;
  font-size: 0.9rem;
  display: block;
  margin-top: 0.25rem;
  font-weight: 500;
}

h2 {
  color: #de5684;
  border-bottom: 3px solid #de5684;
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
  background: linear-gradient(90deg, #de5684 0%, #f8b2cc 100%);
}

/* Add subtle pink accent to the page */
#markdown-toc {
  background: linear-gradient(135deg, #fef7f9 0%, #fdf2f5 100%);
  border: 1px solid #de5684;
  border-radius: 0.5rem;
  padding: 1rem;
}
</style>
