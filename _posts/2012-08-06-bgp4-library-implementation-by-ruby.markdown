---
date: 2012-08-06 22:02:00+00:00
layout: post
title: RubyによるBGP4ライブラリの実装
description: テスト用途のパッシブな BGP4 を作ってみた
image: '/images/victoria-priessnitz-EhPih0l5bjw-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
---

前職にいた時に書いた拙作のBGPのライブラリコードをGitHubに公開する事にしました。

[https://github.com/mshindo/bgp-server](https://github.com/mshindo/bgp-server)

このライブラリを使うと簡単にBGP4を喋るコードを書く事ができます。例えば以下のような感じ。

    
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


このようなコードを書いたきっかけは、ま、ちゃんとしたルータを買うお金がなかったってこともありましたが（笑）、大量の経路をGenieATMというBGPを喋るフローコレクターに注入する必要があったからです。最初はzebraを使って試そうと思ったのですが、さすがに100万経路くらいになると、いったんメモリに全経路を読み込まなければいけないzebraでは厳しくて（貧弱なマシンを使ってましたし）、じゃ、経路を注入するコードを書いちゃおう、と思い立ったのでした。

その後、いろいろなBGP関連の試験をする必要が出てきて、より汎用的なコードになっていきました。例えばJuniperとForce10のCapabilityの送り方が異なる事によるデバッグなどに役に立ちました。JuniperはOptional Parameterの中にCapability Advertisementを複数入れて、それぞれのCapability Advertisementの中に1つずつCapabilityが入っている形で広告されます。一方、Force10のほうはCapability Advertisementは1つだけで、その中に複数のCapabilityが並んだ形で広告されます。このような異なったBGPの実装をシミュレートするには、このコードは大変役に立つと思います。

このような経緯で生まれたコードなので、本コードはBGPの抽象度が少々低く実装されています。もう少し抽象度の高くする事もできたと思うのですが、そうするとBGPメッセージを組み立てる自由度が犠牲になるのでこのような形になっています。

また、このコードを使ってテストをする対象が主にGenieATMだったために、いくつか制限・制約があります。まず、本コードはBGPセッションを自分からは張りにいきません。相手からセッションが張られる事を期待しています（BGP「サーバー」の実装）。また、相手からBGPメッセージが来てもそれを読む事は基本せずに、自分からメッセージを送る事しかしません。いろいろなメッセージを送ってピアのBGPスピーカーの挙動を調べることに主眼を置いているためにこのようになっています。

もちろん、BGPの全ての機能を実装しているわけではありませんし、zebraに比べたら１万分の一の事もしていない「超なんちゃってBGP実装」ですから、あまり有用でないかも知れませんが、世の中の物好きな人に少しでも役に立てば、と思い、恥ずかしながら公開する事にしました。公開に関して私の背中を押してくれた田島＠アイマトリックスさんに感謝をしたいと思います。

Photo by <a href="https://unsplash.com/@victoriapriessnitz?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Victoria Priessnitz</a> on <a href="https://unsplash.com/photos/red-and-silver-beaded-necklace-EhPih0l5bjw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  