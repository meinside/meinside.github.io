---
layout: post
title: How to use Alcatel L800 on Raspberry Pi Zero
tags:
- raspberry pi
published: true
---

I've been waiting quite long for using my spare data SIM cards with Raspberry Pi,

so I bought [Alcatel L800](http://www.alcatel-mobile.com/global-en/products/detail/L800) from AliExpress and set it up immediately.

Here I write down the whole steps of setting for the record:

----

# 0. Make sure it works on PC or MacOS

After putting my SIM card into L800, I plugged it on my iMac to check if it works correctly.

As most guides on the internet say, I installed 'Web Connection' and connected to `192.168.1.1`, the web console with Chrome browser.

There were a couple of things to do for me, like registering the dial number and APN address.

After doing them, it worked as advertised. Cool!

# 1. Plug L800 into Raspberry Pi

I booted my Pi Zero up and plugged L800.

With `lsusb`, I could see it in the list:

```bash
$ lsusb
```

![1_lsusb](https://cloud.githubusercontent.com/assets/185988/21514235/444728fe-cd06-11e6-908c-6db024f7e973.png)

It was named as 'T & A Mobile Phones'.

So far, so good.

# 2. Switch the mode of USB

It is said that L800 works as external USB disk when plugged in.

I needed to use it as USB modem, not USB disk, so I had to use `usb_modeswitch`:

```bash
$ sudo apt-get install usb-modeswitch
```

After installing, I had to edit `/etc/usb_modeswitch.conf`:

```bash
$ sudo vi /etc/usb_modeswitch.conf
```

and add following lines:

```
# for Alcatel L800
DefaultVendor=0x1bbb
DefaultProduct=0xf000

TargetVendor=0x1bbb
TargetProduct=0x0017

MessageContent="55534243123456788000000080000606f50402527000000000000000000000"
```

It was for switching L800's USB mode from **disk** to **modem**.

After that, my `/etc/usb_modeswitch.conf` looked like this:

![2_usb_modeswitch](https://cloud.githubusercontent.com/assets/185988/21514237/46cc9abe-cd06-11e6-83d5-fcc6cf27baad.png)

# 3. Edit network interface

Assuming that my L800 would be working as a modem, I had to turn it up as a network interface.

I opened up `/etc/network/interfaces/`:

```bash
$ sudo vi /etc/network/interfaces
```

and appended:

```
# for Alcatel L800
auto usb0
iface usb0 inet dhcp
```

Then my `/etc/network/interfaces` looked like this:

![3_network_interface](https://cloud.githubusercontent.com/assets/185988/21514238/4929d8bc-cd06-11e6-9a26-8d6a782dc55b.png)

After rebooting, I could see **usb0** device listed as network interface:

```bash
$ /sbin/ifconfig
```

![4_ifconfig](https://cloud.githubusercontent.com/assets/185988/21514239/4bb75e88-cd06-11e6-97cc-71ac8d5e373b.png)

After removing the WiFi dongle, I could see an unfamiliar IP assigned to my Raspberry Pi:

```bash
$ curl ifconfig.co
```

![5_ifconfig_and_ip](https://cloud.githubusercontent.com/assets/185988/21514240/4e753438-cd06-11e6-86b1-1eac4226a3ac.png)

That is, with L800 only, my Raspberry Pi is connected to the world!

----

# 998. One problem

I had to connect Alacatel L800 to a powered USB hub.

My Raspberry Pi Zero did not work properly without it.

It worked for sometime, but stopped after couple of minutes.

Maybe L800 consumes more power than the Raspberry Pi Zero provides.

I haven't tested with Raspberry Pi 3 yet. *Will test and add the result here.*

----

# 999. Wrap-up

With Raspberry Pi, LTE/3G data connection is really useful.

It can be placed outside of the house, or even in harsh environments as long as its mobile data connection is established.

There can be so many ways of making the most use of it!

