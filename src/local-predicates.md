# ローカル述語

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/local-predicates.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/local-predicates.md)ください。

Flix は、_local predicates(ローカル述語)_ と呼ばれる抽象化の仕組みをサポートしています。ローカル述語は、ローカル変数と同じように、外部からは見えません。

ローカル述語を理解するために、次の例を考えてみましょう。グラフに閉路があるかどうかを計算する Datalog プログラム値を返す関数を書くことができます：

```flix
def cyclic(): #{Edge(Int32, Int32), Path(Int32, Int32), Cyclic()} = #{
    Path(x, y) :- Edge(x, y).
    Path(x, z) :- Path(x, y), Edge(y, z).
    Cyclic() :- Path(x, x).
}

def main(): Unit \ IO = 
    let db = #{
        Edge(1, 2).
        Edge(2, 3).
        Edge(3, 1).
    };
    query db, cyclic() select true from Cyclic() |> println
```

ここで `cyclic` 関数は、辺からなるグラフの推移閉包と、ある頂点からその頂点自身への経路が存在するかどうかを計算する 3 つのルールで構成された、_Datalog program value(Datalog プログラム値)_ を返します。`main` の中で `cyclic` 関数を使って、`db` で与えられる小さなグラフに閉路があるかどうかを判定しています。このプログラムをコンパイルして実行すると、`Vector#{true}` が出力されます。

`cyclic` に話を戻すと、その型は次のようになっています：

```flix
def cyclic(): #{Edge(Int32, Int32), Path(Int32, Int32), Cyclic()} = ...
```

この Datalog プログラム値は述語シンボル `Edge`、`Path`、`Cyclic` をそれぞれの型で使用しているので、この型は妥当です。しかし、もう少し考えてみると、`Path` 述語は実際にはこの計算にとってローカルなものであることに気づきます。外部から見えることを意図したものではなく、実装の詳細なのです！本当に望ましいのは、`Edge(Int32, Int32)` が _入力_ であり、`Cyclic()` が _出力_ であることです。さらに重要なのは、`Path(Int32, Int32)` は外部から見えてはならず、型の一部であってもならないということです。これは _predicate abstraction(述語抽象)_ によって実現できます：

```flix
def cyclic(): #{Edge(Int32, Int32), Cyclic()} = 
    #(Edge, Cyclic) -> #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
        Cyclic() :- Path(x, x).
    }
```

ここでは `#(Edge, Cyclic) -> v` という構文を使って、`v` の中の述語のうち `Edge` と `Cyclic` _だけ_ を外部から見えるようにすることを指定しています。これにより、`cyclic` の戻り値の型から `Path(Int32, Int32)` を省略できます。さらに、この Datalog プログラム値には、参照可能な `Path` 述語シンボルはもはや含まれていません。このことは、次のプログラムを観察することで確かめられます：

```flix
def main(): Unit \ IO = 
    let db = #{
        Edge(1, 2).
        Edge(2, 3).
        Edge(3, 1).
    };
    query db, cyclic() select (x, y) from Path(x, y) |> println
```

このプログラムは空のベクター `Vector#{}` を出力します。述語抽象によって `Path` がローカルになっているためです。

<!--
# Local Predicates

Flix supports an abstract mechanism called _local predicates_. A local
predicate, like a local variable, is not visible to the outside. 

To understand local predicates, consider the following example: We can a write a
function that returns a Datalog program value which computes whether a graph has
a cycle: 

```flix
def cyclic(): #{Edge(Int32, Int32), Path(Int32, Int32), Cyclic()} = #{
    Path(x, y) :- Edge(x, y).
    Path(x, z) :- Path(x, y), Edge(y, z).
    Cyclic() :- Path(x, x).
}

def main(): Unit \ IO = 
    let db = #{
        Edge(1, 2).
        Edge(2, 3).
        Edge(3, 1).
    };
    query db, cyclic() select true from Cyclic() |> println
```

Here the `cyclic` function returns a _Datalog program value_ which consists of
three rules that compute the transitive closure of a graph of edges and whether
there is path from a vertex to itself. We use the `cyclic` function inside
`main` to determine if a small graph, given by `db`, has a cyclic. The program
prints `Vector#{true}` when compiled and run. 

Returning to `cyclic`, we see that its type is:

```flix
def cyclic(): #{Edge(Int32, Int32), Path(Int32, Int32), Cyclic()} = ...
```

This is sensible because the Datalog program value uses the predicate symbols
`Edge`, `Path` and `Cyclic` with their given types. However, if we think more
about it, we can realize that the `Path` predicate is really local to the
computation: We are not meant to see it from the outside; it is an
implementation detail! What we really want is that `Edge(Int32, Int32)` should
be an _input_ and `Cyclic()` should be an _output_. More importantly,
`Path(Int32, Int32)` should not be visible from the outside nor part of the
type. We can achieve this with _predicate abstraction_:

```flix
def cyclic(): #{Edge(Int32, Int32), Cyclic()} = 
    #(Edge, Cyclic) -> #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
        Cyclic() :- Path(x, x).
    }
```

Here we use the syntax `#(Edge, Cyclic) -> v` to specific that _only_ the
predicates `Edge` and `Cyclic` from within `v` should be visible to the outside.
Thus we can omit `Path(Int32, Int32)` from the return type of `cyclic`.
Moreover, the Datalog program value no longer contains a `Path` predicate symbol
that can be referenced. We can evidence this by observing that the program:

```flix
def main(): Unit \ IO = 
    let db = #{
        Edge(1, 2).
        Edge(2, 3).
        Edge(3, 1).
    };
    query db, cyclic() select (x, y) from Path(x, y) |> println
```

prints the empty vector `Vector#{}` since `Path` has been made local by the predicate
abstraction.
-->
