---
layout: post
title: "Octopress syntax highlighting issue on Arch Linux"
date: 2014-01-27 00:39
comments: true
tags: [archlinux, octopress]
---
If you run Arch Linux and are trying to run `rake generate` on a Octopress blog, with a post that uses some code and syntax highlighting markdown over it; you will probably see this error:

    /plugins/pygments_code.rb:27:in `rescue in pygments': Pygments can't parse unknown language: ruby. (RuntimeError)

This happens because Arch uses `python3` as default python, while the world hasn't yet made up their mind to switch. The *Pygments* gem that Octopress uses, uses `python2` syntax which Arch's `python3` doesn't undertand fully and hence raises an error. To fix this, just open this file:

{% highlight bash %}
~/.rbenv/versions/2.1.0/lib/ruby/gems/2.1.0/gems/pygments.rb-0.3.4/lib/pygments/mentos.py  
{% endhighlight %}

Use your own version, instead of `2.1.0` that I am using. Change the first line:

{% highlight bash %}
#!/usr/bin/env python 
{% endhighlight %}

to

{% highlight bash %}
#!/usr/bin/env python2 
{% endhighlight %}

After this, your `rake generate` should succeed. Oh, and yes, this file path is for `rbenv`. If you use `rvm` figure out your own path using the command `find ~/ -name mentos.py`.
