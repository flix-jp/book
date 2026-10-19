# 標準出力への出力

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/printing-to-stdout.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/printing-to-stdout.md)ください。

Flix の Prelude(プレリュード)には、標準出力へ出力する `println` 関数が定義されています。例えば：

```flix
println("Hello World")
```

`println` 関数は、型が `ToString` トレイトを実装している値、すなわち `String` に変換できる値であれば、どのような値でも出力できます。例えば：

```flix
let o = Some(123);
let l = 1 :: 2 :: 3 :: Nil;
println(o);
println(l)
```

`println` 関数は当然ながらエフェクトを持つ関数であるため、純粋関数から呼び出すことはできません。純粋関数をデバッグするには、組み込みの[デバッグ機能](./debugging.md)を使用してください。

## Console エフェクト

`Console` エフェクトは、ターミナルからの読み取りとターミナルへの書き込みを行う操作を定義しています：

```flix
eff Console {
    def readln(): Option[String]
    def write(s: String): Unit
    def writeLine(s: String): Unit
    def writeErr(s: String): Unit
    def writeErrLine(s: String): Unit
}
```

これらの操作はプレーンな `String` を対象としています。`Console` エフェクトは `Sys.Console` モジュールの[コンパニオン](./companion-modules.md)であり、このモジュールは上記の操作を基にした4つの出力用関数を定義しています：

```flix
mod Sys.Console {
    def print(x: a): Unit \ Console with ToString[a]
    def println(x: a): Unit \ Console with ToString[a]
    def eprint(x: a): Unit \ Console with ToString[a]
    def eprintln(x: a): Unit \ Console with ToString[a]
}
```

Prelude の `println` と同様に、これらの関数も `ToString` のインスタンスを持つあらゆる値を受け取りますが、`IO` ではなく `Console` エフェクトを持ちます。`eprint` と `eprintln` は標準エラー出力へ書き込みます。例えば：

```flix
use Sys.Console

def main(): Unit \ Console =
    let o = Some(123);
    let l = 1 :: 2 :: 3 :: Nil;
    Console.println(o);
    Console.println(l);
    Console.eprintln("Printed two values.")
```

入力の読み取りやプロンプトについては、[Console](./console.md) の章を参照してください。

<!--
# Printing to Standard Out

The Flix Prelude defines the `println` function which prints to standard out.
For example:

```flix
println("Hello World")
```

The `println` function can print any value whose type implements the `ToString`
trait and consequently can be converted to a `String`. For example:

```flix
let o = Some(123);
let l = 1 :: 2 :: 3 :: Nil;
println(o);
println(l)
```

The `println` function is rightfully effectful, hence it cannot be called from a
pure function. To debug a pure function, use the builtin [debugging
facilities](./debugging.md).

## The Console Effect

The `Console` effect defines operations for reading from and writing to the
terminal:

```flix
eff Console {
    def readln(): Option[String]
    def write(s: String): Unit
    def writeLine(s: String): Unit
    def writeErr(s: String): Unit
    def writeErrLine(s: String): Unit
}
```

The operations work on plain `String`s. The `Console` effect is the
[companion](./companion-modules.md) of the `Sys.Console` module, which defines
four printing functions on top of the operations:

```flix
mod Sys.Console {
    def print(x: a): Unit \ Console with ToString[a]
    def println(x: a): Unit \ Console with ToString[a]
    def eprint(x: a): Unit \ Console with ToString[a]
    def eprintln(x: a): Unit \ Console with ToString[a]
}
```

Like the `println` of the Prelude, these accept any value with a `ToString`
instance, but they have the `Console` effect rather than `IO`. The `eprint` and
`eprintln` functions write to standard error. For example:

```flix
use Sys.Console

def main(): Unit \ Console =
    let o = Some(123);
    let l = 1 :: 2 :: 3 :: Nil;
    Console.println(o);
    Console.println(l);
    Console.eprintln("Printed two values.")
```

See the [Console](./console.md) chapter for reading input and for prompts.
-->
