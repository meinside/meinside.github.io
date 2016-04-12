---
layout: post
title: Running a comic/ebook reader service on Raspberry Pi (Ubooquity)
tags:
- raspberry pi
- ubooquity
published: true
---

[Ubooquity](http://vaemendis.net/ubooquity/) is a free home server for comics and ebooks.

It is built with Java, but lightweight enough for being run on Raspberry Pi.

Here are a few steps for setting it up on your Raspberry Pi.

----

## 0. Install required packages

As mentioned above, Ubooquity is built with Java, so you need Java 7+ on your machine.

{% highlight bash %}
$ sudo apt-get oracle-java8-jdk
{% endhighlight %}

## 1. Download Ubooquity

Download .zip file from [this page](http://vaemendis.net/ubooquity/static2/download), and unzip it.

{% highlight bash %}
$ wget -O ubooquity.zip http://vaemendis.net/ubooquity/service/download.php
$ unzip ubooquity.zip -d ubooquity
$ cd ubooquity
{% endhighlight %}

## 2. Start Ubooquity

Run the unzipped .jar file with java:

{% highlight bash %}
$ java -jar Ubooquity.jar -webadmin -headless -port 8080
{% endhighlight %}

and open your web browser with url: `http://localhost:8080/admin`.

## 3. Administration

When you open the url above, you will be greeted with a page which demands a new administrator password:

![require a new administrator password](https://cloud.githubusercontent.com/assets/185988/14452834/4036dd92-00cc-11e6-91e1-d9a04e9f1b87.png)

Type yours and continue to the administration page.

### A. Create Users

First, click `Edit` on **Security** section.

Check `Protect shared content with user accounts` and click `Apply` there.

![protect shared content with user accounts](https://cloud.githubusercontent.com/assets/185988/14452835/45839bc8-00cc-11e6-8b8d-9660abbd9b4a.png)

Go back to the Security section and click `Create new user` for creating a new user.

Now you will see the newly added user in the Security section.

![newly added users](https://cloud.githubusercontent.com/assets/185988/14452839/4a167c64-00cc-11e6-930a-20df39ea9fec.png)

### B. Setup Comics and Books Directories

Click `Edit` on **Comics** and/or **Books** section.

Type your comics' and ebooks' directories into **Shared directory**,

and also the users into **Authorized users**.

Click `Apply` for applying them.

![after setup](https://cloud.githubusercontent.com/assets/185988/14452843/505caf26-00cc-11e6-9d5c-dc50a29c37d2.png)

### C. Other Settings

You can set other useful values in **General** and **Advanced** section.

## 4. Login as a User

Now open your web browser with url: `http://localhost:8080`.

You can login with the id and password of a new user.

![login as a user](https://cloud.githubusercontent.com/assets/185988/14452853/563f5808-00cc-11e6-91cd-d338e835f6ff.png)

After logging in, you can read your comics and ebooks.

## 5. Running Ubooquity as a Service

Until now, Ubooquity was running from the command line.

It can be a nice choice for testing, but not good at all for everyday use.

So let's register Ubooquity as a systemd service.

First, create a service file:

{% highlight bash %}
$ sudo vi /lib/systemd/system/ubooquity.service
{% endhighlight %}

and fill it with following content:

{% highlight bash %}
[Unit]
Description=Ubooquity
After=network.target

[Service]
User=your-username
WorkingDirectory=/path/to/ubooquity
ExecStart=/usr/bin/java -jar Ubooquity.jar -headless -port 8080

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Edit **User** and **WorkingDirectory** to yours, and if needed, also change the port number.

If you want to start Ubooquity on every boot, do the following:

{% highlight bash %}
$ sudo systemctl enable ubooquity.service
{% endhighlight %}

If you just want to start or stop it now, do the following:

{% highlight bash %}
$ sudo systemctl start ubooquity.service
$ sudo systemctl stop ubooquity.service
{% endhighlight %}

All done.

----

Ubooquity is really a nice solution for serving comics and ebooks.

If you're running your Raspberry Pi as a NAS or alike, I think there would be no better choice for it.

