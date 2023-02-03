---
date: 2020-04-26 22:49:23+00:00
layout: post
title: 宣言的ネットワーキングとインクリメンタル処理
image: '/images/irvan-smith-5eBW5GomfhY-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- datalog
- ddlog
- declarative
- declarative networking
- differential datalog
- Kubernetes
- NSX
- OVN
---

### はじめに


2020年1月に札幌で行われた[JANOG45](https://www.janog.gr.jp/meeting/janog45/program)で、Cisco Systemsの河野美也さんと共同で「宣言的（Declarative）ネットワーキング」というセッションをやらせていただきました。その時の資料は[こちら](https://www.slideshare.net/motonorishindo/declarative-225431574)です。残念ながら30分と時間が限られていたため、十分にお伝えすることができなかった部分もあるため、こちらに補足記事としてまとめておくことにしました。


### そもそも宣言的（Declarative）って何？


最近「宣言的」や「Declarative」という言葉をよく耳にするようになっていると思います。実は、宣言的/Declarativeという言葉はそう新しいものではなく、実はかなり前から使われている言葉です。プログラミング言語の世界では関数や述語論理に基づいたプログラミング言語などが従来型の命令的プログラミング言語との対比として「宣言的プログラミング言語」と呼ばれていました。比較的最近では、Ansibleなどの構成管理ツールやJenkinsなどのCI/CDツールにも宣言的な側面があると言われていますし、FlutterやSwiftUIなどのU/Iフレームワークも「宣言的U/I」などと呼ばれ注目されています。このように「宣言的」という言葉はさまざまな文脈でいろいろな意味で使われており、決定的な定義があるわけではありません。宣言的という言葉が持つ多面性については河野さんがこちらの[記事](https://qiita.com/mkohno/items/ac83cf461632fbf4d862)で言及してくださっていますので是非ご一読ください。

しかし、最近「宣言的」という言葉をよく聞くようになったのはなんと言ってもKubernetesの影響でしょう。Kubernetesが宣言的であると言われるのは、Kubernetesでは各種リソースの望まれる状態（Desired State）をManifestとして定義する事ができて、現在の状態（Current State）とDesired Stateの間にズレがある場合は、コントローラがそれを検知し自動的にそのズレを修復するように動くようになっているからです。「どのようにしてそのズレを解消するかは問わない。ともかくDesiredな状態にしてくれ！」と指示をするので、Kubernetesは宣言的であると考えらています。Desired Stateと現在のStateを常に比較し、差異があればそれをなくそうとする動作はReconciliation Loopと呼ばれており、各コントローラに実装されています。Kubernetesに自己修復（Self-Healing）性があると言われるのは、Kubernetesの各種コントローラにこのようなReconciliation Loopが実装されているためです。


### 宣言的（Declarative）ネットワーキング


JANOG45のセッション後に幾つかのフィードバックをもらいました。その中の一つは以下のようなものでした。

https://twitter.com/rafiror/status/1219984617513472000?s=20

非常に妥当なご意見だと思いますし、他にも同じように感じられた方も多かったようです。なぜ今日のネットワークはKubernetes的な宣言性を持ち得ていないのか、どのようにしたらそれを実現できるのか、という点に興味を抱いている人は多いんですね。これは大変興味深い話であり、私自身も大きな関心を持っています。

個人的には、今日のネットワークがKubernetes的な宣言性を持てていない理由の一つとして「Desired Stateを定義するのが難しく、Reconciliationするのも難しい」という点があるのではないかと思っています。KubernetesのDeploymentであれば、どのようなコンテナがいくつ動いているかを把握したり、アプリケーションが応答するかをlivenessProbeなどで監視して、Desired Stateとのズレがあったら基本的に再度コンテナを立ち上げ直す事でReconciliationを図ります。では、ネットワークのDesired Stateって何でしょう？　おそらく「ホストAからホストBにパケットが届く事」あるいはL4まで考えれば「ホストA上のサービスCからホストB上のサービスDへパケットが届く事」ということになるのではないかと思います。このようなDesired Stateを検出するのはなかなか困難です。コンテナは一次元的な要素なので比較的シンプルですが、ネットワークはsourceとdestinationがある二次元的な要素ですので、N^2問題に直面します。ホストがN台あったとすると、全ての到達性を確認するためにはN\*Nのprobeをしなければいけません。Network Policy（ACL）も含めてネットワークのDesired Stateを考えると複雑性はN\*Nでは済まず、サービス数Mも含めて考えてN\*M\*N\*Mでの到達性を把握する必要があります。パケットの到達性はsourceに依存するので、このようなprobeをどこからしたら良いのか、という点も悩ましいです。また、仮にこのようなProbeできてネットワークのDesired Stateと現在のStateにズレをうまく検出できたとしても、それをどのようにReconcileするかも難しいです。KubernetesのDeployment/Podの場合は、単純にコンテナを作り直す、という戦略を取っているわけですが、ネットワークの場合は、単純にネットワークを実現しているコンテナを再起動しても解決しない場合が多いと思います。ネットワークのエンドポイントに問題はなく、途中の経路に何か問題があって疎通性が阻害されている場合には、いくらエンドポイントを再起動してもおそらく問題は解決しないでしょう。

このように、現在のネットワークにKubernetes的な宣言的特性や自己修復性を持たせるのは簡単なことではないように思います（少しドメインを限定すればネットワークにも宣言性を持たせることは可能という考え方もあります[[3](https://codeout.hatenablog.com/entry/2020/07/06/130233)]）。ただ、上記のような宣言的ネットワーキング / Declarative Networkingの困難さに立ち向かおうとしている人はきっといるはずです。非常に有望な「研究課題」と言えるでしょう。


### もう一つの宣言的ネットワーキングの側面


上記のようなネットワークのKubernetes的な宣言性は非常に興味深いトピックではありますが、私がJANOG45でお話をさせていただいたのは、これとはちょっとまた別の観点からの宣言的ネットワーキングの話で、SDNコントローラに宣言的なエンジンを用いることによっていかに堅牢で効率的なコントローラを実現できるか、という点でした。

ネットワークは非常に複雑な分散システムです。設定している最中に障害が起こり得ます。一般的にネットワーク設定は冪等ではありません（例えばポートを作った後にどこか別のノードで設定が失敗し、再度最初から同じ設定を入れようと思っても、すでにポートが作成済であればエラーになってしまうでしょう）。したがって、設定の途中で障害が起こったら単純に最初からやり直すだけではダメで、綺麗にロールバックしてやらなければいけません。ロールバックしている最中にまた別の障害が起こる、など多重障害が起こる可能性もあります。このようにさまざまなFailureシナリオにきちんと対応するするのは簡単ではありません。堅牢なSDNコントローラを作るのは非常に難しいことなのです。

Niciraの創業者の一人であるMatin Casadoから聞いた話なのですが、「Niciraでは幾つかのSDNコントローラを試作したが、上記のようなネットワークの複雑性からどれもなかなかうまくいかなかった。最終的に最も堅牢に動いたのはDatalogをベースにした宣言的なエンジンを使ったコントローラだった」とのことでした。この話を聞いて以来、私はずーっと宣言的なSDNコントローラに興味を抱いていました。

たまたまCiscoの河野さんも宣言的ネットワークに興味関心をお持ちである、ということを知り、それでは共同でJANOGでセッションをやってみましょう、ということになった次第です。河野さんからは宣言的ネットワークに関するさまざまな観点と考察を、私の方からは宣言的なSDNコントローラエンジンの適用事例の一つとして、OVNのコントローラにおけるDatalog技術の適用のお話をさせていただきました。OVN の概要とアーキテクチャに関しては[こちら](https://blog.shin.do/2015/12/ovn-open-virtual-network-%e3%81%ae%e7%b4%b9%e4%bb%8b)や[こちら](http://onic.jp/_cms/wp-content/uploads/2019/11/ONIC2019_Manabu-Ori.pdf)を参考にしてください。


### SDN コントローラとテーブル更新


一般的にSDNコントローラは多数のテーブルを扱います。その中でもフローテーブルは非常に巨大かつ動的に更新される必要があるものです。VMやコンテナは別のホストに移動する可能性があるので、それらのイベントに追従する形でフローテーブルを動的に更新してやらなければいけません。これを大規模な環境でやるのは思いの外大変です。

仮に2台のハイパーバイザ（Host1とHost2）と仮想マシンが3台（VM1、VM2、VM3）があり、VM1とVM2がHost1上で動いていて、VM3がHost2上で動いていたとしましょう。

![]({{site.baseurl}}/images/Figure-1.png)

SDNコントローラはそのような状況を表す以下のようなテーブルを持っているはずです。

Controller:
<table >
<tbody >
<tr >
VM
IP
Host
Port
</tr>
<tr >

<td >VM1
</td>

<td >10.10.10.101
</td>

<td >Host 1
</td>

<td >2
</td>
</tr>
<tr >

<td >VM2
</td>

<td >10.10.10.102
</td>

<td >Host 1
</td>

<td >3
</td>
</tr>
<tr >

<td >VM3
</td>

<td >10.10.10.103
</td>

<td >Host 2
</td>

<td >2
</td>
</tr>
</tbody>
</table>
SDNコントローラはこのテーブルから各ハイパーバイザ（Host1、Host2）のフローテーブルを以下のように設定することになります。

Host1:
<table >
<tbody >
<tr >
Match
Action
</tr>
<tr >

<td >dst=10.10.10.101
</td>

<td >output:2 (local port)
</td>
</tr>
<tr >

<td >dst=10.10.10.102
</td>

<td >output:3 (local port)
</td>
</tr>
<tr >

<td >dst=10.10.10.103
</td>

<td >output:1 (tunnel port)
</td>
</tr>
</tbody>
</table>
Host2:
<table >
<tbody >
<tr >
Match
Action
</tr>
<tr >

<td >dst=10.10.10.101
</td>

<td >output:1 (tunnel port)
</td>
</tr>
<tr >

<td >dst=10.10.10.102
</td>

<td >output:1 (tunnel port)
</td>
</tr>
<tr >

<td >dst=10.10.10.103
</td>

<td >output:2 (local port)
</td>
</tr>
</tbody>
</table>
このようなフローテーブルの設定を行うには、概ね以下のようなコードをコントローラに実装すれば良いでしょう。

    
    for h in Hosts:
      for vm in VM:
        if vm.host == h:
          h.add_flow(match="dst={vm.ip}", action="output:{vm.port}")
        else
          h.add_flow(match="dst={vm.ip}", action="output:{tunport}")


ここで、仮にVM2がHost1からHost2に移動（vMotion/Live Migration）したとしましょう。

![]({{site.baseurl}}/images/Figure-2.png)

コントローラはこのようなVMの移動を把握することができますので、自分が持っているテーブルを以下のように更新します。

Controller:
<table >
<tbody >
<tr >
VM
IP
Host
Port
</tr>
<tr >

<td >VM1
</td>

<td >10.10.10.101
</td>

<td >Host 1
</td>

<td >2
</td>
</tr>
<tr >

<td >VM2
</td>

<td >10.10.10.102
</td>

<td >Host 2
</td>

<td >3
</td>
</tr>
<tr >

<td >VM3
</td>

<td >10.10.10.103
</td>

<td >Host 2
</td>

<td >2
</td>
</tr>
</tbody>
</table>
当然この変更に合わせてハイパーバイザ上のフローテーブルも更新しなければなりません。一番簡単な方法は、上にあげたロジックを再度実行することです。こうすれば適切なフローテーブルが新たに各ハイパーバイザに設定されるはずです。

Host1:
<table >
<tbody >
<tr >
Match
Action
</tr>
<tr >

<td >dst=10.10.10.101
</td>

<td >output:2 (local port)
</td>
</tr>
<tr >

<td >dst=10.10.10.102
</td>

<td >output:1 (tunnel port)
</td>
</tr>
<tr >

<td >dst=10.10.10.103
</td>

<td >output:1 (tunnel port)
</td>
</tr>
</tbody>
</table>
Host2:
<table >
<tbody >
<tr >
Match
Action
</tr>
<tr >

<td >dst=10.10.10.101
</td>

<td >output:1 (tunnel port)
</td>
</tr>
<tr >

<td >dst=10.10.10.102
</td>

<td >output:3 (local port)
</td>
</tr>
<tr >

<td >dst=10.10.10.103
</td>

<td >output:2 (local port)
</td>
</tr>
</tbody>
</table>
しかしこれはあまりに非効率です。テーブルが小さいうちはこのような実直な方法でも良いかもしれませんが、テーブルサイズが大きくなってきた場合、この方法は明らかにCPUやネットワーク帯域の無駄になります。大規模環境では設定にかかる時間も無視できないほど大きくなってしまうでしょう。もう少し良い方法はないでしょうか？

誰もがすぐに思いつくのはおそらく「変更があったところだけ更新をする」という方法でしょう。いわゆるインクリメンタルなアップデート処理をすれば良いわけです。アルゴリズム的にはこんな感じになるでしょう。

    
    for h in Hosts:
      if old_vm.host == h:
        h.del_flow(...)
      else
        h.del_flow(...)
      if new_vm.host == h:
        h.add_flow(match="dst={new_vm.ip}", action="output:{new_vm.port}")
      else 
        h.add_flow(match="dst={newvm.ip}", action="output:{tunport}")


一見簡単そうな変更に見えるかもしれませんが、実は結構面倒な処理になります。このようなインクリメンタル処理のアルゴリズムはテーブル毎に個別に実装してやらなければいけません。OVNコントローラには数十個のテーブルがありますし、NSX-Tでは百を超えるテーブルがありますので、すべてのテーブルにこのようなインクリメンタル処理のアルゴリズムを個別に実装していくのはそう簡単な作業ではありません。また、テーブル間には依存関係がある場合があるので、一つの変更に対して、整合性を保つように複数のテーブルを注意深く更新していかなければなりません。設定の途中でエラーになった際のリカバリー処理なども考えると、テーブルごとにインクリメンタルな処理を実装する方式はかなり難度が高いことで、場合によってはコントローラの堅牢性を失いかねません。


### 宣言的コントローラエンジンとDatalog


OVNは大規模環境でも堅牢に動くProcution ReadyなSDNコントローラを目指していますので、まさに上記のような課題に直面することになります。

OVNではovn-northdとovn-controllerという2種類のコントローラが動いています。前者がネットワーク全体を管理するコントローラ、後者が各ハイパーバイザで動いているローカルのコントローラです。現在、ovn-northdコントローラにDatalogを使った宣言的な仕組みを適用する試みが行われています。

Datalogは一階述語論理に基づいた宣言的なプログラミング言語です。文法的にはPrologに似ていますが、Prologとはいくつかの点で異なり、演繹的データベースの問い合わせに使われることが多い DSL (Domain Specific Language) の一つです。

例えば、

    
    Parent(Anakin, Luke).
    Parent(Anakin, Leia).
    Sibling(c1, c2) :- Parent(p, c1), Parent(p, c2), c1 != c2.


というDatalogのプログラムからは

    
    Sibling(Luke, Leia).
    Sibling(Leia, Luke).


という結果が導出されます。":-" を含んだ行は「ルール」と呼ばれ、":-" の右側の項が全て成り立つ時に":-" の左側が成り立つ、という意味になります。":-" を含まない行は「事実（Fact）」と呼ばれます。上記のDatalogプログラムをざっくりと日本語にすると、


Lukeの親はAnakinである（事実）。Leiaの親はAnakinである（事実）。c1とc2が共通の親を持っており、c1とc2が異なれば、c1とc2は兄弟である（ルール）。


ということになり、そしてそこから導かれる結果は、


LukeはLeiaと兄弟である。LeiaはLukeと兄弟である。


となります。

Datalogを使うと、先に紹介した手続き的なフローテーブルの書き換え処理を宣言的に記述する事ができます。例えば、

    
    Host(1),
    Host(2);
    VM("VM1", "10.10.10.101", 1, 2),
    VM("VM2", "10.10.10.102", 1, 3),
    VM("VM3", "10.10.10.103", 2, 2);
    Flow(h, "dst=${ip}", "output:${p}") :- Host(h), VM(_, ip, h, p).
    Flow(h, "dst=${ip}", "output:1") :- Host(h), VM(_, ip, other, _), other != h.


というルールとFactから

    
    Flow(1, dst=10.10.10.101, output:2).
    Flow(1, dst=10.10.10.102, output:3).
    Flow(1, dst=10.10.10.103, output:1).
    Flow(2, dst=10.10.10.101, output:1).
    Flow(2, dst=10.10.10.102, output:1).
    Flow(2, dst=10.10.10.103, output:2).


という結果を自動的に演繹的に導出することができます。

また、VM2がHost 1からHost 2のポート3に移動したとすると、

    
    VM("VM2", "10.10.10.102", 2, 3),


のようにFactをアップデートをすれば、それを反映した正しい結果、

    
    Flow(1, dst=10.10.10.101, output:2).
    <span style="color: #ff0000;">Flow(1, dst=10.10.10.102, output:1).</span>
    Flow(1, dst=10.10.10.103, output:1).
    Flow(2, dst=10.10.10.101, output:1).
    <span style="color: #ff0000;">Flow(2, dst=10.10.10.102, output:3).</span>
    Flow(2, dst=10.10.10.103, output:2).


という結果を得ることができます。このように、Datalogを使うとフローテーブルがどのようなルールに則っているべきかを宣言的に記述するだけで良くなり、SDNコントローラを実装する人間はどのようにハイパーバイザのフローテーブルを設定すべきか、そのロジックを考えなくて済むようになります。これはとてもありがたいことです。


### Differential Datalog


Datalogを使うと、フローテーブルなど、SDNコントローラが扱わなければいけない各種テーブルの更新処理を宣言的に記述することができるようになります。手続き的に書くよりはるかにメンテナンス性の良い記述をすることができるようになるので、コントローラの堅牢性の向上に大きく貢献すると考えられます。

ただし、Datalogを使うだけでは、先ほど見たような「インクリメンタルな更新処理をいかに効率よく行うか」という問題を解決することにはなりません。そこで登場するのがDifferential Datalogです（DDlogと呼ばれることもあります）。

Differential DatalogはDifferential Dataflowという仕組みの上に作られていますので、まずDifferential Dataflowについて簡単に説明をしておきましょう。Differential Datalowはいわゆるビックデータ処理システムの一つと考えられますが、インクリメンタルな更新をとても効率よく処理することができるのが特徴です。

例えばTwitterのツイートにつけられているハッシュタグ間の相関関係を調べるために、ハッシュタグをノード、mentionをそれらのノードを繋ぐ辺として見立て、ハッシュタグの「グラフ」を作ることを考えてみましょう。いわゆるグラフから結合成分（Connected Component）を抽出する問題です。現在、Twitterで1日につぶやかれるツイートの数はおよそ5億個です。このような大量のデータからグラフの結合成分を抽出する処理はビッグデータ処理システムの代表格の一つであるMapReduceの得意分野の一つです。ただ、Twitterのように常に新しいツイートが行われる（すなわち入力が随時変化していく）ようなケースはMapReduceはあまり得意ではありません。入力が変化するたびに初めから再度計算をし直さなければいけないからです。一方、Differential Dataflowは入力の変化に応じてアウトプットを効率よく計算することができるため、入力の変化に応じた結果をリアルタイムに得ることができます。

Differential Datalogは、Differential Dataflowをバックエンドに用いたDatalog処理系で、VMware Research Groupで開発されました。Differential Dataflowをベースにしているので、優れたインクリメンタル更新処理性能を持ちつつ、Datalogによる宣言的なプログラミングが可能です。ユーザはどのようにインクリメンタル処理が行われるのかを意識する必要はなく、基本的にDatalogでルールやFactを記述すれば、入力の変化に追従して素早く出力を計算することができます。

Datalogの宣言性とDifferential Dataflowの優れたインクリメンタル更新処理性能を兼ね備えているDifferential Datalogは、SDNコントローラにおけ各種テーブルの更新処理にとても向いている仕組みです。

実際にDifferential Datalogの差分更新がどれくらい高速なのか、簡単な実験をしてみましょう。

仮に100台のハイパーバイザー（Host0〜99）上に10,000台のVM（VM0〜9999）があり、VM0がHost0からHost1に移動した際のフローテーブルを計算するとします。まずは、フローテーブルのあるべき姿を定義するルールをDifferential Datalogで宣言的に定義します。

    
    $ cat flowtable.dl
    input relation Host(id: bigint)
    input relation VM(vm: string, ip: string, host: bigint, port: bigint)
    output relation Flow(host: bigint, matches: string, action: string)
    Flow(h, "dst=${ip}", "output:${p}") :- Host(h), VM(_, ip, h, p).
    Flow(h, "dst=${ip}", "output:1") :- Host(h), VM(_, ip, other, _), other != h.


このルールを使用するためには、これのコードをDifferential Datalogのコンパイラ(ddlog)にかけます。そうすると、このルールが埋め込まれたRustのコードが生成されます。

    
    $ ddlog -i flowtable.dl -L ~/differential-datalog/lib/


ddlogによって生成されたRustコードをDifferential DataflowのライブラリとともにRustコンパイラでビルドすると、Differential Datalogで記述されたルールエンジンを備えた静的ライブラリとCLIツールが生成されます。

    
    $ (cd flowtable_ddlog/ && cargo build --release)
    Finished release [optimized] target(s) in 0.49s
    $ ls -l flowtable_ddlog/target/release/
    total 59100
    drwxrwxr-x 34 shindom shindom 4096 Mar 17 09:23 build
    drwxrwxr-x 2 shindom shindom 57344 Mar 17 09:36 deps
    drwxrwxr-x 2 shindom shindom 4096 Mar 17 09:23 examples
    -rwxrwxr-x 2 shindom shindom 15984984 Mar 17 09:36 flowtable_cli
    -rw-rw-r-- 1 shindom shindom 2325 Mar 17 09:36 flowtable_cli.d
    drwxrwxr-x 10 shindom shindom 4096 Mar 17 09:35 incremental
    -rw-rw-r-- 2 shindom shindom 42098680 Mar 17 09:35 libflowtable_ddlog.a
    -rw-rw-r-- 1 shindom shindom 2260 Apr 12 07:14 libflowtable_ddlog.d
    -rw-rw-r-- 1 shindom shindom 219 Mar 17 09:23 libflowtable_ddlog.la
    -rw-rw-r-- 2 shindom shindom 2350122 Mar 17 09:35 libflowtable_ddlog.rlib


今回の例だと、flowtable.dlに記述されたルールエンジンを持つライブラリlibflowtable_ddlog.a、libflowtable_ddlog.rlibと、CLIコマンドのflowtable_cliが生成されます。

本来であれば、このlibflowtable_ddlog.(a,rlib)を使ったコードをRust、C/C++、Java、Goなどで書くべきなのですが、今回は生成されたCLIツールを使って処理性能を簡易的に調べてみることにします。

100台のホスト上に10,000台の仮想マシンが動いている初期状態の作成と、そのうちの一台の仮想マシン（VM0）がホスト0からホスト1に移動をDifferential Datalogで書くと以下のようになります。これで、VMが一台ホストを移動した際のフローテーブルの更新を模擬することができます。また、かかった時間を調べるために、各処理の途中で時刻を表示するようにしてあります。

    
    start;
    insert Host(0);
    insert Host(1);
          :
    insert Host(98);
    insert Host(99);
    insert VM("VM0", "10.10.0.0", 0, 0);
    insert VM("VM1", "10.10.0.1", 1, 1);
    insert VM("VM2", "10.10.0.2", 2, 2);
          :
    insert VM("VM9997", "10.10.99.97", 97, 9997);
    insert VM("VM9998", "10.10.99.98", 98, 9998);
    insert VM("VM9999", "10.10.99.99", 99, 9999);
    
    echo initial setup started:;
    timestamp;
    commit;
    echo initial setup finished:;
    timestamp;
    start;
    delete VM("VM0", "10.10.0.0", 0, 0);
    insert VM("VM0", "10.10.0.0", 1, 0);
    echo update started:;
    timestamp;
    commit;
    echo update finished:;
    timestamp;


これをflowtable_cliに食わせてやると以下のような結果になります。

    
    $ ./flowtable_ddlog/target/release/flowtable_cli --no-print < flowtable.dat
    initial setup started:
    Timestamp: 490458325506557
    initial setup finished:
    Timestamp: 490467825165438
    update started:
    Timestamp: 490467825243578
    update finished:
    Timestamp: 490467829061228


これを見て分かる通り、最初のフローテーブルの作成には約9.5秒かかっていますが、VMが移動した後のフローテーブルの更新には0.00381765秒しかかかっていません。フローテーブルの更新を実直に再計算するとフローテーブルの初期作成とほぼ同じ時間がかかると考えられるので、Differential Datalogのインクリメンタルアップデートの素晴らしい処理性能をご理解いただけるのではないかと思います。


### Differential Datalog の OVN への適用


上で述べたように、現在OVNのovn-northdコントローラをDifferential Datalogを使って再実装する努力が行われています。この作業はOVNのGithubリポジトリのddlog-dev-v2というブランチで開発作業が進められています。

Differential Datalogを使ってテーブル処理を書く主なメリットは、

  * よりメンテナンス性の高いコードになる
  * インクリメンタルな更新による性能向上

の２点です。まず、メンテナンス性の高いコードになる、とう点をみてみましょう。以下は、OVNのmeterおよびmeter_bandテーブルを管理する部分のコードです（OVNはC言語で書かれています）。

    
    struct band_entry {
        int64_t rate;
        int64_t burst_size;
        const char *action;
    };
    
    static int
    band_cmp(const void *band1_, const void *band2_)
    {
        const struct band_entry *band1p = band1_;
        const struct band_entry *band2p = band2_;
    
        if (band1p->rate != band2p->rate) {
            return band1p->rate > band2p->rate ? -1 : 1;
        } else if (band1p->burst_size != band2p->burst_size) {
            return band1p->burst_size > band2p->burst_size ? -1 : 1;
        } else {
            return strcmp(band1p->action, band2p->action);
        }
    }
    
    static bool
    bands_need_update(const struct nbrec_meter *nb_meter,
                      const struct sbrec_meter *sb_meter)
    {
        if (nb_meter->n_bands != sb_meter->n_bands) {
            return true;
        }
    
        /* A single band is the most common scenario, so speed up that
         * check. */
        if (nb_meter->n_bands == 1) {
            struct nbrec_meter_band *nb_band = nb_meter->bands[0];
            struct sbrec_meter_band *sb_band = sb_meter->bands[0];
    
            return !(nb_band->rate == sb_band->rate
                     && nb_band->burst_size == sb_band->burst_size
                     && !strcmp(sb_band->action, nb_band->action));
        }
    
        /* Place the Northbound entries in sorted order. */
        struct band_entry *nb_bands;
        nb_bands = xmalloc(sizeof *nb_bands * nb_meter->n_bands);
        for (size_t i = 0; i < nb_meter->n_bands; i++) {
            struct nbrec_meter_band *nb_band = nb_meter->bands[i];
    
            nb_bands[i].rate = nb_band->rate;
            nb_bands[i].burst_size = nb_band->burst_size;
            nb_bands[i].action = nb_band->action;
        }
        qsort(nb_bands, nb_meter->n_bands, sizeof *nb_bands, band_cmp);
    
        /* Place the Southbound entries in sorted order. */
        struct band_entry *sb_bands;
        sb_bands = xmalloc(sizeof *sb_bands * sb_meter->n_bands);
        for (size_t i = 0; i < sb_meter->n_bands; i++) {
            struct sbrec_meter_band *sb_band = sb_meter->bands[i];
    
            sb_bands[i].rate = sb_band->rate;
            sb_bands[i].burst_size = sb_band->burst_size;
            sb_bands[i].action = sb_band->action;
        }
        qsort(sb_bands, sb_meter->n_bands, sizeof *sb_bands, band_cmp);
    
        bool need_update = false;
        for (size_t i = 0; i < nb_meter->n_bands; i++) {
            if (nb_bands[i].rate != sb_bands[i].rate
                || nb_bands[i].burst_size != sb_bands[i].burst_size
                || strcmp(nb_bands[i].action, sb_bands[i].action)) {
                need_update = true;
                goto done;
            }
        }
    
    done:
        free(nb_bands);
        free(sb_bands);
    
        return need_update;
    }
    
    /* Each entry in the Meter and Meter_Band tables in OVN_Northbound have
     * a corresponding entries in the Meter and Meter_Band tables in
     * OVN_Southbound.
     */
    static void
    sync_meters(struct northd_context *ctx)
    {
        struct shash sb_meters = SHASH_INITIALIZER(&sb_meters);
    
        const struct sbrec_meter *sb_meter;
        SBREC_METER_FOR_EACH (sb_meter, ctx->ovnsb_idl) {
            shash_add(&sb_meters, sb_meter->name, sb_meter);
        }
    
        const struct nbrec_meter *nb_meter;
        NBREC_METER_FOR_EACH (nb_meter, ctx->ovnnb_idl) {
            bool new_sb_meter = false;
    
            sb_meter = shash_find_and_delete(&sb_meters, nb_meter->name);
            if (!sb_meter) {
                sb_meter = sbrec_meter_insert(ctx->ovnsb_txn);
                sbrec_meter_set_name(sb_meter, nb_meter->name);
                new_sb_meter = true;
            }
    
            if (new_sb_meter || bands_need_update(nb_meter, sb_meter)) {
                struct sbrec_meter_band **sb_bands;
                sb_bands = xcalloc(nb_meter->n_bands, sizeof *sb_bands);
                for (size_t i = 0; i < nb_meter->n_bands; i++) {
                    const struct nbrec_meter_band *nb_band = nb_meter->bands[i];
    
                    sb_bands[i] = sbrec_meter_band_insert(ctx->ovnsb_txn);
    
                    sbrec_meter_band_set_action(sb_bands[i], nb_band->action);
                    sbrec_meter_band_set_rate(sb_bands[i], nb_band->rate);
                    sbrec_meter_band_set_burst_size(sb_bands[i],
                                                    nb_band->burst_size);
                }
                sbrec_meter_set_bands(sb_meter, sb_bands, nb_meter->n_bands);
                free(sb_bands);
            }
    
            sbrec_meter_set_unit(sb_meter, nb_meter->unit);
        }
    
        struct shash_node *node, *next;
        SHASH_FOR_EACH_SAFE (node, next, &sb_meters) {
            sbrec_meter_delete(node->data);
            shash_delete(&sb_meters, node);
        }
        shash_destroy(&sb_meters);
    }
    


かなりの量のコードで、それなりに複雑です。

一方、これと同じ処理をDifferential Datalogで記述すると、

    
    /* Meter_Band table */
    for (mb in nb.Meter_Band) {
        sb.Out_Meter_Band(.uuid_name = uuid2name(mb._uuid),
                          .action = mb.action,
                          .rate = mb.rate,
                          .burst_size = mb.burst_size)
    }
    
    /* Meter table */
    for (meter in nb.Meter) {
        sb.Out_Meter(.name = meter.name,
                     .unit = meter.unit,
                     .bands = set_map_uuid2name(meter.bands))
    }
    


のようになります。非常にシンプルに記述できますので、可読性に優れ、バグの混入の可能性も低くすることができると考えられます。これがDifferential Datalogを使う一つ目のメリットです。

Differential Datalogを適用することのもう一つのメリットは、インクリメンタルアップデートの性能向上です。eBayのHan Zhouが、Cで実装されたovn-northdを使った場合とDifferential Datalog版のovn-northdを使った時のスケールテストの結果を共有してくれています [1]。結果は以下の通りです。

* Cバージョン　67分47秒
* DDlogバージョン　7分39秒

Differential Datalogを使うことで、およそ10倍程度の性能向上があったことになります。残念ながらこのテストの詳しいテストシナリオははっきりしないのですが、OVNを使ったOpenStack環境でスケール試験を行なったようです。

また、RedHatのMark Michelsonらが別の性能評価をしています。Markが行ったテストは、1つの論理ルータの下に159個の論理スイッチを作り、それぞれの仮想スイッチに92個の論理ポートを作った上でACLを適用する、というものです [2]。その結果をグラフしたのが以下になります。

[embed]https://imgur.com/bIsbeMN[/embed]

このテストは約15,000論理ポートを順次作成していく、というシナリオなので、インクリメンタルな処理性能を測るのにあまり適したシナリオではありませんが（インクリメンタルな処理性能が最も顕著に現れるのは、すでにある大規模な環境に若干の更新をした場合です）、それでもCでの実装に比べるとDifferential Datalog (DDlog)の実装の方がかなりいい性能が得られていることが分かります。


### 今後に向けて


テクノロジーとしては非常に有望なDifferential Datalogですが、課題もあります。

Differential Datalogは従来の手続型言語とは大きく異なります。今までCで書いてきた人が「明日からDatalogで書いてね！」と言われてもパラダイムが全く異なりますのでなかなか難しいのではないかと思います。ツールチェーンも少々複雑です。上述の通り、Differential DatalogのコンパイラRustのコードを吐き出します。また、Differential Datalogのコンパイラ自身はHaskellで書かれています。OVNはCで書かれているので、Differential Datalog版のovn-northdの動きを調べる際には、CやDifferential DatalogだけではなくRustや場合によってはHaskellの知識も求められることになります。これらの言語およびそこで使われているツールチェーンを一通り学習するのはそれなりの時間を要するでしょう。また、Differential Datalogにはデバッグの機能も盛り込まれてはいるものの、皆さんが普段使い慣れているデバッガに比べるとまだまだプリミティブです。

ovn-northdのDifferential Datalog実装は現時点ではまだmasterとは別ブランチとなっており、masterブランチではCを使った開発が引き続き行われています。したがって、Cで書かれた新機能がmasterブランチに追加されたら、その機能をDifferential Datalogで実装して、追従していかなければなりません。これをタイムリーにやっていくのものなかなか大変です。完全にDifferential Datalogに移行ができればこのような悩みは無くなりますが、それにはもっともっとテストをしなければなりません。具体的にはCバージョンで生成されるテーブルとDifferential Datalogバージョンから生成されるテーブルがいかなる場合（障害時も含めて）で差異がない、という状態にならないとCバージョンからDifferential Datalog版に移行することはできません。これにはまだ時間がかかると思われます。また、現時点でDifferential Datalogが適用されているのはovn-northdだけです。理論的にはローカルコントローラであるovn-controllerの方にもDifferential Datalogを適用することができますが、まだ未対応です。

これらの課題がありつつも、Differential Datalogは定性的には素晴らしい特性を持っていますので、今後はOVNだけではなく、大規模なSDN環境における宣言的かつインクリメンタル処理を効率よく行う仕組みとして広く使われていくようになるのではないかと思います。NSX-TのコントローラはNiciraのコントローラと同様、Datalogベースの宣言的なエンジンを備えていますが、そこにDifferential Datalogによる差分更新機能を適用することも検討されているようです。また、Differential Datalogは非常に汎用的な仕組みですので、今後はSDNだけでなく広く分散システムに対する宣言的なアプローチとして使われていく可能性もあります。引き続き注視をしていきたいと思います。


#### 参考リンク

* [1] https://mail.openvswitch.org/pipermail/ovs-dev/2019-July/360604.html
* [2] https://mail.openvswitch.org/pipermail/ovs-dev/2019-July/360889.html
* [3] https://codeout.hatenablog.com/entry/2020/07/06/130233

Photo by [Irvan Smith](https://unsplash.com/@mr_vero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/programming?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
