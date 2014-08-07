---
layout: post
title: How to run BitTorrent Sync as service on Raspberry Pi
tags:
- raspberry pi
- linux
published: true
---

As you may know, Raspberry Pi is quite good at long-running services and applications for its low power consumption.

Thanks to that, it can be your personal 24/7 working cloud storage with BitTorrent Sync, and this post is about how that can be achieved.

----

## 1. Download the binary of BitTorrent Sync

Download **btsync_arm.tar.gz** from [here](http://www.bittorrent.com/sync/downloads/complete/os/arm) and extract **btsync** from it.

{% highlight bash %}
$ tar -xzvf btsync_arm.tar.gz
{% endhighlight %}

## 2. Create a config file

Create a sample config file,

{% highlight bash %}
$ ./btsync --dump-sample-config > btsync.conf
{% endhighlight %}

and edit configurations as you need: **device\_name**, **storage\_path**, **login**, and **password** ...

{% highlight javascript %}
{
	"device_name": "My Raspberry Pi + BTSync Server",
	"listening_port" : 0,	// 0 - randomize port

	/* storage_path dir contains auxilliary app files
	if no storage_path field: .sync dir created in the directory
	where binary is located.
	otherwise user-defined directory will be used
	*/
	"storage_path" : "/mnt/my_data/.sync",

	// uncomment next line if you want to set location of pid file
	// "pid_file" : "/var/run/btsync/btsync.pid",

	"check_for_updates" : true, 
	"use_upnp" : true,	// use UPnP for port mapping

	/* limits in kB/s
	0 - no limit
	*/
	"download_limit" : 0,                       
	"upload_limit" : 0, 

	/* remove "listen" field to disable WebUI
	remove "login" and "password" fields to disable credentials check
	*/
	"webui" :
	{
		"listen" : "0.0.0.0:8888",
		"login" : "USERNAME",
		"password" : "PASSWORD"
	}

	/* !!! if you set shared folders in config file WebUI will be DISABLED !!!
	shared directories specified in config file
	override the folders previously added from WebUI.
	*/
	/*
	,
	"shared_folders" :
	[
		{
			//  use --generate-secret in command line to create new secret
			"secret" : "MY_SECRET_1",                   // * required field
			"dir" : "/home/user/bittorrent/sync_test", // * required field

			//  use relay server when direct connection fails
			"use_relay_server" : true,
			"use_tracker" : true, 
			"use_dht" : false,
			"search_lan" : true,
			//  enable SyncArchive to store files deleted on remote devices
			"use_sync_trash" : true,
			//  restore modified files to original version, ONLY for Read-Only folders
			//    "overwrite_changes" : false, 
			//  specify hosts to attempt connection without additional search
			"known_hosts" :
			[
				"192.168.1.2:44444"
			]
		}
	]
	*/

	// Advanced preferences can be added to config file.
	// Info is available in BitTorrent Sync User Guide.
}
{% endhighlight %}

## 3. Register BitTorrent Sync as a service

### A. Create an init.d script

Create a file with following content:

{% gist 7825826 %}

or download from [here](https://gist.githubusercontent.com/meinside/7825826/raw/b68d29d4faefd1dd212565d4d92596684621b489/btsync-service).

### B. Edit the init.d script

{% highlight bash %}
$ vi btsync-service
{% endhighlight %}

Then replace **BTSYNC\_DIR** and **BTSYNC\_CONFIG** with the locations of yours.

### C. Register it as service

{% highlight bash %}
$ sudo cp btsync-service /etc/init.d/
$ sudo chmod +x /etc/init.d/btsync-service

# if you want it to start automatically on each boot,
$ sudo update-rc.d btsync-service defaults
{% endhighlight %}

### D. Run BitTorrent Sync

{% highlight bash %}
# start,
$ sudo service btsync-service start

# or stop the service
$ sudo service btsync-service stop
{% endhighlight %}

While BitTorrent Sync service is running, you can connect to the web UI through:

[http://your.raspberry.pi:8888](http://your.raspberry.pi:8888)

with the username(**login**) and password you set in the config file.


## 4. Trouble Shooting

### X. It doesn\'t work!

If anything goes wrong or doesn\'t work as you expected, try executing the binary directly:

{% highlight bash %}
$ btsync --config /filepath/of/btsync.conf
{% endhighlight %}

then you\'ll see the reason in the error message.

### X. It crashes my Raspberry Pi!

If you see `smsc95xx 1-1.1:1.0: eth0: kevent 2 may have been dropped` in your kernel logs, or your Raspberry Pi stops working on heavy traffic, try these:

* append `smsc95xx.turbo_mode=N` on your */boot/cmdline.txt*

* add or edit */etc/sysctl.conf* following:

{% highlight bash %}
#vm.vfs_cache_pressure = 100
vm.vfs_cache_pressure = 300
#vm.min_free_kbytes = 8192
vm.min_free_kbytes = 32768
{% endhighlight %}

