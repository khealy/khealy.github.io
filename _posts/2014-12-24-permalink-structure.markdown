---
layout: post
title:  "Permalink Structure"
date:   2014-12-24
categories: jekyll
---
The default permalink structure for Jekyll is really long:

> /:categories/:year/:month/:day/:title.html (results in /general/2014/12/23/permalink-structure.html)

"Pretty" permalinks aren't much shorter - only the file extension is removed:

> /:categories/:year/:month/:day/:title (results in /general/2014/12/23/permalink-structure)

Fortunately, permalinks are easily customizable. 

For this site, I've decided that I want the link structure to be:

> /posts/:year/:month/:title (results in /posts/2014/12/permalink-structure)

Setting this up is simple. All that's needed is a single link of code in _config.yml:

> permalink: /posts/:year/:month/:title

TADA!

