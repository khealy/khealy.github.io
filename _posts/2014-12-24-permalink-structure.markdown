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

Seems like overkill for this site. I probably won't be creating multiple posts per day, so I don't think I'll need a day in my file structure. And I don't like the idea of having categories as top-level directories.


Fortunately, Jekyll makes it easy to customize permalinks. 

##Creating a custom permalink style

For this site, I've decided that I want to organize all of my posts into a "post" directory, with subdirectories for year, then date.

Setting this up is simple. All that's needed is a single line of code in _config.yml:

*permalink: /posts/:year/:month/:title*

You'll notice that all the variables are prepended with a colon. For a full list of variables that you can use, see the [Jekyll permalink docs](http://jekyllrb.com/docs/permalinks/).

