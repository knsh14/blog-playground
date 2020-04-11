---
title: "Go Codereview Comments"
date: 2020-03-29T18:23:14+09:00
---

[原文]( https://github.com/golang/go/wiki/CodeReviewComments )

# 目次
- [go fmt](#go-fmt)
- [Contexts](#contexts)
- [Copying](#copying)
- [Crypt Rand](#crypt-rand)
- [Declaring Empty Slices](#declaring-empty-slices)
- [Doc Comments](#doc-comments)
- [Don't Panic](#dont-panic)
- [Error Strings](#error-strings)
- [Examples](#examples)
- [Goroutine Lifetimes](#goroutine-lifetimes)
- [Handle Errors](#handle-errors)
- [Import Blank](#import-blank)
- [Import Dot](#import-dot)
- [In-Band Errors](#in-band-errors)
- [Indent Error Flow](#indent-error-flow)
- [Initialisms](#initialisms)
- [Interfaces](#interfaces)
- [Line Length](#line-length)
- [Mixed Caps](#mixed-caps)
- [Named Result Parameters](#named-result-parameters)
- [Naked Returns](#naked-returns)
- [Package Comments](#package-comments)
- [Package Names](#package-names)
- [Pass Values](#pass-values)
- [Receiver Names](#receiver-names)
- [Receiver Type](#receiver-type)
- [Synchronous Functions](#synchronous-functions)
- [Useful Test Failures](#useful-test-failures)
- [Variable Names](#variable-names)

# go fmt
あなたのコードに ```gofmt``` を走らせると、自動的に機械的に直すことのできるスタイルの大部分を修正してくれます。
世にあるGoのコードのほとんどすべてが ```gofmt``` を使っています。
この文章の残りは機械的に直すことのできないポイントについて解説します。

代わりに ```goimports``` を使う手段もあります。
```gofmt``` に加えて必要に応じてimport内に空行をつけたり消したりする機能があります。

# Comment Sentences
http://golang.org/doc/effective_go.html#commentary を読みましょう。
```func``` や ```struct``` などのドキュメントのためのコメントは多少冗長であっても完全な文章でなくてはいけません。
このアプローチは ```godoc``` ドキュメントにするときにより効果を発揮します。
コメントは以下の例のように対象の名前で始まって、ピリオドで終わらなければいけません。

```Go
// Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

また、コメントが正しい文章の終わりの印であるピリオドで終わらない場合に注意してください。
[github.com/mailru/easyjson](https://github.com/mailru/easyjson) や [github.com/golang/lint]( https://github.com/golang/lint ) など多くのツールで型やメソッドの情報をマークするためにコメントを使っています。

# Contexts
`context.Context` 型はセキュリティの為の証明情報、デバッグトレースのための情報、タイムアウト、そして API やプロセス間のキャンセル処理のためのシグナルを持っています。
Go のプログラムは受け取った RPC や HTTP リクエストから、返すリクエストへ関数呼び出しの流れに沿って明示的に Context を渡します。

大体の関数では Context は最初のパラメータとして渡します。

```go
func F(ctx context.Context, /* other arguments */) {}
```

`context.Background()` を使ってリクエスト固有に関する処理をせず、いらないと思っていてもコンテキストを渡しておきましょう。
Contextを渡す普通の方法は `context.Background()` を直接呼び出す方法だけです。

Contextを構造体のメンバーに含めては行けません。替わりにすべてのメソッドに引数として渡してください。
唯一の例外はライブラリのシグネチャと一致させなければならない時のみです。

カスタムのコンテキスト型を作成したり、関数の引数に取ったコンテキスト以外のインターフェースを使用しないでください。

もしアプリケーションのデータを引き回したい場合には、パラメータやレシーバ、もしくはグローバル変数にしましょう。
それでも実現できず必要な場合にのみContextに詰めましょう。

コンテキストはイミュータブルなので、複数の呼び出しで同じキャンセル処理やトレース情報を使いまわす場合同じ Context を使うと良いでしょう。

# Copying
予期しない参照を避けるために、別のパッケージから構造体をコピーするときは注意深くやりましょう。
例えば、 `bytes.Buffer` 型は `[]byte` と、短い文字列の最適化のためにスライスが参照するための小さいバイト配列を持っています。
あなたが `Buffer` をコピーすると、中のスライスはオリジナルのものを参照してしまい、その後のメソッド呼び出して思わぬ副作用が起きることがあります。

大抵の場合、もしメソッドがポインタの値に関連付けられているのなら `T` ではなく、`*T` を使うべきです。

# Crypt Rand
`math/rand` をたとえ使い捨てのものであっても鍵の生成のために使ってはいけません。
シードを設定されていない場合、ジェネレータが完全に予測可能です。 `time.Nanoseconds()` でシードが与えられていた場合でも、2〜3bitのエントロピーしかありません。
替わりに `crypto/rand` の `Reader` を使いましょう。それを文字としてほしい場合は、16進数か base64 エンコードしましょう。

```go
import (
    "crypto/rand"
    // "encoding/base64"
    // "encoding/hex"
    "fmt"
)

func Key() string {
    buf := make([]byte, 16)
    _, err := rand.Read(buf)
    if err != nil {
        panic(err)  // out of randomness, should never happen
    }
    return fmt.Sprintf("%x", buf)
    // or hex.EncodeToString(buf)
    // or base64.StdEncoding.EncodeToString(buf)
}
```

# Declaring Empty Slices
スライスの宣言の時は

```go
t := []string{}
```

よりも

```go
var t []string
```

を使うようにしましょう。

前者は長さ0のスライスを生成しますが、後者は nil のスライスを宣言します。

JSON オブジェクトをエンコードする際など、限られた状況で nil ではなく長さ0のスライスのほうが好まれる状況があります。
`nil` スライスは `null` に変換されますが、`[]string{}` は `[]` に変換されます。

インターフェースを設計するときは、nil のスライスと長さ0のスライスを区別しないようにしましょう。なぜなら分かりづらいミスを引き起こすことがあるからです。

より詳しい議論は Francesc Campoy の [Understanding Nil という発表](https://www.youtube.com/watch?v=ynoY2xz-F8s)を参照してください

# Doc Comments
全ての最上位にあったり外部に公開されているものは、docコメントがなければいけません。
外部に公開されていない重要な関数や ```type``` 宣言にも同様です。
http://golang.org/doc/effective_go.html#commentary を読むと、よりコメントの規則についての情報を得ることができます。

# Don't Panic
 http://golang.org/doc/effective_go.html#errors を読みましょう。
 通常のエラーハンドリングで ```panic``` を使うのをやめましょう。
 なるべく ```error``` 型を含んだ複数の値を返すようにしましょう。

# Error Strings
エラー文字列は頭文字を大文字にしたり、句読点で終わってはいけません。
なぜならこの文字列は他のコンテキストで使われるケースがあるからです。
つまり、```log.Print("Reading %s: %v", filename, err)``` が途中で大文字を出力することがないように ```fmt.Errorf("Something bad")``` ではなく、 ```fmt.Errorf("something bad")``` を使いましょう。
これはロギングのような暗黙的な行指向や、他のメッセージと組み合わせないものには適用されません。

# Examples
新しいパッケージを追加した時、使い方のサンプルを含めましょう。
実行可能な例や、一連の流れを追えるテストなどです。

詳しくはこちらを読んでみるといいと思います。https://blog.golang.org/examples

# Goroutine Lifetimes
goroutine を生成する時、いつ終了されるか明確にしましょう。
goroutine は channel の送受信をブロックによってメモリリークを起こす場合があります。
ガベージコレクタはブロックされている channel に到達できなくても、goroutine を停止させません。

もしリークしていなかった場合でも、必要にならなくなった物を残しておくのは、調査しにくい問題を起こすことになりかねません。
クローズした channel に何かを送ると panic を起こして終了してくれます。
まだ使われている入力を「結果が必要でなくなったあと」に変更すると、データ競合を引き起こす場合があります。
goroutine を無駄に長く残しておくと、予想しないメモリの使い方をされる恐れがあります。

並行処理はgoroutineの生存期間が明確になるように、十分にシンプルに書きましょう。
それができない場合は、いつどんな理由で goroutine が終了するかドキュメントに書きましょう。

# Handle Errors
http://golang.org/doc/effective_go.html#errors を読みましょう。
 ```_``` でエラーを無視してはいけません。
もし関数がエラーを返してきたら、関数がちゃんと成功したか確認するためにチェックしてください。
エラーをハンドリングして返すか、本当に例外的な場合のみ ```panic``` しましょう。

# Imports
名前の衝突を回避するために package をリネームするのはやめましょう。
いい package はリネームする必要がないものです。
もし衝突した場合はローカルやプロジェクト固有の package 名を変更することをおすすめします。

import を空行で幾つかのグループに分けましょう。
標準パッケージは最初にグループにしましょう。

```go
package main

import (
    "fmt"
    "hash/adler32"
    "os"

    "appengine/foo"
    "appengine/user"

    "github.com/foo/bar"
    "rsc.io/goversion/version"
)
```

```goimports``` がこれは自動で行ってくれます。

# Import Blank
`import _ "pkg"` の書き方を使ってパッケージをインポートした際の副作用だけを利用する方法があります。
この方法はプログラムのメインパッケージもしくはテストでのみ利用しましょう。

# Import Dot
. を使ったインポートは循環参照によってパッケージの一部がテストできない時などに役立ちます。

```go
package foo_test

import (
    "bar/testutil" // also imports "foo"
    . "foo"
)
```

このケースでは、テストファイルはfooをインポートしているbar/testutilを使っていて、循環参照になってしまうのでpackage fooに置くことができません。
なので ```import .``` を使ってpackage fooの一部のように見せかけます。
このケースを除いて、```import .``` をあなたのプログラムで使うべきではありません。
Quuxのような名前が今のパッケージの識別子なのかインポートされたものなのかが非常にわかりにくくなって、可読性が極端に落ちるからです。

# In-Band Errors
C言語や似たようなものの場合、 -1 や null を返すことでエラーや結果を発見できなかったことを表現するのは一般的に行われている方法です。

```go
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check a for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```

Go は複数の値を返すことができるので、よりよい解決方法を使うことができます。
関数はクライアントに戻り値がエラーを表現しているか調べることを求めるのではなく、他の値が正しく取得できたか示す値を追加して返すべきです。
この値は Error 型かもしれませんし、説明不要な場合は bool 値でもいいでしょう。そしてこれは戻り値のなかで最後に位置しているべきです。

```go
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

この方法で呼び出し側に正しくない値が渡るのを防ぐことができます。

```go
Parse(Lookup(key))  // compile-time error
```

そして堅牢で、読みやすいコードを書きやすくなります。

```go
value, ok := Lookup(key)
if !ok  {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

この方法は外向けの func だけではなく、内部で使われているものにも適用できます。

nil, "", 0, や -1 のような値を返すことは、その値が正しいときや、他の値と別に処理する必要が無い場合は良い選択肢です。

strings などいくつかの標準ライブラリでは in-band エラーを返すことがあります。
これによって文字列を操作するためのコードが大幅に削減され、プログラマが真面目に処理する必要が無いようにしています。
一般的には Go のコードはエラーのための値を返すべきです。

# Indent Error Flow
初めにコード、特にエラーハンドリングの分岐を最小になるように努めましょう。
通常通るコードを簡単に追うことができるので、可読性が上がります。

例えば、こうするのではなく、

```go
if err != nil {
    // error handling
} else {
    // normal code
}
```

こう書くべきです。

```go
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

もし初期化のコードでこのようなコードがあるなら

```go
if x, err := f(); err != nil {
    // error handling
    return
} else {
    // use x
}
```

変数宣言の行を個別にしてこうするべきです。

```go
x, err := f()
if err != nil {
    // error handling
    return
}
// use x
```

# Initialisms
変数名にある頭字語は一貫性がなければいけません。
例えば、"URL" は "URL" か "url" でなければならず、"Url" ではいけません。
"ServeHttp" ではなく "ServeHTTP" となります。
複数のイニシャル語から構成される語では、"xmlHTTPRequest" や "XMLHTTPRequest" の様になります。

"identifier" の省略である "ID" にもこのルールは適用され、 "appId" ではなく "appID" となります。

プロトコルバッファのコンパイラによって生成されたコードはこのルールを免除します。
人が書いたコードには、機械が書いたコードよりも厳しい基準を要求します。

# Interfaces
Go のインターフェースは大抵パッケージに含まれており、パッケージに実装されている値を使うのではなく、インターフェース型を使います。(訳微妙)
パッケージを実装する際には(ポインタや構造体のような)具体的な型を返さなければいけません。
この方法だと使う側の修正なしに、新しいメソッドを追加することができます。

モックのためにAPIの実装側にインターフェースを定義してはいけません。替わりに実際に実装された公開APIからテストできるようなAPIを設計しましょう。

使われる前にインターフェースを定義しては行けません。現実的な使い方のサンプルがなければ、そのどんなメソッドを含めるかはもちろん、そのインターフェースが必要かどうかわかりません。

```go
package consumer  // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { ... }
```

```go
package consumer // consumer_test.go

type fakeThinger struct{ ... }
func (t fakeThinger) Thing() bool { ... }
...
if Foo(fakeThinger{...}) == "x" { ... }
```

```go
// DO NOT DO IT!!!
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ ... }
func (t defaultThinger) Thing() bool { ... }

func NewThinger() Thinger { return defaultThinger{ ... } }
```

替わりに具体的な型を返して、consumer側にはproducerの実装をモックしておきましょう。

```go
package producer

type Thinger struct{ ... }
func (t Thinger) Thing() bool { ... }

func NewThinger() Thinger { return Thinger{ ... } }
```

# Line Length
Goでは1行の長さを決めてはいませんが、長過ぎないようにしてください。
同じように繰り返しが多い時など、1行を短く保ちたいがために無理に改行を入れる必要はないでしょう。

人が不自然な位置で改行を挟む時(多かれ少なかれ例外はありますが、関数呼び出しや関数宣言の中頃です)、適切な数の引数と、適切に短い変数名になっていれば、改行は不要なはずです。
長い行は長い名前によって出来上がりますし、長い名前を取り除くことは多くの手助けになります。

言い換えると、行の長さで改行するのではなく、行の意味によって改行するべきです。
もし長過ぎる行を見つけたときは、名前や処理の流れを改善してみるとより良い結果になるはずです。

これは関数がどれだけ長いかについても全く同じです。
「関数はN行以下でなければいけない」というルールはありませんが、長すぎたり短すぎたりする関数はあります。
そういったときに行数を数えるのではなく、この関数はどこで区切れるのかを考えるべきです。

# Mixed Caps
http://golang.org/doc/effective_go.html#mixed-caps を読んでください。
これは他の言語の取り決めとは異なります。
例えば、公開しない定数は `MaxLength` や `MAX_LENGTH` ではなく `maxLength` となります。

同じことが [initialisms]( #initialisms ) でも説明されています。

# Named Result Parameters
このような名前付き戻り値がGodocではどのように見えるか考えてみましょう

```go
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```

なにかぎこちない感じになります。こちらのほうがよいでしょう。

```go
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

一方で関数が同じ型の返り値を複数返したり、結果が文脈からはっきりとわかりにくい場合、名前付き戻り値は有効です。
関数内で戻り値のための変数を宣言するのを省略したいがために名前付き戻り値を使うのはやめましょう。
APIが冗長になる上に、実装の簡潔さも失われます。

この例よりも

```go
func (f *Foo) Location() (float64, float64, error)
```

こちらのほうが、よりわかりやすいでしょう。

```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

ただの `return` は数行の関数なら大丈夫ですが、ある程度の大きさの関数になると、返す変数を明確にしたほうがよいでしょう。
当然ですが、ただの `return` が使えるというだけでは名前付き戻り値を使う理由にはなりません。
明快なドキュメントを書くことはたかが１〜２行を節約するよりも何倍も価値があります。

最後に、幾つかのケースでは遅延クロージャで値を変更するために名前付き戻り値を指定する必要があります。
その場合は問題ありません。

# Naked Returns
[Named Result Parameters]( #named-result-parameters ) と同じ

# Package Comments
Godocで読める全てのコメントと同じようにパッケージのコメントはパッケージ節の前に空行無しで書かなければいけません。

```go
// Package math provides basic constants and mathematical functions.
package math
```

```go
/*
Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
```

main パッケージのコメントは、他のスタイルとして main の替わりにバイナリ名を使っても良いでしょう。(文頭に来る場合はもちろん大文字になります。)
`seegen` というコマンドの main パッケージのコメントは次のようになります。

```go
// Binary seedgen ...
package main
```
```go
// Command seedgen ...
package main
```
```go
// Program seedgen ...
package main
```
```go
// The seedgen command ...
package main
```
```go
// The seedgen program ...
package main
```
```go
// Seedgen ..
package main
```

これらは例であり、状況によって変化しても大丈夫です。

ですがこれらのコメントは公開されるものなので、正しい英語で書かなければ行けません。先頭の文字を小文字で始めたりすることはできません。
バイナリ名が最初の単語に来た場合、コマンドライン実行時と異なったとしても大文字で書きましょう。

http://golang.org/doc/effective_go.html#commentary を読むとより詳細なコメントに関するアドバイスがあります。

# Package Names
パッケージから公開されている全ての識別子への参照はパッケージ名を通して行われるので、識別子にパッケージ名を使っている場合は外すべきです。
例えば、chubbyというパッケージの中ではChubbyFileという名前を作っては、呼び出すときにchubby.ChubbyFileとなってしまうので、避けましょう。
その代わりにFileという名前にして、使う側がchubby.Fileとできるのでよりよいでしょう。
http://golang.org/doc/effective_go.html#package-names にはより多くの情報があります。

# Pass Values
たかだか数バイトを節約するために引数にポインタを指定するのは辞めましょう。
もし関数が全体を通して引数 `x`を `*x` として呼び出しているなら、引数をポインタにするべきではありません。
この一般的なインスタンスは文字列のポインタ(`*string`)や、インターフェースのポインタ(`*io.Reader`)を渡すことも含みます。
どちらのケースでも、変数自身は固定されていて、直接値が渡されます。
このアドバイスは、大きなStructや今後大きくなりそうなものには適用しません。

# Receiver Names
レシーバの名前はそれがなんであるかを適切に表したものでなければいけません。
大抵の場合その型1〜2文字の略称で足ります。 (Clientであれば c や cl のように)
`me` や `this`, `self`のように関数ではなく、メソッドに重点を置いたオブジェクト指向の典型的な名前を使うのは辞めましょう。
レシーバ名はその役割が明らかで、ドキュメントとしての目的もないので、引数名ほど説明的である必要がありません。
そのメソッドのあらゆる行に登場するので、できるだけ短いほうがよいでしょう。慣れてくればとても簡素でよく思えてきます。
レシーバ名は統一してください。あるところで `client` を `c` としたなら、他のところで `cl` としてはいけません。

# Receiver Type
メソッドのレシーバーをポインタにするか否かを選択することは常に難しいものです。特にGoを学び始めた頃には難しいでしょう。
疑わしい場合はポインタを受け取りますが、小さい構造体や、プリミティブな型の場合は、値を受け取るだけのほうが効率的な場合があります。
判断のためのガイドラインを下に示します。

* レシーバが `map`、`func`、チャネル ならポインタを使うべきではありません。レシーバがスライスで、メソッドがスライスを作りなおさない場合は、ポインタを使うべきではありません。
* メソッドが値を変更する必要がある場合、レシーバはポインタでなければいけません。
* レシーバが `sync.Mutex` か、似たような同期するフィールドを持つ構造体なら、レシーバはポインタでなければいけません。
* レシーバが大きな構造体や配列なら、ポインタはとても効果的です。では大きいとはどれくらいなのでしょう？もし構造体の全ての値を引数に渡すと仮定してください。多すぎると感じたなら、それはポインタにしても良いくらいの大きさです。
* 関数が同時に実行されたりメソッドが呼び出された時に、レシーバの値を変更するでしょうか？値渡しではメソッドが実行されるときにレシーバのコピーを生成します。なのでメソッドの外ではレシーバへの変更が適用されません。変更がオリジナルのレシーバに適用される必要があるなら、レシーバはポインタです。
* レシーバが、値を変更されるかも知れない構造体、配列、スライス、その他の要素であった場合、レシーバをポインタにしたほうが、読み手にとってよりわかりやすいでしょう。
* レシーバが小さく、本来値型であったり(例えば `time.Time` のようなもの) 、変更するフィールドやポインタがない構造体や配列、あるいは `int` や `string` のようなシンプルな型の場合は、レシーバが値であるほうが良い場合があります。値のみのレシーバは生成されるゴミの量を減らすことができます。もし値がメソッドに渡されると、ヒープ領域にメモリを確保する代わりに、スタックメモリのコピーが走ります。(コンパイラはこのヒープメモリ確保を避けようとしますが、常には上手く行きません) プロファイラで確認する前にこの理由で値レシーバにするのは辞めましょう。
* これらの理由に当てはまらず、まだ迷っている場合はレシーバをポインタにしましょう。

# Synchronous Functions
非同期処理よりも同期処理(結果を直接返すか、値を返す前にコールバックやchannelの操作が終わっている関数) を好みます。

同期処理はgoroutineは呼び出したメソッドの中で閉じていて、それらの生存期間や、リーク、データ競合を推測するのが簡単になります。
そしてわざわざ同期をとったり待たなくても入力を渡して出力を確認することができて、テストが簡単になります。

もし呼び出し側がより同時並行性を求めるなら、呼び出し側で別の goroutine から呼び出してあげれば簡単に追加できます。
ですが、不必要な並行処理を呼び出し側から減らすのは非常に難しいですし、ときに不可能です。

# Useful Test Failures
テストが失敗した時には、以下の事柄を伝える有効なメッセージを出力しましょう。
* なにが悪かったのか
* どんな入力があったか
* 実際にどんな値が来たのか
* どんな値が来ることを期待していたのか

`AssertFoo` ヘルパーの束をを書くことは魅力的ですが、あなたのヘルパーはもっと役に立つメッセージを出力できるはずです。
あなたの失敗したテストをデバッグする人はあなたや、あなたのチームではないことを前提にしています。
Goの典型的なテスト失敗時のコードはこのようなものです。

```go
if got != tt.want {
    t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
}
```

`actual != order` の順で並んでいて、メッセージでも同じ順番で出力していることに注目してください。
幾つかのテストフレームワークでは `0 != x, "expected 0, got x"` のように、この順を逆にして書くことを推奨していますが、Goではそうではありません。

もしこのやり方で大量にタイプしなければいけないようなら、Table-Driven-Testが欲しくなるかもしれません。

もう1つのテスト失敗時の曖昧さをなくすためのテクニックとして、入力ごとに `TestFoo` をラップした異なるテスト関数を作る小方法があります。この場合関数の名前と共に失敗します。

```go
func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
func TestNoValues(t *testing.T)    { testHelper(t, []int{}) }
```

いずれにせよ、将来あなたのコードをデバッグする人に有効なメッセージを届ける責任はあなたにあります。

# Variable Names
Goでの変数名はより短くあるべきです。
特に限定されたスコープで使われるローカル変数は短くあるべきです。
`lineCount` よりも `c` ですし、 `lineIndex` よりも `i` です。

基本的なルール
* 宣言された箇所よりも離れた場所でその変数が使われるなら、より説明的な名前にするべきです。
* メソッドレシーバは1〜2文字で十分です
* ループの添字や読み込みのリーダーのような一般的な変数は(`i` や `r` のように)1文字でよいでしょう。
* より珍しい物や、パッケージ外で使われるものの名前はより説明的にしましょう。

