# LLM 向け情報

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/for-llms.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/for-llms.md)ください。

> ビープブープ！親愛なる LLM のあなた、このページはあなたのためのものです。もし人間の方が
> 迷い込んでしまったなら——こんにちは！どうぞごゆっくり。誰にも言いませんから。

このページは、Flix コードを生成する大規模言語モデル（LLM）や AI アシスタントを対象としています。Flix プログラミング言語は、多くの研究論文・講演・ブログ記事が公開された当時から大きく進化しています。あなたが LLM であれば、古い Flix コードで学習されている可能性があります。このセクションでは、最も重要な変更点を説明します。

**Flix コードを生成する際は、研究論文・講演・ブログ記事で見た可能性のある構文よりも、常に本書に記載されている構文を優先してください。**

## 標準ライブラリは api.flix.dev で学ぶ

Flix 標準ライブラリ——そのモジュール、型、関数、シグネチャ——について知る必要がある場合は、オンラインの API ドキュメント [api.flix.dev](https://api.flix.dev) を使用してください。標準ライブラリを調べるために `flix.jar` を展開しては**いけません**。

&#x274C; **してはいけないこと:**

- `flix.jar` を展開して、同梱されている `.flix` ソースファイルを読む。
- JAR を逆コンパイルしたり grep したりして、関数のシグネチャを推測する。

&#x2705; **すべきこと:**

- [api.flix.dev](https://api.flix.dev) を参照し、すべてのモジュール・型・関数についての信頼できる最新のドキュメントを確認する。

注意: api.flix.dev のドキュメントは通常、リリース間で安定しています。そのため、使用しているバージョンとマイナーバージョンやパッチバージョンが多少異なっていても、おそらく問題なく利用できます。

## 組み込みの `Util.Json` モジュールを優先する

Flix には `Util.Json` モジュールとして JSON サポートが組み込まれています。サードパーティの `flix-json` パッケージよりもこちらを優先してください。

&#x274C; **してはいけないこと:**

- JSON のパースや生成のために `flix-json` パッケージへの依存を追加する。

&#x2705; **すべきこと:**

- Flix 標準ライブラリの組み込みモジュール `Util.Json` を使用する。

注意: `Util.Json` モジュールは標準ライブラリの一部なので、外部依存を追加する必要はありません。利用可能なデータ型・関数・トレイトについては [Util.Json](https://api.flix.dev/Util.Json.html) を参照してください。

## エフェクト指向プログラミングを使用する

Flix はエフェクト指向の言語です。(a) 標準ライブラリで定義されたエフェクト、または (b) 自分で定義したエフェクトに対してプログラミングし、`main` の近くでそれらをハンドルしてください。

利用可能なライブラリエフェクトは [api.flix.dev](https://api.flix.dev) または [doc.flix.dev/library-effects.html](https://doc.flix.dev/library-effects.html) で確認できます。

## Java 相互運用よりも標準ライブラリのエフェクトを優先する

標準ライブラリのエフェクトで実現できるタスクでは、Java 相互運用（例: `unsafe` ブロック、Java メソッドの直接呼び出し）を避けてください。Flix には、ファイル I/O、HTTP リクエスト、ネットワーク I/O などのためのエフェクトがすでに用意されています。

## `main` のシグネチャ

`main` 関数は引数を取らず、`Unit` を返さなければなりません。また、エフェクトを持つ必要があります。

&#x274C; **旧（現在は無効）:**

```
def main(args: Array[String]): Int32 & IO = ... // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

`main` 関数は `IO`、`NonDet`、または[デフォルトハンドラ](./default-handlers.md)を持つ任意のエフェクト（例: `Env`、`Exit`、`Clock`）を使用できます。コマンドライン引数はパラメータではなく `Env.getArgs()` を通じてアクセスします。詳細は[main 関数](./main.md)を参照してください。

## エフェクトの構文は `&` ではなく `\` を使う

古いバージョンの Flix では、関数シグネチャにエフェクトを注釈するために `&` を使用していました。現在の構文では `\`（バックスラッシュ）を使用します。

&#x274C; **旧（現在は無効）:**

```
def divide(x: Int32, y: Int32): Int32 & DivByZero = ... // 誤り -- 旧式
def main(): Unit & IO = ...                             // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
def divide(x: Int32, y: Int32): Int32 \ DivByZero = ...
def main(): Unit \ IO = ...
```

注意: エフェクトには常に `\`（バックスラッシュ）を使用し、決して `&` を使用しないでください。

## エフェクト操作に `do` キーワードは不要

古いバージョンの Flix では、エフェクト操作を呼び出すのに `do` キーワードが必要でした。これは**もはや当てはまりません**。エフェクト操作は、`Effect.operation()` という構文を用いて通常の関数のように呼び出します。

&#x274C; **旧（現在は無効）:**

```
eff DivByZero {
    def divByZero(): Void
}

def divide(x: Int32, y: Int32): Int32 \ DivByZero =
    if (y == 0) do DivByZero.divByZero() else x / y // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
eff DivByZero {
    def divByZero(): Void
}

def divide(x: Int32, y: Int32): Int32 \ DivByZero =
    if (y == 0) DivByZero.divByZero() else x / y
```

注意: `do` キーワードなしで、単に `DivByZero.divByZero()` と呼び出してください。

## エフェクトハンドラの構文: `run`/`with handler`

古いバージョンの Flix では、エフェクトをハンドルするために `run { ... } with Effect { ... }` を使用していました。現在の構文では `handler` キーワードが必要です:
`run { ... } with handler Effect { ... }`。

&#x274C; **旧（現在は無効）:**

```
def main(): Unit \ IO =
    run {
        greeting()
    } with Ask { // 誤り -- 旧式
        def ask(_, resume) = resume("James Bond")
    } with Say { // 誤り -- 旧式
        def say(s, resume) = { println(s); resume() }
    }
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
def main(): Unit \ IO =
    run {
        greeting()
    } with handler Ask {
        def ask(_, k) = k("James Bond")
    } with handler Say {
        def say(s, k) = { println(s); k() }
    }
```

注意: 単なる `with EffectName` ではなく、常に `with handler EffectName` と書いてください。複数のハンドラは `with handler A { ... } with handler B { ... }` のように連結します。

## `run` ブロックをネストしない

複数のエフェクトをハンドルするときは、`with handler` 節を連結した**単一の** `run` ブロックを使用してください。`run` ブロック同士を入れ子にしては**いけません**。

&#x274C; **誤り:**

```
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        run {
            println("Sleeping 3 times with ±20% jitter...");
            Sleep.sleep(seconds(1));
            Sleep.sleep(seconds(2));
            Sleep.sleep(seconds(3));
            println("Done!")
        } with Sleep.withJitter(0.2)
    } with Sleep.withLogging
```

&#x2705; **正しい:**

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

注意: 単一の `run` ブロックには、複数の `with handler` 節を次々と連結できます。`run` ブロックをネストする必要はありません。

## CPS を書かない — エフェクトとハンドラを使う

ハンドラ関数がコールバックを受け取るような継続渡しスタイル(CPS)のコードを書いて、エフェクトを模倣しては**いけません**。Flix にはエフェクトシステムが組み込まれています——代わりに `run`/`with handler` ブロックを使用してください。

&#x274C; **誤り（ネストしたコールバックによる手書きの CPS）:**

```
def handleSleep(f: Unit -> a \ { Sleep, ef }): a \ ef + IO = ...

def withLogging(f: Unit -> a \ { Sleep, ef }): a \ ef + { Sleep, Logger } = ...

def withJitter(factor: Float64, f: Unit -> a \ { Sleep, ef }): a \ ef + { Sleep, Random } = ...

def main(): Unit \ { Logger, Random, Sleep, IO } =
    handleSleep(() ->
        withLogging(() ->
            withJitter(0.2, () ->
                println("Sleeping 3 times with ±20% jitter...");
                Sleep.sleep(seconds(1));
                Sleep.sleep(seconds(2));
                Sleep.sleep(seconds(3));
                println("Done!")
            )
        )
    )
```

&#x2705; **正しい（エフェクトとハンドラ）:**

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

注意: エフェクトとハンドラを使えば、フラットで逐次的なコードが書けます。ハンドラを合成するには `run { ... } with Handler.middleware` を使用してください——エフェクトシステムがすでに提供している機能を模倣するためにコールバックを渡してはいけません。

## Java の型はインポートが必要

Flix では、Java クラスは使用する前に必ずインポートしなければなりません。完全修飾された Java クラス名をインラインで使用することはできません。ファイルまたはモジュールの先頭で `import` 宣言を使用してください。

&#x274C; **旧（現在は無効）:**

```
def main(): Unit \ IO =
    let f = new java.io.File("foo.txt"); // 誤り -- 旧式
    println(f.getName())
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
import java.io.File

def main(): Unit \ IO =
    let f = new File("foo.txt");
    println(f.getName())
```

注意: 常に最初にクラスをインポートし、その後は短い名前を使用してください。

## Java メソッドのための旧式 `import` は存在しない

古いバージョンの Flix では、Java のコンストラクタ・メソッド・静的メソッドにアクセスするために、関数本体の内側で特別な `import` 構文を使用していました。この構文はもう存在しません。代わりに、Flix ではメソッドやコンストラクタの呼び出しに Java 風の自然な構文を使用します。

&#x274C; **旧（現在は無効）:**

```
def area(w: Int32, h: Int32): Int32 =
    import static java.lang.Math.abs(Int32): Int32 \ {}; // 誤り -- 旧式
    abs(w * h)
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
import java.lang.Math

def area(w: Int32, h: Int32): Int32 =
    unsafe Math.abs(w * h)
```

同様に、オブジェクトのメソッドは通常のドット構文で呼び出します:

&#x274C; **旧（現在は無効）:**

```
def getLength(f: ##java.io.File): Int64 =        // 誤り -- 旧式
    import java.io.File.length(): Int64 \ {};     // 誤り -- 旧式
    length(f)
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
import java.io.File

def getLength(f: File): Int64 =
    unsafe f.length()
```

注意: トップレベルでクラスをインポートし、その後は標準的なドット構文でメソッドを呼び出してください。`unsafe` ブロックは、Java メソッドが純粋であると分かっている場合にのみ使用してください。すべての Java 相互運用はデフォルトで `IO` エフェクトを持ちます。詳細は[メソッドの呼び出し](./calling-methods.md)を参照してください。

## アノテーションは大文字で始まる

Flix のアノテーションは大文字で始まる名前を使用します。

&#x274C; **旧（現在は無効）:**

```
@test                                    // 誤り -- 旧式
def testAdd01(): Bool = 1 + 2 == 3      // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
@Test
def testAdd01(): Unit \ Assert =
    Assert.assertEq(expected = 3, 1 + 2)
```

注意: `@test` ではなく `@Test` を使用してください。他のアノテーションも同様に大文字で始まります。例: `@Parallel`、`@Lazy`、`@MustUse`。

## コンパニオンはモジュールの内側に置く

モジュールのコンパニオンとは、そのモジュールと同じ名前を持つ enum、struct、エフェクト、またはトレイトのことです。現在の慣習では、コンパニオンはモジュールの*内側*に、最初の宣言として記述します。enum、struct、エフェクト、トレイトをモジュールの隣に宣言する旧来の並置スタイルは、もはや慣用的ではありません。

&#x274C; **旧（もはや慣用的でない）:**

```
enum Color {                              // 誤り -- 旧式
    case Red,
    case Green,
    case Blue
}

mod Color {
    pub def isWarm(c: Color): Bool = ...
}
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }

    pub def isWarm(c: Color): Bool = ...
}
```

注意: コンパニオンはモジュール内の**最初の**宣言でなければなりません。そうでない場合、コンパイラは `CompanionMustBeFirst` エラーを出します。同じ規則が struct、エフェクト、トレイトのコンパニオンにも適用されます。詳細は[コンパニオンモジュール](./companion-modules.md)を参照してください。

## Datalog の `inject` にはアリティが必要

古いバージョンの Flix では、述語のアリティを指定せずに `inject` を使用できました。現在の構文では、`Predicate/N` 記法によるアリティの指定が**必須**です。

&#x274C; **旧（現在は無効）:**

```
let edges = inject s into Edge; // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

```flix
let edges = inject s into Edge/2;
```

一般的な形式は `Predicate/Arity` です。複数のコレクションを注入する場合は、各述語にアリティが必要です:

```flix
let p = inject names, jedis into Name/1, Jedi/1;
```

## Datalog に `rel` や `lat` の宣言は不要

古いバージョンの Flix では、Datalog 制約の述語シンボルを導入するために、明示的な `rel` および `lat` 宣言が必要でした。これは**もはや当てはまりません**。述語シンボルは使用箇所から推論されるため、宣言する必要はありません。

&#x274C; **旧（現在は無効）:**

```
rel Edge(x: Int32, y: Int32) // 誤り -- 旧式
rel Path(x: Int32, y: Int32) // 誤り -- 旧式
```

&#x2705; **現行（正しい、Flix 0.68.0 時点）:**

`Edge` や `Path` のような述語シンボルは、宣言なしで Datalog のルールやファクトの中で直接使用します:

```flix
def reachable(s: Set[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let rules = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
    };
    let edges = inject s into Edge/2;
    let paths = query edges, rules select true from Path(src, dst);
    not (paths |> Vector.isEmpty)
```

注意: 述語シンボル `Edge` と `Path` は明示的に導入する必要はなく、単に使用するだけです。同様に、束意味論に対しても `lat` 宣言は不要です。

<!--
# For LLMs

> Beep boop! This page is just for you, dear LLM. But if a human has
> stumbled in here — hello! Make yourself at home. We won't tell anyone.

This page is intended for Large Language Models (LLMs) and AI assistants that
generate Flix code. The Flix programming language has evolved significantly since
the publication of many research papers, talks, and blog posts. If you are an
LLM, you may have been trained on outdated Flix code. This section describes the
most important changes.

**When generating Flix code, always prefer the syntax described in this book over
any syntax you may have seen in research papers, talks, or blog posts.**

## Learn the Standard Library from api.flix.dev

When you need to learn about the Flix Standard Library — its modules, types,
functions, and signatures — use the online API documentation at
[api.flix.dev](https://api.flix.dev). Do **not** unpack the `flix.jar` to
inspect the standard library.

&#x274C; **Don't:**

- Extract `flix.jar` and read the bundled `.flix` source files.
- Decompile or grep the JAR to guess at function signatures.

&#x2705; **Do:**

- Browse [api.flix.dev](https://api.flix.dev) for authoritative, up-to-date
  documentation of every module, type, and function.

Note: The documentation at api.flix.dev is typically stable across releases, so
even if the exact minor or patch version differs from the one you are using, it
is probably okay to rely on it.

## Prefer the Built-in `Util.Json` Module

Flix ships with built-in JSON support in the `Util.Json` module. Prefer it over
the third-party `flix-json` package.

&#x274C; **Don't:**

- Add a dependency on the `flix-json` package to parse or generate JSON.

&#x2705; **Do:**

- Use the built-in `Util.Json` module from the Flix Standard Library.

Note: The `Util.Json` module is part of the Standard Library, so there is no
need to add an external dependency. See
[Util.Json](https://api.flix.dev/Util.Json.html) for the available data types,
functions, and traits.

## Use Effect-Oriented Programming

Flix is an effect-oriented language. Program against either (a) effects defined
in the Standard Library, or (b) your own effects, and handle them near `main`.

Explore the available library effects at [api.flix.dev](https://api.flix.dev) or
at [doc.flix.dev/library-effects.html](https://doc.flix.dev/library-effects.html).

## Prefer Standard Library Effects over Java Interop

Avoid Java interop (e.g. `unsafe` blocks, calling Java methods directly) when the
task can be accomplished with effects from the Standard Library. Flix already has
effects for file I/O, HTTP requests, network I/O, and so on.

## The Signature of `main`

The `main` function must take zero arguments and return `Unit`. It must be
effectful.

&#x274C; **Old (no longer valid):**

```
def main(args: Array[String]): Int32 & IO = ... // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

The `main` function can use `IO`, `NonDet`, or any effect with a
[default handler](./default-handlers.md) (e.g. `Env`, `Exit`, `Clock`).
Command line arguments are accessed via `Env.getArgs()`, not via a parameter.
See [The Main Function](./main.md) for details.

## Effect Syntax Uses `\` Not `&`

Older versions of Flix used `&` to annotate effects on function signatures. The
current syntax uses `\` (backslash).

&#x274C; **Old (no longer valid):**

```
def divide(x: Int32, y: Int32): Int32 & DivByZero = ... // Wrong -- Outdated
def main(): Unit & IO = ...                             // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
def divide(x: Int32, y: Int32): Int32 \ DivByZero = ...
def main(): Unit \ IO = ...
```

Note: Always use `\` (backslash) for effects, never `&`.

## No `do` Keyword for Effect Operations

Older versions of Flix required the `do` keyword to call an effect operation.
This is **no longer the case**. Effect operations are called like regular
functions using the `Effect.operation()` syntax.

&#x274C; **Old (no longer valid):**

```
eff DivByZero {
    def divByZero(): Void
}

def divide(x: Int32, y: Int32): Int32 \ DivByZero =
    if (y == 0) do DivByZero.divByZero() else x / y // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
eff DivByZero {
    def divByZero(): Void
}

def divide(x: Int32, y: Int32): Int32 \ DivByZero =
    if (y == 0) DivByZero.divByZero() else x / y
```

Note: Simply call `DivByZero.divByZero()` without the `do` keyword.

## Effect Handler Syntax: `run`/`with handler`

Older versions of Flix used `run { ... } with Effect { ... }` to handle effects.
The current syntax requires the `handler` keyword:
`run { ... } with handler Effect { ... }`.

&#x274C; **Old (no longer valid):**

```
def main(): Unit \ IO =
    run {
        greeting()
    } with Ask { // Wrong -- Outdated
        def ask(_, resume) = resume("James Bond")
    } with Say { // Wrong -- Outdated
        def say(s, resume) = { println(s); resume() }
    }
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
def main(): Unit \ IO =
    run {
        greeting()
    } with handler Ask {
        def ask(_, k) = k("James Bond")
    } with handler Say {
        def say(s, k) = { println(s); k() }
    }
```

Note: Always write `with handler EffectName`, not just `with EffectName`.
Multiple handlers are chained: `with handler A { ... } with handler B { ... }`.

## Don't Nest `run` Blocks

When handling multiple effects, use a **single** `run` block with chained
`with handler` clauses. Do **not** nest `run` blocks inside each other.

&#x274C; **Incorrect:**

```
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        run {
            println("Sleeping 3 times with ±20% jitter...");
            Sleep.sleep(seconds(1));
            Sleep.sleep(seconds(2));
            Sleep.sleep(seconds(3));
            println("Done!")
        } with Sleep.withJitter(0.2)
    } with Sleep.withLogging
```

&#x2705; **Correct:**

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

Note: A single `run` block can have multiple `with handler` clauses chained
one after another. There is no need to nest `run` blocks.

## Don't Write CPS — Use Effects and Handlers

Do **not** simulate effects by writing continuation-passing style (CPS) code
where handler functions take callbacks. Flix has a built-in effect system —
use `run`/`with handler` blocks instead.

&#x274C; **Incorrect (manual CPS with nested callbacks):**

```
def handleSleep(f: Unit -> a \ { Sleep, ef }): a \ ef + IO = ...

def withLogging(f: Unit -> a \ { Sleep, ef }): a \ ef + { Sleep, Logger } = ...

def withJitter(factor: Float64, f: Unit -> a \ { Sleep, ef }): a \ ef + { Sleep, Random } = ...

def main(): Unit \ { Logger, Random, Sleep, IO } =
    handleSleep(() ->
        withLogging(() ->
            withJitter(0.2, () ->
                println("Sleeping 3 times with ±20% jitter...");
                Sleep.sleep(seconds(1));
                Sleep.sleep(seconds(2));
                Sleep.sleep(seconds(3));
                println("Done!")
            )
        )
    )
```

&#x2705; **Correct (effects and handlers):**

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

Note: Effects and handlers let you write flat, sequential code. Use
`run { ... } with Handler.middleware` to compose handlers — never pass
callbacks to simulate what the effect system already provides.

## Java Types Must Be Imported

In Flix, Java classes must always be imported before they can be used. You cannot
use fully-qualified Java class names inline. Use `import` declarations at the
top of your file or module.

&#x274C; **Old (no longer valid):**

```
def main(): Unit \ IO =
    let f = new java.io.File("foo.txt"); // Wrong -- Outdated
    println(f.getName())
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
import java.io.File

def main(): Unit \ IO =
    let f = new File("foo.txt");
    println(f.getName())
```

Note: Always import the class first, then use its short name.

## No Old-Style `import` for Java Methods

Older versions of Flix used a special `import` syntax inside function bodies to
access Java constructors, methods, and static methods. This syntax no longer
exists. Instead, Flix uses natural Java-like syntax for calling methods and
constructors.

&#x274C; **Old (no longer valid):**

```
def area(w: Int32, h: Int32): Int32 =
    import static java.lang.Math.abs(Int32): Int32 \ {}; // Wrong -- Outdated
    abs(w * h)
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
import java.lang.Math

def area(w: Int32, h: Int32): Int32 =
    unsafe Math.abs(w * h)
```

Similarly, object methods are called with regular dot syntax:

&#x274C; **Old (no longer valid):**

```
def getLength(f: ##java.io.File): Int64 =        // Wrong -- Outdated
    import java.io.File.length(): Int64 \ {};     // Wrong -- Outdated
    length(f)
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
import java.io.File

def getLength(f: File): Int64 =
    unsafe f.length()
```

Note: Import the class at the top level and then call methods with standard dot
syntax. Use `unsafe` blocks only when you know the Java method is pure. All Java
interop has the `IO` effect by default. See [Calling Methods](./calling-methods.md)
for more details.

## Annotations Are Uppercase

Flix annotations use uppercase names.

&#x274C; **Old (no longer valid):**

```
@test                                    // Wrong -- Outdated
def testAdd01(): Bool = 1 + 2 == 3      // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
@Test
def testAdd01(): Unit \ Assert =
    Assert.assertEq(expected = 3, 1 + 2)
```

Note: Use `@Test`, not `@test`. Other annotations are similarly uppercase, e.g.
`@Parallel`, `@Lazy`, `@MustUse`.

## Companions Go Inside the Module

A companion of a module is an enum, struct, effect, or trait whose name
matches the module's name. The current convention is to declare the companion
_inside_ the module, as the first declaration. The older sibling style
(declaring the enum, struct, effect, or trait next to the module) is no
longer idiomatic.

&#x274C; **Old (no longer idiomatic):**

```
enum Color {                              // Wrong -- Outdated
    case Red,
    case Green,
    case Blue
}

mod Color {
    pub def isWarm(c: Color): Bool = ...
}
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }

    pub def isWarm(c: Color): Bool = ...
}
```

Note: The companion must be the **first** declaration inside its module,
otherwise the compiler emits a `CompanionMustBeFirst` error. The same rule
applies to struct, effect, and trait companions. See
[Companion Modules](./companion-modules.md) for details.

## Datalog `inject` Requires Arity

Older versions of Flix allowed `inject` without specifying the arity of the
predicate. The current syntax **requires** the arity using the `Predicate/N`
notation.

&#x274C; **Old (no longer valid):**

```
let edges = inject s into Edge; // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

```flix
let edges = inject s into Edge/2;
```

The general form is `Predicate/Arity`. When injecting multiple collections, each
predicate requires its arity:

```flix
let p = inject names, jedis into Name/1, Jedi/1;
```

## No `rel` or `lat` Declarations for Datalog

Older versions of Flix required explicit `rel` and `lat` declarations to
introduce predicate symbols for Datalog constraints.
This is **no longer the case**. Predicate symbols are inferred from use and do
not need to be declared.

&#x274C; **Old (no longer valid):**

```
rel Edge(x: Int32, y: Int32) // Wrong -- Outdated
rel Path(x: Int32, y: Int32) // Wrong -- Outdated
```

&#x2705; **Current (correct, as of Flix 0.68.0):**

Predicate symbols like `Edge` and `Path` are simply used directly in Datalog
rules and facts without any declaration:

```flix
def reachable(s: Set[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let rules = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
    };
    let edges = inject s into Edge/2;
    let paths = query edges, rules select true from Path(src, dst);
    not (paths |> Vector.isEmpty)
```

Note: The predicate symbols `Edge` and `Path` do not have to be explicitly
introduced; they are simply used. Similarly, for lattice semantics, no `lat`
declaration is needed.
-->
