---
layout: post
title: Gogs on Raspberry Pi
tags:
- golang
- raspberry pi
published: true
---

_(updated: 2015-06-03)_

[Gogs](http://gogs.io) is a [github](https://github.com)-like Git service built with [Golang](http://golang.org).

With Gogs, you can run your own Git service on Raspberry Pi very easily.

![gogs_logo](https://github.com/gogits/gogs/raw/master/public/img/gogs-large-resize.png)

----

## 0. Prerequisite

### A. Prepare a running Raspberry Pi (Raspbian)

You've already got one, haven't you? :-)

### B. Install Golang

You can install golang from source code with [this script](https://github.com/meinside/rpi-configs/blob/master/bin/prep_go.sh).

It checks out latest code from the [repository](https://code.google.com/p/go/), build, and install binaries for Raspberry Pi.

Whole processes will take about 1 hour or so.

After installation, add following to your rc file(eg. `.zshrc`, `.bashrc`, ...):

{% highlight bash %}
# for go
if [ -d /opt/go/bin ]; then
  export GOROOT=/opt/go
  export GOPATH=$HOME/srcs/go
  export PATH=$PATH:$GOROOT/bin
fi
{% endhighlight %}

Close your terminal and open a new one, then you will be able to run golang excutables.

### C. Install other requirements for Gogs

You'll need to install some more packages before going to the next step.

{% highlight bash %}
$ sudo apt-get install git

# Gogs supports MySQL, PostgreSQL, and SQLite. You can choose one.
$ sudo apt-get install mysql-server
# or
$ sudo apt-get install postgresql
{% endhighlight %}

### D. Create a directory for repositories

{% highlight bash %}
$ mkdir ~/repositories
{% endhighlight %}

----

## 1. Checkout and build Gogs source code

Gogs has no pre-built binaries for Raspberry Pi yet, so there's only one choice: [Install From Source Code](http://gogs.io/docs/installation/install_from_source.html).

### a. If you don't need SQLite3/Redis/Memcache support,

In your terminal,

{% highlight bash %}
# download and install dependencies (takes some time)
$ go get -u -tags "cert" github.com/gogits/gogs

# build
$ cd $GOPATH/src/github.com/gogits/gogs
$ go build
{% endhighlight %}

### b. If you need to enable SQLite3/Redis/Memcache,

{% highlight bash %}
# download and install dependencies (takes some time)
$ go get -u -tags "cert sqlite redis memcache" github.com/gogits/gogs

# build
$ cd $GOPATH/src/github.com/gogits/gogs
$ go build -tags "cert sqlite redis memcache"
{% endhighlight %}

Gogs is nearly ready to run!

But before that, you need to configure some more things up.

----

## 2. Configure Gogs

### a. Setup database (MySQL)

Create a MySQL database for Gogs.

{% highlight bash %}
$ mysql -uroot -p
{% endhighlight %}

Grant some priviliges to the created database.

{% highlight sql %}
mysql> DROP DATABASE IF EXISTS gogs;
mysql> CREATE DATABASE IF NOT EXISTS gogs CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> GRANT ALL ON gogs.* to 'gogs'@'localhost' identified by 'SOME_PASSWORD';
{% endhighlight %}

### b. Edit configuration

Gogs needs some [configuration](http://gogs.io/docs/installation/configuration_and_run.md) before running.

It **recommends** *(or forces? I'm not sure.)* to keep custom config files rather than modifying the default ones directly.

So, duplicate the `conf/app.ini` file first.

{% highlight bash %}
$ cd $GOPATH/src/github.com/gogits/gogs

$ mkdir -p custom/conf
$ cp conf/app.ini custom/conf/app.ini
{% endhighlight %}

Now let's edit the duplicated config file.

{% highlight bash %}
$ vi custom/conf/app.ini
{% endhighlight %}

#### (1) user and repository path

Change **ROOT** to the path of the repository directory which you created above:

{% highlight bash %}
[repository]
ROOT = /home/some_user/repositories
{% endhighlight %}

#### (2) database connection

Change username and password for database connection.

{% highlight bash %}
[database]
; Either "mysql", "postgres" or "sqlite3", it's your choice
DB_TYPE = mysql
HOST = 127.0.0.1:3306
NAME = gogs
USER = gogs
PASSWD = SOME_PASSWORD
; For "postgres" only, either "disable", "require" or "verify-full"
SSL_MODE = disable
; For "sqlite3" only
PATH = data/gogs.db
{% endhighlight %}

#### (3) run mode

If you want Gogs to be run in production mode, change it.

{% highlight bash %}
; Either "dev", "prod" or "test", default is "dev"
RUN_MODE = prod
{% endhighlight %}

#### (4) server configuration

Change domain, port, or other values.

{% highlight bash %}
[server]
PROTOCOL = http
DOMAIN = my-raspberrypi-domain.com
ROOT_URL = %(PROTOCOL)s://%(DOMAIN)s:%(HTTP_PORT)s/
HTTP_ADDR =
HTTP_PORT = 3000
; Disable SSH feature when not available
DISABLE_SSH = false
SSH_PORT = 22
; Disable CDN even in "prod" mode
OFFLINE_MODE = false
DISABLE_ROUTER_LOG = false
; Generate steps:
; $ cd path/to/gogs/custom/https
; $ ./gogs cert -ca=true -duration=8760h0m0s -host=myhost.example.com
;
; Or from a .pfx file exported from the Windows certificate store (do
; not forget to export the private key):
; $ openssl pkcs12 -in cert.pfx -out cert.pem -nokeys
; $ openssl pkcs12 -in cert.pfx -out key.pem -nocerts -nodes
CERT_FILE = custom/https/cert.pem
KEY_FILE = custom/https/key.pem
; Upper level of template and static file path
; default is the path where Gogs is executed
STATIC_ROOT_PATH =
; Application level GZIP support
ENABLE_GZIP = false
; Landing page for non-logged users, can be "home" or "explore"
LANDING_PAGE = home
{% endhighlight %}

You may also have to generate certificate files as the comment says:

{% highlight bash %}
$ mkdir -p cd $GOPATH/src/github.com/gogits/gogs/custom/https
$ cd $GOPATH/src/github.com/gogits/gogs/custom/https
$ ../../gogs cert -ca=true -duration=8760h0m0s -host=my-raspberrypi-domain.com
{% endhighlight %}

#### (5) secret key

Change the default secret key for security.

{% highlight bash %}
[security]
; !!CHANGE THIS TO KEEP YOUR USER DATA SAFE!!
SECRET_KEY = _secret_0123456789_
{% endhighlight %}

#### (6) app name

If you want to change the page titles...

{% highlight bash %}
; App name that shows on every page title
APP_NAME = Gogs on my Raspberry Pi
{% endhighlight %}

#### (99) others

There are other useful settings like email, oauth, webhook, ... etc.

I will try them out soon. :-)

----

## 3. Run Gogs

All things up, now Gogs is ready to run!

### A. Run in the shell

Gogs can be started from the shell.

{% highlight bash %}
$ cd $GOPATH/src/github.com/gogits/gogs
$ ./gogs web
{% endhighlight %}

If nothing goes wrong, you can connect to Gogs with any web browser.

Connect to the address, eg: [http://my-raspberrypi-domain.com:3000](http://my-raspberrypi-domain.com:3000).

You'll meet **Install Steps For First-time Run** page demanding some check-ups.

When you see complaints about your settings, do as the error message suggests.

If all things are OK, you can login with the username and password which were set in the previous page.

### B. Run as a service

You cannot run Gogs from the shell manually everytime, so let's try to run it as a service.

Copy an init.d script file from the source codes:

{% highlight bash %}
$ sudo cp $GOPATH/src/github.com/gogits/gogs/scripts/init/debian/gogs /etc/init.d/gogs
{% endhighlight %}

and edit **WORKING_DIR** and **USER**,

{% highlight bash %}
# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin
DESC="Go Git Service"
NAME=gogs
SERVICEVERBOSE=yes
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
WORKINGDIR=/home/some_user/gogs
DAEMON=$WORKINGDIR/$NAME
DAEMON_ARGS="web"
USER=some_user
{% endhighlight %}

and make it run automatically on boot time:

{% highlight bash %}
$ sudo chmod ug+x /etc/init.d/gogs

# if gogs has higher priority that database server, it would fail to launch on boot time
$ sudo update-rc.d gogs defaults 98
{% endhighlight %}

Gogs can now be manually started with `sudo service gogs start` and stopped with `sudo service gogs stop`.

Well done! :-D

----

## 99. Wrap-up

Gogs is not only a useful application, but also good to run on Raspberry Pis.

I decided to learn Golang for further use on Raspberry Pi :-)

