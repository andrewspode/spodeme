---
layout: default
title: Articles
---

{% for article in site.articles reversed %}
  <div class="article-preview">
    {% if article.image %}
      <a href="{{ article.url }}">
        <img src="{{ article.image }}" alt="{{ article.title }}" class="article-thumbnail">
      </a>
    {% endif %}
    <div class="article-content">
      <h2><a href="{{ article.url }}">{{ article.title }}</a></h2>
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
  </div>
{% endfor %}