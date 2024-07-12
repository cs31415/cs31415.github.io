---
layout: default
title: Blog
---

<div>
{% assign sortedPosts = site.posts | sort: 'date' %}
{% assign postsByYear = sortedPosts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in postsByYear reversed %}
      {% for post in year.items reversed %}
        <h3>
            <a href="{{ post.url }}"><span class="postdate">{{ post.date  | date_to_string }}</span>{{ post.title }}</a>
        </h3>
      {% endfor %}
{% endfor %}
</div>