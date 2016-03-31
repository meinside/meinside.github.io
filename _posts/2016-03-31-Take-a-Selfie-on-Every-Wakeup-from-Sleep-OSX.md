---
layout: post
title: Take a selfie on every wake-up from sleep (OSX)
tags:
- osx
published: true
---

For some reason, I wanted to take a selfie whenever my Macbook Air wakes up from sleep.

Here are the steps I took for it:

----

## 0. Install tools that are needed

`imagesnap` is a command line tool which takes a photo with the built-in camera of Macbook.

Install it with brew:

{% highlight bash %}
$ brew install imagesnap
{% endhighlight %}

Then install `sleepwatcher`:

{% highlight bash %}
$ brew install sleepwatcher
{% endhighlight %}

It will be used for monitoring sleep/awake states of the Macbook.

## 1. Create a shell script

Create a script that will save photos using `imagesnap`.

{% highlight bash %}
#!/bin/bash

IMAGESNAP_BIN="/usr/local/bin/imagesnap"
SAVE_PATH="/path/to/the/imagesnap_captures"	# XXX - edit this to yours

$IMAGESNAP_BIN "${SAVE_PATH}/captured_`date +%Y%m%d_%H%M`.jpg" > /dev/null 2>&1
{% endhighlight %}

This script will save captured images into the **SAVE_PATH**.

Saved images will be named like *captured_20160331_2058.jpg*.

Now it's time to set `sleepwatcher` up, which will execute this script.

## 2. Setup sleepwatcher

Copy `de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist` file into `/Library/LaunchDaemons/`:

{% highlight bash %}
$ sudo cp /usr/local/Cellar/sleepwatcher/2.2/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist
{% endhighlight %}

and edit it:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>de.bernhard-baehr.sleepwatcher</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/local/sbin/sleepwatcher</string>
		<string>-w /path/to/script.sh</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>KeepAlive</key>
	<true/>
</dict>
</plist>
{% endhighlight %}

With `-w` option, `/path/to/script.sh` will be executed on wake. (if you need, use `-s` for sleep state)

You can see more options by running `/usr/local/sbin/sleepwatcher` without any option.

Now load this file:

{% highlight bash %}
$ sudo launchctl load /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist
{% endhighlight %}

then `sleepwatcher` starts to monitor the sleep/wake state of your machine.

----

Now your selfie will be captured on every wake up!

