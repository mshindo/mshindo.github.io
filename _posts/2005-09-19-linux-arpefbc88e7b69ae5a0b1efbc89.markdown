---
date: 2005-09-19 02:58:02+00:00
layout: post
title: Linux ARP（続報）
categories:
- コンピュータとインターネット
language:
- 日本語
---

気になったので、freebsd-net メーリングリストで以下のように聞いてみました。

    
    On FreeBSD (and I guess most Operating Systems as well), ARP reply is sent back only when the target IP address in ARP request matches with one of the IP addresses assigned to the interface through which the ARP Request is received. In contrast, on Linux (by default), it responds as long as the target IP address in ARP Request matches with any "local" IP address on the system, which is not necessarily an IP address assigned to the interface through which the ARP request is received.
    Is there any advantage/disadvantage in ARP implementation on FreeBSD over that of Linux? Thanks.
    Regards,
    Motonori Shindo


で、いくつかコメントが返ってきました。

    
    Motonori Shindo wrote:
    > On FreeBSD (and I guess most Operating Systems as well), ARP reply is
    > sent back only when the target IP address in ARP request matches with
    > one of the IP addresses assigned to the interface through which the
    > ARP Request is received.
    This is correct behavior.  Normally, you should only be able to ARP an IP address which is on an interface connected to that subnet.
    > In contrast, on Linux (by default), it responds as long as the target
    > IP address in ARP Request matches with any "local" IP address on the
    > system, which is not necessarily an IP address assigned to the
    > interface through which the ARP request is received.
    This sounds like "proxy ARPing" is enabled by default on your particular flavor of Linux.  I don't think they all do that, hopefully, any more than ipforwarding should be enabled by default just because a machine has two NICs.
    > Is there any advantage/disadvantage in ARP implementation on FreeBSD
    > over that of Linux? Thanks.
    This information disclosure could potentially be a security problem, if Linux is providing the MAC address of a NIC not connected to the subnet without being explicitly configured to do so...although in practice very few people actually implement layer-2 security measures.
    --
    -Chuck


とか、

    
    What Motonori Shindo described is actually the default behaviour for Linux kernels (at least my 2.6.8-kernel does it by default). It could be seen as a sort of proxy-arp, but only for the host itself, not other systems. Let me try to describe when it happens. Say you have
    192.168.42.42 bound on eth0 and have eth1 connected to some ethernet LAN. When a host on that eth1-connected LAN sends an 'arp who-has 192.168.42.42', a Linux system will answer that arp-request with it's
    eth1 MAC-address, although the IP-address is bound on eth0 and the arp request comes in on eth0. FreeBSD obviously doesn't do this.
    >> Is there any advantage/disadvantage in ARP implementation on FreeBSD
    >> over that of Linux? Thanks.
    I was unhappily surprised by this 'feature'. I find it pretty counter-intuitive. I expect two interfaces to be seperated inside a kernel, but Linux more or less binds them together. Incoming traffic on the 'wrong' interface will gladly be accepted, too. This broke things for me, because I didn't want to have that certain IP-address accessible.
    That said, this happens only when you have two interfaces connected to the same subnet, which is a bit evil anyhow. It may be beneficial for Linux to do things this way, perhaps for redundancy-purposes (two interfaces, one IP-address, IP reachable over both interfaces, when one fails, the other takes over.. no idea if that works out-of-the-box).
    --
    Pieter


とか

    
    The linux design philosophy, based on postings from various implementors, is that ip addresses are bound to a host, not to a particular interface.  I believe the arp behaviour reflects this.
    Sam


てな感じです。最初のChuckのコメントはちょっとピントはずしてますが、PieterやSamはなかなかポイント突いてきてます。

ルータの中には、Linuxと同様「IPアドレスはインターフェースに振るものではなく、ホストに振るものである」的な考えで実装されているものがあります。便利なこともありますが、ちょっと直感的ではない部分もありますね。特に、おおかたのHost-basedなTCP/IPの実装（たとえば巷のUNIX-flavorなOS(除くLinux)やWindows）に馴れている者にとっては、奇異な感じがします。が、Linuxのような実装でも、おおかた問題はなさそうです。

このようなARPの振る舞いを調べると、システムがLinuxベースなのかそうでないのかを判断することができそうです。興味深いsignatureの一つですね。
