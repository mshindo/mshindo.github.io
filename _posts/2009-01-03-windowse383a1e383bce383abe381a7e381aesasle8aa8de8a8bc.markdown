---
date: 2009-01-03 02:26:23+00:00
layout: post
title: WindowsメールでのSASL認証
categories:
- コンピュータとインターネット
language:
- 日本語
---

下の娘と二人で実家に来ています。先日、父がPCをVistaにしたので、再度メールの設定をして欲しい、というもので、再設定をしてあげることにしました。メールサーバーは私の自宅に置いてあるサーバーのIMAP/SMTPをover SSLで使うことになります。

私が自己署名したCA証明書を入れて、IMAP over SSLはすんなり動くようになりましたが、SMTP Auth over SSLがうまく動きません。SMTPしようとすると、

    
    Jan 3 10:48:23 ober postfix/smtpd[12381]: warning: xxx.xxx.xxx.xxx.st.bbexcite.jp [xxx.xxx.xxx.xxx]: SASL NTLM authentication failed: authentication failure Jan 3 10:48:23 ober postfix/smtpd[12381]: warning: SASL authentication failure: client didn't issue valid NTLM response


というようなエラーが沢山出ます。SASL側のconfigかなと思い、ちょっと調べてみたものの、NTLM認証が使えるようにするためのオプションは見つからず。

で、困った時のGoogleさんに聞いてみたところ、

[http://shinshu.fm/MHz/77.88/archives/0000181142.html](http://shinshu.fm/MHz/77.88/archives/0000181142.html)

という情報が見つかり、試してみたらバッチリ動くようになりました。本来はSASLのライブラリで対処するべきなのでしょうが、とりあえず今回はクライアント側で回避することにしました。
