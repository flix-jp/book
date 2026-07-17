# 自動導出

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/automatic-derivation.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/automatic-derivation.md)ください。

Flix は、いくつかのトレイトに対する自動導出(Automatic derivation)をサポートしています。これには以下が含まれます：

- `Eq` — 型の値に対する構造的等価性を導出します。
- `Order` — 型の値に対する全順序を導出します。
- `ToString` — 型の値に対する人間が読みやすい文字列表現を導出します。
- `Coerce` - 単純なデータ型をその基となる表現に変換します。

## Eq と Order の導出

`enum` 宣言の `with` 節を使うことで、`Eq` トレイトと `Order` トレイトのインスタンスを自動的に導出できます。例えば：

```flix
enum Shape with Eq, Order {
    case Circle(Int32)
    case Square(Int32)
    case Rectangle(Int32, Int32)
}
```

導出された実装は構造的であり、case 宣言の順序に依存します：

```flix
def main(): Unit \ IO = 
    println(Circle(123) == Circle(123)); // `true` を出力
    println(Circle(123) != Square(123)); // `true` を出力
    println(Circle(123) <= Circle(123)); // `true` を出力
    println(Circle(456) <= Square(123))  // `true` を出力
```

> **注意**: `Eq` と `Order` の自動導出には、`enum` の内部の型自身が `Eq` と
> `Order` を実装していることが必要です。

## ToString の導出

`ToString` インスタンスも自動的に導出できます：

```flix
enum Shape with ToString {
    case Circle(Int32)
    case Square(Int32)
    case Rectangle(Int32, Int32)
}
```

これにより、文字列補間を活用して次のように書けます：

```flix
def main(): Unit \ IO = 
    let c = Circle(123);
    let s = Square(123);
    let r = Rectangle(123, 456);
    println("A ${c}, ${s}, and ${r} walk into a bar.")
```

これは次のように出力します：

```
A Circle(123), Square(123), and Rectangle(123, 456) walk into a bar.
```

## Coerce の導出

`Coerce` トレイトの実装も自動的に導出できます。
`Coerce` トレイトは、単純な（case が1つの）データ型をその基となる実装に変換します。

```flix
enum Shape with Coerce {
    case Circle(Int32)
}

def main(): Unit \ IO =
    let c = Circle(123);
    println("The radius is ${coerce(c)}")
```

case が2つ以上ある enum に対して `Coerce` を導出することは*できません*。
例えば、次のように書こうとすると：

```flix
enum Shape with Coerce {
    case Circle(Int32)
    case Square(Int32)
}
```

Flix コンパイラはコンパイルエラーを出力します：

```
❌ -- Derivation Error --------------------------------------------------

>> Cannot derive 'Coerce' for the non-singleton enum 'Shape'.

1 | enum Shape with Coerce {
                    ^^^^^^
                    illegal derivation

'Coerce' can only be derived for enums with exactly one case.
```

<!--
# Automatic Derivation

Flix supports automatic derivation of several traits, including:

- `Eq` — to derive structural equality on the values of a type.
- `Order` — to derive a total ordering on the values of a type.
- `ToString` — to derive a human-readable string representation on the values of a type.
- `Coerce` - to convert simple data types to their underlying representation.

## Derivation of Eq and Order

We can automatically derive instances of the `Eq` and `Order` traits using
the `with` clause in the `enum` declaration. For example: 

```flix
enum Shape with Eq, Order {
    case Circle(Int32)
    case Square(Int32)
    case Rectangle(Int32, Int32)
}
```

The derived implementations are structural and rely on the order of the case
declarations:

```flix
def main(): Unit \ IO = 
    println(Circle(123) == Circle(123)); // prints `true`.
    println(Circle(123) != Square(123)); // prints `true`.
    println(Circle(123) <= Circle(123)); // prints `true`.
    println(Circle(456) <= Square(123))  // prints `true`.
```

> **Note**: Automatic derivation of `Eq` and `Order` requires that the inner
> types of the `enum` implement `Eq` and `Order` themselves.

## Derivation of ToString

We can also automatically derive `ToString` instances:

```flix
enum Shape with ToString {
    case Circle(Int32)
    case Square(Int32)
    case Rectangle(Int32, Int32)
}
```

Then we can take advantage of string interpolation and write:

```flix
def main(): Unit \ IO = 
    let c = Circle(123);
    let s = Square(123);
    let r = Rectangle(123, 456);
    println("A ${c}, ${s}, and ${r} walk into a bar.")
```

which prints:

```
A Circle(123), Square(123), and Rectangle(123, 456) walk into a bar.
```

## Derivation of Coerce

We can automatically derive implementations of the `Coerce` trait.
The `Coerce` trait converts a simple (one-case) data type
to its underlying implementation.

```flix
enum Shape with Coerce {
    case Circle(Int32)
}

def main(): Unit \ IO =
    let c = Circle(123);
    println("The radius is ${coerce(c)}")
```

We _cannot_ derive `Coerce` for an enum with more than one case.
For example, if we try:

```flix
enum Shape with Coerce {
    case Circle(Int32)
    case Square(Int32)
}
```

The Flix compiler emits a compiler error:

```
❌ -- Derivation Error --------------------------------------------------

>> Cannot derive 'Coerce' for the non-singleton enum 'Shape'.

1 | enum Shape with Coerce {
                    ^^^^^^
                    illegal derivation

'Coerce' can only be derived for enums with exactly one case.
```
-->
