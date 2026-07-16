# 文字列補間

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/string-interpolation.html)を参照してください。

Flix の文字列は補間をサポートしています。文字列の中で `"${e}"` という形式を書くと、`e` を評価して値にし、`ToString` トレイトを使って文字列に変換します。例えば：

```flix
let fstName = "Lucky";
let lstName = "Luke";
"Hello Mr. ${lstName}. Do you feel ${fstName}, punk?"
```

文字列補間(String interpolation)は、`ToString` インスタンスを実装している任意の型に対して機能します。例えば：

```flix
let i = 123;
let o = Some(123);
let l = 1 :: 2 :: 3 :: Nil;
"i = ${i}, o = ${o}, l = ${l}"
```

文字列補間には任意の式を含めることができます。例えば：

```flix
let x = 1;
let y = 2;
"${x + y + 1}"
```

2 つの文字列を連結するには、文字列補間を使うのが推奨される方法です：

```flix
let x = "Hello";
let y = "World";
"${x}${y}" // x + y と同等
```

値を文字列に変換する場合も、文字列補間を使うのが推奨される方法です：

```flix
let o = Some(123);
"${o}"
```

これは、`ToString` トレイトの `toString` 関数を明示的に使うのと同等です：

```flix
ToString.toString(o)
```

文字列補間はネストさせることもできますが、推奨されません。

<!--
# String Interpolation

Flix strings support interpolation. Inside a string, the form `"${e}"` evaluates
`e` to a value and converts it to a string using the `ToString` trait. For
example:

```flix
let fstName = "Lucky";
let lstName = "Luke";
"Hello Mr. ${lstName}. Do you feel ${fstName}, punk?"
```

String interpolation works for any type that implements a `ToString` instance.
For example:

```flix
let i = 123;
let o = Some(123);
let l = 1 :: 2 :: 3 :: Nil;
"i = ${i}, o = ${o}, l = ${l}"
```

String interpolations may contain arbitrary expressions. For example:

```flix
let x = 1;
let y = 2;
"${x + y + 1}"
```

String interpolation is the preferred way to concatenate two strings:

```flix
let x = "Hello";
let y = "World";
"${x}${y}" // equivalent to x + y
```

String interpolation is the preferred way to convert a value to a string:

```flix
let o = Some(123);
"${o}"
```

which is equivalent to an explicit use of the `toString` function from the
`ToString` trait:

```flix
ToString.toString(o)
```

String interpolators may nest, but this is discouraged.
-->
