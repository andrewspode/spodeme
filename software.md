---
layout: default
title: Games & Software
---

{% assign all_projects = site.games %}
{% if site.software %}
  {% assign all_projects = all_projects | concat: site.software %}
{% endif %}
{% assign all_projects = all_projects | sort: 'date' | reverse %}
{% for project in all_projects %}
  {% unless project.exclude_from_listing %}
  <div class="article-preview">
    {% if project.image %}
      <a href="{{ project.url }}">
        <img src="{{ project.image }}" alt="{{ project.title }}" class="article-thumbnail">
      </a>
    {% endif %}
    <div class="article-content">
      <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>

    {% if project.tags %}
      <div class="tags">
        {% for tag in project.tags %}
          <a class="tag">{{ tag }}</a>
        {% endfor %}
      </div>
    {% endif %}

    {% if project.description %}
      <p class="description">{{ project.description }}</p>
    {% endif %}
    </div>
  </div>
  {% endunless %}
{% endfor %}
