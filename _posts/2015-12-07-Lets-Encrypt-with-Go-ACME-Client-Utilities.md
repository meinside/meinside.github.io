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

```bash
$ sudo apt-get install libcap-dev
```

## 1. Install ACME client utilities

Go get the utilities:

```bash
$ go get github.com/hlandau/acme/cmd/acmetool
```

## 2. Quickstart

Run quickstart command with acmetool:

```bash
$ sudo `which acmetool` quickstart
```

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

```
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
```

Then the whole file will look like this:

```
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
```

### B. Request certificates for your host

Restart the web server and request for your domain.

```bash
$ sudo service apache2 restart
$ sudo `which acmetool` want subdomain.mydomain.com
```

If nothing goes wrong, you will see your certificate files in **/var/lib/acme/live/subdomain.mydomain.com** directory.

### C. Finish configuration with generated certificates

Duplicate your original apache site file to create a SSL one:

```bash
$ sudo cp /etc/apache2/sites-available/your-site.conf /etc/apache2/sites-available/your-site-ssl.conf
$ sudo vi /etc/apache2/sites-available/your-site-ssl.conf
```

Add options for generated certificates like this:

```
<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ServerAdmin myemail@mydomain.com
		ServerName subdomain.mydomain.com

		DocumentRoot /var/www/html

		SSLCertificateFile /var/lib/acme/live/subdomain.mydomain.com/fullchain
		SSLCertificateKeyFile /var/lib/acme/live/subdomain.mydomain.com/privkey
	</VirtualHost>
</IfModule>
```

Then enable your new site:

```bash
$ sudo a2enmod ssl
$ sudo a2ensite your-site-ssl.conf
$ sudo service apache2 restart
```

Now you can see your site served over HTTPS!

### D. If you want to redirect all http requests to https

Add following lines in your http site file:

```
RewriteEngine on
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [L,QSA,R=permanent]
```

and restart the server:

```bash
$ sudo a2enmod rewrite
$ sudo service apache2 restart
```

then even when you connect over http, you'll be redirected to https automatically.

## 4. Renew your certificates automatically

`acmetool reconcile --batch` will renew the certificate files when needed, so register it in the crontab for running regularly:

```
# m h  dom mon dow   command
0 3 * * 6 /path/to/acmetool reconcile --batch
```

then it will run on every Saturday, 03:00.

After renewing the certificates, the web server must be restarted for applying them.

It seems that acmetool executes **/usr/lib/acme/hooks/reload** automatically

for reloading/restarting all known http servers like apache2 or nginx.

(I don't have certificates that are about to expire, so I haven't tested this feature yet)

I will see if it really works like that.

## 5. Wrap-up

Using ACME client utilities is faster and cleaner than using [Let's Encrypt's ACME client](http://blog.meinside.pe.kr/Lets-Encrypt-with-Raspberry-Pi/).

I can see what is going on inside the box, and can choose what to do and what not to do.

So far, so good!

