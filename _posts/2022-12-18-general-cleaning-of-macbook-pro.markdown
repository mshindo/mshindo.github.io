---
date: 2022-12-18 23:40:57+00:00
layout: post
title: 塵も積もれば
image: '/images/giorgio-trovato-v_bri4iVuiM-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
- 仕事
language:
- 日本語
keywords:
- kernel_task
- MacBook Pro
- 分解
- 清掃
- 高負荷
---

いつ頃からか定かではないのですが、私が普段使っているMacBook Proがやたら重たくなり、作業がままならなくなることがしばしばありました。このような状態になると文字入力がほぼできなくなり、アクティブなウィンドウの切り替えなどに数秒かかるようになります。比較的CPU負荷が高いと考えられるZoomのバーチャル・バックグラウンド機能を使っている時によく起こります。このような状態に陥っている時にTerminalでtopコマンドを叩くと、多くの場合Load Avgが100を超えており、ひどい時は下のように240とか見たことないような値になります。%CPUで上位を占めるプロセスはWindowServerプロセスやOneDriveプロセスであることが多いのですが、これらのプロセスがそんなにCPUを食い潰すような使い方をしているわけではありません。

![]({{site.baseurl}}/images/Top-on-Terminal-1024x711.png)

このkernel_taskが何をやっているのかを調べてみたら、以下のようなAppleのKBを見つけました。

![]({{site.baseurl}}/images/kernel_task-1024x428.png)
*https://support.apple.com/ja-jp/HT207359 から引用*

どうやら温度に関係しているようです。それでは、ということで、このような操作不能状態に陥ったらMacBook Proを氷枕で冷やしてみることにしました。そうすると、確かに効果は的面、冷えてくるとkernel_taskの負荷は下がり、MacBook Proの操作が正常にできる状態に戻ることが分かりました。

これでほぼ熱が原因ということが分かりました。でも、なんでそんなに熱を持つのか？　一番怪しそうなのは内臓のファンです。そこで、MacBook Proの筐体を開いて調べてみることにしました。MacBook Proに限らず、Appleの製品の多くは通常のネジが使われておらず、特殊な星型ネジが使われています。そこでAmazonで以下の製品を購入しました。

![]({{site.baseurl}}/images/71og4A6SsXL._AC_SX679_-268x300.jpg)
*https://www.amazon.co.jp/dp/B09TP9MQ75/ より引用*

これで道具は揃いました。ただ、MacBook Proの蓋を開けるは若干コツが要ります。以下のサイトを参照しながら慎重に作業を進めました。



無事にMacBook Proをご開帳をしたら、こんな塩梅でした！

![]({{site.baseurl}}/images/IMG_8530-1024x768.jpeg)

買ってから約2年ですが、いやー、よくぞここまで埃が溜まったものです。溜まった埃をエアダスターで吹き飛ばし、蘇ったMacBook Proが以下です。

![]({{site.baseurl}}/images/IMG_8535-1024x768.jpeg)

あとは慎重に蓋を元に戻し、ネジを閉めれば完了。ネジを閉めるときの注意点としては、ネジを垂直方向に入れようとするとうまく入らない事です。ネジの取り付け面に若干の角度がついていますので、それを意識してネジを閉めるようにしましょう。

清掃後しばらく使ってみていますが、至って快調です！　今まで頻繁にしていたファンの音もほとんどすることがなくなり、Zoomのバーチャル背景などを使っても、全く問題なくなりました。

この2年間はほぼ在宅だったので、ほとんど家の埃を吸い込んだものだと思われます。どんだけ汚い部屋にいるんだ、自分（怖）！

それにしても、もっと早くに気づいて対処をしておくべきでした。これからは半年に一度くらいは掃除をしようと思います。

Photo by [Giorgio Trovato](https://unsplash.com/@giorgiotrovato?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/macbook-pro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)








