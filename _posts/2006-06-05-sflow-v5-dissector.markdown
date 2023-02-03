---
date: 2006-06-05 13:17:02+00:00
layout: post
title: sFlow V5 dissector
tags:
- 趣味
language:
- 日本語
---

INTEROP の NOC で GenieATM 6000 を使った NetFlow / sFlow の監視をする傍ら、Ethereal の sFlow V5 の dissector を書いていました。ほぼ完了！

Force10 Networks の機器の吐き出す Expanded Flow Sample は正しく dissect できるようになりました。あとは、Extended でない Flow Sample について少々手直しをするだけ。

sFlow の drafot / RFC と実際のデータグラム見比べていて、最初はどうもよくわからなかったのですが、XDR で encode しているんですね。それが分かってからはすっきり！

いままで、Ethereal で sFlow V5 が dissect できないばっかりに、InMon の sflowtool に頼らなければいけなくて不便だったのですが、これで解消します！ パチパチ。
