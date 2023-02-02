---
date: 2006-02-20 15:39:24+00:00
layout: post
title: Googlebotの謎
categories:
- コンピュータとインターネット
language:
- 日本語
---

Googlebot君が尋ねに来てくれるのを心待ちにしているんですが、なかなか来てくれません。一日一回くらいは来てくれるんんですが、なぜかトップページしかCrawlしてくれなくて、特にキャッシュもされません。

    
    66.249.64.7 - - [20/Feb/2006:13:30:43 +0900] "GET / HTTP/1.0" 200 9272 "-" "Googlebot/2.1 (+http://www.google.com/bot.html)"


一方、

    
    66.249.65.141 - - [19/Feb/2006:11:34:22 +0900] "GET /products/genie/atm6000/index.html HTTP/1.1" 200 8950 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.co
    m/bot.html)"


というMozillaを語るGooglebot君はかなり頻繁に来てくれて、Deep CrawlっぽくあちこちGETしてくれるんですけど、やっぱりキャッシュはされません。真偽のほどは確かではありませんが、このMozillaを語るGooglebot君は、次世代のGooglebotのテスト版である、という噂があるようです。だとすると、Crawlしにきてもキャッシュはされないのもうなづけます。

あー、やはくGooglebot君がDeep Crawlしに来てくれないかなー。
