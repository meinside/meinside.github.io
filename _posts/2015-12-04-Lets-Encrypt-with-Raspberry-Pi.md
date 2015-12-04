---
layout: post
title: Let's encrypt with Raspberry Pi
tags:
- raspberry pi
- letsencrypt
published: true
---

[Let's Encrypt](https://letsencrypt.org/) started public beta today.

It provides free and semi-automated certificate issuance/renewal/revocation.

With this, you can serve your web application securely with HTTPS.

----

## 1. Install Let's Encrypt

You can install Let's Encrypt by simply cloning its [repository on Github](https://github.com/letsencrypt/letsencrypt).

```bash
$ cd /path/to/somewhere
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
```

## 2. Create & install certificate

On Raspberry Pi, it will take some time everytime when you run the command.

```bash
$ ./letsencrypt-auto --help all
```

It will show all options and their descriptions.

Then run it again with desired options:

```bash
$ ./letsencrypt-auto -d MY_DOMAIN1 -d MY_DOMAIN2 --apache -m MY_EMAIL --redirect --agree-tos
```

When **--apache** is given, it will configure things up for Apache by itself.

With **--redirect**, access requests to HTTP will be redirected to HTTPS automatically.

If **certonly --standalone** given, it will just create certificate files, keeping all others untouched.

After some time, it will show a report of the issuance and some guide about backing up your files, so read them carefully.

## 3. Renew certificate

Installed certificates will expire in 3 months due to the policy of Let's Encrypt.

But we can renew them easily with executing the command again:

```bash
$ ./letsencrypt-auto -d MY_DOMAIN1 -d MY_DOMAIN2 --apache -m MY_EMAIL --redirect --agree-tos --renew-by-default
```

It would be handy if you put this command in your crontab:

```
# m h  dom mon dow   command
0 5 1 * * /path/to/somewhere/letsencrypt-auto -d MY_DOMAIN1 -d MY_DOMAIN2 --apache -m MY_EMAIL --redirect --agree-tos --renew-by-default > /dev/null 2>&1
```

Then it will renew certificates on the 1st day of every month, 05:00.

## 4. Wrapping up

I'm running my Raspberry Pi's web server with a free DDNS service: [DuckDNS](http://www.duckdns.org/).

I didn't want to pour my money on issuing SSL certificate for xxxx.duckdns.org, so I couldn't run it on HTTPS.

But now, thanks to Let's Encrypt and DuckDNS, my Raspberrry Pi can serve its web application on HTTPS without any cost.

What a wonderful world!

