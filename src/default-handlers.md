# デフォルトハンドラ

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/default-handlers.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/default-handlers.md)ください。

Flix は **デフォルトハンドラ（default handler）** をサポートしています。これは、
エフェクトが、そのエフェクトを `IO` エフェクトへと変換するハンドラを宣言できる
ことを意味します。これにより、`main`（および `@Test` が付与された任意のメソッド）は、
`run-with` ブロックでハンドラを明示的に提供することなく、そのエフェクトを
使えるようになります。

たとえば、次のように書けます。

```flix
use Sys.Env
use Time.Clock

def main(): Unit \ {Clock, Env, Logger} =
    let ts = Clock.currentTime(TimeUnit.Milliseconds);
    let os = Env.getOsName();
    Logger.info("UNIX Timestamp:   ${ts}");
    Logger.info("Operating System: ${os}")

```

これを Flix コンパイラは次のように変換します。

```flix
use Sys.Env
use Time.Clock

def main(): Unit \ IO =
    run {
        let ts = Clock.currentTime(TimeUnit.Milliseconds);
        let os = Env.getOsName();
        Logger.info("UNIX Timestamp:   ${ts}");
        Logger.info("Operating System: ${os}")
    } with Clock.runWithIO
      with Env.runWithIO
      with Logger.runWithIO
```

すなわち、Flix コンパイラは `Clock.runWithIO`、`Env.runWithIO`、
`Logger.runWithIO` の呼び出しを自動的に挿入します。これらは、それぞれの
エフェクトに対するデフォルトハンドラです。

たとえば、`Clock.runWithIO` は次のように宣言されています。

```flix
use Time.Clock

@DefaultHandler
pub def runWithIO(f: Unit -> a \ ef): a \ (ef - Clock) + IO = ...
```

デフォルトハンドラは `@DefaultHandler` アノテーションを使って宣言します。
各エフェクトが持てるデフォルトハンドラは高々 1 つであり、それはそのエフェクトの
コンパニオンモジュール（companion module）の中になければなりません。

デフォルトハンドラは、次の形式のシグネチャを持たなければなりません。

```flix
def runWithIO(f: Unit -> a \ ef): a \ (ef - E) + IO
```
ここで `E` はエフェクトの名前です。

デフォルトハンドラを持つエフェクトは、テストの中で使えます。たとえば：

```flix
@Test
def myTest01(): Unit \ {Assert, Logger} = 
    Logger.info("Running test!");
    Assert.assertEq(expected = 42, 42)
```

## 多相エフェクト

[多相エフェクト](./polymorphic-effects.md) もデフォルトハンドラを持つことができます。たとえば：

```flix
mod Emit {
    pub eff Emit[t] {
        def emit(x: t): Unit
    }

    @DefaultHandler
    pub def runWithIO(f: Unit -> a \ ef): a \ (ef - Emit[t]) + IO =
        run {
            f()
        } with handler Emit {
            def emit(_, resume) = resume()
        }
}

def main(): Unit \ {Emit[Int32], IO} =
    Emit.emit(42);
    println("Done")
```

ここで `Emit` のデフォルトハンドラは、emit された値を単に捨てます。

多相エフェクト `E[t]` のデフォルトハンドラは、次の形式のシグネチャを持たなければなりません。

```flix
def runWithIO(f: Unit -> a \ ef): a \ (ef - E[t]) + IO
```

ここで `t` は型変数です。このシグネチャにはトレイト制約を持たせることができません。例えば `with ToString[t]` を追加することはできません。これは、デフォルトハンドラがすべての型 `t` に対して機能しなければならないためです。これが、上記のデフォルトハンドラが emit された値を表示できない理由です。

<!--
# Default Handlers

Flix supports **default handlers** which means that an effect can declare a
handler that translates the effect into the `IO` effect. This allows `main` (and
any method annotated with `@Test`) to use that effect without explicitly
providing a handler in a `run-with` block.

For example, we can write:

```flix
use Sys.Env
use Time.Clock

def main(): Unit \ {Clock, Env, Logger} =
    let ts = Clock.currentTime(TimeUnit.Milliseconds);
    let os = Env.getOsName();
    Logger.info("UNIX Timestamp:   ${ts}");
    Logger.info("Operating System: ${os}")

```

which the Flix compiler translates to:

```flix
use Sys.Env
use Time.Clock

def main(): Unit \ IO =
    run {
        let ts = Clock.currentTime(TimeUnit.Milliseconds);
        let os = Env.getOsName();
        Logger.info("UNIX Timestamp:   ${ts}");
        Logger.info("Operating System: ${os}")
    } with Clock.runWithIO
      with Env.runWithIO
      with Logger.runWithIO
```

That is, the Flix compiler automatically inserts calls to `Clock.runWithIO`,
`Env.runWithIO`, and `Logger.runWithIO` which are the default handlers for their
respective effects.

For example, `Clock.runWithIO` is declared as:

```flix
use Time.Clock

@DefaultHandler
pub def runWithIO(f: Unit -> a \ ef): a \ (ef - Clock) + IO = ...
```

A default handler is declared using the `@DefaultHandler` annotation. Each
effect may have at most one default handler, and it must reside in the companion
module of that effect.

A default handler must have a signature of the form: 

```flix
def runWithIO(f: Unit -> a \ ef): a \ (ef - E) + IO
```
where `E` is the name of the effect.

We can use effects with default handlers in tests. For example:

```flix
@Test
def myTest01(): Unit \ {Assert, Logger} = 
    Logger.info("Running test!");
    Assert.assertEq(expected = 42, 42)
```

## Polymorphic Effects

A [polymorphic effect](./polymorphic-effects.md) can also have a default
handler. For example:

```flix
mod Emit {
    pub eff Emit[t] {
        def emit(x: t): Unit
    }

    @DefaultHandler
    pub def runWithIO(f: Unit -> a \ ef): a \ (ef - Emit[t]) + IO =
        run {
            f()
        } with handler Emit {
            def emit(_, resume) = resume()
        }
}

def main(): Unit \ {Emit[Int32], IO} =
    Emit.emit(42);
    println("Done")
```

Here the default handler of `Emit` discards the emitted values.

The default handler of a polymorphic effect `E[t]` must have a signature of the
form:

```flix
def runWithIO(f: Unit -> a \ ef): a \ (ef - E[t]) + IO
```

where `t` is a type variable. The signature cannot have trait constraints, e.g.
we cannot add `with ToString[t]`, because a default handler must work for every
type `t`. This is why the default handler above cannot print the emitted values.
-->
