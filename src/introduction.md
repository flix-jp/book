# Flix へようこそ

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/introduction.html)を参照してください。

Flix は、[オーフス大学](https://cs.au.dk/)で開発され、[オープンソースコントリビューター](https://github.com/flix/flix)のコミュニティによって、[ウォータールー大学](https://uwaterloo.ca/)、[テュービンゲン大学](https://uni-tuebingen.de/)、[コペンハーゲン大学](https://di.ku.dk/)の研究者と協力して開発されている、原則に基づいた関数型・論理型・命令型プログラミング言語です。

Flix は OCaml と Haskell に触発されており、Rust と Scala からのアイデアも取り入れています。Flix は Scala に似た見た目ですが、その型システムは完全な型推論をサポートする Hindley-Milner に基づいています。Flix は以下のような複数の革新的な機能を備えた*最先端*のプログラミング言語です：

- 完全な型推論を備えた多相型およびエフェクトシステム
- リージョンベースのローカルミュータブルメモリ
- ユーザー定義のエフェクトとハンドラ
- 関連型と関連エフェクトを持つ高カインドトレイト
- 組み込みのファーストクラス Datalog プログラミング

Flix は効率的な JVM バイトコードにコンパイルされ、Java 仮想マシン上で動作し、完全な末尾呼び出し最適化をサポートしています。Flix は Java との相互運用性を持ち、JVM のクラスやメソッドを使用できます。そのため、Java エコシステム全体が Flix から利用可能です。

Flix は世界クラスの Visual Studio Code サポートを目指しています。[Flix Visual Studio Code 拡張機能](./vscode.md)は実際の Flix コンパイラを使用するため、Flix 言語とエディタで報告される内容の間に常に 1:1 の対応があります。利点は多数あります：(a) 診断は常に正確、(b) コードナビゲーションが「そのまま動く」、(c) リファクタリングは常に正しい。

## ルック・アンド・フィール

Flix の見た目と使用感を示すいくつかのプログラムを紹介します：

このプログラムは**代数的データ型とパターンマッチング**の使用を示しています：

```flix
/// 図形のための代数的データ型
enum Shape {
    case Circle(Int32),          // 円の半径
    case Square(Int32),          // 辺の長さ
    case Rectangle(Int32, Int32) // 高さと幅
}

/// パターンマッチングと基本的な算術演算を使用して
/// 与えられた図形の面積を計算する
def area(s: Shape): Int32 = match s {
    case Shape.Circle(r)       => 3 * (r * r)
    case Shape.Square(w)       => w * w
    case Shape.Rectangle(h, w) => h * w
}

// 2 x 4 の面積を計算する
def main(): Unit \ IO =
    area(Shape.Rectangle(2, 4)) |> println
```

以下は**多相レコード**を使用した例です：

```flix
/// 多相レコード `r` の面積を返す
/// 型変数 `a` を使用することで、レコード `r` が
/// `x` と `y` 以外のラベルを持つことができる
def polyArea[a : RecordRow](r: {x = Int32, y = Int32 | a}): Int32 = r#x * r#y

/// 様々な長方形レコードの面積を計算する
/// 一部のレコードには追加のラベルがある
def polyAreas(): List[Int32] =
    polyArea({x = 1, y = 2}) ::
    polyArea({x = 2, y = 3, z = 4}) :: Nil

def main(): Unit \ IO =
    polyAreas() |> println
```

以下は**リージョンベースのローカルミューテーション**を使用した例です：

```flix
///
/// リージョンを使用して、内部的なミュータビリティ（不純性）を
/// 使用する純粋関数を定義できる
/// リージョンはミュータビリティを宣言されたスコープにカプセル化する
///
def deduplicate(l: List[a]): List[a] with Order[a] =
    /// 新しいリージョン `rc` を宣言する
    region rc {

        /// リージョン `r` に新しい `MutSet` を作成する
        /// これは `l` 内のユニークな要素を追跡するために使用される
        let s = MutSet.empty(rc);

        /// `filter` の呼び出しで使用されるラムダは
        /// リージョンがなければ不純になる
        List.filter(x -> {
            if (MutSet.memberOf(x, s))
                false // `x` は既に出現済み
            else {
                MutSet.add(x, s);
                true
            }
        }, l)
    }
```

以下は組み込みの**エフェクトとハンドラ**を使用した例です：

```flix
use Net.Http
use Net.Retry
use Net.HttpResponse
use Time.Clock
use Time.Duration.{milliseconds, seconds}

def main(): Unit \ { Clock, Http, Logger, IO } =
    let defaultHeaders = Map#{
        "Accept"        => List#{"application/json"},
        "Authorization" => List#{"Bearer tok123"}
    };
    run {
        let urls = List#{"/api/users", "/api/posts"};
        foreach (url <- urls) {
            match Http.get(url) {
                case Ok(resp) => println("${url} -> ${HttpResponse.status(resp)}")
                case Err(err) => println("${url} -> ${err}")
            }
        };
        match Http.get("https://notfound.flix.dev/") {
            case Ok(resp) => println("notfound -> ${HttpResponse.status(resp)}")
            case Err(err) => println("notfound -> ${err}")
        }
    } with Http.withBaseUrl("https://flix.dev")
      with Http.withDefaultHeaders(defaultHeaders)
      with Http.withRetry(Retry.linear(maxRetries = 2, delay = milliseconds(100)))
      with Http.withCircuitBreaker(failureThreshold = 3, cooldown = seconds(5))
      with Http.withSlidingWindow(maxRequests = 2, window = seconds(1))
      with Http.withLogging
```

以下は**ユーザー定義のエフェクトとハンドラ**の例です：

```flix
eff MyPrint {
    def println(s: String): Unit
}

eff MyTime {
    def getCurrentHour(): Int32
}

def sayGreeting(name: String): Unit \ {MyPrint, MyTime} = {
    let hour = MyTime.getCurrentHour();
    if (hour < 12)
        MyPrint.println("Good morning, ${name}")
    else
        MyPrint.println("Good afternoon, ${name}")
}

def main(): Unit \ IO =
    run {
        (sayGreeting("Mr. Bond, James Bond"): Unit)
    } with handler MyPrint {
        def println(s, k) = { println(s); k() }
    } with handler MyTime {
        def getCurrentHour(_, k) = k(11)
    }
```

以下は**ファーストクラス Datalog 制約**を使用した例です：

```flix
def reachable(edges: List[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let db = inject edges into Edge/2;
    let pr = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
        Reachable() :- Path(src, dst).
    };
    let result = query db, pr select () from Reachable();
    not Vector.isEmpty(result)
```

最後に、**チャネルとプロセスを使用した構造化並行性**の例です：

```flix
/// リストのすべての要素を送信する関数
def sendAll(l: List[Int32], tx: Sender[Int32]): Unit \ Chan =
    match l {
        case Nil     => ()
        case x :: xs => Channel.send(x, tx); sendAll(xs, tx)
    }

/// n 個の要素を受信してリストに集める関数
def recvN(n: Int32, rx: Receiver[Int32]): List[Int32] \ {Chan, NonDet} =
    match n {
        case 0 => Nil
        case _ => Channel.recv(rx) :: recvN(n - 1, rx)
    }

/// receive を呼び出し、結果を tx に送信する関数
def wait(rx: Receiver[Int32], n: Int32, tx: Sender[List[Int32]]): Unit \ {Chan, NonDet} =
    Channel.send(recvN(n, rx), tx)

/// send と wait のプロセスをスポーンし、結果を表示する
def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let l = 1 :: 2 :: 3 :: Nil;
    let (tx1, rx1) = Channel.buffered(100);
    let (tx2, rx2) = Channel.buffered(100);
    spawn sendAll(l, tx1) @ rc;
    spawn wait(rx1, List.length(l), tx2) @ rc;
    println(Channel.recv(rx2))
}
```

その他の例は、本書の各ページや [GitHub の examples フォルダ](https://github.com/flix/flix/tree/master/examples)で確認できます。

<!--
# Introduction to Flix

Flix is a principled functional, logic, and imperative programming language
developed at [Aarhus University](https://cs.au.dk/) and by a community of [open
source contributors](https://github.com/flix/flix) in collaboration with
researchers from the [University of Waterloo](https://uwaterloo.ca/), from the
[University of Tubingen](https://uni-tuebingen.de/), and from the [University of
Copenhagen](https://di.ku.dk/).

Flix is inspired by OCaml and Haskell with ideas from Rust and Scala. Flix looks
like Scala, but its type system is based on Hindley-Milner which supports
complete type inference. Flix is a *state-of-the-art* programming language with
multiple innovative features, including:

- a polymorphic type and effect system with full type inference.
- region-based local mutable memory.
- user-defined effects and handlers.
- higher-kinded traits with associated types and effects.
- embedded first-class Datalog programming.

Flix compiles to efficient JVM bytecode, runs on the Java Virtual Machine, and
supports full tail call elimination. Flix has interoperability with Java and can
use JVM classes and methods. Hence the entire Java ecosystem is available from
within Flix.

Flix aims to have world-class Visual Studio Code support. The [Flix Visual
Studio Code extension](./vscode.md) uses the real Flix compiler hence there is
always a 1:1 correspondence between the Flix language and what is reported in
the editor. The advantages are many: (a) diagnostics are always exact, (b) code
navigation "just works", and (c) refactorings are always correct.

## Look and Feel

Here are a few programs to illustrate the look and feel of Flix:

This program illustrates the use of **algebraic data types and pattern matching**:

```flix
/// An algebraic data type for shapes.
enum Shape {
    case Circle(Int32),          // circle radius
    case Square(Int32),          // side length
    case Rectangle(Int32, Int32) // height and width
}

/// Computes the area of the given shape using
/// pattern matching and basic arithmetic.
def area(s: Shape): Int32 = match s {
    case Shape.Circle(r)       => 3 * (r * r)
    case Shape.Square(w)       => w * w
    case Shape.Rectangle(h, w) => h * w
}

// Computes the area of a 2 by 4.
def main(): Unit \ IO =
    area(Shape.Rectangle(2, 4)) |> println
```

Here is an example that uses **polymorphic records**:

```flix
/// Returns the area of the polymorphic record `r`.
/// Note that the use of the type variable `a` permits the record `r`
/// to have labels other than `x` and `y`.
def polyArea[a : RecordRow](r: {x = Int32, y = Int32 | a}): Int32 = r#x * r#y

/// Computes the area of various rectangle records.
/// Note that some records have additional labels.
def polyAreas(): List[Int32] =
    polyArea({x = 1, y = 2}) ::
    polyArea({x = 2, y = 3, z = 4}) :: Nil

def main(): Unit \ IO =
    polyAreas() |> println
```

Here is an example that uses **region-based local mutation**:

```flix
///
/// We can define pure functions that use
/// internal mutability (impurity) with regions.
/// Regions encapsulate mutability to its declared scope.
///
def deduplicate(l: List[a]): List[a] with Order[a] =
    /// Declare a new region `rc`.
    region rc {

        /// Create a new `MutSet` at region `r`.
        /// This will be used to keep track of
        /// unique elements in `l`.
        let s = MutSet.empty(rc);

        /// The lambda used in the call to `filter`
        /// would be impure without a region.
        List.filter(x -> {
            if (MutSet.memberOf(x, s))
                false // `x` has already been seen.
            else {
                MutSet.add(x, s);
                true
            }
        }, l)
    }
```

Here is an example that uses built-in **effects and handlers**:

```flix
use Net.Http
use Net.Retry
use Net.HttpResponse
use Time.Clock
use Time.Duration.{milliseconds, seconds}

def main(): Unit \ { Clock, Http, Logger, IO } =
    let defaultHeaders = Map#{
        "Accept"        => List#{"application/json"},
        "Authorization" => List#{"Bearer tok123"}
    };
    run {
        let urls = List#{"/api/users", "/api/posts"};
        foreach (url <- urls) {
            match Http.get(url) {
                case Ok(resp) => println("${url} -> ${HttpResponse.status(resp)}")
                case Err(err) => println("${url} -> ${err}")
            }
        };
        match Http.get("https://notfound.flix.dev/") {
            case Ok(resp) => println("notfound -> ${HttpResponse.status(resp)}")
            case Err(err) => println("notfound -> ${err}")
        }
    } with Http.withBaseUrl("https://flix.dev")
      with Http.withDefaultHeaders(defaultHeaders)
      with Http.withRetry(Retry.linear(maxRetries = 2, delay = milliseconds(100)))
      with Http.withCircuitBreaker(failureThreshold = 3, cooldown = seconds(5))
      with Http.withSlidingWindow(maxRequests = 2, window = seconds(1))
      with Http.withLogging
```

Here is an example that **defines its own effects and handlers**:

```flix
eff MyPrint {
    def println(s: String): Unit
}

eff MyTime {
    def getCurrentHour(): Int32
}

def sayGreeting(name: String): Unit \ {MyPrint, MyTime} = {
    let hour = MyTime.getCurrentHour();
    if (hour < 12)
        MyPrint.println("Good morning, ${name}")
    else
        MyPrint.println("Good afternoon, ${name}")
}

def main(): Unit \ IO =
    run {
        (sayGreeting("Mr. Bond, James Bond"): Unit)
    } with handler MyPrint {
        def println(s, k) = { println(s); k() }
    } with handler MyTime {
        def getCurrentHour(_, k) = k(11)
    }
```

Here is an example that uses **first-class Datalog constraints**:

```flix
def reachable(edges: List[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let db = inject edges into Edge/2;
    let pr = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
        Reachable() :- Path(src, dst).
    };
    let result = query db, pr select () from Reachable();
    not Vector.isEmpty(result)
```

And finally here is an example that uses **structured concurrency with channels
and processes**:

```flix
/// A function that sends every element of a list
def sendAll(l: List[Int32], tx: Sender[Int32]): Unit \ Chan =
    match l {
        case Nil     => ()
        case x :: xs => Channel.send(x, tx); sendAll(xs, tx)
    }

/// A function that receives n elements
/// and collects them into a list.
def recvN(n: Int32, rx: Receiver[Int32]): List[Int32] \ {Chan, NonDet} =
    match n {
        case 0 => Nil
        case _ => Channel.recv(rx) :: recvN(n - 1, rx)
    }

/// A function that calls receive and sends the result on d.
def wait(rx: Receiver[Int32], n: Int32, tx: Sender[List[Int32]]): Unit \ {Chan, NonDet} =
    Channel.send(recvN(n, rx), tx)

/// Spawn a process for send and wait, and print the result.
def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let l = 1 :: 2 :: 3 :: Nil;
    let (tx1, rx1) = Channel.buffered(100);
    let (tx2, rx2) = Channel.buffered(100);
    spawn sendAll(l, tx1) @ rc;
    spawn wait(rx1, List.length(l), tx2) @ rc;
    println(Channel.recv(rx2))
}
```

Additional examples can be found in these pages and in the [examples folder on
GitHub](https://github.com/flix/flix/tree/master/examples).
-->
