---
layout: archive
title: "站点地图"
permalink: /sitemap/
author_profile: true
---

{% include base_path %}

## 主要页面

<ul>
{% for link in site.data.navigation.main %}
  <li><a href="{{ base_path }}{{ link.url }}">{{ link.title }}</a></li>
{% endfor %}
</ul>

{% assign visible_posts = site.posts | where_exp: "post", "post.published != false" %}
{% if visible_posts.size > 0 %}
## 博客文章

<ul>
{% for post in visible_posts %}
  <li><a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
{% endif %}

{% assign visible_publications = site.publications | where_exp: "post", "post.published != false" %}
{% if visible_publications.size > 0 %}
## 论文

<ul>
{% for post in visible_publications reversed %}
  <li><a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
{% endif %}

{% assign visible_portfolio = site.portfolio | where_exp: "post", "post.published != false" %}
{% if visible_portfolio.size > 0 %}
## 项目

<ul>
{% for post in visible_portfolio %}
  <li><a href="{{ base_path }}{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
{% endif %}
