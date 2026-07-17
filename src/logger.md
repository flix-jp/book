# Logger

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/logger.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/logger.md)ください。

Flix は、構造化ログ(Structured logging)のためのライブラリエフェクトとして `Logger` を提供しています。`Logger` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。鍵となるモジュールは `Logger` です。

## Logger エフェクト

`Logger` エフェクトは、指定された重大度(Severity)でメッセージをログに記録する、単一の操作を持ちます。

```flix
pub eff Logger {
    /// 指定されたメッセージ `m` を、指定された重大度 `s` でログに記録します。
    def log(s: Severity, m: RichString): Unit
}
```

`Severity` enum は、低いものから高いものまで、5 つのレベルを定義しています。

```flix
pub enum Severity with Eq, Order, ToString {
    case Trace
    case Debug
    case Info
    case Warn
    case Fatal
}
```

## Logger モジュール

`Logger` モジュールは便利な関数を提供しています。

```flix
mod Logger {
    /// メッセージ `m` を Trace レベルでログに記録します。
    def trace(m: a): Unit \ Logger with Formattable[a]

    /// メッセージ `m` を Debug レベルでログに記録します。
    def debug(m: a): Unit \ Logger with Formattable[a]

    /// メッセージ `m` を Info レベルでログに記録します。
    def info(m: a): Unit \ Logger with Formattable[a]

    /// メッセージ `m` を Warn レベルでログに記録します。
    def warn(m: a): Unit \ Logger with Formattable[a]

    /// メッセージ `m` を Fatal レベルでログに記録します。
    def fatal(m: a): Unit \ Logger with Formattable[a]
```

> **注意:** ログ記録用の関数は、`Formattable` トレイトを実装している任意の型を受け取ります。ほとんどの標準的な型（`String`、`Int32`、`Bool` など）は `Formattable` を実装しているため、通常の値をそのままログに記録できます。このトレイトは値を `RichString` に変換します。`RichString` は、スタイル付きのターミナル出力（色、太字など）をサポートしています。

## メッセージのログ記録

これらの便利な関数は、`Formattable` を実装している任意の値を受け取ります。

```flix
def main(): Unit \ { Logger } =
    Logger.info("Application started");
    Logger.debug("Loading configuration...");
    Logger.warn("Cache size exceeds threshold");
    Logger.fatal("Unrecoverable error")
```

デフォルトハンドラは、各メッセージを色付きの重大度プレフィックス付きで標準出力に出力します。例: `[Info] Application started`

<!--
# Logger

Flix provides `Logger` as a library effect for structured logging. The `Logger`
effect has a default handler, so no explicit `runWithIO` call is needed in
`main`. The key module is `Logger`.

## The Logger Effect

The `Logger` effect has a single operation that logs a message at a given
severity:

```flix
pub eff Logger {
    /// Logs the given message `m` at the given severity `s`.
    def log(s: Severity, m: RichString): Unit
}
```

The `Severity` enum defines five levels, from lowest to highest:

```flix
pub enum Severity with Eq, Order, ToString {
    case Trace
    case Debug
    case Info
    case Warn
    case Fatal
}
```

## The Logger Module

The `Logger` module provides convenience functions:

```flix
mod Logger {
    /// Logs the message `m` at the Trace level.
    def trace(m: a): Unit \ Logger with Formattable[a]

    /// Logs the message `m` at the Debug level.
    def debug(m: a): Unit \ Logger with Formattable[a]

    /// Logs the message `m` at the Info level.
    def info(m: a): Unit \ Logger with Formattable[a]

    /// Logs the message `m` at the Warn level.
    def warn(m: a): Unit \ Logger with Formattable[a]

    /// Logs the message `m` at the Fatal level.
    def fatal(m: a): Unit \ Logger with Formattable[a]
```

> **Note:** The logging functions accept any type that implements the
> `Formattable` trait. Most standard types (`String`, `Int32`, `Bool`, etc.)
> implement `Formattable`, so plain values can be logged directly. The trait
> converts values into `RichString`, which supports styled terminal output
> (colors, bold, etc.).

## Logging Messages

The convenience functions accept any value that implements `Formattable`:

```flix
def main(): Unit \ { Logger } =
    Logger.info("Application started");
    Logger.debug("Loading configuration...");
    Logger.warn("Cache size exceeds threshold");
    Logger.fatal("Unrecoverable error")
```

The default handler prints each message to standard output with a colored
severity prefix, for example: `[Info] Application started`.
-->
