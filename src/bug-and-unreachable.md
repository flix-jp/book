# `bug!` と `unreachable!`

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/bug-and-unreachable.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/bug-and-unreachable.md)ください。

Flix は、`bug!` と `unreachable!` という 2 つの特別な「関数」をサポートしています。これらは、プログラム内部の不変条件(Invariant)が破られており、実行を中断すべきであることを示すために使えます。例えば：

```flix
match o {
    case Some(x) => ...
    case None    => bug!("The value of `o` cannot be empty.")
}
```

別の例を挙げます：

```flix
match k {
    case n if n == 0 => ...
    case n if n >= 0 => ...
    case n if n <= 0 => ...
    case n           =>  unreachable!()
}
```

`bug!` と `unreachable!` の使用は、可能な限り避けるべきです。

<!--
# `bug!` and `unreachable!`

Flix supports two special "functions": `bug!` and
`unreachable!` that can be used to indicate when an
internal program invariant is broken and execute
should abort.
For example:

```flix
match o {
    case Some(x) => ...
    case None    => bug!("The value of `o` cannot be empty.")
}
```

As another example:

```flix
match k {
    case n if n == 0 => ...
    case n if n >= 0 => ...
    case n if n <= 0 => ...
    case n           =>  unreachable!()
}
```

Use of `bug!` and `unreachable!` should be avoided
whenever possible.
-->
