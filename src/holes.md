# 匿名ホールと名前付きホール

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/holes.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/holes.md)ください。

Flix では、開発中の未完成なコードに Hole(ホール)を使うことが推奨されています。例えば：

```flix
def sum(x: Int32, y: Int32): Int32 = ???
```

3 つ並んだクエスチョンマーク `???` は匿名ホールを表し、式が期待される場所ならどこでも使えます。上記のコードでは、`???` はまだ書かれていない関数本体を表していますが、式の内部でも使えます。例えば：

```flix
def length(l: List[a]): Int32 = match l {
    case Nil     => 0
    case x :: xs => ???
}
```

プログラムに複数のホールがある場合は、それぞれに名前を付けると便利です。例えば：

```flix
def length(l: List[a]): Int32 = match l {
    case Nil     => ?base
    case x :: xs => ?step
}
```

Flix では、各名前付きホールは一意な名前を持つ必要があります。


## 変数ホールと自動補完

Flix は、型駆動の自動補完候補を提示できる特別な _変数ホール_ をサポートしています。例えば、次のプログラムで：

```flix
def main(): Unit \ IO = 
    let s: String = "Hello World";
    let n: Int32 = s?;
    println("The length of ${s} is ${n}!")
```

カーソルを `s?` の上に置いて自動補完候補を要求すると、Flix は次のような候補を提示します：

- `String.length(s: String): Int32`
- `String.countSubstring(substr: {substr: String}, s: String): Int32`
- `String.levenshtein(s: String, t: String): Int32`
- ...

これらは `String` を `Int32` に変換できる関数だからです。

別の例として、次のプログラムで：

```flix
def main(): Unit \ IO = 
    let l: List[Int32] = List.range(1, 10);
    let n: Int32 = l?;
    println("The value of `n` is ${n}.")
```

カーソルを `l?` の上に置くと、Flix は次のような候補を提示します：

- `List.product(l: List[Int32]): Int32`
- `List.sum(l: List[Int32]): Int32`
- `List.fold(l: List[Int32]): Int32`
- `List.length(l: List[Int32]): Int32`
- `List.count(f: a -> Bool \ ef, l: List[a]): a \ ef`
- ...

これらは `List[Int32]` を `Int32` に変換できる関数だからです。

<!--
# Anonymous and Named Holes

During development, Flix encourages the use of holes for incomplete code. For
example:

```flix
def sum(x: Int32, y: Int32): Int32 = ???
```

The triple question marks `???` represents an anonymous hole and can be used
wherever an expression is expected. In the above code, `???` represents a
missing function body, but it can also be used inside an expression. For
example:

```flix
def length(l: List[a]): Int32 = match l {
    case Nil     => 0
    case x :: xs => ???
}
```

When a program has multiple holes, it can be useful to name them. For example:

```flix
def length(l: List[a]): Int32 = match l {
    case Nil     => ?base
    case x :: xs => ?step
}
```

Flix requires that each named hole has a unique name.


## Variable Holes and Auto-Completion

Flix has support for a special _variable hole_ which enables type-driven
auto-completion suggestions. For example, in the program:

```flix
def main(): Unit \ IO = 
    let s: String = "Hello World";
    let n: Int32 = s?;
    println("The length of ${s} is ${n}!")
```

If we place the cursor on `s?` and ask for auto-completion suggestions, Flix
will suggest:

- `String.length(s: String): Int32`
- `String.countSubstring(substr: {substr: String}, s: String): Int32`
- `String.levenshtein(s: String, t: String): Int32`
- ...

since these are functions that can convert a `String` to an `Int32`.

As another example, in the program:

```flix
def main(): Unit \ IO = 
    let l: List[Int32] = List.range(1, 10);
    let n: Int32 = l?;
    println("The value of `n` is ${n}.")
```

If we place the cursor on `l?`, Flix will suggest:

- `List.product(l: List[Int32]): Int32`
- `List.sum(l: List[Int32]): Int32`
- `List.fold(l: List[Int32]): Int32`
- `List.length(l: List[Int32]): Int32`
- `List.count(f: a -> Bool \ ef, l: List[a]): a \ ef`
- ...

since these are functions that can convert a `List[Int32]` to an `Int32`.
-->
