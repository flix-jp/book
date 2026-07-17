# 不動点

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/fixpoints.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/fixpoints.md)ください。

Flix のユニークな機能のひとつに、*関係に対する制約（constraint on relations）*および*束に対する制約（constraint on lattices）*の不動点計算（Fixpoint computation）を言語組み込みでサポートしていることが挙げられます。

ここでは読者がすでに Datalog に精通していることを前提とし、Flix 固有の機能に焦点を当てます。

## 関係に対する制約を Flix で解く

Flix では、関数の内部で不動点計算を実行できます。

例えば、辺の集合 `s`、始点ノード `src`、終点ノード `dst` が与えられたとき、`src` から `dst` への経路が存在するかどうかを計算してみましょう。この問題は次のようにエレガントに解くことができます：

```flix
def isConnected(s: Set[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let rules = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
    };
    let edges = inject s into Edge/2;
    let paths = query edges, rules select true from Path(src, dst);
    not (paths |> Vector.isEmpty)

def main(): Unit \ IO =
    let s = Set#{(1, 2), (2, 3), (3, 4), (4, 5)};
    let src = 1;
    let dst = 5;
    if (isConnected(s, src, dst)) {
        println("Found a path between ${src} and ${dst}!")
    } else {
        println("Did not find a path between ${src} and ${dst}!")
    }
```

`isConnected` 関数は他の関数とまったく同じように振る舞います。辺の集合（`Int32` のペア）、`Int32` の始点ノード、`Int32` の終点ノードを渡して呼び出すことができます。`isConnected` の興味深い点は、その実装が小さな Datalog プログラムを使って目的のタスクを解いていることです。

`isConnected` 関数の中で、ローカル変数 `rules` は、`Path` 関係を定義する 2 つのルールからなる Datalog プログラムの断片を保持しています。述語シンボルである `Edge` と `Path` は明示的に導入する必要がなく、単に使うだけでよいことに注意してください。ローカル変数 `edges` は、集合 `s` のすべてのタプルを `Edge` ファクトに変換して得られる、辺のファクトのコレクションを保持しています。次に、ローカル変数 `paths` は、これらのファクトとルール（`edges` と `rules`）の不動点を計算し、`Path(src, dst)` というファクトが存在する*場合に*ブール値 `true` を選択した結果を保持します。ここでの `src` と `dst` は、レキシカルに束縛された関数のパラメータであることに注意してください。したがって、`paths` は空の配列（経路が見つからなかった）か、要素が 1 つの配列（経路が見つかった）のいずれかになり、これをそのまま結果として返します。

Flix は強く型付けされた言語です。誤った型の項（あるいは誤ったアリティ）で述語シンボルを使おうとすると、型検査器によって検出されます。また、Flix は型推論をサポートしているため、`Edge` や `Path` の型を宣言する必要がなかったことにも注目してください。

## 第一級制約によるプログラミング

Flix のもうひとつのユニークな機能が、*第一級制約（First-class constraints）*のサポートです。第一級制約とは、構築し、受け渡し、他の制約と合成し、最終的に解くことができる値のことです。制約システムの解はまた別の制約システムであり、それをさらに合成することができます。例えば：

```flix
def getParents(): #{ ParentOf(String, String) | r } = #{
    ParentOf("Pompey", "Strabo").
    ParentOf("Gnaeus", "Pompey").
    ParentOf("Pompeia", "Pompey").
    ParentOf("Sextus", "Pompey").
}

def getAdoptions(): #{ AdoptedBy(String, String) | r } = #{
    AdoptedBy("Augustus", "Caesar").
    AdoptedBy("Tiberius", "Augustus").
}

def withAncestors(): #{ ParentOf(String, String),
                        AncestorOf(String, String) | r } = #{
        AncestorOf(x, y) :- ParentOf(x, y).
        AncestorOf(x, z) :- AncestorOf(x, y), AncestorOf(y, z).
}

def withAdoptions(): #{ AdoptedBy(String, String),
                        AncestorOf(String, String) | r } = #{
    AncestorOf(x, y) :- AdoptedBy(x, y).
}

def main(): Unit \ IO =
    let c = false;
    if (c) {
        query getParents(), getAdoptions(), withAncestors()
            select (x, y) from AncestorOf(x, y) |> println
    } else {
        query getParents(), getAdoptions(), withAncestors(), withAdoptions()
            select (x, y) from AncestorOf(x, y) |> println
    }
```

このプログラムは `ParentOf`、`AncestorOf`、`AdoptedBy` という 3 つの述語シンボルを使っています。`getParents` 関数は生物学上の親を表すファクトのコレクションを返し、一方 `getAdoptions` 関数は養子縁組を表すファクトのコレクションを返します。`withAncestors` 関数は、`ParentOf` 関係を使って `AncestorOf` 関係を導出する 2 つの制約を返します。`withAdoptions` 関数は、`AdoptedBy` 関係を使って `ParentOf` 関係を導出する制約を返します。

`main` 関数では、ローカル変数 `c` によって、生物学上の親のみを考慮する Datalog プログラムに問い合わせるか、養子縁組も含めるかを制御しています。

見てのとおり、これらの関数の型は行多相（Row-polymorphic）です。例えば `getParents` のシグネチャは `def getParents(): #{ ParentOf | r }` であり、ここで `r` は、この関数の結果と合成できる残りの述語を表す行多相型変数です。

> **設計ノート**
>
> 行多相型は、制約システムに現れうる述語の過大近似として理解するのが最も適切です。
> 例えば、ある制約システムが `#{ A(String), B(Int32, Int32) }` という型を持つ場合、
> それは述語シンボル `A` や `B` を使うファクトやルールが必ず含まれることを意味するわけではありませんが、
> 述語シンボル `C` を参照するファクトやルールが一切含まれないことは保証されます。

## 多相な第一級制約

Flix のさらにもうひとつのユニークな機能が、*多相的な*第一級制約のサポートです。つまり、1 つ以上の制約が、その項の型について多相であるような制約です。例えば：

```flix
def edgesWithNumbers(): #{ LabelledEdge(String, Int32 , String) | r } = #{
    LabelledEdge("a", 1, "b").
    LabelledEdge("b", 1, "c").
    LabelledEdge("c", 2, "d").
}

def edgesWithColor(): #{ LabelledEdge(String, String, String) | r } = #{
    LabelledEdge("a", "red", "b").
    LabelledEdge("b", "red", "c").
    LabelledEdge("c", "blu", "d").
}

def closure(): #{ LabelledEdge(String, l, String),
                  LabelledPath(String, l, String) } with Order[l] = #{
    LabelledPath(x, l, y) :- LabelledEdge(x, l, y).
    LabelledPath(x, l, z) :- LabelledPath(x, l, y), LabelledPath(y, l, z).
}

def main(): Unit \ IO =
    query edgesWithNumbers(), closure()
        select (x, l, z) from LabelledPath(x, l, z) |> println;
    query edgesWithColor(), closure()
        select (x, l, z) from LabelledPath(x, l, z) |> println
```

ここでは `LabelledEdge` と `LabelledPath` という 2 つの述語シンボルを使っています。各述語は `l` という型パラメータを持ち、辺や経路に付随する「ラベル」の型について多相になっています。`edgesWithNumbers` はラベルが整数である辺ファクトのコレクションを返し、一方 `edgesWithColor` はラベルが文字列であるファクトのコレクションを返していることに注目してください。`closure` 関数は多相であり、同じラベルを持つ辺の推移閉包（Transitive closure）を計算する 2 つのルールを返します。

Flix の型システムにより、異なる型のラベルを持つ辺（や経路）を誤って混在させることはできないようになっています。

## Datalog へのファクトの注入

Flix には、関数型のデータ構造（リスト、セット、マップなど）を Datalog のファクトに変換できる柔軟なメカニズムが用意されています。

例えば、ペアの Flix リストが与えられたとき、それを Datalog のファクトのコレクションに変換できます：

```flix
let l = (1, 2) :: (2, 3) :: Nil;
let p = inject l into Edge/2;
```

ここで `l` の型は `List[(Int32, Int32)]` です。`inject` 式は `l` を、型 `#{ Edge(Int32, Int32) | ... }` の Datalog 制約集合 `p` に変換します。この式には述語のアリティを指定します：`Edge/2`。一般的な形式は `Predicate/Arity` です。

`inject` 式は、`Foldable` トレイトを実装する任意の型に対して使えます。そのため、リスト、セット、マップなどで利用できます。

`inject` 式は複数のコレクションを同時に扱うこともできます。例えば：

```flix
let names = "Lucky Luke" :: "Luke Skywalker" :: Nil;
let jedis = "Luke Skywalker" :: Nil;
let p = inject names, jedis into Name/1, Jedi/1;
```

ここで `p` の型は `#{ Name(String), Jedi(String) | ... }` です。

## 不動点計算のパイプライン

制約システムの解（すなわち不動点）は、また別の制約システムです。これを利用して、不動点計算の*パイプライン*を構築できます。つまり、ある不動点計算の結果を別の不動点計算に入力として渡すことができます。例えば：

```flix
def main(): Unit \ IO =
    let f1 = #{
        ColorEdge(1, "blue", 2).
        ColorEdge(2, "blue", 3).
        ColorEdge(3, "red", 4).
    };
    let r1 = #{
        ColorPath(x, c, y) :- ColorEdge(x, c, y).
        ColorPath(x, c, z) :- ColorPath(x, c, y), ColorEdge(y, c, z).
    };
    let r2 = #{
        ColorlessPath(x, y) :- ColorPath(x, _, y).
    };
    let m = solve f1, r1 project ColorPath;
    query m, r2 select (x, y) from ColorlessPath(x, y) |> println
```

このプログラムは `ColorEdge`、`ColorPath`、`ColorlessPath` という 3 つの述語を使っています。目標は、色付きの辺の推移閉包を計算し、その後で辺に色のないグラフを構築することです。

このプログラムはまず `f1` と `r1` の不動点を計算し、`ColorPath` ファクトを取り出します。その結果は `m` に格納されます。次に、`m` と `r2` に問い合わせて、すべての `ColorlessPath` ファクトを選択します。

<!--
# Fixpoints

A unique feature of Flix is its built-in support for
fixpoint computations on _constraint on relations_
and _constraint on lattices_.

We assume that the reader is already familiar with
Datalog and focus on the Flix specific features.

## Using Flix to Solve Constraints on Relations

We can use Flix to solve a fixpoint computation
inside a function.

For example, given a set of edges `s`, a `src` node,
and `dst` node, compute if there is a path from `src`
to `dst`.
We can elegantly solve this problem as follows:

```flix
def isConnected(s: Set[(Int32, Int32)], src: Int32, dst: Int32): Bool =
    let rules = #{
        Path(x, y) :- Edge(x, y).
        Path(x, z) :- Path(x, y), Edge(y, z).
    };
    let edges = inject s into Edge/2;
    let paths = query edges, rules select true from Path(src, dst);
    not (paths |> Vector.isEmpty)

def main(): Unit \ IO =
    let s = Set#{(1, 2), (2, 3), (3, 4), (4, 5)};
    let src = 1;
    let dst = 5;
    if (isConnected(s, src, dst)) {
        println("Found a path between ${src} and ${dst}!")
    } else {
        println("Did not find a path between ${src} and ${dst}!")
    }
```

The `isConnected` function behaves like any other
function: We can call it with a set of edges
(`Int32`-pairs), an `Int32` source node, and
an `Int32` destination node.
What is interesting about `isConnected` is that its
implementation uses a small Datalog program to solve
the task at hand.

In the `isConnected` function, the local variable
`rules` holds a Datalog program fragment that
consists of two rules which define the `Path`
relation.
Note that the predicate symbols, `Edge` and `Path` do
not have to be explicitly introduced; they are simply
used.
The local variable `edges` holds a collection of edge
facts that are obtained by taking all the tuples in
the set `s` and turning them into `Edge` facts.
Next, the local variable `paths` holds the result of
computing the fixpoint of the facts and rules
(`edges` and `rules`) and selecting the Boolean
`true` _if_ there is a `Path(src, dst)` fact.
Note that here `src` and `dst` are the
lexically-bound function parameters.
Thus, `paths` is either an empty array (no paths were
found) or a one-element array (a path was found), and
we simply return this fact.

Flix is strongly typed.
Any attempt to use predicate symbol with terms of the
wrong type (or with the wrong arity) is caught by the
type checker.
Note also that Flix supports type inference, hence we
did not have to declare the type of `Edge` nor of
`Path`.

## Programming with First-class Constraints

A unique feature of Flix is its support for
_first-class constraints_.
A first-class constraint is a value that can be
constructed, passed around, composed with other
constraints, and ultimately solved.
The solution to a constraint system is another
constraint system which can be further composed.
For example:

```flix
def getParents(): #{ ParentOf(String, String) | r } = #{
    ParentOf("Pompey", "Strabo").
    ParentOf("Gnaeus", "Pompey").
    ParentOf("Pompeia", "Pompey").
    ParentOf("Sextus", "Pompey").
}

def getAdoptions(): #{ AdoptedBy(String, String) | r } = #{
    AdoptedBy("Augustus", "Caesar").
    AdoptedBy("Tiberius", "Augustus").
}

def withAncestors(): #{ ParentOf(String, String),
                        AncestorOf(String, String) | r } = #{
        AncestorOf(x, y) :- ParentOf(x, y).
        AncestorOf(x, z) :- AncestorOf(x, y), AncestorOf(y, z).
}

def withAdoptions(): #{ AdoptedBy(String, String),
                        AncestorOf(String, String) | r } = #{
    AncestorOf(x, y) :- AdoptedBy(x, y).
}

def main(): Unit \ IO =
    let c = false;
    if (c) {
        query getParents(), getAdoptions(), withAncestors()
            select (x, y) from AncestorOf(x, y) |> println
    } else {
        query getParents(), getAdoptions(), withAncestors(), withAdoptions()
            select (x, y) from AncestorOf(x, y) |> println
    }
```

The program uses three predicate symbols: `ParentOf`,
`AncestorOf`, and `AdoptedBy`.
The `getParents`function returns a collection of facts
that represent biological parents, whereas the
`getAdoptions` function returns a collection of facts
that represent adoptions.
The `withAncestors` function returns two constraints
that populate the `AncestorOf` relation using the
`ParentOf` relation.
The `withAdoptions` function returns a constraint
that populates the `ParentOf` relation using the
`AdoptedBy` relation.

In the `main` function the local variable `c`
controls whether we query a Datalog program that only
considers biological parents or if we include
adoptions.

As can be seen, the types the functions are
row-polymorphic.
For example, the signature of `getParents` is
`def getParents(): #{ ParentOf | r }` where `r`
is row polymorphic type variable that represent the
rest of the predicates that the result of the
function can be composed with.

> **Design Note**
>
> The row polymorphic types are best understood as an
> over-approximation of the predicates that may occur
> in a constraint system.
> For example, if a constraint system has type
> `#{ A(String), B(Int32, Int32) }` that doesn't
> necessarily mean that it will contain facts or rules
> that use the predicate symbols `A` or `B`, but it
> does guarantee that it will not contain any fact or
> rule that refer to a predicate symbol `C`.

## Polymorphic First-class Constraints

Another unique feature of Flix is its support for
first-class _polymorphic_ constraints.
That is, constraints where one or more constraints
are polymorphic in their term types.
For example:

```flix
def edgesWithNumbers(): #{ LabelledEdge(String, Int32 , String) | r } = #{
    LabelledEdge("a", 1, "b").
    LabelledEdge("b", 1, "c").
    LabelledEdge("c", 2, "d").
}

def edgesWithColor(): #{ LabelledEdge(String, String, String) | r } = #{
    LabelledEdge("a", "red", "b").
    LabelledEdge("b", "red", "c").
    LabelledEdge("c", "blu", "d").
}

def closure(): #{ LabelledEdge(String, l, String),
                  LabelledPath(String, l, String) } with Order[l] = #{
    LabelledPath(x, l, y) :- LabelledEdge(x, l, y).
    LabelledPath(x, l, z) :- LabelledPath(x, l, y), LabelledPath(y, l, z).
}

def main(): Unit \ IO =
    query edgesWithNumbers(), closure()
        select (x, l, z) from LabelledPath(x, l, z) |> println;
    query edgesWithColor(), closure()
        select (x, l, z) from LabelledPath(x, l, z) |> println
```

Here we use two predicate symbols: `LabelledEdge` and
`LabelledPath`.
Each predicate has a type parameter named `l` and is
polymorphic in the "label" type associated with the
edge/path.
Note how `edgesWithNumbers` returns a collection of
edge facts where the labels are integers, whereas
`edgesWithColor` returns a collection of facts where
the labels are strings.
The `closure` function is polymorphic and returns two
rules that compute the transitive closure of edges
that have the same label.

The Flix type system ensures that we cannot
accidentally mix edges (or paths) with different
types of labels.

## Injecting Facts into Datalog

Flix provides a flexible mechanism that allows
functional data structures (such as lists, sets,
and maps) to be converted into Datalog facts.

For example, given a Flix list of pairs we can
convert it to a collection of Datalog facts:

```flix
let l = (1, 2) :: (2, 3) :: Nil;
let p = inject l into Edge/2;
```

where `l` has type `List[(Int32, Int32)]`.
The `inject` expression converts `l` into a Datalog
constraint set `p` of type
`#{ Edge(Int32, Int32) | ... }`.
The expression includes the predicate's arity:
`Edge/2`.
The general form is `Predicate/Arity`.

The `inject` expression works with any type that
implements the `Foldable` trait.
Consequently, it can be used with lists, sets, maps,
and so forth.

The `inject` expression can operate on multiple
collections simultaneously.
For example:

```flix
let names = "Lucky Luke" :: "Luke Skywalker" :: Nil;
let jedis = "Luke Skywalker" :: Nil;
let p = inject names, jedis into Name/1, Jedi/1;
```

where `p` has type
`#{ Name(String), Jedi(String) | ... }`.

## Pipelines of Fixpoint Computations

The solution (i.e. fixpoint) of a constraint system
is another constraint system.
We can use this to construct _pipelines_ of fixpoint
computations, i.e. to feed the result of one fixpoint
computation into another fixpoint computation.
For example:

```flix
def main(): Unit \ IO =
    let f1 = #{
        ColorEdge(1, "blue", 2).
        ColorEdge(2, "blue", 3).
        ColorEdge(3, "red", 4).
    };
    let r1 = #{
        ColorPath(x, c, y) :- ColorEdge(x, c, y).
        ColorPath(x, c, z) :- ColorPath(x, c, y), ColorEdge(y, c, z).
    };
    let r2 = #{
        ColorlessPath(x, y) :- ColorPath(x, _, y).
    };
    let m = solve f1, r1 project ColorPath;
    query m, r2 select (x, y) from ColorlessPath(x, y) |> println
```

The program uses three predicates: `ColorEdge`,
`ColorPath`, and `ColorlessPath`.
Our goal is to compute the transitive closure of the
colored edges and then afterwards construct a graph
where the edges have no color.

The program first computes the fixpoint of `f1` and
`r1` and injects out the `ColorPath` fact.
The result is stored in `m`. Next, the program
queries `m` and `r2`, and selects all `ColorlessPath`
facts.
-->
