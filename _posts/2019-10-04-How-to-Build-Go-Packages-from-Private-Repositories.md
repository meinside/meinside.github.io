---
layout: post
title: How to build Go packages from private repositories
tags:
- golang
published: true
---

Let's say you have a Go package on a private repository which is tagged as `v0.0.1`,

and want to build an application with it:

{% highlight bash %}
# (your) go package on a private repository
https://github.com/your-account/your-private-repository
{% endhighlight %}

----

## 1. Create a go.mod file with the private repository

This will be the `go.mod` file of your application:

{% highlight bash %}
module github.com/your-account/your-application

go 1.13

require (
  github.com/your-account/your-private-repository v0.0.1
)
{% endhighlight %}

## 2. Edit .gitconfig file

Put following lines to your `~/.gitconfig` file:

{% highlight bash %}
# https://git-scm.com/docs/git-config#Documentation/git-config.txt-urlltbasegtinsteadOf
[url "ssh://git@github.com/your-account/"]
  insteadOf = https://github.com/your-account/
{% endhighlight %}

## 3. Set GOPRIVATE

Then set an environment variable, `GOPRIVATE`:

{% highlight bash %}
# https://tip.golang.org/cmd/go/#hdr-Module_configuration_for_non_public_modules
export GOPRIVATE=github.com/your-account/your-private-repository
{% endhighlight %}

You can put above lines in the .rc files, or even in your build scripts.

----

Then you'll be able to build your application with packages on private repositories :-)

Please let me know if there is a better way!

