---
title: Arduinos
description: How Arduinos are used/programmed in the nEDM system
layout: basic
order: 50
---

## Arduinos

Newer version of Arduinos, in particular the [Arduino Yun](https://www.arduino.cc/en/Main/ArduinoBoardYun)
can provide much of the functionality of Raspberry Pis, but also have some
advantages including being able to be powered over PoE.  The Yun also has a
linux side, meaning it can run [`pynedm`]({{ site.url }}/Python-Slow-Control)
scripts.

### Field Switch device

This is a device used to handle switching of fields in the `internal_coils`
system.

A backup of the SD card for this device is available in two places:
([Cluster server](http://10.155.59.88/_attachments/nedm%2Fsystem_health/arduino_backup/arduino.img.gz) ) ([Osthalle server](http://db.nedm1/_attachments/nedm%2Fsystem_health/arduino_backup/arduino.img.gz)).

Note that in this image, the username/password for the `pynedm` daemon have
been scrubbed.  To flash this image to a new card one must use `dd`, e.g.:

{% highlight bash %}
gunzip -c arduino.img.gz | sudo dd of=/dev/sd_card
{% endhighlight %}

where the SD card must be at least 4 GB large.

The code on the Arduino Yun runs a simple [sketch](#code) as well as a
[`pynedm` daemon](#code) which is started using an `init.d` script.
Please see the image and code for more details.

Note, that you must enable `extroot` on a new Arduino Yun device to have this
work.  (When doing this, be careful not to format the SD card again!)  See
[here](https://www.arduino.cc/en/Tutorial/ExpandingYunDiskSpace) and
[here](http://wiki.openwrt.org/doc/howto/extroot) for more information.

A couple more points regarding the Yun:

* If the Yun fails to mount the SD card in the `/overlay` directory and instead
mounts it in, `/tmp/overlay-disabled`, this is because the md5sum doesn't
math.  (This will probably be the case.)  Using the SD card flashed from this
version will likely require copying the md5sum, e.g.:
{% highlight bash %}
cp /.extroot.md5sum /tmp/overlay-disabled/etc/extroot.md5sum
{% endhighlight %}
See [here](https://forum.openwrt.org/viewtopic.php?id=28980) for more details.
* The resolution of the name server seems not always to work.  For this reason,
try converting IP names to IP addresses.

#### Code

<script src="https://gist.github.com/mgmarino/9ecdff8780a42ee803f5.js"></script>
