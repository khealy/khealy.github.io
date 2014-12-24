---
layout: post
title:  "Permalink Structure"
date:   2014-12-24
categories: jekyll
---

##Built-in permalink styles
The default permalink structure for Jekyll is really long:

*/:categories/:year/:month/:day/:title.html - which results in /general/2014/12/23/permalink-structure.html*

"Pretty" permalinks aren't much shorter - only the file extension is removed:

*/:categories/:year/:month/:day/:title - which results in /general/2014/12/23/permalink-structure*

Seems like overkill for me - I can't see creating so many posts a day that I'd really need to have a day in my file structure. And I don't like the idea of having posts separated into directories by category.


Fortunately, Jekyll makes it easy to customize permalinks. 

##Creating a custom permalink style

For this site, I've decided that I want to organize all of my posts into a "post" directory, with subdirectories for year, then date.

Setting this up is simple. All that's needed is a single line of code in _config.yml:

*permalink: /posts/:year/:month/:title*

TADA!

