---
layout: post
title: How to build Julia on Raspberry Pi Jessie
tags:
- raspberry pi
- julia
published: true
---

(update 2017-05-15: Julia is [now available as a raspbian package!](https://www.raspberrypi.org/blog/julia-language-raspberry-pi/))

[Julia](http://julialang.org/) is a programming language especially for technical computing, and is often said to be 3x to 5x times faster than [R](https://www.r-project.org/).

I wanted to try it on Raspberry Pi, and here are the steps I have gone through:

----

## 0. Get source codes

Cloned the [git repository](https://github.com/JuliaLang/julia):

{% highlight bash %}
$ cd /tmp
$ git clone https://github.com/JuliaLang/julia.git
{% endhighlight %}

## 1. Install packages for building from the source

I had to install [required packages](https://github.com/JuliaLang/julia#Required-Build-Tools-External-Libraries):

{% highlight bash %}
$ sudo apt-get install -y libblas3gf liblapack3gf libarpack2 libfftw3-dev libgmp3-dev libmpfr-dev libblas-dev liblapack-dev cmake gcc-4.8 g++-4.8 gfortran libgfortran3 m4 libedit-dev
{% endhighlight %}

But due to the cmake package in Raspbian Jessie is outdated(3.0.2, but 3.4.3 or higher was required), I had to remove it:

{% highlight bash %}
$ sudo apt-get purge cmake cmake-data
{% endhighlight %}

and build cmake from the latest source file:

{% highlight bash %}
$ wget https://cmake.org/files/v3.7/cmake-3.7.0.tar.gz
$ tar -xzvf cmake-3.7.0.tar.gz
$ cd cmake-3.7.0
$ ./bootstrap
$ make
$ sudo make install
{% endhighlight %}

## 2. Build Julia

Changed directory into the cloned repository of Julia:

{% highlight bash %}
$ cd /tmp/julia
{% endhighlight %}

and started building:

{% highlight bash %}
$ sudo make all
{% endhighlight %}

Fortunately, nothing went wrong, so it was time to install the built binaries.

## 3. Install Julia

I wanted the binary files to be placed under `/opt/julia`, so did the following:

{% highlight bash %}
$ echo "prefix=/opt/julia" > Make.user
{% endhighlight %}

and started installing:

{% highlight bash %}
$ sudo make install
{% endhighlight %}

## 4. Additional things to be done

After the installation, I changed the permission of installed files:

{% highlight bash %}
$ sudo chown -R $USER /opt/julia
{% endhighlight %}

Without this change, files were not accessible for me, because they were built/installed with root privilege(sudo).

After that, I added `/opt/julia/bin` to my $PATH variable:

{% highlight bash %}
# in .bashrc or .zshrc
PATH=$PATH:/opt/julia/bin
{% endhighlight %}

Julia became excutable from anywhere!

{% highlight bash %}
meinside@raspberrypi:~/files ‹master›$ julia
               _
   _       _ _(_)_     |  A fresh approach to technical computing
  (_)     | (_) (_)    |  Documentation: http://docs.julialang.org
   _ _   _| |_  __ _   |  Type "?help" for help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 0.6.0-dev.1262 (2016-11-16 21:47 UTC)
 _/ |\__'_|_|_|\__'_|  |  Commit 9f999b7 (1 day old master)
|__/                   |  arm-linux-gnueabihf

julia>
{% endhighlight %}

## 5. Wrap-up

I put whole processes into a [bash script here](https://github.com/meinside/rpi-configs/blob/master/bin/install_julia.sh)(not tested yet :-O).

Though it's buildable and excutable on Raspberry Pi, I'm still not sure if Julia runs well on it.

I have to learn more and use it with real world problems :-)

