---
date: 2009-03-04 17:29:15+00:00
layout: post
title: REXML::Document#write
categories:
- コンピュータとインターネット
language:
- 日本語
---

Ruby REXMLを使ったコードを書いていて、ライブラリの不具合を踏んだ。

    
    $ ruby api.rb
    /usr/lib/ruby/1.8/rexml/document.rb:189:in `write': uninitialized constant REXML::Formatters::Transitive (NameError)
    from api.rb:29
    from api.rb:17:in `each'
    from api.rb:17
    $ ruby -v
    ruby 1.8.6 (2008-08-11 patchlevel 287) [i386-linux]


どうやら、

[http://redmine.ruby-lang.org/issues/show/553](http://redmine.ruby-lang.org/issues/show/553)

みたい。

1.9.1では直っていそうだが、これだけのために1.9.1に上げるのもなんなので、手動で1.8.6添付のライブラリにパッチを当てた。おかげでNameErrorは出なくなった。めでたし。
