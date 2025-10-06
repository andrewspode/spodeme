---
layout: default
title: Games
---

{% for game in site.games reversed %}
  {% unless game.exclude_from_listing %}
  <div class="article-preview">
    {% if game.image %}
      <a href="{{ game.url }}">
        <img src="{{ game.image }}" alt="{{ game.title }}" class="article-thumbnail">
      </a>
    {% endif %}
    <div class="article-content">
      <h2><a href="{{ game.url }}">{{ game.title }}</a></h2>

    {% if game.tags %}
      <div class="tags">
        {% for tag in game.tags %}
          <a class="tag">{{ tag }}</a>
        {% endfor %}
      </div>
    {% endif %}

    {% if game.description %}
      <p class="description">{{ game.description }}</p>
    {% endif %}
    </div>
  </div>
  {% endunless %}
{% endfor %}