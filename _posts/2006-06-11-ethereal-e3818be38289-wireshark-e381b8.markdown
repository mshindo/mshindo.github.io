---
date: 2006-06-11 07:50:27+00:00
layout: post
title: Ethereal から Wireshark へ
categories:
- コンピュータとインターネット
language:
- 日本語
---

少々混乱しましたが、


<blockquote>Ethereal のオリジナル作者である Gerald Comb 氏が、勤めていた NSI Inc. を離れ、CACE Technologies に移ったが、"Etehreal" という登録商標は NSI が保持しており、Gerald が持って出るわけにはいかなかったため、やむなく Wireshark という名前に変えて、今後はそちらで開発を続けていく。</blockquote>


ということのようです。既に Web Site や SVN Repository, Wiki, Bugzilla, などの infrastructure は整っています。

少々問題だったのは、このような事が core member 内で水面下行われて、突然アナウンスが出されたことですね。なので、core 以外の人はみんな慌ててしまった、と。

まあ、パブリックにやってたら決まることも決まらなくなる状況が予想されるので、非常に現実的なやり方だったのだとは思います。

ここ数週間 svn repository がアップデートされなかったので（単にシステムの都合で一時的にアップデートされていない、とアナウンスされていた）、変だなー、と思っていたのですが、こういうことだったのですね。

というわけで、私が先日書いたコードも Wireshark のほうにマージされることになると思います。
