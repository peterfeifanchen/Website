---
layout: page
title: Blog
category: general
permalink: /blog/
---

<hr>

<div class="post-section">
  {% assign cat = page.category %}
  <ul class="post-list-section">
  	{% for post in site.categories[cat] %}
	   <a class="post-list-title" href="{{site.baseurl}}{{post.url}}">
	   <li>
	     {{post.title}}
		 <small class="post-list-date">{{ post.date | date_to_string}}</small>
		 {% if post.tags[0] == 'Dabbling' %}
		 <small class="post-tags-blue">{{ post.tags }} </small>
		 {% elsif post.tags[0] == 'Paper Review' %}
		 <small class="post-tags-orange">{{ post.tags }} </small>
		 {% endif %}
		 </li>
	   </a>
	{% endfor %}
  </ul>
</div>
  	

