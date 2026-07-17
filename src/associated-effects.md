# 関連エフェクト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/associated-effects.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/associated-effects.md)ください。

これまで、関連型(Associated types)によって、各インスタンスが関連型に対する具体的な型を指定できるようになり、トレイトの柔軟性が高まることを見てきました。関連*エフェクト*(Associated effects)も同じ仕組みで動作しますが、対象となるのはエフェクトです。

関連エフェクトの必要性を、簡単な例で説明します。

割り算ができる型のためのトレイトを定義できます：

```flix
trait Dividable[t] {
    pub def div(x: t, y: t): t
}
```

そして、例えば `Float32` や `Int32` に対してこのトレイトを実装できます：

```flix
instance Dividable[Float32] {
    pub def div(x: Float32, y: Float32): Float32 = x / y
}

instance Dividable[Int32] {
    pub def div(x: Int32, y: Int32): Int32 = x / y
}
```

しかし、ゼロ除算はどうすればよいのでしょうか？例外を発生させ、それを型・エフェクトシステムに追跡させたいとします。次のように書きたいところです：

```flix
pub eff DivByZero {
    pub def raise(): Void
}

instance Dividable[Int32] {
    pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
        if (y == 0) DivByZero.raise() else x / y
}
````

しかし残念ながら、これはうまくいきません：

```
❌ -- Type Error --------------------------------------------------

>> Mismatched signature 'div' required by 'Dividable'.

14 |     pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
                 ^^^
...
```

コンパイラが説明しているとおり、問題はトレイト `Dividable` における `div` の定義が純粋（pure）として宣言されていることです。そのため、例外を発生させることは許されません。`Dividable.div` のシグネチャを変更することもできますが、それは `Float32` インスタンスにとって問題になります。なぜなら、`Float32` のゼロ除算は `NaN` を返すのであって、例外を発生させないからです。

解決策は、関連エフェクトを使うことです。そうすれば、`Int32` のインスタンスは `DivByZero` 例外が発生する可能性があることを指定でき、一方で `Float32` のインスタンスは純粋のままでいられます。`Dividable` に関連エフェクト `Aef` を追加します：

```flix
trait Dividable[t] {
    type Aef: Eff
    pub def div(x: t, y: t): t \ Dividable.Aef[t]
}
```

そして、`Float32` と `Int32` のインスタンスを再実装します：

```flix
instance Dividable[Float32] {
    type Aef = { Pure } // 例外なし。ゼロ除算は NaN を返します。
    pub def div(x: Float32, y: Float32): Float32 = x / y
}

instance Dividable[Int32] {
    type Aef = { DivByZero }
    pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
        if (y == 0) DivByZero.raise() else x / y
}
```

## 関連エフェクトとリージョン

関連エフェクトは、リージョンと組み合わせて使いたくなることがよくあります。

以前登場した `ForEach` トレイトがあるとします：

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

これまで見てきたように、このトレイトは例えば `List[t]` や `Map[k, v]` に対して実装できます。しかし、`MutList[t, r]` や `MutSet[t, r]` などに対して実装したい場合はどうでしょうか。試してみましょう：

```flix
instance ForEach[MutList[t, r]] {
    type Elm = t
    pub def forEach(f: t -> Unit \ ef, x: MutList[t, r]): Unit \ ef = 
        MutList.forEach(f, x)
}
```

しかし、Flix は次のように報告します：

```
❌ -- Type Error -------------------------------------------------- 

>> Unable to unify the effect formulas: 'ef' and 'ef + r'.

9 |         MutList.forEach(f, x)
            ^^^^^^^^^^^^^^^^^^^^^
            mismatched effect formulas.
```

問題は、`MutList.forEach` がリージョン `r` におけるエフェクトを持っているのに対して、トレイトの `forEach` のシグネチャは関数 `f` に由来する `ef` エフェクトしか許可していないことです。

`ForEach` トレイトを関連エフェクトで拡張することで、この問題を解決できます：

```flix
trait ForEach[t] {
    type Elm
    type Aef: Eff
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef + ForEach.Aef[t]
}
```

`Aef` がエフェクトであることを、カインド注釈(Kind annotation) `Aef: Eff` によって指定する必要があります。カインドを指定しない場合はデフォルトで `Type` になりますが、ここで求めているのはそれではありません。

更新した `ForEach` トレイトを使えば、`List[t]` と `MutList[t]` の両方に対して実装できます：

```flix
instance ForEach[List[t]] {
    type Elm = t
    type Aef = { Pure }
    pub def forEach(f: t -> Unit \ ef, x: List[t]): Unit \ ef = List.forEach(f, x)
}
```

そして、

```flix
instance ForEach[MutList[t, r]] {
    type Elm = t
    type Aef = { r }
    pub def forEach(f: t -> Unit \ ef, x: MutList[t, r]): Unit \ ef + r = 
        MutList.forEach(f, x)
}
```

`List[t]` の実装では関連エフェクトが純粋であると指定しているのに対し、`MutList[t, r]` の実装ではリージョン `r` におけるヒープエフェクトがあると指定していることに注目してください。

<!--
# Associated Effects

We have seen how associated types increase the flexibility of traits by allowing
each instance to specify concrete types for the associated types. Associated
_effects_ work in the same manner, but concern effects. 

We motivate the need for associated effects with a simple example.

We can define a trait for types that can be divded:

```flix
trait Dividable[t] {
    pub def div(x: t, y: t): t
}
```

and we can implement the trait for e.g. `Float32` and `Int32`:

```flix
instance Dividable[Float32] {
    pub def div(x: Float32, y: Float32): Float32 = x / y
}

instance Dividable[Int32] {
    pub def div(x: Int32, y: Int32): Int32 = x / y
}
```

But what about division-by-zero? Assume we want to raise an exception and have
it tracked by the type and effect system. We would like to write:

```flix
pub eff DivByZero {
    pub def raise(): Void
}

instance Dividable[Int32] {
    pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
        if (y == 0) DivByZero.raise() else x / y
}
````

But unfortunately this does not quite work:

```
❌ -- Type Error --------------------------------------------------

>> Mismatched signature 'div' required by 'Dividable'.

14 |     pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
                 ^^^
...
```

The problem, as the compiler explains, is that the definition of `div` in the
trait `Dividable` is declared as pure. Hence we are not allowed to raise an
exception. We could change the signature of `Dividable.div`, but that would be
problematic for the `Float32` instance, because division-by-zero returns `NaN`
and does not raise an exception. 

The solution is to use an associated effect: then the instance for `Int32` can
specify that a `DivByZero` exception may be raised whereas the instance for
`Float32` can be pure. We add an associated effect `Aef` to `Dividable`: 

```flix
trait Dividable[t] {
    type Aef: Eff
    pub def div(x: t, y: t): t \ Dividable.Aef[t]
}
```

and we re-implement the instances for `Float32` and `Int32`:

```flix
instance Dividable[Float32] {
    type Aef = { Pure } // No exception, div-by-zero yields NaN.
    pub def div(x: Float32, y: Float32): Float32 = x / y
}

instance Dividable[Int32] {
    type Aef = { DivByZero }
    pub def div(x: Int32, y: Int32): Int32 \ DivByZero = 
        if (y == 0) DivByZero.raise() else x / y
}
```

## Associated Effects and Regions

We often want to use associated effects in combination with regions.

Assume we have the `ForEach` trait from before:

```flix
trait ForEach[t] {
    type Elm
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef
}
```

As we have seen, we can implement it for e.g. `List[t]` but also `Map[k, v]`.
But what if we wanted to implement it for e.g. `MutList[t, r]` or `MutSet[t,
r]`. We can try: 

```flix
instance ForEach[MutList[t, r]] {
    type Elm = t
    pub def forEach(f: t -> Unit \ ef, x: MutList[t, r]): Unit \ ef = 
        MutList.forEach(f, x)
}
```

But Flix reports:

```
❌ -- Type Error -------------------------------------------------- 

>> Unable to unify the effect formulas: 'ef' and 'ef + r'.

9 |         MutList.forEach(f, x)
            ^^^^^^^^^^^^^^^^^^^^^
            mismatched effect formulas.
```

The problem is that `MutList.forEach` has an effect in the region `r`, but the
signature of `forEach` in the trait only permits the `ef` effect from the
function `f`. 

We can solve the problem by extending the `ForEach` trait with an associated effect:

```flix
trait ForEach[t] {
    type Elm
    type Aef: Eff
    pub def forEach(f: ForEach.Elm[t] -> Unit \ ef, x: t): Unit \ ef + ForEach.Aef[t]
}
```

We must specify that `Aef` is an effect with the kind annotation `Aef: Eff`. If
we don't specify the kind then it defaults to `Type` which is not what we want
here. 

With the updated `ForEach` trait, we can implement it for both `List[t]` and
`MutList[t]`:

```flix
instance ForEach[List[t]] {
    type Elm = t
    type Aef = { Pure }
    pub def forEach(f: t -> Unit \ ef, x: List[t]): Unit \ ef = List.forEach(f, x)
}
```

and 

```flix
instance ForEach[MutList[t, r]] {
    type Elm = t
    type Aef = { r }
    pub def forEach(f: t -> Unit \ ef, x: MutList[t, r]): Unit \ ef + r = 
        MutList.forEach(f, x)
}
```

Notice how the implementation for `List[t]` specifies that the associated effect
is pure, whereas the implementation for `MutList[t, r]` specifies that there is
a heap effect in region `r`. 
-->
