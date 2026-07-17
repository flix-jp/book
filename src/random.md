# Random

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/random.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/random.md)ください。

Flix は、擬似乱数(Pseudorandom number)を生成するためのライブラリエフェクトとして `Random` を提供しています。`Random` エフェクトはデフォルトハンドラを持っているため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。主要なモジュールは `Math.Random` です。

## Random エフェクト

`Random` エフェクトには 2 つの操作があります：

```flix
pub eff Random {
    /// [0.0, 1.0] の範囲の擬似乱数（64 ビット浮動小数点数）を返します。
    def randomFloat64(): Float64

    /// 擬似乱数（64 ビット整数）を返します。
    def randomInt64(): Int64
}
```

## ランダムな値の生成

`Random` の最も簡単な使い方は、値を生成してそれに応じた処理を行うことです：

```flix
use Math.Random

def main(): Unit \ { Random, IO } =
    let flip = Random.randomFloat64() > 0.5;
    if (flip)
        println("heads")
    else
        println("tails")
```

`Random` はデフォルトハンドラを持っているため、このエフェクトは新しいランダムシードを使って自動的に処理されます。

## シード付きの乱数

`runWithSeed` ハンドラは固定のシード(Seed)を使用するため、実行するたびに同じ乱数列が生成されます。これは、再現可能なテストやベンチマークに役立ちます：

```flix
use Math.Random

def main(): Unit \ IO =
    run {
        let a = Random.randomFloat64();
        let b = Random.randomFloat64();
        println("a = ${a}, b = ${b}")
    } with Random.runWithSeed(42i64)
```

シード付きの乱数生成器は完全に決定的であるため、`runWithSeed` は `IO` を導入することなく `Random` エフェクトを除去します。

<!--
# Random

Flix provides `Random` as a library effect for generating pseudorandom numbers.
The `Random` effect has a default handler, so no explicit `runWithIO` call is
needed in `main`. The key module is `Math.Random`.

## The Random Effect

The `Random` effect has two operations:

```flix
pub eff Random {
    /// Returns a pseudorandom 64-bit floating-point number in [0.0, 1.0].
    def randomFloat64(): Float64

    /// Returns a pseudorandom 64-bit integer.
    def randomInt64(): Int64
}
```

## Generating Random Values

The simplest use of `Random` is to generate a value and act on it:

```flix
use Math.Random

def main(): Unit \ { Random, IO } =
    let flip = Random.randomFloat64() > 0.5;
    if (flip)
        println("heads")
    else
        println("tails")
```

Because `Random` has a default handler, the effect is handled automatically with
a fresh random seed.

## Seeded Randomness

The `runWithSeed` handler uses a fixed seed so that every run produces the same
sequence of random values. This is useful for reproducible tests and
benchmarks:

```flix
use Math.Random

def main(): Unit \ IO =
    run {
        let a = Random.randomFloat64();
        let b = Random.randomFloat64();
        println("a = ${a}, b = ${b}")
    } with Random.runWithSeed(42i64)
```

Because a seeded random number generator is fully deterministic, `runWithSeed`
eliminates the `Random` effect without introducing `IO`.
-->
