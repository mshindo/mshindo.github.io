---
date: 2008-07-09 13:34:28+00:00
layout: post
title: sFlow IPv4/6 packet type support for Wireshark
categories:
- コンピュータとインターネット
language:
- 日本語
---

久しぶりにWiresharkのコードをcontributeしました。

現在のWiresharkのsFlow dissectorはHeader Typeしかサポートしていません。多くのsFlowベンダーはHeader Typeしかサポートしていないのですが、AlaxalA/Hitachi/NECはきちんとサポートしているので、こいつをきちんとdissectしてあげたいな、と。

無事にcheck-inされることを祈ります。

久しぶりのpatchのsubmissionだったので、不手際があるかも。以前は単に開発用MLにパッチ投げればよかったんですけど、いまはbugzillaで管理されていて、そちらでbugをopenして、それに対してパッチを投げる、という手順になっていました。

まだまだいけていないWiresharkのsFlow dissectorですが、暇見つけて少しずつ直していこうと思います。
