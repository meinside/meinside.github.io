---
layout: post
title: How to run a NATS Streaming Server Cluster on Raspberry Pi
tags:
- raspberry pi
- nats
- stan
- cluster
- message queue
- kafka
- zookeeper
published: true
---

I've been searching for alternatives to [Kafka](https://kafka.apache.org/) for some time,

and found out that NATS and/or NATS Streaming server(a.k.a. STAN) would be a good fit for it.

Unlike Kafka, NATS and STAN are:

* not dependent on JVM
* not dependent on [ZooKeeper](https://zookeeper.apache.org/)
* and quite light-weight, even run on Raspberry Pis!

They are not that famous as Kafka yet,
but are known to be used by many large companies in their production environments,
so I made up my mind to try it on my Raspberry Pis.

----

This post was written and tested with:

* 1 X Raspberry Pi 3 B+ (Raspbian Stretch)
* 1 X Raspberry Pi 3 B (Raspbian Stretch)
* 1 X Raspberry Pi Zero W (Raspbian Stretch)
* Go [1.11.1](https://github.com/golang/go/releases/tag/go1.11.1)
* [NATS Server](https://github.com/nats-io/gnatsd)
  * gnatsd [v1.3.0](https://github.com/nats-io/gnatsd/releases/tag/v1.3.0)
* [NATS Streaming Server](https://github.com/nats-io/nats-streaming-server)
  * nats-streaming-server [v0.11.0](https://github.com/nats-io/nats-streaming-server/releases/tag/v0.11.0)

----

# Get Ready

I have Go development environments on all of my Pis, so installation is not a big thing.

## 1. Install nats-server

{% highlight bash %}
$ go get -u github.com/nats-io/gnatsd
{% endhighlight %}

then `gnatsd` will be installed as `$GOPATH/bin/gnatsd`.

## 2. Install nats-streaming-server

{% highlight bash %}
$ go get -u github.com/nats-io/nats-streaming-server
{% endhighlight %}

then `nats-streaming-server` will be installed as `$GOPATH/bin/nats-streaming-server`.

# Configure & Run

There are two Pis in my house, connected behind a router:

* **3B+**: running on an external HDD, being used as a file server
* **Zero W**: being used as an application server

And there is one at work, also behind a router:

* **3B**: camera connected, being used as an application server (mostly for hobby projects)

I plan to build a NATS cluster made up of **3B+** and **Zero W**,
and run STAN cluster on all three: **3B+**, **Zero W**, and **3B**.

## 1. Generate self-signed certificates

My NATS servers will be exposed to the outer world,
so they should be run with TLS options.

For TLS support, I needed to generate some self-signed certificates.

I used [this script](https://raw.githubusercontent.com/meinside/rpi-configs/master/bin/gen_selfsigned_certs.sh):

{% highlight bash %}
$ ./gen_selfsigned_certs.sh myhostname.com 192.168.0.101 192.168.0.102
{% endhighlight %}

`myhostname.com` is the domain name to my home router.

`192.168.0.101` and `192.168.0.102` are internal IP addresses of my Pis: **3B++** and **Zero W**.

## 2. Build a NATS/STAN cluster with two Pis

As mentioned above, two Pis - **3B+** and **Zero W** - are in the same local area network.

I configured my router to port-forward these two through port 60100-60102 and 60200-60202.

### A. 3B+: will work as STAN (with embedded NATS)

**3B+** will work as a seed(primary) NATS cluster member,
and also as a boostrap node of STAN cluster.

Configuration file for both NATS and STAN is like following:

{% highlight bash %}
# filename: stan.conf

# Configuration file for nats-streaming-server: Seed (Primary)

################
# NATS specific configurations

listen: 0.0.0.0:60100 # host/port to listen for client connections

https: 0.0.0.0:60102 # HTTPS monitoring port

tls: {
	cert_file: "./certs/server-cert.pem"
	key_file: "./certs/server-key.pem"
	ca_file: "./certs/ca.pem"
	verify: true
	timeout: 10
}

# Authorization for client connections
# (Generage passwords with: gnatsd/util/mkpasswd -p)
authorization {
	users = [
		{
			user: USER
			password: PASSWORD
		}
	]
}

# Cluster definition (seed)
cluster {
	listen: 0.0.0.0:60101 # host/port for inbound route connections

	tls {
		cert_file: "./certs/server-cert.pem"
		key_file: "./certs/server-key.pem"
		ca_file: "./certs/ca.pem"
		verify: true
		timeout: 10
	}

	# Authorization for route connections
	authorization {
		user: CLUSTER_USER
		password: PASSWORD
	}

	# Routes are actively solicited and connected to from this server.
	# Other servers can connect to us if they supply the correct credentials
	# in their routes definitions from above.
	routes = [
#		"nats://CLUSTER_USER:PASSWORD@192.168.0.102:60201"
	]
}

# logging options
debug: false
trace: false
logtime: true
log_file: "/tmp/nats.log"

# pid file
pid_file: "/tmp/nats.pid"

# Some system overides

# max_connections
max_connections: 100

# max_subscriptions (per connection)
max_subscriptions: 100

# maximum protocol control line
max_control_line: 512

# maximum payload
max_payload: 65536

# Duration the server can block on a socket write to a client.
# Exceeding the deadline will designate a client as a slow consumer.
write_deadline: "10s"

################
# STAN specific configurations

streaming: {
	id: "stan-rpi"

	store: "file"
	dir: "/home/pi/stan/datastore"

	hb_interval: "10s"
	hb_timeout: "10s"
	hb_fail_count: 5

	#ft_group: "ft"	# for fault-tolerance setting

	sd: false	# debug logging
	sv: false	# trace logging

	tls: {
		client_cert: "./certs/cert.pem"
		client_key: "./certs/key.pem"
		client_ca: "./certs/ca.pem"
	}

	store_limits: {
		max_channels: 100
		max_msgs: 1000
		max_bytes: 10MB
		max_age: "6h"
		max_subs: 100
		max_inactivity: "6h"
	}

	file: {
		compact: true
		compact_fragmentation: 50
		compact_interval: 300
		compact_min_size: 100MB
		buffer_size: 2MB
		crc: true
		crc_poly: 3988292384
		sync_on_flush: true
		file_descriptors_limit: 100
		parallel_recovery: 2
	}

	cluster: {
		node_id: "stan-node-rpi3bplus"
		bootstrap: true
		#peers: ["stan-node-rpi0w", "stan-node-rpi3b"]

		log_path: "/home/pi/stan/log"
		log_cache_size: 1024
		log_snapshots: 1
		trailing_logs: 256
		sync: true
		raft_logging: false
	}
}
{% endhighlight %}

With this config, NATS will be run on port 60100 and 60101(for cluster),
and web interface for monitoring will be run on port 60102.

Now run it with the config file:

{% highlight bash %}
$ nats-streaming-server -sc stan.conf -c stan.conf -user USER -pass PASSWORD
{% endhighlight %}

`-sc` option specifies a config file for STAN, and `-c` for NATS,

so this STAN server (with an embedded NATS server) can be run with this one config file only.

### B. Zero W: will work as STAN (with standalone NATS)

**Zero W** will work as a secondary NATS cluster member,
and also as a non-bootstrap node of STAN cluster.

I wanted to configure it up like **3B+** as one STAN instance,
but I couldn't make it run properly due to [this issue](https://github.com/nats-io/nats-streaming-server/issues/676).

So I had to run NATS and STAN servers respectively.

Configuration files for NATS and STAN are:

{% highlight bash %}
# filename: nats.conf

# Configuration file for nats-streaming-server: Secondary

################
# NATS specific configurations

listen: 0.0.0.0:60200 # host/port to listen for client connections

https: 0.0.0.0:60202 # HTTPS monitoring port

tls: {
	cert_file: "./certs/server-cert.pem"
	key_file: "./certs/server-key.pem"
	ca_file: "./certs/ca.pem"
	verify: true
	timeout: 10
}

# Authorization for client connections
# (Generage passwords with: gnatsd/util/mkpasswd -p)
authorization {
	users = [
		{
			user: USER
			password: PASSWORD
		}
	]
}

# Cluster definition
cluster {
	listen: 0.0.0.0:60201 # host/port for inbound route connections

	tls {
		cert_file: "./certs/server-cert.pem"
		key_file: "./certs/server-key.pem"
		ca_file: "./certs/ca.pem"
		verify: true
		timeout: 10
	}

	# Authorization for route connections
	authorization {
		user: CLUSTER_USER
		password: PASSWORD
	}

	# Routes are actively solicited and connected to from this server.
	# Other servers can connect to us if they supply the correct credentials
	# in their routes definitions from above.
	routes = [
		"nats://CLUSTER_USER:PASSWORD@192.168.0.101:60101"
	]
}

# logging options
debug: false
trace: false
logtime: true
log_file: "/tmp/nats.log"

# pid file
pid_file: "/tmp/nats.pid"

# Some system overides

# max_connections
max_connections: 100

# max_subscriptions (per connection)
max_subscriptions: 100

# maximum protocol control line
max_control_line: 512

# maximum payload
max_payload: 65536

# Duration the server can block on a socket write to a client.
# Exceeding the deadline will designate a client as a slow consumer.
write_deadline: "10s"
{% endhighlight %}

and

{% highlight bash %}
# filename: stan.conf

# Configuration file for nats-streaming-server: Secondary

################
# NATS specific configurations

# (There is `nats_server_url` for STAN, so other NATS options are not needed)

# logging options
debug: false
trace: false
logtime: true
log_file: "/tmp/nats.log"

# pid file
pid_file: "/tmp/nats.pid"

# Some system overides

# max_connections
max_connections: 100

# max_subscriptions (per connection)
max_subscriptions: 100

# maximum protocol control line
max_control_line: 512

# maximum payload
max_payload: 65536

# Duration the server can block on a socket write to a client.
# Exceeding the deadline will designate a client as a slow consumer.
write_deadline: "10s"

################
# STAN specific configurations

streaming: {
	id: "stan-rpi"

	store: "file"
	dir: "/home/pi/stan/datastore"

	nats_server_url: "nats://USER:PASSWORD@127.0.0.1:60200"

	hb_interval: "10s"
	hb_timeout: "10s"
	hb_fail_count: 5

	#ft_group: "ft"	# for fault-tolerance setting

	sd: false	# debug logging
	sv: false	# trace logging

	tls: {
		client_cert: "./certs/cert.pem"
		client_key: "./certs/key.pem"
		client_ca: "./certs/ca.pem"
	}

	store_limits: {
		max_channels: 100
		max_msgs: 1000
		max_bytes: 10MB
		max_age: "6h"
		max_subs: 100
		max_inactivity: "6h"
	}

	file: {
		compact: true
		compact_fragmentation: 50
		compact_interval: 300
		compact_min_size: 100MB
		buffer_size: 2MB
		crc: true
		sync_on_flush: true
		file_descriptors_limit: 100
		parallel_recovery: 1
	}

	cluster: {
		node_id: "stan-node-rpi0w"
		bootstrap: false
		#peers: ["stan-node-rpi3bplus", "stan-node-rpi3b"]

		log_path: "/home/pi/stan/log"
		log_cache_size: 1024
		log_snapshots: 1
		trailing_logs: 256
		sync: true
		raft_logging: false
	}
}
{% endhighlight %}

With these config files, NATS will be run on port 60200 and 60201(for cluster),
and web interface will be run on port 60202.

They can be run with following commands:

{% highlight bash %}
$ gnatsd -c nats.conf
{% endhighlight %}

and

{% highlight bash %}
$ nats-streaming-server -sc stan.conf -user USER -pass PASSWORD
{% endhighlight %}

## 3. Build a remote STAN cluster

**3B** is not in the same network, and not even exposed to the outer world,
but still can be a node of a STAN cluster.

### A. 3B: will work as STAN node

Configuration file for STAN:

{% highlight bash %}
# filename: stan.conf

# Configuration file for nats-streaming-server: External

# logging options
debug:   false
trace:   false
logtime: true
log_file: "/tmp/nats.log"

# pid file
pid_file: "/tmp/nats.pid"

# Some system overides

# max_connections
max_connections: 100

# max_subscriptions (per connection)
max_subscriptions: 100

# maximum protocol control line
max_control_line: 512

# maximum payload
max_payload: 65536

# Duration the server can block on a socket write to a client.
# Exceeding the deadline will designate a client as a slow consumer.
write_deadline: "15s"

################
# STAN specific configurations

streaming: {
	id: "stan-rpi"

	store: "file"
	dir: "/home/pi/stan/datastore"

	nats_server_url: "nats://USER:PASSWORD@myhostname.com:60100, nats://USER:PASSWORD@myhostname.com:60200"

	hb_interval: "10s"
	hb_timeout: "10s"
	hb_fail_count: 5

	#ft_group: "ft"	# for fault-tolerance setting

	sd: false	# debug logging
	sv: false	# trace logging

	tls: {
		client_cert: "./certs/cert.pem"
		client_key: "./certs/key.pem"
		client_ca: "./certs/ca.pem"
	}

	store_limits: {
		max_channels: 100
		max_msgs: 1000
		max_bytes: 10MB
		max_age: "6h"
		max_subs: 100
		max_inactivity: "6h"
	}

	file: {
		compact: true
		compact_fragmentation: 50
		compact_interval: 300
		compact_min_size: 100MB
		buffer_size: 2MB
		crc: true
		sync_on_flush: true
		file_descriptors_limit: 100
		parallel_recovery: 1
	}

	cluster: {
		node_id: "stan-node-rpi3b"
		bootstrap: false
		#peers: ["stan-node-rpi3bplus", "stan-node-rpi0w"]

		log_path: "/home/pi/stan/log"
		log_cache_size: 1024
		log_snapshots: 1
		trailing_logs: 256
		sync: true
		raft_logging: false
	}
}
{% endhighlight %}

Now run it with:

{% highlight bash %}
$ nats-streaming-server -sc stan.conf
{% endhighlight %}

then it will connect to the NATS server which is already configured on `myhostname.com`, and start STAN.

# Resulting Network Topology

![stan_network_topology](https://user-images.githubusercontent.com/185988/47211634-f44ac800-d3d0-11e8-8c00-999d9f88c03c.png)

# Monitor

You can make sure all servers are running well by looking into logs saved in `/tmp/nats.log`.

And also, you can connect to the web interfaces through `https://192.168.0.101/60102` or `https://192.168.0.102/60202`.

# Run as services

Following files are systemd service files I made.

They can be enabled/started/stopped with commands like these:

{% highlight bash %}
$ sudo cp stan.service /lib/systemd/system/

$ sudo systemctl enable stan.service
$ sudo systemctl start stan.service
$ sudo systemctl stop stan.service
{% endhighlight %}

## 1. STAN service file for 3B+

`stan.service` file:

{% highlight bash %}
[Unit]
Description=NATS Streaming Server
After=syslog.target network.target

[Service]
PrivateTmp=true
Type=simple
WorkingDirectory=/home/pi/stan
ExecStart=/home/pi/go/bin/nats-streaming-server -sc stan.conf -c stan.conf -user USER -pass PASSWORD
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s SIGINT $MAINPID
User=pi
Group=pi

[Install]
WantedBy=multi-user.target
{% endhighlight %}

## 2. NATS and STAN service files for Zero W

`nats.service` file:

{% highlight bash %}
[Unit]
Description=NATS Server
After=syslog.target network.target

[Service]
PrivateTmp=true
Type=simple
WorkingDirectory=/home/pi/stan
ExecStart=/home/pi/go/bin/gnatsd -c nats.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s SIGINT $MAINPID
User=pi
Group=pi

[Install]
WantedBy=multi-user.target
{% endhighlight %}

`stan.service` file:

{% highlight bash %}
[Unit]
Description=NATS Streaming Server
Wants=nats.service
After=syslog.target network.target nats.service

[Service]
PrivateTmp=true
Type=simple
WorkingDirectory=/home/pi/stan
ExecStart=/home/pi/go/bin/nats-streaming-server -sc stan.conf -user USER -pass PASSWORD
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s SIGINT $MAINPID
User=pi
Group=pi

[Install]
WantedBy=multi-user.target
{% endhighlight %}

STAN depends on NATS, so **nats.service** was added to **After** in `stan.service` file.

## 3. STAN service file for 3B

`stan.service` file:

{% highlight bash %}
[Unit]
Description=NATS Streaming Server
After=syslog.target network.target

[Service]
PrivateTmp=true
Type=simple
WorkingDirectory=/home/pi/stan
ExecStart=/home/pi/go/bin/nats-streaming-server -sc stan.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s SIGINT $MAINPID
User=pi
Group=pi

[Install]
WantedBy=multi-user.target
{% endhighlight %}

# Test with a client

Create a sample client from [this code](https://github.com/meinside/stan-client-go/tree/master/example/pingpong):

{% highlight go %}
// main.go

package main

import (
	"encoding/json"
	"log"
	"os"
	"os/signal"
	"time"

	"github.com/meinside/stan-client-go"
	"github.com/nats-io/go-nats-streaming"
)

const (
	delaySeconds = 5

	queueGroupUnique = "unique"
	durableDefault   = "durable"

	natsServerURL1 = "nats://myhostname.com:60100"
	natsServerURL2 = "nats://myhostname.com:60200"

	clientCertPath = "./certs/cert.pem"
	clientKeyPath  = "./certs/key.pem"
	rootCaPath     = "./certs/ca.pem"

	natsUsername = "USER"
	natsPassword = "PASSWORD"

	clusterID = "stan-rpi"
	clientID  = "pingpong-client"

	subjectPing = "ping"
	subjectPong = "pong"
)

type pingPong struct {
	Message string `json:"message"`
}

type logger struct {
}

func (l *logger) Log(msg string) {
	log.Println(msg)
}

func (l *logger) Error(err string) {
	log.Println("ERROR: " + err)
}

var sc *stanclient.Client

func main() {
	// for catching signals
	interrupt := make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)

	sc = stanclient.Connect(
		[]string{natsServerURL1, natsServerURL2},
		stanclient.AuthOptionWithUsernameAndPassword(natsUsername, natsPassword),
		stanclient.SecOptionWithCerts(clientCertPath, clientKeyPath, rootCaPath),
		clusterID,
		clientID,
		[]stanclient.ToSubscribe{
			stanclient.ToSubscribe{
				Subject:        subjectPing,
				QueueGroupName: queueGroupUnique,
				DurableName:    durableDefault,
				DeliverAll:     true,
			},
			stanclient.ToSubscribe{
				Subject:        subjectPong,
				QueueGroupName: queueGroupUnique,
				DurableName:    durableDefault,
				DeliverAll:     true,
			},
		},
		handlePingPong,
		publishFailed,
		&logger{},
	)

	go func() {
		time.Sleep(delaySeconds * time.Second)

		sc.Publish(subjectPing, pingPong{Message: "initial ping"})
	}()

	go sc.Poll()

	// wait...
loop:
	for {
		select {
		case <-interrupt:
			break loop
		}
	}

	sc.Close()

	log.Println("Application terminating...")
}

func handlePingPong(message *stan.Msg) {
	var data pingPong
	err := json.Unmarshal(message.Data, &data)

	if err != nil {
		log.Printf("Failed to unmarshal data: %s", err)
		return
	}

	switch message.Subject {
	case subjectPing:
		log.Printf("Received PING: %s", data.Message)

		time.Sleep(delaySeconds * time.Second)
		sc.Publish(subjectPong, pingPong{Message: "pong for ping"})
	case subjectPong:
		log.Printf("Received PONG: %s", data.Message)

		time.Sleep(delaySeconds * time.Second)
		sc.Publish(subjectPing, pingPong{Message: "ping for pong"})
	}
}

func publishFailed(subject, nuid string, obj interface{}) {
	log.Printf("Failed to publish to subject: %s, nuid: %s, value: %+v", subject, nuid, obj)

	// resend it later
	go func(subject string, obj interface{}) {
		time.Sleep(delaySeconds * time.Second)

		log.Println("Retrying publishing...")

		sc.Publish(subject, obj)
	}(subject, obj)
}
{% endhighlight %}

and run it with:

{% highlight bash %}
$ go run main.go
{% endhighlight %}

Pings and pongs!

Now we have a tiny, but clustered message queue system on Raspberry Pis!

# Trouble Shooting

## 1. Bad certificates?

If you see errors about certificates, make sure there are all needed IP addresses and domain names in them.

You can see the contents of your certificates with:

{% highlight bash %}
$ openssl x509 -in server-cert.pem -text
{% endhighlight %}

## 2. Timeout Errors?

If you see any timeout error on server launches, increase the `timeout` values in the config files and retry.

Some tasks may be too much for our little Raspberry Pis... :-(

# Wrap-Up

NATS/STAN is a light-weight, but powerful message queue system that can be run even on Raspberry Pis.

Message queue system is often said to be the best infra structure for microservices,

so it would be a good start to play with our spare Raspberry Pis :-)

