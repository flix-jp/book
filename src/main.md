# main 関数

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/main.html)を参照してください。

すべての Flix プログラムのエントリーポイント(Entry point)は `main` 関数です。`main` 関数は引数を取らず、`Unit` を返す必要があります：

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

## main のエフェクト

`main` 関数では、次のものを自由に組み合わせて使うことができます：

- **プリミティブエフェクト:** `IO` と `NonDet`。
- **デフォルトハンドラを持つ任意のエフェクト:** 例えば `Env`、`Exit`、`Clock`、`Logger` など。

デフォルトハンドラを持つエフェクトは、Flix コンパイラによって自動的に `IO` へと変換されます。詳細は[デフォルトハンドラ](./default-handlers.md)を参照してください。

例えば、main では `Env` と `Exit` エフェクトを使うことができます：

```flix
use Sys.Env
use Sys.Exit

def main(): Unit \ {Env, Exit} =
    let args = Env.getArgs();
    match List.head(args) {
        case None    =>
            println("Missing argument.");
            Exit.exit(1)
        case Some(a) =>
            println("Hello ${a}!")
    }
```

## コマンドライン引数へのアクセス

プログラムに渡されたコマンドライン引数には、`Env` エフェクトを通じて `Env.getArgs()` を呼び出すことでアクセスできます：

```flix
use Sys.Env

def main(): Unit \ {Env, IO} =
    let args = Env.getArgs();
    println("Arguments: ${args}")
```

## プログラムの終了

`Exit.exit` を使うと、特定の終了コード(Exit code)を指定してプログラムを終了できます：

```flix
use Sys.Exit

def main(): Unit \ Exit =
    Exit.exit(0)
```

## なぜ main はエフェクトを持たなければならないのか？

Flix は、`main` がエフェクトを持つことを要求します。もし `main` が純粋であれば、そのプログラムを実行する理由がないからです。通常、この要件は `main` がコンソールへ出力したり、その他の副作用を持つことで満たされます。

<!--
# The Main Function

The entry point of any Flix program is the `main` function which must take zero
arguments and return `Unit`:

```flix
def main(): Unit \ IO =
    println("Hello World!")
```

## Effects of Main

The `main` function can use any combination of:

- **Primitive effects:** `IO` and `NonDet`.
- **Any effect with a default handler:** for example `Env`, `Exit`, `Clock`,
  `Logger`, and others.

Effects with default handlers are automatically translated into `IO` by the Flix
compiler. See [Default Handlers](./default-handlers.md) for details.

For example, main can use the `Env` and `Exit` effects:

```flix
use Sys.Env
use Sys.Exit

def main(): Unit \ {Env, Exit} =
    let args = Env.getArgs();
    match List.head(args) {
        case None    =>
            println("Missing argument.");
            Exit.exit(1)
        case Some(a) =>
            println("Hello ${a}!")
    }
```

## Accessing Command Line Arguments

The command line arguments passed to the program can be accessed by calling
`Env.getArgs()` through the `Env` effect:

```flix
use Sys.Env

def main(): Unit \ {Env, IO} =
    let args = Env.getArgs();
    println("Arguments: ${args}")
```

## Exiting the Program

The program can be terminated with a specific exit code using `Exit.exit`:

```flix
use Sys.Exit

def main(): Unit \ Exit =
    Exit.exit(0)
```

## Why Must Main Be Effectful?

Flix requires `main` to be effectful. If `main` was pure there would be no
reason to run the program. Typically this requirement is satisfied because `main`
prints to the console or has another side-effect.
-->
