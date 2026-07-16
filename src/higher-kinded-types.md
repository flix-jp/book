# 高カインド型

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/higher-kinded-types.html)を参照してください。

Flix は[高カインド型(Higher-kinded types)](https://en.wikipedia.org/wiki/Kind_(type_theory))をサポートしています。そのため、トレイトは*型コンストラクタ(Type constructor)*を抽象化できます。

例えば、`t[a]` という形をした任意のコレクションに対する反復処理を表現するトレイトを書くことができます。ここで、`t` はカインド `Type -> Type` の型コンストラクタであり、`a` はカインド `Type` の要素型です：

```flix
trait ForEach[t: Type -> Type] {
    pub def forEach(f: a -> Unit \ ef, x: t[a]): Unit \ ef
}
```

高カインド型を使うには、Flix はカインド注釈(Kind annotation)を書くことを*要求*します。つまり、`ForEach` が型コンストラクタを抽象化していることを Flix に伝えるために、`t: Type -> Type` と書く必要がありました。

`ForEach` トレイトのインスタンスを `Option` に対して実装できます：

```flix
instance ForEach[Option] {
    pub def forEach(f: a -> Unit \ ef, o: Option[a]): Unit \ ef = match o {
        case None    => ()
        case Some(x) => f(x)
    }
}
```

また、`List` に対するインスタンスも実装できます：

```flix
instance ForEach[List] {
    pub def forEach(f: a -> Unit \ ef, l: List[a]): Unit \ ef = List.forEach(f, l)
}
```

## Flix のカインド

Flix は以下のカインドをサポートしています：

- `Type`: Flix の型のカインド。
    - 例：`Int32`、`String`、`List[Int32]`。
- `RecordRow`: レコードで使われる行のカインド。
    - 例：`{x = Int32, y = Int32 | r}` において、型変数 `r` はカインド `RecordRow` を持ちます。
- `SchemaRow`: 第一級 Datalog 制約で使われる行のカインド。
    - 例：`#{P(Int32, Int32) | r}` において、型変数 `r` はカインド `SchemaRow` を持ちます。

Flix は通常、カインドを推論できます。例えば、次のように書くと：

```flix
def sum(r: {x = t, y = t | r}): t with Add[t] = r#x + r#y
```

`t: Type` と `r: RecordRow` というカインドが自動的に推論されます。

次のように明示的に指定することもできます：

```flix
def sum[t: Type, r: RecordRow](r: {x = t, y = t | r}): t with Add[t] = r#x + r#y
```

しかし、このスタイルは慣用的とは見なされません。

Flix が明示的なカインド注釈を要求するのは、次の 4 つの状況です：

- enum 上の、`Type` 以外のカインドを持つ型パラメータ。
- 型エイリアス上の、`Type` 以外のカインドを持つ型パラメータ。
- トレイト上の、`Type` 以外のカインドを持つ型パラメータ。
- トレイト内の、`Type` 以外のカインドを持つ型メンバー。

カインド注釈が必要になる最も一般的なシナリオは、型パラメータや型メンバーをエフェクトの範囲で動かしたい場合です。

## 高カインド型と関連型の比較

実際のところ、高カインド型と関連型は、よく似た抽象を定義するために使うことができます。

例えば、これまで見てきたように、`ForEach` トレイトは 2 つの異なる方法で定義できます。

*高カインド型*を使う方法：

```flix
trait ForEach[t: Type -> Type] {
    pub def forEach(f: a -> Unit \ ef, x: t[a]): Unit \ ef
}
```

そして、*関連型*を使う方法：

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

`ForEach` の場合は、関連型を使った定義の方が柔軟です。関連する要素型を `Char` として、`String` に対するインスタンスを実装できるからです。しかし、高カインド型も依然として有用です。例えば、Flix 標準ライブラリは `Functor` トレイトを次のように定義しています：

```flix
trait Functor[m : Type -> Type] {
    pub def map(f: a -> b \ ef, x: m[a]): m[b] \ ef
}
```

注目すべきは、`m` のカインドによって、すべての `Functor` 実装が構造を保存することが保証される点です。つまり、例えば `Option[a]` に対して `map` を適用すると、必ず `Option[b]` が返ってくることが分かります。

<!--
# Higher-Kinded Types

Flix supports [higher-kinded
types](https://en.wikipedia.org/wiki/Kind_(type_theory)), hence traits can
abstract over _type constructors_.

For example, we can write a trait that capture iteration over any
collection of the shape `t[a]` where `t` is a type constructor of kind
 `Type -> Type` and `a` is the element type of kind `Type`:

```flix
trait ForEach[t: Type -> Type] {
    pub def forEach(f: a -> Unit \ ef, x: t[a]): Unit \ ef
}
```

To use higher-kinded types Flix _requires_ us to provide kind annotations, i.e.
we had to write `t: Type -> Type` to inform Flix that `ForEach` abstracts over
type constructors.

We can implement an instance of the `ForEach` trait for `Option`:

```flix
instance ForEach[Option] {
    pub def forEach(f: a -> Unit \ ef, o: Option[a]): Unit \ ef = match o {
        case None    => ()
        case Some(x) => f(x)
    }
}
```

and we can implement an instance for `List`:

```flix
instance ForEach[List] {
    pub def forEach(f: a -> Unit \ ef, l: List[a]): Unit \ ef = List.forEach(f, l)
}
```

## The Flix Kinds

Flix supports the following kinds:

- `Type`: The kind of Flix types.
    - e.g. `Int32`, `String`, and `List[Int32]`.
- `RecordRow`: The kind of rows used in records
    - e.g. in `{x = Int32, y = Int32 | r}` the type variable `r` has kind `RecordRow`.
- `SchemaRow`: The kind of rows used in first-class Datalog constraints
    - e.g. in `#{P(Int32, Int32) | r}` the type variable `r` has kind `SchemaRow`.

Flix can usually infer kinds. For example, we can write:

```flix
def sum(r: {x = t, y = t | r}): t with Add[t] = r#x + r#y
```

and have the kinds of `t: Type` and `r: RecordRow` automatically inferred.

We can also explicitly specify them as follows:

```flix
def sum[t: Type, r: RecordRow](r: {x = t, y = t | r}): t with Add[t] = r#x + r#y
```

but this style is not considered idiomatic.

Flix requires explicit kind annotations in four situations:

- For type parameters of non-Type kind on enums.
- For type parameters of non-Type kind on type aliases.
- For type parameters of non-Type kind on traits.
- For type members of a non-Type kind in a trait.

The most common scenario where you will need a kind annotation is when you want
a type parameter or type member to range over an effect.

## Higher-Kinded Types vs. Associated Types

In practice higher-kinded types and associated types can be used to define
similar abstractions.

For example, as we have seen, we can define the `ForEach` trait in two different
ways:

With a _higher-kinded type_:

```flix
trait ForEach[t: Type -> Type] {
    pub def forEach(f: a -> Unit \ ef, x: t[a]): Unit \ ef
}
```

and with an _associated type_:

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

In the case of `ForEach`, the definition with an associated type is more
flexible, since we can implement an instance for `String` with associated
element type `Char`. However, higher-kinded types are still useful. For example,
the Flix Standard Library defines the `Functor` trait as:

```flix
trait Functor[m : Type -> Type] {
    pub def map(f: a -> b \ ef, x: m[a]): m[b] \ ef
}
```

Notably the kind of `m` ensures that every `Functor` implementation is structure
preserving. That is, we know that when we `map` over e.g. an `Option[a]` then we
get back an `Option[b]`.
-->
