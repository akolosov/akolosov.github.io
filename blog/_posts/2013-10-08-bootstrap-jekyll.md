---
layout: post
title:  "And so, let's start!"
date:   2013-10-07 16:00:25
tags:
- jekyll
- bootstrap
- blog
---
I decided to try [Bootstrap 3.0](http://getbootstrap.com/). All the more so with the version [2.3](http://getbootstrap.com/2.3.2/) I have long been on the ["hey you"](http://akolosov.github.io/houston/). At the same time decided to touch [Jekyll](http://jekyllrb.com), had long dreamed of. The beauty of Jekyll that his "out of the box" support [GitHub Pages](http://pages.github.com/). That is, there is no need to search for and set up hosting for simple and cozy blog.
Of course the need to blog I was not, but I'm used to study the technology in specific working examples. As they say "huyak-huyak in production"! :) So here I am with some regularity (yep!) will write notes about what I'm studying, I would like to study or have studied. Sometimes (almost always) I will be frank nonsense and heresy. So welcome to the comments. I am always happy to argue with the "trolls." In general, I'm always happy to chat with interesting people.

{% highlight ruby %}
	
	posts = Blog.all.order('published_at DESC')

	posts.each do |post|
		post.read
		post.comment
	end
	
{% endhighlight %}
