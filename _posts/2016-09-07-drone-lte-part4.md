---
layout: post
title: Drone LTE (Part IV)
comments: true
---

This part provides a demonstration of the LTE-connected drone.

### Final Setup

![3DR Solo with LTE telemetry](/images/ltedrone.jpg)

### Live Demo

<iframe width="560" height="315" src="https://www.youtube.com/embed/7ARJ3PadnUo" frameborder="0" allowfullscreen></iframe>

### Open Issues

The Cloud Control Station (CCS) periodically loses link (MAVLink messaging level). Note that the MAVLink messages are using UDP. TCP would not help here, as the head-of-line blocking issues would create new problems. This area requires additional investigation into the protocol behavior, and the MAVProxy implementation.

### Next Steps

* Investigate link drop issues
* Characterize LTE communication during flight
* Stream video over LTE
* Investigate ICE/STUN/TURN options for MAVLink messaging via NATs