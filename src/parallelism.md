# 並列性

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/parallelism.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/parallelism.md)ください。

これまでに、`spawn` 式を使うことで式を新しいスレッドで評価できることを見てきました：

```flix
region rc {
    spawn (1 + 2) @ rc
}
```

これにより、構造化並行性を用いて並行・並列プログラムを書くことができます。ただし欠点として、スレッド間の通信をチャネルを使って手動で調整しなければなりません。
並行性は不要で並列性だけが欲しい場合には、より軽量な方法として `par-yield` 式を使うことができます：

```flix
par (x <- e1; y <- e2; z <- e3)
    yield x + y + z
```

この式は `e1`、`e2`、`e3` を並列に評価し、それらの結果を `x`、`y`、`z` に束縛します。

`par-yield` を使うと、並列版の `List.map` 関数を書くことができます：

```flix
def parMap(f: a -> b, l: List[a]): List[b] = match l {
    case Nil     => Nil
    case x :: xs =>
        par (r <- f(x); rs <- parMap(f, xs))
            yield r :: rs
}
```

この関数は `f(x)` と `parMap(f, xs)` を並列に評価します。

> **注意:** `par-yield` 構文は純粋な式に対してのみ機能します。

エフェクトを伴う操作を並列に実行したい場合は、明示的なリージョンとスレッドを使う必要があります。

<!--
# Parallelism

We have seen how the `spawn` expression allows us to evaluate an expression in a
new thread:

```flix
region rc {
    spawn (1 + 2) @ rc
}
```

This allows us to write concurrent and parallel programs using structured
concurrency. The downside is that we must manually coordinate communication
between threads using channels.
If we want parallelism, but not concurrency, a more light-weight approach
is to use the `par-yield` expression:

```flix
par (x <- e1; y <- e2; z <- e3)
    yield x + y + z
```

which evaluates `e1`, `e2`, and `e3` in parallel and binds their results to `x`, `y`, and `z`.

We can use `par-yield` to write a parallel `List.map` function:

```flix
def parMap(f: a -> b, l: List[a]): List[b] = match l {
    case Nil     => Nil
    case x :: xs =>
        par (r <- f(x); rs <- parMap(f, xs))
            yield r :: rs
}
```

This function will evaluate `f(x)` and `parMap(f, xs)` in parallel.

> **Note:** The `par-yield` construct only works with pure expressions.

If you want to run effectful operations in parallel, you must use explicit
regions and threads.
-->
