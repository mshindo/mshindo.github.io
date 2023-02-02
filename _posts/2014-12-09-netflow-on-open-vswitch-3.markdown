---
date: 2014-12-09 23:49:01+00:00
layout: post
title: NetFlow on Open vSwitch
categories:
- コンピュータとインターネット
- 仕事
language:
- 日本語
post_translations:
- pll_54864e1186265
---

Open vSwitch (OVS)はかなり以前（2009年頃から）NetFlowをサポートしています。OVSでNetFlowをenableするには以下のようなコマンドを使います:

    
    # ovs-vsctl -- set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67\"


また、NetFlowをdisableにするには次のようにすれば良いです。

    
    # ovs-vsctl -- clear Bridge br0 netflow


NetFlowには幾つかのバージョンがあります。その中でV5とV9が最もよく使われいるバージョンだと思います。OVSがサポートしているのはNetFlow V5のみです。NetFlow V9は現時点ではサポートしていません（OVSはNetFlow V9の直接の後継にあたるIPFIXをすでにサポートしているので、OVSがNetFlow V9をサポートするようになることはまずないと思われます）。

以下に、NetFlow V5パケットのヘッダーとフローレコード（1 NetFlowパケットに最大30個まで入ります）を示します。

[![NetFlow V5 header format](http://blog.shin.do/wp-content/uploads/2014/12/NetFlow-V5-Header.svg)](http://blog.shin.do/wp-content/uploads/2014/12/NetFlow-V5-Header.svg) NetFlow V5 header format

[![NetFlow V5 flow record format](http://blog.shin.do/wp-content/uploads/2014/12/NetFlow-V5-Flow-Record.svg)](http://blog.shin.do/wp-content/uploads/2014/12/NetFlow-V5-Flow-Record.svg) NetFlow V5 flow record format

NetFlow V5はIPv6のフローレコードを扱うことができません。IPv6トラフィックをモニターしたい場合は、sFlowかIPFIXを使ってください。

一般的なルータ/スイッチでのNetFlow実装と比較して、OVSのNetFlowの実装にはいくつかユニークな点がありますので、それらについて以下で述べます。

多くのNetFlow対応ルータ/スイッチではいわゆる「サンプリング」をサポートしてます。サンプリングとは、全パケットを処理するのではなく、一部だけを処理対象とするものです（サンプリング手法はいくつかありますが、このBlogの趣旨からはずれるのでここでは詳しくは述べません）。一方、OVSのNetFlowはサンプリングを行いません。OVSでサンプリングを使ったフロー処理をしたい場合はsFlowかIPFIXを使う必要があります。

OVSのNetFlowがサンプリングをしない、という点とも若干関連しますが、NetFlowフローレコード中にある「バイト数(dOctets)」と「パケット数(dPkts)」が32bitのフィールドなので、巨大なフロー（elephantフローなどと呼ばれることもあります）があると、これらのフィールドは比較的容易に溢れてしまいます。OVS内部では64bit値でバイト数、パケット数を数えていますので、これらの数が32bitでは収まらないフローがあった場合には、1つのフローの情報を複数のフローレコードに分けてエクスポートするようになっており、できるかぎり正確な値をコレクターに伝えるように努力をしています。

典型的なルータ/スイッチのNetFlow設定では、グローバルな設定（例えばエクスポート先の指定、など）に加え、インターフェース毎にNetFlow処理のenable/disalbe設定がある場合が多いと思います。一方、OVSではインターフェース毎の設定ではなく、ブリッジ毎にNetFlowの設定を行う形になっています。

多くのルータ/スイッチベースのNeflowエクスポータは、NetFlowパケットのソースIPアドレスを明示的に設定することができます（この際loopbackアドレスを使うのが一般的でしょう）。一方、OVSにはこのような設定はありません。OVSのNetFlowパケットのソースIPアドレスはホストOSのIPスタックによって決定さます。通常、このアドレスはパケットが送出されるインターフェースに紐付いているIPアドレスになります。NetFlow V5にはsFlowの「agent address」のような概念がないため、コレクターはNetFlowパケットのソースIPアドレスでエクスポータを区別するのが一般的です。OVSではNetFlowパケットのソースIPアドレスを明示的に設定することができませんので、なにかしらの理由でNetFlowパケットを送出するインターフェースが変わった場合は、NetFlowパケットのソースIPアドレスも変わる可能性があることに留意をしておきましょう。

ドキュメントにははっきりとは書かれていませんが、OVSはNetFlowエクスポータを複数指定するすることができるようになっています。これによりコレクタの冗長構成を実現することができます。設定は以下のとおりです:

    
    # ovs-vsctl -- set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\[\"10.127.1.67:2055\",\"10.127.1.68:2055\"\]


通常フローベースのネットワーク管理を行う場合、フローレコードに含まれるインターフェースのIn/Outの番号がとても重要な意味を持ちます。なぜなら、このインターフェース番号の情報を使って、興味関心のあるトラフィックのフィルタをすることが多いからです。商用コレクターの多くは洗練されたフィルタ機能を持っていることが多いです。ルータ/スイッチのNetFlowエクスポータの場合は、このインターフェース番号にはSNMPのIfIndexが使われます。一方、OVSのNetFlowではOpenFlowのポート番号が使われます。この番号はovs-ofctlコマンドで確認することができます。

    
    # ovs-ofctl show br0
    OFPT_FEATURES_REPLY (xid=0x2): dpid:0000000c29eed295
    n_tables:254, n_buffers:256
    capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
    actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
    1(eth1): addr:00:0c:29:ee:d2:95
    config: 0
    state: 0
    current: 1GB-FD COPPER AUTO_NEG
    advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
    supported: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
    speed: 1000 Mbps now, 1000 Mbps max
    LOCAL(br0): addr:00:0c:29:ee:d2:95
    config: 0
    state: 0
    speed: 0 Mbps now, 0 Mbps max
    OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0


この例ではeth1のOpenFlowポート番号が1であることを示しています。幾つかのインターフェース番号は特定の意味を持っています。ホスト自身が持つインターフェース（上の例で"LOCAL"と示されているインターフェース）のインターフェース番号は65534になります。また、パケットがブロードキャスト/マルチキャストの場合にはOutのインターフェース番号は65535になります。一般的なルータ/スイッチではこれはどちらの場合もインターフェース番号は0になることが多いと思います。

あれ、最近OVSにはIfIndexが追加されたんじゃ、と思われる方もおられるかもしれません。ご指摘の通りで、最近OVSの"Interface"テーブルにIfIndexが追加されています。ではNetFlowでもこのIfIndexを使うべきではと考えるのはごもっともですが、実はそんなに簡単な話ではありません。例えば、OVSによって作られるトンネルインターフェース（GRE, VXLAN, STTなど）はIfIndexを持っていませんので、単純にNetFlowのフローレコードでIfIndexを使おうとすると、これらのインターフェースに流れるトラフィックの情報を伝えられなくなってしまいます。

NetFlow V5のヘッダには「Engine ID」と「Engine Type」というフィールドがあります。デフォルトでこれらのフィールドがどのような値になるかはデータパスの種類によって異なります。OVSがnetdevを使ったユーザスペースで動いている場合、Engine ID、Engine Typeはおのおのデータパス名のハッシュから得られた値の上位８bit、下位8bitになります。一方、Linux Netlinkを使ったkernelデータパスの場合は、Engine ID、Engine TypeともにデータパスのIfIndexが使われます。OVSのデータパスのIfIndexは以下のコマンドで確認することができます:

    
    # cat /proc/sys/net/ipv4/conf/ovs-system


もちろんこららのデフォルト値は明示的に設定することも可能です。例えば以下のお通り:

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\”10.127.1.67:2055\” engine_id=10 engine_type=20


上のコマンドは最初にNetFlowの設定を行うのと同時にEngine TypeとEngine IDを設定をする場合の例ですが、NetFlowの設定をしたあとに、関連パラメータを変更をすることも可能です。

    
    # ovs-vsctl clear NetFlow br0 engine_id=10 engine_type=20


典型的なEngine TypeとEngine IDのユースケースは、物理的には一つだが論理的には複数あるエクスポータを区別するというものだと思います。Cisco 6500のケースなどがいい例です。Cisco 6500はMSFCとPFCがそれぞれ独立したNetFlowエンジンを持っているので、これらを区別したい場合にEngine Type、Engine IDを使う場合があります。OVSの場合はには、複数のブリッジから出力されるNetFlowフロレコードを区別したいケースなどで使用することができます。先に述べた通り、OVSでは、NetFlowパケットのソースIPアドレスはホストの標準的なIPスタックにによって決められます（そしてこれは、必ずしもNetFlowがenableにされたブリッジインターフェスのIPアドレスと同じものにはなりません）。したがって、NetFlowパケットのソースIPアドレスを使ってどのブリッジからエクスポートされたフローレコードなのかを特定することができません。しかし、Engine TypeやEngine IDにそれぞれ個別な値を設定することによって、どのブリッジからのフローレコードなのかを識別することができるようになります。とはいうものの、Engine Type/Engine IDをみて論理的なエクスポータを識別することができるコレクターは私の知る限りそう多くはないと思います。

Engine IDに関してはもう一つの使い道があります。既に述べたように、OVSはNetFlowフローレコードのIn/Outのインターフェース番号としてOpenFlowのポート番号を使用します。OpenFlowのポート番号はブリッジごとにユニークな値です。したがって、複数のブリッジでは同じOpenFlowポート番号が使われる可能性があります。このままではコレクターはそれらのインターフェースを区別することができません。このような場合にはadd_to_interfaceという設定をtrueにしてやることで解決することができます。

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67:2055\" add_to_interface=true


このパラメータがtrueの場合、In/Outのインターフェース番号の上位7bitがEngine IDの下位7bitで置き換えられるようになります。ブリッジごとに異なるEngine IDを振るようにすれば、ブリッジ間のインターフェース番号の衝突を避けることができるようになります。

一般的なルータ/スイッチベースのNetFlowエクスポータと同様、OVSのNetFlowにもactiveおよびinactiveなタイムアウトの概念があります。activeタイムアウトは以下のようなコマンドで設定できます（秒で指定）。

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67:2055\" active_timeout=60


明示的に設定されたない場合はactiveタイムアウトは600秒になります。また、activeタイムアウトに-1が指定された場合はactiveタイムアウトはしないようになります。

OVSのNetFlowにもinactiveタイムアウト機構が備わっていますが、明示的にこれを設定することはできません。OVSが管理しているフローの情報がinactiveタイムアウトでdatapathから削除された際にNetFlowのフローレコードもエクスポートされるようになっています。このinactiveタイムアウトは動的であり、さまざまな要素（OVSのバージョン、CPUやメモリの使用状況、など）によって決まります。このタイムアウト時間は最近のOVSでは通常1〜2秒と短めになっています。これは一般的なルータ/スイッチベースのNetFlowエクスポータのデフォルトのinactiveタイムアウト時間（通常15秒）と比べるとかなり短い時間となっています。

ICMPのフローがエクスポートされる際の扱いですが、OVSは一般的なルータ/スイッチと同様な動きをします。すなわち、フローレコードのソースポート番号に "ICMP Type * 256 + Code" の値を入れ、デスティネーションポート番号には0を入れる、というものです。

フローレコード中のNextHop、ソース/デスティネーションAS番号およびnetmaskは0になります。これはOVSが「スイッチ」であることを考えると妥当な動きと言えます。

ここまでで述べてきたように、OVSでNetFlowを使うにあたってはいくつかのポイントについて留意をしておく必要があります。sFlowやIPFIXと比べた時のNetFlowの強みの一つは、NetFlowをサポートしたオープンソースや商用のコレクタが豊富にあることだと思います。どのようなコレクタを選択してもまず間違いなくNetFlow V5はサポートされているはずです。ぜひ、OVSのNetFlow機能を試してみてください。きっとあなたのネットワークにいままでにない可視性を提供してくれることと思います。
