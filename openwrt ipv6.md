---
title: raspberry pi enable ipv6 in HEU
date: 2016-03-06
---
#raspberry pi enable ipv6 in HEU

1.add fwllowing line at /etc/ppp/options
+ipv6

2.add fellowing lins at /etc/sysctl.conf
```
net.ipv6.conf.all.forwarding=1

net.ipv6.conf.eth0.accept_ra=2
net.ipv6.conf.ppp0.accept_ra=2
```
3.execute ```sysctl -p via sudo user```

4.add ```ip6tables -t nat -I POSTROUTING -s fc00:101:102::/64 -j MASQUERADE``` at anywhere that can excute once

5.edit /etc/network/interface set wlan0 to static ipv6 address fc00:101:102::1/64

6.install radvd and configure radvd.conf as fellows
```interface wlan0 # LAN interface
{
AdvManagedFlag off; # no DHCPv6 server here.
AdvOtherConfigFlag off; # not even for options.
AdvSendAdvert on;
AdvDefaultPreference high;
AdvLinkMTU 1280;
prefix ::/64 #pick one non-link-local prefix assigned to the interface and start advertising it
{
AdvOnLink on;
AdvAutonomous on;
};
};
```
5.done
