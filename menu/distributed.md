---
layout: section
title: Cloud Native / Distributed Systems
---

{::nomarkdown}

{% for post in site.posts %}
{% if post.section contains "distributed" %}
<div class="posts">
  <h1>
    <a href="{{ site.github.url }}{{ post.url }}">{{ post.title }}</a>
  </h1>
  {% if post.image %}
  <div class="thumbnail-container">
    <a href="{{ site.github.url }}{{ post.url }}"><img style="min-width:100%;max-height:400px;" src="{{ site.github.url }}/assets/img/{{ post.image }}"></a>
  </div>
  {% endif %}
  <p>
    {{ post.content | strip_html | truncate: 350 }} <a href="{{ site.github.url }}{{ post.url }}">Read more</a>
    <span class="post-date"><i class="fa fa-calendar" aria-hidden="true"></i> {{ post.date | date_to_string }} - <i class="fa fa-clock-o" aria-hidden="true"></i> {% include read-time.html %}</span>
  </p>
</div>
{% endif %}
{% endfor %}
{:/}
