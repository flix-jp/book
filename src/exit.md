# Exit

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/exit.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/exit.md)ください。

Flix は、プログラムを終了させるためのライブラリエフェクト(Library effect)として `Exit` を提供しています。`Exit` エフェクトはデフォルトハンドラを持っているため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。中心となるモジュールは `Sys.Exit` です。

## Exit エフェクト

`Exit` エフェクトはただ 1 つの操作を持ち、指定された終了コードで JVM を即座に停止させます。

```flix
pub eff Exit {
    /// 指定された `exitCode` で JVM を即座に終了します。
    def exit(exitCode: Int32): Void
}
```

戻り値の型が `Void` であることは、`exit` が正常に戻ることは決してないことを示しています。

## プログラムの終了

`Exit` の最も単純な使い方は、特定の終了コードでプログラムを終了させることです。

```flix
use Sys.Exit

def main(): Unit \ { Exit, IO } =
    println("Goodbye!");
    Exit.exit(0)
```

慣例として、終了コードがゼロであれば成功を、非ゼロであればエラーを表します。

<!--
# Exit

Flix provides `Exit` as a library effect for terminating the program. The `Exit`
effect has a default handler, so no explicit `runWithIO` call is needed in
`main`. The key module is `Sys.Exit`.

## The Exit Effect

The `Exit` effect has a single operation that immediately stops the JVM with a
given exit code:

```flix
pub eff Exit {
    /// Immediately exits the JVM with the specified `exitCode`.
    def exit(exitCode: Int32): Void
}
```

The return type `Void` indicates that `exit` never returns normally.

## Exiting the Program

The simplest use of `Exit` is to terminate with a specific exit code:

```flix
use Sys.Exit

def main(): Unit \ { Exit, IO } =
    println("Goodbye!");
    Exit.exit(0)
```

A zero exit code conventionally signals success, while a non-zero code signals
an error.
-->
