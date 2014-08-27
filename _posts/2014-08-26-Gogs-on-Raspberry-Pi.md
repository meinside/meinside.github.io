---
layout: post
title: Gogs on Raspberry Pi
tags:
- golang
- raspberry pi
published: true
---

[Gogs](http://gogs.io) is a [github](https://github.com)-like Git service built with [Golang](http://golang.org).

With Gogs, you can run your own Git service on Raspberry Pi very easily.

![gogs_logo](http://gogs.io/imgs/gogs-lg.png)

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

----

## 1. Checkout and build Gogs source code

Gogs has no pre-built binaries for Raspberry Pi yet, so there's only one choice: [Install From Source Code](http://gogs.io/docs/installation/install_from_source.html).

### a. If you don't need SQLite3 support,

In your terminal,

{% highlight bash %}
# download and install dependencies (takes some time)
$ go get -u github.com/gogits/gogs

# build
$ cd $GOPATH/src/github.com/gogits/gogs
$ go build
{% endhighlight %}

### b. If you need to enable SQLite3,

{% highlight bash %}
# download and install dependencies (takes some time)
$ go get -u -tags sqlite github.com/gogits/gogs

# build
$ cd $GOPATH/src/github.com/gogits/gogs
$ go build -tags sqlite
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

### b. Setup repository

Create a directory where your raw repository data will be saved.

{% highlight bash %}
$ mkdir -p ~/somewhere/gogs-repositories
{% endhighlight %}

### c. Edit configuration

Gogs needs some [configuration](http://gogs.io/docs/installation/configuration_and_run.md) before running.

It recommends*(or forces? I'm not sure.)* to keep custom config files rather than modifying the default ones directly.

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

#### (1) repository path

Change **ROOT** to the path of the directory which you created above.

{% highlight bash %}
[repository]
ROOT = /home/pi/somewhere/gogs-repositories
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
HTTP_PORT = 8080
SSH_PORT = 22
; Disable CDN even in "prod" mode
OFFLINE_MODE = false
DISABLE_ROUTER_LOG = false
; Generate steps:
; $ cd path/to/gogs/custom/https
; $ go run $GOROOT/src/pkg/crypto/tls/generate_cert.go -ca=true -duration=8760h0m0s -host=my-raspberrypi-domain.com
CERT_FILE = custom/https/cert.pem
KEY_FILE = custom/https/key.pem
; Upper level of template and static file path
; default is the path where Gogs is executed
STATIC_ROOT_PATH =
{% endhighlight %}

You may also have to generate certificate files as the comment says:

{% highlight bash %}
$ cd $GOPATH/src/github.com/gogits/gogs
$ mkdir -p custom/https
$ cd custom/https
$ go run $GOROOT/src/pkg/crypto/tls/generate_cert.go -ca=true -duration=8760h0m0s -host=my-raspberrypi-domain.com
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

I will try them out soon. :godmode:

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

Connect to the address, eg: [http://my-raspberrypi-domain.com:8080](http://my-raspberrypi-domain.com:8080).

You'll meet **Install Steps For First-time Run** page demanding some check-ups.

If you see a complaint about **Run User**, change it to your current user name.

It can be changed manually in the `custom/conf/app.ini` file:

{% highlight bash %}
; Change it if you run locally
RUN_USER = some_user
{% endhighlight %}

If all things are OK, you can login with the username and password which were set in the previous page.

### B. Run as a service

You cannot run Gogs from the shell everytime, so let's try to run as a service.

Create an init.d script file for Gogs:

{% highlight bash %}
$ sudo touch /etc/init.d/gogs-service
$ sudo chmod +x /etc/init.d/gogs-service
{% endhighlight %}

Edit `/etc/init.d/gogs-service` file with following content:

{% highlight bash %}
#!/bin/bash
#/etc/init.d/gogs-service
#

### BEGIN INIT INFO
# Provides:          gogs-service
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Gogs service
# Description:       init.d script for Gogs service
### END INIT INFO

# XXX - edit these
GOGS_DIR="/some/where/gogs"
GOGS_USER="some_user"

GOGS_USER_HOME="/home/$GOGS_USER"
GOGS_START_SCRIPT="$GOGS_DIR/start.sh"
GOGS_LOG_DIR="$GOGS_DIR/log"

start_gogs() {
  echo "Starting gogs..."
  su - $GOGS_USER -c "nohup $GOGS_START_SCRIPT > $GOGS_LOG_DIR/gogs.log 2>&1 &"
}

# Gogs needs a home path.
export HOME=$GOGS_USER_HOME

# commands
case "$1" in
  start)
    ps aux | grep 'gogs' | grep -v grep | grep -vq start
    if [ $? = 0 ]; then
      echo "Gogs is already running."
    else
      start_gogs
    fi
    ;;
  stop)
    ps aux | grep 'gogs' | grep -v grep | grep -vq stop
    if [ $? = 0 ]; then
      echo "Stopping Gogs."
      kill `ps -ef | grep 'gogs' | grep -v grep | awk '{print $2}'`
    else
      echo "Gogs is already stopped."
    fi
    ;;
  restart)
    ps aux | grep 'gogs' | grep -v grep | grep -vq restart
    if [ $? = 0 ]; then
      kill `ps -ef | grep 'gogs' | grep -v grep | awk '{print $2}'`
    fi
    start_gogs
    ;;
  status)
    ps aux | grep 'gogs' | grep -v grep | grep -vq status
    if [ $? = 0 ]; then
      echo "Gogs is running."
    else
      echo "Gogs is stopped."
    fi
    ;;
  *)
    echo "Usage: /etc/init.d/gogs-service {start|stop|restart|status}"
    exit 1
    ;;
esac

exit 0
{% endhighlight %}

Edit **GOGS_DIR** and **GOGS_USER** as you need,

then Gogs can now be started with `sudo service gogs-service start` and stopped with `sudo service gogs-service stop`.

:heavy_exclamation_mark: For making Gogs start automatically on every boot,

{% highlight bash %}
$ sudo update-rc.d gogs-service defaults
{% endhighlight %}

Well done! :beer:

----

## 99. Wrap-up

Gogs is not only a useful application, but also good to run on Raspberry Pis. :+1:

I decided to learn Golang for further use on Raspberry Pi :-)

