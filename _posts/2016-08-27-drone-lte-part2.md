---
layout: post
title: Drone LTE (Part II)
---

This part introduces a cloud-based Ground Control Station (GCS), sets up telemetry forwarding from the secondary companion computer, and tests via a simulated vehicle.

### Cloud GCS

[MAVProxy](http://qgroundcontrol.org/mavlink/mavproxy_startpage) running on an Ubuntu 16.04 droplet on [Digital Ocean](https://www.digitalocean.com/).

For the initial proof-of-concept, a simple GCS in the cloud will do. This will be the final endpoint for the [MAVLink](http://qgroundcontrol.org/mavlink/start) telemetry messaging. Ideally, ICE/STUN/TURN (Voice-over-IP folks will recognize these !) would allow a GCS on a laptop (also behind a NAT) to communicate directly with the LTE-connected drone.

Install MAVProxy on the droplet (note that only Python3 is installed by default):
{% highlight bash %}
sudo apt-get update
sudo apt-get install python2.7 python-dev python-opencv python-wxgtk3.0 python-pip python-matplotlib python-pygame python-lxml
sudo -H pip install MAVProxy
{% endhighlight %}

### Telemetry Forwarding (Secondary Companion Computer)

[MAVProxy](http://qgroundcontrol.org/mavlink/mavproxy_startpage) also supports forwarding of [MAVLink](http://qgroundcontrol.org/mavlink/start) telemetry.

Install MAVProxy on Odroid C0:
{% highlight bash %}
sudo apt-get update
sudo apt-get install python-dev python-opencv python-wxgtk3.0 python-pip python-matplotlib python-pygame python-lxml
sudo -H pip install MAVProxy
{% endhighlight %}

### Testing via Simulation

Install [DroneKit-Python](http://python.dronekit.io/) and [DroneKit-SITL](http://python.dronekit.io/develop/sitl_setup.html) on Odroid C0:
{% highlight bash %}
sudo -H pip install dronekit
sudo -H pip install dronekit-sitl
{% endhighlight %}

The copter simulation would normally start but...
{% highlight bash %}
odroid@odroid:~$ dronekit-sitl copter
os: linux, apm: copter, release: stable
SITL already Downloaded and Extracted.
Ready to boot.
Execute: /home/odroid/.dronekit/sitl/copter-3.3/apm --home=-35.363261,149.165230,584,353 --model=quad
Traceback (most recent call last):
  File "/usr/local/bin/dronekit-sitl", line 11, in <module>
    sys.exit(main())
  File "/usr/local/lib/python2.7/dist-packages/dronekit_sitl/__init__.py", line 475, in main
    sitl.launch(args, verbose=True)
  File "/usr/local/lib/python2.7/dist-packages/dronekit_sitl/__init__.py", line 266, in launch
    p = Popen([self.path] + args, cwd=wd, shell=sys.platform == 'win32', stdout=PIPE, stderr=PIPE)
  File "/usr/lib/python2.7/subprocess.py", line 711, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1343, in _execute_child
    raise child_exception
OSError: [Errno 8] Exec format error
{% endhighlight %}

The included models are not built for the ARM architecture on the Odroid C0 !

Re-build the models:
{% highlight bash %}
sudo apt-get install git
git clone git://github.com/ArduPilot/ardupilot.git
cd ardupilot/ArduCopter
git submodule init
git submodule update .
make sitl
{% endhighlight %}

Start a copter simulation on the Odroid C0 (binds to port 5760):
{% highlight bash %}
dronekit-sitl ~/ardupilot/ArduCopter/ArduCopter.elf
{% endhighlight %}

Connect Odroid C0 via LTE as described in Part I.

Forward the telemetry to the cloud GCS from the Odroid C0:
{% highlight bash %}
mavproxy.py --master tcp:127.0.0.1:5760 --out <Droplet IP>:14550
{% endhighlight %}

Start the cloud GCS on the droplet:
{% highlight bash %}
mavproxy.py —master=udp:<Droplet IP>:14550
{% endhighlight %}

The cloud GCS should now have control of the simulated copter via LTE !

