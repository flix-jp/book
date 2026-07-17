# 末尾再帰

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/tail-recursion.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/tail-recursion.md)ください。

Flix では、そして一般に関数型プログラミングでは、反復処理は[再帰](https://en.wikipedia.org/wiki/Recursion_(computer_science))によって表現されます。

例えば、リストがある要素を含むかどうかを判定したい場合、再帰関数を次のように書くことができます：

```flix
def memberOf(x: a, l: List[a]): Bool with Eq[a] = 
    match l {
        case Nil     => false
        case y :: ys => if (x == y) true else memberOf(x, ys)
    }
```

`memberOf` 関数はリスト `l` に対してパターンマッチを行います。リストが空であれば `false` を返します。そうでなければ、要素 `y` とリストの残り `ys` が得られます。`x == y` であれば要素が見つかったので `true` を返します。そうでなければ、リストの残り `ys` に対して _再帰_ します。

`memberOf` への再帰呼び出しは _末尾位置(Tail position)_ にあります。つまり、`memberOf` 関数の中で最後に行われる処理です。これには 2 つの重要な利点があります。(a) Flix コンパイラが `memberOf` を（関数呼び出しよりも効率的な）通常のループに書き換えられること、そしてより重要なのは (b) 呼び出しスタックの高さが決して増えないため、`memberOf` の呼び出しでスタックがオーバーフローすることが _あり得ない_ ことです。

> **ヒント**: Flix は完全な[末尾呼び出し](https://en.wikipedia.org/wiki/Tail_call)除去をサポートしています。これは、末尾位置にある再帰呼び出しがスタックの高さを決して増やさず、したがってスタックオーバーフローを引き起こし得ないことを意味します！

_特筆すべき点として_、Flix が備えているのは単なる末尾呼び出し最適化ではなく、___完全な___ 末尾呼び出し除去です。これにより、次のプログラムは正常にコンパイルされ、実行されます：

```flix
def isOdd(n: Int32): Bool =
    if (n == 0) false else isEvn(n - 1)

def isEvn(n: Int32): Bool =
    if (n == 0) true else isOdd(n - 1)

def main(): Unit \ IO =
    isOdd(12345) |> println
```

これは他の多くのプログラミング言語では成り立たないことです。


## 非末尾呼び出しとスタックオーバーフロー

Flix コンパイラは末尾呼び出しがスタックをオーバーフローさせないことを _保証_ しますが、末尾位置にない関数呼び出しについては同じことは言えません。

例えば、次の[階乗関数](https://en.wikipedia.org/wiki/Factorial)の実装は呼び出しスタックをオーバーフローさせます：

```flix
def factorial(n: Int32): Int32 = match n {
    case 0 => 1
    case _ => n * factorial(n - 1)
}
```

それは次のプログラムで確認できます：

```flix
def main(): Unit \ IO = 
    println(factorial(1_000_000))
```

これをコンパイルして実行すると、次の出力が得られます：

```
java : Exception in thread "main" java.lang.StackOverflowError
	at Cont%Int32.unwind(Cont%Int32)
	at Def%factorial.invoke(Unknown Source)
	at Cont%Int32.unwind(Cont%Int32)
	at Def%factorial.invoke(Unknown Source)
	at Cont%Int32.unwind(Cont%Int32)
    ... many more frames ...
```

よく知られたテクニックとして、`factorial` をアキュムレータ(Accumulator)を使う形に書き換える方法があります：

```flix
def factorial(n: Int32): Int32 = 
    def visit(x, acc) = match x {
        case 0 => acc
        case _ => visit(x - 1, x * acc)
    };
    visit(n, 1)
```

ここでは `visit` 関数が末尾再帰になっているため、スタックをオーバーフローさせることはありません。

## @Tailrec アノテーション

Flix は `@Tailrec` アノテーションを提供しています。これは、関数内のすべての自己再帰呼び出しが末尾位置にあることを検証するようコンパイラに指示するものです。このアノテーションは省略可能で、実行時の挙動を変えることはありません。ドキュメントおよび検証のためのツールとして機能します。

例えば、アキュムレータスタイルの `sum` 関数は末尾再帰です：

```flix
@Tailrec
def sum(l: List[Int32], acc: Int32): Int32 = match l {
    case Nil     => acc
    case x :: xs => sum(xs, acc + x)
}
```

`sum` への再帰呼び出しは関数内の最後の操作であり、その結果に対してそれ以上の処理は行われないため、コンパイラはこれを受理します。

対照的に、次の関数は拒否されます：

```flix
@Tailrec
def length(l: List[Int32]): Int32 = match l {
    case Nil     => 0
    case _ :: xs => length(xs) + 1
}
```

ここでは `length(xs)` の結果が加算（`+ 1`）に使われているため、再帰呼び出しは末尾位置に _ありません_。コンパイラは次のエラーを報告します：

```
>> Non-tail recursive call in @Tailrec function 'length'.

   ... length(xs) + 1
       ^^^^^^^^^^
       non-tail recursive call
```

これを修正するには、前述のようにアキュムレータを使う形に関数を書き換えます。

> **ヒント:** `@Tailrec` アノテーションは純粋にコンパイル時のチェックです。コード生成には影響しません。Flix はアノテーションの有無にかかわらず、末尾位置にあるあらゆる呼び出しに対して既に完全な末尾呼び出し除去を行います。コードが進化しても関数が末尾再帰の _ままである_ ことをコンパイラに保証してほしい場合に、`@Tailrec` を使ってください。

<!--
# Tail Recursion

In Flix, and in functional programming in general, iteration is expressed
through [recursion](https://en.wikipedia.org/wiki/Recursion_(computer_science)).

For example, if we want to determine if a list contains an element, we can write
a recursive function:

```flix
def memberOf(x: a, l: List[a]): Bool with Eq[a] = 
    match l {
        case Nil     => false
        case y :: ys => if (x == y) true else memberOf(x, ys)
    }
```

The `memberOf` function pattern matches on the list `l`. If it is empty then it
returns `false`. Otherwise, we have an element `y` and the rest of the list is
`ys`. If `x == y` then we have found the element and we return `true`. Otherwise
we _recurse_ on the rest of the list `ys`. 

The recursive call to `memberOf` is in _tail position_, i.e. it is the last
thing to happen in the `memberOf` function. This has two important benefits: (a)
the Flix compiler is able to rewrite `memberOf` to use an ordinary loop (which
is more efficient than a function call) and more importantly (b) a call to
`memberOf` _cannot_ overflow the stack, because the call stack never increases
in height.

> **Tip**: Flix has support for [full tail
call](https://en.wikipedia.org/wiki/Tail_call) elimination which means that
recursive calls in tail position never increase the stack height and hence
cannot cause the stack to overflow!

We _remark_ that Flix has ___full___ tail call elimination, not just tail call
optimization. This means that the following program compiles and runs
successfully: 

```flix
def isOdd(n: Int32): Bool =
    if (n == 0) false else isEvn(n - 1)

def isEvn(n: Int32): Bool =
    if (n == 0) true else isOdd(n - 1)

def main(): Unit \ IO =
    isOdd(12345) |> println
```

which is not the case in many other programming languages.


## Non-Tail Calls and StackOverflows

While the Flix compiler _guarantees_ that tail calls cannot overflow the stack,
the same is not true for function calls in non-tail positions.

For example, the following implementation of the [factorial
function](https://en.wikipedia.org/wiki/Factorial) overflows the call stack: 

```flix
def factorial(n: Int32): Int32 = match n {
    case 0 => 1
    case _ => n * factorial(n - 1)
}
```

as this program shows:

```flix
def main(): Unit \ IO = 
    println(factorial(1_000_000))
```

which when compiled and run produces:

```
java : Exception in thread "main" java.lang.StackOverflowError
	at Cont%Int32.unwind(Cont%Int32)
	at Def%factorial.invoke(Unknown Source)
	at Cont%Int32.unwind(Cont%Int32)
	at Def%factorial.invoke(Unknown Source)
	at Cont%Int32.unwind(Cont%Int32)
    ... many more frames ...
```

A well-known technique is to rewrite `factorial` to use an accumulator:

```flix
def factorial(n: Int32): Int32 = 
    def visit(x, acc) = match x {
        case 0 => acc
        case _ => visit(x - 1, x * acc)
    };
    visit(n, 1)
```

Here the `visit` function is tail recursive, hence it cannot overflow the stack.

## The @Tailrec Annotation

Flix provides the `@Tailrec` annotation, which asks the compiler to verify that
every self-recursive call in a function is in tail position. The annotation is
optional and does not change runtime behavior — it serves as a documentation and
verification tool.

For example, an accumulator-style `sum` function is tail recursive:

```flix
@Tailrec
def sum(l: List[Int32], acc: Int32): Int32 = match l {
    case Nil     => acc
    case x :: xs => sum(xs, acc + x)
}
```

The compiler accepts this because the recursive call to `sum` is the last
operation in the function — nothing further is done with its result.

In contrast, the following function is rejected:

```flix
@Tailrec
def length(l: List[Int32]): Int32 = match l {
    case Nil     => 0
    case _ :: xs => length(xs) + 1
}
```

Here, the result of `length(xs)` is used in an addition (`+ 1`), so the
recursive call is _not_ in tail position. The compiler reports an error:

```
>> Non-tail recursive call in @Tailrec function 'length'.

   ... length(xs) + 1
       ^^^^^^^^^^
       non-tail recursive call
```

To fix this, rewrite the function to use an accumulator, as shown above.

> **Tip:** The `@Tailrec` annotation is purely a compile-time check. It does
> not affect code generation — Flix already performs full tail call elimination
> for any call in tail position, whether annotated or not. Use `@Tailrec` when
> you want the compiler to guarantee that a function _remains_ tail recursive as
> the code evolves.
-->
