# オブジェクトの生成

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/creating-objects.html)を参照してください。

Flix では、Java に似た構文でオブジェクトを生成できます。

例えば：

```flix
import java.io.File

def main(): Unit \ IO = 
    let f = new File("foo.txt");
    println("Hello World!")
```

ここでは `java.io.File` クラスをインポートし、`new` キーワードを使ってコンストラクタの1つを呼び出すことで、`File` オブジェクトをインスタンス化しています。

`File` クラスには複数のコンストラクタがあるため、次のように書くこともできます：

```flix
import java.io.File

def main(): Unit \ IO = 
    let f1 = new File("foo.txt");
    let f2 = new File("bar", "foo.txt");
    println("Hello World!")
```

Flix は、引数の個数とその型に基づいてコンストラクタを解決します。

別の例として、次のように書けます：

```flix
import java.io.File
import java.net.URI

def main(): Unit \ IO = 
    let f1 = new File("foo.txt");
    let f2 = new File("bar", "foo.txt");
    let f3 = new File(new URI("file://foo.txt"));
    println("Hello World!")
```

Java の名前と Flix のモジュールが衝突する場合は、_リネームインポート(renaming import)_ を使って解決できます：

```flix
import java.lang.{String => JString}

def main(): Unit \ IO = 
    let s = new JString("Hello World");
    println("Hello World!")
```

ここで `JString` は Java クラスの `java.lang.String` を指し、`String` は Flix のモジュールを指します。なお、内部的には Flix の文字列と Java の文字列は同じものです。

## スーパーコンストラクタの呼び出し

Java クラスの匿名サブクラス(anonymous subclass)を生成する際には、`super` を使って親のコンストラクタを呼び出すコンストラクタを定義できます：

```flix
import java.lang.Thread

def main(): Unit \ IO =
    let t = new Thread {
        def new(): Thread \ IO = super("my-thread")
        def run(_this: Thread): Unit \ IO =
            println("Hello from ${Thread.currentThread().getName()}")
    };
    t.start()
```

ここでは `Thread` を継承し、スレッド名として `"my-thread"` を親のコンストラクタに渡しています。コンストラクタは `def new()` で定義し、その本体は厳密に `super(...)` の呼び出しでなければなりません。1つの `new` 式につき、定義できるコンストラクタは最大1つです。

コンストラクタが定義されていない場合、Flix は自動的に親の引数なしコンストラクタを呼び出します。

> **注意:** Java コードとのやり取りは、常に `IO` エフェクトを伴います。

> **注意:** Flix では、Java クラスは使用する前に `import` されている必要があります。特に、`new java.io.File(...)` のように書くことは _できません_。

<!--
# Creating Objects

In Flix, we can create objects using syntax similar to Java.

For example:

```flix
import java.io.File

def main(): Unit \ IO = 
    let f = new File("foo.txt");
    println("Hello World!")
```

Here we import the `java.io.File` class and instantiate a `File` object by
calling one of its constructors using the `new` keyword. 

The `File` class has multiple constructors, so we can also write:

```flix
import java.io.File

def main(): Unit \ IO = 
    let f1 = new File("foo.txt");
    let f2 = new File("bar", "foo.txt");
    println("Hello World!")
```

Flix resolves the constructor based on the number of arguments and their types.

As another example, we can write:

```flix
import java.io.File
import java.net.URI

def main(): Unit \ IO = 
    let f1 = new File("foo.txt");
    let f2 = new File("bar", "foo.txt");
    let f3 = new File(new URI("file://foo.txt"));
    println("Hello World!")
```

We can use a _renaming import_ to resolve a clash between a Java name and a Flix
module: 

```flix
import java.lang.{String => JString}

def main(): Unit \ IO = 
    let s = new JString("Hello World");
    println("Hello World!")
```

Here `JString` refers to the Java class `java.lang.String` whereas `String`
refers to the Flix module. Note that internally Flix and Java strings are the
same.

## Calling Super Constructors

When creating an anonymous subclass of a Java class, we can define a constructor
that calls the parent constructor using `super`:

```flix
import java.lang.Thread

def main(): Unit \ IO =
    let t = new Thread {
        def new(): Thread \ IO = super("my-thread")
        def run(_this: Thread): Unit \ IO =
            println("Hello from ${Thread.currentThread().getName()}")
    };
    t.start()
```

Here we extend `Thread` and pass `"my-thread"` as the thread name to the parent
constructor. The constructor is defined with `def new()` and its body must be
exactly a `super(...)` call. At most one constructor can be defined per `new`
expression.

If no constructor is defined, Flix automatically calls the parent's no-argument
constructor.

> **Note:** Any interaction with Java code always has the `IO` effect.

> **Note:** In Flix, Java classes must be `import`ed before they can be used. In
> particular, we _cannot_ write `new java.io.File(...)`.
-->
