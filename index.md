---
layout: default
title: Blog
---

<div>
{% assign postsByYear =
    site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}
{% for year in postsByYear reversed %}
      {% for post in year.items reversed %}
        <h3>
            <a href="{{ post.url }}">{{ post.title }}</a>
        </h3>
      {% endfor %}
{% endfor %}
</div>