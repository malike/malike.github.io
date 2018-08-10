---
layout: section
title: Projects
---


{::nomarkdown}
{% for post in site.posts %}
{% if post.projectname %}
<div class="card">
  <a href="{{ post.projecturl }}">  
  <img src="http://via.placeholder.com/220x220?text=Logo">
  <div class="card-container">
    <h4><b>{{ post.projectname }}</b></h4> 
    <div class="description">{{ post.projectdescription }}</div > 
  </div>
  </a>
</div>
{% endif %}
{% endfor %}
{:/}

<p style="clear:both">&nbsp;</p>