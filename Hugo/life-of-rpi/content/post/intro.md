+++
Description = ""
Tags = ["RPI2", "RPI3", "RPIZero", "ADSB", "HypriotOS"]
date = "2016-08-31T13:17:25-07:00"
title = "Introduction to my current Hardware"

+++
As an introduction, here is a description of my current Raspberry PI hardware 
and what currently runs on them.


+ **Raspberry PI 2 (my first one)**. Currently in our upstairs attic at home
 running the
 [ADS-B Receiver Project](https://www.adsbreceiver.net "ADS-B Receiver Project")
 on stock Raspian-Jessie.
 Home-made ADS-B receivers are a fascinating topic and a perfect fit for the
 Raspberry PI. In fact, while writing this blog post, I noticed that a new
 version of the ADS-B Receiver software stack has been released. Definetely
 a topic for a future post.

+ **Raspberry PI 3**. Currently connected to our main Wifi-Router via Ethernet
 and running [HypriotOS](http://blog.hypriot.com "HypriotOS Blog"). The whole
 topic of Docker containers is another area worthy of future posts

+ **Raspberry PI Zero**. Currently connected directly to our DSL-Modem/Router
 via Ethernet and hence accessible from the wild and scary internet. It
 currently runs Raspian-Jessie Lite, on top of which is an Apache web-server
 with PHP running the
 [Ampache Music Streaming Server](http://ampache.org "Ampache - Music Streaming Server").

+ **Raspberry PI 3**. This RPI doesn't really have a fixed function yet, it is
 currently sitting in my office, also running
 [HypriotOS](http://blog.hypriot.com "HypriotOS Blog"). Unlike the other Raspberry's
 this one is primarily controlled over the serial TTY interface and not SSH.
