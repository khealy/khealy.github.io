---
layout: post
title:  "Category and tag pages"
date:   2014-12-25
category: code
tags: [jekyll, github, liquid]
---

There's a [fantastic post on minddust.com](http://www.minddust.com/post/tags-and-categories-on-github-pages/) that explains how to set up category and tag pages if you're hosting your Jekyll site on GitHub. I used it as the basis for my category and tag pages, with only minor alterations.

I only had one problem&mdash;my site was set up to display all pages in the top navigation section of the site, which makes sense for my "About" page, but I didn't want a bunch of category and tag displaying in the navigation.

When I looked at my default layout file, I saw that the code that generated the top navigation was header.html, and in that file, I spotted the section I needed to modify:

{% raw %} 
	{% for page in site.pages %}
      {% if page.title %}
	    ...
	  {% endif %}
	{% endfor %}
 {% endraw %} 

Since I've set the titles of my category and tag pages to start, respectively, with "Posts by category" and "Posts by tag", it was relatively simple to exclude them using a bit of Liquid template engine markup:

{% raw %} 
	{% for page in site.pages %}
      {% if page.title %}
        {% unless page.title contains 'Posts by' %}
          ...
        {% endunless %}
	  {% endif %}
	{% endfor %}
 {% endraw %} 

 Note the addition of the "unless" block of code.

 Simple once I discovered Liquid's "unless" tag and the "contains" operator.

