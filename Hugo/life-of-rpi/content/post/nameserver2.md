+++
Description = ""
Tags = ["RPIZero", "RPI3", "Docker", "HypriotOS", "DNS", "BIND"]
date = "2016-10-14T21:08:22-07:00"
title = "Setting up Nameservers Part 2 - Slave Configuration"

+++
As mentioned in my previous post on 
[setting up Nameservers](/2016/10/05/nameserver/ "Setting up Nameservers"),
I have set up a total of 3 nameservers for my local home network.

* A primary (master) nameserver running on my private network on a 
Raspberry PI Zero, running in a Docker container

* A secondary (slave) nameserver running on my
[Netgear ReadyNAS 314](http://www.netgear.com/home/products/connected-storage/RN314.aspx#?cid=wmt_netgear_organic "ReadyNAS 314") 
(not dockerized).

* A third (slave) nameserver running on my Docker Raspberry PI 3.

In this post, I want to describe the `named.conf.*` files for the 
respective nameservers in a bit more detail


**named.conf.local for Docker slave nameserver**

```
key docker-ns1.home.nikolaischlegel.com {
        algorithm hmac-md5;
        secret "************************";
};

server 192.168.1.77 {
        keys docker-ns1.home.nikolaischlegel.com;
};

zone "home.nikolaischlegel.com" {
        type slave;
        file "/etc/bind/zones/bak.home.nikolaischlegel.com";
        masters { 192.168.1.77; };
};

zone "168.192.in-addr.arpa" {
        type slave;
        file "/etc/bind/zones/bak.192.168";
        masters { 192.168.1.77; };
};
```

**named.conf.options for Docker slave nameserver**

```
options {
        directory "/var/cache/bind";

        # Hide version string for security
        version "not currently available";
        
        # this is a secondary nameserver
        recursion yes;
        allow-transfer { none; };
        allow-notify { none; };
        allow-update-forwarding { none; };

        # Forward all DNS queries to the Google Public DNS.
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

        # Allow queries from 
        #  - local subnets (192.168.0 or 192.168.1)
        #  - Docker (172.17)
        allow-query { 
                192.168.0.0/23;
                172.17.0.0/16;
        };

        dnssec-validation auto;
        auth-nxdomain no;    # conform to RFC1035
};
```
Some things to mention about this configuration:

* Because the Docker Raspberry does not necessarily have a fixed IP 
(it appears to the master nameserver as the IP assigned by my ISP 
because my wireless router is also my DMZ host), I define a 
TSIG key for communicating with the master nameserver.

* The nameserver runs as the user `bind`, hence the zone files recieved
from the master nameserver are stored in a seperate subdirectory
`/etc/bind/zones/` that the `bind` user has write permissions to.

* I define both the zone for name -> IP and zone for reverse IP -> name 
lookup

* Since this is a secondary / slave nameserver nobody should be allowed
to further transfer / notify or update anything.

* Any queries for non-local zones get forwarded to Google's DNS severs

* For the Docker nameserver, queries are allowed only from the local
subnet (clients hanging of my wireless router) and Docker containers (
which by default have an IP address in the `172.17.0.0/16` range)

* `dnssec-validation auto` has to do on whether and how a secure
chain of trust to the DNS root servers is established, allowing
for secure / trusted DNS loookups. Quite frankly, 
`dnssec-validation no` would have been perfectly fine, but
`auto` seems to work just fine too.
[dnssec-validation explained](https://users.isc.org/~jreed/dnssec-guide/dnssec-guide.html#dnssec-validation-explained "dnssec-validation explained") 
has some more details.

* `auth-nxdomain no` seems to override default BIND behavior in that 
it will only answer non-authoratively for domain names that don't exist.
I am not entirely sure if it matters, but since all `named.conf.*` files
I have seen so far seem to have it set to `no`, I left it in place.


**named.conf.local on NAS slave nameserver**
```
zone "home.nikolaischlegel.com" {
        type slave;
        file "/etc/bind/zones/bak.home.nikolaischlegel.com";
        masters { 192.168.1.77; };
};

zone "168.192.in-addr.arpa" {
        type slave;
        file "/etc/bind/zones/bak.192.168";
        masters { 192.168.1.77; };
};
```

**named.conf.options for NAS slave nameserver**
```
options {

        directory "/apps/bindr6/tmp";
  
        # Hide version string for security
        version "not currently available";
        
        listen-on port 53 { 
                192.168.1.79;
        };
            
        # this is a secondary nameserver
        recursion yes;
        allow-transfer { none; };
        allow-notify { none; };
        allow-update-forwarding { none; };
        notify no;
               
        # Forward all DNS queries to the Google Public DNS.
        forwarders { 
                8.8.8.8; 
                8.8.4.4;
        };

        # Allow queries from 
        #  - local subnets (192.168.0 or 192.168.1)
        #  - Docker (172.17)
        #  - AT&T UVerse which will include my DMZ host (75.0.0.0/10)

        allow-query { 
                192.168.0.0/23;
                172.17.0.0/16;
                75.0.0.0/10;
        };

        dnssec-validation auto;
        auth-nxdomain no;    # conform to RFC1035
};
```

The differences with this slave nameserver running un-dockerized
on the NAS compared to the Docker nameserver are as follows.

* Since the NAS does have a fixed IP in my local network, I can
do authentication based on IP and don't need to define a key
for it's communication with the master nameserver.

* The ReadyNAS runs a different program on port 53 on localhost
(the connection manager `connmand`), hence
I restrict the nameserver to listen on the explicit IP's port 53.

* Unlike the Docker nameserver, this one will get queries from the DMZ
host (the wireless router). It's IP is dynamically assigned by my ISP
(AT&T UVerse), hence I can't put it here explicitly, all I know is
the subnet range from which it will come.
