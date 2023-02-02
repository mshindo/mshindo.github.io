---
date: 2017-06-19 15:00:42+00:00
layout: post
title: NAT still can be tricky
categories:
- Computer and Networking
language:
- English
post_translations:
- pll_594789e5bcf97
---

The following discussion applies to both NAPT and NAT. Hence I refer to it simply as NAT hereafter.

Viptela provides a way to configure the authentication method for IPsec, as you can see below:

    
    security
     ipsec
      authentication-type ah-sha1-hmac sha1-hmac ah-no-id
     !
    !
    


ah-sha1-hmac and sha1-hmac must be self-explanatory, but what the hell is ah-no-id?

In general, IPsec AH won't work if there is a NAT somewhere between the endpoints, as the fields that NAT would overwrite are protected by AH. That said, AH can still be used in Viptela's solution even if NAT is present because the architecture allows determining the IP address and port number before and after the translation by NAT. This works as long as the overwritten fields by NAT are limited to IP address and port number. Still, it will break if NAT overwrites other fields, such as the Identification field (I simply call it "ID field" hereafter) in the IP header. "ah-no-id" parameter is available to work around this situation by ignoring the ID field when computing AH.

In the beginning, I was skeptical that such a NAT implementation that overwrites the ID field really exists. Still, it turned out that there are a few NAT implementations that specifically do this, notably Apple AirMac Extreme and AirMac Express. Luckily, I had AirMac Express handy, so I did some tests by

    
    $ sudo hping3 -c 1 -N 12345 -S -s 11111 -p 22222 8.8.8.8


and compare the packet before and after NAT translation (please note that SYN flag by -S option is set here because most NAT implementation is "stateful" and simply drops the packet if SYN packet doesn't proceed).

Before NAT Translation （AirPort Express）:

    
    06:21:06.670952 IP (tos 0x0, ttl 64, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    10.0.1.3.11111 > 8.8.8.8.22222: Flags [S], cksum 0xa76d (correct), seq 868543788, win 512, length 0


After NAT Translation （AirPort Express）:

    
    15:20:52.361257 IP (tos 0x0, ttl 63, id 14428, offset 0, flags [none], proto TCP (6), length 40)
    10.156.250.80.44210 > 8.8.8.8.22222: Flags [S], cksum 0x2c38 (correct), seq 868543788, win 512, length 0


As you can see, the ID field is overwritten from 12345 to 14428.

Let's compare this with other NAT implementations. I took VyOS as an example.

Before NAT Translation （VyOS）:

    
    08:37:57.157399 IP (tos 0x0, ttl 64, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    10.200.1.11.11111 > 8.8.8.8.22222: Flags [S], cksum 0x5623 (correct), seq 7492057, win 512, length 0


After NAT Translation （VyOS）:

    
    08:37:57.157410 IP (tos 0x0, ttl 63, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    221.245.168.210.11111 > 8.8.8.8.22222: Flags [S], cksum 0xdb2d (correct), seq 7492057, win 512, length 0


In this case, the ID field (12345) is kept intact before and after the NAT translation.

Up until recently, I've believed that no intervening device should modify the ID field as this field is used for reassembling, which will only happen at the ultimate destination of the IP datagram and hence NAT implementation that rewrites the ID field is considered to be "misbehaving" if not "violating." This changed after I read [RFC6864](https://tools.ietf.org/html/rfc6864), "Updated Specification of the IPv4 ID Field".

This RFC classifies datagrams into two kinds:



 	Atomic

 	    Datagrams that are not yet fragmented and for which further fragmentation has been inhibited.

 	Non-Atomic

 	    Datagrams that are either already been fragmented or for which fragmentation remains possible.


Historically, the ID field had other usages than reassembling (e.g., de-duplication). This RFC clearly limits the use of the ID field only for fragmentation and reassembling and relaxing some requirements imposed on the ID field as it has no meaning for atomic datagrams. However, for the non-atomic datagram, this RFC still mandates (just like RFC791 does) that the value in the ID field maintains the uniqueness within MDL (Max Datagram Lifetime, typically 2 minutes).

This requirement puts a challenge on middle-box like NAT. Specifically, this RFC says:


<blockquote>NATs/ASMs/rewriters present a particularly challenging situation for
fragmentation. Because they overwrite portions of the reassembly
tuple in both directions, they can destroy tuple uniqueness and
result in a reassembly hazard. Whenever IPv4 source address,
destination address, or protocol fields are modified, a
NAT/ASM/rewriter needs to ensure that the ID field is generated
appropriately, rather than simply copied from the incoming datagram.

Specifically:
>> Address-sharing or rewriting devices MUST ensure that the IPv4 ID
field of datagrams whose addresses or protocols are translated
comply with these requirements as if the datagram were sourced by
that device.</blockquote>


Now I fully agree with this RFC; to guarantee the uniqueness of the value of the ID field, whenever a device like NAT rewrites an IP address, it should generate a unique ID as if the packet was sent from that device and should not simply copy the ID field from the original packet. Although the majority of NAT implementations today would behave like VyOS based on my past experience, I think Apple AirMac Extreme/Express is more compliant with the specification and well-behaving.

Sorry Apple, I've been treating you bad...
