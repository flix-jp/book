# Console

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/console.html)を参照してください。

Flix は、ターミナル I/O のためのライブラリエフェクトとして `Console` を提供しています。`Console` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。主要なモジュールは `Sys.Console` です。

## Console エフェクト

`Console` エフェクトは、標準入力からの読み取りと、標準出力および標準エラー出力への書き込みをサポートしています：

```flix
pub eff Console {
    /// コンソールから 1 行を読み取ります。
    def readln(): String

    /// 与えられた文字列 `s` を標準出力に出力します。
    def print(s: String): Unit

    /// 与えられた文字列 `s` を標準エラー出力に出力します。
    def eprint(s: String): Unit

    /// 与えられた文字列 `s` を標準出力に出力し、続けて改行を出力します。
    def println(s: String): Unit

    /// 与えられた文字列 `s` を標準エラー出力に出力し、続けて改行を出力します。
    def eprintln(s: String): Unit
}
```

## Console モジュール

`Console` モジュールは、`Console` エフェクトの上に構築された、いくつかの高レベルな関数を提供します：

```flix
mod Sys.Console {
    /// プロンプト `p` を出力して 1 行を読み取り、入力が空の場合は `default` を返します。
    def readlnWithDefault(p: a, default: String): String \ Console

    /// プロンプト `p` を出力して 1 行を読み取り、入力に `f` を適用します。
    /// `Err(msg)` の場合は再度プロンプトを表示し、`Ok(v)` の場合は `v` を返します。
    def readlnWith(p: a, f: String -> Result[String, b]): b \ Console

    /// yes/no のヒント付きでプロンプト `p` を出力し、ブール値の回答を読み取ります。
    /// 入力が空または認識できない場合は `default` を返します。
    def confirm(p: a, default: {default = Bool}): Bool \ Console

    /// 番号付きの選択肢リストとともにプロンプト `p` を出力し、選択を読み取ります。
    /// 入力が無効な場合は `None` を返します。
    def pick(p: a, choices: List[b]): Option[b] \ Console

    /// `pick` と同様ですが、ユーザーが有効な選択をするまで再度プロンプトを表示します。
    def pickWith(p: a, choices: List[b]): b \ Console
}
```

## 基本的なコンソール I/O

`Console` の最もシンプルな使い方は、プロンプトを出力し、入力を読み取り、応答することです：

```flix
use Sys.Console

def main(): Unit \ Console =
    Console.print("What is your name? ");
    let name = Console.readln();
    Console.println("Hello ${name}!")
```

## 確認付き入力

`Console.confirm` 関数は yes/no の質問をして `Bool` を返します。ユーザーが何も入力せずに Enter を押した場合に使われるデフォルト値を指定できます：

```flix
use Sys.Console

def main(): Unit \ Console =
    let proceed = Console.confirm("Deploy to production?", default = true);
    if (proceed)
        Console.println("Deploying...")
    else
        Console.println("Aborted.")
```

## バリデーション付き入力

`Console.readlnWith` 関数は、入力がバリデータを通過するまで繰り返しユーザーにプロンプトを表示します。バリデータは、成功時には `Ok(value)` を返し、再度プロンプトを表示させるには `Err(message)` を返します：

```flix
use Sys.Console

def main(): Unit \ Console =
    let n = Console.readlnWith("Enter a number (1-10): ", s ->
        match Int32.fromString(s) {
            case Some(i) if i >= 1 and i <= 10 => Ok(i)
            case _ => Err("Please enter a number between 1 and 10.")
        }
    );
    Console.println("You entered: ${n}")
```

<!--
# Console

Flix provides `Console` as a library effect for terminal I/O. The `Console`
effect has a default handler, so no explicit `runWithIO` call is needed in
`main`. The key module is `Sys.Console`.

## The Console Effect

The `Console` effect supports reading from standard input and writing to
standard output and standard error:

```flix
pub eff Console {
    /// Reads a single line from the console.
    def readln(): String

    /// Prints the given string `s` to the standard out.
    def print(s: String): Unit

    /// Prints the given string `s` to the standard err.
    def eprint(s: String): Unit

    /// Prints the given string `s` to the standard out followed by a new line.
    def println(s: String): Unit

    /// Prints the given string `s` to the standard err followed by a new line.
    def eprintln(s: String): Unit
}
```

## The Console Module

The `Console` module provides several higher-level functions built on the
`Console` effect:

```flix
mod Sys.Console {
    /// Prints prompt `p`, reads a line, and returns `default` if the input is empty.
    def readlnWithDefault(p: a, default: String): String \ Console

    /// Prints prompt `p`, reads a line, and applies `f` to the input.
    /// Re-prompts on `Err(msg)`, returns `v` on `Ok(v)`.
    def readlnWith(p: a, f: String -> Result[String, b]): b \ Console

    /// Prints prompt `p` with a yes/no hint and reads a boolean answer.
    /// Empty or unrecognized input returns `default`.
    def confirm(p: a, default: {default = Bool}): Bool \ Console

    /// Prints prompt `p` with a numbered list of choices and reads a selection.
    /// Returns `None` if the input is invalid.
    def pick(p: a, choices: List[b]): Option[b] \ Console

    /// Like `pick`, but re-prompts until the user makes a valid selection.
    def pickWith(p: a, choices: List[b]): b \ Console
}
```

## Basic Console I/O

The simplest use of `Console` is to print a prompt, read input, and respond:

```flix
use Sys.Console

def main(): Unit \ Console =
    Console.print("What is your name? ");
    let name = Console.readln();
    Console.println("Hello ${name}!")
```

## Confirmed Input

The `Console.confirm` function asks a yes/no question and returns a `Bool`. You
can supply a default value that is used when the user presses Enter without
typing anything:

```flix
use Sys.Console

def main(): Unit \ Console =
    let proceed = Console.confirm("Deploy to production?", default = true);
    if (proceed)
        Console.println("Deploying...")
    else
        Console.println("Aborted.")
```

## Validated Input

The `Console.readlnWith` function repeatedly prompts the user until the input
passes a validator. The validator returns `Ok(value)` on success or
`Err(message)` to re-prompt:

```flix
use Sys.Console

def main(): Unit \ Console =
    let n = Console.readlnWith("Enter a number (1-10): ", s ->
        match Int32.fromString(s) {
            case Some(i) if i >= 1 and i <= 10 => Ok(i)
            case _ => Err("Please enter a number between 1 and 10.")
        }
    );
    Console.println("You entered: ${n}")
```
-->
