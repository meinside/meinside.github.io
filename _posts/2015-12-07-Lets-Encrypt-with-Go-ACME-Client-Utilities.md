---
layout: post
title: Let's encrypt with Go ACME client utilities
tags:
- letsencrypt
- raspberry pi
- golang
published: true
---

With [Go ACME client utilities](https://github.com/hlandau/acme), acquiring and renewing certificates from [Let's Encrypt](https://letsencrypt.org/) is a lot easier.

Here are how things go so easily.

----

## 0. Prerequisite

Go needs to be installed on the machine.
(If you don't know how on Raspberry Pi, use [this script](https://github.com/meinside/rpi-configs/blob/master/bin/prep_go.sh))

Also, **libcap-dev** needs to be installed.

{% highlight bash %}
$ sudo apt-get install libcap-dev
{% endhighlight %}

## 1. Install ACME client utilities

Go get the utilities:

{% highlight bash %}
$ go get github.com/hlandau/acme/cmd/acmetool
{% endhighlight %}

## 2. Quickstart

Run quickstart command with acmetool:

{% highlight bash %}
$ sudo `which acmetool` quickstart
{% endhighlight %}

You'll be asked to select or enter several things like validation option, email address, and etc.:

### A. Select ACME Server

I selected **Let's Encrypt Live Server - I want live certificateses** for real certificates.

### B. Select Challenge Conveyance Method

I selected **WEBROOT - Place challenges in a directory**.

I didn't want the challenge files to be in the webroot or my web application's directory,
  
so I entered **/var/run/acme/acme-challenge** for that.

Placing it in other directories needs some more configuration on the web server.

### C. Agreement to the Terms of Service

I selected **I Agree**. (What else could I do???)

### D. Email Address

I entered my email address here.

It will be used when Let's Encrypt tries to reach you, so make sure it is the correct one.

## 3. Setup your web server

I prefer Apache2, and here's what I did for it:
(See [this page](https://github.com/hlandau/acme/blob/master/_doc/WSCONFIG.md) for the detail)

### A. Add alias for challenge files' directory

Before requesting certificates, you need to prepare for the challenge conveyance.

For that, edit your apache site file by adding following lines:

{% highlight bash %}
Alias "/.well-known/acme-challenge/" "/var/run/acme/acme-challenge/"
<Directory "/var/run/acme/acme-challenge">
	AllowOverride None
	Options None

	# If using Apache 2.4+:
	Require all granted

	# If using Apache 2.2 and lower:
	#Order allow,deny
	#Allow from all
</Directory>
{% endhighlight %}

Then the whole file will look like this:

{% highlight bash %}
<VirtualHost *:80>
	ServerAdmin myemail@mydomain.com
	ServerName subdomain.mydomain.com

	DocumentRoot /var/www/html

	Alias "/.well-known/acme-challenge/" "/var/run/acme/acme-challenge/"
	<Directory "/var/run/acme/acme-challenge">
		AllowOverride None
		Options None
		
		# If using Apache 2.4+:
		Require all granted
	</Directory>
</VirtualHost>
{% endhighlight %}

### B. Request certificates for your host

Stop the web server for a moment and request for your domain.

{% highlight bash %}
$ sudo systemctl stop apache2
$ sudo `which acmetool` want subdomain.mydomain.com
{% endhighlight %}

If nothing goes wrong, you will see your certificate files in **/var/lib/acme/live/subdomain.mydomain.com** directory.

### C. Finish configuration with generated certificates

Duplicate your original apache site file to create a SSL one:

{% highlight bash %}
$ sudo cp /etc/apache2/sites-available/your-site.conf /etc/apache2/sites-available/your-site-ssl.conf
$ sudo vi /etc/apache2/sites-available/your-site-ssl.conf
{% endhighlight %}

Add options for generated certificates like this:

{% highlight bash %}
<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ServerAdmin myemail@mydomain.com
		ServerName subdomain.mydomain.com

		DocumentRoot /var/www/html

		SSLEngine on
		SSLCertificateFile /var/lib/acme/live/subdomain.mydomain.com/fullchain
		SSLCertificateKeyFile /var/lib/acme/live/subdomain.mydomain.com/privkey
	</VirtualHost>
</IfModule>
{% endhighlight %}

Then enable your new site:

{% highlight bash %}
$ sudo a2enmod ssl
$ sudo a2ensite your-site-ssl.conf
$ sudo systemctl start apache2
{% endhighlight %}

Now you can see your site served over HTTPS!

### D. (optional) If you want to redirect all http requests to https

Add following lines in your http site file:

{% highlight bash %}
RewriteEngine on
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [L,QSA,R=permanent]
{% endhighlight %}

and restart the server:

{% highlight bash %}
$ sudo a2enmod rewrite
$ sudo systemctl restart apache2
{% endhighlight %}

then even when you connect over http, you'll be redirected to https automatically.

It may make challenge files inaccessible, so please be careful.

## 4. Renew your certificates automatically

Firstly, you can find the absolute path of acmetool by:

{% highlight bash %}
$ which acmetool
{% endhighlight %}

`/path/to/acmetool reconcile --batch` will renew the certificate files when needed, so register it in the crontab for running regularly:

{% highlight bash %}
# m h  dom mon dow   command
0 3 * * 6 /path/to/acmetool reconcile --batch
{% endhighlight %}

then it will run on every Saturday, 03:00.

After renewing the certificates, the web server must be reloaded or restarted for applying them.

It seems that acmetool executes **/usr/lib/acme/hooks/reload** automatically

for reloading/restarting all known http servers like apache2 or nginx.

(I don't have certificates that are about to expire, so I haven't tested this feature yet)

I will see if it really works like that.

**Edited**: Yes, it renewed and reloaded certificates without any problem :-)

## 5. Wrap-up

Using ACME client utilities is faster and cleaner than using [Let's Encrypt's ACME client](/2015/12/04/Lets-Encrypt-with-Raspberry-Pi/).

I can see what is going on inside the box, and can choose what to do and what not to do.

So far, so good!

