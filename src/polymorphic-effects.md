# 多相エフェクト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/polymorphic-effects.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/polymorphic-effects.md)ください。

> **注意:** 多相エフェクトは実験的な機能です。

_多相エフェクト（polymorphic effect）_ とは、1つ以上の型によってパラメータ化されるエフェクトのことです。

例えば、型 `t` の値を emit するエフェクトを次のように宣言できます。

```flix
eff Emit[t] {
    def emit(x: t): Unit
}
```

ここで `Emit` エフェクトは型パラメータ `t` を持ち、これは `emit` 操作の引数の型です。`Emit` を使って、整数や文字列、あるいは他の任意の型の値を emit できます。

例えば、整数の範囲を emit する関数を次のように書けます。

```flix
def range(b: Int32, e: Int32): Unit \ Emit[Int32] =
    if (b >= e)
        ()
    else {
        Emit.emit(b);
        range(b + 1, e)
    }
```

そして、2つの文字列を emit する関数も書けます。

```flix
def greetings(): Unit \ Emit[String] =
    Emit.emit("Hello");
    Emit.emit("World")
```

`range` 関数はエフェクト `Emit[Int32]` を持つのに対し、`greetings` 関数はエフェクト `Emit[String]` を持ちます。この操作は、型引数なしで `Emit.emit` として呼び出します。Flix は、emit する値から型引数を推論します。

> **注意:** 多相エフェクトは、[エフェクト多相](./effect-polymorphism.md)と混同しないようにしてください。多相エフェクトとは、型によってパラメータ化される _エフェクト_ であるのに対し、エフェクト多相な関数とは、エフェクトによってパラメータ化される _関数_ のことです。

## 多相エフェクトのハンドリング

多相エフェクトは、他のエフェクトと同じようにハンドルします。

```flix
def main(): Unit \ IO =
    run {
        range(1, 4)
    } with handler Emit {
        def emit(x, resume) = { println(x); resume() }
    }
```

これは次のように出力します。

```
1
2
3
```

`with handler Emit` は型引数なしで書きます。Flix は、ハンドルしているのが `Emit[Int32]` であり、したがって `x` の型が `Int32` であることを推論します。

ハンドラは、任意の型引数に対して機能することができます。例えば、emit された値をリストに収集する関数を次のように書けます。

```flix
def collect(f: Unit -> Unit \ ef): List[t] \ ef - Emit[t] =
    run {
        f();
        Nil
    } with handler Emit {
        def emit(x, resume) = x :: resume()
    }
```

ここで `collect` は、任意の型 `t` に対してエフェクト `Emit[t]` をハンドルし、`List[t]` を返します。これは `range` と `greetings` のどちらにも使えます。

```flix
def numbers(): List[Int32] = collect(() -> range(1, 4))

def words(): List[String] = collect(() -> greetings())

def main(): Unit \ IO =
    println(numbers());
    println(words())
```

これは次のように出力します。

```
1 :: 2 :: 3 :: Nil
Hello :: World :: Nil
```

## 多相関数

関数は、エフェクトの型引数について多相にすることができます。

```flix
def emitAll(l: List[t]): Unit \ Emit[t] =
    foreach (x <- l)
        Emit.emit(x)
```

ここで `emitAll` は、リストのすべての要素を emit します。`emitAll` を `List[Int32]` で呼び出すと、その呼び出しはエフェクト `Emit[Int32]` を持ち、`List[String]` で呼び出すと、その呼び出しはエフェクト `Emit[String]` を持ちます。

## 複数の型パラメータ

エフェクトは複数の型パラメータを持つことができます。

```flix
eff Ask[q, a] {
    def ask(question: q): a
}

def age(): Int32 \ Ask[String, Int32] =
    Ask.ask("How old are you?")
```

ここで `Ask` エフェクトは、質問の型 `q` と回答の型 `a` によってパラメータ化されています。

型パラメータはエフェクトに属するものであり、操作が独自の型パラメータを宣言することはできません。さらに、エフェクトの各型パラメータは、そのエフェクトの少なくとも1つの操作で使われていなければなりません。

## 関数ごとに1つのインスタンス化

`range` と `greetings` がそうであるように、異なる関数は、多相エフェクトを異なる型引数で使うことができます。しかし、1つの関数の _内部_ では、多相エフェクトはどこでも同じ型引数で使われなければなりません。

例えば、次のように書いた場合を考えます。

```flix
def f(): Unit \ Emit[Int32] + Emit[String] =
    Emit.emit(42);
    Emit.emit("Hello")
```

Flix コンパイラは次のエラーメッセージを出力します。

```
-- Type Error [E6795] -------------------------------------------- src/Main.flix

>> Mismatched type arguments for effect 'Emit': 'Int32' and 'String'.

5 | def f(): Unit \ Emit[Int32] + Emit[String] =
                                  ^^^^^^^^^^^^
                                  mismatched effect type argument.

The effect 'Emit' is used with different types for its 1st type parameter 't'.

Effect One: Emit[Int32]
Effect Two: Emit[String]
```

この制限は、関数全体に適用されます。つまり、そのシグネチャ、その本体（ラムダ式やローカル定義を含む）、そしてその関数の内部で _ハンドルされる_ エフェクトにも適用されます。

例えば、次の `main` 関数は、両方のエフェクトをハンドルしているにもかかわらず、拒否されます。

```flix
def main(): Unit \ IO =
    println(collect(() -> range(1, 4)));
    println(collect(() -> greetings()))
```

問題は、`main` が `Emit[Int32]` と `Emit[String]` の両方を使っていることです。解決策は、上記の `numbers` と `words` で行ったように、それぞれのエフェクトを個別の関数でハンドルすることです。

この制限はまた、あるエフェクトを異なる型引数の同じエフェクトを使ってハンドルする関数を書けないことも意味します。

```flix
def render(f: Unit -> Unit \ Emit[Int32]): Unit \ Emit[String] = ...
```

ここで `render` は、そのシグネチャが `Emit[Int32]` と `Emit[String]` の両方を使っているため、拒否されます。

この制限は、_同じ_ エフェクトを複数回使う場合にのみ関係します。関数は、`Emit[Int32]` と `Ask[String, Int32]` のように、異なる多相エフェクトを自由に使うことができます。

## デフォルトハンドラ

多相エフェクトは、デフォルトハンドラを持つことができます。詳細は、[デフォルトハンドラ](./default-handlers.md)の節で説明します。

<!--
# Polymorphic Effects

> **Note:** Polymorphic effects are an experimental feature.

A _polymorphic effect_ is an effect that is parameterized by one or more types.

For example, we can declare an effect that emits values of type `t`:

```flix
eff Emit[t] {
    def emit(x: t): Unit
}
```

Here the `Emit` effect has the type parameter `t`, which is the type of the
argument of the `emit` operation. We can use `Emit` to emit integers, strings,
or values of any other type.

For example, we can write a function that emits a range of integers:

```flix
def range(b: Int32, e: Int32): Unit \ Emit[Int32] =
    if (b >= e)
        ()
    else {
        Emit.emit(b);
        range(b + 1, e)
    }
```

and a function that emits two strings:

```flix
def greetings(): Unit \ Emit[String] =
    Emit.emit("Hello");
    Emit.emit("World")
```

The `range` function has the effect `Emit[Int32]` whereas the `greetings`
function has the effect `Emit[String]`. We call the operation as `Emit.emit`
without a type argument. Flix infers the type argument from the value that we
emit.

> **Note:** Polymorphic effects should not be confused with [effect
> polymorphism](./effect-polymorphism.md). A polymorphic effect is an _effect_
> that is parameterized by a type, whereas an effect polymorphic function is a
> _function_ that is parameterized by an effect.

## Handling a Polymorphic Effect

We handle a polymorphic effect like any other effect:

```flix
def main(): Unit \ IO =
    run {
        range(1, 4)
    } with handler Emit {
        def emit(x, resume) = { println(x); resume() }
    }
```

which prints:

```
1
2
3
```

We write `with handler Emit` without a type argument. Flix infers that we handle
`Emit[Int32]` and hence that `x` has type `Int32`.

A handler can work for every type argument. For example, we can write a function
that collects the emitted values into a list:

```flix
def collect(f: Unit -> Unit \ ef): List[t] \ ef - Emit[t] =
    run {
        f();
        Nil
    } with handler Emit {
        def emit(x, resume) = x :: resume()
    }
```

Here `collect` handles the effect `Emit[t]`, for any type `t`, and returns a
`List[t]`. We can use it with both `range` and `greetings`:

```flix
def numbers(): List[Int32] = collect(() -> range(1, 4))

def words(): List[String] = collect(() -> greetings())

def main(): Unit \ IO =
    println(numbers());
    println(words())
```

which prints:

```
1 :: 2 :: 3 :: Nil
Hello :: World :: Nil
```

## Polymorphic Functions

A function can be polymorphic in the type argument of an effect:

```flix
def emitAll(l: List[t]): Unit \ Emit[t] =
    foreach (x <- l)
        Emit.emit(x)
```

Here `emitAll` emits every element of a list. If we call `emitAll` with a
`List[Int32]` then the call has the effect `Emit[Int32]`, and if we call it with
a `List[String]` then the call has the effect `Emit[String]`.

## Multiple Type Parameters

An effect can have several type parameters:

```flix
eff Ask[q, a] {
    def ask(question: q): a
}

def age(): Int32 \ Ask[String, Int32] =
    Ask.ask("How old are you?")
```

Here the `Ask` effect is parameterized by the type of the question `q` and by
the type of the answer `a`.

The type parameters belong to the effect: an operation cannot declare type
parameters of its own. Moreover, every type parameter of an effect must be used
by at least one of its operations.

## One Instantiation per Function

Different functions can use a polymorphic effect with different type arguments,
as `range` and `greetings` do. But _within_ a function, a polymorphic effect
must be used with the same type arguments everywhere.

For example, if we write:

```flix
def f(): Unit \ Emit[Int32] + Emit[String] =
    Emit.emit(42);
    Emit.emit("Hello")
```

The Flix compiler emits the error message:

```
-- Type Error [E6795] -------------------------------------------- src/Main.flix

>> Mismatched type arguments for effect 'Emit': 'Int32' and 'String'.

5 | def f(): Unit \ Emit[Int32] + Emit[String] =
                                  ^^^^^^^^^^^^
                                  mismatched effect type argument.

The effect 'Emit' is used with different types for its 1st type parameter 't'.

Effect One: Emit[Int32]
Effect Two: Emit[String]
```

The restriction applies to the whole function: to its signature, to its body
(including lambda expressions and local definitions), and to the effects that
are _handled_ inside the function.

For example, the following `main` function is rejected, even though it handles
both effects:

```flix
def main(): Unit \ IO =
    println(collect(() -> range(1, 4)));
    println(collect(() -> greetings()))
```

The problem is that `main` uses both `Emit[Int32]` and `Emit[String]`. The
solution is to handle each effect in its own function, as we did with `numbers`
and `words` above.

The restriction also means that we cannot write a function that handles an
effect by using the same effect with a different type argument:

```flix
def render(f: Unit -> Unit \ Emit[Int32]): Unit \ Emit[String] = ...
```

Here `render` is rejected because its signature uses both `Emit[Int32]` and
`Emit[String]`.

The restriction only concerns multiple uses of the _same_ effect. A function can
freely use different polymorphic effects, e.g. `Emit[Int32]` and
`Ask[String, Int32]`.

## Default Handlers

A polymorphic effect can have a default handler. We describe the details in the
section on [Default Handlers](./default-handlers.md).
-->
