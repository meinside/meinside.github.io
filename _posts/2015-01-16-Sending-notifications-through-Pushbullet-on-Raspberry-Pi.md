---
layout: post
title: Sending notifications through Pushbullet on Raspberry Pi
tags:
- raspberry pi
- pushbullet
- ruby
published: true
---

[Pushbullet](https://www.pushbullet.com/) is a very useful service.

Users can send & receive notifications of various types(message, url, file, ...) through it, and all those notifications are synchronized across all of their devices.

It even provides HTTP APIs, for **free**.

I used to receive status report email of my Raspberry Pi server every morning, and I made up my mind to replace the report system to Pushbullet.

With following content, you too, will be able to send Pushbullet notifications on your Raspberry Pi:

----

## 0. Prerequisite

[Ruby](http://www.ruby-lang.org/) needs to be installed on the machine. If it isn't yet, you can get it done with [this script](https://raw.githubusercontent.com/meinside/rpi-configs/master/bin/install_ruby.sh).

## 1. Install pushbullet-ruby gem

{% highlight bash %}
$ gem install pushbullet-ruby
{% endhighlight %}

You can find further information about this gem [here](https://github.com/meinside/pushbullet-ruby).

## 2. Get your Pushbullet access token

Visit [this page](https://www.pushbullet.com/account) and get your access token.

## 3. Get a sample Ruby script

This is a sample ruby script that sends basic status of Raspberry Pi.

{% gist 667f04ec7106c93e4223 %}

Edit **ACCESS_TOKEN** value to yours.

## 4. Try running

Now run the script in the terminal.

{% highlight bash %}
$ ./pushbullet-rpi.rb
{% endhighlight %}

Then you'll receive a Pushbullet notification immediately that is filled with the status report of your Raspberry Pi.

![Notification shown on Pushbullet Chrome extension](https://cloud.githubusercontent.com/assets/185988/5773730/13e5f822-9da9-11e4-972c-9d0631f2331e.png)

![Whole notification on Pushbullet website](https://cloud.githubusercontent.com/assets/185988/5773729/13bc8c76-9da9-11e4-8edd-4aaf8a4aa56b.png)

## 5. Run it periodically

You can run commands periodically using [Cron](https://help.ubuntu.com/community/CronHowto).

{% highlight bash %}
$ crontab -e
{% endhighlight %}

Add following line for running it at 6AM every morning:

{% highlight bash %}
0 6 * * * bash -l -c "/path/to/pushbullet-rpi.rb > /dev/null 2>&1"
{% endhighlight %}

## 6. Now, do your own things

Based on this script, you can add or change stuffs for your use.

For example, you can send notifications

* when [fail2ban](http://www.fail2ban.org/) spotted an intruder in your system,
* when [transmission-daemon](https://www.transmissionbt.com/) finished a download,
* ... and so on!

Have fun with Pushbullet! :-)

