# 冗長性

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/redundancy.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/redundancy.md)ください。

Flix コンパイラは、未使用の要素を含むプログラムを積極的に拒否します。これは、プログラマーが分かりにくいバグを避けられるように支援するためのものです[^1]。慣れるまで少し時間がかかるかもしれませんが、それに見合う価値のあるトレードオフだと私たちは考えています。

具体的には、Flix コンパイラはプログラムが次のものを含まないことを保証します：

- [未使用のローカル変数](#未使用のローカル変数unused-local-variables)：宣言されているものの、一度も使用されないローカル変数。
- [シャドーイングされたローカル変数](#シャドーイングされたローカル変数shadowed-local-variables)：他のローカル変数をシャドーイングするローカル変数。
- [無意味な式](#無意味な式useless-expressions)：値が破棄される純粋な式。

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

[^1] See e.g. [Using Redundancies to Find Errors](https://dl.acm.org/doi/abs/10.1145/605466.605475).
-->
