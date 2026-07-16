# 遅延評価

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/laziness.html)を参照してください。

Flix はほとんどの場面で先行評価(Eager evaluation)を用いますが、`lazy` キーワードを使うことで、適切な場面でプログラマが遅延評価(Lazy evaluation)を選択できるようになっています：

```flix
let x: Lazy[Int32] = lazy (1 + 2);
```

この式は、*強制(force)* されるまで評価されません：

```flix
let y: Int32 = force x;
```
> **注意:** `lazy` 構文に与える式は純粋でなければなりません。

> **注意:** すでに評価済みの遅延値を force しても、再度評価されることはありません。

## 遅延データ構造

遅延評価を利用すると、使用されるのに合わせて評価される遅延データ構造(Lazy data structure)を作ることができます。これにより、無限のデータ構造を作ることさえ可能になります。

例えば次に示すのは、1 ずつ増えていく整数の無限長ストリームを実装したデータ構造です：

```flix
mod IntStream {

    enum IntStream { case SCons(Int32, Lazy[IntStream]) }

    pub def from(x: Int32): IntStream =
        IntStream.SCons(x, lazy from(x + 1))
}
```

これをもとに、`map` や `take` といった関数を実装できます：

```flix
    pub def take(n: Int32, s: IntStream): List[Int32] =
        match n {
            case 0 => Nil
            case _ => match s {
                case SCons(h, t) => h :: take(n - 1, force t)
            }
        }

    pub def map(f: Int32 -> Int32, s: IntStream): IntStream =
        match s {
            case SCons(h, t) => IntStream.SCons(f(h), lazy map(f, force t))
        }
```

例えば：

```flix
IntStream.from(42) |> IntStream.map(x -> x + 10) |> IntStream.take(10)
```

は次を返します：

```flix
52 :: 53 :: 54 :: 55 :: 56 :: 57 :: 58 :: 59 :: 60 :: 61 :: Nil
```

Flix は、この機能やそれ以上の機能をすでに実装済みの `DelayList` と `DelayMap` というデータ構造を提供しています：

```flix
DelayList.from(42) |> DelayList.map(x -> x + 10) |> DelayList.take(10)
```

<!--
# Laziness

Flix uses eager evaluation in most circumstances, but allows the programmer to opt-in to lazy evaluation when appropriate with the `lazy` keyword:

```flix
let x: Lazy[Int32] = lazy (1 + 2);
```

The expression won't be evaluated until it's *forced*:

```flix
let y: Int32 = force x;
```
> **Note:** The `lazy` construct requires the expression it's given to be pure.

> **Note:** Forcing a lazy value that's already been evaluated won't evaluate it for a second time.

## Lazy data structures

Laziness can be used to create lazy data structures which are evaluated as they're used. This even allows us to create infinite data structures.

Here for example, is a data structure which implements an infinitely long stream of integers which increase by one each time:

```flix
mod IntStream {

    enum IntStream { case SCons(Int32, Lazy[IntStream]) }

    pub def from(x: Int32): IntStream =
        IntStream.SCons(x, lazy from(x + 1))
}
```

Given this, we can implement functions such as `map` and `take`:

```flix
    pub def take(n: Int32, s: IntStream): List[Int32] =
        match n {
            case 0 => Nil
            case _ => match s {
                case SCons(h, t) => h :: take(n - 1, force t)
            }
        }

    pub def map(f: Int32 -> Int32, s: IntStream): IntStream =
        match s {
            case SCons(h, t) => IntStream.SCons(f(h), lazy map(f, force t))
        }
```

So, for example:

```flix
IntStream.from(42) |> IntStream.map(x -> x + 10) |> IntStream.take(10)
```

Will return:

```flix
52 :: 53 :: 54 :: 55 :: 56 :: 57 :: 58 :: 59 :: 60 :: 61 :: Nil
```

Flix provides `DelayList` and `DelayMap` data structures which already implement this functionality and more:

```flix
DelayList.from(42) |> DelayList.map(x -> x + 10) |> DelayList.take(10)
```
-->
