# 次のステップ

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/next-steps.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/next-steps.md)ください。

いよいよ、最初の本格的なプログラムを書く準備が整いました！

ここでは、UNIX の由緒あるワードカウント（`wc`）プログラムの簡単な変種を書いてみます。

この機会に、Flix で代数エフェクト(Algebraic effects)をどのように使うかを紹介します。

```flix
use Fs.FileRead

def wc(file: String): Unit \ { FileRead, IO } =
    match FileRead.readLines(file) {
        case Ok(lines) =>
            let totalLines = List.length(lines);
            let totalWords = List.sumWith(numberOfWords, lines);
            println("Lines: ${totalLines}, Words: ${totalWords}")
        case Err(_) =>
            println("Unable to read file: ${file}")
    }

def numberOfWords(s: String): Int32 =
     s |> String.words |> List.length

def main(): Unit \ { FileRead, IO } =
    wc("Main.flix")
```

このプログラムは次のように動作します。

まず、ファイル名を受け取り、`FileRead` エフェクトを使ってファイルからすべての行を読み込む `wc` 関数を定義します。

ファイルの読み込みに成功した場合は、次の値を計算します：

- `List.length` を使った行数。
- 各行に `numberOfWords` を適用した結果を合計して求めた単語数。

計算結果は `println` を使ってターミナルに出力されます。

ファイルを読み込めなかった場合は、代わりにエラーメッセージが出力されます。

`wc` 関数の型とエフェクトのシグネチャには、エフェクト集合 `{FileRead, IO}` が指定されており、これらのエフェクトが必要であることを示しています。`FileRead` と `IO` はどちらもデフォルトハンドラを持っているため、`main` の中で明示的にハンドラを呼び出す必要はありません。

<!--
# Next Steps

We are now ready write our first real program! 

We will write a simple variant of the venerable wordcount (`wc`) program from
UNIX. 

We will use the opportunity to illustrate how to use algebraic effects in Flix.

```flix
use Fs.FileRead

def wc(file: String): Unit \ { FileRead, IO } =
    match FileRead.readLines(file) {
        case Ok(lines) =>
            let totalLines = List.length(lines);
            let totalWords = List.sumWith(numberOfWords, lines);
            println("Lines: ${totalLines}, Words: ${totalWords}")
        case Err(_) =>
            println("Unable to read file: ${file}")
    }

def numberOfWords(s: String): Int32 =
     s |> String.words |> List.length

def main(): Unit \ { FileRead, IO } =
    wc("Main.flix")
```

The program works as follows:

We define a `wc` function that takes a filename and reads all lines from the
file using the `FileRead` effect.

If the file is successfully read, we calculate:

- The number of lines using `List.length`.
- The number of words by summing the results of applying `numberOfWords` to each
  line.

The results are printed to the terminal using `println`.

If the file cannot be read, an error message is printed instead.

The `wc` function's type and effect signature specifies the `{FileRead, IO}`
effect set, indicating these effects are required. Both `FileRead` and `IO` have
default handlers, so no explicit handler calls are needed in `main`.
-->
