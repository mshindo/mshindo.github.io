---
date: 2020-01-06 14:13:12+00:00
layout: post
title: Antrea - Yet Another CNI Plug-in for Kubernetes
categories:
- コンピュータとインターネット
language:
- 日本語
tags:
- antrea
- cni
- Kubernetes
- octant
- Open vSwitch
- OVS
---

### Antreaとは


2019年11月にSan Diegoで行われたKubeConのタイミングで、Kubernetesのためのネットワークプラグインとして、新たにAntreaというプロジェクトが公開されました。CNI (Container Networking Interface) に準拠したプラグインで、現在はVMwareが中心となって開発をしていますが、オープンソースのプロジェクトですので、今後は徐々にコミュティーからのサポートが得られていくのではないかと思います。GitHubのリポジトリは[ここ](https://github.com/vmware-tanzu/antrea)にあります。

"Antrea"という名前ですが、実は[Antrea Net](https://en.wikipedia.org/wiki/Antrea_Net)から来ています。Antreaはフィンランドに近いロシアの街で、世界最古の漁網が見つかった場所として知られており、その網は見つかった場所にちなみんで「Antrea Net」と呼ばれています。Kubernetesという名前は「操舵手」というギリシャ語から付けられたものですが、これに倣ってKubernetes関連のプロジェクトには船や海にちなんだ名前（例えばhelm/tillerなど）がつけられることが多くなっています。一方、CNIのプラグインであるCalicoやFlannelはどちらも繊維（ファブリック）に関係のある名前になっています（「ネットワーク」と「ファブリック」をかけている？？）。この「海」と「ファブリック」の二つの要素を兼ね合わせた「漁網」であるAntrea Netに因んでAntreaという名前がつけられました。

Antreaの特徴は以下のような点になります。



 	
  * Open vSwitchによるデータプレーン

 	
  * Network Policyのサポート（iptablesからの脱却）

 	
  * LinuxとWindowsサポート（予定）

 	
  * 幅広いプラットフォームサポート

 	
  * Octantとの連携

 	
  * rspanやIPFIXのサポートなど（予定）


Antreaは仮想スイッチとして広く使われているOpen vSwitch (OVS)をデータパスに使用しているのがもっとも大きな特徴です。OVSをデータパスに使用することでいくつかのメリットが得られます。KubbernetesのNetwork PolicyやServiceのロードバランスを実現するために、他のCNIプラグインの多くはiptablesを使っています。iptablesは古くからある実績豊富な機能ではありますが、性能はあまりよろしくありません。iptabblesはルール数が多くなると、CPUに対する負荷、遅延、設定時間が非常に大きくなることが知られています。複雑なNetwork Policyや多数のServiceを使っているKubernetes環境では、iptablesベースのCNIでは期待した性能が得られないことが予想されます。一方、AntreaはOVSをデータプパスに使っているため、iptablesに起因する性能的問題を回避することができます。

OVSはLinux以外のOS（例えばWindows）もサポートしているので、多様な環境にまたがったKubernetesネットワーク環境を実現することができます。現時点のAntreaはLinuxのみのサポートですが、将来的にはWindowsもサポートする予定です。KubernetesはすでにWindowsをサポートしていますので、AntreaがWindowsをサポートするようになれば、LinuxとWindowsにまたがったネットワークを作って、そこに統一的なNetwork Policyを適用することができるようになります。また、OVSは仮想スイッチの機能だけではなく、他にもさまざまな機能（NetFlow/sFlow/IPFIXやRSPANなど）をサポートしています。将来的にはこれらの機能をAntreaから利用できるようにしていく予定です。

Antreaは幅広いプラットフォームで実行されれることを想定しています。オンプレの仮想環境はもちろん、ベアメタルや各種パブリッククラウドで実行することを想定して作られています。AntreaはKindで作ったKubernetesクラスタもサポートしていますので、簡易的にクラスタを用意してテストをすることもできて便利です。ただし、現在はLinuxの上のKindのみのサポートになります。MacOSのKindはまだサポートされてません。Kindで作ったクラスタでAntreaを動かす場合は、OVSのカーネルモジュールは使われず、OVSのユーザ空間のデータパスが使われます（ブリッジ作成時にdatapath_type=netdevが指定される）。

EncapsulationはデフォルトではVXLANが使われますが、GeneveやGRE、STTを使うこともできます。またこれらのEncapsulationとIPsecを組み合わせて使うこともできます。将来的には、Encapsulationをしないモードもサポートが予定されています。

VMwareはNSX-TのためのCNIプラグインとしてNCPを提供しています。それなのになぜ別のCNIプログインを開発しているのか疑問に思われる方もいるでしょう。NCPはすでに多くのプロダクション環境で使われており、実績面ではAntreaよりもはるかに成熟しています。また、NCPはiptablesに依存しないServiceのロードバランシングを既に実現しているので、スケーラビリティ的にも優れています。一方、Antreaはまだ出来立てのホヤホヤで商用の実績はありません。ただ、Antreaは100%オープンソースで手軽に始めることができるのは魅力です。幅広いプラットフォームやOSで利用できるようになる潜在的能力に魅力を感じる人も多いでしょう。Antreaが機能面、堅牢面においてすぐにNCPのレベルに達するとは考えにくいですが、コミュニティのサポートが得られればかなりのスピードで進化していく可能性を秘めていると思います。現時点でAntreaをプロダクション環境で使うことはお勧めしませんが、将来的には何かしらの形でNSX/NCPとの連携のようなこともできるようになるかもしれません。その頃にはAntreaも十分に成熟しているはずですので、ユーザにとって環境やユースケースによって使い分けられる選択肢が広がることは喜ばしいことでしょう。


### アーキテクチャ


Antrea はKubernetesに特化したネットワークプラグインです。例えばCalicoはKubernetesだけでなくOpenStackなど他のオーケストレーションシステムをサポートしていますが、AntreaはKubernetesのみを対象としています。そのぶんAntreaはKubernetesの持つ仕組み（API、Tool、Operator）を最大限に利用するようになっています。

アーキテクチャーは以下通りです。

![](http://blog.shin.do/wp-content/uploads/2020/01/arch.svg-300x277.png)](http://blog.shin.do/wp-content/uploads/2020/01/arch.svg.png)

Antreaはいくつかのコンポーネントによって構成されます。Antrea Conrollerはいわゆるコントローラで、Deploymentとして作成さられます。Kubernetes APIを使って関連するオブジェクトの追加や削除を監視しています。一方、Antrea AgentはDaemonSetとして作られ、各ノードで動作します。Antrea agentはOpenFlowの設定をOVS に対して行います。antrea-cniはKubeletからのCNIコマンドをAntrea Agentに引き渡します。antctlコマンドはAntrea Controllerに対するCLIインターフェースを提供するもので、現在絶賛開発中です。

現時点ではkube-proxyはKubernetes標準のものを使用しています（したがって、Serviceのロードバランシングにはiptablesを使います）が、将来的にはこれはOVSベースのロードバランシングを使ったkube-proxyに置き換えられる予定です。Network Policyは既に完全にOVSベースで、iptablesは使われていません。なお、Antreaの最初のリリース（v0.1.0）では、OVSのOpenFlowテーブルを設定するのにovs-ofctlコマンドを使ってしましたが、現在はlibOpenflowライブラリを使うように変更されています。

OctantはKubernetes Clusterの状況を可視化してくれるダッシュボードです。AntreaはOctant用のプラグインを用意しており、Octantから簡単にAntreaの状況を確認することができます。


### インストールと動作確認


今回はオンプレの環境でAntreaを動かしてみます。Kubernetes環境はESXiの上にCluster APIで2つのWorker Nodesを作った素のKubernetes環境を用意することにします。ESXi向けCluster API（CAPV; Cluster API Provider for vSphere) はCentOS 7、Ubuntu 18.04、Photon3用のOVAイメージを提供していますが、今回はUbuntu 18.04を使うことにします。AntreaはKubernetes 1.16以降を要求しますので、v1.16.3のOVAイメージを使用することにします。

標準的な方法でCluster APIでKubernetes環境を作ります。ノードが上がってきたら、各ノードでOVSのカーネルモジュールを入れます。Cluster APIで作られるノードは再構築される可能性があるため、本来ならばCluster APIで使うOVAテンプレートに予めOVSを組み込んでおくべきですが、今回はテストなのでそのステップは省略することにします。各ノードで 以下のコマンド

    
    sudo apt update ; sudo apt -y install openvswitch-switch


を実行して、OVSをインストールします。次にカーネルモジュールが正しく読み込まれたか確認しましょう。

    
    $ lsmod | grep openvswitch
    openvswitch           131072  0
    nsh                    16384  1 openvswitch
    nf_nat_ipv6            16384  1 openvswitch
    nf_defrag_ipv6         20480  3 nf_conntrack_ipv6,openvswitch,ip_vs
    nf_nat_ipv4            16384  2 openvswitch,iptable_nat
    nf_nat                 32768  5 nf_nat_masquerade_ipv4,nf_nat_ipv6,nf_nat_ipv4,xt_nat,openvswitch
    nf_conntrack          131072  12 xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_ipv6,nf_conntrack_ipv4,nf_nat,nf_nat_ipv6,ipt_MASQUERADE,nf_nat_ipv4,xt_nat,openvswitch,nf_conntrack_netlink,ip_vs
    libcrc32c              16384  4 nf_conntrack,nf_nat,openvswitch,ip_vs


この時点ではKubernetesのノードは以下のような状態になっているはずです。

    
    bash-3.2$ kubectl get nodes
    NAME                                       STATUS     ROLES    AGE   VERSION
    workload-cluster-1-controlplane-0          NotReady   master   30m   v1.16.3
    workload-cluster-1-md-0-78469c8cf9-5r9bv   NotReady   <none>   21m   v1.16.3
    workload-cluster-1-md-0-78469c8cf9-tlkn9   NotReady   <none>   21m   v1.16.3


STATUSが "Not Ready" になっていますが、これはCNIプラグインがまだ何もインストールされていないためで、期待された動きです。

Antreaのインストールは非常に簡単です。以下のコマンドを実行するだけで必要なコンポーネントが全てインストールされます。

    
    kubectl apply -f https://raw.githubusercontent.com/vmware-tanzu/antrea/master/build/yamls/antrea.yml


ここではAntreaの最新のものを使うようにしていますが、Antreaの開発のペースは早いので、一時的に最新版ではうまく動作しないケースもあるかもしれません。その場合には、

    
    kubectl apply -f https://github.com/vmware-tanzu/antrea/releases/download/<TAG>/antrea.yml


の<TAG>の部分を適当なものに置き換えてください（現時点で最新のリリース版タグは "v0.2.0" です）。

インストールが無事に終了するとkube-system Namespaceに以下のようなPodができて、全てRunningのSTATUSになっているはずです。

    
    bash-3.2$ kubectl get pods -n kube-system
    NAME                                                        READY   STATUS    RESTARTS   AGE
    antrea-agent-dvpn2                                          2/2     Running   0          8m50s
    antrea-agent-qqhpc                                          2/2     Running   0          8m50s
    antrea-agent-tm28v                                          2/2     Running   0          8m50s
    antrea-controller-6946cdc7f8-jnbxt                          1/1     Running   0          8m50s
    coredns-5644d7b6d9-878x2                                    1/1     Running   0          46m
    coredns-5644d7b6d9-pvj2j                                    1/1     Running   0          46m
    etcd-workload-cluster-1-controlplane-0                      1/1     Running   0          46m
    kube-apiserver-workload-cluster-1-controlplane-0            1/1     Running   0          46m
    kube-controller-manager-workload-cluster-1-controlplane-0   1/1     Running   2          46m
    kube-proxy-4km5k                                            1/1     Running   0          38m
    kube-proxy-cgxcp                                            1/1     Running   0          38m
    kube-proxy-h4css                                            1/1     Running   0          46m
    kube-scheduler-workload-cluster-1-controlplane-0            1/1     Running   2          46m
    vsphere-cloud-controller-manager-gfgbp                      1/1     Running   1          45m
    vsphere-csi-controller-0                                    5/5     Running   0          45m
    vsphere-csi-node-6mvpt                                      3/3     Running   0          7m4s
    vsphere-csi-node-gwr67                                      3/3     Running   0          7m4s
    vsphere-csi-node-hb6rk                                      3/3     Running   0          7m4s


DaemonSetも確認してみましょう。

    
    bash-3.2$ kubectl get daemonset -n kube-system
    NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
    antrea-agent                       3         3         3       3            3           beta.kubernetes.io/os=linux       9m1s
    kube-proxy                         3         3         3       3            3           beta.kubernetes.io/os=linux       47m
    vsphere-cloud-controller-manager   1         1         1       1            1           node-role.kubernetes.io/master=   46m
    vsphere-csi-node                   3         3         3       3            3           <none>                            46m


antrea-agentがDeamonSetとして実行されているのを確認することができます。

ConfigMapを確認してみましょう。

    
    bash-3.2$ kubectl get configmap -n kube-system
    NAME                                 DATA   AGE
    antrea-config-tm7bht9mg6             3      10m
    coredns                              1      48m
    extension-apiserver-authentication   6      48m
    kube-proxy                           2      48m
    kubeadm-config                       2      48m
    kubelet-config-1.16                  1      48m
    vsphere-cloud-config                 1      47m
    bash-3.2$ kubectl get configmap antrea-config-tm7bht9mg6 -n kube-system -o yaml
    apiVersion: v1
    data:
      antrea-agent.conf: |
        # Name of the OpenVSwitch bridge antrea-agent will create and use.
        # Make sure it doesn't conflict with your existing OpenVSwitch bridges.
        #ovsBridge: br-int
        # Datapath type to use for the OpenVSwitch bridge created by Antrea. Supported values are:
        # - system
        # - netdev
        # 'system' is the default value and corresponds to the kernel datapath. Use 'netdev' to run
        # OVS in userspace mode. Userspace mode requires the tun device driver to be available.
        #ovsDatapathType: system
        # Name of the interface antrea-agent will create and use for host <--> pod communication.
        # Make sure it doesn't conflict with your existing interfaces.
        #hostGateway: gw0
        # Encapsulation mode for communication between Pods across Nodes, supported values:
        # - vxlan (default)
        # - geneve
        # - gre
        # - stt
        #tunnelType: vxlan
        # Default MTU to use for the host gateway interface and the network interface of each Pod. If
        # omitted, antrea-agent will default this value to 1450 to accomodate for tunnel encapsulate
        # overhead.
        #defaultMTU: 1450
        # Whether or not to enable IPSec encryption of tunnel traffic.
        #enableIPSecTunnel: false
        # CIDR Range for services in cluster. It's required to support egress network policy, should
        # be set to the same value as the one specified by --service-cluster-ip-range for kube-apiserver.
        #serviceCIDR: 10.96.0.0/12
      antrea-cni.conf: |
        {
            "cniVersion":"0.3.0",
            "name": "antrea",
            "type": "antrea",
            "ipam": {
                "type": "host-local"
            }
        }
      antrea-controller.conf: ""
    kind: ConfigMap
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          {"apiVersion":"v1","data":{"antrea-agent.conf":..<snipped>..}}
      creationTimestamp: "2020-01-06T06:57:51Z"
      labels:
        app: antrea
      name: antrea-config-tm7bht9mg6
      namespace: kube-system
      resourceVersion: "4819"
      selfLink: /api/v1/namespaces/kube-system/configmaps/antrea-config-tm7bht9mg6
      uid: e6115864-4179-471d-a382-a1bb4afdeca1


これをみて分かるように、ConfigMapでは、



 	
  * Integration Bridge名（default: br-int）

 	
  * DataPath タイプ（default: system）

 	
  * ホストと通信するインターフェース名（default: gw0)

 	
  * トンネル（Encapsulation）タイプ（default: vxlan）

 	
  * MTU値（default: 1450）


などを設定することができます。

それでは何かDeploymentを作ってみましょう。今回はお手軽にnginxのDeploymentをreplicas=2で作って、port-forwardでPodにアクセスしてみましょう。

    
    bash-3.2$ cat nginx.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      selector:
        matchLabels:
          app: nginx
      replicas: 2
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.7.9
            ports:
            - containerPort: 80
    bash-3.2$ kubectl apply -f nginx.yaml
    deployment.apps/nginx-deployment created
    bash-3.2$ kubectl get pods
    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-54f57cf6bf-fx4mz   1/1     Running   0          44s
    nginx-deployment-54f57cf6bf-mg74x   1/1     Running   0          44s
    bash-3.2$ kubectl port-forward nginx-deployment-54f57cf6bf-fx4mz 8080:80
    Forwarding from 127.0.0.1:8080 -> 80
    Forwarding from [::1]:8080 -> 80
    


![](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-20.41.34-300x141.png)](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-20.41.34.png)

無事にnginxにアクセスするすることができました。

次にantrea-agentの内部がどう動いているのか見てみましょう。

    
    bash-3.2$ kubectl exec -ti antrea-agent-dvpn2 -n kube-system /bin/bash
    Defaulting container name to antrea-agent.
    Use 'kubectl describe pod/antrea-agent-dvpn2 -n kube-system' to see all of the containers in this pod.
    root@workload-cluster-1-md-0-78469c8cf9-tlkn9:/# ovs-vsctl show
    2fa33701-4b83-4016-a070-08fc90921b2d
        Bridge br-int
            Port "coredns--f4e323"
                Interface "coredns--f4e323"
            Port "tun0"
                Interface "tun0"
                    type: vxlan
                    options: {key=flow, remote_ip=flow}
            Port "nginx-de-70ebff"
                Interface "nginx-de-70ebff"
            Port "vsphere--7be010"
                Interface "vsphere--7be010"
            Port "gw0"
                Interface "gw0"
                    type: internal
        ovs_version: "2.11.1"


Pod用のインターフェースが作成されて、Integrationブリッジ（br-int）に接続されている様子が分かると思います。

最後にOctantとの連携を見てみましょう。

AntreaにはOctant用のプラグインが用意されていて、Octant側からAntreaの状況を確認することができるようになっています。AntreaのOctantプラグインはPodとして動かす方法とプロセスとして動かす方法と2つあります。今回はPodとして動かしてみます。

まず、必要なSecretを作ります。

    
    kubectl create secret generic octant-kubeconfig --from-file=/Users/shindom/work/cluster-api/out/workload-cluster-1/kubeconfig -n kube-system


次にbuild/yamls/antrea-octant.ymlを使用したkubeconfigに合わせてファイル名を書き換えます。

    
    # Change admin.conf to the name of kubeconfig file in your set up.
    - name: KUBECONFIG
      value: "/kube/kubeconfig"
    


<del>Antrea用のOctantプラグインはAntreaのレポジトリにソースがあるので自分でビルドすることもできますが、私のDockerhubにビルド済みのものを置いておきましたので、こちらをお使いいただくのが簡単です。build/yamls/antrea-octant.ymlを以下のように書き換えて、</del>

    
    <del>containers:
     - name: antrea-octant
       image: mshindo/octant-antrea-ubuntu:latest
    </del>


（アップデート2020/10/25アップデート: 　AntreaのDockerhub Repositoryに適切なバージョンのAntrea用Octantプラグインが置かれるようになりましたので、上記の私のイメージは不要になりました。したがって、私のRepositoryからイメージも削除しいています）。

Podとしてデプロイします。

    
    kubectl apply -f build/yamls/antrea-octant.yml


上記のyamlファイルはNodePortを作るようになっていますので、そこに接続するとOctantのUIにアクセスすることができます。

![](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-18.45.50-300x191.png)](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-18.45.50.png)![](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-18.59.47-300x191.png)](http://blog.shin.do/wp-content/uploads/2020/01/Screen-Shot-2020-01-06-at-18.59.47.png)


### 今後の予定


Windowsの対応、No Encapsulationモードの対応、OVSベースのService Load Balancing、RSPAN/IPFIXなどのDay 2 Operation機能への対応などは既に述べた通りですが、他にも



 	
  * IPv6でのPod Network対応

 	
  * DPDKやAF_XDPなどの高速データパス技術への対応


などがロードマップに上がっています。

本当はAntreaが採用しているデータパスのパイプラインについても説明したかったのですが、力尽きたので今回はここまでで（笑）。

Photo by [Manuel Sardo](https://unsplash.com/@manuelsardo?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/sea-fabric?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
