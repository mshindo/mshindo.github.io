---
date: 2017-06-18 22:41:27+00:00
layout: post
title: NATは意外と難しい
description: 一見奇妙に見える Apple の AirMax Express の NAT の振る舞いは実は真っ当だった！
image: '/images/edurne-tx-5Z8mR4vqJD4-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- NAT
---

以下の議論はNATでもNAPTでも成り立つので、どちらの場合でも単にNATと書くことにします。

ViptelaのIPsecの認証方法はauthentication-typeというパラメータで指定することができます。例えばこんな感じです。

    
    security
     ipsec
      authentication-type ah-sha1-hmac sha1-hmac ah-no-id
     !
    !
    
ah-sha1-hmac、sha1-hmacはほぼ自明だと思いますが、ah-no-idという見慣れない設定は何のためにあるのでしょう？

一般的にはNATが途中にあるとAHは機能しません。NATが書き換えるフィールドがAHで守られているからです。しかし、Viptelaのアーキテクチャでは、仮にNATが途中にあっても変換前と後のIPアドレス、ポート番号をシステムが把握しているため、NAT環境であってもAHを使用することができるようになっています。ただし、NATで書き換えられるのがIPアドレスとポート番号だけであれば良いのですが、一部のNATの中にはIPヘッダのIdentificationフィールド（以下、IDフィールド）を書き換えるものがあります。ah-no-idというパラメータはこのようなNATデバイスがある場合に使用するもので、AHによる検証をする際にIDフィールドを無視する、という意味になります。

世の中のNATにIDフィールドを書き換えるNATがいるのか最初は疑っていたのですが、実際このようなNAT実装は存在するようで、少なくともApple AirMac ExtremeやAirMac Expressはそのように動くようです。実際、手元にあったAirMac Expressで試してみました。

    $ sudo hping3 -c 1 -N 12345 -S -s 11111 -p 22222 8.8.8.8

のようにパケットを出してみて、NATされる前と後のパケットを比較してみました（ちなみに-SオプションをつけてSYNフラグを付けているのは、NATの実装の多くはTCPのステートをみていて、SYNパケットがくる前にパケットが来てもトランスレーションせずに廃棄するからです）。

NAT変換前（AirPort Express）：
    
    06:21:06.670952 IP (tos 0x0, ttl 64, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    10.0.1.3.11111 > 8.8.8.8.22222: Flags [S], cksum 0xa76d (correct), seq 868543788, win 512, length 0

NAT変換後（AirPort Express）：

    15:20:52.361257 IP (tos 0x0, ttl 63, id 14428, offset 0, flags [none], proto TCP (6), length 40)
    10.156.250.80.44210 > 8.8.8.8.22222: Flags [S], cksum 0x2c38 (correct), seq 868543788, win 512, length 0

このように確かにIDフィールドが12345から14428に変換されています。

比較のためにVyOSのNATを使って同じように試験をしてみましょう。

NAT変換前（VyOS）：
    
    08:37:57.157399 IP (tos 0x0, ttl 64, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    10.200.1.11.11111 > 8.8.8.8.22222: Flags [S], cksum 0x5623 (correct), seq 7492057, win 512, length 0

NAT変換前（VyOS）：

    08:37:57.157410 IP (tos 0x0, ttl 63, id 12345, offset 0, flags [none], proto TCP (6), length 40)
    221.245.168.210.11111 > 8.8.8.8.22222: Flags [S], cksum 0xdb2d (correct), seq 7492057, win 512, length 0

こちらはNAT変換前も変換後もIDフィールドは12345のままです。

今まで私はIDフィールドはデータグラムがフラグメントされた際のリアセンブルに使用されるためのものなので、途中の機器で書き換えられるべきではない、と考えていました。したがって、このようにIDフィールドまで書き換えるNATはかなり特殊で、いわゆるキワモノ（変態）であると思っていたのですが、先日 [RFC6864](https://tools.ietf.org/html/rfc6864) "Updated Specification of the IPv4 ID Field" を見つけて読んでみると、この考えが変わりました。

このRFCではIPデータグラムをAtomicなものとNon-Atomicなものに分類して考えています。

 	Atomicなデータグラム

 	    まだフラグメントされておらず、その後もフラグメントされることのない（フラグメントが禁じられている）データグラム

 	Non-Atomicなデータグラム

 	    すでにフラグメントされているか、今後フラグメントされる可能性のあるデータグラム

歴史的にはIDフィールドはリアセンブル以外の目的（例えば重複検知、など）での使い方も考えられていたものの、このRFCでは改めてIDフィールドはリアセンブル以外の目的で使うことを禁止することを明確化し、それに伴いAtomicなデータグラムに対してはIDフィールドに関する要求条件を緩めています（AtomicなデータグラムにIDフィールドは不必要なため）。しかしながら、Non-Atomicなデータグラムに対しては依然RFC791で規定されている「MDL (Max Datagram Lifetime、典型的には2分）の間は同じIDを使い廻してはいけない」という要求をしています。

これはNATのようないわゆるミドルボックスにはかなりややこしい問題です。本RFCでも、

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
that device.

（拙訳）NAT/ASM (Address-Sharing Mechanism)/rewriterはフラグメンテーションに困難をもたらす。両方向においてリアセンブルするためのタプルの一部を書き換えるため、タプルのユニーク性を担保することができなくなり、リアセンブルができなくなるからである。IPv4送信元、送信先アドレスまたはプロトコルフィールドが書き換えられる場合は、NAT/ASM/rewriterは単に元のデータグラムのIDフィールドをコピーするのではなく、IDフィールドが正しく生成されることを保障しなければならない。

具体的には、Address-sharingまたはrewriterデバイスがIPv4データグラムのアドレスやプロトコル番号を書き換えた場合は、IDフィールドが、あたかもデータグラムがそのデバイスから出たかのごとく、これらの要求条件を満たすようにIDフィールドを使わなければいけない。</blockquote>

と書かれています。確かによく考えると、送信元、送信先アドレス、プロトコルのタプルでのIDフィールドのユニーク性を担保するためには、アドレスを書き換えるNATデバイスは、元のパケットのIDフィールドを単純にコピーするべきではなく、あたかも自分が出したパケットかのようにユニーク性を担保したIDを新たに払い出す必要があります。経験的には多くのNATデバイスはVyOS（多くのLinuxベースのもの同様でしょう）のように動くように思いますが、Apple AirMac Extreme/Expressの動作の方が正しい（というか素性がよろしい）ということになります。

Appleさん、今まで変態扱いしてごめんなさい。
