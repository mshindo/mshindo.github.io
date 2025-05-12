---
date: 2025-05-11 08:40:00+09:00
layout: post
title: Aruba Switchをお家Lab用に買ってみた
description: バーゲン品だったので、とりあえず1台購入
image: '/images/Aruba-1930.jpg'
tags:
- コンピュータ＆ネットワーク
keywords:
- おうちLab
- Aruba
- SFP+
language:
- 日本語
---

先日、たままた以下のようなXでのポストが目に止まりました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">SFP+ x4ポート、PoE+ にも対応しているマネージドL2スイッチの在庫が大量に補充されてる。お値段異常（新品、保証付きで14,080円）なので、10GbEデビューしようと思ってる人はとりあえず買っておけばいいと思います。<a href="https://t.co/96PFm46kem">https://t.co/96PFm46kem</a></p>&mdash; YAMAMOTO Masaya (@pandax381) <a href="https://twitter.com/pandax381/status/1904380250873909459?ref_src=twsrc%5Etfw">March 25, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


お、こりゃお得！　今まで使っていたBuffalo BS-GS2024 (24PortのGbEスイッチ)の代替ができたらいいかなー、と思い、おうちLab用のスイッチとして1台買ってみることにしました（あいにく現在は在庫切れになっています）。

BS-GS2024ををAruba 1930に置き換えることで得られる主なメリットは、

* SFP+のポートが4ポートある
* PoEが使える

という2点です。とりあえず今の自宅にはPoEのデバイスはないので、メリットはSFP+のポートのみ、ということになります。今までBS-GS2024と上位のQNAPの10GbEスイッチ(QSW-M1208-8C)とのTrunkは1GbEのポートでやっていたので、それが10GbE化できるだけでも大きなメリットです。また、QNAPスイッチの10GbEポート8つは全て使い切ってしまっているので、将来10GbEのデバイスが追加しようとした時にAruba側の10GbE SFP+ポートが使えるのはとても助かります。15,000円弱で10GbEのポートが4つ買える（おまけで1GbEポートも24個付いてくるw）と考えてもそう悪くない話です。

あまりに安かったのでちょっと心配でしたが、注文して2週間ほどでちゃんとモノが届きました。買ってみて気づいたことは、

* とても静か。ファンレスではないようですが、居住スペースにおいてもほとんど気にならないレベルです。
* いわゆるCloud Manageなネットワーク機器。アプリ/Webから管理します。イマドキですね。アプリの小さな画面でちまちま設定するのはちょっと不便ですが、日々触るののでもないので、まあそこは許容できます。Webからやれば普通に設定できます。
* SFP+にベンダーロックがかかっている！　

最後の点が想定外でした。手持ちのDACでQNAPスイッチとTrunkしようと思ったら、ケーブル刺した途端、機器のLEDにアラートが点灯。。CLIのある Arubaスイッチであればallow-unsupported-transceiverというコマンドで凌げるようなのですが、あいにく1930はCloud ManageでCLIはないので逃げようがありません。

「仕方ない、おとなしくAruba純正のDACを買うか・・」、と諦めモードでAmazonを探してみたところ、何と純正のDACは14,000円くらいするではありませんか！　スイッチ本体が14,000円なのにケーブル一本に同じ値段、というのはどうにも納得がいきません（笑）。せっかくのお買い得感も台無しです。ということで、ものは試しで以下の[互換品](https://www.amazon.co.jp/dp/B0DLGKJ5RS/)と思われる商品を買ってみることにしました。

{: align="center"}
![]({{site.baseurl}}/images/Aruba-compatible-DAC.jpg)
*Amazon.co.jpより引用*

2,000円くらいですし、ダメでも人柱！と諦められるかなと思い試してみたら、何の問題もなく使えました。スイッチに点灯していたアラートも消えました。めでたしめでたし！ ちと汚いですが、こんな感じになってます。

![]({{site.baseurl}}/images/Lab-Equipment.jpg)

と、ここで今回の話は終わるはずだったのですが、後日談がありまして。。。先日ネットを徘徊していたら、HPEの[コミュニティーページ](https://community.instant-on.hpe.com/communities/community-home/digestviewer/viewthread?GroupId=7&MessageKey=25ce08ba-1eb3-4f66-baa5-73e32aae59bb&CommunityKey=84902a4b-1c23-44a8-8387-5ef7a53d4a4a)にHPE/Aruba社員の方が、

> We identify unsupported transceivers (SX, LH, etc…) by flashing the port LED and syslog message but don't block or disable the port.

というコメントを書かれているのを見つけまして。。。実際試してみたら、サードパーティ製のDACでも問題なく動きました！　アラートが点灯したので、てっきりサードパーティ製のSFP+ではダメなのかと早とちりしてしまったのでした。さて、2,000円でアラートを消せるのは安いのか、高いのか、意見が分かれるところかな、と思います。ま、色々知見が得られたので、私的には満足しています。

当初は、BS-GS2024を置き換える目論みだったのですが、結局BS-GS2024も残して使い続けることにしました。おかげさまでだいぶポートにゆとりができて幸せになりました。今回購入したAruba Instant Onスイッチシリーズ、せっかくのCloud Manageで集中管理できる機器なので、また安く出たら買ってみてもいいかなー、と思っています。その時こそ本当にBS-GS2024君には退役いただくかな。