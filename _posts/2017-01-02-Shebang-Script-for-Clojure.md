---
layout: post
title: Shebang Script for Clojure
tags:
- clojure
published: true
---

Shebang lines come in handy when writing and running scripts in CLIs.

With [Leiningen](http://leiningen.org/) and its plugin, [Clojure](http://clojure.org/) can also be run with shebang script.

----

# 1. Install something

## A. Install Leiningen

Firstly, we need Java:

{% highlight bash %}
$ sudo apt-get install oracle-java8-jdk
{% endhighlight %}

After that, download and place `lein` script in a preferred place:

{% highlight bash %}
$ sudo wget "https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein" -O "/usr/local/bin/lein"
{% endhighlight %}

We need additional permissions for running `lein`:

{% highlight bash %}
$ sudo chown $USER.$USER "/usr/local/bin/lein"
$ sudo chmod uog+x "/usr/local/bin/lein"
{% endhighlight %}

After all things are setup correctly, we can see the version of `lein` with following command:

{% highlight bash %}
$ lein version
Leiningen 2.7.1 on Java 1.8.0_65 Java HotSpot(TM) Client VM
$
{% endhighlight %}

## B. Install lein-exec

We need [lein-exec](https://github.com/kumarshantanu/lein-exec) for executing Clojure files with ease.

Add `[lein-exec "0.3.6"]` to your `~/.lein/profiles.clj`, then it will look like:

{% highlight clojure %}
{:user {:plugins [[lein-exec "0.3.6"]]}}
{% endhighlight %}

`lein-exec` will be installed automatically when next time you run `lein`.

# 2. Write the shebang line

Append following line on the first line of your script:

{% highlight bash %}
#!/usr/bin/env lein exec
{% endhighlight %}

and add execute permission to your script:

{% highlight bash %}
$ chmod +x aaaaaa.clj
{% endhighlight %}

then it will be runnable by itself:

{% highlight bash %}
$ ./aaaaaa.clj
{% endhighlight %}

----

# 999. Wrap-up

Clojure may not be intended to be used like this,

but in this way, it will be a lot easier to run and test in the terminal :-)

