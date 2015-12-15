---
layout: post
title: "ScalaのREPLを拡張するには"
date: 2014-12-06 18:04:35 +0900
comments: true
categories: scala repl android
---
こちらは[Scala AdventCalendar 2014](http://qiita.com/advent-calendar/2014/scala)の7日目の記事です。
今日はScalaのカスタムREPLの作り方についての話。なお今回は(Scala REPL同様)StandaloneなREPLアプリの作成を目的としているので、[`:power`モード](http://www.ne.jp/asahi/hishidama/home/tech/scala/repl/power.html)は主眼ではありません。

#モチベーション
ScalaのREPLは手元のローカルマシンでAPIを試してみたいときや、ちょっとした計算をしたいときにはとても便利なのですが、やや凝ったことをしたいときなど、そのまま使うには不便さを感じることがあります。

具体的にはAndroidのAPIをScalaのREPLから叩けるようにしたかったのですが、Android環境をJVMでエミュレートするためにはカスタムClassLoaderを使ってAndroid APIのClassを書き換える必要があって。。。という感じ。

これはちょっと特殊なモチベーションかもしれませんが、クラスター上などの特定の環境で実行させたいとき(e.g.[`spark-shell`](http://spark.apache.org/docs/latest/quick-start.html#interactive-analysis-with-the-spark-shell))、自作ライブラリのsandbox環境を提供するにあたって特定の場面でよく使うコマンドを追加したい、など思われる方はいるかもしれません。

そんなとき、意外とScala REPLを拡張する記事を書いている人が少なかったので、今回はScalaのソースコードを読みながらカスタムClassLoaderを使用する方法、コマンドを追加する方法について書くことにしました。

参考:[Create your custom Scala REPL](http://www.michaelpollmeier.com/create-your-custom-scala-repl/) ...  数少ないREPL拡張方法に関する記事。

#成果物
[`taisukeoe/MyCLRepl`](https://github.com/taisukeoe/MyCLRepl)

```scala MyCLRepl DEMO
scala> val hello = "hello"

MyClassLoader loads classOf <root>.$line3
<<中略>>
MyClassLoader loads classOf scala.collection.mutable.StringBuilder
MyClassLoader loads classOf scala.runtime.ScalaRunTime$
hello: String = hello

scala> :myCommand hello

This is a custom command example. You can do something from value:"hello" with custom Scala interpreter.
```
ClassLoaderの差し替え(Classのロード時にクラス名をprint)と、myCommandというコマンドの追加をしています。

<!--more-->

#REPLとは
[Read–Eval–Print Loop](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)の略で、対話型の開発環境。ユーザーの入力したコードを一行から評価する。
ScalaのREPLは`scala`コマンドから開始できる。

#ScalaのREPLの構成
[`scala.tools.nsc.interpreter`](https://github.com/scala/scala/tree/v2.11.4/src/repl/scala/tools/nsc)パッケージがREPLに相当。インタプリタのソースコードがそれなりの量あるので一見大変そうだが、構成自体はシンプル。REPLをカスタマイズする上で最低限見る必要がある場所に絞って、以下では解説します。

##エントリポイント
scalaのスクリプトの中身を見ると、以下の通り`scala.tools.nsc.MainGenericRunner`の`main`関数をエントリポイントとしてREPLを起動していることがわかる。

```bash /usr/local/bin/scala#L202-212
execCommand \
  "${JAVACMD:=java}" \
  $JAVA_OPTS \
  "${java_args[@]}" \
  $(classpathArgs) \
  -Dscala.home="$SCALA_HOME" \
  $OVERRIDE_USEJAVACP \
  "$EMACS_OPT" \
  $WINDOWS_OPT \
  scala.tools.nsc.MainGenericRunner  "$@"
```

##Main関数の中でLoopを呼び出し
[`scala.tools.nsc.MainGenericRunner`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/MainGenericRunner.scala#L74)の`main`関数の処理は、同コンパニオンクラスの`process`関数に移譲している。この関数の中で、[`ILoop`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala)の`process`関数を呼んでおり、このクラスがREPLのLoopに相当。

```scala scala.tools.nsc.MainGenericRunner https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/MainGenericRunner.scala#L74
class MainGenericRunner {
...
  def process(args: Array[String]): Boolean = {
    ...
    def run(): Boolean = {
      ....
      
      def runTarget(): Either[Throwable, Boolean] = howToRun match {
        ...
        case _  =>
          // We start the repl when no arguments are given.
          Right(new interpreter.ILoop process settings)
      }
     ...
    }
    ...
      run()
  }
}

object MainGenericRunner extends MainGenericRunner {
  def main(args: Array[String]): Unit = if (!process(args)) sys.exit(1)
}
```

参考:[scala.tools.nsc.MainGenericRunner.run](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/MainGenericRunner.scala#L74)

##Loopの中でインタプリタを呼び出し
[`scala.tools.nsc.interpreter.ILoop`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala)はREPLのLoopに相当するクラス。コマンドの判定や、入力したコードのインタプリタへの移譲などを行っている。

REPLのコマンド(e.g.`:help`)の定義は`standardCommands`変数。overrideした`commands`関数において追加することでカスタムCommandを実装できる。

また、[`ILoop`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala)の`createInterpreter`関数内で、インタプリタのメンバ変数`var intp:IMain`を初期化している。

なおインタプリタ `intp`の実装クラスは[`IMain`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/IMain.scala)を継承した[`ILoopInterpreter`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala#109)。

```scala scala.tools.nsc.interpreter.ILoop https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala
class ILoop(in0: Option[BufferedReader], protected val out: JPrintWriter)
                extends AnyRef
                   with LoopCommands
{
...
var intp: IMain = _
...

　  lazy val standardCommands = List(
    cmd("edit", "<id>|<line>", "edit history", editCommand),
    cmd("help", "[command]", "print this summary or command-specific help", helpCommand),
    historyCommand,
    cmd("h?", "<string>", "search the history", searchHistory),
    cmd("imports", "[name name ...]", "show import history, identifying sources of names", importsCommand),
    cmd("implicits", "[-v]", "show the implicits in scope", intp.implicitsCommand),
    cmd("javap", "<path|class>", "disassemble a file or class name", javapCommand),
    cmd("line", "<id>|<line>", "place line(s) at the end of history", lineCommand),
    cmd("load", "<path>", "interpret lines in a file", loadCommand),
    cmd("paste", "[-raw] [path]", "enter paste mode or paste a file", pasteCommand),
    nullary("power", "enable power user mode", powerCmd),
    nullary("quit", "exit the interpreter", () => Result(keepRunning = false, None)),
    cmd("replay", "[options]", "reset the repl and replay all previous commands", replayCommand),
    cmd("require", "<path>", "add a jar to the classpath", require),
    cmd("reset", "[options]", "reset the repl to its initial state, forgetting all session entries", resetCommand),
    cmd("save", "<path>", "save replayable session to a file", saveCommand),
    shCommand,
    cmd("settings", "<options>", "update compiler options, if possible; see reset", changeSettings),
    nullary("silent", "disable/enable automatic printing of results", verbosity),
    cmd("type", "[-v] <expr>", "display the type of an expression without evaluating it", typeCommand),
    cmd("kind", "[-v] <expr>", "display the kind of expression's type", kindCommand),
    nullary("warnings", "show the suppressed warnings from the most recent line which had any", warningsCommand)
  )
  
  ...

    /** Available commands */
  def commands: List[LoopCommand] = standardCommands ++ (
    if (isReplPower) powerCommands else Nil
  )
 
  ...
 
  class ILoopInterpreter extends IMain(settings, out) {
    outer =>

    override lazy val formatting = new Formatting {
      def prompt = ILoop.this.prompt
    }
    override protected def parentClassLoader =
      settings.explicitParentLoader.getOrElse( classOf[ILoop].getClassLoader )
  }     
  /** Create a new interpreter. */
  def createInterpreter() {
    if (addedClasspath != "")
      settings.classpath append addedClasspath

    intp = new ILoopInterpreter
  }
  
  ...
  
  // start an interpreter with the given settings
  def process(settings: Settings): Boolean = savingContextLoader {
    this.settings = settings
    createInterpreter()
     ...
    printWelcome()
     ...
    try loop() match {
      case LineResults.EOF => out print Properties.shellInterruptedString
      case _               =>
    }
    catch AbstractOrMissingHandler()
    finally closeInterpreter()
      
    true
  }
  ...
```

##インタプリタの中でClassLoaderを生成
[`scala.tools.nsc.interpreter.IMain`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/IMain.scala)がインタプリタの主たるコード。

`private var _classLoader:ClassLoader`がClassLoaderのメンバ変数だが、privateなのでclassLoader関数をoverrideしてカスタムClassLoaderを生成する。

ただし、`IMain`の`classLoader`関数の戻り値の型は[`scala.reflect.internal.util.AbstractFileClassLoader`](https://github.com/scala/scala/blob/2.11.x/src/repl/scala/tools/nsc/interpreter/AbstractFileClassLoader.scala)になので、カスタムClassLoaderもこれを継承させる必要がある点に注意。

```scala scala.tools.nsc.interpreter.IMain https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/IMain.scala
class IMain(@BeanProperty val factory: ScriptEngineFactory, initialSettings: Settings, protected val out: JPrintWriter) extends AbstractScriptEngine with Compilable with Imports {
  imain =>

  ...
  
  private var _classLoader: util.AbstractFileClassLoader = null                              // active classloader
  ....

  /** Parent classloader.  Overridable. */
  protected def parentClassLoader: ClassLoader =
    settings.explicitParentLoader.getOrElse( this.getClass.getClassLoader() )

  def resetClassLoader() = {
    repldbg("Setting new classloader: was " + _classLoader)
    _classLoader = null
    ensureClassLoader()
  }
    final def ensureClassLoader() {
    if (_classLoader == null)
      _classLoader = makeClassLoader()
  }
  def classLoader: util.AbstractFileClassLoader = {
    ensureClassLoader()
    _classLoader
  }

```

#カスタムREPLを作る
基本的には解説した以下のクラスに相当するものを継承なり自前で作るなりすれば、カスタムREPLが作れます。

 * [`scala.tools.nsc.MainGenericRunner`]() ... REPLを起動するmain関数、Loopの呼び出し
 * [`scala.tools.nsc.interpreter.ILoop`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala) ... Loopの実装。Commandの判定。インタプリタへ処理の移譲。
 * [`scala.tools.nsc.interpreter.IMain`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/IMain.scala)を継承した[`scala.tools.nsc.interpreter.ILoop.ILoopInterpreter`](https://github.com/scala/scala/blob/v2.11.4/src/repl/scala/tools/nsc/interpreter/ILoop.scala#109) ... インタプリタの実装。ClassLoaderの生成。
 * [`scala.reflect.internal.util.AbstractFileClassLoader`](https://github.com/scala/scala/blob/2.11.x/src/repl/scala/tools/nsc/interpreter/AbstractFileClassLoader.scala)を継承したカスタムClassLoader ... インタプリタに使用させる。
 
先ほども記載しましたが、完成物は以下の通り。試し方はREADME参照のこと。

[taisukeoe/MyCLRepl](https://github.com/taisukeoe/MyCLRepl)
 

#REPLを拡張している実例
[Apache Spark](https://spark.apache.org/)のspark-shellは、REPL上で入力したコマンドをcluster上で実行させるために、REPLを拡張している。以下のファイル群がカスタムREPLに相当する。

* [spark-shell](https://github.com/apache/spark/blob/master/bin/spark-shell#L61)
* [org.apache.spark.repl.Main](https://github.com/apache/spark/blob/e895e0cbecbbec1b412ff21321e57826d2d0a982/repl/scala-2.11/src/main/scala/org/apache/spark/repl/Main.scala) 
* [org.apache.spark.repl.SparkILoop](https://github.com/apache/spark/blob/e895e0cbecbbec1b412ff21321e57826d2d0a982/repl/scala-2.11/src/main/scala/org/apache/spark/repl/SparkILoop.scala#L902)
* [org.apache.spark.repl.SparkIMain](https://github.com/apache/spark/blob/e895e0cbecbbec1b412ff21321e57826d2d0a982/repl/scala-2.11/src/main/scala/org/apache/spark/repl/SparkIMain.scala)


[ScalaMatsuri](http://scalamatsuri.org/)で発表された[Scalive](https://github.com/xitrum-framework/scalive)もREPLを拡張してJVM環境を触れるようにしている。

##余談:Scalaの:power モード
Scala REPLに備わっている[`:power`モード](http://www.ne.jp/asahi/hishidama/home/tech/scala/repl/power.html)を使用すると、Scala REPLコードのpublicな変数・関数へのアクセスが可能で、varなら置き換えも可能。

例えば上記のカスタムClassLoader挿入は、`:power`モードを利用してこんな風にも書けます。（ClassLoaderのサイズが膨らんでくるとつらさはありますが）

```scala
scala> :power

** Power User mode enabled - BEEP WHIR GYVE **
** :phase has been set to 'typer'.          **
** scala.tools.nsc._ has been imported      **
** global._, definitions._ also imported    **
** Try  :help, :vals, power.<tab>           **

scala> :paste
// Entering paste mode (ctrl-D to finish)

import scala.reflect.internal.util.ScalaClassLoader
import scala.tools.nsc.interpreter.JPrintWriter
import scala.tools.nsc.io.AbstractFile
import scala.tools.nsc.util
repl.intp = new repl.ILoopInterpreter {
   override def classLoader = new util.AbstractFileClassLoader(root:AbstractFile,parent:ClassLoader)  with ScalaClassLoader{
  override def loadClass(name: String) = {
    println(s"MyClassLoader loads classOf ${name}")
    super.loadClass(name)
  }
 }
}

scala> val hoge = "hoge"

MyClassLoader loads classOf <root>.$line2
MyClassLoader loads classOf $line2
<<中略>>
MyClassLoader loads classOf scala.runtime.ScalaRunTime$
MyClassLoader loads classOf scala.runtime.BoxedUnit

hoge: String = hoge
```

ただその一方でREPLのコマンドの判別はILoopクラスで行わており、MainGenericRunnerの内部関数中で直接呼び出されているのでPowerModeからのカスタムCommandの追加は不可能なはず（とはいいつつ、何がしかのhackもありそうな気がするのでもしあればコメント欄で教えてください。）

##余談
肝心の、オレオレClassLoaderを使ってAndroid APIを叩けるScala REPLアプリは間に合わず。現状でも、未解決のwarningが出る問題とか、タブ補完が効かない問題など、色々と雑な感じにはなっています。。。

そのあたりのフォローはまた後日に('・ω・`)