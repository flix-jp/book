# 必須のトレイト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/essential-traits.html)を参照してください。

Flix で実践的なプログラミングを行うには、少なくとも 3 つのトレイト(Trait)、すなわち `Eq`、`Order`、`ToString` についての知識が必要です。

## Eq トレイト

`Eq` トレイトは、ある特定の型の 2 つの値が等しいのはどのようなときかを表現します：

```flix
trait Eq[a] {

    ///
    /// `x` が `y` と等しい場合、かつその場合に限り `true` を返します。
    ///
    pub def eq(x: a, y: a): Bool

    // ... その他のメンバーは省略 ...
}
```

`Eq` を実装するには、`eq` 関数を実装するだけで済みます。`eq` を実装すると、`Eq.neq` の実装が自動的に得られます。

## Order トレイト

`Order` トレイトは、ある値が同じ型の別の値以下であるのはどのようなときかを表現します：

```flix
trait Order[a] with Eq[a] {

    ///
    /// `x` < `y` の場合は `Comparison.LessThan` を、
    /// `x` == `y` の場合は `Equal` を、
    /// `x` > `y` の場合は `Comparison.GreaterThan` を返します。
    ///
    pub def compare(x: a, y: a): Comparison

    // ... その他のメンバーは省略 ...
}
```

`Order` トレイトを実装するには、`Comparison` 型の値を返す `compare` 関数を実装しなければなりません。`Comparison` データ型は次のように定義されています：

```flix
enum Comparison {
    case LessThan
    case EqualTo
    case GreaterThan
}
```

`compare` を実装すると、`Order.less`、`Order.lessThan`、`Order.greater`、`Order.greaterEqual`、`Order.max`、`Order.min` の実装が自動的に得られます。

## ToString トレイト

`ToString` トレイトは、特定の値の文字列表現を得るために使用されます：

```flix
trait ToString[a] {
    ///
    /// 与えられた `x` の文字列表現を返します。
    ///
    pub def toString(x: a): String
}
```

Flix は文字列補間(String interpolation)において `ToString` トレイトを使用します。

例えば、次の補間された文字列

```flix
"Good morning ${name}, it is ${hour} o'clock."
```

は、実際には次の式に対する糖衣構文です：

```flix
"Good morning " + ToString.toString(name) + ", it is " 
                + ToString.toString(hour) + " o'clock."
```

続くサブセクションでは、`Eq`、`Order`、`ToString` トレイトの実装を自動的に導出する方法について説明します。

<!--
# Essential Traits

Practical programming in Flix requires knowledge of at least three traits: `Eq`,
`Order`, and `ToString`. 

## The Eq Trait

The `Eq` trait captures when two values of a specific type are equal:

```flix
trait Eq[a] {

    ///
    /// Returns `true` if and only if `x` is equal to `y`.
    ///
    pub def eq(x: a, y: a): Bool

    // ... additional members omitted ...
}
```

To implement `Eq`, we only have to implement the `eq` function. When we
implement `eq` we automatically get an implementation of `Eq.neq`.

## The Order Trait

The `Order` trait captures when one value is smaller or equal to another value
of the same type:

```flix
trait Order[a] with Eq[a] {

    ///
    /// Returns `Comparison.LessThan` if `x` < `y`, 
    /// `Equal` if `x` == `y` or 
    /// `Comparison.GreaterThan` if `x` > `y`.
    ///
    pub def compare(x: a, y: a): Comparison

    // ... additional members omitted ...
}
```

To implement the `Order` trait, we must implement the `compare` function which
returns value of type `Comparison`. The `Comparison` data type is defined as:

```flix
enum Comparison {
    case LessThan
    case EqualTo
    case GreaterThan
}
```

When we implement `compare`, we automatically get implementations of
`Order.less`, `Order.lessThan`, `Order.greater`, `Order.greaterEqual`,
`Order.max`, and `Order.min`.

## The ToString Trait

The `ToString` trait is used to obtain a string representation of a specific value:

```flix
trait ToString[a] {
    ///
    /// Returns a string representation of the given `x`.
    ///
    pub def toString(x: a): String
}
```

Flix uses the `ToString` trait in string interpolations. 

For example, the interpolated string

```flix
"Good morning ${name}, it is ${hour} o'clock."
```

is actually syntactic sugar for the expression:

```flix
"Good morning " + ToString.toString(name) + ", it is " 
                + ToString.toString(hour) + " o'clock."
```

In the following subsection, we discuss how to automatically derive
implementations of the `Eq`, `Order`, and `ToString` traits.
-->
