# 用語集

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/glossary.html)を参照してください。

***Algebraic Data Type（代数的データ型）.*** 直和型と直積型、すなわち列挙型とタプル型を用いて定義されるデータ型です。

***Algebraic Effect（代数エフェクト）.***  <a name="associated-effect"></a> ハンドリング可能な、ユーザー定義のエフェクトです。ハンドラには、そのエフェクトの（限定）継続が渡されます。継続は、破棄することも、一度だけ再開することも、複数回再開することもできます。

***Associated Type（関連型）.***  <a name="associated-type"></a> トレイトに属する型です。各トレイトインスタンスは、そのインスタンスにおける具体的な関連型を指定します。したがって、異なるトレイトインスタンスは異なる関連型を持つことができます。

***Associated Effect（関連エフェクト）.*** トレイトに属するエフェクトです。各トレイトインスタンスは、そのインスタンスにおける具体的な関連エフェクトを指定します。したがって、異なるトレイトインスタンスは異なる関連エフェクトを持つことができます。

***Checked Cast（検査付きキャスト）.*** コンパイラが正しさを保証する安全なキャストです。実行時に失敗することはありません。

***Effect（エフェクト）.*** Flix は3種類のエフェクトをサポートしています。組み込みエフェクト（例：`IO` や `NonDet`）、リージョンベースのエフェクト、そしてユーザー定義エフェクトです。

***Effect Cast（エフェクトキャスト）.*** 式の*エフェクト*を変更するキャストです。

***Effect Member（エフェクトメンバ）.*** [関連エフェクト](#associated-effect)を参照してください。

***Effect Polymorphic（エフェクト多相）.*** 関数引数のエフェクトに応じて、自身のエフェクトが決まる関数のことです。[高階関数](#higher-order-function)も参照してください。

***Effect Handler（エフェクトハンドラ）.*** ユーザー定義エフェクトをハンドリングする式です。

***Higher-Order Function（高階関数）.*** <a name="higher-order-function"></a> 関数を引数として受け取るか、関数を返す関数です。

***IO Effect（IOエフェクト）.*** 外部世界とのあらゆるやり取りを表す、組み込みの汎用エフェクトです。

***Pure（純粋）.*** エフェクトを一切持たない関数（または式）のことです。

***String Interpolation（文字列補間）.*** 文字列の中に式を含めることを可能にする言語機能です。

***Tail Call（末尾呼び出し）.*** 末尾位置にある関数呼び出しのことで、追加のスタック領域を必要としません。

***Trait（トレイト）.*** 関数シグネチャとデフォルト関数の集まりを規定するインターフェースです。1つのトレイトは複数のデータ型によって実装できます。Flix におけるトレイトは型クラスです。

***Type Class（型クラス）.*** Trait を参照してください。

***Type Cast（型キャスト）.*** 式の*型*を変更するキャストです。

***Type Inference（型推論）.*** プログラマによる注釈を必要とせずに、コンパイラが式の型を推論できるようにする言語機能です。

***Type Member（型メンバ）.*** [関連型](#associated-type)を参照してください。

***Unchecked Cast（未検査キャスト）.*** コンパイラによって検証されない、安全でないキャストです。実行時に失敗する可能性があります。

***Uninterpretable Effect（解釈不能エフェクト）.*** ハンドリングできない（またはすべきでない）エフェクトです。例：`IO`。

<!--
# Glossary

***Algebraic Data Type.*** A data type defined using sum and product types, i.e.
using enumerated types and tuple types.

***Algebraic Effect.***  <a name="associated-effect"></a> A user-defined effect
which can be handled. The handler is supplied with the (delimited) continuation
of the effect. The continuation can be dropped, resume once, or resumed
multiple-times.

***Associated Type.***  <a name="associated-type"></a> A type that belongs to a
trait. Each trait instance specifies the specific associated type for that
instance. Hence different trait instances can have different associated types.

***Associated Effect.*** An effect that belongs to a trait. Each trait instance
specifies the specific associated effect for that instance. Hence different
trait instances can have different associated effects.

***Checked Cast.*** A safe cast that the compiler ensures is correct. Cannot
fail at runtime. 

***Effect.*** Flix supports three types of effects: built-in effects (e.g.
`IO` and `NonDet`), region-based effects, and user-defined effects.

***Effect Cast.*** A cast that changes the _effect_ of an expression.

***Effect Member.*** See [associated effect](#associated-effect).

***Effect Polymorphic.*** A function whose effect(s) depend on the effect(s) of
its function argument. See also [higher-order function](#higher-order-function).

***Effect Handler.*** An expression which handles a user-defined effect.

***Higher-Order Function.*** <a name="higher-order-function"></a> A function
that takes a function argument or returns a function.

***IO Effect.*** A built-in generic effect which represents any interaction with
the outside world. 

***Pure.*** A function (or expression) which has no effects.

***String Interpolation.*** A language feature that allows a string to contain
expressions. 

***Tail Call.*** A function call in tail-position and hence does not require
additional stack space. 

***Trait.*** An interface that specifies a collection of function signatures and
default functions. A trait can be implemented by several data types. Traits in
Flix are type classes. 

***Type Class.*** See Trait. 

***Type Cast.*** A cast that changes the _type_ of an expression.

***Type Inference.*** A language feature that allows the compiler to infer the
type of an expression without requiring annotations from the programmer. 

***Type Member.*** See [associated type](#associated-type).

***Unchecked Cast.*** An unsafe cast which is not verified by the compiler. Can
fail at runtime. 

***Uninterpretable Effect.*** An effect that cannot (or should not) be handled, e.g. `IO`.
-->
