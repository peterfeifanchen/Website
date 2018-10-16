---
layout: page
title: Slytherin Thoughts
permalink: /slytherinthoughts/
---
<!--
<div class="post-section"></div>
As one of my friend once put it, this blog of my running stream of consciousness may contain 
dissident ideas that may potentially burn most of my social capital with Gryffindors, Ravenclaws
and Hufflepuffs. However, I believe dissident ideas are important. If certain posts make you 
angry, instead of spreading that anger, I invite you to ask yourself why. Since the internet
etches capricious and simplistic idealisms of youth permanently into its darkest recesses 
at the time of your reading, I may or may not still believe these words to be true due to the 
natural process of personal growth and maturity. 

<span style="color:red">Warning:</span> I do not believe I am especially gifted in any of these 
topics. Expert opinions should always be sourced from those who do not have to write their own 
Wikipedia pages.
-->
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
