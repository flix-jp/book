# 関連型

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/associated-types.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/associated-types.md)ください。

関連型(Associated type)とは、トレイトの型メンバであり、各トレイトインスタンスごとに指定されるものです。関連型は、[多引数型クラス](https://en.wikipedia.org/wiki/Type_class#Multi-parameter_type_classes)に代わる、より自然な選択肢と見なされることがよくあります。

例を使って関連型を説明します。

加算できる型のためのトレイトを次のように定義できます：

```flix
trait Addable[t] {
    pub def add(x: t, y: t): t
}
```

浮動小数点数、整数、文字列などの型に対して、`Addable` トレイトの複数のインスタンスを実装できます。例えば、`Int32` のインスタンスは次のようになります：

```flix
instance Addable[Int32] {
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

`String` のインスタンスは次のとおりです：

```flix
instance Addable[String] {
    pub def add(x: String, y: String): String = "${x}${y}"
}
```

しかし、Set に要素を追加したい場合はどうでしょうか？

直感的には、次のように書きたくなります：

```flix
instance Addable[Set[a]] with Order[a] {
    pub def add(s: Set[a], x: a): Set[a] = Set.insert(x, s)
}
```

しかし、この `add` のシグネチャは `Addable` で宣言されたシグネチャと一致しません。

この問題は、関連型を使って `Addable` の柔軟性を高めることで解決できます：

```flix
trait Addable[t] {
    type Rhs
    pub def add(x: t, y: Addable.Rhs[t]): t
}
```

`Addable` トレイトは `Rhs` という名前の関連型を持つようになりました。`add` のシグネチャに見られるように、この関連型は `Addable.Rhs[t]` として参照します。`Addable` のインスタンスを宣言するときは、必ず関連型を指定しなければなりません。

これまでどおり、整数や文字列に対するインスタンスも実装できます。例えば：

```flix
instance Addable[Int32] {
    type Rhs = Int32
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

さらに、Set に要素を追加できるインスタンスも実装できます：

```flix
instance Addable[Set[a]] with Order[a] {
    type Rhs = a
    pub def add(s: Set[a], x: a): Set[a] = Set.insert(x, s)
}
```

重要なのは、_各トレイトインスタンスが関連型を指定する_という点です。

ここで、`Set[a]` に対して2つのインスタンス、すなわち (a) 上記のように Set に要素を追加するもの、(b) 2つの Set を足し合わせるもの、を指定できるのではないかと考えるかもしれません：

```flix
instance Addable[Set[a]] with Order[a] {
    type Rhs = Set[a]
    pub def add(x: Set[a], y: Set[a]): Set[a] = Set.union(x, y)

}
```

しかし、それぞれのインスタンスは単独では有効であるものの、両方を同時に持つことはできません：

```
❌ -- Instance Error -------------------------------------------------- 

>> Overlapping instances for 'Addable'.

...
```

このような重複したインスタンスが存在すると、`Addable.add(Set#{}, Set#{})` のような式が曖昧になってしまいます。2つの Set を足し合わせているのでしょうか？それとも、空の Set を Set に追加しているのでしょうか？

## 例: `ForEach` トレイト

関連型を使って、`forEach` 関数を持つコレクションのためのトレイトを定義できます：

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

ここで `t` はコレクションの型であり、関連型 `Elm` はその要素の型です。`ForEach` に対していくつかのインスタンスを実装できます。例えば、`List[a]` のインスタンスを実装できます：

```flix
instance ForEach[List[a]] {
    type Elm = a
    pub def forEach(f: a -> Unit \ ef, x: List[a]): Unit \ ef = List.forEach(f, x)
}
```

`Map[k, v]` のインスタンスも実装できます：

```flix
instance ForEach[Map[k, v]] {
    type Elm = (k, v)
    pub def forEach(f: ((k, v)) -> Unit \ ef, x: Map[k, v]): Unit \ ef = 
        Map.forEach(k -> v -> f((k, v)), x)
}
```

興味深く、また有用なのは、要素型をキーと値のペアとして定義できる点です。`f` の引数にはペアを受け取らせたいので、引数の周りに追加の括弧が必要になります。

`String` に対しては、個々の文字を1つずつ反復処理できるインスタンスを実装できます：

```flix
instance ForEach[String] {
    type Elm = Char
    pub def forEach(f: Char -> Unit \ ef, x: String): Unit \ ef = 
        x |> String.toList |> List.forEach(f)
}
```

## 例: `Collection` トレイト

別の例として、コレクションのためのトレイトを定義できます：

```flix
trait Collection[t] {
    type Elm
    pub def empty(): t
    pub def insert(x: Collection.Elm[t], c: t): t
    pub def toList(c: t): List[Collection.Elm[t]]
}
```

ここで `t` はコレクションの型であり、`Elm` はその要素の型です。すべてのコレクションは、`empty`、`insert`、`toList` という3つの操作をサポートしなければなりません。

`Vector[a]` に対する `Collection` のインスタンスを実装できます：

```flix
instance Collection[Vector[a]] {
    type Elm = a
    pub def empty(): Vector[a] = Vector.empty()
    pub def insert(x: a, c: Vector[a]): Vector[a] = Vector.append(c, Vector#{x})
    pub def toList(c: Vector[a]): List[a] = Vector.toList(c)
}
```

そして、`Set[a]` に対する `Collection` のインスタンスも実装できます：

```flix
instance Collection[Set[a]] with Order[a] {
    type Elm = a
    pub def empty(): Set[a] = Set.empty()
    pub def insert(x: a, c: Set[a]): Set[a] = Set.insert(x, c)
    pub def toList(c: Set[a]): List[a] = Set.toList(c)
}
```

## 等値制約

多相関数を書くとき、関連型を_制限_したい場合があります。

例えば、先ほどの `Collection` トレイトの例に戻ると、要素型が `Int32` であることを要求する関数を書くことができます。これにより、合計を計算する関数を書けるようになります：

```flix
def sum(c: t): Int32 with Collection[t] where Collection.Elm[t] ~ Int32 = 
    Collection.toList(c) |> List.sum
```

ここで `where` 節には、_型の等値制約(Type equality constraint)_のリストが含まれています。具体的には、等値制約 `Collection.Elm[t] ~ Int32` は、`Collection` のインスタンスが存在する任意の型 `t` について、そのインスタンスの要素型が `Int32` に等しい限り、`sum` を使用できることを表明しています。この制限により、コレクションの要素が整数であることが保証され、`List.sum` を呼び出せるようになります。

## デフォルト型

関連型にデフォルト型を定義できます。

`Addable` に戻ると、関連型 `Rhs` のデフォルトを `t` として定義できます：

```flix
trait Addable[t] {
    type Rhs = t  // デフォルト型を持つ関連型
    pub def add(x: t, y: Addable.Rhs[t]): t
}
```

ここでは、インスタンスの実装で `Rhs` が定義されていない場合、デフォルトで `t` になることを指定しています。その結果、`Int32` のインスタンスを次のように定義できます：

```flix
instance Addable[Int32] {
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

`type Rhs = Int32` を明示的に定義する必要はありません。

<!--
# Associated Types

An associated type is a type member of a trait that is specified by each trait
instance. Associated types are often considered a more natural alternative to
[multi-parameter type classes](https://en.wikipedia.org/wiki/Type_class#Multi-parameter_type_classes). 

We illustrate associated types with an example. 

We can define a trait for types that can be added:

```flix
trait Addable[t] {
    pub def add(x: t, y: t): t
}
```

We can implement multiple instances of the `Addable` trait for types such as
floating-point numbers, integers, and strings. For example, here is the instance
for `Int32`:

```flix
instance Addable[Int32] {
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

and here is one for `String`:

```flix
instance Addable[String] {
    pub def add(x: String, y: String): String = "${x}${y}"
}
```

But what if we wanted to add an element to a set?

Intuitively, we would like to write:

```flix
instance Addable[Set[a]] with Order[a] {
    pub def add(s: Set[a], x: a): Set[a] = Set.insert(x, s)
}
```

But the signature of `add` does not match the signature declared in `Addable`.

We can overcome this problem and increase the flexibility of `Addable` with an
associated type: 

```flix
trait Addable[t] {
    type Rhs
    pub def add(x: t, y: Addable.Rhs[t]): t
}
```

The `Addable` trait now has an associated type called `Rhs`. We refer to it as
`Addable.Rhs[t]` as seen in the signature of `add`. Whenever we declare an
instance of `Addable`, we must specify the associated type. 

We can still implement instances for integers and strings, as before. For
example:

```flix
instance Addable[Int32] {
    type Rhs = Int32
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

But we can also implement an instance that allows adding an element to a set: 

```flix
instance Addable[Set[a]] with Order[a] {
    type Rhs = a
    pub def add(s: Set[a], x: a): Set[a] = Set.insert(x, s)
}
```

The important point is that _each trait instance specifies the associated type_.

We might wonder if we can specify two instances for `Set[a]`: (a) one for adding
an element to a set, as above, and (b) one for adding two sets:

```flix
instance Addable[Set[a]] with Order[a] {
    type Rhs = Set[a]
    pub def add(x: Set[a], y: Set[a]): Set[a] = Set.union(x, y)

}
```

But while each instance is valid on its own, we cannot have both:

```
❌ -- Instance Error -------------------------------------------------- 

>> Overlapping instances for 'Addable'.

...
```

If we had such overlapping instances, an expression like `Addable.add(Set#{},
Set#{})` would become ambiguous: Are we adding two sets? Or are we adding the
empty set to a set? 

## Example: A `ForEach` Trait

We can use associated types to define a trait for collections that have a
`forEach` function: 

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

Here `t` is the type of the collection and the associated type `Elm` is the type
of its elements. We can implement several instances for `ForEach`. For example,
we can implement an instance for `List[a]`:

```flix
instance ForEach[List[a]] {
    type Elm = a
    pub def forEach(f: a -> Unit \ ef, x: List[a]): Unit \ ef = List.forEach(f, x)
}
```

We can also implement an instance for `Map[k, v]`:

```flix
instance ForEach[Map[k, v]] {
    type Elm = (k, v)
    pub def forEach(f: ((k, v)) -> Unit \ ef, x: Map[k, v]): Unit \ ef = 
        Map.forEach(k -> v -> f((k, v)), x)
}
```

What is interesting and useful is that we can define the element type to be
key-value pairs. We need extra parentheses around the argument to `f` because we
want it to take a pair. 

We can implement an instance for `String` where we can iterate through each
individual character: 

```flix
instance ForEach[String] {
    type Elm = Char
    pub def forEach(f: Char -> Unit \ ef, x: String): Unit \ ef = 
        x |> String.toList |> List.forEach(f)
}
```

## Example: A `Collection` Trait

As another example, we can define a trait for collections:

```flix
trait Collection[t] {
    type Elm
    pub def empty(): t
    pub def insert(x: Collection.Elm[t], c: t): t
    pub def toList(c: t): List[Collection.Elm[t]]
}
```

Here `t` is the type of the collection and `Elm` is the type of its elements.
Every collection must support three operations: `empty`, `insert`, and `toList`. 

We can implement an instance of `Collection` for `Vector[a]`: 

```flix
instance Collection[Vector[a]] {
    type Elm = a
    pub def empty(): Vector[a] = Vector.empty()
    pub def insert(x: a, c: Vector[a]): Vector[a] = Vector.append(c, Vector#{x})
    pub def toList(c: Vector[a]): List[a] = Vector.toList(c)
}
```

And we can implement an instance of `Collection` for `Set[a]`: 

```flix
instance Collection[Set[a]] with Order[a] {
    type Elm = a
    pub def empty(): Set[a] = Set.empty()
    pub def insert(x: a, c: Set[a]): Set[a] = Set.insert(x, c)
    pub def toList(c: Set[a]): List[a] = Set.toList(c)
}
```

## Equality Constraints

We sometimes want to write polymorphic functions where we _restrict_ an
associated type. 

For example, returning to the example of the `Collection` trait, we can write a
function where we require that the element type is an `Int32`. This allows us to
write a sum function:

```flix
def sum(c: t): Int32 with Collection[t] where Collection.Elm[t] ~ Int32 = 
    Collection.toList(c) |> List.sum
```

Here the `where` clause contains a list of _type equality constraints_.
Specifically, the equality constraint `Collection.Elm[t] ~ Int32` assert that
`sum` can be used with any type `t` for which there is an instance of
`Collection` as long as the element type of that instance is equal to `Int32`.
This restriction ensures that the elements of the collection are integers and
allows us to call `List.sum`.

## Default Types

We can define a default type for an associated type.

Returning to `Addable`, we can define the associated type `Rhs` with `t` as its
default:

```flix
trait Addable[t] {
    type Rhs = t  // Associated type with default type.
    pub def add(x: t, y: Addable.Rhs[t]): t
}
```

Here we specify that if `Rhs` is not defined by an instance implementation then
it defaults to `t`. The upshot is that we can define an instance for `Int32`:

```flix
instance Addable[Int32] {
    pub def add(x: Int32, y: Int32): Int32 = x + y
}
```

without having to explicit define `type Rhs = Int32`.
-->
