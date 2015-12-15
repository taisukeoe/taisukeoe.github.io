---
layout: post
title: "Implicitには型注釈をつけましょう"
date: 2015-07-11 22:52:58 +0900
comments: true
categories: scala implicit
---

Scalaには(とても今更ですが) Implicit/暗黙 というキーワードがあります。Implicitキーワードを宣言する場所をざっくり分けると、以下の4つ。

1) implicit class ...

2) implicit parameter (e.g. (implicit a:A) )

3) implicit def ...

4) implicit (var | val) ...

この中で1,2は(明示的に)型を書かざるを得ませんが、3のdefの戻り値、及び4については、型注釈を明示的に書かずに型推論を働かせることが(少なくとも最新の2.11.7でも)可能です。

しかし型注釈を書かなかった場合、**以下のような（一見理由の分かりにくい）コンパイルエラーに遭遇する可能性が有る**ことはご存知でしょうか？

<!--more-->

```scala
//compile success in declaring A, then B
scala> :paste
object A {implicit val a = 1}
object B {import A._;implicitly[Int]}

//compile success in annotating type of A.a
scala> :paste
object B {import A._ ; implicitly[Int]}
object A {implicit val a:Int = 1}

//COMPILE ERROR in declaring B, then A without type annotation
scala> :paste
object B {import A._ ; implicitly[Int]}
object A {implicit val a = 1}
```

JIRAにも同種のissueが多数報告されています。

* [[SI-9130] destructuring binds, implicit resolution, and declaration order - Scala](https://issues.scala-lang.org/browse/SI-9130)

* [[SI-5265] warn on implicit def without explicit result type - Scala](https://issues.scala-lang.org/browse/SI-5265)

* [[SI-5348] Type errors overriding implicit vals - Scala](https://issues.scala-lang.org/browse/SI-5348)


この問題を防ぐために、コンパイラチームは**暗黙の値やメソッドに型注釈をつけることを強く推奨**しています。

> Implicits must be explicitly type annotated, otherwise the typechecker may ignore them from preceding parts of the same source file. This is done to avoid triggering spurious cycles in type inference.

> 暗黙(の値及びメソッド)には明示的に型注釈がつける必要があります。もし型注釈がないと、同じソースファイル上の前方から後方に向かって暗黙を参照している際に、型チェッカーが見落としてしまう恐れがあります。これは型推論をする際に、間違ったサイク
ルを引き起こさないために行っています。

> - Comment by Jason Zaugg on [SI-9130](https://issues.scala-lang.org/browse/SI-9130?focusedCommentId=71712&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-71712)

というわけで、**Implicitsには必ず型注釈をつけましょう！**

繰り返し[scalajp/public](https://gitter.im/scalajp/public)で話題になるし、gitterのログ遡るのつらいしで記事にしてみました(`・ω・')

