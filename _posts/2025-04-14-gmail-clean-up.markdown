---
date: 2025-04-14 23:40:00+09:00
layout: post
title: Gmailの整理
description: 要らないメールをまとめて処分する方法
image: '/images/stephen-phillips-hostreviews-co-uk-3Mhgvrk4tjM-unsplash.jpg'
tags:
- コンピュータ＆ネットワーク
keywords:
- Gmail
- Apps Script
language:
- 日本語
---

私の今の個人ドメイン（shin.do）のメールアドレスでGmailを始めたのは2011年1月からです。14年ほど使ってきて、メールボックスには20万通のメールが溜まり、与えられた20GBのストレージ容量のうち70%以上を消費していました（Google Photoは使っていないので、ほぼメールのみで70%使用している状況）。14年でこんな感じですので、このペースならまだしばらく使えそうでしたが「思い立ったが吉日！」ということで、Gmailのメールボックスを整理することにしました。

小さなメールをちまちま消しても埒が空かないので、まずはサイズの大きいメールを探しました。Gmailでは、検索ウィンドウで "larger:10mb" などとすれば10MB以上のメールを見つけることができます。ただ、今回は個人メールアドレスなので、サイズの大きなメールはそう多くはありませんでした。

次の整理のターゲットは広告・通知系メールです。長年使っているとこれらのメールが大量に溜まっているはずです。しかし、Gmailには送信メール件数の多いメールアドレスを教えてくれるような機能はありません。というわけで、Google Apps Scriptを使ってGoogle Sheetに送信者アドレス毎のメール数を集計することにしました。

コーディングはChatGTP先生にお手伝いいただきました（笑）　最初のコードはメールの最初100件しかみてくれなかったり、全メールを対象にするとtimeoutしてしまったり（Google Apps Scriptは実行に30分以上かかるものはtimeoutになる）、累計処理がうまくいかなかったり、で、何回かChatGTP先生とやりとりしながら、やっと期待通りに動くようになったのが以下のコードです。

{% highlight js %}
function countEmailsBySenderWithResume() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

  const PROGRESS_CELL = "A2"; // 処理済みスレッド数を保存
  const STATUS_CELL = "A1";   // ステータスメッセージ
  const SENDER_CELL = "A4";   // 送信者
  const COUNT_CELL = "B4";    // メール数累計
  const START_ROW = 5;        // データ書き出し開始行
  const BATCH_SIZE = 100;     // 1回で処理するスレッド数

  // 初期化（初回だけ）
  if (sheet.getRange(STATUS_CELL).getValue() === "") {
    sheet.clear();
    sheet.getRange(STATUS_CELL).setValue("実行中");
    sheet.getRange(PROGRESS_CELL).setValue(0);
    sheet.getRange(SENDER_CELL).setValue("送信者");
    sheet.getRange(COUNT_CELL).setValue("件数");
  }

  let start = Number(sheet.getRange(PROGRESS_CELL).getValue());
  const threads = GmailApp.getInboxThreads(start, BATCH_SIZE);
  if (threads.length === 0) {
    sheet.getRange(STATUS_CELL).setValue("完了！");
    return;
  }

  // 今回分のメールを集計
  const senderCounts = {};

  for (let i = 0; i < threads.length; i++) {
    const messages = threads[i].getMessages();
    for (let j = 0; j < messages.length; j++) {
      const sender = messages[j].getFrom();
      senderCounts[sender] = (senderCounts[sender] || 0) + 1;
    }
  }

  // 既存の集計を取得してマージ
  const totalCounts = {};
  const lastRow = sheet.getLastRow();

  if (lastRow >= START_ROW) {
    const dataRange = sheet.getRange(START_ROW, 1, lastRow - START_ROW + 1, 2);
    const data = dataRange.getValues();
    for (let i = 0; i < data.length; i++) {
      const sender = data[i][0];
      const count = Number(data[i][1]);
      if (sender) totalCounts[sender] = count;
    }
  }

  // 今回の集計を加算
  for (let sender in senderCounts) {
    totalCounts[sender] = (totalCounts[sender] || 0) + senderCounts[sender];
  }

  // ソートして書き出し
  const sorted = Object.entries(totalCounts).sort((a, b) => b[1] - a[1]);
  sheet.getRange(START_ROW, 1, sorted.length, 2).setValues(sorted);

  // 進捗更新
  sheet.getRange(PROGRESS_CELL).setValue(start + BATCH_SIZE);
  sheet.getRange(STATUS_CELL).setValue(`処理中：${start + BATCH_SIZE} スレッド`);
}
{% endhighlight %}

これを実行するとBATCH_SIZEで指定されたメール数を処理して終了します。もう一度実行すると次のバッチを処理してくれるので、これを繰り返し実行することで、timeoutを避けつつ全メールを対象にすることができます。このスクリプトを手で何回も実行するのは現実的ではないので、時間主導型のトリガを使って定期的に起動するようにします。

![]({{site.baseurl}}/images/time-based-triggering.png)

ここでは10分おきに実行するように指定しています。1分間隔にすればもっと早く動作させることができますが、Google Apps Scriptは1日に起動できるサービスの回数に上限がありますので、あまり間隔を短くし過ぎるとエラーになってしまいます。というわけで、私はのんびりと10分間隔で動かしました。

トリガされるとApps Scriptを動かしているシートが以下のように徐々に結果が更新されていくはずです。

![]({{site.baseurl}}/images/result-sheet-mosaic.png)

私の場合は全メールが処理されるのに何日かかかりました。このスクリプトは完全にクラウド側で動きますので、一旦トリガの設定をすれば、あとはクライアントPC側はネットワークから切り離されても問題なく動き続けてくれます。なので、何日かかってもさして大きな問題はないと思います。全メールが処理されたら、シートの左上に「完了！」と表示されるはずです。ただ、トリガが自動的に解除されるようなロジックは入っていないので、自分のメールボックスにあるメール数が処理されたらトリガを削除しましょう。

ここまでくれば、あとはメール送信件数が多い送信者からのメールを選択して消していくだけです。Gmailでは検索ウィンドウで、"from:foo@bar.com" などとすれば、foo@bar.comから送られたメールを抽出できます。このように抽出されたメールを全選択して、一括削除をすればOKです。この作業もApps Scriptで自動化することができると思いますが、メールの削除を伴うので、スクリプトの開発・デバッグ中に消してはないけないメールを謝って消してしまう事故が心配です。というわけで、今回は削除作業は目で確認しながら手動でやることにしました。削除まではせずとも、こうして得られたメール送信元リストは、不要な広告メールを送ってくるところを効率的にunsubscribeするのにもきっと役立つはずです。

今回の一連の作業で私のGmailのメール件数は約20万通から7万通ほどに削減され、使用率も70%から40%にまで低下しました。この作業を5年に一度くらいやっていけば、おそらく一生Gmailを使い続けていくことができるのではないかと思います（Gooleがサービスを辞めたりしなければ、ですが^^;）。皆さんも、たまにGmailのお掃除をしてみてはいかがでしょう？　スッキリしますよ！

Photo by <a href="https://unsplash.com/@hostreviews?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Stephen Phillips - Hostreviews.co.uk</a> on <a href="https://unsplash.com/photos/black-laptop-computer-3Mhgvrk4tjM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
