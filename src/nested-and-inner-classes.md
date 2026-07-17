# ネストクラスと内部クラス

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/nested-and-inner-classes.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/nested-and-inner-classes.md)ください。

Java は、ネストされた static クラスと、非 static な内部クラス(Inner class)をサポートしています。

例えば：

```java
package Foo.Bar;

class OuterClass {
    ...
    class InnerClass {
        ...
    }
    static class StaticInnerClass {
        public static String hello() { return "Hi"; }
    }
}
```

Flix では、`import` 文を使って `StaticInnerClass` にアクセスできます：

```flix
import Foo.Bar.{OuterClass$StaticInnerClass => Inner}

def main(): Unit \ IO = 
    println(Inner.hello())
```

典型的な例として、`Map.Entry` クラスへのアクセスが挙げられます：

```flix
import java.util.{Map$Entry => Entry}
```

> **注意:** Flix は、ネストされた非 static な内部クラスへのアクセスをサポートしていません。

<!--
# Nested and Inner Classes

Java supports nested static and non-static inner classes:

For example:

```java
package Foo.Bar;

class OuterClass {
    ...
    class InnerClass {
        ...
    }
    static class StaticInnerClass {
        public static String hello() { return "Hi"; }
    }
}
```

In Flix, we can access the `StaticInnerClass` using the `import` statement:

```flix
import Foo.Bar.{OuterClass$StaticInnerClass => Inner}

def main(): Unit \ IO = 
    println(Inner.hello())
```

A typical example is to access the `Map.Entry` class:

```flix
import java.util.{Map$Entry => Entry}
```

> **Note:** Flix does not support accessing nested non-static inner classes.
-->
