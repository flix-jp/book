# Java との相互運用

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/interoperability.html)を参照してください。

Flix は [Java Virtual Machine](https://en.wikipedia.org/wiki/Java_virtual_machine)（JVM）ベースのプログラミング言語です。したがって：

- Flix プログラムは効率的な JVM バイトコードにコンパイルされます。
- Flix プログラムは任意の Java Virtual Machine 上で動作します[^1]。
- Flix プログラムは Java コードを呼び出すことができます。

Flix は、相互運用(Interoperability)に必要な Java の機能のほとんどをサポートしています：

- [クラスからのオブジェクト生成](./creating-objects.md)
- [クラスやオブジェクトのメソッド呼び出し](./calling-methods.md)
- [クラスやオブジェクトのフィールドの読み書き](./reading-and-writing-fields.md)
- [クラスやインターフェースの匿名拡張](./extending-classes-and-interfaces.md)
- [内部クラスへのアクセス](./nested-and-inner-classes.md)
- [例外のキャッチとスロー](./exceptions.md)
- [プリミティブ値のボックス化とアンボックス化](./boxing-and-unboxing.md)

このため、Flix プログラムは Java クラスライブラリを再利用できます。さらに、Flix のパッケージマネージャは Maven をサポートしています。

Flix と Java は同じ基本型を共有していますが、次の表に示すように名前が異なります：

| Flix の型 | Java の型 |
|-----------|-----------|
| Bool      | boolean   |
| Char      | char      |
| Float32   | float     |
| Float64   | double    |
| Int8      | byte      |
| Int16     | short     |
| Int32     | int       |
| Int64     | long      |
| String    | String    |

Flix では、プリミティブ型は常にアンボックス化されています。そのため、`java.lang.Integer` を期待する Java メソッドを呼び出すには、Flix の `Int32` を持っている場合、`java.lang.Integer.valueOf` を呼び出してボックス化する必要があります。

[^1]: Flix には少なくとも Java 21 が必要です。

<!--
# Interoperability with Java

Flix is a [Java Virtual Machine](https://en.wikipedia.org/wiki/Java_virtual_machine) (JVM)-based programming language,
hence:

- Flix programs compile to efficient JVM bytecode.
- Flix programs run on any Java Virtual Machine[^1].
- Flix programs can call Java code.

Flix supports most Java features necessary for interoperability:

- [Creating objects from classes](./creating-objects.md)
- [Calling methods on classes and objects](./calling-methods.md)
- [Reading and writing fields on classes and objects](./reading-and-writing-fields.md)
- [Anonymous extension of classes and interfaces](./extending-classes-and-interfaces.md)
- [Accessing inner classes](./nested-and-inner-classes.md)
- [Catching and throwing exceptions](./exceptions.md)
- [Boxing and unboxing of primitive values](./boxing-and-unboxing.md)

Thus Flix programs can reuse the Java Class Library. In addition, the Flix
package manager has Maven support. 

Flix and Java share the same base types, but they have different names, as shown
in the table:

| Flix Type | Java Type |
|-----------|-----------|
| Bool      | boolean   |
| Char      | char      |
| Float32   | float     |
| Float64   | double    |
| Int8      | byte      |
| Int16     | short     |
| Int32     | int       |
| Int64     | long      |
| String    | String    |

In Flix primitive types are always unboxed.
Hence, to call a Java method that expects a `java.lang.Integer`,
if you have a Flix `Int32`, it must be boxed by calling `java.lang.Integer.valueOf`.

[^1]: Flix requires at least Java 21.
-->
