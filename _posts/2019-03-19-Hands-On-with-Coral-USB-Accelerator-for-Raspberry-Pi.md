---
layout: post
title: Hands-on with Coral USB Accelerator (for Raspberry Pi)
tags:
- google
- coral
- tpu
- raspberry pi
published: true
---

[Coral](https://coral.withgoogle.com/) launched [dev board](https://coral.withgoogle.com/products/dev-board/) and [usb accelerator](https://coral.withgoogle.com/products/accelerator/) a couple of weeks ago and they are now available on the store.

I though the usb accelerator would be enough for me, so I ordered one of it and tested on my Raspberry Pi 3B+.

----

## 1. Purchase

Fortunately, the shipping and delivery fee to my country was *zero* on Mouser.

Ordered on Saturday and received on Monday.

What an amazing delivery!

## 2. Unboxing

![coral_box](https://user-images.githubusercontent.com/185988/54582450-174ad800-4a54-11e9-80d3-2aa5d2b7a57c.jpg)

![coral_box_contents](https://user-images.githubusercontent.com/185988/54582451-174ad800-4a54-11e9-9dde-d40ad99a511d.jpg)

![coral_box_contents_rear](https://user-images.githubusercontent.com/185988/54582453-174ad800-4a54-11e9-8327-fb1b1e6db5e8.jpg)

![coral_box_opened](https://user-images.githubusercontent.com/185988/54582454-17e36e80-4a54-11e9-9708-80378bfc7e8f.jpg)

![coral_manual](https://user-images.githubusercontent.com/185988/54582456-17e36e80-4a54-11e9-9d33-02cdf171d8f3.jpg)

## 3. Configuration

Following [this guide](https://coral.withgoogle.com/tutorials/accelerator/), I downloaded and unzipped [edgetpu_api.tar.gz](https://coral.withgoogle.com/tutorials/accelerator/)

and executed `python-tflite-source/install.sh`.

It installed several packages, changed udev rules, and copied library files.

The guide advices to plug in & out the usb accelerator, but to make sure, I just rebooted the machine.

## 4. Test

In the `edgetpu` directory, ran following command:

{% highlight bash %}
python3 demo/classify_image.py \
--model test_data/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
--label test_data/inat_bird_labels.txt \
--image test_data/parrot.jpg
{% endhighlight %}

and got almost the same result:

{% highlight bash %}
W0320 09:13:03.732145    6245 package_registry.cc:65] Minimum runtime version required by package (5) is lower than expected (10).
---------------------------
Ara macao (Scarlet Macaw)
Score :  0.61328125
---------------------------
Platycercus elegans (Crimson Rosella)
Score :  0.15234375
{% endhighlight %}

There was a warning message: `Minimum runtime version required by package (5) is lower than expected (10).`.

It doesn't seem like a critical issue, but needs to be fixed sooner or later.

## 5. What now?

So, I got the usb accelerator and made sure it runs ok on my Raspberry Pi, but what to do now?

Maybe I need to learn TensorFlow Lite and related stuffs, train my own models, and build some applications with them.

