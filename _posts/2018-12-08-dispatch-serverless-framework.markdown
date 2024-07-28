---
date: 2018-12-08 13:48:36+00:00
layout: post
title: Dispatch Serverless Framework
abstract: VMware がオープンソースで開発中の Serverless Framework (FaaS) について
image: '/images/celine-chung-1CQsrW0oT1g-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
language:
- 日本語
keywords:
- cloudevent
- Dispatch
- FaaS
- Knative
- kong
- kubeless
- Kubernetes
- openfaas
- riff
- Serverless
- vmware
---

![]({{site.baseurl}}/images/logo-small-200-1-1024x355.png)

Dispatch は Function as a Service (FaaS) のためのフレームワークで、VMware が中心となってオープンソースで開発が進められています。

一般的に FaaS は大きく分けるとホスティングされたサービスとして提供されるもの（e.g. AWS Lambda、Azure Functions、Google Cloud Function、など）と、自らインストールして使うもの（Apache OpenWhisk、OpenFaaS、Fission、Kubreless、Riff、など）がに分けられますが、この分類でいくと Dispatch は後者に分類されるものになります。ただし、Dispatch はいわゆる FaaS 機能の中核となる部分（エンジン）を提供しているわけでありません。OpenFaaS、Kubeless、Riff など、現在 FaaS エンジンには多くの選択肢があります。このような状況の中で Dispatch がこれらの FaaS エンジンと似たような機能を提供してもあまり価値がありません。まだ、どの FaaS エンジンが今後主流になって行きそうかも分からない状況なので、特定のエンジンを前提にするのもあまり筋がよろしくなさそう、ということで、Dispatch は FaaS エンジン部分を pluggable にして、その周辺機能を実装する、という戦略を取っています。Dispatch が「FaaS フレームワーク」と呼ばれる所以です。

現在、Dispatch には2つのバージョン（ブランチ）があります。一つは Dispatch-Solo と呼ばれるもの、もう一つは Dispatch-Knative と呼ばれるものです。

Knative というのは Google Cloud Next 2018 で発表されたプロジェクトで、Google、Pivotal、RedHat、IBM などが開発に関わっています。その狙いは Dispatch に近く、FaaS / Serverless に対して、Build / Serving / Events といった基本的な building block を提供しよう、というものです。Dispatch は Knative が発表された際に、Knative と競合路線を取るのではなく、Knative と協調していく、という判断をしました。つまり、Knative が提供する基本機能（Build、Serving、Events）は Knative のものを利用し、マルチテナンシーなど、Knative にはない Dispatch ならではの機能を Dispatch として提供していこう、というわけです。この Knative  をベースにした Dispatch が Dispatch-Knativeで、商用環境で使われることを想定したフル機能版の Dispatch の開発を目指しています。Dispatch-Knative はプラットフォームとして Kubernetes を利用しており、現在盛んに開発が行われています。

![]({{site.baseurl}}/images/dispatch-v2-architecture-1024x501.png)Dispatch-Knative アーキテクチャ

一方、Dispatch-Solo は、Dispatch-Knative の持つ様々な依存関係を極力最小化して、出来るだけ手軽に使えるものを提供することを目的に作られています。Dispatch-Solo はプラットフォームに Kubernetes を必要とせず、Docker だけで動くようになっています。また Photon OS をベースとした OVA としてパッケージされていますので、手軽に ESXi、VMware Workstation、VMware Fusion 上に展開して使うことができます。Dispatch-Solo は Dispatch-Knative とほぼ同様な機能を提供していますが、スケールアウト、マルチテナンシー、サービス連携の機能がない等、一部機能に制限があります。プロダクション利用ではなく、お手軽に利用できることに注力をしているバージョンです。

![]({{site.baseurl}}/images/dispatch-solo-architecture-1024x655.png)Dispatch-Solo アーキテクチャ

Dispatch は以下のような機能を持っています。  

  * Kubernetes 上で動作（Dispatch-Knative のみ）
  * Pluggable FaaS Engine (OpenFaaS, Riff, Kubeless)
  * Runtime (Pyhton3, Nodejs, Java, Powershell, Clojure)
  * Let’s Encrypt サポート
  * OpenTracing 対応
  * 複数 IDP (Google, Auth0, vIDM, GitHub) のサポート（Dispatch-Knativeのみ）
  * Organization に基づいたマルチテナンシー（Dispatch-Knative のみ）
  * RBAC 機能
  * Cloud Events 対応
  * 拡張可能なイベントドライバ
  * Open Service Broker 統合（Dispatch-Knative のみ）

Dispatch によって作られた Function は大きく分けて2つの方法で呼び出されます。一つは Function に API の Endpoint を設けて、それを明示的（同期的）に呼び出す方法です。Dispatch には Kong というオープンソースの API Gateway が組み込まれており、これを利用して API Endpoint を Function に紐づけることができます。もう一つの Function の呼び出し方法はイベントドライバによる呼び出しです。イベントドライバがサポートしているイベントと Dispatch によって作られた Function を binding することにより、非同期的に発生するイベントによって Function を呼び出すことができます。

現在 Dispatch がサポートしているイベントドライバは以下の通りです。  

  * CloudEvent
  * vCenter
  * AWS
  * EventGrid
  * Cron

VMware が開発を主導していることもあり、vCenter のイベントドライバがサポートされているのが特徴的です。これにより例えば ESXi 上の仮想マシンの Power ON/OFF などをトリガーにして Dispatch に登録された Function を実行しオペレーションの自動化を実現する、といったことが可能になります。

今回はお手軽に利用できる Dispatch-Solo を使って、簡単に動作をみてみることにしましょう。

Dispatch-Solo の最新版の OVA は以下の方法で入手することができます。

    $ curl -OL <a href="http://vmware-dispatch.s3-website-us-west-2.amazonaws.com/ova/dispatch-latest-solo.ova">http://vmware-dispatch.s3-website-us-west-2.amazonaws.com/ova/dispatch-latest-solo.ova</a>

次にこの OVA ファイルをデプロイします。今回は VMware Fusion を使うことにします。アプリケーションおよびネットワークの設定を聞かれますが、root パスワードだけ設定すれば大丈夫です。

![]({{site.baseurl}}/images/Screen-Shot-2018-12-08-at-16.08.49.png)

Dispatch のバイナリをダウンロードしてインストールします。現在 Mac 用と Linux 用のバイナリが用意されています（残念ながら Windows 用のバイナリは今のところ用意されていません）。
    
    (Mac/Linux共通)<br>$ export LATEST=$(curl -s https://api.github.com/repos/vmware/dispatch/releases/latest | jq -r .name)

    (Macの場合)<br>$ curl -OL https://github.com/vmware/dispatch/releases/download/$LATEST/dispatch-darwin$ chmod +x dispatch-darwin$ mv dispatch-darwin /usr/local/bin/dispatch

    (Linuxの場合)<br>$ curl -OL https://github.com/vmware/dispatch/releases/download/$LATEST/dispatch-linux$ chmod +x dispatch-linux$ mv dispatch-linux /usr/local/bin/dispatch

インストールした Dispatch-Solo の仮想マシンに振られた IP アドレスを確認し、それを DISPATCH_HOST という環境変数に設定して Dispatch が使用する設定ファイルを作っておきます。

![]({{site.baseurl}}/images/Screen-Shot-2018-12-08-at-16.13.02.png)

    $ export DISPATCH_HOST=10.156.250.95<br>$ cat << EOF > $HOME/.dispatch/config.json<br>{<br>    "current": "solo",<br>    "contexts": {<br>        "solo": {<br>            "host": "${DISPATCH_HOST}",<br>            "port": 8080,<br>            "scheme": "http",<br>            "organization": "dispatch",<br>            "cookie": "cookie",<br>            "insecure": true,<br>            "api-https-port": 443<br>        }<br>    }<br>}<br>EOF

これで Dispatch を使う準備が一通り整いました。ここから Dispatch を使って Function を作っていきます。まず、Funciton を実行するベースとなるイメージを作ります。各ランタイム（言語）ごとにイメージが用意されていますが、"create seed-image" コマンドを使うと、Node.js、Python 3、Java、Powershell 用のイメージを一発で作ることができるので便利です。

    $ dispatch create seed-images<br>Created BaseImage: nodejs-base<br>Created BaseImage: python3-base<br>Created BaseImage: powershell-base<br>Created BaseImage: java-base<br>Created Image: nodejs<br>Created Image: python3<br>Created Image: powershell<br>Created Image: java<br>$ dispatch get images<br>     NAME    | URL |    BASEIMAGE    |  STATUS  |         CREATED DATE         <br>--------------------------------------------------------------------------<br>  java       |     | java-base       | CREATING | Sat Dec  8 16:16:24 JST 2018 <br>  nodejs     |     | nodejs-base     | CREATING | Sat Dec  8 16:16:24 JST 2018 <br>  powershell |     | powershell-base | CREATING | Sat Dec  8 16:16:24 JST 2018 <br>  python3    |     | python3-base    | CREATING | Sat Dec  8 16:16:24 JST 2018 <br>

"create seed-image" コマンド実行直後は "get image" コマンド結果の STATUS の欄が "CREATING" になっていますが、しばらく待つと全てのランタイムの STATUS が "READY" になります。

    $ dispatch get images<br>     NAME    |                         URL                          |    BASEIMAGE    | STATUS |         CREATED DATE         <br>-------------------------------------------------------------------------------------------------------------------------<br>  java       | dispatch/dd3c73a4-2cf7-40db-8944-6c0d11f9157b:latest | java-base       | READY  | Sat Dec  8 16:16:24 JST 2018 <br>  nodejs     | dispatch/bbd15fbc-cfef-43d6-9d56-105c359fdedd:latest | nodejs-base     | READY  | Sat Dec  8 16:16:24 JST 2018 <br>  powershell | dispatch/1efa88c4-55cc-4bf8-a6d9-0a80c084cc89:latest | powershell-base | READY  | Sat Dec  8 16:16:24 JST 2018 <br>  python3    | dispatch/bcd7a157-117e-47c3-a276-3012d07b1848:latest | python3-base    | READY  | Sat Dec  8 16:16:24 JST 2018 


次に実際に実行される Function を作っていきます。今回は Python を使って簡単な関数を作ってみましょう。

    $ cat << EOF > <a href="http://hello.py/">hello.py</a><br>def handle(ctx, payload):<br>    name = "Noone"<br>    place = "Nowhere"<br>    if payload:<br>        name = payload.get("name", name)<br>        place = payload.get("place", place)<br>    return {"myField": "Hello, %s from %s" % (name, place)}<br>EOF

handle() という関数がいわゆるハンドラ関数で、関数のエントリポイントとなります。この handle() という関数は2つの引数を取ります。"ctx" は Context の意で、ホスト側から渡される情報（例えば Secret など）が入ってきます。一方、"payload" の方は HTTP(S) のリクエストで渡されれてくる情報が入ってきます。今回の例では HTTP POST されてくる "name" というアトリビュートと "place" というアトリビュートを payload から取り出しています。

次に "create function" コマンドで今しがた作った関数を Dispatch に登録します。今回はその関数に "hello-world" という名前を付けることにします。

    $ dispatch create function hello-world --image python3 ./hello.py<br>Created function: hello-world<br>

"get function" コマンドで関数が作られたのを確認することができます（STATUS の欄が READY になれば OK です）。
    
    $ dispatch get function<br>     NAME     |                       FUNCTIONIMAGE                       | STATUS |         CREATED DATE         <br>--------------------------------------------------------------------------------------------------------------<br>  hello-world | dispatch/func-02625403-7cf4-477f-aca8-793a9cc2d55c:latest | READY  | Sat Dec  8 16:18:18 JST 2018 

では、この関数を実際に呼び出してみましょう。それには "exec" コマンドを使います。HTTP のパラメータは "--input" に続く文字列で指定できます。
    
    $ dispatch exec hello-world --input '{"name": "Jon", "place": "Winterfell"}' --wait<br>{<br>    "blocking": true,<br>    "executedTime": 1544253540926528854,<br>    "faasId": "02625403-7cf4-477f-aca8-793a9cc2d55c",<br>    "finishedTime": 1544253540930869980,<br>    "functionId": "4a550524-85da-4d28-9d5f-47b35a425cd8",<br>    "functionName": "hello-world",<br>    "input": {<br>        "name": "Jon",<br>        "place": "Winterfell"<br>    },<br>    "logs": {<br>        "stderr": null,<br>        "stdout": [<br>            "Serving on <a href="http://0.0.0.0:9000/">http://0.0.0.0:9000</a>"<br>        ]<br>    },<br>    "name": "440256c2-7710-4ea1-bc20-5231c2bd4363",<br>    "output": {<br>        "myField": "Hello, Jon from Winterfell"<br>    },<br>    "reason": null,<br>    "secrets": [],<br>    "services": null,<br>    "status": "READY",<br>    "tags": []<br>}

"--wait" というパラメータはこの関数の呼び出しを同期的に行う、という意味です。"--wait" を付けないと、関数は実行されますが、"exec" コマンドはその関数の終了を待たずに終了しますので、結果は別途確認する必要があります。今回は "--wait" を付けて実行していますので、返された JSON の中の "output" に期待された結果が返ってきているのが確認できます。

次に、この関数に API Endpoint を設定してみましょう。API Endpoint の設定には "create api" コマンドを使用します。

    $ dispatch create api api-hello hello-world --method POST --path /hello<br>Created api: api-hello<br>shindom-a01:Downloads shindom$ dispatch get api api-hello<br>    NAME    |  FUNCTION   | PROTOCOL  | METHOD | DOMAIN |                   PATH                   |  AUTH  | STATUS | ENABLED <br>--------------------------------------------------------------------------------------------------------------------------------<br>  api-hello | hello-world | http      | POST   |        | <a href="http://10.156.250.95:8081/dispatch/hello">http://10.156.250.95:8081/dispatch/hello</a> | public | READY  | true    <br>            |             | https     |        |        | <a href="https://10.156.250.95/dispatch/hello">https://10.156.250.95/dispatch/hello</a>     |        |        |         <br>--------------------------------------------------------------------------------------------------------------------------------<br>

では、この API Endpoint を呼び出してみましょう。

    $ curl <a href="http://10.156.250.95:8081/dispatch/hello">http://10.156.250.95:8081/dispatch/hello</a> -H "Content-Type: application/json" -d '{"name": "Motonori", "place": "Tokyo"}'<br>{"myField":"Hello, Motonori from Tokyo"}

さて、ここまでは関数を CLI もしくは API Endpoint から同期的に関数を呼び出す例をみてきましたが、今度はイベントドライバによる非同期的な関数の呼び出しを試してみましょう。

Dispatch でサポートされているいくつかのイベントドライバの中から、今回は最もシンプルな Cron Event Driver を使ってみましょう。Cron Event Driver はその名の通りで、定期的に関数を呼び出してくれるイベントドライバで、UNIX の cron と同じような書式で起動するタイミングを指定をすることができます。

Dispatch のイベントドライバを使うためには3つのステップを踏む必要があります。1つ目は Event Driver Type の設定です。今回はdispatchframework にある Cron Event Driver に "cron" という名前を付けてやります。

    $ dispatch create eventdrivertype cron dispatchframework/dispatch-events-cron:0.0.1<br>Created event driver type: cron

次に行うのは Event Driver の登録です。先ほど作った "cron" という Event Driver Type のドライバを作り "every-10-seconds" という名前を付け、イベントドライバに cron 書式（e.g. "0/10 * * * *"）で実行されるべきタイミングを指定します。

    $ dispatch create eventdriver cron --name every-10-seconds --set cron="0/10 * * * *"<br>Created event driver: every-10-seconds

最後にイベントに対する subscription の設定を行います。具体的には Funciotn（今回の例で hello-world）とイベントドライバが生成するイベント（今回の例では cron.trigger）の紐付け（ binding） 設定を行うことになります。

    $ dispatch create subscription hello-world --name cron-sub --event-type cron.trigger<br>created subscription: cron-sub

これで設定は終わりです。この設定を行うと、10秒ごとに hello-world という Function が呼ばれることになり、その結果は "get runs" コマンドで確認をすることができます。

    $ dispatch get runs<br>                   ID                  |  FUNCTION   | STATUS |               STARTED               |              FINISHED               <br>-------------------------------------------------------------------------------------------------------------------------------------<br>  5f9a79e0-1612-4169-90e8-e28e1f26640c | hello-world | READY  | 2018-12-08T16:35:30.003557189+09:00 | 2018-12-08T16:35:30.006887242+09:00 <br>  55fc2066-bd53-4e04-b0b6-bc1bb7940833 | hello-world | READY  | 2018-12-08T16:35:20.003363373+09:00 | 2018-12-08T16:35:20.007476442+09:00 <br>  19878a0d-312d-44ca-94b7-dc4999caf433 | hello-world | READY  | 2018-12-08T16:35:11.482092927+09:00 | 2018-12-08T16:35:11.485768711+09:00 <br>  14d2f6dc-2aa6-4bef-b9c4-a1cd6db6fe1f | hello-world | READY  | 2018-12-08T16:35:00.003163345+09:00 | 2018-12-08T16:35:00.007290792+09:00 <br>  d55328b0-53a5-48b0-b828-0ddff6648cd9 | hello-world | READY  | 2018-12-08T16:34:50.002078415+09:00 | 2018-12-08T16:34:50.006185049+09:00 <br>  3bc0d86b-1d73-4a94-bd7b-142f6d4743ae | hello-world | READY  | 2018-12-08T16:34:41.469205488+09:00 | 2018-12-08T16:34:41.472881958+09:00 <br>

いかがでしたでしょうか？　Dispatch はまだまだ若いプロジェクトで、活発に開発が行われています。みなさん、是非 Dispatch を使ってみてフィードバックをしていただだけると助かります。また、Dispatch はまだまだ把握しやすい規模のプロジェクトですので、開発にも参加しやすいと思います。色々な形で皆で Dispatch を盛り上げていけると嬉しいです！  

Photo by <a href="https://unsplash.com/@celinecky?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Celine Chung</a> on <a href="https://unsplash.com/photos/white-concrete-wall-with-glass-window-1CQsrW0oT1g?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  