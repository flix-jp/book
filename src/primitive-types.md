# プリミティブ型

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/primitive-types.html)を参照してください。

Flix は以下のプリミティブ型をサポートしています：

| 型           | 構文                                     | 説明                              |
|--------------|------------------------------------------|-----------------------------------|
| Unit         | `()`                                     | Unit 値                           |
| Bool         | `true`, `false`                          | 真偽値                            |
| Char         | `'a'`, `'b'`, `'c'`                      | 文字[^1]値                        |
| Float32      | `0.0f32`, `21.42f32`, `-21.42f32`        | 32ビット浮動小数点数              |
| Float64      | `0.0f64`, `21.42f64`, `-21.42f64`        | 64ビット浮動小数点数              |
| Int8         | `0i8`, `1i8`, `-1i8`, `127i8`, `-128i8`  | 符号付き8ビット整数               |
| Int16        | `0i16`, `123i16`, `-123i16`              | 符号付き16ビット整数              |
| Int32        | `0i32`, `123i32`, `-123i32`              | 符号付き32ビット整数              |
| Int64        | `0i64`, `123i64`, `-123i64`              | 符号付き64ビット整数              |
| String       | `"hello"`, `"world"`                     | 文字列値                          |
| BigInt       | `0ii`, `123ii`, `-123ii`                 | 任意精度整数                      |
| BigDecimal   | `0.0ff`, `123.45ff`, `-123.45ff`         | 任意精度小数                      |

`Float64` と `Int32` の値はサフィックスなしで記述できます。
つまり、`123.0f64` は単に `123.0` と、`123i32` は `123` と書くことができます。

## リテラル

Flix には List(リスト)、Set(セット)、Map(マップ)、正規表現のための組み込みの糖衣構文があります。

### List リテラル

List リテラルは中置演算子 `::` を使って記述します。例えば：

```flix
1 :: 2 :: 3 :: Nil
```

これは以下の糖衣構文です：

```flix
Cons(1, Cons(2, Cons(3, Nil)))
```

また、同じ List は以下のようにも書けます：

```flix
List#{1, 2, 3}
```

### Set リテラル

Set リテラルは `Set#{v1, v2, ...}` という記法で記述します。例えば：

```flix
Set#{1, 2, 3}
```

これは以下の糖衣構文です：

```flix
Set.insert(3, Set.insert(2, Set.insert(1, Set.empty())))
```

要素は左から右へ挿入されるため、1 が最初に挿入されることに注意してください。

### Map リテラル

Map リテラルは `Map#{k1 => v1, k2 => v2, ...}` という記法で記述します。
例えば：

```flix
Map#{1 => "Hello", 2 => "World"}
```

これは以下の糖衣構文です：

```flix
Map.insert(2, "World", Map.insert(1, "Hello", Map.empty()))
```

上記の Set と同様に、エントリは左から右へ挿入されることに注意してください。特に、複数のエントリが同じキーを持つ場合、最も右のエントリが以前の値を上書きします。

### 正規表現リテラル

正規表現リテラルは `regex"..."` という記法で記述します。例えば：

```flix
Regex.isMatch(regex"abc", "abc")
```

さらに、正規表現リテラルは `regex"\\..."` という記法で正規表現のエスケープシーケンスをサポートしています。例えば：

```flix
Regex.isMatch(regex"\\w", "W")
```

[^1]: より正確には、`Char` 値は単一の UTF-16 コードユニットに対応します。UTF-16 は可変長エンコーディングです：
    一部の Unicode コードポイントは単一の UTF-16 コードユニットで表現されますが、他のコードポイントは2つのコードユニットを必要とします。
    （また、通常1つの文字として認識されるものを表現するために、複数の Unicode コードポイントの組み合わせが必要になる場合があることにも注意してください。）

<!--
# Primitives

Flix supports the primitive types:

| Type         | Syntax                                   | Description                       |
|--------------|------------------------------------------|-----------------------------------|
| Unit         | `()`                                     | The unit value.                   |
| Bool         | `true`, `false`                          | A boolean value.                  |
| Char         | `'a'`, `'b'`, `'c'`                      | A character[^1] value.            |
| Float32      | `0.0f32`, `21.42f32`, `-21.42f32`        | A 32-bit floating point integer.  |
| Float64      | `0.0f64`, `21.42f64`, `-21.42f64`        | A 64-bit floating point integer.  |
| Int8         | `0i8`, `1i8`, `-1i8`, `127i8`, `-128i8`  | A signed 8-bit integer.           |
| Int16        | `0i16`, `123i16`, `-123i16`              | A signed 16-bit integer.          |
| Int32        | `0i32`, `123i32`, `-123i32`              | A signed 32-bit integer.          |
| Int64        | `0i64`, `123i64`, `-123i64`              | A signed 64-bit integer.          |
| String       | `"hello"`, `"world"`                     | A string value.                   |
| BigInt       | `0ii`, `123ii`, `-123ii`                 | An arbitrary precision integer.   |
| BigDecimal   | `0.0ff`, `123.45ff`, `-123.45ff`         | An arbitrary precision decimal.   |

`Float64` and `Int32` values can be
written without suffix, i.e. `123.0f64` can simply be written
as `123.0` and `123i32` can be written as `123`.

## Literals

Flix has built-in syntactic sugar for lists, sets, maps and regex.

### List Literals

A list literal is written using the infix `::` constructor. For example:

```flix
1 :: 2 :: 3 :: Nil
```

which is syntactic sugar for:

```flix
Cons(1, Cons(2, Cons(3, Nil)))
```

Alternatively, the same list can also be written as:

```flix
List#{1, 2, 3}
```

### Set Literals

A set literal is written using the notation `Set#{v1, v2, ...}`. For example:

```flix
Set#{1, 2, 3}
```

which is syntactic sugar for:

```flix
Set.insert(3, Set.insert(2, Set.insert(1, Set.empty())))
```

Note that the elements are inserted from left to right, thus 1 is inserted first.

### Map Literals

A map literal is written using the notation
`Map#{k1 => v1, k2 => v2, ...}`.
For example:

```flix
Map#{1 => "Hello", 2 => "World"}
```

which is syntactic sugar for:

```flix
Map.insert(2, "World", Map.insert(1, "Hello", Map.empty()))
```

Note that similar to sets above, the entries are inserted left to right. In particular, if multiple entries share the same key, the rightmost one overwrites the previous values.

### Regex Literals

A regex literal is written using the notation `regex"..."`. For example:

```flix
Regex.isMatch(regex"abc", "abc")
```

Additionally, regex literals support regex escape sequences with the following notation `regex"\\..."`. For example:

```flix
Regex.isMatch(regex"\\w", "W")
```

[^1]: More precisely, a `Char` value corresponds to a single UTF-16 code unit. UTF-16 is a variable length encoding:
    some of the Unicode code points are represented by a single UTF-16 code unit, but others need two code units.
    (Also note that a combination of several Unicode code points may be needed to represent what is usually perceived as
    a single character.)
-->
