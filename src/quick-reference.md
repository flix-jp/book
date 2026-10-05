# クイックリファレンス

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/quick-reference.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/quick-reference.md)ください。

Flix 言語の1ページチートシートです。各節は、詳細が書かれている対応する章へのリンクを含んでいます。

## プログラムの構造

```flix
/// ドキュメントコメント。通常のコメントは // で始まります。
def main(): Unit \ IO =
    println("Hello, world!")
```

式は `;` で区切ります。`def` は関数を、`let` はローカル変数を定義します
（[関数](./functions.md)を参照）：

```flix
def area(w: Int32, h: Int32): Int32 = w * h

def greeting(name: String): String \ IO =
    let msg = "Hello, ${name}!";
    println(msg);
    msg
```

無名関数には `->` を使います：`x -> x + 1`。パイプラインには `|>` を使います：
`l |> List.length |> println`。

## プリミティブ型

`Unit`、`Bool`、`Char`、`String`、`Int8`、`Int16`、`Int32`、`Int64`、
`BigInt`、`Float32`、`Float64`、`BigDecimal`
（[プリミティブ型](./primitive-types.md)を参照）。

よく使われる演算子：`+ - * / %`、`== != < <= > >=`、`and or not`、
`::`（リストのコンス）、`:::`（リストの連結）。

## 分岐

```flix
if (x < 0) {
    "negative"
} else if (x == 0) {
    "zero"
} else {
    "positive"
}
```

[If-Then-Else](./if-then-else.md) を参照してください。

## 列挙型とパターンマッチング

```flix
enum Shape {
    case Circle(Int32)
    case Rectangle(Int32, Int32)
}

def area(s: Shape): Int32 = match s {
    case Shape.Circle(r)       => 3 * (r * r)
    case Shape.Rectangle(h, w) => h * w
}
```

列挙型はパラメトリックにできます：`enum Tree[a] { case Leaf(a), case Node(Tree[a], Tree[a]) }`。
ワイルドカードには `case _ => ...` を使います。タプルも同様に分解できます
（[列挙型](./enums.md)と[パターンマッチング](./pattern-matching.md)を参照）：

```flix
match pair {
    case (1, _) => "starts with one"
    case _      => "something else"
}
```

## リスト

```flix
let l1 = 1 :: 2 :: 3 :: Nil;
let l2 = l1 ::: 4 :: Nil;
match l1 {
    case Nil     => 0
    case _ :: xs => 1 + List.length(xs)
}
```

[リスト](./lists.md)を参照してください。

## 文字列

```flix
let name = "Ada";
let msg = "Hello, ${name}!";  // 文字列補間
```

[文字列補間](./string-interpolation.md)を参照してください。

## レコード

```flix
let r = {name = "Ada", age = 36};
r#name                            // フィールドへのアクセス
{name = "Grace" | r}              // フィールドの更新
{+lang = "Flix" | r}              // フィールドの追加
{-age | r}                        // フィールドの削除
```

[レコード](./records.md)を参照してください。

## モジュール

```flix
mod Calc {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
}

use Calc.sum;
use Calc.{sum};
```

[モジュール](./modules.md)を参照してください。

## トレイトとインスタンス

```flix
trait Show[t] {
    pub def show(x: t): String
}

instance Show[Int32] {
    pub def show(x: Int32): String = "${x}"
}
```

[トレイト](./traits.md)を参照してください。

## エフェクト

純粋関数は空のエフェクト `\ {}` を持ちます。不純な関数は、使用するものを
宣言します。例えば `\ IO` です。エフェクト多相な関数は、変数を使ってエフェクトを
抽象化します（[エフェクトシステム](./effect-system.md)を参照）：

```flix
def inc(x: Int32): Int32 \ {} = x + 1
def twice(f: Int32 -> Int32 \ ef, x: Int32): Int32 \ ef = f(f(x))
```

## Option と Result

```flix
match opt {
    case Some(x) => x
    case None    => 0
}

match res {
    case Ok(v)  => v
    case Err(e) => 0
}
```

## ループ

```flix
def main(): Unit \ IO =
    let l = 1 :: 2 :: 3 :: Nil;
    foreach (x <- l)
        println(x)
```

[Foreach](./foreach.md) を参照してください。

<!--
# Quick Reference

A one-page cheat sheet for the Flix language. Each section links to the
relevant chapter for full details.

## Program structure

```flix
/// Doc comment. Ordinary comments start with //.
def main(): Unit \ IO =
    println("Hello, world!")
```

Expressions are separated with `;`. `def` defines a function, `let` a local
(see [Functions](./functions.md)):

```flix
def area(w: Int32, h: Int32): Int32 = w * h

def greeting(name: String): String \ IO =
    let msg = "Hello, ${name}!";
    println(msg);
    msg
```

Anonymous functions use `->`: `x -> x + 1`. Pipelines use `|>`: `l |> List.length |> println`.

## Primitive types

`Unit`, `Bool`, `Char`, `String`, `Int8`, `Int16`, `Int32`, `Int64`,
`BigInt`, `Float32`, `Float64`, `BigDecimal`
(see [Primitives](./primitive-types.md)).

Common operators: `+ - * / %`, `== != < <= > >=`, `and or not`,
`::` (list cons), `:::` (list append).

## Branching

```flix
if (x < 0) {
    "negative"
} else if (x == 0) {
    "zero"
} else {
    "positive"
}
```

See [If-Then-Else](./if-then-else.md).

## Enums and pattern matching

```flix
enum Shape {
    case Circle(Int32)
    case Rectangle(Int32, Int32)
}

def area(s: Shape): Int32 = match s {
    case Shape.Circle(r)       => 3 * (r * r)
    case Shape.Rectangle(h, w) => h * w
}
```

Enums can be parametric: `enum Tree[a] { case Leaf(a), case Node(Tree[a], Tree[a]) }`.
Use `case _ => ...` as a wildcard. Tuples destructure the same way
(see [Enums](./enums.md) and [Pattern Matching](./pattern-matching.md)):

```flix
match pair {
    case (1, _) => "starts with one"
    case _      => "something else"
}
```

## Lists

```flix
let l1 = 1 :: 2 :: 3 :: Nil;
let l2 = l1 ::: 4 :: Nil;
match l1 {
    case Nil     => 0
    case _ :: xs => 1 + List.length(xs)
}
```

See [Lists](./lists.md).

## Strings

```flix
let name = "Ada";
let msg = "Hello, ${name}!";  // string interpolation
```

See [String Interpolation](./string-interpolation.md).

## Records

```flix
let r = {name = "Ada", age = 36};
r#name                            // field access
{name = "Grace" | r}              // update a field
{+lang = "Flix" | r}              // add a field
{-age | r}                        // drop a field
```

See [Records](./records.md).

## Modules

```flix
mod Calc {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
}

use Calc.sum;
use Calc.{sum};
```

See [Modules](./modules.md).

## Traits and instances

```flix
trait Show[t] {
    pub def show(x: t): String
}

instance Show[Int32] {
    pub def show(x: Int32): String = "${x}"
}
```

See [Traits](./traits.md).

## Effects

Pure functions carry the empty effect `\ {}`; impure ones declare what they
use, e.g. `\ IO`. Effect-polymorphic functions abstract over it with a
variable (see [Effect System](./effect-system.md)):

```flix
def inc(x: Int32): Int32 \ {} = x + 1
def twice(f: Int32 -> Int32 \ ef, x: Int32): Int32 \ ef = f(f(x))
```

## Option and Result

```flix
match opt {
    case Some(x) => x
    case None    => 0
}

match res {
    case Ok(v)  => v
    case Err(e) => 0
}
```

## Loops

```flix
def main(): Unit \ IO =
    let l = 1 :: 2 :: 3 :: Nil;
    foreach (x <- l)
        println(x)
```

See [Foreach](./foreach.md).
-->
