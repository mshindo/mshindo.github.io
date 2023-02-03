---
date: 2007-01-13 06:03:10+00:00
layout: post
title: メール Subject の MIME encoding
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
---

メールのSubjectヘッダに日本語が含まれる場合、通常MIMEでエンコードするわけですが、日本語の場合は（おそらく）歴史的経緯からCharsetはISO-2022-JPを使う場合がほとんどです。じゃ、他のCharsetを使っていけないか、というと、必ずしもそう決まっているわけではないと思うんですね。

そこでちょっと調べてみました。私がここ一年ほどで受け取ったメールで、SubjectがMIME encodeされていたものを対象に、使われていたCharsetを調べると

    
          1 BIG5
          3 EUC-JP
         38 GB2312
       9041 ISO-2022-JP
          5 ISO-8859-1
          3 ISO-8859-15
          1 KOI8-R
          1 SHIFT-JIS
         45 SHIFT_JIS
          1 UNICODE-1-1-UTF-7
          1 US-ASCII
         16 UTF-8
         86 WINDOWS-1251
          1 WINDOWS-1252
          1 WINDOWS-1254
          1 WINDOWS-874


な感じでした（各行の先頭の数字は出現回数）。

当然ISO-2022-JPがダントツなわけですがその他のCharsetもぼちぼちあります。気になるのはにSHIFT_JISやSHIFT-JIS。これはら正しいCharsetと言えるのか、判断が難しいところです。Shift JISなCharset自体はももちろんあり得る話だとは思うのですが、メールでこのような使い方をするのがOKなのかどうか微妙・・。

どなたか、ご存知の方お教えくださいまし。
