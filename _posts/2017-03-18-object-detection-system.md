---
layout: post
title: YOLO Object Detection on a Security Camera
comments: true
---

**Intro**

The idea is to rapidly develop (in a few hours) a system that applies state-of-the-art object detection ([YOLO](https://pjreddie.com/darknet/yolo/)) to images from a security camera. This has the benefit of significantly reducing false detections, and lays the groundwork for more intelligent handling of detection of people, vehicles etc.

**Design**

Although this is a rapid prototype, there is a desire for an elegant, sustainable, design. The camera automatically drops images into Dropbox once it detects motion (from video). Several new Java microservices are built on a [Digital Ocean](https://www.digitalocean.com/) droplet. The microservices communicate with each other via a local [Redis](https://redis.io/) message queue, and with Dropbox APIs. The overall flow is as follows:

1. Camera detects motion and drops new image into a specific Dropbox folder.
2. *Longpoll* microservice maintains a long poll connection with the Dropbox APIs, waits for a new image file notification, retrieves the image, and pushes a message into *new_image_queue*.
3. *Detect* microservice waits for a new message in *new_image_queue*, pops the message, and passes the image to a local [YOLO](https://pjreddie.com/darknet/yolo/) install for object detection. If the detected objects are not in the defined *exclusions* file, a message is pushed to *notify_queue*, and a JSON blob with detection info (filename, objects, confidences) is added to a Redis sorted set, *events*, and indexed by Unix timestamp.
4. *Notify* microservice waits for a new message in *notify_queue*, pops the message, and uploads the image to a new folder via Dropbox APIs. 

**Results**

The system has been running without issue for several weeks. Detection of vehicles and people works remarkably well. Only 1 in 5 motion detections from the camera result in a real notification event, providing a substantial decrease in false positives.

Redis Events
```
127.0.0.1:6379> zrange events 0 -1
  1) "{\"filename\":\"pa_2017-03-02_18-36-30_940.jpg\",\"objects\":[{\"truck\":47}]}"
  2) "{\"filename\":\"pa_2017-03-02_18-36-35_193.jpg\",\"objects\":[{\"truck\":27},{\"car\":63}]}"
  3) "{\"filename\":\"pa_2017-03-02_18-36-36_400.jpg\",\"objects\":[{\"car\":81}]}"
  4) "{\"filename\":\"pa_2017-03-02_19-17-35_391.jpg\",\"objects\":[{\"person\":57}]}"
  5) "{\"filename\":\"pa_2017-03-02_19-17-36_651.jpg\",\"objects\":[{\"person\":76}]}"
  6) "{\"filename\":\"pa_2017-03-02_19-23-07_420.jpg\",\"objects\":[{\"person\":30},{\"person\":44},{\"truck\":26},{\"car\":55}]}"
  7) "{\"filename\":\"pa_2017-03-02_19-23-17_440.jpg\",\"objects\":[{\"car\":58},{\"truck\":30}]}"
```

Vehicle Detection
![vehicle detection](/images/detection-vehicle.jpg)

Person Detection
![person detection](/images/detection-person.jpg)

[YOLO](https://pjreddie.com/darknet/yolo/) hasn't seen too many deer !
![deer detection](/images/detection-deer.jpg)

Night vision and moths can be challenging !
![moth detection](/images/detection-moth.jpg)


**Code**

The code for the Java microservices is available [here](https://github.com/jamesjackson/object-detection-system).

**References**

[How To Install and Configure Redis on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-redis-on-ubuntu-16-04)

[A Java library for the Dropbox Core API](https://github.com/dropbox/dropbox-sdk-java)

[A blazingly small and sane redis java client](https://github.com/xetorthio/jedis)





