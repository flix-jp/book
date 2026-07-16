# クラスとインターフェースの拡張

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/extending-classes-and-interfaces.html)を参照してください。

Flix では、Java のクラスを継承したり、Java のインターフェースを実装したりするオブジェクトを作成できます。

この機能は、概念的には Java の[匿名クラス(Anonymous Classes)](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)に似ています。つまり、インターフェースを実装する、あるいはクラスを継承する（名前のない）クラスを定義し、そのクラスのオブジェクトを生成する、という一連の処理を 1 つの式で行うことができます。

例えば、`java.lang.Runnable` インターフェースを実装するオブジェクトは次のように作成できます：

```flix
import java.lang.Runnable

def newRunnable(): Runnable \ IO = new Runnable {
    def $run(_this: Runnable): Unit \ IO = 
        println("I am running!")
}
```

`newRunnable` を呼び出すたびに、`java.lang.Runnable` を実装した*新しい*オブジェクトが得られます。

> **注意:** 暗黙の `this` 引数は、new 式の中では常に第 1 引数として明示的に渡されます。

別の例として、`java.io.Closeable` インターフェースを実装するオブジェクトも作成できます：

```flix
import java.io.Closeable

def newClosable(): Closeable \ IO = new Closeable {
    def close(_this: Closeable): Unit \ IO = 
        println("I am closing!")
}
```

クラスを継承することもできます。例えば、`hashCode` メソッドと `toString` メソッドをオーバーライドした `java.lang.Object` を作成できます：

```flix
def newObject(): Object \ IO = new Object {
    def hashCode(_this: Object): Int32 = 42
    def toString(_this: Object): String = "Hello World!"
}
```

## スーパーメソッドの呼び出し

匿名サブクラスでメソッドをオーバーライドするとき、`super.methodName(args)` を使って親クラスの実装を呼び出すことができます。

例えば、`Thread` を継承して `toString` をオーバーライドし、親クラスのデフォルトの表現を含めることができます：

```flix
import java.lang.Thread

def main(): Unit \ IO =
    let t = new Thread {
        def new(): Thread \ IO = super("my-thread")
        def run(_this: Thread): Unit \ IO =
            println("Hello from ${Thread.currentThread().getName()}")
        def toString(_this: Thread): String \ IO =
            "MyThread(" + super.toString() + ")"
    };
    println(t)
```

ここで `super.toString()` は、（親クラスである）`Thread` が定義する `toString` メソッドを呼び出し、その結果を `"MyThread(...)"` で包んでいます。

> **注意:** スーパーメソッド呼び出しは `new` 式の内部、すなわち匿名サブクラスを定義するときにのみ使用できます。

<!--
# Classes and Interfaces

Flix allows us to create objects that extend a Java class or implements a Java interface.

This feature is conceptually similar to Java's [Anonymous Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html): 
We can define an (unnamed class) which implements an interface or extends a class and create an object of that class. All in one expression. 

For example, we can create an object that implements the `java.lang.Runnable` interface:

```flix
import java.lang.Runnable

def newRunnable(): Runnable \ IO = new Runnable {
    def $run(_this: Runnable): Unit \ IO = 
        println("I am running!")
}
```

Every time we call `newRunnable` we get a *fresh* object that implements `java.lang.Runnable`.

> **Note:** The implicit `this` argument is always explicitly passed as the first argument in a new expression.

As another example, we can create an object that implements the `java.io.Closeable` interface:

```flix
import java.io.Closeable

def newClosable(): Closeable \ IO = new Closeable {
    def close(_this: Closeable): Unit \ IO = 
        println("I am closing!")
}
```

We can also extend classes. For example, we can create a
`java.lang.Object` where we override the `hashCode` and `toString` methods:

```flix
def newObject(): Object \ IO = new Object {
    def hashCode(_this: Object): Int32 = 42
    def toString(_this: Object): String = "Hello World!"
}
```

## Calling Super Methods

When overriding a method in an anonymous subclass, we can call the parent
class's implementation using `super.methodName(args)`.

For example, we can extend `Thread` and override `toString` to include the
parent's default representation:

```flix
import java.lang.Thread

def main(): Unit \ IO =
    let t = new Thread {
        def new(): Thread \ IO = super("my-thread")
        def run(_this: Thread): Unit \ IO =
            println("Hello from ${Thread.currentThread().getName()}")
        def toString(_this: Thread): String \ IO =
            "MyThread(" + super.toString() + ")"
    };
    println(t)
```

Here `super.toString()` calls the `toString` method defined by `Thread` (the
parent class), and we wrap the result in `"MyThread(...)"`.

> **Note:** Super method calls can only be used inside `new` expressions, i.e.
> when defining an anonymous subclass.
-->
