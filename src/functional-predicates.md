# 関数述語

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/functional-predicates.html)を参照してください。

論理述語を使いたいものの、そのタプルをすべて網羅的に列挙することは避けたい、という状況に遭遇することがあります。

例えば、`prime` が範囲 `[from; to]` 内の素数であるときに成り立つ述語 `PrimeInRange(from, to, prime)` が欲しい状況を考えてみましょう。このような述語を思い描くことはできますが、実際に計算するのは現実的ではありません。その代わりに多くの場合で望まれるのは、`PrimeInRange` を **関数述語**(Functional)、すなわち `from` と `to` を _入力_ として受け取り、素数の集合を _出力_ として生成する関数として扱うことです。具体的には、次のようなルールを書きたいとします。

```flix
R(p) :- P(x), Q(y), PrimeInRange(x, y, p).
```

ただし、すべての `x`、`y`、`p` に対して `PrimeInRange` を評価することは避けたいのです。

これは次のようにして実現できます。まず、関数を書きます。

```flix
def primesInRange(b: Int32, e: Int32): Vector[Int32] = 
    Vector.range(b, e) |> Vector.filter(isPrime)
```

重要なのは、`primesInRange` が、開始インデックス `b` と終了インデックス `e` を受け取ってタプル（この場合は単一要素）の Vector を返す _関数_ であるという点です。これにより、`primesInRange` は私たちが関心を持つタプルを効率的に計算できます。これをルールの中で使うには、次のように書きます。

```flix
R(p) :- P(b), Q(e), let p = primesInRange(b, e).
```

ここでは、`b` と `e` が `primesInRange` の入力として、`p` がその出力として明確に識別されています。具体的には、Flix は `b` と `e` が正に束縛されている（Positively bound）こと（すなわち、非否定のボディアトム——この場合は `P` と `Q` ——によって束縛されていること）を要求します。この例では `primesInRange` は `Int32` の Vector を返しますが、一般に関数述語はタプルの Vector を返すことができます。

> **注意:** 現在の実装における重要な制限として、関数述語の左辺（LHS）にある変数は再束縛して**はいけません**。つまり、関数述語が `let (a, b) = f(x, y)` という形式である場合、`a` と `b` はそのルールの中で再束縛されてはいけません。

<!--
# Functional Predicates

We sometimes run into a situation where we would like to use a logic predicate,
but not to exhaustively enumerate all of its tuples. 

For example, we can image a situation where we want a predicate
`PrimeInRange(from, to, prime)` which holds if `prime` is a prime number in the
range `[from; to]`. While we can imagine such a predicate, it is not feasible to
compute with. Instead, what we often want, is that we want to treat
`PrimeInRange` as a **functional**, i.e. a function that given `from` and `to`
as _input_ produces a set of primes as _output_. To make matters concrete, we
might want to write the rule:

```flix
R(p) :- P(x), Q(y), PrimeInRange(x, y, p).
```

but without having to evaluate `PrimeInRange` for every `x`, `y`, and `p`.

We can achieve this as follows. We write a function:

```flix
def primesInRange(b: Int32, e: Int32): Vector[Int32] = 
    Vector.range(b, e) |> Vector.filter(isPrime)
```

The key is that `primesInRange` is a _function_ which returns a vector of tuples
(in this case single elements), given a begin `b` and end index `e`. Thus
`primesInRange` can efficiently compute the tuples we are interested in. To use
it in our rule, we write: 

```flix
R(p) :- P(b), Q(e), let p = primesInRange(b, e).
```

where `b` and `e` are clearly identified as the input of `primesInRange` and `p`
as its output. Specifically, Flix requires that `b` and `e` are positively bound
(i.e. bound by a non-negative body atom-- in this case `P` and `Q`.) In this
case, `primesInRanges` returns a vector of `Int32`s, but in general a functional
may return a vector of tuples. 

> **Note:** An important limitation in the current implementation is that the
> variables in the LHS of a functional predicate _must not_ be rebound. That is,
> if the functional predicate is of the form `let (a, b) = f(x, y)` then `a` and
> `b` must not be rebound in the rule. 
-->
