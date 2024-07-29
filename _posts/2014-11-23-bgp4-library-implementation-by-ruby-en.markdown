---
date: 2014-11-23 01:55:38+00:00
layout: post
title: BGP4 library implementation by Ruby
description: Passive BGP4 implementation primarily for testing
image: '/images/victoria-priessnitz-EhPih0l5bjw-unsplash.jpg'
tags:
- Computer and Networking
language:
- English
---

A while back, I posted a simple BGP4 library implemented by Ruby to GitHub. I wrote this code when I was working at Fivefront, which was selling GenieATM, a NetFlow/sFlow/IPFIX collector.

[https://github.com/mshindo/bgp-server](https://github.com/mshindo/bgp-server)

Using this library, you can easily write a code that speaks BGP4 as shown below:

    #
    # example1.rb
    #
    require 'bgp-server'
    
    open = Open.new(7675, '172.16.167.1')
    openmsg = BgpMsg.new(open)
    
    origin = Origin.new
    pathseg1 = AsPathSegment.new([100, 101, 102])
    aspath = AsPath.new
    aspath.add(pathseg1)
    nexthop = NextHop.new('11.0.0.2')
    localpref = LocalPreference.new
    
    path_attr = PathAttribute.new
    path_attr.add(origin)
    path_attr.add(aspath)
    path_attr.add(nexthop)
    path_attr.add(localpref)
    
    nlri = Nlri.new(['10.0.0.0/8', '20.0.0.0/16'])
    update = Update.new(path_attr, nlri)
    updatemsg = BgpMsg.new(update)
    
    keepalive = KeepAlive.new
    keepalivemsg = BgpMsg.new(keepalive)
    
    bgp = Bgp.new
    bgp.start
    bgp.send openmsg
    puts "OPEN sent"
    bgp.send keepalivemsg
    puts "KEEPALIVE sent"
    
    bgp.send updatemsg
    puts "UPDATE sent"
    
    loop do
      sleep 30
      bgp.send keepalivemsg
      puts "KEEPALIVE sent"
    end


I wrote this code primarily because we were so "poor" at that time (we were a startup with just 5 people) and couldn't afford to buy a decent physical router such as Cisco or Juniper :-) Another reason was that we needed to inject large number of routes (e.g. 1,000,000 routes) to GenieATM, in which case Zebra did do a good job as it has to keep all routes in memory. These situations motivated me to write a stateless BGP code for route injection. 

Over time, I added more features as needed and the code became more generic. Sometimes it gave me a great help, for example, when I needed to debug an issue introduced by a subtle difference of how BGP4 Capability Announcement is made by Juniper and Force10 (now DELL). Juniper announces capability by listing multiple Capability Advertisement that includes only one capability. On the other hand, Force10 announces single Capability Advertisement that includes multiple capabilities. This BGP4 library was useful to simulate these two different behaviors.

Because of the intended use case of this library code, the abstraction level of BGP was set low intentionally. It could have been abstracted more, but then it would in turn loose the flexibility of crafting a BGP message arbitrarily. 

Please note that there are some limitations and constrains because this code was primarily developed to test GenieATM, a flow collector. First, while BGP session can be established from either peer in general, this code will never initiate BGP session from itself. Instead, it simply expects the peer to establish a session (a.k.a. passive mode). Second, no matter what messages are sent by the peer, it doesn't do anything with it. In essence, this BGP4 library aims to experiment how the peer BGP implementation behaves when various kinds of BGP messages are sent.

Not to mention, this code is not a complete implementation of BGP4. Compared to other complete BGP4 implementations like Zebra/Quagga, this code is just a toy. That said, I decided to make this code available to public hoping that someone who has a similar use case as mine may find this code useful. I would like to say thanks to Tajima-san, my colleague, who suggested me to publish this code to Github.
