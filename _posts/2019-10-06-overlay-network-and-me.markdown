---
date: 2019-10-06 10:46:01+00:00
layout: post
title: オーバーレイネットワークと私
description: なぜ今まで私がさまざまなオーバーレイ技術に関わってきたのか
image: '/images/robert-anasch-WbNhARVd3tg-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
- 仕事
language:
- 日本語
keywords:
- Ascend
- cosine
- ipsec
- l2tp
- nicira
- NSX
- overlay
- sd-wan
- vmware
---

日頃から仕事でお世話になっている方からこんな[記事](https://pc.watch.impress.co.jp/docs/2004/0917/config018.htm)を教えてもらった。2004年の記事だが、当時IntelのCTOだったPat Gelsinger（現VMwareのCEO）がオーバーレイネットワークの構想を持っていたのには驚かされる。Patが思い描いていた世界は（当時彼がIntelにいたということを考えると）全てのデスクトップやノートブックPCにまでオーバーレイネットワークが伸びていき、そこで色々なネットワークサービス面に接続して使う、というような姿だったのではないかと想像する。今日のオーバーレイネットワークは我々の手元のPCまで届いているとは言い難い状況だが、クラウドやデータセンターにおいてはそれに近い姿が実現されているので、2004年にこのような世界観を持っていたPatの先見性には驚かされるばかりである。

Patのような先見の明を持ち合わせていたわけではないが、なぜか私もこれまでオーバーレイネットワークに携わることがとても多かった。今回はその辺を少し振り返ってみようと思う。

私は1997年にAscend Communicationsに入社したのだが、当時Ascendはアクセスサーバーとして圧倒的なシェアを持っていて、どのISPもAscendを使っていた時代だった。そんな中、ネットワーク業界の経験がない私がAscendのSEとして飛び込んでいったわけで、最初の1年は「自分は本当に役に立っているのだろうか？」と自問自答する日々だった。PPPの16進ダンプを見るだけですらすらデバッグができる人たちがゴロゴロしている中で、自分は何をしたらいいのかと悩んでいたのである。であれば、他の人とは少し違ったことをやろう、と思い、当時Ascend MAXが持っていたAscend Tunnel Management Protocol (ATMP) を触り始めたのが、私にとってオーバーレイネットワークとの最初の出会いであった。当時はダイアルアップ系のトンネリングプロトコルとしては、ATMP、PPTP、L2Fなどベンダー色が強いものが乱立していたため、IETFとして標準的なトンネリングプロトコルを作ろうということでL2TP (Layter 2 Tunneling Protocol) が生まれた。AscendがL2TPの仕様策定にも関わっていたこともあり、私も比較的早い段階でL2TPの実装に触ることができた。ちょうどその頃、VPN周りの検証や運用に関する知見などの共有を目的とするグループvpnops（発起人は当時IRIにおられた松本直人さん）でL2TPの相互接続性の試験をしようということになり、幸い私にもお声がけを頂き検証に参加することとなった。この相互運用試験時のデバッグのためにtcpdump用のL2TP dissectorを書いて持って行ったのだが、このコードをのちにitojunさんがtcpdumpのtrunkにマージをしてくださったのは良い思い出である。

こんなことをしているうちにそこそこL2TPに関する知見が溜まっていったわけだが、丁度そんなタイミングでNTTが初のインターネット定額サービス（フレッツISDN）を始めるにあたりL2TPを使うかもしれないということで、Ascendで一番L2TPと戯れていたと思われる私がこのプロジェクトに引き込まれることになった。このプロジェクトは私にとって人生の転機となるプロジェクトであり、その際にお世話になった方々には本当に感謝しても仕切れない。そんなこんなでL2TPどっぷりな日々を送ることになり、気がつけば私もL2TPプロコルのコントロールメッセージを全てそらで言えるようになっていた（笑）

Ascendでの心残りはIPsecができなかったことだ。プロトタイプの製品はあったものの、残念ながら市場に出る製品にはならなかった。個人的にはIPsecにとても興味があったので、2000年にAscendからCoSine Communicationsという会社に移った（IPsecだけが理由ではないが、IPsecは大きな動機の一つであった）。CoSineがやっていたのは、完全な仮想ルータをIPsecでトンネルすることでオーバーレイ型のVPNを実現する、というものであった。当時は仮想ルータという概念はまだ一般的ではなく、VRFが出てきたばかりという時代だったので、CoSineがやろうとしていたことは当時としてはかなり先進的であったと思う（一方、ソフトウェアの品質はお世辞にも高くはなかったので、色々ご迷惑をおかけしました。すみません）。

当時はVPNに関しては2大派閥があった。Peerモデル派とオーバーレイモデル派である。Peerモデル派の代表はBGP/MPLS VPN（いわゆるRFC2547/RFC4364）。それに対してCoSineがやっていたのはオーバーレイモデルなVPNである。注意したのいのは、ここでいう「Peer vs オーバーレイ」というのはデータプレーンでのオーバーレイ（トンネル/カプセル化）をするかどうかの話ではなく、サービスプロバイダが顧客の経路情報に関与するかどうか、という話である。Peerモデルは顧客の経路情報とサービスプロバイダの経路情報を同じように扱うモデルである。顧客経路とサービスプロバイダのバックボーンの経路を対等（peer）に扱うのでこのように呼ばれている。一方、オーバーレイモデルではサービスプロバイダは顧客の経路情報には関与せず、顧客側の経路制御は顧客側で行う。典型的には、トンネル上でルーティングプロトコルを動かして顧客経路を交換するモデルだ。

Peerモデル vs オーバーレイモデルに関してはIETFのMailing Listでもしばしば宗教論争が起こった。当時はPeerモデル派が優勢で、Peerモデル派からは「オーバーレイモデルなどスケールするはずがない」など、こてんぱんに言われることも多く、悔しい思いをしていたのを良く憶えている。真っ向から対立していた両陣営の主張の正当性はさておき、商業的に成功したのは圧倒的にBGP/MPLS VPNであったのはみなさんよくご存知の通りである。

ひょんなことから2011年にNiciraと出会い、Niciaに行くことになった経緯については[こちら](https://blog.shin.do/2014/03/%e7%a7%81%e3%81%8cnicira%e3%81%ab%e5%85%a5%e3%81%a3%e3%81%9f%e3%82%8f%e3%81%91/)をご参照いただきたいが、はからずもNiciraもまたオーバーレイ技術をベースとする会社であった。Niciraが開発していた製品「NVP」は、ネットワーク機能をハードウェアから切り離しネットワークの抽象化を行う、というアーキテクチャとなっており、STT（Stateless Transport Tunneling)を使って仮想スイッチ間をトンネルして仮想ネットワークを作り出す、というものであった。Niciraは2012年にVMwareに買収され、NVPはNSXとして製品に取り込まれ、年間1000億円の市場に成長した。

Nicira/VMwareのあと、2016年にViptelaというSD-WANのスタートアップに行ったが、SD-WANもやはりオーバーレイ技術をベースにしたものである。多くのSD-WAN製品はIPsecトンネルで作られるオーバーレイネットワークだ。SD-WANの柔軟性と普遍性はコントローラの存在だけでなく、オーバーレイアーキテクチャによるところも大きい。先の「Peerモデル vs オーバーレイモデル」的な観点で見ると、CPE同士で経路を交換するSD-WANはオーバーレイモデル的なVPNということになる。2000年の頃には「スケールしない」などと散々非難をされたオーバーレイモデルのVPNが、CPEの性能向上やクラウド上のコントロールプレーンの利用により、15年の時を経て実際に大規模環境で動いているのを見るとなんだか嬉しい気分になる。

オーバーレイにはオーバーヘッドは付き物である。この性能的なオーバーヘッドを理由にオーバーレイ技術が批判されるケースをしばしば目にするが、これは非常に近視的なものの見方ではないかと思う。新しいオーバーレイネットワーク技術が生まれると、それは通常ハードウェアが想定していないカプセル化であるため、性能が劣化する。しかし、そのような問題はハードウェアの進化とともに解決されていくことが多い。特にカプセル化などはハードウェアで扱いやすい問題に属すると思うので、多くの場合時とともに性能的な問題は解決されていく。長期的にはオーバーレイによる「抽象化」がもたらす技術の進化の方がはるかに大きな意味を持つと思う。

ネットワークの話ではないが、いくつか例をあげてみたい。近代的なコンピュータとオペレーティングシステムは仮想メモリシステムを使っている。仮想アドレス空間を物理アドレス空間にマップしてメモリを使っているわけだが、これはCPUが持つMMUやTLBなどの仕組みによって大きな性能劣化なく仮想メモリシステムが使えているわけである。今日、性能的なオーバーヘッドを理由に仮想メモリシステムを否定する人はいないだろう。それよりもアドレス空間の仮想化によって得られるメリットのが方が遥かに大きいのは明らかだからだ。もう一つ別の例を挙げよう。x86 CPUの仮想化も当初は性能的なオーバーヘッドが大きく実現困難と思われていたが（それを現実的な速度でやって見せて世の中をびっくりさせたのがVMwareだったわけだが）、その後Intel VTやAMD-VなどのCPUによる仮想化支援機能によって、大きな性能の劣化なくx86の仮想化ができるようになった。今日、CPUの仮想化のメリットはおそらく誰も否定しないだろうし、性能的オーバーヘッドは相対的に無視できるほど小さくなっている。オーバーレイネットワークによるネットワーク抽象化もこれらと同じ話であると思う。イノベーションは抽象化から生まれ、抽象化する事で発生するオーバーヘッドは、多くの場合ハードウェアがのちに解決してくれるのである。

私自身、初めからこのようなことを考えて約20年間一貫してオーバーレイ・ネットワークに携わってきたわけではない。「たまたま」といえばそうなのだが、きっとオーバーレイによる抽象化とそこから生まれるワクワク感がきっと本質的に好きなのだろうと思う。

今までネットワーク機器は自分たちの手の届くところにあったので、ネットワーク機器を直接「触る」ことでネットワークを構築、管理をしてきた。しかし、これからはクラウドの時代である。クラウドにも当然物理ルーターや物理スイッチはあるが、通常我々はそれらを直接触ることはできない。物理ネットワーク機器に直接触れない環境でどのようにネットワークをエンド to エンドで構築・管理していけば良いのだろうか？　クラウド時代においては、オーバーレイでネットワークを作りエンド to エンドで管理していくのは必然の事のように思える。そんなわけで、これからも大いにオーバーレイネットワークに絡んでいきたいと思う次第である。

Photo by <a href="https://unsplash.com/@diesektion?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Robert Anasch</a> on <a href="https://unsplash.com/photos/pile-of-gray-and-brown-stones-WbNhARVd3tg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  
