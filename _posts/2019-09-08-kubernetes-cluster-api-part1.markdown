---
date: 2019-09-08 21:44:30+00:00
layout: post
title: Kubernetes Cluster API - Part 1 -
categories:
- コンピュータとインターネット
language:
- 日本語
tags:
- cluster-api
- Kubernetes
---

Cluster APIはKubernetesスタイルのAPIを使ってKubernetesのクラスタの管理するための仕組みで、Kubernetes SIG (Special Interest Group) のうちの1つであるCluster Lifecycle SIGで仕様の策定が行われています。

Kubernetes環境の構築ツールにはkubeadm、kops、Kubesprayなど多数存在していますが、それぞれスコープが異なります。kubeadmはKubernetesのコンポーネントのインストールをしてくれますが、それを動かすためのプラットフォームの面倒は見てくれません。kubeadmの実行時にはすでにKubernetesを動かすための環境（サポートされているOSが動いているベアメタルや仮想マシン、container runtime、kubeletやkubectlなどのツール、など）が整っていることが前提になっています。一方、kopsはKubernetesを動かすプラットフォーム（Cluster）まで含めて構築をしてくれますが、プラットフォームを個別にサポートしていく必要があります。2019年9月時点でオフィシャルにサポートされているプラットフォームはAWSのみ、GCEとOpenStackがbetaサポート、vSphereとDigitalOceanがalphaサポートとなっています（コードを見るとAliCloudやベアメタルもサポートしようとしているように見えます）。

今後もこのようなKubernetesクラスタ環境設定ツールが他にも出てくる可能性がありますが、今のままだとそれぞれのツールで個別にプラットフォームをサポートしていかなければいけません。また、Kubernetesを動かしたいプラットフォームも、AWSやGCPだけでなく他のパッブリッククラウドやベアメタルなど、多様な環境をサポートしたくなるに違いありません。さらに、クラスタを構築する（Day1 Operation）だけでなく、クラスターを大きくしたい/小さくしたい、クラスタノードのOSのバージョンアップをしたい、といったいわゆるクラスタのDay2 Operationも必要になりますが、これらもプラットフォームごとに考えなければいけないことになります。これはKubernetesのユーザーにとっても環境構築ツールのエコシステムにとっても大きな苦痛です。

このような問題を解決するために、クラスタを含むKubernetes環境全体を管理するための汎用的な仕組みを提供することを目的に生まれたのがCluster APIです（CAPIと呼べばれることもあります）。”API" という名前が付いていますが、どちらかというとフレームワーク的な色合いが強いと思います。

「汎用的」と言っても、当然プラットフォームごとに必要な処理は異なりますので、Cluster APIではプラットフォームごとに個別の "Provider" を用意してプラットフォームの差異を吸収し、どのようなプラットフォームに対しても統一的なのAPIでコントロールできるように作られています。現在さまざまなProviderが用意されており、これらのProviderがサポートしているプラットフォーム上であればCluster APIを使ってKubernetes環境を簡単に構築・管理することができます。2019年9月時点でサポートされているProviderは以下の通りです。



 	
  * AWS

 	
  * Azure

 	
  * Baidu

 	
  * Baremetal

 	
  * DigitalOcean

 	
  * Exoscale

 	
  * GCP

 	
  * IBM Cloud

 	
  * OpenStack

 	
  * Packet

 	
  * Talos

 	
  * Tencent Cloud

 	
  * vSphere


一般的にはKubernetesはコンテナ管理のプラットフォーム（オーケストレータ）として知られていますが、実はKubernetesはコンテナ以外のものも管理することできる枠組みが備わっています。Kubernetes のアーキテクチャの根本をなす部分として、システムをあるべき状態（Desired State）に保つ、という機能があります。Kubernetesでは、さまざまなリソースのあるべき状態を宣言的に定義しリソースの作成を行うと、それらのリソースの状態は常に監視され、あるべき状態と現在の状態（Current State）に差が生じている事を検出すると、あるべき状態に戻そうという動きをします。これはKubernetesのReconciliation Loopと呼ばれるメカニズムによって実現されています。[![Reconciliation Loop]({{site.baseurl}}/images/ReconciliationLoop.png)

例えば、Podが3つ動いているはずなのに実際にはPodが2つしか動いていなければ、Kubernetesは自動的にPodを1つ新たに立ち上げて、常に3つのPodが動いているように調整（修復）をしてくれます。このようないわゆる自己修復（Self Healing）機能が働くのはReconciliation Loopのおかげです。

KubernetesではDeployment、Replicaset、Pod、Node、Secret、StorageClassなど50種類ほどのリソースがあらかじめ定義されていますが、Kubernetesはこれらの定義済みのリソース以外のものを定義する仕組みが備わっており、Custom Resource Definition（CRD）と呼ばれています。CRDを定義しそれに対応するコントローラを書けば、自由にさまざまなリソースをKubernetesで扱うことができるようになります。これらのCRDで定義されたリソースにもReconciliation Loopによる自己修復機能を働かせることができるため、Kubernetesは変化に富む分散システムのライフサイクル管理をする汎用的なフレームワークとしても使うこともできるわけです。

Cluster APIはこのようなKubernetesの特性を活かし、クラスタなどのプラットフォームを規定するリソースをCRDとして定義して、それらをKubernetes自体で管理することで、KubernetesをKubernetesで管理する、というアーキテクチャになっています。

ClusterAPIでは以下の5つのCRDを用意しています。



 	
  * Cluster

 	
  * Machine

 	
  * MachineSet

 	
  * MachineDeployment

 	
  * MachinClass




### Cluster


Clusterリソースはクラスター全体の設定を規定します。例えば、ServiceやPodで使用するCIDRブロックの指定やドメイン名を指定します。またproviderSpecアトリビュートでクラスタ全体に関わるProvider固有の設定も行えます。

    
    apiVersion: "cluster.k8s.io/v1alpha1"
    kind: Cluster
    metadata:
      name: workload-cluster-1
    spec:
      clusterNetwork:
        services:
          cidrBlocks: ["100.64.0.0/13"]
        pods:
          cidrBlocks: ["100.96.0.0/11"]
        serviceDomain: "cluster.local"
      providerSpec:
        ...
    
    




### Machine


Machineリソースは、Kubernetesを動作させるノードの設定を規定します。例えばkubeletやKubernetesのコントロールプレーンのバージョンなどを指定します。また、providerSpecアトリビュートではノードが使用するメモリ量やCPU数、インスタンスのタイプなどを指定します。

    
    apiVersion: cluster.k8s.io/v1alpha1
    kind: Machine
    metadata:
      name: "workload-cluster-1-controlplane-1"
      labels:
        cluster.k8s.io/cluster-name: "workload-cluster-1"
    spec:
      providerSpec:
        ...
    versions:
        kubelet: "1.13.6"
        controlPlane: "1.13.6"




### MachineSet


MachineSetリソースではMachineリソースがいくつ動いているべきかをreplicasアトリビュートで指定します。

    
    apiVersion: cluster.k8s.io/v1alpha1
    kind: MachineSet
    metadata:
      name: "workload-cluster-1-machineset-1"
      labels:
        machineset-name: "workload-cluster-1-machineset-1"
        cluster.k8s.io/cluster-name: "workload-cluster-1"
    spec:
      replicas: 3
      selector:
        matchLabels:
          machineset-name: "workload-cluster-1-machineset-1"
          cluster.k8s.io/cluster-name: "workload-cluster-1"
      template:
        metadata:
          ...
        spec:
          ...




### MachineDeployment


MachineDeploymentリソースはMachine、MachineSetのアップデートポリシーを規定します。現在サポートされているアップデートポリシーはRollingUpdateのみです。

    
    apiVersion: "cluster.k8s.io/v1alpha1"
    kind: MachineDeployment
    metadata:
      name: "workload-cluster-1-machineset-1"
      labels:
        cluster.k8s.io/cluster-name: "workload-cluster-1"
    spec:
      replicas: 3
      selector:
        matchLabels:
          cluster.k8s.io/cluster-name: "workload-cluster-1"
          set: node
      template:
        metadata:
          ...
        spec:
          ...




### MachineClass


MachineClassリソースは、どのProviderSpecを使用するのかを規定します。Machine、MachineSet、MachineDeploymentリソースは、どれもproviderSpecアトリビュートでプロバイダに固有な情報を設定することができますが、プロバイダ情報は共通であることが多いので、何箇所も同じ設定をするのは面倒です。そこでMachineClassを使ってプロバイダ固有の情報を記述し、他のリソースからそれを参照するようにすることで、よりシンプルにMachine関係のリソースを記述することができるようになります。

    
    apiVersion: "cluster.k8s.io/v1alpha1"
    kind: MachineClass
    metadata:
      name: "workload-cluster-1-machine"
    providerSpec:
      value:
        apiVersion: ...
        kind: ...


お気付きの方も多いかと思いますが、Cluster APIのCRDは、標準でサポートされているリソースと類似性があります。

Machine -- Pod
MachineSet -- ReplicaSet
MachineDeployment -- Deployment
MachineClass -- StorageClas

このような類似性を踏まえると、Kubernetesをすでに使っている人にとってはClusterAPIは理解しやすいのになるのではないかと思います。

次回はCluster APIとvSphere用のProviderを使ってvSphere上にKubernetesのクラスタを立てる解説し、、、ようと思ったのですが、ちょうど@masanaraさんがQiitaにこのトピックについて記事をあげて下さいましたので、皆さん[こちら](https://qiita.com/masanara/items/47aec0e1d2ab1fc4f664)をご参照ください。

Photo by [apoorv mittal](https://unsplash.com/@aprvm?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
