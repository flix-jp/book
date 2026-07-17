# 停止性検査

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/termination-checking.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/termination-checking.md)ください。

Flix は `@Terminates` アノテーションをサポートしています。これは、関数が*構造的再帰(Structural recursion)*である――つまり、すべての入力に対して停止することが保証されている――ことをコンパイラに検証させるものです。`@Terminates` が付与された関数は、再帰呼び出しを仮引数の厳密な部分構造(Strict substructure)に対してのみ行わなければなりません。コンパイラはこれをコンパイル時に検査し、関数が構造的再帰の要件を満たさない場合はエラーを報告します。

## 構造的再帰

`@Terminates` の中心となる考え方は*構造的再帰*です。すべての再帰呼び出しは、仮引数に対するパターンマッチによってコンストラクタの内部から取り出された構成要素を引数として渡さなければなりません。この構成要素は元の値よりも厳密に小さいため、再帰は必ずいつか基底ケースに到達します。

例えば、以下は独自のリスト型に対する構造的再帰の `length` 関数です：

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def length(l: MyList[Int32]): Int32 = match l {
    case MyList.Nil         => 0
    case MyList.Cons(_, xs) => 1 + length(xs)
}
```

再帰呼び出しは `xs` を渡していますが、これは `l` の `Cons` コンストラクタの内部で束縛されたものです。`xs` は `l` よりも厳密に小さいため、コンパイラはこの関数を受理します。

## 木構造の再帰

構造的再帰はリストに限らず、任意の代数的データ型に対して機能します。各呼び出しが仮引数の厳密な部分構造を受け取っている限り、関数は同じ分岐の中で複数の再帰呼び出しを行うことができます。

例えば、以下は二分木に対する `size` 関数です：

```flix
enum MyTree[a] {
    case Leaf(a)
    case Node(MyTree[a], MyTree[a])
}

@Terminates
def size(t: MyTree[Int32]): Int32 = match t {
    case MyTree.Leaf(_)    => 1
    case MyTree.Node(l, r) => size(l) + size(r)
}
```

`l` と `r` はどちらも `t` の `Node` コンストラクタの内部で束縛されているため、両方の再帰呼び出しが有効です。

## 複数のパラメータ

関数が複数のパラメータを持つ場合、再帰呼び出しごとに減少する必要があるのは*ひとつ*のパラメータだけです。それ以外のパラメータは変更せずにそのまま渡して構いません。

例えば、`append` は `l2` を変更せずに渡しながら、`l1` に対して再帰します：

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def append(l1: MyList[Int32], l2: MyList[Int32]): MyList[Int32] = match l1 {
    case MyList.Nil         => l2
    case MyList.Cons(x, xs) => MyList.Cons(x, append(xs, l2))
}
```

コンパイラは `xs` が `l1` の厳密な部分構造であることを認識し、それで十分だと判断します。`l2` が減少しないことは問題ありません。

> **警告:** `@Terminates` は関数が停止することを保証しますが、末尾再帰であることは保証*しません*。例えば、上の `append` 関数は構造的再帰ですが、末尾再帰では*ありません*――再帰呼び出しが `MyList.Cons(x, ...)` に包まれているためです。そのため、非常に長いリストに対してはスタックオーバーフローを起こす可能性があります。スタックセーフな再帰関数の書き方については、[末尾再帰](./tail-recursion.md)のセクションを参照してください。

## ローカル定義

`@Terminates` 関数の内部にあるローカル定義は、それぞれ独立に検査されます。ローカル関数は自分自身のパラメータに対して再帰できます：

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def length(l: MyList[Int32]): Int32 =
    def loop(ll: MyList[Int32], acc: Int32): Int32 = match ll {
        case MyList.Nil         => acc
        case MyList.Cons(_, xs) => loop(xs, acc + 1)
    };
    loop(l, 0)
```

ここでは `loop` が自分自身のパラメータ `ll` に対して再帰しており、その厳密な部分構造である `xs` を渡しています。外側の関数 `length` は再帰していないため、自明に停止します。

## 高階関数

`@Terminates` 関数は、仮引数として受け取ったクロージャを適用することができます。これにより、`map` のような高階のパターンが可能になります：

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def map(f: Int32 -> Int32, l: MyList[Int32]): MyList[Int32] = match l {
    case MyList.Nil         => MyList.Nil
    case MyList.Cons(x, xs) => MyList.Cons(f(x), map(f, xs))
}
```

`f` は `map` の仮引数であるため、`f(x)` という適用は許可されます。コンパイラは `f` がパラメータに由来することを追跡し、この呼び出しを許可します。

一方、*ローカルに構築された*クロージャを適用することは禁止されています：

```flix
@Terminates
def bad(x: Int32): Int32 =
    let c = y -> y + 1;
    c(x)
```

これは拒否されます。`c` は仮引数ではなく、ローカルに定義されたクロージャであり、一般には任意の計算を隠し持つ可能性があるためです。

> **警告:** `@Terminates` は、関数引数 `f` も停止するという*仮定のもとで* `map` が停止することを保証します。`f` が停止しない関数であれば、`map` も停止しないかもしれません。このアノテーションは `map` 自身の構造的再帰を検証するだけであり、`f` の振る舞いは検査しません。

## 他の関数の呼び出し

`@Terminates` 関数が呼び出せるのは、同じく `@Terminates` が付与された関数だけです。アノテーションのない関数を呼び出すとエラーになります。

例えば、以下は拒否されます：

```flix
def g(x: Int32): Int32 = x * 2

@Terminates
def f(x: Int32): Int32 = g(x)
```

コンパイラは次のように報告します：

```
>> Call to non-@Terminates function 'g' in @Terminates function 'f'.

   ... g(x)
       ^^^^^^^^^
       non-terminating call
```

修正方法は、呼び出される側の関数にもアノテーションを付けることです：

```flix
@Terminates
def g(x: Int32): Int32 = x * 2

@Terminates
def f(x: Int32): Int32 = g(x)
```

## 厳密正値性

構造的再帰に使われる enum 型は*厳密正(Strictly positive)*でなければなりません。ある型が厳密正であるとは、どのコンストラクタにおいても、矢印の左側に再帰的な出現を含まないことをいいます。

例えば、以下の enum は厳密正では**ありません**。`MkBad` の引数において、`Bad` が `->` の左側に現れているためです：

```flix
enum Bad {
    case MkBad(Bad -> Int32)
}

@Terminates
def f(x: Bad): Int32 = match x {
    case Bad.MkBad(_) => 0
}
```

コンパイラはこれを次のエラーで拒否します：

```
>> Non-strictly positive type in 'f'.

   ... case MkBad(Bad -> Int32)
                  ^^^^^^^^^^^^
                  negative occurrence
```

## よくあるエラー

最もよくある間違いは、パターンマッチで取り出した部分構造ではなく、元のパラメータをそのまま渡してしまうことです：

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def f(x: MyList[Int32]): Int32 = match x {
    case MyList.Nil         => 0
    case MyList.Cons(_, xs) => f(x)
}
```

再帰呼び出しが、パターンから取り出した末尾の `xs` ではなく、元のパラメータである `x` を渡していることに注目してください。コンパイラは次のように報告します：

```
>> Non-structural recursion in 'f'.

   ... f(x)
       ^^^^
       non-structural recursive call

   Parameter   Argument   Status
   x           x          alias of 'x' (not destructured)
```

診断テーブルは、どの引数に問題があるかを示しています。修正方法は、`x` の代わりに `xs` を渡すことです：

```flix
case MyList.Cons(_, xs) => f(xs)
```

<!--
# Termination Checking

Flix supports the `@Terminates` annotation, which asks the compiler to verify
that a function is _structurally recursive_ — meaning it is guaranteed to
terminate on all inputs. A function annotated with `@Terminates` must make
recursive calls only on strict substructures of its formal parameters. The
compiler checks this at compile time and reports an error if the function does
not satisfy the structural recursion requirement.

## Structural Recursion

The core idea behind `@Terminates` is _structural recursion_: every recursive
call must pass an argument that was obtained by pattern matching on a formal
parameter and extracting a component from inside a constructor. This component is
strictly smaller than the original, so the recursion must eventually reach a base
case.

For example, here is a structurally recursive `length` function on a custom list
type:

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def length(l: MyList[Int32]): Int32 = match l {
    case MyList.Nil         => 0
    case MyList.Cons(_, xs) => 1 + length(xs)
}
```

The recursive call passes `xs`, which is bound inside the `Cons` constructor of
`l`. Since `xs` is strictly smaller than `l`, the compiler accepts this function.

## Tree Recursion

Structural recursion works on any algebraic data type, not just lists. A function
may make multiple recursive calls in the same branch, as long as each call
receives a strict substructure of a formal parameter.

For example, here is a `size` function on a binary tree:

```flix
enum MyTree[a] {
    case Leaf(a)
    case Node(MyTree[a], MyTree[a])
}

@Terminates
def size(t: MyTree[Int32]): Int32 = match t {
    case MyTree.Leaf(_)    => 1
    case MyTree.Node(l, r) => size(l) + size(r)
}
```

Both `l` and `r` are bound inside the `Node` constructor of `t`, so both
recursive calls are valid.

## Multiple Parameters

When a function has multiple parameters, only _one_ parameter needs to decrease
per recursive call. The other parameters may be passed unchanged.

For example, `append` recurses on `l1` while passing `l2` unchanged:

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def append(l1: MyList[Int32], l2: MyList[Int32]): MyList[Int32] = match l1 {
    case MyList.Nil         => l2
    case MyList.Cons(x, xs) => MyList.Cons(x, append(xs, l2))
}
```

The compiler sees that `xs` is a strict substructure of `l1`, which is
sufficient. The fact that `l2` does not decrease is fine.

> **Warning:** `@Terminates` guarantees that a function terminates, but it does
> _not_ guarantee that it is tail recursive. For example, the `append` function
> above is structurally recursive but _not_ tail recursive — the recursive call
> is wrapped in `MyList.Cons(x, ...)`. This means it can overflow the stack on
> very long lists. See the [Tail Recursion](./tail-recursion.md) section for how
> to write stack-safe recursive functions.

## Local Definitions

Local definitions inside a `@Terminates` function are checked independently.
A local function may recurse on its own parameters:

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def length(l: MyList[Int32]): Int32 =
    def loop(ll: MyList[Int32], acc: Int32): Int32 = match ll {
        case MyList.Nil         => acc
        case MyList.Cons(_, xs) => loop(xs, acc + 1)
    };
    loop(l, 0)
```

Here `loop` recurses on its own parameter `ll`, passing `xs` which is a strict
substructure. The outer function `length` is non-recursive, so it is
trivially terminating.

## Higher-Order Functions

A `@Terminates` function may apply closures that come from its formal parameters.
This allows higher-order patterns like `map`:

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def map(f: Int32 -> Int32, l: MyList[Int32]): MyList[Int32] = match l {
    case MyList.Nil         => MyList.Nil
    case MyList.Cons(x, xs) => MyList.Cons(f(x), map(f, xs))
}
```

The application `f(x)` is allowed because `f` is a formal parameter of
`map`. The compiler tracks that `f` originates from a parameter and permits
the call.

However, applying a _locally-constructed_ closure is forbidden:

```flix
@Terminates
def bad(x: Int32): Int32 =
    let c = y -> y + 1;
    c(x)
```

This is rejected because `c` is not a formal parameter — it is a locally
defined closure that could, in general, hide arbitrary computation.

> **Warning:** `@Terminates` guarantees that `map` terminates _assuming_ its
> function argument `f` also terminates. If `f` is a non-terminating function,
> then `map` may not terminate either. The annotation only verifies the
> structural recursion of `map` itself — it does not check the behavior of `f`.

## Calling Other Functions

A `@Terminates` function may only call other functions that are also annotated
with `@Terminates`. Calling a function without the annotation is an error.

For example, the following is rejected:

```flix
def g(x: Int32): Int32 = x * 2

@Terminates
def f(x: Int32): Int32 = g(x)
```

The compiler reports:

```
>> Call to non-@Terminates function 'g' in @Terminates function 'f'.

   ... g(x)
       ^^^^^^^^^
       non-terminating call
```

The fix is to annotate the callee:

```flix
@Terminates
def g(x: Int32): Int32 = x * 2

@Terminates
def f(x: Int32): Int32 = g(x)
```

## Strict Positivity

Enum types used for structural recursion must be _strictly positive_. A type is
strictly positive if it does not contain a recursive occurrence to the left of
an arrow in any constructor.

For example, the following enum is **not** strictly positive because `Bad`
appears to the left of `->` in the argument to `MkBad`:

```flix
enum Bad {
    case MkBad(Bad -> Int32)
}

@Terminates
def f(x: Bad): Int32 = match x {
    case Bad.MkBad(_) => 0
}
```

The compiler rejects this with:

```
>> Non-strictly positive type in 'f'.

   ... case MkBad(Bad -> Int32)
                  ^^^^^^^^^^^^
                  negative occurrence
```

## Common Errors

The most common mistake is passing the original parameter instead of a
substructure obtained from pattern matching:

```flix
enum MyList[a] {
    case Nil
    case Cons(a, MyList[a])
}

@Terminates
def f(x: MyList[Int32]): Int32 = match x {
    case MyList.Nil         => 0
    case MyList.Cons(_, xs) => f(x)
}
```

Notice that the recursive call passes `x` (the original parameter) instead of
`xs` (the tail extracted from the pattern). The compiler reports:

```
>> Non-structural recursion in 'f'.

   ... f(x)
       ^^^^
       non-structural recursive call

   Parameter   Argument   Status
   x           x          alias of 'x' (not destructured)
```

The diagnostic table shows which arguments are problematic. The fix is to pass
`xs` instead of `x`:

```flix
case MyList.Cons(_, xs) => f(xs)
```
-->
