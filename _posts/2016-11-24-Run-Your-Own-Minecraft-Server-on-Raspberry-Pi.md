---
layout: post
title: Run your own Minecraft server on Raspberry Pi
tags:
- raspberry pi
- minecraft
published: true
---

When you want to play Minecraft with your friends, there are two choices:

* play it on the [Realms](http://minecraft.gamepedia.com/Realms), or
* setup and run your own server.

Realms is a subscription-based service which will take you several bucks a month, so if it's possible, running your own server would be a better choice.

Running your own server has one downside: *you need to keep your (server) machine turned on*.

There would be worries about the electric charges, but with Raspberry Pi: a beast with low power consumption, they will not be yours.

Now, here are steps to configure and run the Minecraft server on Raspberry Pi:

----

## 0. Prepare

Java needs to be installed on your machine.

In my case, I installed oracle-java8-jdk:

```bash
$ sudo apt-get install oracle-java8-jdk
```

My Raspberry Pi has the minimum amount of GPU memory(16MB), and an external HDD attached for root partition.

Additionally, the server uses port no: **25565**, so you should open the port if you want to connect from the outside world.

## 1. Download the build tools

On your Raspberry Pi, download the build tools:

```bash
$ wget https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar
```

and run it (you need Java installed):

```bash
$ java -jar BuildTools.jar
```

After some minutes, you'll see the built file named: `spigot-1.10.2.jar`.
(Of course, the version number may vary.)

## 2. Agree with the End User License Agreement

Run the jar file for the first time:

```bash
$ java -jar -Xms512M -Xmx1008M spigot-1.10.2.jar nogui
```

then you'll get a file named: `eula.txt` with this output:

```
Loading libraries, please wait...
[17:40:10 INFO]: Starting minecraft server version 1.10.2
[17:40:10 INFO]: Loading properties
[17:40:10 WARN]: server.properties does not exist
[17:40:10 INFO]: Generating new properties file
[17:40:10 WARN]: Failed to load eula.txt
[17:40:10 INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
[17:40:10 INFO]: Stopping server
```

Read `eula.txt` carefully and do the following if you agree with it:

```bash
$ sed -i -e 's/eula=false/eula=true/g' eula.txt
```

## 3. Run the server

After accepting EULA, now it's the time to launch the server.

Run the jar file again:

```bash
$ java -jar -Xms512M -Xmx1008M spigot-1.10.2.jar nogui
```

In a few minutes, you'll get into a prompt like this:

```
...
[17:53:38 INFO]: Preparing spawn area: 83%
[17:53:39 INFO]: Preparing spawn area: 87%
[17:53:40 INFO]: Preparing spawn area: 90%
[17:53:41 INFO]: Preparing spawn area: 94%
[17:53:42 INFO]: Preparing spawn area: 97%
[17:53:43 INFO]: Done (256.835s)! For help, type "help" or "?"
>
```

Now your server is up and running!

If you want to stop the server, just type `stop` in the prompt.

## 4. Run the server as a service

You cannot run/stop the server manually every time, so it needs to be managed as a service.

Firstly, create a systemd service file like this:

```bash
$ sudo vi /lib/systemd/system/minecraft-server.service
```

then fill with following lines:

```
[Unit]
Description=Spigot Minecraft Server
After=syslog.target
After=network.target

[Service]
Type=simple
User=pi
Group=pi
WorkingDirectory=/home/pi/minecraft
ExecStart=/usr/bin/java -jar -Xms512M -Xmx1008M spigot-1.10.2.jar --noconsole
Restart=always
RestartSec=5
Environment=

[Install]
WantedBy=multi-user.target
```

Replace **User**, **Group**, **WorkingDirectory**, and **ExecStart** values with yours, then all is done.

### Run the server everytime when your Raspberry Pi is turned on

If you want to launch the server automatically when your Raspberry Pi is up:

```bash
$ sudo systemctl enable minecraft-server.service
```

### For starting/stopping the server

If you want to start or stop the server manually:

```bash
$ sudo systemctl start minecraft-server.service
$ sudo systemctl stop minecraft-server.service
```

It will take several seconds to start/stop the server, so please be patient.

```
● minecraft-server.service - Spigot Minecraft Server
   Loaded: loaded (/lib/systemd/system/minecraft-server.service; disabled)
   Active: active (running) since Thu 2016-11-24 18:46:46 KST; 1min 3s ago
 Main PID: 17214 (java)
   CGroup: /system.slice/minecraft-server.service
           └─17214 /usr/bin/java -jar -Xms512M -Xmx1008M spigot-1.10.2.jar --noconsole

Nov 24 18:47:34 raspberry java[17214]: [18:47:34 INFO]: Preparing spawn area: 50%
Nov 24 18:47:35 raspberry java[17214]: [18:47:35 INFO]: Preparing spawn area: 58%
Nov 24 18:47:36 raspberry java[17214]: [18:47:36 INFO]: Preparing spawn area: 67%
Nov 24 18:47:37 raspberry java[17214]: [18:47:37 INFO]: Preparing spawn area: 76%
Nov 24 18:47:38 raspberry java[17214]: [18:47:38 INFO]: Preparing spawn area: 85%
Nov 24 18:47:39 raspberry java[17214]: [18:47:39 INFO]: Preparing spawn area: 91%
Nov 24 18:47:40 raspberry java[17214]: [18:47:40 INFO]: Preparing start region for level 2 (Seed: 6136538408987226186)
Nov 24 18:47:41 raspberry java[17214]: [18:47:41 INFO]: Preparing spawn area: 46%
Nov 24 18:47:42 raspberry java[17214]: [18:47:42 INFO]: Server permissions file permissions.yml is empty, ignoring it
Nov 24 18:47:42 raspberry java[17214]: [18:47:42 INFO]: Done (30.277s)! For help, type "help" or "?"
```

## 5. Trouble shooting

### Server is outdated

If you cannot connect to the server with 'server is outdated' error, update the .jar file with:

```bash
# if the newest version is '1.11'
$ java -jar BuildTools.jar --rev 1.11
```

## 6. Wrap-up

Raspberry Pi may not be perfect for running full-sized Minecraft server yet, but it would worth a try :-)

