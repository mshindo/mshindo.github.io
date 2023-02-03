---
date: 2015-12-20 01:18:59+00:00
layout: post
title: OVN (Open Virtual Network) の紹介
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
post_translations:
- pll_567610d39f7d7
keywords:
- GENEVE
- Open vSwitch
- OVN
- STT
- VXLAN
- 仮想化
---

OVN（Open Virtual Network）は、L2、L3、ACLなどの仮想ネットワーキング機能を提供するオープンソースソフトウェアです。OVNは、仮想スイッチとして非常に広く使われているOpen vSwitch（OVS）と連携して動作します。開発もOVSと同じコミュニティで行われていおり、OVSのサブプロジェクト的な位置付けになっています。OVSと同様、ソースコードは完全にオープン、議論はパブリックなメーリングリスト及びIRCを使って進められています。VMware、RedHatなどが中心となって開発を進めていますが、誰でも開発に参加することができます。

現在OVNがターゲットにしているプラットフォームはLinux系ハイパーバイザ（KVM、Xen）やコンテナなどです。DPDKにも対応します。今のところESXiのサポートの予定はありません。なお、OVSはHyper-Vに移植されていますので、OVNのHyper-Vサポートも将来的にはあるかもしれません。

OVN自体はCMS（Cloud Management System）中立で、OpenStack、DockerやMesosなどとの連携を視野に入れています。その中でも特にOpenStackとの連携はOVNにとって大きな意味を持ち、OpenStackとのより良いインテグレーションはOVN開発の大きな動機の一つになっています。

OVNの目指すところはOVSと同様で、ハイパーバイザ1000台規模の大規模環境をサポートし、かつ、商用利用に耐える品質を持ったものを提供する、というものです。

以下に、OVNの基本アーキテクチャを以下に示します。

[![OVN Architecture]({{site.baseurl}}/images/OVN-Architecture.png) OVNアーキテクチャ

OVNには主に二つのコンポーネント（プロセス）があります。一つはovn-northd、もう一つはovn-controllerです。ovn-northdはその名前からも分かる通り、CMSに対して "northbound" （北向き）なインターフェースを提供しています。現状、ovn-northdはOVNのデプロイメントに1つだけ存在しますが、将来的にはこの部分は冗長化、スケールアウト化していく予定です。

OVNのアーキテクチャでとてもユニークなのは、CMSとのインテーグレーションポイントがデータベース（DB）である、という点です。この部分にRESTfulなAPIを期待される方も多いかと思いますが、現状ovn-northdはCMSとDBを使って連携をするようになっています。

ovn-northdは2つのデータベースを使います。一つは "Northbound DB" で、こちらは仮想ネットワークの「あるべき」状態（desired state）を保持します。具体的には論理スイッチ、論理ポート、論理ルータ、論理ルータポート、ACLなどに関する情報が保持されます。ここには一切物理的な情報は含まれません。ovn-northdから見てnorth側にいる実体（典型的にはCMS）はこのNorhbound DBにデータを書き込むことでovn-northdと連携します。ovn-northdが扱うもう一つのDBは "Southbound DB" で、こちらは物理、論理、バインディングなどのランタイム情報を保持します。このDBにはChassis、Datapath Binding、Encapsulation、Port Binding、Logical Flowといった情報が含まれます。ovn-northdの主な役割は、Northbound DBをSouthbound DBに変換し、Logical Flowを生成することです。

OVNを構成するもう一つのプロセスはovn-controllerというもので、こちらは各ホストにインストールされる分散されたローカル コントローラです。ovn-controllerはSouthbound DBから情報を読み取り、しかるべき設定を各ホスト上のOVS（ovsdb-server, ovs-vswitchd）に対して行います。また、ホストやVIFが追加された場合には、それらの情報をSouthbound DBへ書き込みます。Logical Flow情報をPhysical Flow情報に変換してOVSに設定を投入するのもovn-controllerの役目です。

ちなみに現在、OVN用のDBとしてOVSDBを使っています。本質的にはDBは何でも良いのですが、OVSにはOVSDBがすでに用意されていて、開発者側もその特性をよく理解しているので、手っ取り早くOVSDBを使っています。

OVNが提供する機能はL2、L3、ACL (Security Group)の3つです。

OVNのL2機能は論理スイッチ機能を提供します。具体的には、各ホスト間でL2 over L3なオーバーレーネットワークを自動的に作成してくれます。OVNがデフォルトで使うカプセル化技術はGeneveです。メタデータのサポート、マルチパスとの親和性、ハードウェアアクセラレーションの必要性を考慮すると、Geneveが最も妥当な選択でしょう。ただし、Geneveのハードウェアアクセラレーションに対応したNICはまださほど多くはないので、Geneveに対応していないNICで高い性能が必要な場合はSTTを使うこともできます。また、通常HW-VTEPはGeneveやSTTには対応していないので、HW-VTEPと通信する場合はVXLANを使います。

OVNのL3の機能はいわゆる分散論理ルーティング機能を提供します。OVNのL3機能はどこか特定のホストで動くわけではなく、すべてのホストが自律的にL3機能を実行します。OVNが現在サポートしているL3構成はとてもシンプルなもので、ルータの下に接続されている論理スイッチ間のルーティング及びDefault Gatewayへのルーティングに限られます。今のところ、Default Gateway以外のStatic Routeを設定することはできません。シンプルですが、OpenStack NeutronのL3基本機能を実現するのには十分です。NATは近々サポートされる予定です。

従来、OpenStack NeutronのOVS Pluginでは、Linux Bridge上のtapインターフェース（vnet）にiptablesを使ってACLをかけることによってSecurity Groupを実現していました。OVSのBridgeとLinux Bridgeが混在しているので、いささか複雑なアーキテクチャだったと言えます。

[![under-the-hood-scenario-1-ovs-compute]({{site.baseurl}}/images/under-the-hood-scenario-1-ovs-compute.png) 従来のOpenStack Neutron OVS pluginのアーキテクチャ（http://docs.ocselected.org/openstack-manuals/kilo/networking-guide/content/figures/6/a/a/common/figures/under-the-hood-scenario-1-ovs-compute.png より引用）

しかしOVS 2.4からOVSがconntrackと連携できるようになったので、iptablesに頼ることなくOVSでステートフルなACLをネイティブに実現できるようになりました。OVNはこのOVSとconntrackの連携機能を使ってACLを実現しています。

conntrack連携機能はOVSの機能なので、OVNなしでも利用はできますが、OVNからはconntrackを意識することなく、論理的なACLを設定するだけでそれがOVSのconntrack機能に自動的にコンパイルされて設定されますので大変便利です。

次回以降で、L2、L3、Security Groupの機能をもう少し深堀りしていこうと思います。
