# 冗長性

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/redundancy.html)を参照してください。

Flix コンパイラは、未使用の要素を含むプログラムを積極的に拒否します。これは、プログラマーが分かりにくいバグを避けられるように支援するためのものです[^1]。慣れるまで少し時間がかかるかもしれませんが、それに見合う価値のあるトレードオフだと私たちは考えています。

具体的には、Flix コンパイラはプログラムが次のものを含まないことを保証します：

- [未使用のローカル変数](#未使用のローカル変数unused-local-variables)：宣言されているものの、一度も使用されないローカル変数。
- [シャドーイングされたローカル変数](#シャドーイングされたローカル変数shadowed-local-variables)：他のローカル変数をシャドーイングするローカル変数。
- [無意味な式](#無意味な式useless-expressions)：値が破棄される純粋な式。
- [使用必須の値](#使用必須の値must-use-values)：値が未使用であるものの、その型が `@MustUse` としてマークされている式。

## 未使用のローカル変数（Unused Local Variables）

Flix は未使用の変数を含むプログラムを拒否します。

例えば、次のプログラムは拒否されます：

```flix
def main(): Unit \ IO =
    let x = 123;
    let y = 456;
    println("The sum is ${x + x}")
```

エラーメッセージは次のとおりです：

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Unused local variable 'y'. The variable is not referenced within its scope.

3 |     let y = 456;
            ^
            unused local variable.
```

未使用のローカル変数は、アンダースコア `_` を接頭辞として付けることで、このエラーを抑制できます。例えば、`y` を `_y` に置き換えると、上記のプログラムはコンパイルできます：

```flix
def main(): Unit \ IO =
    let x = 123;
    let _y = 456; // OK
    println("The sum is ${x + x}")
```

## シャドーイングされたローカル変数（Shadowed Local Variables）

Flix はシャドーイング(Shadowing)された変数を含むプログラムを拒否します。

例えば、次のプログラムは拒否されます：

```flix
def main(): Unit \ IO =
    let x = 123;
    let x = 456;
    println("The value of x is ${x}.")
```

エラーメッセージは次のとおりです：

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Shadowed variable 'x'.

3 |     let x = 456;
            ^
            shadowing variable.

The shadowed variable was declared here:

2 |     let x = 123;
            ^
            shadowed variable.
```



## 無意味な式（Useless Expressions）

Flix は、結果が破棄される*純粋な*式を含むプログラムを拒否します。

例えば、次のプログラムは拒否されます：

```flix
def main(): Unit \ IO =
    123 + 456;
    println("Hello World!")
```

エラーメッセージは次のとおりです：

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Useless expression: It has no side-effect(s) and its result is discarded.

2 |     123 + 456;
        ^^^^^^^^^
        useless expression.

The expression has type 'Int32'
```

副作用を持たず、かつ結果も使用されない式は疑わしいものです。なぜなら、その式はプログラムの意味を変えることなく、そのまま削除できてしまうからです。

## 使用必須の値（Must Use Values）

Flix は、値が破棄されるにもかかわらず、その型に `@MustUse` アノテーションが付けられている式を含むプログラムを拒否します。関数型、および Flix 標準ライブラリの `Result` 型と `Validation` 型は `@MustUse` としてマークされています。

例えば、次のプログラムは拒否されます：

```flix
def main(): Unit \ IO =
    File.creationTime("foo.txt");
    println("Hello World!")
```

エラーメッセージは次のとおりです：

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Unused value but its type is marked as @MustUse.

2 |     File.creationTime("foo.txt");
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        unused value.

The expression has type 'Result[String, Int64]'
```

`File.creationTime` は副作用を持ちますが、少なくとも操作が成功したことを確認するためには、結果である `Result[String, Int64]` を使用するべきでしょう。

非純粋な式の結果が本当に不要な場合は、`discard` 式を使用できます：

```flix
def main(): Unit \ IO =
    discard File.creationTime("foo.txt");
    println("Hello World!")
```

これにより、式が非純粋である限り、`@MustUse` の値を破棄することが許可されます。

[^1] 例えば [Using Redundancies to Find Errors](https://dl.acm.org/doi/abs/10.1145/605466.605475) を参照してください。

<!--
# Redundancy

The Flix compiler aggressively rejects programs that contain unused elements.
The idea is to help programmers avoid subtle bugs[^1]. While this can take some
time getting used to, we believe the trade-off is worth it.

Specifically, the Flix compiler will ensure that a program does not have:

- [Unused local variables](#unused-local-variables): local variables that are declared, but never used.
- [Shadowed local variables](#shadowed-local-variables): local variables that shadow other local variables.
- [Useless expressions](#useless-expressions): pure expressions whose values are discarded.
- [Must use values](#must-use-values): expressions whose values are unused but their type is marked as `@MustUse`.

## Unused Local Variables

Flix rejects programs with unused variables.

For example, the following program is rejected:

```flix
def main(): Unit \ IO =
    let x = 123;
    let y = 456;
    println("The sum is ${x + x}")
```

with the message:

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Unused local variable 'y'. The variable is not referenced within its scope.

3 |     let y = 456;
            ^
            unused local variable.
```

Unused local variables can be prefixed by an underscore `_` to suppress the error.
For example, if we replace `y` by `_y` the above program compiles:

```flix
def main(): Unit \ IO =
    let x = 123;
    let _y = 456; // OK
    println("The sum is ${x + x}")
```

## Shadowed Local Variables

Flix rejects programs with shadowed variables.

For example, the following program is rejected:

```flix
def main(): Unit \ IO =
    let x = 123;
    let x = 456;
    println("The value of x is ${x}.")
```

with the message:

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Shadowed variable 'x'.

3 |     let x = 456;
            ^
            shadowing variable.

The shadowed variable was declared here:

2 |     let x = 123;
            ^
            shadowed variable.
```



## Useless Expressions

Flix rejects programs with _pure_ expressions whose results are discarded.

For example, the following program is rejected:

```flix
def main(): Unit \ IO =
    123 + 456;
    println("Hello World!")
```

with the message:

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Useless expression: It has no side-effect(s) and its result is discarded.

2 |     123 + 456;
        ^^^^^^^^^
        useless expression.

The expression has type 'Int32'
```

An expression that has no side-effect and whose result is unused is suspicious,
since it could just be removed from the program without changing its meaning.

## Must Use Values

Flix rejects programs with expressions whose values are discarded but where
their type is marked with the `@MustUse` annotation. Function types, and the
`Result` and `Validation` types from the Flix Standard Library are marked as
`@MustUse`.

For example, the following program is rejected:

```flix
def main(): Unit \ IO =
    File.creationTime("foo.txt");
    println("Hello World!")
```

with the message:

```
❌ -- Redundancy Error -------------------------------------------------- Main.flix

>> Unused value but its type is marked as @MustUse.

2 |     File.creationTime("foo.txt");
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        unused value.

The expression has type 'Result[String, Int64]'
```

Even though `File.creationTime` has a side-effect, we should probably be using the result `Result[String, Int64]`.
At least to ensure that the operation was successful.

If the result of an impure expression is truly not needed, then the `discard` expression can be used:

```flix
def main(): Unit \ IO =
    discard File.creationTime("foo.txt");
    println("Hello World!")
```

which permits a `@MustUse` value to be thrown away as long as the expression is non-pure.

[^1] See e.g. [Using Redundancies to Find Errors](https://dl.acm.org/doi/abs/10.1145/605466.605475).
-->
