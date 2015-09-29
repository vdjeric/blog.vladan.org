---
author: Vladan
comments: true
date: 2015-09-28
layout: post
title: 'New Jekyll blog'
---
I moved my blog from [Mozilla.com-hosted WordPress](https://blog.mozilla.com/vdjeric) and turned it into a statically-generated [Jekyll](http://jekyllrb.com/) blog hosted on [GitHub Pages](https://pages.github.com/). I did it partly out of interest in Jekyll and partly because of limitations imposed by Mozilla's WordPress deployment (e.g. [bug 1197542](https://bugzilla.mozilla.org/show_bug.cgi?id=1197542)).

I managed to migrate all my posts and even readers' comments(!) from my old blog. The migration turned out to be a bit of a challenge -- I assumed that once I had my data out of WordPress and converted into Markdown blog posts and [Disqus](https://help.disqus.com/customer/portal/articles/466179-what-is-disqus-) comments, that the rest would be simple.

Of course, I could have just started an empty new blog, but I was curious about Ruby and Jekyll and I figured this would be a good learning exercise.

If anyone is interested in migrating their blog from WordPress to Jekyll or Octopress, this is what I did:

1. First, I exported my WordPress blog (posts and comments) to an XML file using the built-in export feature
1. I used [exitwp](https://github.com/thomasf/exitwp) to convert the posts in the WordPress XML file to Markdown files. Exitwp is far from perfect, so I had to fix up posts by hand
1. I then imported the comments from the XML file into Disqus, but first I had to manually edit the XML file to create associations between the old comments and the posts on the new site
1. I also had to migrate parts of the old WordPress blog's theme CSS to the new blog to fix display issues with old posts. I chose not to customize the blog's default Jekyll theme
1. Further fixed up the posts to correct links etc
1. Lots of debugging and learning of Jekyll, Ruby package management (Gem and Bundler), SCSS, GitHub Pages, Disqus quirks, my DNS provider, etc :)

I think the process would have been simpler if I had admin privileges on the WordPress deployment -- I came across WordPress plugins and scripts for exporting that needed admin rights to deploy. This would have made the migration more automatic.

I also looked into [Octopress](http://octopress.org/docs/). I didn't like Octopress 2 ([for these kinds of reasons](http://octopress.org/2015/01/15/octopress-3.0-is-coming/)) and I found Octopress 3 unfinished & not well documented, which is understandable, since it's not officially released yet.

Jekyll and GitHub Pages aren't perfect either (Jekyll can take a while to re-generate a simple site, GH Pages produces unhelpful "Page build failed" errors, etc), but this migration has been a different kind of challenge and I enjoyed figuring out all the little bugs that popped up.
