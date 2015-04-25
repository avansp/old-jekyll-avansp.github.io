---
layout: post
title:  "A blog is born"
date:   2014-10-09
tags: [en]
comments: true
---

A new adventure in blogging. Locally created using Jekyll and is hosted by GitHub. A geeky way to blog, but I like it. More freedom with a lot of experimental feature opportunities.

If you want to try yourself, here's a really simple way to setup. First, check that you have [RubyGems][ruby-gem] package management installed in your system. Then, open your terminal and fire these commands:

{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

Go to a folder where you want to create your blog sites, say the folder is `sites` and you want to create a site call `myblog`.
{% highlight bash %}
$ cd sites
$ jekyll new myblog
$ cd myblog
$ jekyll serve --watch
{% endhighlight %}

Open `http://localhost:4000` in your browser, and *voila* ! You've got a new blog.

Look into the new `myblog` folder to see what happens. For the rest, you just need a text editor, but please do try [Atom][atom-site] - I really enjoy this hackable text editor. The neat thing is your edits are alive instantly. Everytime you save your text, the jekyll service updates it immediately.

Want to publish your blog? Try using [GitHub pages][github-page].

This post was created using the default Jekyll theme.

[ruby-gem]: http://guides.rubygems.org
[atom-site]: https://atom.io
[github-page]: https://pages.github.com
