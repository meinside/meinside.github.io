---
layout: post
title: When Oracle Cloud's Ubuntu instance doesn't accept connections to ports other than 22
tags:
- oracle cloud
- ubuntu
- linux
published: true
---

Thanks to the free trial periods, I'm testing this and that on Oracle Clouds.

While running a Ubuntu instance, I was stuck with a weird problem that made me pull out hairs for hours.


My problem was this:

I added several ingress rules in my default security list,

but the instance didn't accept any connection to ports other than 22, the SSH port.

<img width="1207" alt="2020-01-15_20 44 58" src="https://user-images.githubusercontent.com/185988/72434470-020f1900-37df-11ea-8ef5-5464e79e3668.png">

I looked into nearly every network setting in the Oracle Cloud console and searched for similar problems on the web,

but nothing worked.

----

After some time and more tests, I began to suspect that a firewall or something was blocking connections.

When SSH server on port 22 was down, it said **connection was refused**.

But when I tried connecting to other ports, it said **no route to host**.

Hmmmm... interesting, something was blocking connections.

----

I searched for any configuration for `iptables`.

There were two files in `/etc/iptables/`, and gotcha!

I found rules that were rejecting connections in `/etc/iptables/rules.v4` file:

{% highlight bash %}
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
# iptables configuration for Oracle Cloud Infrastructure

# See the Oracle-Provided Images section in the Oracle Cloud Infrastructure
# documentation for security impact of modifying or removing these rule

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [463:49013]
:InstanceServices - [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p udp --sport 123 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
...
...
{% endhighlight %}

Because of these things, changes in the console would never have effects.

Amazon AWS's Ubuntu images don't have such configurations by default, so it maybe due to Oracle's security policy.

I didn't want to take Oracle's default security policy down, so I just added following lines:

{% highlight bash %}
...
...
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT

################
# my custom rules
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3000 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8888 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22022 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 60000:60100 -j ACCEPT
################

-A INPUT -j REJECT --reject-with icmp-host-prohibited
...
...
{% endhighlight %}

By both adding ingress rules on the console and inserting lines in `/etc/iptables/rules.v4`,

I could finally connect through other ports without any problem!

----

Oracle is a relatively new runner in the cloud race.

It needs some more work on it, but I think the new `Always Free` service is a really attractive starter for developers like me.

