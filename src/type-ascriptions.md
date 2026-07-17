# 型注釈

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/type-ascriptions.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/type-ascriptions.md)ください。

Flix はローカル型推論をサポートしていますが、式や let 束縛にその型を注釈しておくと便利な場合があります。このような注釈を型注釈(Type ascription)と呼びます。型注釈によって式の型を変えることはできず、型安全性を破るために使うこともできません。

型注釈は式の後ろに置くことができます：

```flix
(("Hello" :: "World" :: Nil) : List[String])
```

ただし、他の式と区別するために括弧で囲む必要があります。

let 束縛に対しては、括弧なしで型注釈を置くこともできます：

```flix
let l: List[String] = "Hello" :: "World" :: Nil
```

## カインド注釈

Flix はカインド注釈(Kind ascription)もサポートしています。型注釈が_式_の_型_を指定するのに対して、カインド注釈は_型_の_カインド_を指定します。

カインド注釈は型パラメータに対して使うことができます。例えば：

```flix
def fst1[a: Type, b: Type](p: (a, b)): a = let (x, _) = p; x
```

ここでは、2 つの型パラメータ `a` と `b` の_カインド_が `Type` であることを指定しています。このようなカインドは推論できるため、通常は指定する必要はありません。

代数的データ型にカインド注釈を付けることもできます：

```flix
enum A[t: Type] {
    case A(t, t)
}
```

トレイトにも付けられます：

```flix
trait MyTrait[t: Type] {
    // ...
}
```

カインド注釈は通常、[高カインド型](./higher-kinded-types.md)に対してのみ使用します。

<!--
# Type Ascriptions

While Flix supports local type inference, it can sometimes be useful to annotate
an expression or a let-binding with its type. We call such annotations *type
ascriptions*. A type ascription cannot change the type of an expression nor can
it be used to violate type safety.

A type ascription can be placed after an expression:

```flix
(("Hello" :: "World" :: Nil) : List[String])
```

but it must be wrapped in parentheses to disambiguate it from other expressions.

It can also be placed on a let-binding without parentheses:

```flix
let l: List[String] = "Hello" :: "World" :: Nil
```
## Kind Ascriptions

Flix also supports kind ascriptions. Where a type ascription specifies the
_type_ of an _expression_, a kind ascription specifies the _kind_ of a _type_.

We can use kind ascriptions on type parameters. For example:

```flix
def fst1[a: Type, b: Type](p: (a, b)): a = let (x, _) = p; x
```

Here we have specified that the _kind_ of the two type parameters `a` and `b` is
`Type`. We will typically never have to specify such kinds since they can
be inferred.

We can also provide kind ascriptions on algebraic data types:

```flix
enum A[t: Type] {
    case A(t, t)
}
```

and on traits:

```flix
trait MyTrait[t: Type] {
    // ...
}
```

We typically only use kind ascriptions for [higher-kinded
types](./higher-kinded-types.md).
-->
