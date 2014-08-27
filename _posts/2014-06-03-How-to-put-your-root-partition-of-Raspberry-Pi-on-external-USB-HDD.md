---
layout: post
title: How to put your root partition of Raspberry Pi on external USB HDD
tags:
- raspberry pi
published: true
---

SD cards have limited read/write cycles, so after using them with Raspberry Pi for some time, they often get broken.

For me, who uses Raspberry Pi as a development server, it is a critical problem.

So I decided to run it with USB HDDs, which are faster and more durable then SD cards.

Here is the way:

----

## 1. Get a clean raspberry pi installation,

----

## 2. Attach your external USB HDD,

----

## 3. Create partitions,

* you should replace **X** with your HDD drive\'s letter (eg. /dev/sd**a**, /dev/sd**b**, ...)

{% highlight bash %}
$ sudo fdisk /dev/sdX
{% endhighlight %}


### A. Create a swap partition (in my case, its size is 1GB)

~~~~
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-625142446, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-625142446, default 625142446): +1G
~~~~

~~~~
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 82
Changed system type of partition 1 to 82 (Linux swap / Solaris)
~~~~

### B. Create a root partition

~~~~
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2): 2
First sector (2099200-625142446, default 2099200): 
Using default value 2099200
Last sector, +sectors or +size{K,M,G} (2099200-625142446, default 625142446): 
Using default value 625142446
~~~~

### C. And save these to the partition table

~~~~
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: If you have created or modified any DOS 6.x
partitions, please see the fdisk manual page for additional
information.
Syncing disks.
~~~~

Now the swap partition is **/dev/sdX1**, and the root partition is **/dev/sdX2**.

----

## 4. Edit /etc/fstab file,

{% highlight bash %}
$ sudo vi /etc/fstab
{% endhighlight %}

#### From:

~~~~
proc            /proc    proc    defaults          0    0
/dev/mmcblk0p1  /boot    vfat    defaults          0    2
/dev/mmcblk0p2  /        ext4    defaults,noatime  0    1
~~~~

#### To:

~~~~
proc            /proc    proc    defaults          0    0
/dev/mmcblk0p1  /boot    vfat    defaults          0    2
/dev/sdX1       none     swap    sw                0    0
/dev/sdX2       /        ext4    defaults,noatime  0    1
~~~~

----

## 5. Format partitions and copy contents to the HDD from SD card,

{% highlight bash %}
# (format partitions)
$ sudo mkswap /dev/sdX1
$ sudo mkfs.ext4 /dev/sdX2

# (copy root partition from sdcard to hdd)
$ sudo dd if=/dev/mmcblk0p2 of=/dev/sdX2 bs=32M conv=noerror,sync

# (check partition)
$ sudo e2fsck -f /dev/sdX2

# (resize partition)
$ sudo resize2fs /dev/sdX2
{% endhighlight %}

----

## 6. Set it up,

{% highlight bash %}
# (backup and edit /boot/cmdline.txt)
$ sudo cp /boot/cmdline.txt /boot/cmdline.old
$ sudo vi /boot/cmdline.txt
{% endhighlight %}

edit /boot/cmdline.txt file as following:

#### From:

{% highlight bash %}
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait
{% endhighlight %}

#### To:

{% highlight bash %}
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/sdX2 rootfstype=ext4 elevator=deadline bootdelay rootdelay rootwait
{% endhighlight %}

* __bootdelay__ and __rootdelay__ is important here. Without them, it was not bootable with horrible error messages for me.

----

## 7. ...and reboot!

{% highlight bash %}
$ sudo rm /etc/rc2.d/S02dphys-swapfile
$ sudo sync
$ sudo shutdown -r now
{% endhighlight %}

All done!

