# フィールドの読み書き

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/reading-and-writing-fields.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/reading-and-writing-fields.md)ください。

Flix は、標準的な Java の構文によるオブジェクトフィールド(Object field)および静的フィールド(Static field)（クラスフィールド）の読み取りをサポートしています。

## オブジェクトフィールドの読み取り

オブジェクトフィールドは次のように読み取ることができます：

```flix
import java.awt.Point

def area(p: Point): Int32 \ IO = p.x * p.y
```

## 静的フィールドの読み取り

静的フィールドは次のように読み取ることができます：

```flix
import java.lang.Math

def area(radius: Float64): Float64 = (unsafe Math.PI) * radius * radius
```

ここでは `java.lang.Math` クラスをインポートし、静的な `PI` フィールドにアクセスしています。

`PI` フィールドは決して変化しないことが分かっているため、`unsafe` を使ってエフェクトをキャストして取り除いています。

<!--
# Reading and Writing Fields

Flix supports reading object fields and static (class) fields with standard Java
syntax.

## Reading Object Fields

We can read an object field as follows:

```flix
import java.awt.Point

def area(p: Point): Int32 \ IO = p.x * p.y
```

## Reading Static Fields

We can read a static field as follows:

```flix
import java.lang.Math

def area(radius: Float64): Float64 = (unsafe Math.PI) * radius * radius
```

We import the `java.lang.Math` class and then we access the static `PI` field. 

We know that the `PI` field will never change, hence we cast away the effect with `unsafe`.
-->
