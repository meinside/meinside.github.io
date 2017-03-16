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

After doing them, it just worked as advertised. Cool!

# 1. Plug L800 into Raspberry Pi

I booted my Pi Zero up and plugged L800 in.

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

After installing, I had to create a file named `/etc/udev/rules.d/41-usb_modeswitch.rules`:

```bash
$ sudo vi /etc/udev/rules.d/41-usb_modeswitch.rules
```

and add following line:

```
ATTRS{idVendor}=="1bbb", ATTRS{idProduct}=="f000", RUN+="/usr/sbin/usb_modeswitch -v 1bbb -p f000 -P 0017 -M 55534243123456788000000080000606f50402527000000000000000000000"
```

With this file, L800's USB mode will be switched automatically from **disk** to **modem** on every boot.

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

After removing the WiFi dongle, I could see an *unfamiliar IP* assigned to my Raspberry Pi:

```bash
$ curl ifconfig.co
# or, test on usb0
$ curl ifconfig.co --interface usb0
```

![5_ifconfig_and_ip](https://cloud.githubusercontent.com/assets/185988/21514240/4e753438-cd06-11e6-86b1-1eac4226a3ac.png)

That is, with L800 only, my Raspberry Pi is connected to the world!

----

# 998. Trouble shooting

## A. Not enough power

Without a powered USB hub, my Raspberry Pi Zero did not work properly.

It worked for sometime, but stopped after couple of minutes.

Maybe L800 consumes more power than the Raspberry Pi Zero provides.

I have also tested on Raspberry Pi 3, and **it worked great** without any external power or hub.

## B. usb0 is visible, but not working

Try turning it off and on with:

```bash
$ sudo ifdown usb0
$ sudo ifup usb0
```

and test again.

----

# 999. Wrap-up

LTE/3G data connection is really useful with Raspberry Pi.

It can communicate with you outside of the building, or even in harsh environments as long as its mobile data connection is established.

There can be so many ways of making the most use of it!

