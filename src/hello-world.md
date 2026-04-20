# Hello World

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/hello-world.html)を参照してください。

それでは、`src/Main.flix` ファイルにある、かの有名な _Hello World_ プログラムを見てみましょう：

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

これだけです！

他のプログラミング言語とは異なる点がいくつかあることにすぐ気づくでしょう：

- `main` 関数には仮引数がありません。特に、引数の配列を受け取りません。代わりに、コマンドライン引数は `Env` エフェクトを通じて利用できます（[メイン関数](./main.md)を参照）。
- `main` 関数の戻り値の型は `Unit` です。
- `main` 関数はターミナルに出力するため、`IO` エフェクトを持ちます。

<!--
# Hello World

We can now see the famous _Hello World_ program in `src/Main.flix` file:

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

That's it!

You will immediately notice a few things are different from other programming
languages:

- The `main` function has no formal parameters, in particular it does not take
  an arguments array. Instead the command line arguments are available through
  the `Env` effect (see [The Main Function](./main.md)).
- The return type of the `main` function is `Unit`.
- The `main` function has the `IO` effect since it prints to the terminal.
-->
