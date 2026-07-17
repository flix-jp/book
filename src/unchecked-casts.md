# 未検査キャスト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/unchecked-casts.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/unchecked-casts.md)ください。

Flix は、*未検査キャスト*(Unchecked cast)、すなわち検査されない型キャストとエフェクトキャストもサポートしています。

## 未検査型キャスト

*未検査型キャスト*(Unchecked type cast)は、ある式が特定の型を持つことをコンパイラに指示します。

> **警告:** 型キャストは非常に危険であり、最大限の注意を払って使用しなければなりません！

Flix プログラマは、通常、未検査型キャストを使う必要はまったくないはずです。

### 例：スーパータイプへの安全なキャスト

以下の式は、`String` を `Object` にキャストします：

```flix
unchecked_cast("Hello World" as Object)
```

注：`checked_cast` 式を使う方が安全です。

### 例：Null からオブジェクト型への安全なキャスト

以下の式は、（`Null` 型の）`null` 値を `String` にキャストします：

```flix
unchecked_cast(null as String)
```

注：`checked_cast` 式を使う方が安全です。

### 例：安全でない型キャスト

以下の式は不正なキャストを含んでおり、実行時に `ClassCastException` を引き起こします：

```flix
unchecked_cast((123, 456) as Integer)
```

## エフェクトキャスト

*未検査エフェクトキャスト*(Unchecked effect cast)は、ある式が特定のエフェクトを持つことをコンパイラに指示します。

> **警告:** エフェクトキャストは極めて危険であり、細心の注意を払って使用しなければなりません！

Flix プログラマは、通常、未検査エフェクトキャストを使う必要はまったくないはずです。

### 例：安全でないエフェクトキャスト

純粋でない式を、あたかも純粋であるかのように見せかけることができます：

```flix
def main(): Unit =
    unchecked_cast(println("Hello World") as _ \ {})
```

ここでは、`IO` エフェクトを持つ `println` を呼び出したうえで、その式が純粋であるかのように見せかけて、明示的かつ安全でない方法でエフェクトを取り除いています。

> **警告:** エフェクトを持つ式を純粋な式にキャストしては絶対にいけません。警告はしましたよ。

<!--
# Unchecked Type and Effect Casts

Flix also supports _unchecked_ type and effect casts.

## Unchecked Type Casts

An *unchecked type cast* instructs the compiler that an expression has a specific type.

> **Warning:** Type casts are very dangerous and should be used with utmost
> caution!

Flix programmers should normally never need to use an unchecked type cast.

### Example: Safe Cast to a Super-Type

The expression below casts a `String` to an `Object`:

```flix
unchecked_cast("Hello World" as Object)
```

Note: It is safer to use the `checked_cast` expression.

### Example: Safe Cast from Null to an Object-Type

The expression below casts the `null` value (of type `Null`) to `String`:

```flix
unchecked_cast(null as String)
```

Note: It is safer to use the `checked_cast` expression.

### Example: Unsafe Type Cast

The expression below contains an illegal cast and triggers a
`ClassCastException` at runtime:

```flix
unchecked_cast((123, 456) as Integer)
```

## Effect Casts

An *unchecked effect cast* instructs the compiler that an expression has a
specific effect.

> **Warning:** Effect casts are extremely dangerous and should be used with
> extreme caution!

Flix programmers should normally never need to use an unchecked effect cast.

### Example: Unsafe Effect Cast

We can pretend an impure expression is pure:

```flix
def main(): Unit =
    unchecked_cast(println("Hello World") as _ \ {})
```

Here we call `println` which has the `IO` effect and then we explicitly, and
unsafely, cast away the effect, pretending that the expression is pure.

> **Warning:** Never cast effectful expressions to pure. You have been warned.
-->
