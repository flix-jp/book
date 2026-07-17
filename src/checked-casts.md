# 検査付き型キャストとエフェクトキャスト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/checked-casts.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/checked-casts.md)ください。

Flix の型・エフェクトシステムは――設計上――サブタイピング(sub-typing)もサブエフェクティング(sub-effecting)もサポートしていません。この制限は実際にはめったに問題になりませんが、回避するために Flix には 2 つの _安全な_ アップキャスト(upcast)構文が用意されています：

- 検査付き*型*キャスト: `checked_cast(exp)`
- 検査付き*エフェクト*キャスト: `checked_ecast(exp)`

> **注意:** `checked_cast` と `checked_ecast` の式は _安全_ であることが保証されています。Flix コンパイラは、すべての検査付きキャストが決して失敗しないことをコンパイル時に検査します。

## 検査付き型キャスト

次のプログラム：

```flix
import java.lang.Object

def main(): Unit =
    let s = "Hello World";
    let _: Object = s;
    ()
```

はコンパイルできません：

```
❌ -- Type Error --------------------------------------------------

>> Unexpected type: expected 'java.lang.Object', found 'String'.

5 |     let _: Object = s;
                        ^
                        expression has unexpected type.
```

なぜなら、Flix では `String` 型は `Object` のサブタイプ _ではない_ からです。

検査付き型キャストを使えば、`String` から `Object` へ安全にアップキャストできます：

```flix
import java.lang.Object;

def main(): Unit =
    let s = "Hello World";
    let _: Object = checked_cast(s);
    ()
```

`checked_cast` 構文を使うと、任意の Java 型をそのスーパータイプのいずれかへ安全にアップキャストできます：

```flix
let _: Object       = checked_cast("Hello World");
let _: CharSequence = checked_cast("Hello World");
let _: Serializable = checked_cast("Hello World");
let _: Object       = checked_cast(null);
let _: String       = checked_cast(null);
```

## 検査付きエフェクトキャスト

次のプログラム：

```flix
def hof(f: Int32 -> Int32 \ IO): Int32 \ IO = f(42)

def main(): Int32 \ IO =
    hof(x -> x + 1)
```

はコンパイルできません：

```
❌ -- Type Error --------------------------------------------------

>> Expected argument of type 'Int32 -> Int32 \ IO', but got 'Int32 -> Int32'.

4 |     hof(x -> x + 1)
            ^^^^^^^^^^
            expected: 'Int32 -> Int32 & Impure \ IO'

The function 'hof' expects its 1st argument to be of type 'Int32 -> Int32 \ IO'.

Expected: Int32 -> Int32 & Impure \ IO
  Actual: Int32 -> Int32
```

なぜなら、Flix では _純粋な_ 関数は不純な関数のサブタイプ _ではない_ からです。具体的には、`hof` は `IO` エフェクトを持つ関数を要求していますが、渡しているのは純粋な関数です。

検査付きエフェクトキャストを使えば、純粋な式を不純な式へ安全にアップキャストできます：

```flix
def main(): Int32 \ IO =
    hof(x -> checked_ecast(x + 1))
```

`checked_ecast` 構文によって、`x + 1` が `IO` エフェクトを持っているかのように扱うことができます。

> **注意:** Flix では――一般的な経験則として――高階関数はその関数引数に特定のエフェクトを要求する _べきではありません_。代わりに、エフェクト多相にするべきです。

## 関数型

`checked_cast` と `checked_ecast` のいずれの構文も、関数型に対しては機能しません。

例えば、次のコードは動作しません：

```flix
let f: Unit -> ##java.lang.Object = checked_cast(() -> "Hello World")
```

これは関数型 `Unit -> String` を `Unit -> Object` へキャストしようとしているためです。

代わりに、次のように書くべきです：

```flix
let f: Unit -> ##java.lang.Object = (() -> checked_cast("Hello World"))
```

こちらは `String` を `Object` へ直接キャストしているからです。

<!--
# Checked Type and Effect Casts

The Flix type and effect system – by design – does not support sub-typing nor
sub-effecting. To work around these limitations, which are rare in practice,
Flix has two _safe_ upcast constructs: 

- A checked *type* cast: `checked_cast(exp)`, and 
- A checked *effect* cast `checked_ecast(exp)`.

> **Note:** The `checked_cast` and `checked_ecast` expressions are guaranteed to
> be _safe_. The Flix compiler will check at compile-time that every checked
> cast cannot go wrong. 

## Checked Type Casts

The following program:

```flix
import java.lang.Object

def main(): Unit =
    let s = "Hello World";
    let _: Object = s;
    ()
```

does not compile:

```
❌ -- Type Error --------------------------------------------------

>> Unexpected type: expected 'java.lang.Object', found 'String'.

5 |     let _: Object = s;
                        ^
                        expression has unexpected type.
```

because in Flix the `String` type is _not_ a subtype of `Object`.

We can use a checked type cast to safely upcast from `String` to `Object`:

```flix
import java.lang.Object;

def main(): Unit =
    let s = "Hello World";
    let _: Object = checked_cast(s);
    ()
```

We can use the `checked_cast` construct to safely upcast any Java type to one of
its super-types:

```flix
let _: Object       = checked_cast("Hello World");
let _: CharSequence = checked_cast("Hello World");
let _: Serializable = checked_cast("Hello World");
let _: Object       = checked_cast(null);
let _: String       = checked_cast(null);
```

## Checked Effect Casts

The following program:

```flix
def hof(f: Int32 -> Int32 \ IO): Int32 \ IO = f(42)

def main(): Int32 \ IO =
    hof(x -> x + 1)
```

does not compile:

```
❌ -- Type Error --------------------------------------------------

>> Expected argument of type 'Int32 -> Int32 \ IO', but got 'Int32 -> Int32'.

4 |     hof(x -> x + 1)
            ^^^^^^^^^^
            expected: 'Int32 -> Int32 & Impure \ IO'

The function 'hof' expects its 1st argument to be of type 'Int32 -> Int32 \ IO'.

Expected: Int32 -> Int32 & Impure \ IO
  Actual: Int32 -> Int32
```

because in Flix a _pure_ function is _not_ a subtype of an impure function.
Specifically, the `hof` requires a function with the `IO` effect, but we are
passing in a pure function. 

We can use a checked effect cast to safely upcast a pure expression to an
impure expression: 

```flix
def main(): Int32 \ IO =
    hof(x -> checked_ecast(x + 1))
```

The `checked_ecast` construct allows us to pretend that `x + 1` has the `IO` effect. 

> **Note:** In Flix – as a general rule – higher-order functions should _not_
> require their function arguments to have a specific effect. Instead they
> should be effect polymorphic. 

## Function Types

Neither the `checked_cast` nor the `checked_ecast` constructs work on function types. 

For example, the following does not work:

```flix
let f: Unit -> ##java.lang.Object = checked_cast(() -> "Hello World")
```

because it tries to cast the function type `Unit -> String` to `Unit ->
Object`.

Instead, we should write:

```flix
let f: Unit -> ##java.lang.Object = (() -> checked_cast("Hello World"))
```

because it directly casts `String` to `Object`.
-->
