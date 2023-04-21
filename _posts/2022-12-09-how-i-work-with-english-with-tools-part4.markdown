---
date: 2022-12-09 00:05:47+00:00
layout: post
title: 私、英語とこんな付き合い方をしています（ツール編）〜 Part 4 〜
description: 各種機械翻訳の紹介と比較
image: '/images/tim-wildsmith-9hssKCV2HKY-unsplash.jpg'
tags:
- 仕事
language:
- 日本語
keywords:
- DeepL
- English
- Google Translate
- Study
- みらい翻訳
- 英語学習
---

## 翻訳ツール


[前回](https://blog.shin.do/2022/12/how-i-work-with-english-with-tools-part3/)は辞書サービスについて紹介をしました。今回は、翻訳ツールについてお話をします。翻訳ツールは皆さんも日々お世話になっていることと思います。一番よく知られているのは**Google翻訳**でしょう。5,000文字までならブラウザに翻訳したい文章を貼り付けるだけで翻訳をしてくれます。この冒頭のパラグラフをGoogle翻訳にかけると以下のようになります。

![]({{site.baseurl}}/images/Google-Translate-1024x460.png)

また、Webページをライブ（オンザフライ）で翻訳することもできます。この機能は結構便利です。英語圏の人に日本語のWebページを読んでもらいたいときに、Google翻訳したページのURLを送れば、英語で読んでもらうことができます。例えば、この記事を英訳したものを読みたければこの[URL](https://blog-shin-do.translate.goog/2022/12/how-i-work-with-english-with-tools-part4/?_x_tr_sl=auto&_x_tr_tl=ja&_x_tr_hl=ja&_x_tr_pto=wapp)をアクセスすると英語で読むことができます。

Google翻訳は便利なんですが、翻訳クオリティ的にはちょっとイマイチかなという感触です。Goolgle翻訳に代わるものとしては、その精度の高さから最近特に広く使われるようになっているのが**DeepL **（「ディープル」ではなく「ディープ・エル」と発音するのが正しいようです）というサービスです。Google翻訳同様、Webのインターフェースでテキストを貼り付けると翻訳をしてくれます。私は使ったことはないですが、WordやPowerPointのファイルを与えると、その中に含まれるテキストを翻訳してくれる機能などもあるようです。

![]({{site.baseurl}}/images/DeepL-1024x540.png)

DeepLにも無料のプランと有料のプランがあって、有料にすると文字数の制限がなくなったり、機密保持の観点から翻訳をかけたテキストを翻訳後に自動的に削除してくれる機能なども使えるようになります。パーソナルな使い方なら無料版でも十分な気がしますが、社内のメールやドキュメント書きに使う場合は、会社のポリシーを十分に確認したほうが良いと思います。

では、具体的にGoogle翻訳とDeepL、それぞれどれくらいの品質なのかを比べてみましょう。以下のような私の以前のブログ記事の冒頭部分を翻訳にかけてみましょう。日本語特有の主語の省略などがあって、かなり翻訳しにくい文章ではあるとは思います。

原文：「​日頃から仕事でお世話になっている方からこんな記事を教えてもらった。2004年の記事だが、当時IntelのCTOだったPat Gelsinger（現VMwareのCEO）がオーバーレイネットワークの構想を持っていたのには驚かされる。Patが思い描いていた世界は（当時彼がIntelにいたということを考えると）全てのデスクトップやノートブックPCにまでオーバーレイネットワークが伸びていき、そこで色々なネットワークサービス面に接続して使う、というような姿だったのではないかと想像する。」

まずはGoogle翻訳で英訳してみます。

Google翻訳： ​I was taught this kind of article by a person who usually takes care of me at work. In a 2004 article, it's surprising that Intel CTO Pat Gelsinger (now VMware CEO) had a vision for an overlay network. Pat's vision is that the overlay network will grow to all desktops and notebooks (given he was in Intel at the time), where he will connect and use various network service aspects. Imagine that he was like that.

どうでしょう？ 冒頭の "I was taught this kind of article by a person who usually takes care of me at work." はかなり違和感がありますね。中盤はそこそこですが、最後はまた "where he will connect and use various network service aspects." とか "Imagine that he was like that." などとなっていて意味が不明瞭です。

（後記：上記は2020年6月時点のGoogle翻訳の結果です。ちなみに2022年12月時点で同じ日本語をGoogle翻訳で翻訳してみると、

"I was told this article by someone who has been taking care of me on a daily basis. It's an article from 2004, but it's surprising that Intel's CTO at the time, Pat Gelsinger (now VMware's CEO), had the idea of an overlay network. The world Pat envisioned (given that he was at Intel at the time) would have an overlay network extending to all desktops and notebooks, where they would connect to and use various network services. I imagine that it was a figure."

となります。幾分マシになっていることがわかりますが、やはりまだ小慣れた訳にはなっていません。）

次に、同じ文章をDeepLで英訳すると以下のようになります。

DeepL翻訳： I learned about this article from someone who has been working with me on a regular basis. Pat Gelsinger, Intel's CTO at the time (now CEO of VMware) It's amazing that he had a vision for an overlay network, the world Pat envisioned. (given that he was at Intel at the time) every desktop and notebook PC, the overlay network will extend to the overlay network, connecting to the various network service fronts, and I imagine it was the kind of thing that was used by the people who used it.

後半はちょっと怪しいですが、それでもGoogle翻訳よりはかなりマシな英語になっています。特に、うまく主語が補われているところなどはGoogle翻訳よりかなり秀逸です。

（後記：上記のDeepLによる翻訳も2020年6月時点のものです。同じことを2022年12月時点でやってみると、

"I was told about this article by a business associate of mine, and I was surprised to learn that Pat Gelsinger (now CEO of VMware), then CTO of Intel, had a vision of overlay networking. I imagine that Pat envisioned a world where the overlay network would extend to all desktops and notebooks (given that he was at Intel at the time), where they could connect to various network service surfaces."

となりました。前半部分は2020年の時よりさらに洗練された感があります。一方、最後の部分「というような姿だったのではないかと想像する。」という部分の翻訳がバッサリ削られてしまっています。これはDeepLの特性（クセ）だと思うのですが、DeepLは訳しにく文章に出くわすと、その部分の翻訳をバッサリとスキップしてしまうことが多いです。変な訳ながらでも残してくれれば後で読み返したときに気づきやすいのですが、無視されてしまうとスキップされてしまった事に気づかないケースも出てきます。DeepLを使うときはこの部分に注意が必要です。）

このようにDeepLはかなり高い精度を持っていることがわかります。今回は英訳しずらい日本語を英訳対象にしたので不自然なところが若干残ってしまいましたが、論文の翻訳などについてはDeepLは非常に高い精度を持っていると言われています。

（後記：もう一つ、DeepLに引けを取らないとも言われている**みらい翻訳**というサービスを試してみましょう。同様の文章をみらい翻訳にかけてみると以下のようになりました。

みらい翻訳： I received this article from someone who has been helping me with my work. Back in 2004, it was surprising that then Intel CTO Pat Gelsinger (now CEO of VMware) had the idea for an overlay network. I imagine the world Pat envisioned (given that he was at Intel at the time) would be one where overlay networks would extend to every desktop and laptop, where they would connect to and use various aspects of network services.

いかがですか？　DeepLよりもさらにイケている気がしますよね！　これは期待の星かもしれません。）

翻訳サービスは機械学習技術の進化を直接的に享受できる分野なので、これからもますます進化を続けていくと思われます。何年かして上記と同じことをやってみて、各サービスの進化の度合いを見比べてみるのも面白いかもしれませんね。


## 次回


さて、翻訳サービスについては以上となります。[次回](https://blog.shin.do/2022/12/how-i-work-with-english-with-tools-part5/)は、リスニング編をお届けする予定です。

Photo by [Tim Wildsmith](https://unsplash.com/fr/@timwildsmith?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/translation?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
