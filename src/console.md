# Console

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/console.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/console.md)ください。

Flix は、ターミナル I/O のためのライブラリエフェクトとして `Console` を提供しています。`Console` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。主要なモジュールは `Sys.Console` です。

## Console エフェクト

`Console` エフェクトは5つの操作を持ちます。1つは標準入力から1行を読み取り、残りの4つは標準出力または標準エラー出力へ文字列を書き込みます：

```flix
pub eff Console {
    /// コンソールから 1 行を読み取ります。
    /// 入力の終端に達した場合は `None` を返します。
    def readln(): Option[String]

    /// 与えられた文字列 `s` を標準出力に書き込みます。
    def write(s: String): Unit

    /// 与えられた文字列 `s` を標準出力に書き込み、続けて改行を書き込みます。
    def writeLine(s: String): Unit

    /// 与えられた文字列 `s` を標準エラー出力に書き込みます。
    def writeErr(s: String): Unit

    /// 与えられた文字列 `s` を標準エラー出力に書き込み、続けて改行を書き込みます。
    def writeErrLine(s: String): Unit
}
```

これらの操作はハンドラが実装するインターフェースです。プログラムがこれらを直接呼び出すことはほとんどありません。代わりに、これらの操作をもとに定義された `Console` モジュールの関数を呼び出します。

## Console モジュール

`Console` エフェクトは `Sys.Console` モジュールの[コンパニオン](./companion-modules.md)であり、このモジュールは出力用の関数といくつかのプロンプト用関数を提供します：

```flix
mod Sys.Console {
    /// `x` の文字列表現を標準出力に出力します。
    def print(x: a): Unit \ Console with ToString[a]

    /// `x` の文字列表現を標準出力に出力し、続けて改行を出力します。
    def println(x: a): Unit \ Console with ToString[a]

    /// `x` の文字列表現を標準エラー出力に出力します。
    def eprint(x: a): Unit \ Console with ToString[a]

    /// `x` の文字列表現を標準エラー出力に出力し、続けて改行を出力します。
    def eprintln(x: a): Unit \ Console with ToString[a]

    /// プロンプト `p` を出力して 1 行を読み取り、入力に `f` を適用します。
    /// `Err(msg)` の場合は `msg` を出力して再度プロンプトを表示し、`Ok(v)` の場合は `Some(v)` を返します。
    /// 入力の終端に達した場合は `None` を返します。
    def readlnWith(p: a, f: String -> Result[String, b]): Option[b] \ Console with ToString[a]

    /// プロンプト `p` を出力して 1 行を読み取り、`b` 型の値としてパースします。
    /// パースできない場合はエラーを出力して再度プロンプトを表示します。
    /// 入力の終端に達した場合は `None` を返します。
    def readlnAs(p: a): Option[b] \ Console with ToString[a], FromString[b]

    /// yes/no のヒント付きでプロンプト `p` を出力し、ブール値の回答を読み取ります。
    /// 入力が空の場合は `default` を返します。"y"、"yes"、"n"、"no" 以外の入力の場合は
    /// エラーを出力して再度プロンプトを表示します。
    def confirm(p: a, default: {default = Bool}): Bool \ Console with ToString[a]
}
```

操作とは異なり、`print`、`println`、`eprint`、`eprintln` は `String` だけでなく、`ToString` のインスタンスを持つあらゆる値を受け取ります。

## 基本的なコンソール I/O

`Console` の最もシンプルな使い方は、プロンプトを出力し、入力を読み取り、応答することです。`readln` は入力の終端で `None` を返すため、プログラムはその結果をパターンマッチします：

```flix
use Sys.Console

def main(): Unit \ Console =
    Console.print("What is your name? ");
    match Console.readln() {
        case Some(name) => Console.println("Hello ${name}!")
        case None       => Console.println("No name given.")
    }
```

## 値の出力

出力用の関数は `ToString` のインスタンスを持つあらゆる値を受け取るため、値をあらかじめ `String` に変換する必要はありません。`print` と `eprint` は末尾に改行を付けずに書き込み、`println` と `eprintln` は改行を付け加えます。`eprint` と `eprintln` は標準エラー出力に書き込みます：

```flix
use Sys.Console

def main(): Unit \ Console =
    let o = Some(123);
    let l = 1 :: 2 :: 3 :: Nil;
    Console.print("o = ");
    Console.println(o);
    Console.print("l = ");
    Console.println(l);
    Console.eprint("sum = ");
    Console.eprintln(List.sum(l))
```

このプログラムは標準出力に `o = Some(123)` と `l = 1 :: 2 :: 3 :: Nil` を、標準エラー出力に `sum = 6` を出力します。

## 確認付き入力

`Console.confirm` 関数は yes/no の質問をして `Bool` を返します。プロンプトの後ろにはデフォルト値を示す `[Y/n]` または `[y/N]` というヒントが続き、これはユーザーが何も入力せずに Enter を押した場合に使われます。`y`、`yes`、`n`、`no`（大文字・小文字を区別しない）以外の回答を入力した場合はエラーを出力して再度プロンプトを表示します。入力の終端に達した場合はデフォルト値が返されます：

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

`Console.readlnWith` 関数は、入力がバリデータを通過するまで繰り返しユーザーにプロンプトを表示します。バリデータは、成功時には `Ok(value)` を返し、メッセージを出力して再度プロンプトを表示させるには `Err(message)` を返します。入力が通過する前に入力の終端に達した場合、結果は `None` になります：

```flix
use Sys.Console

def main(): Unit \ Console =
    let n = Console.readlnWith(
        "Enter a number (1-10): ",
        s -> match Int32.fromString(s) {
            case Some(i) if i >= 1 and i <= 10 => Ok(i)
            case _                             => Err("Please enter a number between 1 and 10.")
        }
    );
    match n {
        case Some(i) => Console.println("You entered: ${i}")
        case None    => Console.println("No number given.")
    }
```

## パース付き入力

`Console.readlnAs` 関数は、`FromString` のインスタンスを持つ任意の型の値を求めるプロンプトを表示します。入力行をトリムしてパースし、パースできる行が得られるまで再度プロンプトを表示します。パースする型は、型注釈または結果の使われ方によって決まります：

```flix
use Sys.Console

def main(): Unit \ Console =
    let n: Option[Int32] = Console.readlnAs("Enter an integer: ");
    match n {
        case Some(i) => Console.println("${i} squared is ${i * i}.")
        case None    => Console.println("No integer given.")
    }
```

<!--
# Console

Flix provides `Console` as a library effect for terminal I/O. The `Console`
effect has a default handler, so no explicit `runWithIO` call is needed in
`main`. The key module is `Sys.Console`.

## The Console Effect

The `Console` effect has five operations: one reads a line from standard input,
and four write a string to standard output or standard error:

```flix
pub eff Console {
    /// Reads a single line from the console.
    /// Returns `None` if the end of input has been reached.
    def readln(): Option[String]

    /// Writes the given string `s` to the standard out.
    def write(s: String): Unit

    /// Writes the given string `s` to the standard out followed by a new line.
    def writeLine(s: String): Unit

    /// Writes the given string `s` to the standard err.
    def writeErr(s: String): Unit

    /// Writes the given string `s` to the standard err followed by a new line.
    def writeErrLine(s: String): Unit
}
```

The operations are the interface that a handler implements. A program rarely
calls them directly. Instead it calls the functions of the `Console` module,
which are defined in terms of the operations.

## The Console Module

The `Console` effect is the [companion](./companion-modules.md) of the
`Sys.Console` module, which provides the printing functions and several
prompts:

```flix
mod Sys.Console {
    /// Prints the string representation of `x` to the standard out.
    def print(x: a): Unit \ Console with ToString[a]

    /// Prints the string representation of `x` to the standard out followed by a new line.
    def println(x: a): Unit \ Console with ToString[a]

    /// Prints the string representation of `x` to the standard err.
    def eprint(x: a): Unit \ Console with ToString[a]

    /// Prints the string representation of `x` to the standard err followed by a new line.
    def eprintln(x: a): Unit \ Console with ToString[a]

    /// Prints prompt `p`, reads a line, and applies `f` to the input.
    /// Prints `msg` and re-prompts on `Err(msg)`. Returns `Some(v)` on `Ok(v)`.
    /// Returns `None` if the end of input has been reached.
    def readlnWith(p: a, f: String -> Result[String, b]): Option[b] \ Console with ToString[a]

    /// Prints prompt `p`, reads a line, and parses it as a value of type `b`.
    /// Prints an error and re-prompts if the input cannot be parsed.
    /// Returns `None` if the end of input has been reached.
    def readlnAs(p: a): Option[b] \ Console with ToString[a], FromString[b]

    /// Prints prompt `p` with a yes/no hint and reads a boolean answer.
    /// Empty input returns `default`. Prints an error and re-prompts on any other
    /// input than "y", "yes", "n", or "no".
    def confirm(p: a, default: {default = Bool}): Bool \ Console with ToString[a]
}
```

Unlike the operations, `print`, `println`, `eprint`, and `eprintln` accept any
value with a `ToString` instance, not just a `String`.

## Basic Console I/O

The simplest use of `Console` is to print a prompt, read input, and respond.
Since `readln` returns `None` at the end of input, the program matches on the
result:

```flix
use Sys.Console

def main(): Unit \ Console =
    Console.print("What is your name? ");
    match Console.readln() {
        case Some(name) => Console.println("Hello ${name}!")
        case None       => Console.println("No name given.")
    }
```

## Printing Values

Since the printing functions accept any value with a `ToString` instance, a
value does not have to be converted to a `String` first. `print` and `eprint`
write without a trailing newline, whereas `println` and `eprintln` add one.
`eprint` and `eprintln` write to standard error:

```flix
use Sys.Console

def main(): Unit \ Console =
    let o = Some(123);
    let l = 1 :: 2 :: 3 :: Nil;
    Console.print("o = ");
    Console.println(o);
    Console.print("l = ");
    Console.println(l);
    Console.eprint("sum = ");
    Console.eprintln(List.sum(l))
```

The program prints `o = Some(123)` and `l = 1 :: 2 :: 3 :: Nil` to standard
out, and `sum = 6` to standard error.

## Confirmed Input

The `Console.confirm` function asks a yes/no question and returns a `Bool`. The
prompt is followed by a `[Y/n]` or `[y/N]` hint that shows the default, which is
used when the user presses Enter without typing anything. Any answer other than
`y`, `yes`, `n`, or `no` (ignoring case) prints an error and re-prompts. At the
end of input the default is returned:

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
`Err(message)` to print the message and re-prompt. The result is `None` if the
end of input is reached before a line passes:

```flix
use Sys.Console

def main(): Unit \ Console =
    let n = Console.readlnWith(
        "Enter a number (1-10): ",
        s -> match Int32.fromString(s) {
            case Some(i) if i >= 1 and i <= 10 => Ok(i)
            case _                             => Err("Please enter a number between 1 and 10.")
        }
    );
    match n {
        case Some(i) => Console.println("You entered: ${i}")
        case None    => Console.println("No number given.")
    }
```

## Parsed Input

The `Console.readlnAs` function prompts for a value of any type with a
`FromString` instance. It trims the line, parses it, and re-prompts until the
line parses. The type to parse is determined by a type annotation or by how the
result is used:

```flix
use Sys.Console

def main(): Unit \ Console =
    let n: Option[Int32] = Console.readlnAs("Enter an integer: ");
    match n {
        case Some(i) => Console.println("${i} squared is ${i * i}.")
        case None    => Console.println("No integer given.")
    }
```
-->
