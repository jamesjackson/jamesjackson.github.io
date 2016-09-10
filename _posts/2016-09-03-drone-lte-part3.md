---
layout: post
title: Drone LTE (Part III)
comments: true
---

This part establishes the communication between the secondary companion computer and the drone (the final piece of the puzzle), and tests end-to-end telemetry between Solo and the Cloud Control Station (CCS).

### Drone/Companion Computer USB Connection

A [breakout board](http://viresity.com/product/3dr-solo-accessory-breakout-board/) provides a micro USB OTG interface to Solo via the [accessory port](https://dev.3dr.com/hardware-accessorybay.html).

Connecting the Odroid C0 to Solo via a USB OTG cable creates a serial device (/dev/ttyACM0) on the Odroid C0, and a serial device (/dev/ttyGS0) on the Solo. By default, Solo supports login via the /dev/ttyGS0 device. 

SSH into Solo as described [here](https://dev.3dr.com/starting-network.html).

Disable login via /dev/ttyGS0 on Solo by commenting out this line in /etc/inittab:
{% highlight bash %}
2:2345:respawn:/sbin/getty 115200 ttyGS0 vt100
{% endhighlight %}
and re-reading inittab:
{% highlight bash %}
init q
{% endhighlight %}

Connect Solo to the Internet as described [here](https://dev.3dr.com/starting-utils.html).

Install MAVProxy on Solo:
{% highlight bash %}
smart install mavproxy
{% endhighlight %}

### End-to-end Testing

Start the CCS on the droplet:
{% highlight bash %}
mavproxy.py —master=udp:<Droplet IP>:14550
{% endhighlight %}

Connect Odroid C0 via LTE as described in Part I.

Setup Odroid C0 to forward the telemetry from Solo (/dev/ttyACM0) to the CCS:
{% highlight bash %}
sudo mavproxy.py --master /dev/ttyACM0 --out <Droplet IP>:14550
{% endhighlight %}

Setup Solo to forward the telemetry to Odroid C0 (/dev/ttyGS0):
{% highlight bash %}
mavproxy.py --master=127.0.0.1:14550 --out /dev/ttyGS0
{% endhighlight %}

The CCS should now have control of Solo via LTE !