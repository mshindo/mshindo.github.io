---
date: 2023-02-19 11:36:35+00:00
layout: post
title: WordPressからJekyllへのお引越し
image: '/images/robinson-greig-HrnAxAUwle8-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- Wordpress
- Jekyll
- GitHub Pages
---
個人ブログのプラットフォームを長年使ってきたWordPressからJekyll (GitHub Pages)に引っ越しました。

## 背景

今まで個人ブログは自宅のESXiサーバ上の仮想マシンでWordPressを動かしていたのですが、今回家のリフォームに伴い仮住まいに引っ越しをすることになり、WordPress環境もどこかに移設しなければいけなくなった、というのが今回のブログプラットフォーム変更の直接の契機です。

作戦としては、ざっと

1. WordPressをどこかのパブリッククラウドで動かして凌ぐ
2. WordPress自体をやめて別のプラットフォームへ移行する

の2つが考えられます。一番楽なのは案1ですが、もともとWordPressにはいくつか課題感を感じていたので、この機に何か別のプラットフォームに引っ越そうと思い、案2で行くことにしました。

WordPressはとてもよくできたブログプラットフォームではあるのですが、

- アタックサーフェイスが大きく、頻繁にPHPのバージョンアップを迫られる
- DBを使っているので「縛られ感」が強く、他のプラットフォームに移行するのが大変
- 色々なプラグインを使っていくと、WordPress環境が汚染されていく感がある

というのが課題でした。

これらの課題をうまく解決できそうなのが、いわゆる**静的サイトジェネレータ(Static Site Generator; SSG)**で、Gatsby、Jekyll、Hugoなどが有名どころです。どれを使っても良かったのですが、これらをデプロイするプラットフォームを考えた場合、VercelやNetlifyよりもGitHub Pagesがサービスとしての永続性がありそうだったので、GitHub Pagesのデフォルトの静的サイトジェネレータであるJekyllを選択することにしました（Jekyllが自分が慣れているRubyで書かれているのもポイントでした）。

## コンテンツ移行作業

WordPressからJekyllへの移行は色々な人が既に試みていて、ツールもいくつか用意されています。まずは、こちらの[記事](https://dev.to/rupeshtiwari/importing-wordpress-or-blogger-blogs-to-jekyll-blog-mpg)を参考にしながらWordPressからJekyllへの変換を試みました。これで無事にWordPressのコンテンツ（記事やイメージ）をダウンロードすることができます。ただ、この記事では「ブログ記事はMarkdownとしてダウンロードされる」と書いてあるものの、実際にはHTMLのままダウンロードされてきてしまいます。HTMLのままでもJekyllは問題なく処理できるので大きな問題ではないのですが、せっかくならブログ記事は全てMarkdownに統一したいところです。

結局この方法ではMarkdownにすることは出来なさそうだったので、こちらは諦めて別の[記事](https://taroyabuki.github.io/2018/08/18/switching-to-jekyll-from-wordpress/)にある方法を試してみました。こちらの方法では本文は問題なくMarkdownになってダウンロードできるのですが、この方法では画像などはダウンロードしてくれません。仕方がないので、イメージファイルはWordPressサーバの `/wp-content/uploads/` から直接scpで取得しました（私は自前でWordPressを立てていたのでこれができましたが、WordPressのホスティングサービスを使っている場合はこの方法は使えないので、最初に紹介したツールでダウンロードすると良いでしょう）。

ここまでできれば概ね必要なコンテンツはローカルに揃っていますので、あとは手作業でいくつかやっていきます。一番面倒なのはイメージのリンクの書き換えです。先の方法でダウンロードされたMarkdownファイルは、元のWordPress上のイメージへのリンクが埋め込まれたままになっていますので、これをローカルのファイルを参照するように書き換える必要があります。この作業はVisual Studio Codeの正規表現置換機能を駆使して行いました。複数ファイルに跨った置換が一括でできるのでとっても便利です（これが無かったら心が折れてたと思います）。

ここまでできれば、Jekyllを使ってサイト生成をしてある程度確認をすることができます。

## Jekyllのテーマ選定

Jekyllにはたくさんのテーマが用意されています。例えば、

- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [https://jamstackthemes.dev/ssg/jekyll/](https://jamstackthemes.dev/ssg/jekyll/)

などにライブラリがありますのでこの中から好きなものを選んで使えば良いです。沢山ありすぎて選ぶのが大変ですが、私は今回以下のような基準でテーマを選定しました。

- Blogに適している
- デザインがシンプル（ケバくない）
- 広告フリー
- トップページに記事がリストされ、ページネーションされる
- 記事のトップ（Eye Catch）画像がサムネールとして表示される
- OGPに対応している

これの条件を満たしていそうなものとして目に止まったのが[Menca](https://jekyllthemes.io/theme/menca-blog-jekyll-theme)というテーマです。

Jekyllには無料のテーマが沢山ありますが、このテーマは有料です。でもこれが一番私の要求に近いものだったので、今回はこれを購入することにしました。十分に値段の価値はあったと思います。

このテーマで概ね私の要求は満たせているのですが、一つだけ不満な点がありました。Eye Catch画像のない記事の表示があまり見栄えが良く無かったのです。そこで、Eye Catch画像の設定がない記事でデフォルトで表示されるイメージを指定できるように、[こちら](https://github.com/mshindo/mshindo.github.io/commit/28af0763e79bb124acc1e377941f9267b984dc3a)のような簡単な変更を加えました。

そして `_config.yml` で

`default_image: "/images/lukas-blazek-GnvurwJsKaY-unsplash.jpg"`

のように指定するようにしています。

あとは以下のような微調整をしました。

- 不要なFront Matterの削除
- categoryをtagにマップ
- YouTube動画を埋め込んでいた部分は手動で対応

また、Markdownではセンタリングができないので、センタリングが必要な部分はこの[記事](https://choose0or7.github.io/posts/ja/center-text-and-image-in-markdown)の方法でセンタリングをしています。

現時点ではまだ幾つか対応できていない点があります。

- TwitterのTweetの埋め込み
- TableのMarkdown化
- Search機能が日本語では動かない
- SEO施策
- ~~リンクの表示が目立たない~~(2023/02/20解決)

でも、これはらとりあえず急ぎではないので、追々対応していけたらと思っています。

## まとめ

今回は、仮住まいへの引っ越しを契機として、長年の課題だった個人ブログで使っていたWordPressからの脱却を図り、Jekyll (GitHub Pages) に移行をしました。移行には結構手間がかかりましたが、晴れてWordPressの呪縛から逃れた喜びと、`git push` するだけでブログの更新ができるエクスペリエンスは思いのほか快適で、十分に労力に見合うものだったな、と思っています。

Photo by [Robinson Greig](https://unsplash.com/@robinson?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/photos/HrnAxAUwle8?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
