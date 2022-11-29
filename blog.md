---
layout: main
title: "Posts"
---

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        <h3>{{ post.title }}</h3>
        <p>{{ post.description }}</p>
      </a>
    </li>
  {% endfor %}
</ul>
