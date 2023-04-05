---
layout: default
title: Setup DNS Forwarder/Resolver in Cloud Environment
parent: Linux
nav_order: 3
---

## How to Setup DNS Forwarder/Resolver in GCP

**Setup a new VM to install bind9**
```shell
#install bind9
sudo apt install bind9 bind9utils bind9-doc
```

```shell

sudo cat /etc/default/named
#
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4" # Use only IPv4 addresses
```

```shell
# Disable ipv6 dns resolution
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf

sysctl -p
```

```shell
# set hostname
 hostnamectl set-hostname test-1.play.local
```

### Bind9 DNS configuration
```shell
root@test-1:~# cat /etc/bind/named.conf.options
acl "trusted" {
	localhost;
	localnets;
        10.100.100.0/28;   # gke master authorized network
        10.0.0.0/22;  	   # kube subnetwork
        10.190.191.0/24;   # vm subnetwork

};

options {

        directory "/var/cache/bind";
        recursion yes;                 # enables resursive queries
        allow-query { trusted; };      # allows queries from "trusted" - referred to ACL
        allow-query-cache { trusted; };      # allows queries caching from "trusted" - referred to ACL
	listen-on {
		10.190.191.4;
	};
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
	querylog yes;
	dnssec-validation no;
        auth-nxdomain no;    # conform to RFC1037
};
```

```shell

root@test-1:~# cat /etc/bind/named.conf.local

########################
# Internal domain zone #
########################

zone "c.xxx-xxx-xxx.internal." {
    type forward;
    forward only;
    forwarders {
       169.254.169.254; #for private DNS zone NS
       216.239.32.108;  #for public DNS zone NS
       216.239.34.108;  #for public DNS zone NS
       216.239.36.108;  #for public DNS zone NS
       216.239.38.108;  #for public DNS zone NS
    };
};

###################
# play.local zone #
###################
zone "play.local.com." {
    type forward;
    forward only;
    forwarders {
       169.254.169.254;
       216.239.32.108;
       216.239.34.108;
       216.239.36.108;
       216.239.38.108;
    };
};


################
# Reverse zone #
################

zone "190.10.in-addr.arpa." {
    type forward;
    forward only;
    forwarders {
       169.254.169.254;
       216.239.32.108;
       216.239.34.108;
       216.239.36.108;
       216.239.38.108;
    };
};

zone "0.10.in-addr.arpa." {
    type forward;
    forward only;
    forwarders {
       169.254.169.254;
       216.239.32.108;
       216.239.34.108;
       216.239.36.108;
       216.239.38.108;

    };
};



```

### Setup VM IP as nameserver

```shell
# add ip of VM as a nameserver
root@test-1:~$ cat /etc/netplan/50-cloud-init.yaml
network:
    ethernets:
        ens4:
            dhcp4: true
            match:
                macaddress: 42:01:0a:be:bf:04
            set-name: ens4
            nameservers:
                addresses:
                - 10.190.191.4
                search: [ play.local.com ]
    version: 2


root@test-1:~$ resolvectl status
Link 2 (ens4)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 10.190.191.4
       DNS Servers: 10.190.191.4 169.254.169.254
        DNS Domain: c.XXX-XXX.internal europe-west3-c.c.XXX-XXX.internal google.internal play.local

```

### Verify domain hosted in cloud-dns

```shell
root@test-1:~# nslookup test-1.play.local
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	test-1.play.local
Address: 10.190.191.4

root@test-1:~# nslookup grafana.play.local
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	grafana.play.local
Address: 34.89.213.90
```