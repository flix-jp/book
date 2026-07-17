# トレイト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/traits.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/traits.md)ください。

トレイト(Trait)は[型クラス](https://en.wikipedia.org/wiki/Type_class)としても知られており、抽象化とモジュール性を支える仕組みです。Flix のトレイトシステムは Haskell や Rust のものと似ていますが、同一ではありません。Flix のトレイトは、関連型、関連エフェクト、高カインド型をサポートしています。

例を使ってトレイトを説明します。

`Option[Int32]` に対する等価性は、次のように定義できます：

```flix
def equals(x: Option[Int32], y: Option[Int32]): Bool = 
    match (x, y) {
        case (None, None)         => true
        case (Some(v1), Some(v2)) => v1 == v2
        case _                    => false
    }
```

同様に、`List[Int32]` に対する等価性も次のように定義できます：

```flix
def equals(x: List[Int32], y: List[Int32]): Bool = 
    match (x, y) {
        case (Nil, Nil)           => true
        case (v1 :: xs, v2 :: ys) => v1 == v2 and equals(xs, ys)
        case _                    => false
    }
```

しかし、等価性をサポートするデータ型に対する共通の抽象化が欲しい場合はどうすればよいでしょうか？

ここでトレイトの出番です。`Equatable` トレイトを次のように定義できます：

```flix
trait Equatable[t] {
    pub def equals(x: t, y: t): Bool
}
```

このトレイトは、単一の `equals` という*トレイトシグネチャ*を持ちます。このトレイトは型パラメータ `t` について多相的であるため、`Option[t]` と `List[t]` の両方に対して `Equatable` を実装できます：

```flix
instance Equatable[Option[t]] with Equatable[t] {
    pub def equals(x: Option[t], y: Option[t]): Bool = 
        match (x, y) {
            case (None, None)         => true
            case (Some(v1), Some(v2)) => Equatable.equals(v1, v2)
            case _                    => false
        }
}
```

ここで注目してほしいのは、`Equatable` を `Option[Int32]` に対して実装したのではなく、`t` 自体が等価比較可能である限り*任意の* `Option[t]` に対して実装したという点です。さらに、`v1` と `v2` を `==` で直接比較する代わりに、それらに対して `Equatable.equals` を呼び出しています。

`List[t]` に対しても `Equatable` を実装できます：

```flix
instance Equatable[List[t]] with Equatable[t] {
    pub def equals(x: List[t], y: List[t]): Bool = 
        use Equatable.equals;
        match (x, y) {
            case (Nil, Nil)           => true
            case (v1 :: xs, v2 :: ys) => equals(v1, v2) and equals(xs, ys)
            case _                    => false
        }
}
```

`Int32` に対しても `Equatable` を実装したと仮定すると、`Equatable` を使って 2 つの `Option[Int32]` 値が等しいかどうかを計算できます。それだけでなく、2 つの `Option[List[Int32]]` 値が等しいかどうかも計算できるのです！これは抽象化の力を示しています。`Option[t]` と `List[t]` に対するインスタンス(instance)を一度実装すれば、それらのインスタンスをあらゆる場所で再利用できます。

新しく定義した `Equatable` トレイトを使って、多相的な関数を書くことができます。

例えば、ある要素がリストに含まれているかどうかを計算する関数を定義できます：

```flix
def memberOf(x: t, l: List[t]): Bool with Equatable[t] = 
    match l {
        case Nil     => false
        case y :: ys => Equatable.equals(x, y) or memberOf(x, ys)
    }
```

要素の型が `Equatable` を実装してさえいれば、任意の型のリストに対して `memberOf` を使うことができます。

> **注意:** Flix 標準ライブラリでは、`Equatable` トレイトは `Eq` という名前です。さらに、`==` 演算子はトレイトシグネチャ `Eq.eq` の糖衣構文です。

## Sealed トレイト

トレイトを `sealed` として宣言すると、そのトレイトを誰が実装できるかを制限できます。

例えば：

```flix
mod Zoo {
    sealed trait Animal[a] {
        pub def isMammal(x: a): Bool
    }

    instance Animal[Giraffe] {
        pub def isMammal(_: Giraffe): Bool = true
    }

    instance Animal[Penguin] {
        pub def isMammal(_: Penguin): Bool = false
    }

    pub enum Giraffe
    pub enum Penguin
}
```

ここでは `Animal` と `Giraffe` のインスタンスを実装できます。それらが `Animal` トレイトと同じモジュール内に存在するからです。しかし、`Zoo` モジュールの外側から `Animal` を実装することはできません。試しに次のように書くと：

```flix
mod Lake {
    pub enum Swan

    instance Zoo.Animal[Swan] {
        pub def isMammal(_: Swan): Bool = false
    }
}
```

Flix は次のように報告します：

```
❌ -- Resolution Error -------------------------------------------------- 

>> Trait 'Zoo.Animal' is sealed from the module 'Lake'.

21 |     instance Zoo.Animal[Swan] {
                  ^^^^^^^^^^
                  sealed trait.
```


## 不正な形のトレイト

トレイトは C\# や Java スタイルのインターフェースでは*ありません*。具体的には：

- すべてのトレイトはちょうど 1 つの型パラメータを持たなければならず、
- すべてのシグネチャはその型パラメータに言及しなければなりません。

例えば、次のトレイトは正しくありません：

```flix
trait Animal[a] {
    pub def isMammal(x: a): Bool      // OK     -- a に言及している。
    pub def numberOfGiraffes(): Int32 // NG     -- a に言及していない。
}
```

上記のトレイトをコンパイルすると、Flix は次のように報告します：

```
❌ -- Resolution Error -------------------------------------------------- 

>> Unexpected signature 'numberOfGiraffes' which does not mention the type 
>> variable of the trait.

7 |     pub def numberOfGiraffes(): Int32 
                ^^^^^^^^^^^
                unexpected signature.
```

問題は、`numberOfGiraffes` のシグネチャが型パラメータ `a` に言及していないことです。

## 複雑なインスタンス

トレイトの*インスタンス*は、次の条件を満たす型に対して定義しなければなりません：

- ちょうど 1 つの型コンストラクタであり、
- それが 0 個以上の相異なる型変数に適用されていること。

例えば、先ほどの `Equatable` トレイトがあるとします：

```flix
trait Equatable[t] {
    pub def equals(x: t, y: t): Bool
}
```

次のような型に対してはインスタンスを実装できます：

- `Option[a]`
- `List[a]`
- `(a, b)`

しかし、次のような型に対してはインスタンスを実装*できません*：

- `Option[Int32]`
- `List[String]`
- `(a, Bool)` 
- `Map[Int32, v]`

例えば `List[Int32]` に対してインスタンスを実装しようとすると、Flix は次のように報告します：

```
❌ -- Instance Error -------------------------------------------------- 

>> Complex instance type 'List[Int32]' in 'Equatable'.

6 | instance Equatable[List[Int32]] {
             ^^^^^^^^^
             complex instance type

An instance type must be a type constructor applied to zero or more 
distinct type variables.
```

## 重複するインスタンス

重複する型に対して、同じトレイトのインスタンスを 2 つ実装することはできません。

例えば、`List[t]` に対して `Equatable` のインスタンスを 2 つ実装しようとすると：

```flix
instance Equatable[List[t]] {
    pub def equals(x: List[t], y: List[t]): Bool = ???
}

instance Equatable[List[t]] {
    pub def equals(x: List[t], y: List[t]): Bool = ???
}
```

Flix は次のように報告します：

```
❌ -- Instance Error -------------------------------------------------- 

>> Overlapping instances for 'Equatable'.

1 | instance Equatable[List[t]] {
              ^^^^^^^^^
              the first instance was declared here.

4 | instance Equatable[List[t]] {
             ^^^^^^^^^
             the second instance was declared here.
```

<!--
# Traits

Traits, also known as [type classes](https://en.wikipedia.org/wiki/Type_class),
support abstraction and modularity. The Flix trait system is similar to that of
Haskell and Rust, but not identical. Traits in Flix support associated types,
associated effects, and higher-kinded types. 

We illustrate traits with an example.

We can define equality on `Option[Int32]` as follows:

```flix
def equals(x: Option[Int32], y: Option[Int32]): Bool = 
    match (x, y) {
        case (None, None)         => true
        case (Some(v1), Some(v2)) => v1 == v2
        case _                    => false
    }
```

We can also define equality on `List[Int32]` as follows:

```flix
def equals(x: List[Int32], y: List[Int32]): Bool = 
    match (x, y) {
        case (Nil, Nil)           => true
        case (v1 :: xs, v2 :: ys) => v1 == v2 and equals(xs, ys)
        case _                    => false
    }
```

But what if we wanted a common abstraction for data types which support
equality? 

Here we can use traits. We can define an `Equatable` trait:

```flix
trait Equatable[t] {
    pub def equals(x: t, y: t): Bool
}
```

which has a single `equals` _trait signature_. The trait is polymorphic over the
type parameter `t` which means that we can implement `Equatable` for both
`Option[t]` and `List[t]`: 

```flix
instance Equatable[Option[t]] with Equatable[t] {
    pub def equals(x: Option[t], y: Option[t]): Bool = 
        match (x, y) {
            case (None, None)         => true
            case (Some(v1), Some(v2)) => Equatable.equals(v1, v2)
            case _                    => false
        }
}
```

Notice that we did not implement `Equatable` for `Option[Int32]`, but instead
for _any_ `Option[t]` as long as `t` itself is equatable. Moreover, instead of
comparing `v1` and `v2` directly using `==`, we call `Equatable.equals` on them. 

We can also implement `Equatable` for `List[t]`:

```flix
instance Equatable[List[t]] with Equatable[t] {
    pub def equals(x: List[t], y: List[t]): Bool = 
        use Equatable.equals;
        match (x, y) {
            case (Nil, Nil)           => true
            case (v1 :: xs, v2 :: ys) => equals(v1, v2) and equals(xs, ys)
            case _                    => false
        }
}
```

Assuming we also implement `Equatable` for `Int32`, we can use `Equatable` to
compute whether two `Option[Int32]` values are equal. But we can also compute if
two `Option[List[Int32]]` values are equal! This demonstrates the power of
abstraction: We have implemented instances for `Option[t]` and `List[t]` and we
can now reuse these instances everywhere. 

We can use our newly defined `Equatable` trait to write polymorphic functions.

For example, we can define a function to compute if an element occurs in a list:

```flix
def memberOf(x: t, l: List[t]): Bool with Equatable[t] = 
    match l {
        case Nil     => false
        case y :: ys => Equatable.equals(x, y) or memberOf(x, ys)
    }
```

We can use `memberOf` for a list of any type, as the element type implements
`Equatable`.

> **Note:** In the Flix Standard Library the `Equatable` trait is called `Eq`.
> Moreover, the `==` operator is syntactic sugar for the trait signature
> `Eq.eq`.

## Sealed Traits

We can declare a trait as `sealed` to restrict who can implement the trait.

For example:

```flix
mod Zoo {
    sealed trait Animal[a] {
        pub def isMammal(x: a): Bool
    }

    instance Animal[Giraffe] {
        pub def isMammal(_: Giraffe): Bool = true
    }

    instance Animal[Penguin] {
        pub def isMammal(_: Penguin): Bool = false
    }

    pub enum Giraffe
    pub enum Penguin
}
```

Here we can implement instances for `Animal` and `Giraffe` because they occur in
the same module as the `Animal` trait. But we cannot implement `Animal` from
outside the `Zoo` module. If we try: 

```flix
mod Lake {
    pub enum Swan

    instance Zoo.Animal[Swan] {
        pub def isMammal(_: Swan): Bool = false
    }
}
```

then Flix reports:

```
❌ -- Resolution Error -------------------------------------------------- 

>> Trait 'Zoo.Animal' is sealed from the module 'Lake'.

21 |     instance Zoo.Animal[Swan] {
                  ^^^^^^^^^^
                  sealed trait.
```


## Malformed Traits

A trait is _not_ a C\# or Java-style interface. Specifically:

- every trait must have exactly one type parameter, and
- every signature must mention that type parameter.

For example, the following trait is incorrect:

```flix
trait Animal[a] {
    pub def isMammal(x: a): Bool      // OK     -- mentions a.
    pub def numberOfGiraffes(): Int32 // NOT OK -- does not mention a.
}
```

If we compile the above trait, Flix reports:

```
❌ -- Resolution Error -------------------------------------------------- 

>> Unexpected signature 'numberOfGiraffes' which does not mention the type 
>> variable of the trait.

7 |     pub def numberOfGiraffes(): Int32 
                ^^^^^^^^^^^
                unexpected signature.
```

The problem is that the signature for `numberOfGiraffes` does not mention the
type parameter `a`. 

## Complex Instances

A trait _instance_ must be defined on:

- exactly one type constructor —
- that is applied to zero or more distinct type variables. 

For example, given the `Equatable` trait from before:

```flix
trait Equatable[t] {
    pub def equals(x: t, y: t): Bool
}
```

We can implement instances for e.g.:

- `Option[a]`
- `List[a]`
- `(a, b)`

but we _cannot_ implement instances for e.g.:

- `Option[Int32]`
- `List[String]`
- `(a, Bool)` 
- `Map[Int32, v]`

If we try to implement an instance for e.g. `List[Int32]` Flix reports:

```
❌ -- Instance Error -------------------------------------------------- 

>> Complex instance type 'List[Int32]' in 'Equatable'.

6 | instance Equatable[List[Int32]] {
             ^^^^^^^^^
             complex instance type

An instance type must be a type constructor applied to zero or more 
distinct type variables.
```

## Overlapping Instances

We cannot implement two instances of of the same trait for overlapping types.

For example, if we try to implement two instances of `Equatable` for `List[t]`:

```flix
instance Equatable[List[t]] {
    pub def equals(x: List[t], y: List[t]): Bool = ???
}

instance Equatable[List[t]] {
    pub def equals(x: List[t], y: List[t]): Bool = ???
}
```

then Flix reports:

```
❌ -- Instance Error -------------------------------------------------- 

>> Overlapping instances for 'Equatable'.

1 | instance Equatable[List[t]] {
              ^^^^^^^^^
              the first instance was declared here.

4 | instance Equatable[List[t]] {
             ^^^^^^^^^
             the second instance was declared here.
```
-->
