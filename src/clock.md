# Clock

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/clock.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/clock.md)ください。

Flix は、現在の実時間(wall-clock time)を問い合わせるためのライブラリエフェクトとして `Clock` を提供しています。`Clock` エフェクトはデフォルトハンドラを持つため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。中心となるモジュールは `Time.Clock` です。

## Clock エフェクト

`Clock` エフェクトは、エポック(epoch)からの経過時間を指定した単位で返す、単一の操作を持ちます。

```flix
pub eff Clock {
    /// エポックからの経過時間を、指定された時間単位 `u` で返す。
    def currentTime(u: TimeUnit): Int64
}
```

結果の粒度は `TimeUnit` enum によって決まります。

```flix
pub enum TimeUnit with Eq, ToString {
    case Days,
    case Hours,
    case Microseconds,
    case Milliseconds,
    case Minutes,
    case Nanoseconds,
    case Seconds
}
```

## 現在時刻の取得

`Clock` のもっとも単純な使い方は、現在時刻を読み取って表示することです。

```flix
use Time.Clock
use Time.TimeUnit

def main(): Unit \ { Clock, IO } =
    let timestamp = Clock.currentTime(TimeUnit.Milliseconds);
    println("${timestamp} ms since the epoch")
```

`Clock` はデフォルトハンドラを持つため、このエフェクトは自動的に処理されます。

## `now` 関数

`Clock.now` 関数は、`Clock.currentTime(TimeUnit.Milliseconds)` の短縮形です。

```flix
use Time.Clock

def main(): Unit \ { Clock, IO } =
    let before = Clock.now();
    // ... 何らかの処理を行う ...
    let after = Clock.now();
    println("Elapsed: ${after - before} ms")
```

<!--
# Clock

Flix provides `Clock` as a library effect for querying the current wall-clock
time. The `Clock` effect has a default handler, so no explicit `runWithIO` call
is needed in `main`. The key module is `Time.Clock`.

## The Clock Effect

The `Clock` effect has a single operation that returns the time since the epoch
in a given unit:

```flix
pub eff Clock {
    /// Returns a measure of time since the epoch in the given time unit `u`.
    def currentTime(u: TimeUnit): Int64
}
```

The `TimeUnit` enum determines the granularity of the result:

```flix
pub enum TimeUnit with Eq, ToString {
    case Days,
    case Hours,
    case Microseconds,
    case Milliseconds,
    case Minutes,
    case Nanoseconds,
    case Seconds
}
```

## Reading the Current Time

The simplest use of `Clock` is to read the current time and print it:

```flix
use Time.Clock
use Time.TimeUnit

def main(): Unit \ { Clock, IO } =
    let timestamp = Clock.currentTime(TimeUnit.Milliseconds);
    println("${timestamp} ms since the epoch")
```

Because `Clock` has a default handler, the effect is handled automatically.

## The `now` Function

The `Clock.now` function is a shorthand for
`Clock.currentTime(TimeUnit.Milliseconds)`:

```flix
use Time.Clock

def main(): Unit \ { Clock, IO } =
    let before = Clock.now();
    // ... do some work ...
    let after = Clock.now();
    println("Elapsed: ${after - before} ms")
```
-->
