---
date: 2021-12-08 23:48:15+00:00
layout: post
title: おうち Lab と GitDNSOps
categories:
- 仕事
- 日常
- 趣味
language:
- 日本語
tags:
- coredns
- dns
- git
- gitops
- homelab
- Kubernetes
- metallb
- microk8s
- tanzu
---

本記事は、[VMware DevOps Meetup #10](https://vmware.connpass.com/event/223160/) でお話した内容に修正加筆をしたものです。

我が家の「おうち Lab」は、他の方の「逸般の誤家庭」に比べるとかなりおとなしめです。

![](https://blog.shin.do/wp-content/uploads/2021/12/HomeLab.jpg)] 

ちょっと変わったところというと、インターネット回線が2本引いてあることくらいでしょうか。なぜ2本引いているのかは、我が家のネットワークの歴史的背景によるものなので、まずちょっとだけその話を。

私が住んでいるのは築20年ほど経ったマンションです。マンション購入時に光回線を引くことを条件にしたのですが、当時はまだ光によるインターネットサービスは一般的ではなかったので、PRI 回線を引き込んでほぼ休止扱いにしてありました。その後、マンションにインターネットサービスが2つ提供されることになり、一つはケーブル TV 系の VDSL サービスで、もう一つは UCOM（現アルテリア・ネットワークス）によるもので、宅内までイーサを引き込むものでした。当然、私は後者を選択。帯域的なメリットもありましたが、それよりも当時 UCOM のインターネットサービスは /29 のグローバルアドレスをくれるという太っ腹ネットワークだった、いうのが UCOM を選択した大きな理由でした。この UCOM のネットワークは非常に品質の高いネットワークサービスを提供してくれていてとても満足しています（詳しくは[「自宅のインターネット回線品質をVeloCloudで測ってみた」](https://blog.shin.do/2020/03/internet-links-measured-by-velocloud/)を参照）

ただ、UCOM のネットワーク帯域は 100Mbps で今となっては少々寂しいですし、IPv6 のサービスも提供されていません。また、いっとき私は SD-WAN のスタートアップ (Viptela) に勤めていたので、SD-WAN のテストをするために複数回線が欲しかった、ということもあり、複数ネットワークを引き込むことにしました。前に引いてあった光回線を活用して FLETS 光を利用し、更に NURO を加え3回線にした後、現在は FLETS を解約し、UCOM と NURO の2本構成に縮退しています。

![](https://blog.shin.do/wp-content/uploads/2021/12/Lab-Network-1024x752.png)] 

その他は至って普通の Lab 環境です。Intel NUC や自作サーバで vSphere を利用して仮想環境を用意していろいろなものを検証しています。外向きに出しているサービスは個人用 blog サーバと、DNS くらいです。自分が保有しているいくつかのドメイン用の権威 primary サーバとして coreDNS、secondary サーバとして NSD を GCP 上で動かしています。以前 secondary DNS は実家の Raspberry Pi 上で動かしていたのですが、今年実家に誰もいなくなりネットワークも廃止したため、引っ越しを迫られ、引越し先をいろいろと検討しました。もちろん自宅環境に移すこともできたのですが（幸いグローバルアドレスもありますし）、同じ failure domain に primary と secondary を置くのは避けたかったので、別の方法を模索しました。いくつかのドメインは「お名前.com」で取得したもので、お名前.com はそこで取ったドメインに関しては無料で secondary DNS のサービスをしているので、これが使えたらベストだったのですが、あいにく私がメインで使っているドメイン "shin.do" はお名前.comで取ったものではなかったので、この方法は見送りとなりました。色々検討した結果、最終的には GCP 上のフリーインスタンス (e2-micoro) を使って secondary DNS (NSD) を動かすことにしました。無料のはずなのですが、毎月110円ほど費用が発生しているのが謎ですが、今のところ深く気にしないことにしています（笑）また、Lab ネットワークのための DNS cache サーバ兼簡易権威 DNS サーバとして unbound を使っています。

前置きが長くなりましたが、今回はこの外向きの coreDNS で GitOps をしてみる、というのをやってみました。GitOps とはなんぞや、という話は [ここ](https://qiita.com/ozora/items/80c913b0790c88c24f67) や [ここ](https://codezine.jp/article/detail/14310) などを参照ください。要は Git リポジトリ内容を唯一の真 (Single Source of Truth) として扱って宣言的にオペレーションを廻してやろう、という考え方/方法、です。Git を使うので、バージョン・コントロールしやすく、アクセス・コントロールや監査が容易、というメリットなどを享受することができます。

ただ、私の管理している DNS サーバは基本「ワンオペ」なので、アクセス・コントールや監査のしやすさはあまり関係ありません。気軽にロールバックできたりするのは少しだけありがたいですが、そこまで大きな魅力でもありません。それでも DNS に GitOps を適用しようと思うのは、半分ノリと、DNS の変更を git push だけ済ませられるというちょっとした心地よさからでしょうか。。GitOps を DNS に適用するので、これを私は "GitDNSOps" と呼んでいます。はい、勝手に私がそう呼んでいるだけで、一般的に受け入れられている言葉ではないです。でも、流行ったら少し嬉しいので、GitDNSOps、GitDNSOps、GitDNSOps、と3回叫んでおくことにします。![](https://blog.shin.do/wp-content/uploads/2021/12/GitDNSOps-1024x524.png)] 今回、Kubernetes 環境は microk8s を使って構築し、LoadBalancer には MetalLB を使いました。この選択に特段深い意味はないので、どっかのタイミングで Tanzu Community Edition に置き換えてもいいかな、と思っています。また、GitOps には ArgoCD を使いましたが、flux などを使っても全然構いません。

![](https://blog.shin.do/wp-content/uploads/2021/12/ArgoCD-1024x432.png)] 

coreDNS の設定は Kubernetes の ConfigMap で食わせています。特に変わった設定はありませんが、Corefile で ”reload 10s" と指定している部分だけ説明だけ説明をしておこうと思います。この設定をすると coreDNS は指定された時間（この例だと10秒）で再度設定を読み直してくれます。この設定をしないと、git push して ConfigMap が書き換わっても、coreDNS はそれに気づいてくれないので、DNSのリソースレコードが更新されません。というわけで、若干エレガントさに欠けますが、reload 設定をして設定を読み直すようにしています。

    
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: coredns
    data:
      Corefile: |
        shin.do:5353 {
            file /etc/coredns/shin.do.zone
            reload 10s
        }
      shin.do.zone: |
        $TTL  3600
        @ IN  SOA ns1.shin.do.  root.shin.do. (
                2021081404 3600 900 3600000 3600 )  
             IN  NS  ns1.shin.do.
             IN  NS  ns2.shin.do.
             IN  MX  10 ASPMX.L.GOOGLE.COM.
        ns1  IN  A 221.245.168.210
        ns2  IN  A 35.247.124.32
        blog IN  A 221.245.168.211
    


それではみなさん、GitDNSOps で気持ち良い DNS 生活を！

（Photo by [Roman Synkevych](https://unsplash.com/@synkevych?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/git?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) ）
