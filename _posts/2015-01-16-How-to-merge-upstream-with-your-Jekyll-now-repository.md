---
layout: post
title: How to merge upstream with your Jekyll-now repository
tags:
- jekyll
- git
published: true
---

Sometimes there are some updates on the [upstream(original) Jekyll-now repository](https://github.com/barryclark/jekyll-now), and you want to apply those changes to your forked repository.

Here is how to do that:

----

## 0. Add remote url as upstream

{% highlight bash %}
$ git remote add upstream https://github.com/barryclark/jekyll-now.git
{% endhighlight %}

Then you can see the added url:

{% highlight bash %}
$ git remote -v
origin  git@github.com:your-username/your-username.github.io.git (fetch)
origin  git@github.com:your-username/your-username.github.io.git (push)
upstream        https://github.com/barryclark/jekyll-now.git (fetch)
upstream        https://github.com/barryclark/jekyll-now.git (push)
$ 
{% endhighlight %}

## 1. Fetch upstream's changes

{% highlight bash %}
$ git fetch upstream
{% endhighlight %}

## 2. Merge upstream

If there's any change, merge it to your repository.

{% highlight bash %}
$ git merge upstream/master
{% endhighlight %}

When you encounter conflicts, edit those files personally and add/commit them.

## 3. Push to Github

You made changes to your codes, so push them to Github.

{% highlight bash %}
$ git push origin master
{% endhighlight %}

Then check your pages after some minutes.

If there were too many changes on the upstream, it may break something, so I recommend you to test yourself before pushing.

