---
date: 2006-12-19 16:47:32+00:00
layout: post
title: Wireshark NetFlow V9 dissector
categories:
- コンピュータとインターネット
language:
- 日本語
---

現在のWiresharkのNetFlow V9 dissectorはちょっと手を抜いていて、Templateの管理をする際にSource IDやPDUのsource IP addressを無視するようになっています。したがって、異なったSource IDや別のExporterから同じTemplate IDでTemplateが送られてくると、それらを使ったDate FlowSetを誤って解釈してしまう場合があります。多くのExporterの実装では、Template IDを256から順に振っていくので、このようなconfilictは結構な頻度で発生します。

この問題は随分前から気づいていたのですが、ずっと手一杯で直せませんでした。が、ようやく重い腰を上げてやっつけました！

以下のようにwiresharkのMLに投げておいたので、（特に問題なければ）次のリリース（0.99.5）にこの修正が入って出てくると思います。

    
    Subject: Re: [Wireshark-users] cflow v9 dissector oddity
    From: Motonori Shindo <mshindo@mshindo.net>
    To: wireshark-users@wireshark.org, yb@bashibuzuk.net
    Date: Wed, 20 Dec 2006 01:23:14 +0900 (JST)
    Sender: wireshark-users-bounces@wireshark.org
    Reply-To: Community support list for Wireshark <wireshark-users@wireshark.org>
    X-Mailer: Mew version 5.0.50 on Emacs 21.4 / Mule 5.0 (SAKAKI)
    
    Hi,
    
    I have addressed this issue. Please find attached the patch against
    the current svn repository.
    
    As per NetFlow V9 protocol, Template ID is guaranteed to be unique per
    Observation Domain (identified by Source ID) and the Exporter
    (identified by the source IP address of NetFlow PDU).
    
    The former code was ignoring these information for simplicity, but
    noticing such a necessity.
    
    Regards,
    ---
    Motonori Shindo
    Fivefront Corporation
    http://www.fivefront.com
    
    From: Yann Berthier <yb@bashibuzuk.net>
    Subject: Re: [Wireshark-users] cflow v9 dissector oddity
    Date: Sun, 3 Dec 2006 19:49:02 -0500
    >
    >    Hello,
    >
    >
    >    Thanks for your feedback,
    >
    > On Thu, 30 Nov 2006, at 17:57, Stephen Fisher wrote:
    >
    > > On Sun, Nov 26, 2006 at 11:10:05PM -0500, Yann Berthier wrote:
    > >
    > > >    On a capture of netflow v9 traffic from 2 routers, where r1 exports
    > > >    data flowsets using template id 257 and template flowsets of said id
    > > >    of 21 fields, and r2 exports a template flowset for id == 257 of 23
    > > >    fields, wireshark (0.99.4) mixes-up the templates when decoding the
    > > >    flowsets from r1 - it uses the last template cached, be it from r1
    > > >    or r2, to decode the data flowsets from r1
    > >
    > > This sounds like a problem with the dissector.  Could you file a bug
    > > at http://bugzilla.wireshark.org/ and attach a capture file that you
    > > see the problem with?
    >
    >
    >    Sure for the former, the latter may be harder, i would have preferred
    >    to provide it privately. If not, i'd need to check what's in the
    >    capture obviously
    >
    >    thanks,
    >
    >       - yann
    > _______________________________________________
    > Wireshark-users mailing list
    > Wireshark-users@wireshark.org
    > http://www.wireshark.org/mailman/listinfo/wireshark-users
