---
layout: post
title: How to get Pushbullet notifications on Fail2ban's ban actions
tags:
- raspberry pi
- pushbullet
- golang
published: true
---

Goals:

* Receive Pushbullet notifications when Fail2ban's ban action is triggered.
* Those notifications should include the protocol, banned ip, and geo location.

----

## 0. Prerequisite

[Fail2ban](http://www.fail2ban.org/wiki/index.php/Downloads) and [Golang](https://golang.org/dl/) must be installed on your machine.

## 1. Get your Pushbullet access token

Visit [this page](https://www.pushbullet.com/account) and get your access token.

## 2. Build

{% gist a5afeda2a854919dae12 %}

Download the code,

{% highlight bash %}
$ wget https://gist.githubusercontent.com/meinside/a5afeda2a854919dae12/raw/1e7ea48d5cef31bf024337bcd69b81919c9a51b0/pushbullet-fail2ban.go
$ vi pushbullet-fail2ban.go
{% endhighlight %}

change **MyPushbulletChannel** and **MyPushbulletToken** to yours,

{% highlight go %}
const (
	MyPushbulletChannel = ""                                 // XXX - empty string when not needed
	MyPushbulletToken   = "abcdefghijklmnopqrstuv0123456789" // XXX - your pushbullet API token here
)
{% endhighlight %}

and build it:

{% highlight bash %}
$ go get -u github.com/mitsuse/pushbullet-go
$ go build pushbullet-fail2ban.go
{% endhighlight %}

Now you got the executable binary: **pushbullet-fail2ban**.

To make things sure, you can test it:

{% highlight bash %}
$ ./pushbullet-fail2ban "SSH" "8.8.8.8"
{% endhighlight %}

If you get a message like this, everything is good so far:

![fail2ban_pushbullet_sample](https://cloud.githubusercontent.com/assets/185988/10960323/38786f12-83c7-11e5-95b4-f431a84c53d9.png)

## 5. Configure Fail2ban

Firstly, duplicate your current ban action:

{% highlight bash %}
$ cd /etc/fail2ban/action.d
$ sudo cp iptables-multiport.conf iptables-multiport-letmeknow.conf
$ sudo vi iptables-multiport-letmeknow.conf
{% endhighlight %}

then append a line at the end of **actionban**, which will execute **pushbullet-fail2ban**:

{% highlight bash %}
# (example)
actionban = iptables -I fail2ban-<name> 1 -s <ip> -j DROP
            /path/to/this/pushbullet-fail2ban "<name>" "<ip>"
{% endhighlight %}

(Of course, you should edit */path/to/this/* to yours.)

Now, edit your jail.local file:

{% highlight bash %}
$ sudo vi /etc/fail2ban/jail.local
{% endhighlight %}

**banaction** should be edited like this:

{% highlight bash %}
# ACTIONS
#banaction = iptables-multiport
banaction = iptables-multiport-letmeknow
{% endhighlight %}

Finally, restart the Fail2ban service:

{% highlight bash %}
$ sudo service fail2ban restart
{% endhighlight %}

## 6. Additionally

Geo ip info is provided by [FreeGeoIP](https://freegeoip.net/).

If you want to see more about the geo ip (like zip code, longitude/latitude, or etc.),

edit the lines near the call of *getFreeGeoIpResult()* function.

