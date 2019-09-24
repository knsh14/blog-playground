---
title: "技術書典7に参加してきました"
date: 2019-09-24T19:36:24+09:00
draft: false
---


[技術書典7]( https://techbookfest.org/event/tbf07 )に参加してきました。
自分は今回は"[かまてん]( https://techbookfest.org/event/tbf07/circle/5741270278340608 )"というサークルでGoのテストについての本を頒布していました。

他にも
* [golang.tokyo]( https://techbookfest.org/event/tbf07/circle/5174941137764352 )
* [メルカリ技術書典部]( https://techbookfest.org/event/tbf07/circle/5642184086781952 )

に記事を寄稿しました。

# 寄稿した記事の概要
## かまてん
かまてんは私かまたと[@tenntenn]( https://twitter.com/tenntenn )の二人のサークルです。  
今回はGoのテストについて使い方やテストのためのテクニック、テストしやすいコードを書くためにはどこを意識して書いたらよいのかを解説しています。

ベースにしたのはGopherCon 2017でMitchell Hashimotoが発表した"Advanced Testing with Go"です。
彼の発表資料や動画をベースにそれぞれの項目を噛み砕いてサンプルをつけました。
発表資料へのリンクは次の3つです。

* https://speakerdeck.com/mitchellh/advanced-testing-with-go
* https://www.youtube.com/watch?v=8hQG7QlcLBk
* https://about.sourcegraph.com/go/advanced-testing-in-go

二人なので常にギリギリの戦いをしながら本を書いています。
今回も当日まで頑張って色々修正してなんとか頒布することができました。

後日Boothにて当日頒布したものからもう少し追記したものを販売する予定です。
当日買いそびれた方はそちらで購入していただけます。

技術書典6で頒布した[静的解析についての本]( https://knsh14.booth.pm/items/1319336 )もBoothに置いています。

## golang.tokyo
Goのコミュニティであるgolang.tokyoでは有志のメンバーが集って、それぞれが好きなトピックについて書きました。
自分はGopherCon 2019で観てすごいなーと感じたGioというGoのGUIライブラリについて、サンプルを使いながら紹介しました。

Gioについての資料は次の4つがあります。

* https://git.sr.ht/~eliasnaur/gio
* https://godoc.org/gioui.org
* https://www.youtube.com/watch?v=9D6eWP4peYM
* https://go-talks.appspot.com/github.com/eliasnaur/gophercon-2019-talk/gophercon-2019.slide#1

GioはGitHubを使っていないのでコントリビュートするのも大変なのですが、自分がコントリビュートした時の手順も紹介しています。

golang.tokyoは今回表紙を[tottie]( https://twitter.com/tottie_designer )さんに依頼しました。
tottieさんはGoのスタンプを制作されたり、かわいいGopherの画像をtwitterにアップされてるデザイナーの方です。
今回作成して頂いた表紙がとても可愛かったので自分でもポスター欲しいなと思いました。

## メルカリ技術書典部
私が所属しているメルペイとメルカリのエンジニアで記事を書きました。  
私は先日のMERPAY TECH OPENNESS MONTHの際に投稿した"メルペイにおけるお客さま残高の管理手法"をブラッシュアップして寄稿しました。

私が担当しているマイクロサービスがどのような設計になっているかについて解説しています。
お客様の残高を預かる大事なサービスなので取引の整合性をどのように担保するかについて工夫している仕組みについて解説しています。
内容的にはほぼ変わっていませんが、いくつか文章を直したり画像を入れ替えたりしています。
"ちょっと手直しするだけでいいから楽勝だよ！"と言われたのでほいほいついて行きましたが、結果的に半分くらいは書き直したり、詳しく説明し直した気がします。

そこから更に[@mhidaka]( https://twitter.com/mhidaka )先生が神編集してくださって、よい文章に改造していただきました。


# 反省
私は文章を書くのが苦手で、修正項目の8割ほどは文章の校正に費やされています。
textlintなどを使って機械的にレビューしていますが、細かい接続詞などでまだまだ大量に修正点が見つかります。

参考資料として次のリンクを紹介してもらったので、次に見直せるように紹介します。
* [技術的な文章を書くための第0歩 ～読者に伝わる書き方～]( https://qiita.com/mhidaka/items/c5fe729716c640b50ff7 )
* [技術的な文章を書くための1歩、2歩、3歩]( https://qiita.com/vvakame/items/d657baf26cf83ac98bd0 )

他にも、次の書籍は文章作成の参考になると伺いました。

* 理科系の作文技術
* 技術者のためのテクニカルライティング入門講座

どちらもKindleで読めます。気になった方はぜひ読んでみてください。

# 感想
今回も徹夜で文章を修正して、朝8時に見本誌を印刷するというバタバタっぷりでした。
しかし、当日は多くの方に手にとっていただけたので参加してよかったなと思います。
技術書典8も頑張って記事を書いていきます。

