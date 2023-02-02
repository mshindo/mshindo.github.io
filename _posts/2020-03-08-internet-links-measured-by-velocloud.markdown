---
date: 2020-03-08 07:44:32+00:00
layout: post
title: 自宅のインターネット回線品質をVeloCloudで測ってみた
categories:
- コンピュータとインターネット
language:
- 日本語
tags:
- flets
- internet
- nuro
- sd-wan
- ucom
- velocloud
---

[こちら](http://blog.shin.do/2020/02/nuro-has-come-to-my-house/)の記事で書いた通り、現在我が家には3つのインターネット回線が引かれている状態です。VeloCloudのEdgeは回線のlatency、jitter、packet lossを測って可視化してくれる機能を持っていますので、今回はこの機能を使って各回線の品質を調べてみました。

結果は以下の通りです（グラフをクリックすると拡大されます）。

ちなみに、"So-net" はNURO光、"NTT" はフレッツ光（OCN）、"ARTERIA Networks Corporation" はUCOMを表しています。VeloCloud EdgeにWAN回線を繋ぎこむと、WAN側のアドレスを認識して、そのアドレスから自動的にその回線の通信サービスロバイダ名をWANリンクのNameとして設定してくれるようになっていますので便利です（EdgeがNATの内側にあってもこの機能は動きます）。

フレッツ光（OCN）- Latency -![](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Latency-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Latency.jpg)

フレッツ光（OCN）- Jitter -![](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Jitter-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Jitter.jpg)

フレッツ光（OCN）- Packet Loss -![](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Packet-Loss-1024x693.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/FLETS-Packet-Loss.jpg)

NURO光 - Latency -![](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Latency-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Latency.jpg)

NURO光 - Jitter -![](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Jitter-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Jitter.jpg)

NURO光 - Packet Loss -![](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Pakcet-Loss-1024x693.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/NURO-Pakcet-Loss.jpg)

UCOM - Latency -![](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Latency-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Latency.jpg)

UCOM - Jitter -![](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Jitter-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Jitter.jpg)

UCOM - Packet Loss -![](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Packet-Loss-1024x693.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/UCOM-Packet-Loss.jpg)

これらの結果をみて分かる通り、フレッツは想像通りインターネットが混雑する時間帯（21:00から0:00くらい）はLatency、Jitter共に大きくなっており、Packet Lossも若干見られます。前回のテスト結果で見られたような通信速度の低下を裏付ける形になっています。一方、NUROとUCOMは全ての時間帯で安定しており、Jitter、Packet Lossもほとんどありません。なお、現時点ではVeloCloud EdgeはIPv6のサポートをしていないので、これらの測定結果はIPv4のものです。IPv6で測れればフレッツ（OCN）はもっと良好な結果になったと思われます。

それぞれのグラフの縦軸のスケールが異なるので、パッと見比較しにくいと思いますので、Latency、Jitter、Packet Lossについて全ての回線を重ね合わせたグラフ（Stackグラフ）も参考のため載せておきます。色が全て青系の色なのでちょっと分かりにくいですが・・・。

3回線重ね合わせ - Latency - ![](http://blog.shin.do/wp-content/uploads/2020/03/All-Latency-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/All-Latency.jpg)

3回線重ね合わせ - Jitter -![](http://blog.shin.do/wp-content/uploads/2020/03/All-Jitter-1024x677.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/All-Jitter.jpg)

3回線重ね合わせ - Packet Loss -![](http://blog.shin.do/wp-content/uploads/2020/03/All-Packet-Loss-1024x693.jpg)](http://blog.shin.do/wp-content/uploads/2020/03/All-Packet-Loss.jpg)

Photo by [Joe Neric](https://unsplash.com/@jneric?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/speed?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
