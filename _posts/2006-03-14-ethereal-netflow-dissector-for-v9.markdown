---
date: 2006-03-14 17:12:15+00:00
layout: post
title: Ethereal NetFlow dissector for V9
categories:
- コンピュータとインターネット
language:
- 日本語
---

現在のEtherealのNetFlow  dissectorは、V9 Template Flowsetには1つしかTemplate Recordが無い、と仮定してしまっています。でも、この仮定は誤りです。いまのままのdissectorだと、1つのTemplate Flowsetに2つ以上のTemplate Recordがあるとうまく解釈できません。

困っている人がいたので、パッチを作って本家MLに投げておきました。そのうちマージされると思います。
