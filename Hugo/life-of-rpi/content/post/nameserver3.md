+++
Description = ""
Tags = ["RPIZero", "RPI3", "Docker", "HypriotOS", "DNS", "BIND"]
date = "2016-10-16T08:11:34-07:00"
title = "Setting up Nameservers Part 3 - Master Configuration"

+++
Moving on from my previous post describing the configuration
of my [nameserver slaves](/2016/10/14/nameserver2/ "Slave Configuration"),
this post describes the configuration of the nameserver master.

**named.conf.options for master nameserver**
```
key docker-ns1.home.nikolaischlegel.com {
        algorithm hmac-md5;
        secret "************************";
};

options {
        directory "/var/cache/bind";

        # Allow queries from 
        #  - local subnets (192.168.0 or 192.168.1)
        #  - Docker (172.17)
        #  - AT&T UVerse which will include my DMZ host (75.0.0.0/10)

        allow-query { 
            192.168.0.0/23;
            172.17.0.0/16;
            75.0.0.0/10;
        };
        recursion yes;

        # transfer requests could come either from secondary nameserver
        # or Docker Host which appears as DMZ host
        allow-transfer { 
            192.168.1.79;
            key docker-ns1.home.nikolaischlegel.com;
        };

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
        forward only;

        # only zone specific notifications
        notify no;

        dnssec-validation auto;
        auth-nxdomain no;    # conform to RFC1035
};
```

** named.conf.local for master nameserver**
```
zone "home.nikolaischlegel.com" {
        type master;
        file "/etc/bind/db.home.nikolaischlegel.com";
        forwarders {};
        notify explicit;
        also-notify { 192.168.1.79; };
};

zone "168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.192.168";
        forwarders {};
        notify explicit;
        also-notify { 192.168.1.79; };
};
```

Some comments about this configuration:

* I define the same TSIG key here as in the Docker slave nameserver
configuration, this key is then used to specify that the Docker slave
is authorized to do transfers from this master.

* In addition to the Docker slave, the NAS slave nameserver is also
authorized to do transfers, it is authorized by IP address rather than 
a key

* In the `named.conf.options` file, I disable all notify's globally, but
then in the `named.conf.local`, I enable it explicitly only for 
the NAS slave (the Docker slave doesn't get notified, it will eventually
come and query the master for zone updates on it's own).

* For the zones that the master is responsible for, lookups don't get
forwarded at all (since the hosts in these zones are only visible 
in my private network anyway).

**db.home.nikolaischlegel.com zone configuration**
```
;
; BIND data file for home.nikolaischlegel.com
;
$TTL    3h
@       IN      SOA     ns1.home.nikolaischlegel.com. admin.home.nikolaischlegel.com. (
                             18         ; Serial
                             3h         ; Refresh
                             1h         ; Retry
                             1w         ; Expire
                             1h )       ; Negative Cache TTL
; Nameserver records
            IN      NS      ns1.home.nikolaischlegel.com.
            IN      NS      nas.home.nikolaischlegel.com.

; Primary Nameserver
ns1         IN      A       192.168.1.77

; Secondary Nameserver (also NAS)
nas         IN      A       192.168.1.79

; other computers
dlinkm      IN      A       192.168.0.1
docker      IN      A       192.168.0.100   
...

; Aliases
wifi        IN  CNAME       dlinkm
...
```

**db.192.168 zone configuration**
```
;
; BIND reverse data file for local loopback interface
;
$TTL    3h
@       IN      SOA     ns1.home.nikolaischlegel.com. admin.nikolaischlegel.com. (
                             18         ; Serial
                             3h         ; Refresh
                             1h         ; Retry
                             1w         ; Expire
                             1h )       ; Negative Cache TTL
; Nameserver records
        IN      NS      ns1.home.nikolaischlegel.com.
        IN      NS      nas.home.nikolaischlegel.com.

; other computers
1.0     IN      PTR     dlinkm.home.nikolaischlegel.com.
100.0   IN      PTR     docker.home.nikolaischlegel.com.
...
```

Further comments:

* As mentioned before, I advertise the primary master
(on Raspberry PI Zero) and secondary slave (on NAS) nameserver
but don't advertise the slave on the Docker Raspberry.

* For each of my internal hosts (not all are listed), I have
one `A` record so there is a 1:1 mapping between host name
and IP, but then I define one or more aliases with `CNAME` records

* Be aware on how picky BIND is about trailing dots for 
host and domain names !
and 
