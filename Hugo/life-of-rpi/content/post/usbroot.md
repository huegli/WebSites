+++
Description = ""
Tags = ["RPI3", "USB", "RootFS", "Docker"]
date = "2016-09-24T08:35:34-07:00"
title = "Using a USB HDD at Root Filesystem"

+++
A quick detour during my efforts to get `Dump1090` running properly in 
[Docker](https://hub.docker.com/ "Docker").

For a while now, it has been bothering me that the Raspberry PI have 
assigned as my main Docker Machine is running of the usual SD Card 
(32 GB in this case). While I have not personally experienced it,
apparently the SD Card on a Raspberry can wear out if there are too
many writes, which is why Raspberry PI distributions make use of
[tmpfs](https://en.wikipedia.org/wiki/Tmpfs "tmpfs"). As Docker in
particular does lot's of writing for image generation and as I also
anticipate compiling lot's of SW on this Raspberry, I wanted to find 
a better solution.

The Docker Raspberry has an old 1.5 TB USB HDD connected to it 
and until now, I had this HDD formatted as an ext4 partition, mounted
it during boot and then had the diretory tree under `/var/lib` (where
most of Docker dynamic data resides) mounted on the external HDD.

However, ideally I wanted the entire filesystem on the external HDD. 
My first attempt to make this happen was following an
[article on raspberrypi.org]
(https://www.raspberrypi.org/blog/pi-3-booting-part-i-usb-mass-storage-boot/ "Booting USB Mass Storage").
However, after following the steps, the Docker Raspberry became unresponsive 
and I could no longer log in via SSH. One reason could be that the 
instructions were written for [Raspbian](https://www.raspbian.org/ "Raspbian")
and I was trying to do it with the Docker-optimized 
[HypriotOS](http://blog.hypriot.com/ "HypriotOS"). However, more likely,
as the article on USB boot uses a memory stick as opposed to a spinning
disk HDD, my guess is that the HDD simply doesn't come up fast enough
for the boot procedure to work (the article mentions a 2 second limit).

Anyway, my second attempt of having an all-HDD filesystem on the Docker
Raspberry was to follow [this article on AdaFruit]
(https://learn.adafruit.com/external-drive-as-raspberry-pi-root/ "External Drive as Raspberry PI Root"). 
In this case, the working filesystem is on the USB HDD but the inital 
boot volume is still read from the SD Card. Not perfect, especially
considering that I am vasting a 32 GB SD Card for providing a boot volume
that's no larger than a few 100 MB, but it get's the job done of reducing
wear-and-tear on the Docker Raspberry SD Card and unlike the first
attempt, this one worked on the first try.





