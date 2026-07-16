# Flix を使って束上の制約を解く

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/lattice-semantics.html)を参照してください。

Flix は、_関係に対する制約_ だけでなく、_束(Lattice)に対する制約_ もサポートしています。このような制約を作るには、まず束の演算（半順序(Partial order)、最小上界(Least upper bound)など）を関数として定義し、それらをある型に関連付け、そして束意味論(Lattice semantics)を持つ述語シンボルを宣言する必要があります。

まず、`Sign` データ型の定義から始めます：

```flix
use Sign.{Top, Neg, Zer, Pos, Bot};

enum Sign {
    case Top,
    case Neg,
    case Zer,
    case Pos,
    case Bot
}
```

この新しい型に対して、いつもの `Eq`・`Order`・`ToString` の各インスタンスを定義する必要があります。この `Order` インスタンスは、後で定義する半順序のインスタンスとは無関係であり、単に整形出力などのために要素をソートする目的で使われる点に注意してください。

```flix
instance Eq[Sign] {
    pub def eq(x: Sign, y: Sign): Bool = match (x, y) {
        case (Bot, Bot) => true
        case (Neg, Neg) => true
        case (Zer, Zer) => true
        case (Pos, Pos) => true
        case (Top, Top) => true
        case _          => false
    }
}

instance Order[Sign] {
    pub def compare(x: Sign, y: Sign): Comparison =
        let num = w -> match w {
            case Bot => 0
            case Neg => 1
            case Zer => 2
            case Pos => 3
            case Top => 4
        };
        num(x) <=> num(y)
}

instance ToString[Sign] {
    pub def toString(x: Sign): String = match x {
        case Bot => "Bot"
        case Neg => "Neg"
        case Zer => "Zer"
        case Pos => "Pos"
        case Top => "Top"
    }
}
```

これらのトレイトインスタンスが揃ったので、`Sign` に対する束の演算を定義できるようになりました。

ボトム要素(Bottom element)と半順序を定義します：

```flix
instance LowerBound[Sign] {
    pub def minValue(): Sign = Bot
}

instance PartialOrder[Sign] {
    pub def lessEqual(x: Sign, y: Sign): Bool =
        match (x, y) {
            case (Bot, _)   => true
            case (Neg, Neg) => true
            case (Zer, Zer) => true
            case (Pos, Pos) => true
            case (_, Top)   => true
            case _          => false
        }
}
```

次に、最小上界と最大下界(Greatest lower bound)を定義します：

```flix
instance JoinLattice[Sign] {
    pub def leastUpperBound(x: Sign, y: Sign): Sign =
        match (x, y) {
            case (Bot, _)   => y
            case (_, Bot)   => x
            case (Neg, Neg) => Neg
            case (Zer, Zer) => Zer
            case (Pos, Pos) => Pos
            case _          => Top
        }
}

instance MeetLattice[Sign] {
    pub def greatestLowerBound(x: Sign, y: Sign): Sign =
        match (x, y) {
            case (Top, _)   => y
            case (_, Top)   => x
            case (Neg, Neg) => Neg
            case (Zer, Zer) => Zer
            case (Pos, Pos) => Pos
            case _          => Bot
        }
}
```

これらの定義がすべて揃えば、束意味論を持つ Datalog 制約を書く準備は完了です。しかし先に進む前に、単調関数(Monotone function)をひとつ書いておきましょう：

```flix
def sum(x: Sign, y: Sign): Sign = match (x, y) {
    case (Bot, _)   => Bot
    case (_, Bot)   => Bot
    case (Neg, Zer) => Neg
    case (Zer, Neg) => Neg
    case (Zer, Zer) => Zer
    case (Zer, Pos) => Pos
    case (Pos, Zer) => Pos
    case (Pos, Pos) => Pos
    case _          => Top
}
```

これでようやく、すべてを組み合わせて使うことができます：

```flix
pub def main(): Unit \ IO =
    let p = #{
        LocalVar("x"; Pos).
        LocalVar("y"; Zer).
        LocalVar("z"; Neg).
        AddStm("r1", "x", "y").
        AddStm("r2", "x", "y").
        AddStm("r2", "y", "z").
        LocalVar(r; sum(v1, v2)) :-
            AddStm(r, x, y), LocalVar(x; v1), LocalVar(y; v2).
    };
    query p select (r, v) from LocalVar(r; v) |> println
```

束意味論を示すために `;` が注意深く使われている点に注目してください。

## 束意味論を使った最短経路の計算

束意味論は、最短経路の計算にも使えます。

鍵となるのは、独自の新しいデータ型 `D` を定義することです。これは単なる `Int32` ですが、整数の逆順によって束をなします（つまり、最小の要素は `Int32.maxValue()` です）。

```flix
use D.D;

pub enum D with Eq, Order, ToString {
    case D(Int32)
}

instance PartialOrder[D] {
    pub def lessEqual(x: D, y: D): Bool =
        let D(n1) = x;
        let D(n2) = y;
        n1 >= n2        // 注意：順序が反転しています。
}

instance LowerBound[D] {
    // 注意：順序が反転しているため、最大の値が最小の要素になります。
    pub def minValue(): D = D(Int32.maxValue())
}

instance UpperBound[D] {
    // 注意：順序が反転しているため、最小の値が最大の要素になります。
    pub def maxValue(): D = D(Int32.minValue())
}

instance JoinLattice[D] {
    pub def leastUpperBound(x: D, y: D): D =
        let D(n1) = x;
        let D(n2) = y;
        D(Int32.min(n1, n2))        // 注意：順序が反転しています。
}

instance MeetLattice[D] {
    pub def greatestLowerBound(x: D, y: D): D =
        let D(n1) = x;
        let D(n2) = y;
        D(Int32.max(n1, n2))        // 注意：順序が反転しています。
}

def shortestPath(g: Set[(t, Int32, t)], o: t): Map[t, D] with Order[t] =
    let db = inject g into Edge/3;
    let pr = #{
        Dist(o; D(0)).
        Dist(y; add(d1 , D(d2))) :- Dist(x; d1), Edge(x, d2, y).
    };
    query db, pr select (x , d) from Dist(x; d) |> Vector.toMap

def add(x: D, y: D): D =
    let D(n1) = x;
    let D(n2) = y;
    D(n1 + n2)

def main(): Unit \ IO =
    let g = Set#{
        ("Aarhus", 200, "Flensburg"),
        ("Flensburg", 150, "Hamburg")
    };
    println(shortestPath(g, "Aarhus"))

```

実は、Flix には `D` のような型が組み込みで用意されています。それは `Down` と呼ばれ、基となる型の順序を単純に反転させます。これを使うと、プログラムは次のように書けます：

```flix
use Down.Down;

def shortestPaths(g: Set[(t, Int32, t)], o: t): Map[t, Down[Int32]] with Order[t] =
    let db = inject g into Edge/3;
    let pr = #{
        Dist(o; Down(0)).
        Dist(y; add(d1 , Down(d2))) :- Dist(x; d1), Edge(x, d2, y).
    };
    query db, pr select (x , d) from Dist(x; d) |> Vector.toMap

def add(x: Down[Int32], y: Down[Int32]): Down[Int32] =
    let Down(n1) = x;
    let Down(n2) = y;
    Down(n1 + n2)

def main(): Unit \ IO =
    let g = Set#{
        ("Aarhus", 200, "Flensburg"),
        ("Flensburg", 150, "Hamburg")
    };
    println(shortestPaths(g, "Aarhus"))

```

<!--
# Using Flix to Solve Constraints on Lattices

Flix supports not only _constraints on relations_,
but also _constraints on lattices_.
To create such constraints, we must first define the
lattice operations (the partial order, the least
upper bound, and so on) as functions, associate them
with a type, and then declare the predicate symbols
that have lattice semantics.

We begin with the definition of the `Sign` data type:

```flix
use Sign.{Top, Neg, Zer, Pos, Bot};

enum Sign {
    case Top,
    case Neg,
    case Zer,
    case Pos,
    case Bot
}
```

We need to define the usual `Eq`, `Order`, and
`ToString` instances for this new type.
Note that the order instance is unrelated to the
partial order instance we will later define, and is
simply used to sort elements for pretty printing etc.

```flix
instance Eq[Sign] {
    pub def eq(x: Sign, y: Sign): Bool = match (x, y) {
        case (Bot, Bot) => true
        case (Neg, Neg) => true
        case (Zer, Zer) => true
        case (Pos, Pos) => true
        case (Top, Top) => true
        case _          => false
    }
}

instance Order[Sign] {
    pub def compare(x: Sign, y: Sign): Comparison =
        let num = w -> match w {
            case Bot => 0
            case Neg => 1
            case Zer => 2
            case Pos => 3
            case Top => 4
        };
        num(x) <=> num(y)
}

instance ToString[Sign] {
    pub def toString(x: Sign): String = match x {
        case Bot => "Bot"
        case Neg => "Neg"
        case Zer => "Zer"
        case Pos => "Pos"
        case Top => "Top"
    }
}
```

With these trait instances in place, we can now
define the lattice operations on `Sign`.

We define the bottom element and the partial order:

```flix
instance LowerBound[Sign] {
    pub def minValue(): Sign = Bot
}

instance PartialOrder[Sign] {
    pub def lessEqual(x: Sign, y: Sign): Bool =
        match (x, y) {
            case (Bot, _)   => true
            case (Neg, Neg) => true
            case (Zer, Zer) => true
            case (Pos, Pos) => true
            case (_, Top)   => true
            case _          => false
        }
}
```

Next, we define the least upper bound and greatest
lower bound:

```flix
instance JoinLattice[Sign] {
    pub def leastUpperBound(x: Sign, y: Sign): Sign =
        match (x, y) {
            case (Bot, _)   => y
            case (_, Bot)   => x
            case (Neg, Neg) => Neg
            case (Zer, Zer) => Zer
            case (Pos, Pos) => Pos
            case _          => Top
        }
}

instance MeetLattice[Sign] {
    pub def greatestLowerBound(x: Sign, y: Sign): Sign =
        match (x, y) {
            case (Top, _)   => y
            case (_, Top)   => x
            case (Neg, Neg) => Neg
            case (Zer, Zer) => Zer
            case (Pos, Pos) => Pos
            case _          => Bot
        }
}
```

With all of these definitions we are ready to write
Datalog constraints with lattice semantics.
But before we proceed, let us also write a single
monotone function:

```flix
def sum(x: Sign, y: Sign): Sign = match (x, y) {
    case (Bot, _)   => Bot
    case (_, Bot)   => Bot
    case (Neg, Zer) => Neg
    case (Zer, Neg) => Neg
    case (Zer, Zer) => Zer
    case (Zer, Pos) => Pos
    case (Pos, Zer) => Pos
    case (Pos, Pos) => Pos
    case _          => Top
}
```

We can now finally put everything to use:

```flix
pub def main(): Unit \ IO =
    let p = #{
        LocalVar("x"; Pos).
        LocalVar("y"; Zer).
        LocalVar("z"; Neg).
        AddStm("r1", "x", "y").
        AddStm("r2", "x", "y").
        AddStm("r2", "y", "z").
        LocalVar(r; sum(v1, v2)) :-
            AddStm(r, x, y), LocalVar(x; v1), LocalVar(y; v2).
    };
    query p select (r, v) from LocalVar(r; v) |> println
```

Note the careful use of `;` to designate lattice
semantics.

## Using Lattice Semantics to Compute Shortest Paths

We can also use lattice semantics to compute shortest paths.

The key is to define our own new data type `D` which is simple an `Int32` with
forms a lattice with the reverse order of the integers (e.g. the smallest
element is `Int32.maxValue()`).

```flix
use D.D;

pub enum D with Eq, Order, ToString {
    case D(Int32)
}

instance PartialOrder[D] {
    pub def lessEqual(x: D, y: D): Bool =
        let D(n1) = x;
        let D(n2) = y;
        n1 >= n2        // Note: Order reversed.
}

instance LowerBound[D] {
    // Note: Because the order is reversed, the largest value is the smallest.
    pub def minValue(): D = D(Int32.maxValue())
}

instance UpperBound[D] {
    // Note: Because the order is reversed, the smallest value is the largest.
    pub def maxValue(): D = D(Int32.minValue())
}

instance JoinLattice[D] {
    pub def leastUpperBound(x: D, y: D): D =
        let D(n1) = x;
        let D(n2) = y;
        D(Int32.min(n1, n2))        // Note: Order reversed.
}

instance MeetLattice[D] {
    pub def greatestLowerBound(x: D, y: D): D =
        let D(n1) = x;
        let D(n2) = y;
        D(Int32.max(n1, n2))        // Note: Order reversed.
}

def shortestPath(g: Set[(t, Int32, t)], o: t): Map[t, D] with Order[t] =
    let db = inject g into Edge/3;
    let pr = #{
        Dist(o; D(0)).
        Dist(y; add(d1 , D(d2))) :- Dist(x; d1), Edge(x, d2, y).
    };
    query db, pr select (x , d) from Dist(x; d) |> Vector.toMap

def add(x: D, y: D): D =
    let D(n1) = x;
    let D(n2) = y;
    D(n1 + n2)

def main(): Unit \ IO =
    let g = Set#{
        ("Aarhus", 200, "Flensburg"),
        ("Flensburg", 150, "Hamburg")
    };
    println(shortestPath(g, "Aarhus"))

```

Flix actually comes with a type like `D` built-in. It's called `Down` and simply
reverses the order on the underlying type. We can use it and write the program
as:

```flix
use Down.Down;

def shortestPaths(g: Set[(t, Int32, t)], o: t): Map[t, Down[Int32]] with Order[t] =
    let db = inject g into Edge/3;
    let pr = #{
        Dist(o; Down(0)).
        Dist(y; add(d1 , Down(d2))) :- Dist(x; d1), Edge(x, d2, y).
    };
    query db, pr select (x , d) from Dist(x; d) |> Vector.toMap

def add(x: Down[Int32], y: Down[Int32]): Down[Int32] =
    let Down(n1) = x;
    let Down(n2) = y;
    Down(n1 + n2)

def main(): Unit \ IO =
    let g = Set#{
        ("Aarhus", 200, "Flensburg"),
        ("Flensburg", 150, "Hamburg")
    };
    println(shortestPaths(g, "Aarhus"))

```
-->
