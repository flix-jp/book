# 標準出力への出力

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/printing-to-stdout.html)を参照してください。

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
    def readln(): String
    def print(s: String): Unit
    def eprint(s: String): Unit
    def println(s: String): Unit
    def eprintln(s: String): Unit
}
```

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
    def readln(): String
    def print(s: String): Unit
    def eprint(s: String): Unit
    def println(s: String): Unit
    def eprintln(s: String): Unit
}
```
-->
