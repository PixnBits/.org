---
title: Articles
layout: default
---

<h3>Articles</h3>
<ul>
  {% for post in site.posts limit: 20 %}
  <li>
    <h4>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
      <time datetime="{{ post.date | date: "%Y-%m-%d" }}" itemprop="datePublished">
        ({{ post.date | date: "%Y/%-m/%-d" }})
      </time>
    </h4>

    <summary>
      <p>
        {% if post.summary %}
          {{ post.summary }}
        {% else %}
          {{ post.excerpt }}
        {% endif %}
      </p>
    </summary>
  </li>
  {% endfor %}
</ul>
