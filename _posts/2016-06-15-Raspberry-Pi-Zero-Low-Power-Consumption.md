---
layout: post
title: Raspberry Pi Zero's Low Power Consumption
tags:
- raspberry pi
published: true
---

I don't have any device to measure the exact amount of power consumption,

but I tested it with three peripherals:

- WiFi dongle,
- HAT,
- and camera module.

I wanted to see if all of these peripherals work ok or not with my cheap power supply.

----

## A. Tested Peripherals

### 1. WiFi dongle

AirLink101's USB WiFi dongle. For your informatioin, [this](http://airlink101.com/products/awll5099.php) one.

It worked mostly well on Pi 1B and 2B, given a 2A power supply.

### 2. HAT

Pimoroni's [Scroll pHat](https://shop.pimoroni.com/collections/raspberry-pi/products/scroll-phat).

### 3. Camera Module

v1.3[(old one)](https://www.raspberrypi.org/products/camera-module/) was used.

It seems to draw much power on Pi 1B and 2B:

they often crashed even with powered USB hub, especially while WiFi connected.

### 4. USB Power Supply

Printed specification on its back says:

```
INPUT:  AC 100-240V
        50-60Hz 0.15A
OUTPUT: DC 5.0v 700mA
```

So it's 0.7A output, much less than [the official one](https://www.raspberrypi.org/products/universal-power-supply/)'s 2.0A.

With this power supply, Pi 1B needed powered USB hub for using WiFi dongle.

But Pi 1B+ and Pi 2B worked fine with this one only.

## B. Test Method on Pi Zero

![settings](https://cloud.githubusercontent.com/assets/185988/16071555/feb944ae-3316-11e6-836a-e574fec0d7df.jpg)

While connected to WiFi, I repeatedly captured images with `raspistill`

and turned on/off the LEDs on Scroll pHat.

## C. Result

Pi Zero didn't fall unresponsive with the test method. It kept alive.

I even ran several commands simultaneously during the test, but it didn't make any change.

## D. Conclusion

It is not a precise test, so there is no guarantee at all.

But at least, Pi Zero seems to bear normal load on 0.7A power supply with WiFi dongle, HAT, and camera module running.

Pi Zero is often advertised as a low power consumer compared to its older brothers, so it may be a proof of that.

----

## Wrap-up

Raspberry Pi Zero lacks several components like LAN or USB ports,

but its small form factor and low power consumption will cover them all up!

