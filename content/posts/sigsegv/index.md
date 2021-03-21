---
title: "なぜ Segumentation Fault は SIGSEG'V' と表現するの?"
date: 2020-06-28T17:57:58+09:00
draft: false
---

こんにちは。久々のブログ投稿です。たまたま Cloudflare のブログ記事を読んでいたら、確かに言われてみればどうして? と思ったタイトルの記事「[Why is there a "V" in SIGSEGV Segmentation Fault?](https://blog.cloudflare.com/why-is-there-a-v-in-sigsegv-segmentation-fault/)」という記事がありました。なるほどなぁと思ったのでかるく翻訳したものを紹介します。(ガバ翻訳なので間違っていたりする場合は、原文をお読みください)

ちなみに Segmentation Fault はよくC言語やC++言語などでコードを書いているとよくでてくるやつで、コンソールではこんな感じででてきますよね?

```
$ clang skynet.c -o skynet
$ ./skynet.out 
Segmentation fault (core dumped)
```

よくある例だと、`確保したメモリ領域以上にアクセスしたときに発生します。プログラムはSIGSEGVシグナルを受信して、「Segmentation fault」と出力して死んでしまいます。さて、「Segmentation fault」はなぜSIGSEGVと表すのでしょうか? Linux開発者が間違えたのでしょうか? それとも読み方を間違えていて、本当は「Segmentation Valut」なのでしょうか。

## Segmentation Violetion の略だった

初めに考えられる可能性として、「Segmentation Violation」 を表していたものが、そのまま使われているのでないか? というものです。

遥か昔、コンピュータにおけるメモリ管理は、[セグメント方式](https://ja.wikipedia.org/wiki/%E3%82%BB%E3%82%B0%E3%83%A1%E3%83%B3%E3%83%88%E6%96%B9%E5%BC%8F)が使われていました。
この仕組みでは各メモリセグメントが、事前に定めたサイズの領域を持っておりそれを Segmentation Limit と呼んでいました。
この制限を超えてデータにアクセスしようとすると、プロセッサ違反が引き起こされます。このエラーコードはページング方式を利用した新しいシステムにも引き継がれました。つまり今でもこの違反がトリガーされると、SIGSEGVシグナルとしてユーザー空間に報告され、コンソール上に表示されている、という仕組みです。

## SIGSEG が時を経て SIGSEGV になった

別の可能性としては、古い第6版のUNIXドキュメントにおける「シグナル」の章に答えがありました。これは1978年頃のものです。

![sigsegv](https://uc7669a5cbc565fb7395ec125674.previews.dropboxusercontent.com/p/thumb/AA2MR7mFktTA6-lUL-MLwgb6iphrO0WRzwBVoFpxnBZCeVNmUmB5pEeFoq6aBT-d8nyaWFWjm8dfnXL7U9FR56bpYw7_5BhOTlbe4p96RmbjbQ-vkRNTeAhPTDBtTxFkb4s6V_e-c4oj8GRLt9YrmPRZPweDESFC_oqUU6NS1YUnh9DX6tY5Wop1HGEjZ5P8yVeAoJSKwdoR-gIR9HDAtPWQg-MhMmVBn28NB0MgrdqpV7WvGh9d_MniEpSirxQAAeMYKlateYTwK6lriJFnAyIbcg0DBZD4y_zqyaMkwSIFMJujcHNP5QV48xEZDsWQLLk/p.jpeg)

よくみてみると、SIGSEGVはなく、シグナルナンバー11 (Segmentation Violation を表している) は、SIGSEG になっています。

UNIXツリーのユーザー空間部分 (/usr/include/signal.h など) は、かなり早い段階でSIGSEGVに切り替えられたようです。しかしカーネル空間では長い間、SIGSEGと言う名前が利用され続けました。

PDP11 [^a] のトラップベクターには、「Segmentation Violation」と言う単語が使われていました。これは、UNIXヒストリーリポジトリの Research V4エディションにありますが、V4で導入されたと表しているわけではありません。というのも、V4が現存する最初のバージョン [^b] なのです。このトラップは、[SIGSEGという名前に trap.c 内で書き換えられて](https://github.com/dspinellis/unix-history-repo/blob/23055b9b5eb219e7870b225475b823762ffee47f/sys/ken/trap.c#L73)いました。

Research V7 のツリーでは、 /usr/include/signal.h に [SIGSEGV という名前](https://github.com/dspinellis/unix-history-repo/blob/caf79156722645e699fd38cead6f24378ef51ef5/usr/include/signal.h) があります。しかしカーネルでは、[SIGSEG という名前](https://github.com/dspinellis/unix-history-repo/blob/c0648e43a5fc3f7fdfa4a4c69acccf6a17aecbe6/usr/sys/sys/trap.c#L177)が利用されています。[カーネルでは、BSD-4のときにSIGSEGVに改名](https://github.com/dspinellis/unix-history-repo/blob/b41c1385b99723d468877994cc776ed594d41f7d/usr/src/sys/sys/trap.c#L67)されました。

つまり、元々はこのシグナルは、SIGSEGと呼ばれていましたが、時を経つにつれ、SIGSEGVに改名され、1980年代以降はSIGSEGVがユーザー空間でもカーネル空間でも利用されてきた、というわけです。

 ![OS timelines](https://www2.dmst.aueb.gr/dds/pubs/jrnl/2016-EMPSE-unix-history/html/timeline.png)
(UNIXのリリース変遷)

# まとめ

おそらくどちらも、Cloudflareのエンジニアたちの推測にすぎないのですが、どちらも説得力のある理由だな、と思います。
とくにUNIXのコードツリーはヒストリリポジトリとしてGitHubに残されていたのですね。それは知らなかったです。

[^a]: PDP11 とは何ぞや? と思い調べたところ、1970年代にあった16ビットの[ミニコンピューター](https://ja.wikipedia.org/wiki/PDP-11)だそうです。
[^b]: [このリポジトリの作成者の論文](https://www2.dmst.aueb.gr/dds/pubs/jrnl/2016-EMPSE-unix-history/html/unix-history.html) によれば、Research-V1 から存在しており、GitHubにも公開されているのでもしかするとさらに古いコードにその変遷が隠されているのかもしれません。筆者はリポジトリをクローンしましたが、1.3GBと大きすぎて調べるのを諦めました。
