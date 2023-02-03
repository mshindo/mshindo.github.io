---
date: 2019-05-04 06:30:57+00:00
layout: post
title: Blogお引っ越し
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- wordpress all-in-one-wp-migration
---

長年の懸案であったこのBlogサイトの引っ越しを行いました。2011年よりさくらインターネットのVPSインスタンス上に立てたWordpressを使ってきましたが（さくらインターネットさん、長い間ありがとうございました！）、メモリ1GB、ディスク20GBという小さなインスタンスだったたので、（攻撃を受けた時など）度々Out of Memoryで落ちたり、運用的に苦労もしてました（最近のさくらさんは、ping監視してメールやSlackで通知してくれるサービスを提供してくれているようですね。知らなかったw）。

ただ、いまどき自分でBlogをマネージするのもイケてないなと思い、今回他のサービスへの乗り換えも検討しました。カスタムドメインをサポートしていたMedium.comが筆頭候補だったのですが、残念ながらMedium.comは最近カスタムドメインのサービスを廃止してしまったようです。その他、note、Qiitaなどのサービスも検討はしたものの、現サイトのコンテンツに対する他のサイトからのリンクなども結構あるので、できることなら記事のURLを変えたくなく、結局自宅のESXi上の仮想マシンにWordpressを立てる、という技術的にはあまり代わり映えのしないお引越しとなってしまいました。

なお、今回の引っ越しにはAll-in-One WP Migrationというプラグインを使いました。これを使うとほぼ全てのWordpressのデータとプラグインなどを含む環境を簡単にマイグレーションすることができて大変便利です。前の環境がPHP5系、新しい環境がPHP7系だったので少し心配でしたが、特に問題なく移行する事ができました。

ESXi環境になりスナップショットを気軽に取れるようになったので、いろいろ設定変更しやすい環境にはなりました。しばらくはこの構成で運用して行こうと思います。

という記事を新環境でポストしてみるテスト。

追伸：Simple Tweetが動かなくなってしまったようです。原因調査中。

(Photo by [Harshil Gudka](https://unsplash.com/photos/aKcVSSDotgo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash)](https://unsplash.com/search/photos/migration?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
