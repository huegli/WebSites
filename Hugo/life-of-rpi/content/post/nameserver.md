+++
Description = ""
Tags = ["RPIZero", "RPI3", "Docker", "HypriotOS", "DNS", "BIND"]
date = "2016-10-05T14:00:36-07:00"
title = "Setting up Nameservers"

+++
For the longest time, I wanted to set up DNS in my local network, so as
to not have to always type `http://192.168.0.100:9000` when I wanted
to access the [DockerUI](https://github.com/kevana/ui-for-docker 
"UI-for-Docker") on my Docker Raspberry or `\\192.168.1.79\Music`
to mount the Music Share on my NAS.

While it would have been possible to go the quick-and-dirty route of
just using an `/etc/hosts` file sync'd across all my Linux machines 
(and probably Windows & Mac PC's as well), I wanted something that 
would also work on iPad's, Phones etc. So it had to be full-blown 
[DNS](https://en.wikipedia.org/wiki/Domain_Name_System "Domain Name
Resolution). And even there, I could have gone the eas(ier) route
of using something like
[dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html "DNSMasq").
Instead I decided to go full-hog and run a name server using
[BIND](https://en.wikipedia.org/wiki/BIND "BIND"), and if that wasn't
enough, to run it in a Docker container and to have a primary
and secondary name server on different hosts.

As a primary name server, I wanted to use my Raspberry Zero
as it is hooked up to Ethernet (via the one-and-only USB port) and other
than running the Ampache Music server (which I plan to dockerize and 
relocate to a different Raspberry), it has nothing else to do.

Setting up the configuration for the BIND name server on the Raspberry Zero
in Docker had it's own challenges that I will detail in a future post, but
by far the most difficult thing was to get the other clients in my network
to use this nameserver instead of the default choice (my DSL Modem / Router
from AT&T UVerse provides it's own IP address, which is `192.168.1.254` in
my network as the DNS server to use).

Again, the easy way would have been to configure each client directly, 
through `/etc/resolv.conf` on Linux or similar hardwired overrides in 
Windows and Mac OS X. But what I wanted to do was to have that 
configuration happen automatically. The DSL Modem / Router did not
provide the ability to specify a custom IP for DNS primary and secondary
server, so I was stuck with relying on my wireless router, a 
[D-Link DIR-880L](http://us.dlink.com/products/connect/ac1900-wi-fi-router/ "D-Link DIR-880L")
which provided configurable settings to do just that.

However, I learned a few lessons about this router the hard way:

1. While the router allows one to specify any IP address, including
reserved IP's such as ones in the `192.168.255.255` range, it is a bad
idea to have the name server be WITHIN the range of IP's that this router
provides to it's clients. The information in the 
[D-Link support forum](http://forums.dlink.com/index.php?topic=54498.0
"D-Link support") came in handy here. (UPDATE: [this D-Link suppor article]
(http://forums.dlink.com/index.php?topic=60881.0 "D-Link support") suggests 
that this is a bug in the DIR-880L firmware.)

2. The router by default provides it's own IP (`192.168.0.1` in my case
as the DNS server), this feature called
[DNS Relay](http://forums.dlink.com/index.php?topic=45143.0 "DNS Relay")
can and should be disabled.

Initially I ran all my testing on my Docker Raspberry which was a client 
of the D-Link router and therefore had an IP from the range handled by
that router (`192.168.0.255`). The frustrating thing was that things
would SOMETIMES work on some machines only to fail at a later time.

Eventually I ended up with the following set-up:

* A Primary DNS Server for my local domain on my Raspberry Zero on 
a fixed Etherned connection with an IP provided by the AT&T DSL Modem 
/ Router, OUTSIDE of the network served by the wireless D-Link Router. 
This BIND runs inside a Docker container.

* A Secondary DNS Server running on my NAS (a 
[Netgear ReadyNAS 314](http://www.netgear.com/home/products/connected-storage/RN314.aspx#?cid=wmt_netgear_organic "ReadyNAS 314") ).
On this NAS, I have enabled
root access to get to it's Debian-Linux derivative OS, but for which
I haven't dared to install Docker, but instead relied on one of the
[Apps](https://apps.readynas.com/pages/ "ReadyNAS Apps") provided 
for this NAS. This server is a DNS slave
to the Raspberry Zero DNS master. The NAS is also OUTSIDE of the 
network served by the Wireless router.

* Both primary and secondary DNS server IP's are provided to all
clients of the Wireless D-Link router.

* A third DNS server runs on the Docker Raspberry which is IN the
network and a client of the Wireless D-Link router. This DNS server
is not advertised and needs to be explicitly configured by a client
to be used (so far only the Docker Raspbery itself, with a custom
`/etc/resolv.conf` entry).

