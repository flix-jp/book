# ボックス化とアンボックス化

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/boxing-and-unboxing.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/boxing-and-unboxing.md)ください。

Java とは異なり、Flix が値の暗黙的なボックス化(Boxing)やアンボックス化(Unboxing)を行うことは決してありません。

私たちは、自動ボックス化は設計上の欠陥であると考えており、サポートする予定はありません。したがって、プリミティブ値は手動でボックス化・アンボックス化する必要があります。

## ボックス化

次の例は、プリミティブな整数をボックス化する方法を示しています：

```flix
def f(x: Int32): String \ IO = 
    let i = Box.box(x); // Integer
    i.toString()
```

ここで `Box.box(x)` の呼び出しは `Integer` オブジェクトを返します。`i` はオブジェクトなので、`toString` を呼び出すことができます。ボックス化は純粋な操作ですが、`toString` の呼び出しは `IO` エフェクトを持ちます。

## アンボックス化

次の例は、2 つの Java の `Integer` オブジェクトをアンボックス化する方法を示しています：

```flix
import java.lang.Integer

def sum(x: Integer, y: Integer): Int32 = 
    Box.unbox(x) + Box.unbox(y)
```

ここで `Box.unbox` の呼び出しは `Int32` のプリミティブ値を返します。

アンボックス化は純粋な操作です。

<!--
# Boxing and Unboxing

Unlike Java, Flix never performs implicit boxing or unboxing of values. 

We believe auto boxing is a design flaw and do not plan to support it. Hence,
primitive values must be manually boxed and unboxed. 

## Boxing

The following example shows how to box a primitive integer:

```flix
def f(x: Int32): String \ IO = 
    let i = Box.box(x); // Integer
    i.toString()
```

Here the call to `Box.box(x)` returns an `Integer` object. Since `i` is an
object, we can call `toString` on it. Boxing is a pure operation, but calling
`toString` has the `IO` effect. 

## Unboxing

The following example shows how to unbox two Java `Integer` objects:

```flix
import java.lang.Integer

def sum(x: Integer, y: Integer): Int32 = 
    Box.unbox(x) + Box.unbox(y)
```

Here the call to `Box.unbox` returns an `Int32` primitive value. 

Unboxing is a pure operation.
-->
