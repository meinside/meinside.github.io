---
layout: post
title: How to get pushbullet notifications on fail2ban's ban actions and successful ssh logins
tags:
- raspberry pi
- pushbullet
- golang
published: true
---

Goals of this post:

* Receive Pushbullet notifications
  * whenever a fail2ban's ban action is triggered
  * whenever a user successfully logs into the server
* These notifications will also show geo location of the given ip addresses

----

## 0. Prerequisite

[fail2ban](http://www.fail2ban.org/wiki/index.php/Downloads) and [golang](https://golang.org/dl/) must be installed on your machine.

## 1. Get your access token and key

Visit [pushbullet](https://www.pushbullet.com/#settings/account) and [ipstack](https://ipstack.com/) to get your access token/key.

## 2. Intsall

### A. Install pb-send

**pb-send** is a small application that sends messages through pushbullet.

{% highlight bash %}
$ go get -u github.com/meinside/pb-send
{% endhighlight %}

### B. Install ip2loc

**ip2loc** fetches geo locations of given ip addresses.

{% highlight bash %}
$ go get -u github.com/meinside/ipstack-go/cmd/ip2loc
{% endhighlight %}

## 3. Setup

Create config files for **pb-send**:

{% highlight bash %}
$ vi ~/.config/pb-send.json
{% endhighlight %}

{% highlight json %}
{
	"access_token": "PUT_YOUR_PUSHBULLET_ACCESS_TOKEN_HERE"
}
{% endhighlight %}

and **ip2loc**:

{% highlight bash %}
$ vi ~/.config/ip2loc.json
{% endhighlight %}

{% highlight json %}
{
	"access_key": "PUT_YOUR_IPSTACK_ACCESS_KEY_HERE",
	"is_premium": false
}
{% endhighlight %}

Now you can test them with:

{% highlight bash %}
$ ip2loc 8.8.8.8
$ pb-send "test message"
{% endhighlight %}


**NOTE**: fail2band and PAM is run by root privilege,

so `pb-send.json` and `ip2loc.json` should also be placed in `/root/.config/`.

## 4. Configure fail2ban

Firstly, create `notify-fail2ban.sh` file that will be run by fail2ban:

{% gist bdbcfc1c15bc6aca57bada57dcce06c0 %}

Edit **LOCATOR** and **SENDER** paths to yours, and make it executable:

{% highlight bash %}
$ chmod +x /path/to/your/notify-fail2ban.sh
{% endhighlight %}

Now duplicate a fail2ban ban action:

{% highlight bash %}
$ cd /etc/fail2ban/action.d
$ sudo cp iptables-multiport.conf iptables-multiport-letmeknow.conf
$ sudo vi iptables-multiport-letmeknow.conf
{% endhighlight %}

then append a line at the end of **actionban**, which will execute `notify-fail2ban.sh`:

{% highlight bash %}
# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>
            /path/to/your/notify-fail2ban.sh <ip> <port>
{% endhighlight %}

(You should edit */path/to/your/* to yours.)

Now, create your custom `jail.local` file:

{% highlight bash %}
$ sudo vi /etc/fail2ban/jail.local
{% endhighlight %}

with following content:

{% highlight bash %}
[DEFAULT]

#
# MISCELLANEOUS OPTIONS
#

# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space (and/or comma) separator.
ignoreip = 127.0.0.1/8

# External command that will take an tagged arguments to ignore, e.g. <ip>,
# and return true if the IP is to be ignored. False otherwise.
#
# ignorecommand = /path/to/command <ip>
ignorecommand =

# "bantime" is the number of seconds that a host is banned.
bantime  = 36000

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 600

# "maxretry" is the number of failures before a host get banned.
maxretry = 3

# custom ban action
banaction = iptables-multiport-letmeknow
{% endhighlight %}

Finally, restart the fail2ban service:

{% highlight bash %}
$ sudo systemctl restart fail2ban
{% endhighlight %}

## 5. Configure PAM

Create `notify-ssh-login.sh` file that will be run by PAM:

{% gist 091376305f6577b142e1e55204f192c1 %}

Again, edit **LOCATOR** and **SENDER** paths to yours, and make it executable:

{% highlight bash %}
$ chmod +x /path/to/your/notify-ssh-login.sh
{% endhighlight %}

After that, open `/etc/pam.d/sshd` file:

{% highlight bash %}
$ sudo vi /etc/pam.d/sshd
{% endhighlight %}

and append following lines at the end of it:

{% highlight bash %}
# for notifying successful logins
session optional pam_exec.so seteuid /path/to/this/notify-ssh-login.sh
{% endhighlight %}

(Of course, you should edit */path/to/this/* to yours.)

## 6. See it running

As long as all the things are setup correctly, you will receive notifications on each ssh login and fail2ban's ban action:

<img width="405" alt="ban-action" src="https://user-images.githubusercontent.com/185988/58946706-79002000-87c1-11e9-9234-73fd0e8c27d4.png">

<img width="405" alt="ssh-login" src="https://user-images.githubusercontent.com/185988/58946707-7998b680-87c1-11e9-83c1-5374d6e588a7.png">

Now you can see when and where each login and ban action occurred in one place!

