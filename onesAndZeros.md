---
layout: page
title: Ones and Zeros
category: systems
permalink: /onesandzeros/
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
	   </li>
	   </a>
	{% endfor %}
  </ul>
</div>
  	

