---
date: 2014-06-30 21:12:33+00:00
layout: post
title: Geneve on Open vSwitch
categories:
- コンピュータとインターネット
- 仕事
language:
- 日本語
post_translations:
- pll_53b216f740c8f
tags:
- GENEVE
- Open vSwitch
---

[先日のBlog](https://blog.shin.do/2014/05/geneve-encapsulation)でGeneveという新しいEncapsulation方式について紹介をしましたが、さっそくOpen vSwitch（OVS）に実装されmasterにマージされましたので試してみました。今回はGeneveを検証する事が目的ですので、KVMなどは使わずに、単純に異なるホストにある2つのOVS BridgeをGeneveトンネルで繋いでみることにします。VMware FusionでUbuntu 14.04を2つ（host-1、host-2）動かし、それぞれgithubから取ってきた最新のOVSをインストールしました。各仮想マシンにはNATによるEthernetインターフェースを1つ設定し、DHCPでアドレスをとる事にします。以下の例ではそれぞれのホストで192.168.203.151と192.168.203.149というIPアドレスがDHCPで取れた場合の例です。Bridgeを2つ（br0、br1）作成し、br0を物理ネットワークと接続するブリッジとし、br1の間にGeneveのトンネルを張る事にします。

[![Geneve Test with Open vSwitch]({{site.baseurl}}/images/geneve-test.svg) Geneve Test with Open vSwitch

host-1, host-2でのOVSの設定は以下の通りです。

    
    mshindo@host-1:~$ sudo ovs-vsctl add-br br0
    mshindo@host-1:~$ sudo ovs-vsctl add-br br1
    mshindo@host-1:~$ sudo ovs-vsctl add-port bra eth0
    mshindo@host-1:~$ sudo ifconfig eth0 0
    mshindo@host-1:~$ sudo dhclient br0
    mshindo@host-1:~$ sudo ifconfig br1 10.0.0.1 netmask 255.255.255.0
    mshindo@host-1:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.149



    
    mshindo@host-2:~$ sudo ovs-vsctl add-br br0
    mshindo@host-2:~$ sudo ovs-vsctl add-br br1
    mshindo@host-2:~$ sudo ovs-vsctl add-port bra eth0
    mshindo@host-2:~$ sudo ifconfig eth0 0
    mshindo@host-2:~$ sudo dhclient br0
    mshindo@host-2:~$ sudo ifconfig br1 10.0.0.2 netmask 255.255.255.0
    mshindo@host-2:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.151


このような設定をすれば、host-1とhost-2のbr1間のpingが通るようになり、物理ネットワーク側にパケットが出る際にはGenveでトンネルされます。

    
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


Wiresharkでパケットを見てみましょう。Wiresharkにも2014/06/16にGeneveのdissectorが追加されていますので、最新のWiresharkをビルドすればGeneveパケットの中身を見る事ができます。

[![Geneve Frame by Wireshark]({{site.baseurl}}/images/geneve-300x207.png) Geneve Frame by Wireshark

Geneveが使うポート番号は6081/udpです（このポート番号は2014/03/27にIANAに登録されています）。また、今回のように単純にOVSのBridge同士を繋ぐだけであれば特にVNIは必要はありませんので、明示的にVNIの値を指定していません。その場合はVNIの値は0になります。VNIの値を明示的に設定したい場合は、

    
    mshindo@host-1:~$ sudo ovs-vsctl add-port br1 geneve1 -- set interface geneve1 type=geneve options:remote_ip=192.168.203.149 options:key=5000


というようにoptionsでkeyというパラメータで指定すればOKです。その際のパケットは以下のようになります（10進数の5000は16進数では0x1388）。

[![Geneve Frame with VNI 5000 by Wireshark]({{site.baseurl}}/images/geneve-vni5000-300x207.png) Geneve Frame with VNI 5000 by Wireshark

GeneveはEthernetフレームだけではなく、他のタイプのフレームもトンネルできるようになっています。そのためにProtocol Typeというフィールドが用意されています。今回の例ではEthernetフレームをトンネルしていますので、Protocol Typeの値はTransparent Ethernet Bridgingを示す0x6558が指定されています。

現時点の実装では、Geneveの特徴であるGeneve Optionsを指定する事はまだできません。追ってサポートが加えられると思います。

Geneveの真価が発揮されるのは、Geneveのフレームフォーマットを理解してTSOをすることができるNICが出現した場合です。そのようなNICは現時点ではまだ世の中には出てないですが、少なくともソフトウェア的（OVS的）にはGeneve Readyになりました。現在はまだmasterブランチにしかGeneveのコードが入っていなかったのでgithubの最新masterブランチから取ったコードをビルドして試しましたが、OVSのバージョン2.2がリリースされれば、今回試したコードも含まれてくることになるので、簡単に試す事ができるようになるはずです。
