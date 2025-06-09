---
layout: home
---

# Welcome to devnomadic

A software engineer's journey through remote work, digital nomadism, and technology. Here I share insights on building software while traveling the world, tips for working remotely, and thoughts on the intersection of technology and lifestyle.

## Recent Posts

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: site.dash.date_format }}
{% endfor %}

---

*Follow along as I document the adventures of being a digital nomad in the tech world.*
