---
date: 2014-07-01 07:48:50+00:00
layout: post
title: Geneve on Open vSwitch
categories:
- Computer and Networking
language:
- English
post_translations:
- pll_53b216f740c8f
tags:
- GENEVE
- Open vSwitch
---

A few weeks back, I posted a blog (sorry it was done only in Japanese) about a new encapsulation called “Geneve” which is being proposed to IETF as an Internet-Draft. Recently the first implementation of Geneve became available for Open vSwitch (OVS) contributed by Jesse Gross, a main author of Geneve Internet Draft, and the patch was upstream to a master branch on github where the latest OVS code resides. I pulled the latest source code of OVS from github and played with Geneve encapsulation. The following part of this post explains how I tested it. Since this effort is purely for testing Geneve and nothing else, I didn’t use KVM this time. Instead I used two Ubuntu 14.04 VM instances (host-1 and host-2) running on VMware Fusion with the latest OVS installed. In terms of VMware Fusion configuration, I assigned 1 Ethernet NIC on each VM which obtains an IP address from DHCP provided by Fusion by default. In the following example, let’s assume that host-1 and host-2 obtained an IP address 192.168.203.151 and 192.168.203.149 respectively. Next, two bridges are created (called br0 and br1), where br0 connecting to the network via eth0 while br1 on each VM talking with each other using Geneve encapsulation.

[![Geneve Test with Open vSwitch](http://blog.shin.do/wp-content/uploads/2014/07/geneve-test.svg)](http://blog.shin.do/wp-content/uploads/2014/07/geneve-test.svg) Geneve Test with Open vSwitch

OVS configuration for host-1 and host-2 are shown below:

    
    mshindo@host-1:~$ sudo ovs-vsctl add-br br0
    mshindo@host-1:~$ sudo ovs-vsctl add-br br1
    mshindo@host-1:~$ sudo ovs-vsctl add-port br0 eth0
    mshindo@host-1:~$ sudo ifconfig eth0 0
    mshindo@host-1:~$ sudo dhclient br0
    mshindo@host-1:~$ sudo ifconfig br1 10.0.0.1 netmask 255.255.255.0
    mshindo@host-1:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.149



    
    mshindo@host-2:~$ sudo ovs-vsctl add-br br0
    mshindo@host-2:~$ sudo ovs-vsctl add-br br1
    mshindo@host-2:~$ sudo ovs-vsctl add-port br0 eth0
    mshindo@host-2:~$ sudo ifconfig eth0 0
    mshindo@host-2:~$ sudo dhclient br0
    mshindo@host-2:~$ sudo ifconfig br1 10.0.0.2 netmask 255.255.255.0
    mshindo@host-2:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.151


Once this configuration has been done, now ping should work between br1 on each VM and those ping packets are encapsulated by Geneve.

    
    mshindo@host-1:~$ ping 10.0.0.2
    PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
    64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.759 ms
    64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.486 ms
    64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.514 ms
    64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.544 ms
    64 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=0.527 ms
    ^C
    --- 10.0.0.2 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 3998ms
    rtt min/avg/max/mdev = 0.486/0.566/0.759/0.098 ms
    mshindo@host-1:~$


Let’s take a closer look how Genve encapsulated packets look like using Wireshark. A Geneve dissector for Wireshark became available recently (this is also a contribution from Jesse, thanks again!) and merged into the latest master branch. Using this latest Wireshark, we can see how Geneve packet looks like as follows:

[![Geneve Frame by Wireshark](http://blog.shin.do/wp-content/uploads/2014/07/geneve-300x207.png)](http://blog.shin.do/wp-content/uploads/2014/07/geneve.png) Geneve Frame by Wireshark

As you can see, Geneve uses 6081/udp as its port number. This is a port number officially assigned by IANA on Mar.27, 2014. Just to connect two bridges together by Geneve tunnel, there’s no need to specify a VNI (Virtual Network Identifier) specifically. If VNI is not specified, VNI=0 will be used as you can see in this Wireshark capture.

On the other hand if you need to multiplex more than 1 virtual networks over a single Geneve tunnel, VNI needs to be specified. In such a case, you can designate VNI using a parameter called “key” as an option to ovs-vsctl command as shown below:

    
    mshindo@host-1:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.149 options:key=5000


The following is a Wireshark capture when VNI was specified as 5000 (0x1388):

[![Geneve Frame with VNI 5000 by Wireshark](http://blog.shin.do/wp-content/uploads/2014/07/geneve-vni5000-300x207.png)](http://blog.shin.do/wp-content/uploads/2014/07/geneve-vni5000.png) Geneve Frame with VNI 5000 by Wireshark

Geneve is capable of encapsulating not only Ethernet frame but also arbitrary frame types. For this purpose Geneve header has a field called “Protocol Type”. In this example, Ethernet frames are encapsulated so this filed is specified as 0x6558 meaning "Transparent Ethernet Bdiging".

As of this writing, Geneve Options are not supported (more specifically, there is no way which Geneve Options to be added to Geneve header). Please note that Geneve Options are yet to be defined in Geneve's Internet Draft. Most likely a separate Internet Draft will be submitted to define Geneve Options sooner or later. As such a standardization process progresses, Geneve implementation in OVS will also evolve for sure.

Although Geneve-aware NIC which can perform TSO against Geneve encapsulated packets is not available on the market yet, OVS is at least "Geneve Ready” now. Geneve code is only included in the latest master branch of OVS at this point, but it will be included in the subsequent official release of OVS (hopefully 2.2). When that happens you can play with it more easily. Enjoy!
