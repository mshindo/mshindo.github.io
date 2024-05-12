---
date: 2024-05-11 20:40:00+09:00
layout: post
title: Supermicro E300-9D で vSAN するぞ！
description: Supermicro E300-9D で vSAN OSA する際のストレージ構成に関する試行錯誤
image: '/images/andrey-matveev-WUqkLR1fyaw-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
keywords:
- E300-9D
- vSAN
- ESXi
- NVMe
- M.2
language:
- 日本語
---

我が家の「おうちLab」のサーバーは、数台の Intel NUC と 2 台の Supermicro の E300-9D-8CN8TP（以下、単に E300 と呼ぶことにします）構成なのですが、今までは E300 をシンプルに ESXi + iSCSI NAS で使っていました。E300 は比較的パワフルな機器なので、その価値をもう少し活用すべく、今回 2-Node vSAN を組んでみることにしました。以下は、その際のちょっとした苦労話です。

2年半ほど前に購入した E300 のスペックはざっくり以下のとおりです。
- Xeon D-2146NT 8 core Processor
- 128GB ECC RDIMM
- 256GB M.2 NVMe SSD
- 2TB 2.5" SSD
- 10GbE SFP+ * 2、10GbE RJ45 * 2、1GbE RJ45 * 4、IPMI * 1

現在 vSAN には OSA (Original Storage Architecture) と ESA (Express Storage Architecture) の二つのアーキテクチャが用意されています。ESA は All NVMe で組む新しいストレージ・アーキテクチャですが、おうちのハードウェア環境で動かすのは少々荷が重いため、今回は OSA を使うことにしました。

vSAN を OSA で使うためには、キャッシュ層とキャパシティ層の二つのストレージを用意する必要があります。それに加えて、ESXi 自体を動作させるストレージも必要ですので、結局三つのストレージが必要、ということになります。私の E300 は NVMe SSD と 2.5" SSD を持っていたので、これらをそれぞれキャッシュ層、キャパシティ層に使うのが妥当です。しかし、これらを vSAN のために使ってしまうと ESXi をインストールするストレージがなくなってしまいます。というわけで、E300 に ESXi インストール用のストレージを増設しなければならなくなりました。

このマシンに ESXi 用のストレージを増設するのにはいくつかの選択肢が考えられます。

### 選択肢1: USBメディア
一番お手軽な方法は ESXi を USB メディアにインストールして、そこからブートする、という方法なのですが、ESXi 7 以降、USB メディアを ESXi のブートメディアとするのは非推奨となりました（詳しくはこちらの[記事](https://note.com/hiroaki_minami/n/nd8c6b5f2c111)などを参照してください）。所詮今回の環境は Lab 環境なので壊れたたらまた作り直せば良いので、一瞬 USB ブートも考えたのですが、やはりここはきちんとお作法に従おう（一応社員ですしねw）、ということで、別の方法を模索することにしました。

### 選択肢2: NVMe on PCI-e
今回の E300 に搭載されている NVMe SSD は、オンボードの NVMe M.2 スロットに挿さっています。幸いこのマシンのマザーボードには PCI-e 3.0 x16 のスロットがあり、それが空いていたため、M.2 <-> PCI-e の変換アダプタを使えば M.2 の NVMe を PCI-e に挿すことができます。というわけで、以下のようなものを購入してみました。Amazon で 1,000 円程度で売ってます。

![]({{site.baseurl}}/images/61z-UV80hKL._AC_SX679_.jpg)
*https://www.amazon.co.jp/gp/product/B079546NPR/ より引用*

これに M.2 NVMe メモリを装着して E300 の PCI-e スロットにに挿せば、NVMe として認識され、ここに ESXi をインストールできるはず。あれれ、でもこれ、サーバからはみ出てるぞ！！

![]({{site.baseurl}}/images/IMG_0778.jpg)
*PCI-e に挿したNVMeがちょっとシャーシからはみ出ている様子。*

![]({{site.baseurl}}/images/IMG_0779.jpg)
*横から見た図。これではシャーシの蓋が閉まらない。*

どうやら確認が甘かったようです。この変換アダプタは高さが44mmあり、E300 のような 1U サーバには入らないようです。Amazon の商品タイトルにも「２Uサーバーに設置できる」って書いてあるやん。よく読め！ > 自分w

というわけで、できる限り高さの低い変換アダプタを探すことにしました。見つけたのは以下のようなもの。こちらは高さ35mmで、これならなんとか 1U に収まりそうだし、ヒートシンクもついててなかなかいい感じ。ということで購入してみました。こちらもお値段1,000円ほど。

![]({{site.baseurl}}/images/71nRLvuwCgL._AC_SX679_.jpg)
*https://www.amazon.co.jp/gp/product/B0C89JNKKT/ より引用*

で、いざ E300 に挿してみたところ、今度はいい感じに収まっています。

![]({{site.baseurl}}/images/IMG_0782.jpg)
*これならなんとか 1U 筐体に収まる*

が、あれれ、また蓋が閉まらない。よく見ると、この E300 の蓋の裏側には 2.5" SSD がマウントされて、この部分が PCI-e の真上にあるので、2.5" SSD と PCI-e に挿した NVME がぶつかってしまい蓋が閉まらないという事態となりました。

![]({{site.baseurl}}/images/IMG_0786.jpg)
*2.5" SSD の厚さがあるので、下の PCI-e に挿した NVMe とぶつかってしまい、蓋が閉まらない*

この 2.5" SSD を他の場所に動かすためのスペースはこの筐体にはなく（CPU やメモリがキチキチで空きスペースなし）、この 2.5" SSD を活用しようとする限り、E300 の PCI-e のスロットを活用するのは無理っぽそう。

というわけで作戦変更を余儀なくされました。

### 選択肢3: もう一つの M2 スロット
今回の E300 ではマザーボード上の M Key 2280 のスロットに M2.NVMe が挿さっていました（下の写真右上の部分）。実は、この E300 にはもう一つ M.2 のスロットが用意されていて、こいつが空いていました（赤い四角で囲った部分）。E300 のドキュメントを読むと、この M.2 のフォームファクタは「B Key 3042」である、と書かれています。これに合うものが見つけられれば、これをブートデバイスに使うことができそうです。

![]({{site.baseurl}}/images/IMG_0700.jpg)
*E300の筐体。NVMe がインストールされていた M Key 2280 の M.2 に加え、もう一つ M.2 のスロットが用意されている。*

でも「B Key 3042」なんて聞いたことないぞ、と思いながら Amazon などを探し回りましたが、案の定見つけることができません。なんでこんな変態的なフォームファクタのコネクタ積んでるんだ、そもそもそんなデバイスないじゃん、と半ば諦めかけていたんですが、一晩寝たら「ひょっとして 3042 じゃなくてもいいのでは？」と思いつきました。

M.2 の仕様を表している数字は、実は幅と長さを示しています。例えば NVMe の SSD で多く使われてる 2280 は、幅22mm、長さ80mm、である事を表しています。一方、Key の方は端子の切れ込みの位置を表していて、M Key と B Key では切れ込みの位置が異なります。

M.2 のデバイスを挿す場合、長さの方はネジの位置等があるので、基本ピッタリそのサイズである必要があるのですが（例えば XX80 のスロットに XX42 の M.2 デバイスは装着できないし、その逆も然り）、幅の方は特に物理的に干渉等がなければ装着できないこともないのでは？と、思ったわけです（ただし、Type は合っている必要があります）。というわけで、Type B に対応した XX42 という M.2 デバイスを Amazon で探してみたところ、2242 サイズの SSD は結構存在しています。ESXi のブートに使うだけなので、さして大きな容量は必要ありませんから、今回は Type B 2242 60GB のもので良さそう。で、見つけたのが以下のような製品。この SSD は端子の両側に切れ込みがあるタイプで、M Key でも B Key でも対応できるハズ。お値段も2,000円ちょっとですから、ダメもとで買って試してみました。

![]({{site.baseurl}}/images/61I4dVw+HEL._AC_SY450_.jpg)
*https://www.amazon.co.jp/gp/product/B07H7BWN4G/ より引用*

これを E300 に挿して ESXi のインストールを試みたところ、バッチリ ATA SSD として認識されました！

![]({{site.baseurl}}/images/ESXi-install-on-M2-SSD.png)

これでようやく E300 で無事 M.2 NVMe をキャッシュ層、2.5" SSD をキャパシティ層にした vSAN OSA を組むことができました。めでたし、めでたし！

![]({{site.baseurl}}/images/vSAN-datastore.png)
*vSAN ストレージを vSphere Client からみたところ*

### 選択肢4: SATADOM（番外編）
今回購入には至らなかったのですが、同僚の川満さんから SATA DOM (SATA Disk-On-Motherboard) というものがある、と教えてもらいました。これは文字通り、マザーボード上の SATA コネクタに直接指すタイプの SSD モジュールです。耐久性が求められる使い方を想定しているため、多くの場合 SLC (Single Level Cell) NAND でできています。そのため高価になりがちで、32GB のものだと 3 万円以上するようです。ちょっと試してみよう、という金額ではなかったので、今回は購入には至りませんでした。今回は E300 に空きの M.2 スロットがあったので良かったのですが、そのようなスロットが無い場合の ESXi のブートメディアとしては SATADOM は結構有力な方法なのではないかと思います。

![]({{site.baseurl}}/images/31iiVqA40OL._AC_.jpg)
*https://www.amazon.co.jp/dp/B08NPMRWPB/ より引用*

Photo by <a href="https://unsplash.com/@zelebb?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Andrey Matveev</a> on <a href="https://unsplash.com/photos/a-collection-of-memory-cards-laid-out-on-a-table-WUqkLR1fyaw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  
