---
layout: post
title: Drone LTE (Part I)
---

### Drone
[3DR Solo](https://3dr.com/solo-drone/specs/) w/Gimbal and GoPro HERO4 camera 

The [Solo architecture](https://dev.3dr.com/concept-architecture.html) shows the primary companion computer (iMX.6) that runs Yocto Linux. The [accessory port](https://dev.3dr.com/hardware-accessorybay.html) provides USB access to the iMX.6 via a breakout board, but the kernel only includes support for USB serial devices (FTDI and CDC-ACM). Adding a secondary companion computer provides some additional flexibility, and reduces the chances of bricking the primary.

### Secondary Companion Computer

[Odroid C0](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G145326484280) with battery

One really nice aspect of the C0 is the integrated battery power circuit. The Solo accessory port can provide 5V power, but it's limited to 1.05A.

Load the Ubuntu 16.04 image onto the micro SD card from a Macbook Pro:

{% highlight bash %}
gunzip -d ~/Downloads/ubuntu-16.04-mate-odroid-c1-20160727.img.xz
diskutil unmountDisk /dev/disk2
sudo dd if=~/Downloads/ubuntu-16.04-mate-odroid-c1-20160727.img of=/dev/rdisk2 bs=1m
{% endhighlight %}


### LTE dongle

Aircard 340u (AT&T Beam)

Some good background on [LTE in Linux](https://aleksander.es/data/FOSDEM%202014%20-%20LTE%20in%20your%20Linux%20based%20system.pdf).

The Aircard 340u is not correctly recognized in Linux until a [firmware patch](http://www.downloads.netgear.com/files/aircard/AC340U_linux_patch_v4.zip) is applied. The patch disables the Windows 8 MBIM Identity Morphing.

Note that the dongle is not always recognized on boot. However, it does show up after pulling/re-inserting. Modem Manager should show something like this:

{% highlight bash %}
odroid@odroid:~$ mmcli -L

Found 1 modems:
	/org/freedesktop/ModemManager1/Modem/0 [Sierra Wireless, Incorporated] AirCard 340U
	
odroid@odroid:~$ mmcli -m 0

/org/freedesktop/ModemManager1/Modem/0 (device id 'f9ec6c6d9e9a9e248e7fd69e9b51e480bb493cd0')
  -------------------------
  Hardware |   manufacturer: 'Sierra Wireless, Incorporated'
           |          model: 'AirCard 340U'
           |       revision: 'NTG9X15C_01.13.12.14 r18981 ntgrbc-fwbuild1 2014/07/11 13:48:17'
           |      supported: 'gsm-umts
           |                  lte
           |                  gsm-umts, lte'
           |        current: 'gsm-umts, lte'
           |   equipment id: '013323000538430'
  -------------------------
  System   |         device: '/sys/devices/lm0/usb2/2-1'
           |        drivers: 'qmi_wwan, qcserial'
           |         plugin: 'Sierra'
           |   primary port: 'cdc-wdm0'
           |          ports: 'cdc-wdm0 (qmi), wwan0 (net)'
  -------------------------
  Numbers  |           own : '1512*******'
  -------------------------
  Status   |           lock: 'sim-pin2'
           | unlock retries: 'sim-pin (3), sim-pin2 (3), sim-puk (10), sim-puk2 (10)'
           |          state: 'registered'
           |    power state: 'on'
           |    access tech: 'lte'
           | signal quality: '55' (recent)
  -------------------------
  Modes    |      supported: 'allowed: 2g, 3g, 4g; preferred: none'
           |        current: 'allowed: 2g, 3g, 4g; preferred: none'
  -------------------------
  Bands    |      supported: 'dcs, egsm, pcs, g850, u2100, u1900, u850, eutran-ii, eutran-iv, eutran-v, eutran-xvii'
           |        current: 'dcs, egsm, pcs, g850, u2100, u1900, u850, eutran-ii, eutran-iv, eutran-v, eutran-xvii'
  -------------------------
  IP       |      supported: 'ipv4, ipv6, ipv4v6'
  -------------------------
  3GPP     |           imei: '***************'
           |  enabled locks: 'net-pers'
           |    operator id: '310410'
           |  operator name: 'AT&T'
           |   subscription: 'unknown'
           |   registration: 'home'
  -------------------------
  SIM      |           path: '/org/freedesktop/ModemManager1/SIM/0'

  -------------------------
  Bearers  |          paths: ‘none'
{% endhighlight %}

Install QMI utilities:
{% highlight bash %}
odroid@odroid:~$ sudo apt-get install libqmi-utils
{% endhighlight %}

Check signal strength:
{% highlight bash %}
odroid@odroid:~$ sudo qmicli -d /dev/cdc-wdm0    --nas-get-signal-strength --device-open-proxy
[/dev/cdc-wdm0] Successfully got signal strength
Current:
	Network 'lte': '-81 dBm'
Other:
	Network 'cdma-1xevdo': '-125 dBm'
RSSI:
	Network 'lte': '-81 dBm'
	Network 'cdma-1xevdo': '-125 dBm'
ECIO:
	Network 'lte': '-31.5 dBm'
	Network 'cdma-1xevdo': '-2.5 dBm'
IO: '-106 dBm'
SINR: (8) '9.0 dB'
RSRQ:
	Network 'lte': '-16 dB'
SNR:
	Network 'lte': '-3.6 dB'
RSRP:
	Network 'lte': '-115 dBm’
{% endhighlight %}

Start the connection:
{% highlight bash %}
odroid@odroid:~$ sudo qmicli -d /dev/cdc-wdm0 --wds-start-network=  --client-no-release-cid --device-open-proxy

[/dev/cdc-wdm0] Network started
	Packet data handle: '1143419264'
[/dev/cdc-wdm0] Client ID not released:
	Service: 'wds'
	    CID: '1'
{% endhighlight %}

Bring up the IP interface:
{% highlight bash %}
odroid@odroid:~$ sudo dhclient wwan0

odroid@odroid:~$ ifconfig wwan0
wwan0     Link encap:Ethernet  HWaddr 02:41:05:b1:61:86  
          inet addr:10.165.72.130  Bcast:10.165.72.131  Mask:255.255.255.252
          inet6 addr: fe80::41:5ff:feb1:6186/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1430  Metric:1
          RX packets:3 errors:0 dropped:0 overruns:0 frame:0
          TX packets:49 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:906 (906.0 B)  TX bytes:7368 (7.3 KB)
{% endhighlight %}

Remove the default route associated with the WiFi interface:
{% highlight bash %}
odroid@odroid:~$ netstat -nr
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.165.72.129   0.0.0.0         UG        0 0          0 wwan0
0.0.0.0         192.168.1.254   0.0.0.0         UG        0 0          0 enx8cae4cf5675c
10.165.72.128   0.0.0.0         255.255.255.252 U         0 0          0 wwan0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 enx8cae4cf5675c
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 enx8cae4cf5675c

odroid@odroid:~$ sudo route del -net 0.0.0.0/0 gw 192.168.1.254
{% endhighlight %}

It's alive !
{% highlight bash %}
odroid@odroid:~$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=127 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=160 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=66.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=57 time=131 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=57 time=318 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=57 time=198 ms
^C
--- 8.8.8.8 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5849ms
rtt min/avg/max/mdev = 66.814/167.198/318.361/78.230 ms
{% endhighlight %}


