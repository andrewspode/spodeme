---
layout: default
title: Articles
---

{% for article in site.articles reversed %}
  <div class="article-preview">
    <h2><a href="{{ article.url | remove: '/index' }}">{{ article.title }}</a></h2>
    <time datetime="{{ article.date | date_to_xmlschema }}">
      {{ article.date | date: "%B %d, %Y" }}
    </time>
    
    {% if article.tags %}
      <div class="tags">
        {% for tag in article.tags %}
          <a class="tag">{{ tag }}</a>
        {% endfor %}
      </div>
    {% endif %}

    {% if article.description %}
      <p class="description">{{ article.description }}</p>
    {% endif %}
  </div>
{% endfor %}