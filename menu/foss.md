---
layout: section
title: Open Source Software
---
{::nomarkdown}
{% for post in site.posts %}
{% if post.fossname %}
<div class="card">
  <a href="{{ post.fossurl }}">  
  <img src="{{ post.fossimage }}" style="width: 220px">
  <div class="card-container">
    <h4><b>{{ post.fossname }}</b></h4> 
    <div class="description">{{ post.fossdescription }}</div> 
  </div>
  </a>
</div>
{% endif %}
{% endfor %}
{:/}

<p style="clear:both">&nbsp;</p>