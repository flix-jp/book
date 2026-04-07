# データ型

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/data-types.html)を参照してください。

Flix には、ブール値、浮動小数点数、整数などの組み込みデータ型と、タプルやレコードなどの複合型が用意されています。
さらに、標準ライブラリでは `Option[a]`、`Result[e, t]`、`List[a]`、`Set[a]`、`Map[k, v]` などの型が定義されています。

これらの型に加えて、Flix ではプログラマが独自の型を定義できます。*列挙型*、*再帰型*、*多相型*などを定義することが可能です。

Flix は型エイリアス（新しい型）もサポートしています。

<!--
# Data Types

Flix comes with a collection of built-in data types,
such as booleans, floats and integers, and
compound types, such as tuples and records.
Moreover, the standard library defines types such as
`Option[a]`, `Result[e, t]`, `List[a]`, `Set[a]`,
and `Map[k, v]`.

In addition to these types, Flix allows programmers
to define their own types, including *enumerated
types*, *recursive types*, and *polymorphic types*.

Flix also supports type aliases (new types).
-->
