# デバッグ

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/debugging.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/debugging.md)ください。

デバッグの際には、式や変数の値を出力できると便利なことがよくあります。

そこで、次のように書いてみたくなるかもしれません：

```flix
def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    println("The sum of ${x} and ${y} is ${result}");
    result
```

残念ながら、これは動作しません：

```
❌ -- Type Error -------------------------------------------------- Main.flix

>> Unable to unify the effect formulas: 'IO' and 'Pure'.

1 |> def sum(x: Int32, y: Int32): Int32 =
2 |>     let result = x + y;
3 |>     println("The sum of ${x} and ${y} is ${result}");
4 |>     result
```

問題は、`println` が `IO` エフェクトを持つことです。そのため、純粋な関数の中でプリントデバッグ(Print debugging)のために `println` を使うことはできません。`sum` 関数に `IO` エフェクトを持たせることもできますが、それが望ましいことはめったにありません。その代わりに、Flix にはプリントデバッグを可能にする組み込みのデバッグ機能が用意されています。

## `Debug.dprintln` 関数

代わりに、`Debug.dprintln` 関数を使って次のように書けます：

```flix
use Debug.dprintln;

def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    dprintln("The sum of ${x} and ${y} is ${result}");
    result
```

`sum` 関数の内部では、`dprintln` は `Debug` エフェクトを持ちますが、その特殊な性質により、関数を抜けると `Debug` エフェクトは「消えます」。つまり、関数の型とエフェクトのシグネチャには含まれません。

## ソース位置付きのデバッグ

特殊な _デバッグ文字列補間子(Debug string interpolator)_ を使うと、print 文にソース位置(Source location)を付加できます：

```flix
use Debug.dprintln;

def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    dprintln(d"The sum of ${x} and ${y} is ${result}");
    result
```

> `dprintln` のより詳しい紹介は、ブログ記事
> [Effect Systems vs Print Debugging: A Pragmatic Solution](https://blog.flix.dev/blog/effect-systems-vs-print-debugging/) で読むことができます。

<!--
# Debugging

When debugging, it is often helpful to output the value of an expression or
variable.

We might try something like:

```flix
def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    println("The sum of ${x} and ${y} is ${result}");
    result
```

Unfortunately this does not work:

```
❌ -- Type Error -------------------------------------------------- Main.flix

>> Unable to unify the effect formulas: 'IO' and 'Pure'.

1 |> def sum(x: Int32, y: Int32): Int32 =
2 |>     let result = x + y;
3 |>     println("The sum of ${x} and ${y} is ${result}");
4 |>     result
```

The problem is that `println` has the `IO`. Hence, we cannot use it to for 
print debugging inside pure functions. We could make our `sum` function have
the `IO` effect, but that is rarely what we want. Instead, Flix has a
built-in debugging facility that allows us to do print-line debugging.

## The `Debug.dprintln` Function

Instead, we can use the `Debug.dprintln` function and write:

```flix
use Debug.dprintln;

def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    dprintln("The sum of ${x} and ${y} is ${result}");
    result
```

Inside the `sum` function, the `dprintln` has the effect `Debug`, but due to
its special nature, the `Debug` effect "disappears" once we exit the function,
i.e. it is not part of its type and effect signature.

## Debugging with Source Locations

We can use the special _debug string interpolator_ to add source locations
to our print statements:

```flix
use Debug.dprintln;

def sum(x: Int32, y: Int32): Int32 =
    let result = x + y;
    dprintln(d"The sum of ${x} and ${y} is ${result}");
    result
```

> A longer introduction to `dprintln` is available in the
> blog post [Effect Systems vs Print Debugging: A Pragmatic Solution](https://blog.flix.dev/blog/effect-systems-vs-print-debugging/)
-->
