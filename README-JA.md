# DatabricksによるScalaガイド

我々の知る限りではApache Sparkは1000を超えるコントリビューターがいて、Big Dataにおけるオープンソースのプロジェクトとしては最大規模であり、Scalaで書かれたプロジェクトの中で最もアクティブなものです。このガイドはSparkに貢献してきたエンジニアや、我々の [Databricks](http://databricks.com/) のエンジニアチームとの経験やコーチング、また、共に作業をしてきた中で描かれたものです。

コードは、その作者によって __一度書かれます__ が、多数のエンジニアによって __読まれたり修正されることは何度もあります__ 。ほとんどのバグは未来の修正によって引き起こされるので、長期間、グローバルな可読性や保守性を保つためにはコードの基盤を最適化する必要があります。これを実現する為にはシンプルなコードを書くのが一番です。

Scalaは様々なパラダイムに順応できるとてつもなく強力な言語です。我々は以下に示すガイドラインに従うことで素早くプロジェクトを進めていけることに気づきました。ただし、チームのニーズによっては違ったやり方が必要かもしれません。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## <a name='TOC'>Table of Contents</a>

  0. [改版履歴](#history)
  1. [構文のスタイル(Syntactic Style)](#syntactic)
    - [命名規則](#naming)
    - [変数の命名規則(Variable Naming Convention)](#variable-naming)
    - [行の長さ](#linelength)
    - [30の原則(Rule of 30)](#rule_of_30)
    - [スペースとインデント](#indent)
    - [空行(Blank Lines (Vertical Whitespace))](#blanklines)
    - [括弧(Parentheses)](#parentheses)
    - [中括弧(Curly Braces)](#curly)
    - [Longリテラル](#long_literal)
    - [ドキュメントのスタイル](#doc)
    - [クラス内の要素の順番](#ordering_class)
    - [import文](#imports)
    - [パターンマッチ](#pattern-matching)
    - [インフィックスメソッド(Infix Methods)](#infix)
    - [匿名メソッド(Anonymous Methods)](#anonymous)
  2. [Scala言語の機能](#lang)
    - [applyメソッド](#apply_method)
    - [override指定子](#override_modifier)
    - [構造化代入 (Destructuring Binds)](#destruct_bind)
    - [名前渡し(call-by-name)](#call_by_name)
    - [複数の引数リスト(Multiple Parameter Lists)](#multi-param-list)
    - [記号を用いたメソッド (演算子オーバーロード)](#symbolic_methods)
    - [型推論](#type_inference)
    - [return](#return)
    - [再帰処理(Recursion)と末尾再帰(Tail Recursion)](#recursion)
    - [implicit](#implicits)
    - [例外処理 (Try vs try)](#exception)
    - [Option](#option)
    - [モナドによるメソッドチェーン(Monadic Chaining)](#chaining)
  3. [同時並行性(Concurrency)](#concurrency)
    - [Scala concurrent.Map](#concurrency-scala-collection)
    - [明示的な同期(Synchronization) vs 並列コレクション(Concurrent Collections)](#concurrency-sync-vs-map)
    - [明示的な同期(Synchronization) vs アトミック変数 vs @volatile](#concurrency-sync-vs-atomic)
    - [privateフィールド(Private Fields)](#concurrency-private-this)
    - [分離性(Isolation)](#concurrency-isolation)
  4. [パフォーマンス](#perf)
    - [マイクロベンチマーク](#perf-microbenchmarks)
    - [走査(traversal)とzipWithIndex](#perf-whileloops)
    - [Optionとnull](#perf-option)
    - [Scalaコレクションライブラリ](#perf-collection)
    - [private[this]](#perf-private)
  5. [Javaとの互換性](#java)
    - [JavaにはあってScalaには無い機能](#java-missing-features)
    - [トレイト(trait)と抽象(abstract)クラス](#java-traits)
    - [型エイリアス(Type Aliases)](#java-type-alias)
    - [デフォルト引数(Default Parameter Values)](#java-default-param-values)
    - [複数の引数リスト(Multiple Parameter List)](#java-multi-param-list)
    - [可変長引数(vararg)](#java-varargs)
    - [implicit](#java-implicits)
    - [コンパニオンオブジェクト、staticメソッド、フィールド](#java-companion-object)
  6. [その他](#misc)
    - [currentTimeMillisよりもnanoTime推奨](#misc_currentTimeMillis_vs_nanoTime)
    - [URLよりもURI推奨](#misc_uri_url)



## <a name='history'>改版履歴</a>
- 2015-03-16: 初版作成。
- 2015-05-25: [override指定子](#override_modifier)追加。
- 2015-08-23: いくつかのルールを「してはいけない」から「避けましょう」に表現を変更
- 2015-11-17: [applyメソッド](#apply_method)更新: コンパニオンオブジェクトのapplyメソッドはコンパニオンクラスを返すべき。
- 2015-11-17: このガイドは[中国語に翻訳](README-ZH.md)されました。コミュニティのメンバーである [Hawstein](https://github.com/Hawstein) によって翻訳されました。翻訳版が最新の状態を常に保つ保証はされていません。
- 2015-12-14: このガイドは[韓国語に翻訳](README-KO.md)されました。翻訳は [Hyukjin Kwon](https://github.com/HyukjinKwon) によって行われ、 [Yun Park](https://github.com/yunpark93), [Kevin (Sangwoo) Kim](https://github.com/swkimme), [Hyunje Jo](https://github.com/RetrieverJo) and [Woochel Choi](https://github.com/socialpercon) によってレビューされました。翻訳版が最新の状態を常に保つ保証はされていません。
- 2016-06-15: [匿名メソッド](#anonymous) を追加しました.

## <a name='syntactic'>構文のスタイル(Syntactic Style)</a>

### <a name='naming'>命名規則</a>

命名規則については概ねJavaとScalaの標準的なものに則っています。

- class, trait, object はJavaの命名規則に従います。 (i.e. パスカルケース(PascalCase))
  ```scala
  class ClusterManager

  trait Expression
  ```

- package はJavaの命名規則に従います。 (i.e. ASCII文字で全て小文字)
  ```scala
  package com.databricks.resourcemanager
  ```

- methods/function は キャメルケース(camelCase) にします。

- 定数(constant)は全て大文字にしてコンパニオンオブジェクトに入れます。
  ```scala
  object Configuration {
    val DEFAULT_PORT = 10000
  }
  ```

- enum はパスカルケース(PascalCase)にします。

- アノテーションもJavaの命名規則に従います。（i.e. パスカルケース(PascalCase)。Scalaの公式ガイドとは異なるので注意してください）
  ```scala
  final class MyAnnotation extends StaticAnnotation
  ```


### <a name='variable-naming'>変数の命名規則(Variable Naming Convention)</a>

- 変数はキャメルケース(camelCase)で、わかりやすい名前をつけます。
  ```scala
  val serverPort = 1000
  val clientPort = 2000
  ```

- 小さなスコープの中であれば1文字で変数を表現しても問題ありません。例えば "i" は単純な(e.g. 10行程度の)ループの添字としてよく使われます。ただし、 "l" (Larryのl) を識別子に使ってはいけません。 "l", "1", "|", "I" を見分けるのが難しいからです。

### <a name='linelength'>行の長さ</a>

- 1行につき100文字まで
- 例外としてimport文やURLは許容してもかまいません。（その場合でも100文字以内に収める努力はしてください）

### <a name='rule_of_30'>30の原則(Rule of 30)</a>

『一つの要素が30以上のサブ要素で構成されている場合、そこには大きな問題が潜んでいる可能性が高いです』 - [Refactoring in Large Software Projects](http://www.amazon.com/Refactoring-Large-Software-Projects-Restructurings/dp/0470858923).

一般的に:

- メソッドは30行以内に収めましょう
- クラス内のメソッドは30以内に収めましょう


### <a name='indent'>スペースとインデント</a>

- インデントにはスペースを2文字使います。
  ```scala
  if (true) {
    println("Wow!")
  }
  ```

- メソッドの宣言の際、パラメータが1行に収まらない場合はスペースを4文字使ってインデントします。戻り値の型はパラメータと同じ行でも良いですし、改行してスペース2文字でインデントしても良いです。
  ```scala
  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration): RDD[(K, V)] = {
    // メソッドの中身
  }

  def newAPIHadoopFile[K, V, F <: NewInputFormat[K, V]](
      path: String,
      fClass: Class[F],
      kClass: Class[K],
      vClass: Class[V],
      conf: Configuration = hadoopConfiguration)
    : RDD[(K, V)] = {
    // メソッドの中身
  }
  ```

- classのヘッダ定義が1行に収まらない場合はextend以降を改行してスペース2文字でインデントして、classヘッダの後に空行を1行加えます。
  ```scala
  class Foo(
      val param1: String,  // パラメータはスペース4文字インデント
      val param2: String,
      val param3: Array[Byte])
    extends FooInterface  // ここはスペース2文字インデントとなります
    with Logging {

    def firstMethod(): Unit = { ... }  // この上は空行にします
  }
  ```

- 縦方向にコードの要素を揃えるのは、余計な懸念事項を増やすのでやめましょう。
  ```scala
  // このように縦方向は揃えないでください
  val plus     = "+"
  val minus    = "-"
  val multiply = "*"

  // 以下のように自然に記載してください
  val plus = "+"
  val minus = "-"
  val multiply = "*"
  ```


### <a name='blanklines'>空行(Blank Lines (Vertical Whitespace))</a>

- 下記の場合空行を1行挿入します:
  - クラス内の連続したメンバー（または初期化子）間: クラス、フィールド、コンストラクタ、メソッド、ネストしたクラス、static初期化子、インスタンス初期化子
    - 例外: フィールド間（間に他のコードがないもの）の空行は任意です。論理的にグループ分けをしたい場合に空行を挿入しましょう。
  - メソッド内で論理的なグループ分けをしたい場合
  - クラス内の最初のメンバーの前、あるいは最後のメンバーの後（必須ではなく任意）
- classの定義間には1〜2行の空行を挿入します。
- 極端に空行を使いすぎるのはやめましょう。


### <a name='parentheses'>括弧(Parentheses)</a>

- メソッドは括弧つきで定義します。副作用の無いアクセサメソッドの場合に限り括弧無しでも可。（ある状態を変えることや、I/Oが関連する動作は副作用とみなします）
  ```scala
  class Job {
    // 悪い例: killJobは状態を変化させるので()つきで定義すべき
    def killJob: Unit

    // 良い例:
    def killJob(): Unit
  }
  ```
- 呼び出し側が括弧をつけるかどうかはメソッドの定義に合わせます。(メソッドが括弧付きで定義されていたら括弧付きで呼び出します)
  これは構文上の話だけでなく、 `apply` においては処理そのものも変わる可能性があるので注意が必要です。
  ```scala
  class Foo {
    def apply(args: String*): Int
  }

  class Bar {
    def foo: Foo
  }

  new Bar().foo  // これはFooを返します
  new Bar().foo()  // これはIntを返してしまいます!
  ```


### <a name='curly'>中括弧(Curly Braces)</a>

1行で終わるような条件付き処理やループであっても中括弧で囲みます。ただひとつの例外として、if/elseで3項演算(ternary operator)かつ副作用を伴わない処理を行う場合のみ中括弧を省略できます。
```scala
// 良い例:
if (true) {
  println("Wow!")
}

// 良い例:
if (true) statement1 else statement2

// 良い例:
try {
  foo()
} catch {
  ...
}

// 悪い例:
if (true)
  println("Wow!")

// 悪い例:
try foo() catch {
  ...
}
```


### <a name='long_literal'>Longリテラル</a>

long型リテラルには大文字の `L` を用います。 `l` と `1` の区別がつきにくいからです。
```scala
val longValue = 5432L  // 良い例

val longValue = 5432l  // 悪い例
```


### <a name='doc'>ドキュメントのスタイル</a>

ドキュメントにはJavaDocのスタイルを採用します。（ScalaDocのスタイルは使わない）
```scala
/** これは1行で短い説明を書く場合の正しい書き方です */

/**
 * これは複数行に跨る場合のJavaDocの正しい書き方で、
 * 2行目に入るときはこのようになりますし、さらに書き続けて
 * 3行目はこのように書きます。
 */

/** Sparkに於いてはScalaDocのスタイルは使いません。なので
  * この書き方は誤っています。
  */
```


### <a name='ordering_class'>クラス内の要素の順番</a>

クラスが肥大化してきた場合は、論理的なグループ分けをして、コメントヘッダを用いて整理します。
```scala
class DataFrame {

  ///////////////////////////////////////////////////////////////////////////
  // DataFrame operations
  ///////////////////////////////////////////////////////////////////////////

  ...

  ///////////////////////////////////////////////////////////////////////////
  // RDD operations
  ///////////////////////////////////////////////////////////////////////////

  ...
}
```

もちろん、これだけクラスが大きくなってしまうのは問題であり、特定のpublic APIを作る場合だけに限定すべきです。


### <a name='imports'>import文</a>

- __ワイルドカードを用いたimportを使うのは控えます__, ただし、7個以上になる場合や、implicitなメソッドをimportする場合は許容します。ワイルドカードを用いたimportは外部の改修に対して弱くなります。
- パッケージをimportする場合は必ず絶対パスで行います (e.g. `scala.util.Random` は正しく `util.Random` は誤りです)
- さらに、importは以下の順番で記載します:
  * `java.*` and `javax.*`
  * `scala.*`
  * サードパーティ製のライブラリ (`org.*`, `com.*`, etc)
  * プロジェクト内のクラス (`com.databricks.*` またはSparkを使っているなら `org.apache.spark` )
- それぞれのグループ内ではアルファベット順に並べます
- IntelliJの import organizer を使えば以下の設定で自動的に制御可能です:

  ```
  java
  javax
  _______ blank line _______
  scala
  _______ blank line _______
  all other imports
  _______ blank line _______
  com.databricks
  ```


### <a name='pattern-matching'>パターンマッチ</a>

- パターンマッチのみが実装されたメソッドの場合、可能であれば `match` をメソッドの宣言と同じ行に記載し、無駄なインデントを1段減らします。
  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

- closure（または部分関数）を含んだ関数を呼び出す場合でcaseが一つしか無い場合は、caseを関数の呼び出しと同じ行に記載します。
  ```scala
  list.zipWithIndex.map { case (elem, i) =>
    // ...
  }
  ```
  caseが複数ある場合はインデントを使って中括弧で囲みます
  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```


### <a name='infix'>インフィックスメソッド(Infix Methods)</a>

記号で構成されていないメソッドは __インフィックス表記法(infix notation)を避けます__ (i.e. 演算子オーバーロード)。
```scala
// いい例
list.map(func)
string.contains("foo")

// 悪い例
list map (func)
string contains "foo"

// ただし、演算子オーバーロードされているものはインフィックススタイルを使います
arrayBuffer += elem
```


### <a name='anonymous'>匿名メソッド(Anonymous Methods)</a>

匿名メソッドでは __無駄な中括弧を避けます__。
```scala
// 良い例
list.map { item =>
  ...
}

// 良い例
list.map(item => ...)

// 悪い例
list.map(item => {
  ...
})

// 悪い例
list.map { item => {
  ...
}}

// 悪い例
list.map({ item => ... })
```


## <a name='lang'>Scala言語の機能</a>


### <a name='apply_method'>applyメソッド</a>

クラスにapplyメソッドを定義するのは避けます。Scalaに詳しくない人にとって読みにくいコードになりますし、IDE(またはgrep)にとってもコードを追いにくくさせます。最悪の場合[括弧(Parentheses)](#parentheses)で示したように、処理自体が変わってしまうこともあります。

applyメソッドをコンパニオンオブジェクトのファクトリメソッドとして定義するのは問題ありません。この場合applyメソッドはコンパニオンクラスの型を返すようにします。
```scala
object TreeNode {
  // これは正しい使い方
  def apply(name: String): TreeNode = ...

  // これはTreeNode型を返していないので誤りです
  def apply(name: String): String = ...
}
```

### <a name='override_modifier'>override指定子</a>
メソッドをオーバーライドする場合やabstractメソッドを実装する場合共に必ずoverride指定子を記載します。Scalaのコンパイラはabstractメソッドを実装する場合に `override` を必須としませんが、必ず `override` を明示的に用いるべきです。 `override` を自明にすることの他に異なるシグネチャによる `override` の空振りを防ぎます。
```scala
trait Parent {
  def hello(data: Map[String, String]): Unit = {
    print(data)
  }
}

class Child extends Parent {
  import scala.collection.Map

  // 以下は親メソッドのParent.helloをオーバーライドしません。
  // なぜなら、パラメータのMapの型が異なるからです。
  // "override"指定子を記載していればコンパイラがこの誤りを指摘することができます
  def hello(data: Map[String, String]): Unit = {
    print("これは親メソッドをオーバーライドするはずでしたが、実際はしません！")
  }
}
```

### <a name='destruct_bind'>構造化代入 (Destructuring Binds)</a>

構造化代入(Destructuring bindまたはtuple extractionとも呼ばれる)は一つの式で二つの変数に値を代入する方法です。
```scala
val (a, b) = (1, 2)
```

ただし、コンストラクタで使ってはいけません。特に `a` や `b` がtransientの場合はScalaのコンパイラが余分なTuple2フィールドを生成してしまいます。
```scala
class MyClass {
  // これは正常に動作しません。(a, b)に対してコンパイラはtransientではないTuple2を
  // 生成してしまいます。
  @transient private val (a, b) = someFuncThatReturnsTuple2()
}
```


### <a name='call_by_name'>名前渡し(call-by-name)</a>

__名前渡し(call-by-name)を使うのは避けます__。 明示的に `() => T` を使います。

理由: Scalaはメソッドのパラメータをby-nameで定義することができます。 e.g. 次の例は動作します:
```scala
def print(value: => Int): Unit = {
  println(value)
  println(value + 1)
}

var a = 0
def inc(): Int = {
  a += 1
  a
}

print(inc())
```
上のコードで `inc()` は `print` にクロージャとして渡されてprintメソッド内で2回実行されます。 `1` が `print` に渡されるわけではありません。名前渡し(call-by-name)の問題点としては、呼び出し側が名前渡し(call-by-namme)と値渡し(call-by-value)の区別をつけることができない点です。よって、式が実行されるかどうか判断がつきません（最悪の場合複数回実行されるかもしれません）。副作用を含む式の場合は特に危険です。


### <a name='multi-param-list'>複数の引数リスト(Multiple Parameter Lists)</a>

__複数の引数リストを使うのは避けます__。演算子オーバーロードを複雑化しますし、Scalaに明るくないプログラマを惑わせます。例えば： 

```scala
// これは避けましょう!
case class Person(name: String, age: Int)(secret: String)
```

例外として低水準ライブラリを作る際にimplicitを用いる場合は許容します。ただし、[implicitも使用を控えるべきです](#implicits)！


### <a name='symbolic_methods'>記号を用いたメソッド (演算子オーバーロード)</a>

__メソッド名に記号は使いません__。ただし算数の演算子(e.g. `+`, `-`, `*`, `/`)を定義する場合はかまいません。それ以外は絶対に使用してはいけません。記号を用いたメソッド名はそのメソッドの用途を読み取るのが非常に困難になります。以下の例を参考にしてください：
```scala
// 記号を用いたメソッド名は意図が理解しにくい
channel ! msg
stream1 >>= stream2

// 記号を用いない方が意図がわかりやすい
channel.send(msg)
stream1.join(stream2)
```


### <a name='type_inference'>型推論</a>

Scalaの型推論、特に式の左側の推論やクロージャの推論はコードを簡潔にします。よって明示的に型を書く場面は限られています：

- __publicメソッドは明示的に型を記載します__。そうしなければコンパイラの推論に度々驚かされることでしょう。
- __implicitメソッドは明示的に型を記載します__。そうしなければインクリメンタルコンパイルの際にScalaのコンパイラがクラッシュする可能性があります。
- __型を人の目で推論し難い変数やクロージャは明示的に型を記載します__。コードレビューする人が3秒で型を推論できないものが一つの目安です。


### <a name='return'>return</a>

__クロージャ内でreturnを使うのは避けます__. `return` はコンパイラによって ``scala.runtime.NonLocalReturnControl`` の ``try/catch`` に変換され、予期しない動作を引き起こす可能性があります。下記の例をみてください：
  ```scala
  def receive(rpc: WebSocketRPC): Option[Response] = {
    tableFut.onComplete { table =>
      if (table.isFailure) {
        return None // これはやってはいけません！
      } else { ... }
    }
  }
  ```
`.onComplete` は匿名クロージャ `{ table => ... }` を受け取って違うスレッドに渡します。このクロージャはいずれその __違うスレッドで__ `NonLocalReturnControl` をスローしてキャッチされます。悲しいことに元々のメソッドには何の影響も起こさないのです。

しかしながら、いくつかの場面で `return` は推奨されています。

- guardによる早期returnを行う場合。これによって余分なインデントを避けることができます。
  ```scala
  def doSomething(obj: Any): Any = {
    if (obj eq null) {
      return null
    }
    // 何かしらの処理 ...
  }
  ```

- ループを早期にreturnする場合。これによって状態フラグでの制御が不要になります。
  ```scala
  while (true) {
    if (cond) {
      return
    }
  }
  ```

### <a name='recursion'>再帰処理(Recursion)と末尾再帰(Tail Recursion)</a>

__再帰処理の使用は避けます__。ただし、再帰処理を行うことが自明(e.g. グラフ走査、木の走査)な場合は許容します。

末尾再帰メソッドであれば `@tailrec` アノテーションをつけてコンパイラにチェックしてもらいましょう。(クロージャや関数変換の影響で実際は末尾再帰ではないことに頻繁に気付かされることでしょう)

ほとんどのコードは単純なループや状態マシンを使う方が簡単です。末尾再帰（あるいはアキュムレータ）を使うと逆に冗長になったり、読み難くなることがあります。下の例であれば、上の再帰処理版よりも下の手続き型版の方が読みやすいコードであることがわかります：

```scala
// 末尾再帰版
def max(data: Array[Int]): Int = {
  @tailrec
  def max0(data: Array[Int], pos: Int, max: Int): Int = {
    if (pos == data.length) {
      max
    } else {
      max0(data, pos + 1, if (data(pos) > max) data(pos) else max)
    }
  }
  max0(data, 0, Int.MinValue)
}

// 手続き型なループ版
def max(data: Array[Int]): Int = {
  var max = Int.MinValue
  for (v <- data) {
    if (v > max) {
      max = v
    }
  }
  max
}
```


### <a name='implicits'>implicit</a>

__implicitを使うのは避けましょう__。ただし、以下の場合は可：
- DSL(ドメイン固有言語)を作っている
- implicit型パラメーターとして使っている(e.g. `ClassTag`, `TypeTag`)
- クラス内だけで冗長な型の変換を行っている(e.g. ScalaクロージャからJavaクロージャへ)

implicitを使う場合は、他者がそのimplicitの定義そのものを読まずとも意味が汲み取れることを保証しなければいけません。implicitはとても複雑なルールをもっていて、コードをとてつもなく追いづらく、理解のし難いものにします。TwitterのEffective Scalaによれば、『implicitを使うときには必ずimplicitを使わずに実装できないか自問してください』とあります。

使わざるを得ない場合(e.g. DSLを作る場合)は、implicitメソッドをオーバーロードしてはいけません。必ず違う名前のメソッドを作って、使い手が個別にメソッドをimportしやすいようにします。
```scala
// これはやってはいけません。使い手が片方のメソッドだけimportすることができません。
object ImplicitHolder {
  def toRdd(seq: Seq[Int]): RDD[Int] = ...
  def toRdd(seq: Seq[Long]): RDD[Long] = ...
}

// 以下のように定義します:
object ImplicitHolder {
  def intSeqToRdd(seq: Seq[Int]): RDD[Int] = ...
  def longSeqToRdd(seq: Seq[Long]): RDD[Long] = ...
}
```


## <a name='exception'>例外処理 (Try vs try)</a>

- ThrowableやExceptionはcatchしてはいけません。 `scala.util.control.NonFatal` を使います:
  ```scala
  try {
    ...
  } catch {
    case NonFatal(e) =>
      // 例外ハンドラ; NonFatalはInterruptedExceptionとマッチしないことに注意してください
    case e: InterruptedException =>
      // InterruptedExceptionのハンドラ
  }
  ```
このように書くことで `NonLocalReturnControl` をcatchしないことを保証します。(詳細については [return](#return-statements) 参照)

- `Try` をAPIでは使いません。言い換えるとTryをreturnしてはいけません。その代わり異常処理の場合は明示的に例外をスローしてJavaのようにtry/catchで例外をハンドルします。

上記のようにした背景：Scalaは( `Try`, `Success`, `Failure` を通して)モナドでエラーをハンドルしてアクションをチェーンしやすくすることができます。しかしながら、我々の経験上この方法をとると逆にネストが増えて読み難くなる傾向があります。さらに、期待されるエラー(expected error)と例外とで何が違うのかということがはっきりしておらず、 `Try` にも述べられていません。よって、 `Try` をエラーハンドルに使うことを推奨しません。具体的には：

  わざとらしい例ですが:
  ```scala
  class UserService {
    /** データベースからユーザの情報を取得 */
    def get(userId: Int): Try[User]
  }
  ```
  上の例よりは下記のような書き方を推奨します
  ```scala
  class UserService {
    /**
     * データベースからユーザの情報を取得
     * @return ユーザ情報が見つからない場合は何も返しません
     * @throws DatabaseConnectionException データベースに接続できない場合/
     */
    @throws(DatabaseConnectionException)
    def get(userId: Int): Option[User]
  }
  ```
  2つ目の方が呼び出し側がどのようにエラーをハンドルすべきか明解です。


### <a name='option'>Option</a>

- 値が空になる可能性がある場合は `Option` を使います。 `null` と比較すると `Option` は明示的に値が `None` になり得ることを示唆します。
- `Option` を作るときには値が `null` である可能性を踏まえて `Some` ではなく `Option` を使います。
  ```scala
  def myMethod1(input: String): Option[String] = Option(transform(input))

  // transformはnullを返す可能性があるのでmyMethod2はSome(null)を返す可能性があります。
  // よって、上の例が推奨されます。
  def myMethod2(input: String): Option[String] = Some(transform(input))
  ```
- Noneを例外の代わりに使ってはいけません。例外は明示的にスローします。
- `Option` の中に必ず値が入っていると確信できる場合を除いて `get` を `Option` に対して使ってはいけません。


### <a name='chaining'>モナドによるメソッドチェーン(Monadic Chaining)</a>

Scala言語の強みの一つとしてモナドによるメソッドチェーンがあります。ほとんど全てのもの(e.g. collection, Option, Future, Try)がモナドであり、それぞれの処理は繋げる（チェーンする）ことができます。これはとてつもなく強力なコンセプトなのですが、メソッドチェーンはほどほどに抑えておくべきです。具体的には：

- 4つ以上の処理をチェーン(あるいはネスト)するのは避けましょう
- ロジックを理解するのに5秒を超えるようであれば、どうしたらメソッドチェーンを使わずに実現できるかよく考えてください。一般的にflatMapやfoldには気をつけましょう。
- 必ずと言っていいほどflatMapのあとは一度メソッドチェーンを区切るべきです。(型が変わるので)

メソッドチェーンは大抵の場合中間結果を変数に格納して、明示的に型情報を記載し、より手続き型なスタイルで書くことでわかりやすくなります。大げさな例で示すと：
```scala
class Person(val data: Map[String, String])
val database = Map[String, Person]
// "address"に"null"を設定することがあります

// モナドによるメソッドチェーンな書き方
def getAddress(name: String): Option[String] = {
  database.get(name).flatMap { elem =>
    elem.data.get("address")
      .flatMap(Option.apply)  // nullを扱う
  }
}

// 行数は増えますが、より可読性の高い書き方
def getAddress(name: String): Option[String] = {
  if (!database.contains(name)) {
    return None
  }

  database(name).data.get("address") match {
    case Some(null) => None  // nullを扱う
    case Some(addr) => Option(addr)
    case None => None
  }
}

```

## <a name='concurrency'>同時並行性(Concurrency)</a>

### <a name='concurrency-scala-collection'>Scala concurrent.Map</a>

__`scala.collection.concurrent.Map` よりも `java.util.concurrent.ConcurrentHashMap` を使うようにしてください__。具体的には `scala.collection.concurrent.Map` の `getOrElseUpdate` はアトミックではありません (ただしScala 2.11.6でfix済み  [SI-7943](https://issues.scala-lang.org/browse/SI-7943))。我々の手がけているプロジェクトは全てScala 2.10とScala2.11向けにクロスビルドするので `scala.collection.concurrent.Map` は避けます。

### <a name='concurrency-sync-vs-map'>明示的な同期(Synchronization) vs 並列コレクション(Concurrent Collections)</a>

状態を安全に並列アクセスするためには以下の3つの方法を推奨します。 __これらを混ぜて使ってはいけません__。最悪の場合デッドロックに繋がる可能性があります。

1. `java.util.concurrent.ConcurrentHashMap`: 状態が全てmap内にキャプチャされていて競合する頻度が高い場合に使います
  ```scala
  private[this] val map = new java.util.concurrent.ConcurrentHashMap[String, String]
  ```

2. `java.util.Collections.synchronizedMap`: 状態が全てmap内にキャプチャされていて、競合が発生しないはずだが安全なコードにしておきたい場合に使います。競合が発生しない場合はJVM JITコンパイラはsynchronizeのオーバーヘッドをBiased Lockingで取り除きます。
  ```scala
  private[this] val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])
  ```

3. synchronizedを用いた明示的な同期：複数の変数をガードしたいときに使います。2と同様、JVM JITコンパイラはBiased Lockingでsynchronizeのオーバーヘッドを取り除きます。

  ```scala
  class Manager {
    private[this] var count = 0
    private[this] val map = new java.util.HashMap[String, String]
    def update(key: String, value: String): Unit = synchronized {
      map.put(key, value)
      count += 1
    }
    def getCount: Int = synchronized { count }
  }
  ```

1.、 2.においてはビューやコレクションのイテレータが同期されている範囲を抜けださないように注意が必要です。これは `Map.keySet` や `Map.values` をreturnする場合などに発生し得ます。ビューや値を受け渡す必要がある場合はコピーを作るようにしてください。
  ```scala
  val map = java.util.Collections.synchronizedMap(new java.util.HashMap[String, String])

  // 悪い例!
  def values: Iterable[String] = map.values

  // 要素をコピーするようにしてください
  def values: Iterable[String] = map.synchronized { Seq(map.values: _*) }
  ```

### <a name='concurrency-sync-vs-atomic'>明示的な同期(Synchronization) vs アドミック変数 vs @volatile</a>

`java.util.concurrent.atomic` パッケージにはプリミティブが用意されていて、プリミティブ型へロックを意識せずにアクセスできるようになります。例として `AtomicBoolean`、 `AtomicInteger`、 `AtomicReference`があります。

いかなる場合も `@volatile` よりアトミック変数を使います。アトミック変数は機能的に対応するプリミティブ型の厳密なスーパーセットであり、コード上も役割がはっきりとしています。また、アトミック変数は内部実装で `@volatile` を使用しています。

次の場合は明示的な同期(synchronization)よりもアトミック変数の使用が推奨されます： (1) あるオブジェクトに対する全てのクリティカルな更新が *ただ一つの* 変数に依存していて、かつ競合が予想されている場合。アトミック変数はロックを気にする必要が無く、競合時の処理もより効率よく行います。 (2) 同期処理(synchronization)が `getAndSet` で置き換え可能な場合。例として：
  ```scala
  // 良い例: 並行処理に関連するコードが一度だけ実行されるのがわかりやすく表現されています
  val initialized = new AtomicBoolean(false)
  ...
  if (!initialized.getAndSet(true)) {
    ...
  }

  // 悪い例: synchronizedによって何が制御されているかわかりづらい。不必要にsynchronizedされる可能性があります。
  val initialized = false
  ...
  var wasInitialized = false
  synchronized {
    wasInitialized = initialized
    initialized = true
  }
  if (!wasInitialized) {
    ...
  }
  ```

### <a name='concurrency-private-this'>privateフィールド(Private Fields)</a>

`private` フィールドは同じクラスの別のインスタンスであればアクセス可能であることに注意しなければいけません。故に `this.synchronized` (あるいは単純に `synchronized`) では競合を防ぐには十分ではありません。代わりに `private[this]` を使いましょう。
```scala
// 以下は十分に安全ではありません
class Foo {
  private var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}

// 以下であれば安全です
class Foo {
  private[this] var count: Int = 0
  def inc(): Unit = synchronized { count += 1 }
}
```


### <a name='concurrency-isolation'>分離性(Isolation)</a>

一般的に並行性(concurrency)や同期(synchronization)に関わるロジックは可能な限り分離されているべきです。言い換えると：

- APIやユーザーが使う可能性のあるメソッド、またはコールバックにおいて同期プリミティブ(synchronization primitive)を表に出すのは避けましょう。
- 複雑なモジュールに於いては、小さなインナーモジュールを作って、その中で並行性プリミティブ(concurrency primitive)を閉じ込めておくようにしましょう。


## <a name='perf'>パフォーマンス</a>

あなたの書くコードのほとんどはパフォーマンスのことをあまり意識しなくても大丈夫なはずです。ただし、パフォーマンスを意識しなければならない場合は以下のTIPSを参考にしてください：

### <a name='perf-microbenchmarks'>マイクロベンチマーク</a>

ScalaコンパイラとJVM JITコンパイラは裏側で様々なことをやっているのでマイクロベンチマークを書くのはとてつもなく難しいです。大抵の場合、書き上げたマイクロベンチマークのコードはあなたが意図したものを正常に計測できていません。

マイクロベンチマークコードを書くならば[jmh](http://openjdk.java.net/projects/code-tools/jmh/)を使ってください。必ず[すべてのマイクロベンチマークのサンプル](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/)に目を通してください。デッドコードを排除、定数の畳み込み(constant folding)、ループアンローリング(loop unrolling)した場合の効果について理解が深まります。

### <a name='perf-whileloops'>走査(traversal)とzipWithIndex</a>

`for` ループや関数型のトランスフォーム関数(e.g. `map`, `foreach`)よりも `while` ループを使いましょう。 `for` ループや関数型のトランスフォーム関数は処理が遅いです(仮想関数(virtual function)の呼び出しやボックス化が要因)。
Use `while` loops instead of `for` loops or functional transformations (e.g. `map`, `foreach`). For loops and functional transformations are very slow (due to virtual function calls and boxing).
```scala

val arr = // intの配列
// 添字が偶数の場合に0埋め
val newArr = list.zipWithIndex.map { case (elem, i) =>
  if (i % 2 == 0) 0 else elem
}

// 以下は上の処理の高性能版
val newArr = new Array[Int](arr.length)
var i = 0
val len = newArr.length
while (i < len) {
  newArr(i) = if (i % 2 == 0) 0 else arr(i)
  i += 1
}
```

### <a name='perf-option'>Optionとnull</a>

パフォーマンスを意識する必要がある場合は `Option` よりも `null` を使って仮想関数(virtual method)呼び出しやボックス化が発生するのを防ぎましょう。nullになり得るフィールドに関してはNullableで装飾しておきましょう。
```scala
class Foo {
  @javax.annotation.Nullable
  private[this] var nullableField: Bar = _
}
```

### <a name='perf-collection'>Scalaコレクションライブラリ</a>

パフォーマンスを意識する必要がある場合はScalaよりもJavaのコレクションライブラリを使いましょう。Scalaのコレクションライブラリの方がJavaのそれよりパフォーマンスが劣るからです。

### <a name='perf-private'>private[this]</a>

パフォーマンスを意識する必要がある場合は `private` よりも `private[this]` を使いましょう。 `private[this]` はフィールドを生成しますが `private` はアクセサメソッドを生成してしまうからです。我々の経験上JVM JITコンパイラは必ずしも `private` フィールドのアクセサメソッドをインライン化するとは限らないので `private[this]` を使って仮想関数の呼び出しが発生しないように実装する方が安全です。
```scala
class MyClass {
  private val field1 = ...
  private[this] val field2 = ...

  def perfSensitiveMethod(): Unit = {
    var i = 0
    while (i < 1000000) {
      field1  // これは仮想関数を呼び出す可能性があります
      field2  // これはただのフィールドアクセスになります
      i += 1
    }
  }
}
```


## <a name='java'>Javaとの互換性</a>

このセクションではJavaとの互換をもつAPIを作る際のガイドラインについて述べます。Javaとの互換性が必要ない場合はこれらを適用する必要はありません。この内容のほとんどは我々がSparkのJava APIを作った際の経験から築かれたものです。


### <a name='java-missing-features'>JavaにはあってScalaには無い機能</a>

以下に述べるJavaの機能はScalaにはありません。これらが必要な場合はJavaで定義してください。ただし、Javaのファイルに関してはScalaDocsは生成されないので注意してください。

- staticフィールド
- static内部クラス(inner class)
- Java enum
- アノテーション


### <a name='java-traits'>トレイト(trait)と抽象(abstract)クラス</a>

For interfaces that can be implemented externally, keep in mind the following:

- デフォルト実装を持ったトレイトのメソッドはJavaでは使えません。代わりに抽象クラスを使いましょう。
- 一般的にトレイトを使うのは避けてください。ただし、未来永劫そのインタフェースがデフォルト実装されない確信がある場合は許容します。
```scala
// トレイトのデフォルト実装はJavaでは動きません
trait Listener {
  def onTermination(): Unit = { ... }
}

// 以下はJavaでも動きます
abstract class Listener {
  def onTermination(): Unit = { ... }
}
```


### <a name='java-type-alias'>型エイリアス(Type Aliases)</a>

型エイリアスは使ってはいけません。バイトコード上（Java含む）でこれらは見えません。


### <a name='java-default-param-values'>デフォルト引数(Default Parameter Values)</a>

デフォルト引数を使ってはいけません。代わりにオーバーロードしましょう。
```scala
// 以下はJavaとの互換性を崩します
def sample(ratio: Double, withReplacement: Boolean = false): RDD[T] = { ... }

// 以下2つは問題ありません
def sample(ratio: Double, withReplacement: Boolean): RDD[T] = { ... }
def sample(ratio: Double): RDD[T] = sample(ratio, withReplacement = false)
```

### <a name='java-multi-param-list'>複数の引数リスト(Multiple Parameter List)</a>

複数の引数リストは使ってはいけません。

### <a name='java-varargs'>可変長引数(vararg)</a>

- 可変長引数(vararg)メソッドには `@scala.annotation.varargs` アノテーションを適用して、Javaからも使えるようにしましょう。ScalaコンパイラはScala用(バイトコード上はパラメータをSeqとして)とJava用(バイトコード上はパラメータを配列として)の2つのメソッドを生成します。
  ```scala
  @scala.annotation.varargs
  def select(exprs: Expression*): DataFrame = { ... }
  ```

- Scalaコンパイラのバグ([SI-1459](https://issues.scala-lang.org/browse/SI-1459), [SI-9013](https://issues.scala-lang.org/browse/SI-9013))によりabstractな可変長引数(vararg)メソッドはJavaでは動きません。

- 可変長引数(vararg)メソッドをオーバーロードする場合は注意が必要です。可変長引数メソッドを違う型の可変長引数メソッドでオーバーロードした場合はソース互換性が無くなる可能性があります。
  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...

    // 以下のオーバーロードを定義すると引数無しのremove()に対するソース互換性が無くなります
    @scala.annotation.varargs
    def remove(elems: People*): Unit = ...
  }

  // 以下は曖昧さ故コンパイルできなくなります
  new Database().remove()
  ```
  対処法として、第一引数を明示的に定義して、第二引数以降を可変長引数(vararg)にします：
  ```scala
  class Database {
    @scala.annotation.varargs
    def remove(elems: String*): Unit = ...

    // 以下はOK
    @scala.annotation.varargs
    def remove(elem: People, elems: People*): Unit = ...
  }
  ```


### <a name='java-implicits'>implicit</a>

implicitをクラスやメソッドで使ってはいけません。 `ClassTag` と `TypeTag` も含みます。
```scala
class JavaFriendlyAPI {
  // このメソッドはimplicitパラメータ(ClassTag)を含んでいるのでJavaから見た場合に良いメソッドではありません
  def convertTo[T: ClassTag](): T
}
```

### <a name='java-companion-object'>コンパニオンオブジェクト、staticメソッド、フィールド</a>

コンパニオンオブジェクト、staticメソッド、フィールドを扱うにいい当たってはいくつかのことに注意しなければいけません。

- コンパニオンオブジェクトをJavaで扱う場合は不自然な形になります（ `Foo` というコンパニオンオブジェクトがあったらそれは `Foo$` クラスの中の `Foo$` 型の `MODULE$` というstaticフィールドになります）。
  ```scala
  object Foo

  // Javaでは以下のようになります
  public class Foo$ {
    Foo$ MODULE$ = // オブジェクトをインスタンス化
  }
  ```
  コンパニオンオブジェクトをどうしても使わないといけない場合はJavaのstaticフィールドの別のクラスで用意してください

- 残念ながらScalaでJVMのstaticフィールドを定義する方法はありません。Javaのファイルを作って定義するしかありません。
- コンパニオンオブジェクトのメソッドは自動的にコンパニオンクラスのstaticメソッドに変換されます。ただし、メソッド名で競合が起きた場合はこの限りではありません。staticメソッドの生成を（将来も見据えて）保証したい場合はJavaで書いたテストファイルを作ってその中でstaticメソッドを呼ぶことです。
  ```scala
  class Foo {
    def method2(): Unit = { ... }
  }

  object Foo {
    def method1(): Unit = { ... }  // staticメソッド Foo.method1がバイトコード上で生成されます
    def method2(): Unit = { ... }  // staticメソッド Foo.method2はバイトコード上で生成されません
  }

  // FooJavaTest.java (test/scala/com/databricks/...で作ったと想定)
  public class FooJavaTest {
    public static compileTest() {
      Foo.method1();  // コンパイルは問題なく通ります
      Foo.method2();  // これはmethod2が生成されていないので失敗するはずです
    }
  }
  ```

- caseオブジェクト（あるいはただのコンパニオンオブジェクト）MyClassというものは実際はMyClass型ではありません。
  ```scala
  case object MyClass

  // Test.java
  if (MyClass$.MODULE instanceof MyClass) {
    // 上の式は必ずfalseになります
  }
  ```
  型の階層を正しく実装したい場合はコンパニオンクラスを定義して、caseオブジェクトにそのクラスを継承させましょう：
  ```scala
  class MyClass
  case object MyClass extends MyClass
  ```


## <a name='misc'>その他</a>

### <a name='misc_currentTimeMillis_vs_nanoTime'>currentTimeMillisよりもnanoTime推奨</a>

*期間* の演算や *タイムアウト* のチェックをする場合は `System.currentTimeMillis()` を使うのは控えましょう。サブミリ秒精度を必要としない場合でも `System.nanoTime()` を使いましょう。

`System.currentTimeMillis()` は現在時刻を返し、システム時計の時刻が変わった場合には影響を受けます。よって、システム時計の時間を戻した場合にはタイムアウトが長時間(時間が元々の値に戻るまで)「ハング」してしまう可能性があります 。この現象はネットワークが切れてしまった後にntpdが `step` を実行した場合に発生し得ます。よくある例として、システムのブート時にDHCPが普段よりも時間がかかるというものがあります。これは異常処理を誘発し、意味を理解するのも困難であれば再現性も低い問題となります。 `System.nanoTime()` の値はシステム時間に関係なく増えることしか無いことが保証されています。

注意:
- 絶対に `nanoTime()` の絶対値はシリアライズしてはいけませんし、他のシステムに渡してもいけません。絶対値はシステム特有の値であり、リブート時にはリセットされるので意味がありません。
- `nanoTime()` の絶対値は正数であることは保証されていません(ただし、 `t2 - t1`は正しい値を算出していることを保証されています。
- `nanoTime()` は292年という期間を行き来します。なので、Sparkのジョブが長時間かかる場合は考えなおす必要があります。

### <a name='misc_uri_url'>URLよりもURI推奨</a>

あるサービスのURLを格納するときには `URI` を使いましょう。

`URL`の [等価性チェック(equality check)](http://docs.oracle.com/javase/7/docs/api/java/net/URL.html#equals(java.lang.Object)) には実際にIPアドレスの解決を行うために(ブロックを引き起こす)ネットワークへの通信が行われます。 `URI` はフィールドの等価性チェックを行い、機能としては `URL` のスーパーセットとなります。

