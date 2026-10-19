# 注釈

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/annotations.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/annotations.md)ください。

Flix は、限られた組み込みの注釈(Annotation)の集合をサポートしています。

例えば、`@Test` 注釈は関数をテストとしてマークします。テストフレームワークはそのような関数をすべて実行し、合格したかどうかを報告します：

```flix
@Test
def testSum01(): Unit \ Assert =
    Assert.assertEq(expected = 6, sum(1 :: 2 :: 3 :: Nil, 0))
```

もう一つの例として、`@Tailrec` 注釈は、関数内のすべての再帰呼び出しが末尾呼び出しであることをコンパイラに検証させます。下記の `sum` 関数は、再帰呼び出しが関数の最後に行われる処理であるため、このチェックに合格します：

```flix
@Tailrec
def sum(l: List[Int32], acc: Int32): Int32 = match l {
    case Nil     => acc
    case x :: xs => sum(xs, acc + x)
}
```

Flix には、次の組み込み注釈があります：

| 注釈                | 意味                                                                                                        |
|---------------------|-------------------------------------------------------------------------------------------------------------|
| `@Test`             | 関数をテストとして実行します。[テストフレームワーク](./test-framework.md)を参照してください。               |
| `@Skip`             | `@Test` 関数をスキップします。テストは実行されず、スキップとして報告されます。                              |
| `@DefaultHandler`   | エフェクトのデフォルトハンドラであることを示します。[デフォルトハンドラ](./default-handlers.md)を参照してください。 |
| `@Tailrec`          | 自己再帰呼び出しがすべて末尾呼び出しであることを検証します。[末尾再帰](./tail-recursion.md)を参照してください。 |
| `@Terminates`       | 関数が構造的再帰であることを検証します。[停止性検査](./termination-checking.md)を参照してください。           |
| `@Inline`           | コンパイラに関数を常にインライン化させます。                                                                |
| `@DontInline`       | コンパイラに関数を決してインライン化させません。                                                            |
| `@Deprecated`       | その宣言が非推奨であることを文書化します。                                                                  |
| `@Experimental`     | その宣言が予告なく変更または削除される可能性があることを文書化します。                                      |
| `@Lazy`             | その関数が遅延評価を使用することを文書化します。                                                            |
| `@LazyWhenPure`     | 純粋な関数が与えられた場合に遅延評価を使用することを文書化します。[純粋性リフレクション](./purity-reflection.md)を参照してください。 |
| `@Parallel`         | その関数が並列評価を使用することを文書化します。                                                            |
| `@ParallelWhenPure` | 純粋な関数が与えられた場合に並列評価を使用することを文書化します。[純粋性リフレクション](./purity-reflection.md)を参照してください。 |

最後の6つの注釈はドキュメント用です。これらは API ドキュメントやエディタ上のヒントとして表示されますが、コンパイラが関数をどのように変換するかには影響しません。

> **注意:** Flix には `@Override` 注釈はありません。トレイトのデフォルト実装を上書きするインスタンスは、代わりに `redef` キーワードで関数をマークします。

<!--
# Annotations

Flix supports a limited set of built-in annotations.

For example, the `@Test` annotation marks a function as a test. The test
framework runs every such function and reports whether it passed:

```flix
@Test
def testSum01(): Unit \ Assert =
    Assert.assertEq(expected = 6, sum(1 :: 2 :: 3 :: Nil, 0))
```

As another example, the `@Tailrec` annotation asks the compiler to verify that
every recursive call in a function is a tail call. The `sum` function below
passes the check, because the recursive call is the last thing it does:

```flix
@Tailrec
def sum(l: List[Int32], acc: Int32): Int32 = match l {
    case Nil     => acc
    case x :: xs => sum(xs, acc + x)
}
```

Flix has the following built-in annotations:

| Annotation          | Meaning                                                                                                                |
|---------------------|------------------------------------------------------------------------------------------------------------------------|
| `@Test`             | Runs the function as a test. See [Test Framework](./test-framework.md).                                                |
| `@Skip`             | Skips a `@Test` function. The test is reported as skipped, not run.                                                    |
| `@DefaultHandler`   | Marks the default handler of an effect. See [Default Handlers](./default-handlers.md).                                 |
| `@Tailrec`          | Verifies that every self-recursive call is a tail call. See [Tail Recursion](./tail-recursion.md).                     |
| `@Terminates`       | Verifies that the function is structurally recursive. See [Termination Checking](./termination-checking.md).           |
| `@Inline`           | Makes the compiler always inline the function.                                                                         |
| `@DontInline`       | Makes the compiler never inline the function.                                                                          |
| `@Deprecated`       | Documents that the declaration is deprecated.                                                                          |
| `@Experimental`     | Documents that the declaration may change or be removed without warning.                                               |
| `@Lazy`             | Documents that the function uses lazy evaluation.                                                                      |
| `@LazyWhenPure`     | Documents that the function uses lazy evaluation when given a pure function. See [Purity Reflection](./purity-reflection.md). |
| `@Parallel`         | Documents that the function uses parallel evaluation.                                                                  |
| `@ParallelWhenPure` | Documents that the function uses parallel evaluation when given a pure function. See [Purity Reflection](./purity-reflection.md). |

The last six annotations are documentation. They are shown in the API docs and
as hints in the editor, but they do not change how the compiler translates the
function.

> **Note:** Flix has no `@Override` annotation. An instance that overrides a
> default implementation from its trait marks the function with the `redef`
> keyword instead.
-->
