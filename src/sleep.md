# Sleep

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/sleep.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/sleep.md)ください。

Flix は、現在のスレッドを一時停止するためのライブラリエフェクトとして `Sleep` を提供しています。`Sleep` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。中心となるモジュールは `Time.Sleep` です。

## Sleep エフェクト

`Sleep` エフェクトは、次の 1 つの操作を持ちます。

```flix
pub eff Sleep {
    /// 現在のスレッドを、指定された期間 `d` だけスリープさせます。
    def sleep(d: Duration): Unit
}
```

Duration（期間）は、`Time.Duration` モジュールの `seconds`、`milliseconds`、`minutes` などのヘルパー関数を使って作成します。

## 基本的なスリープ

`Sleep` のもっとも単純な使い方は、一定の期間だけ一時停止することです。

```flix
use Time.Duration.seconds
use Time.Sleep

def main(): Unit \ { Sleep, IO } =
    println("Going to sleep...");
    Sleep.sleep(seconds(1));
    println("Woke up!")
```

## 何もしないスリープ

`withNoOp` ハンドラはすべてのスリープをスキップします。これは、遅延を含むコードを実際に待つことなくテストしたい場合に便利です。

```flix
use Time.Duration.seconds
use Time.Sleep

def main(): Unit \ IO =
    run {
        println("Going to sleep...");
        Sleep.sleep(seconds(10));
        println("Woke up instantly!")
    } with Sleep.withNoOp
```

`withNoOp` は `Sleep` エフェクトを完全に処理するため、結果の型にはもはや `Sleep` は含まれません。

## Middleware

`Time.Sleep` モジュールは、いくつかの Middleware(ミドルウェア)ハンドラを提供しています。これらは、スリープの期間をインターセプトして変換したうえで、背後にある `Sleep` エフェクトへ転送します。Middleware は `Sleep` を再送出（re-raise）するため、複数の層を合成できます。

| Middleware          | 説明                                                      |
|---------------------|-----------------------------------------------------------|
| `withConstant`      | すべてのスリープ期間を固定値に置き換えます。              |
| `withScale`         | 各期間に係数を掛けます。                                  |
| `withMaxSleep`      | 個々のスリープを最大値までに制限します。                  |
| `withMinSleep`      | 各スリープが最小値以上になるようにします。                |
| `withMaxTotalSleep` | すべての呼び出しにわたる累積スリープを上限内に抑えます。  |
| `withJitter`        | 各期間にランダムなジッター（±係数）を加えます。           |
| `withLogging`       | `Logger` エフェクトを介して各スリープ期間をログに記録します。 |
| `withCollect`       | スリープする代わりに、すべての期間をリストに収集します。  |

たとえば、`withMaxSleep` は各スリープを最大値までに制限し、`withLogging` は各期間をログに記録します。

```flix
use Time.Duration.{milliseconds, seconds}
use Time.Sleep

def main(): Unit \ { Logger, Sleep, IO } =
    run {
        println("Sleeping for 2 seconds (capped to 500ms)...");
        Sleep.sleep(seconds(2));
        println("Done!")
    } with Sleep.withMaxSleep(milliseconds(500))
      with Sleep.withLogging
```

## Middleware の合成

各 Middleware は `Sleep` を再送出するため、自然に積み重ねる（スタックする）ことができます。次の例では、各スリープに ±20% のランダムなジッターを加え、その結果の期間をログに記録します。

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

/// `withJitter` と `withLogging` を合成して、スリープ期間に ±20% の
/// ランダムなジッターを加え、各スリープを `Logger` エフェクトを介してログに記録します。
def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

順序が重要です。`withJitter` は元の期間をインターセプトしてジッターを適用し、`Sleep` を再送出します。その後、`withLogging` ハンドラはジッターが適用された期間を見ることになります。

<!--
# Sleep

Flix provides `Sleep` as a library effect for pausing the current thread. The
`Sleep` effect has a default handler, so no explicit `runWithIO` call is needed
in `main`. The key module is `Time.Sleep`.

## The Sleep Effect

The `Sleep` effect has a single operation:

```flix
pub eff Sleep {
    /// Sleeps the current thread for the given duration `d`.
    def sleep(d: Duration): Unit
}
```

Durations are created with helpers from the `Time.Duration` module, such as
`seconds`, `milliseconds`, `minutes`, and so on.

## Basic Sleep

The simplest use of `Sleep` is to pause for a fixed duration:

```flix
use Time.Duration.seconds
use Time.Sleep

def main(): Unit \ { Sleep, IO } =
    println("Going to sleep...");
    Sleep.sleep(seconds(1));
    println("Woke up!")
```

## No-Op Sleep

The `withNoOp` handler skips all sleeps, which is useful for testing code that
contains delays without actually waiting:

```flix
use Time.Duration.seconds
use Time.Sleep

def main(): Unit \ IO =
    run {
        println("Going to sleep...");
        Sleep.sleep(seconds(10));
        println("Woke up instantly!")
    } with Sleep.withNoOp
```

Because `withNoOp` fully handles the `Sleep` effect, the result type no longer
includes `Sleep`.

## Middleware

The `Time.Sleep` module provides several middleware handlers that intercept and
transform sleep durations before forwarding to the underlying `Sleep` effect.
Middleware re-raises `Sleep`, so multiple layers can be composed.

| Middleware          | Description                                               |
|---------------------|-----------------------------------------------------------|
| `withConstant`      | Replaces every sleep duration with a fixed value.         |
| `withScale`         | Multiplies each duration by a factor.                     |
| `withMaxSleep`      | Caps each individual sleep to a maximum.                  |
| `withMinSleep`      | Ensures each sleep is at least a minimum.                 |
| `withMaxTotalSleep` | Caps the cumulative sleep across all calls to a budget.   |
| `withJitter`        | Adds random jitter (±factor) to each duration.            |
| `withLogging`       | Logs each sleep duration via the `Logger` effect.         |
| `withCollect`       | Collects all durations into a list instead of sleeping.   |

For example, `withMaxSleep` caps each sleep to a maximum and `withLogging` logs
each duration:

```flix
use Time.Duration.{milliseconds, seconds}
use Time.Sleep

def main(): Unit \ { Logger, Sleep, IO } =
    run {
        println("Sleeping for 2 seconds (capped to 500ms)...");
        Sleep.sleep(seconds(2));
        println("Done!")
    } with Sleep.withMaxSleep(milliseconds(500))
      with Sleep.withLogging
```

## Composing Middleware

Because each middleware re-raises `Sleep`, they stack naturally. The following
example adds ±20% random jitter to each sleep and logs the resulting durations:

```flix
use Math.Random
use Time.Duration.{seconds}
use Time.Sleep

/// Composes `withJitter` and `withLogging` to add ±20% random jitter
/// to sleep durations and log each sleep via the `Logger` effect.
def main(): Unit \ { Logger, Random, Sleep, IO } =
    run {
        println("Sleeping 3 times with ±20% jitter...");
        Sleep.sleep(seconds(1));
        Sleep.sleep(seconds(2));
        Sleep.sleep(seconds(3));
        println("Done!")
    } with Sleep.withJitter(0.2)
      with Sleep.withLogging
```

The order matters: `withJitter` intercepts the original durations, applies
jitter, and re-raises `Sleep`. The `withLogging` handler then sees the
jittered durations.
-->
