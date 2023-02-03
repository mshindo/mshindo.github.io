---
date: 2020-03-08 07:44:32+00:00
layout: post
title: 自宅のインターネット回線品質をVeloCloudで測ってみた
image: '/images/joe-neric-HHunRG19kF8-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- flets
- internet
- nuro
- sd-wan
- ucom
- velocloud
---

[こちら](https://blog.shin.do/2020/02/nuro-has-come-to-my-house/)の記事で書いた通り、現在我が家には3つのインターネット回線が引かれている状態です。VeloCloudのEdgeは回線のlatency、jitter、packet lossを測って可視化してくれる機能を持っていますので、今回はこの機能を使って各回線の品質を調べてみました。

結果は以下の通りです（グラフをクリックすると拡大されます）。

ちなみに、"So-net" はNURO光、"NTT" はフレッツ光（OCN）、"ARTERIA Networks Corporation" はUCOMを表しています。VeloCloud EdgeにWAN回線を繋ぎこむと、WAN側のアドレスを認識して、そのアドレスから自動的にその回線の通信サービスロバイダ名をWANリンクのNameとして設定してくれるようになっていますので便利です（EdgeがNATの内側にあってもこの機能は動きます）。

フレッツ光（OCN）- Latency -![]({{site.baseurl}}/images/FLETS-Latency-1024x677.jpg)

フレッツ光（OCN）- Jitter -![]({{site.baseurl}}/images/FLETS-Jitter-1024x677.jpg)

フレッツ光（OCN）- Packet Loss -![]({{site.baseurl}}/images/FLETS-Packet-Loss-1024x693.jpg)

NURO光 - Latency -![]({{site.baseurl}}/images/NURO-Latency-1024x677.jpg)

NURO光 - Jitter -![]({{site.baseurl}}/images/NURO-Jitter-1024x677.jpg)

NURO光 - Packet Loss -![]({{site.baseurl}}/images/NURO-Pakcet-Loss-1024x693.jpg)

UCOM - Latency -![]({{site.baseurl}}/images/UCOM-Latency-1024x677.jpg)

UCOM - Jitter -![]({{site.baseurl}}/images/UCOM-Jitter-1024x677.jpg)

UCOM - Packet Loss -![]({{site.baseurl}}/images/UCOM-Packet-Loss-1024x693.jpg)

これらの結果をみて分かる通り、フレッツは想像通りインターネットが混雑する時間帯（21:00から0:00くらい）はLatency、Jitter共に大きくなっており、Packet Lossも若干見られます。前回のテスト結果で見られたような通信速度の低下を裏付ける形になっています。一方、NUROとUCOMは全ての時間帯で安定しており、Jitter、Packet Lossもほとんどありません。なお、現時点ではVeloCloud EdgeはIPv6のサポートをしていないので、これらの測定結果はIPv4のものです。IPv6で測れればフレッツ（OCN）はもっと良好な結果になったと思われます。

それぞれのグラフの縦軸のスケールが異なるので、パッと見比較しにくいと思いますので、Latency、Jitter、Packet Lossについて全ての回線を重ね合わせたグラフ（Stackグラフ）も参考のため載せておきます。色が全て青系の色なのでちょっと分かりにくいですが・・・。

3回線重ね合わせ - Latency - ![]({{site.baseurl}}/images/All-Latency-1024x677.jpg)

3回線重ね合わせ - Jitter -![]({{site.baseurl}}/images/All-Jitter-1024x677.jpg)

3回線重ね合わせ - Packet Loss -![]({{site.baseurl}}/images/All-Packet-Loss-1024x693.jpg)

Photo by [Joe Neric](https://unsplash.com/@jneric?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/speed?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
