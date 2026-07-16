# 純粋性リフレクション

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/purity-reflection.html)を参照してください。

> **注意:** これは高度な機能であり、エキスパートのみが使用すべきです。

純粋性リフレクション(Purity reflection)を使うと、高階関数が引数として受け取った関数の純粋性を調べられるようになります。

これにより、*選択的な*遅延評価や並列評価を行う関数を書くことができます。

例えば、以下は `Set.count` の実装です：

```flix
@ParallelWhenPure
pub def count(f: a -> Bool \ ef, s: Set[a]): Int32 \ ef =
    match purityOf(f) {
        case Purity.Pure(g) =>
            if (useParallelEvaluation(s))
                let h = (k, _) -> g(k);
                let Set(t) = s;
                RedBlackTree.parCount(h, t)
            else
                foldLeft((b, k) -> if (f(k)) b + 1 else b, 0, s)
        case Purity.Impure(g) => foldLeft((b, k) -> if (g(k)) b + 1 else b, 0, s)
    }
```

ここでは `purityOf` 関数を使って、`f` の純粋性をリフレクションしています：

- `f` が純粋であれば、`Set.count` はセットの要素に対して*並列に*評価されます（セットが並列化に見合うだけ十分に大きい場合）。
- `f` がエフェクトを持つ場合は、通常の（シングルスレッドの）畳み込みを使用します。

この利点は、`f` が純粋で*ありさえすれば*、並列性を無償で得られることです。

<!--
# Purity Reflection

> **Note:** This is an advanced feature and should only be used by experts.

Purity reflection enables higher-order functions to inspect the purity of their
function arguments. 

This allows us to write functions that use _selective_ lazy and/or parallel
evaluation.

For example, here is the implementation of `Set.count`:

```flix
@ParallelWhenPure
pub def count(f: a -> Bool \ ef, s: Set[a]): Int32 \ ef =
    match purityOf(f) {
        case Purity.Pure(g) =>
            if (useParallelEvaluation(s))
                let h = (k, _) -> g(k);
                let Set(t) = s;
                RedBlackTree.parCount(h, t)
            else
                foldLeft((b, k) -> if (f(k)) b + 1 else b, 0, s)
        case Purity.Impure(g) => foldLeft((b, k) -> if (g(k)) b + 1 else b, 0, s)
    }
```

Here the `purityOf` function is used to reflect on the purity of `f`:

- If `f` is pure then we evaluate `Set.count` in _parallel_ over the elements of
  the set (if the set is big enough to warrant it). 
- If `f` is effectful then we use an ordinary (single-threaded) fold.

The advantage is that we get parallelism for free – _if_ `f` is pure.
-->
