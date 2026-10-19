# Monadic For-Yield

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/monadic-for-yield.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/monadic-for-yield.md)ください。

Flix は、Scala の `for` 内包表記や Haskell の do 記法に似た、モナドの _forM-yield_ 構文をサポートしています。_forM_ 構文は、`point` と `flatMap`（これらは `Monad` トレイトによって提供されます）を使用するための糖衣構文です。_forM_ 構文は、`empty`（これは `MonadZero` トレイトによって提供されます）を使用する _guard_ 式もサポートしています。

たとえば、次のモナドの `forM` 式：

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2)
    yield (x, y)
```

は、次のリストに評価されます：

```flix
(1, 1) :: (1, 2) :: (2, 1) :: (2, 2) :: Nil
```

## ガード式を使用する

`forM` 式の中で _ガード式_ を使用することができます。たとえば、次のプログラム：

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2; if x < y)
    yield (x, y)
```

は、次のリストに評価されます：

```flix
(1, 2) :: Nil
```

## Option と Result を扱う

`forM` を使って `Option` データ型を扱うこともできます。たとえば：

```flix
def divide(x: Int32, y: Int32): Option[Int32] = 
    if (y == 0) None else Some(x / y)

def f(): Option[Int32] = 
    forM (
        x <- divide(5, 2);
        y <- divide(x, 8);
        z <- divide(9, y)
    ) yield x + y + z
```

ここで、関数 `f` は `None` を返します。なぜなら `x = 5 / 2 = 2` であり、`2 / 8 = 0` となるため、最後の除算が失敗するからです。

同様に、`forM` を使って `Result[e, t]` データ型を扱うことができます。たとえば：

```flix
use Sys.Console

def readLine(): Result[String, String] \ Console =
    Option.toOk("Unexpected end of input.", Console.readln())

def greet(): Result[String, Unit] \ Console =
    forM (
        fstName <- readLine();
        lstName <- readLine();
        ageLine <- readLine();
        ageNum  <- Int32.parse(10, ageLine)
    ) yield {
        Console.println("Hello ${lstName}, ${fstName}.");
        Console.println("You are ${ageNum} years old!")
    }

def main(): Unit \ Console =
    Console.println("Please enter your first name, last name, and age:");
    match greet() {
        case Ok(_)    => ()
        case Err(msg) => Console.eprintln(msg)
    }
```

ここで `main` は、ユーザーに名（first name）、姓（last name）、年齢の入力を促し、`greet` がそれらを読み取ります。`Console.readln` は `Option[String]` を返し、入力の終端では `None` になるため、`readLine` は `Option.toOk` を使ってこれを `Result[String, String]` に変換します。したがって、ローカル変数 `fstName`、`lstName`、`ageLine` は `String` です。`Int32.parse` を使用して `ageLine` を `Int32` にパースします。これは `Result[String, Int32]` 値を返します。すべての操作が成功した場合、あいさつを表示して `Ok(())`（すなわち `Unit` の `Ok`）を返します。そうでなければ、最初の `Err(msg)` 値を返します。`main` は `Unit` を返さなければならないため、その結果をパターンマッチし、エラーメッセージがあれば出力します。

## その他の Monad を扱う

`forM` は、`Chain` や `Nel`（空でないリスト）を含む、他の種類の `Monad` でも使用できます。たとえば、次のように書くことができます：

```flix
let l1 = Nel(1, 2 :: Nil);
let l2 = Nel(1, 2 :: Nil);
forM (x <- l1; y <- l2)
    yield (x, y)
```

これは、次の空でないリストに評価されます：

```flix
Nel((1, 1), (1, 2) :: (2, 1) :: (2, 2) :: Nil)
```

> **注意:** 空でないリストでは `if`-ガードを使用することができません。なぜなら、そのような `if`-ガードは `MonadZero` トレイトのインスタンスを必要としますが、空でないリストはこれを実装していないからです（そのようなリストは空になることができないため）。

## 脱糖

`forM` 式は、`Monad.flatMap`、`Applicative.point`、`MonadZero.empty` を使用するための糖衣構文です。

たとえば、次の式：

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2; if x < y)
    yield (x, y)
```

は、次のように脱糖されます：

```flix
Monad.flatMap(x -> 
    Monad.flatMap(y -> 
        if (x < y)
            Applicative.point((x, y))
        else 
            MonadZero.empty(), 
    l2), 
l1);
```

<!--
# Monadic For-Yield

Flix supports a monadic _forM-yield_ construct similar to Scala's
for-comprehensions and Haskell's do notation. The _forM_ construct is syntactic
sugar for uses of `point` and `flatMap` (which are provided by the `Monad`
trait). The _forM_ construct also supports a _guard_-expression that uses
`empty` (which is provided by the `MonadZero` trait).

For example, the monadic `forM` expression:

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2)
    yield (x, y)
```

evaluates to the list:

```flix
(1, 1) :: (1, 2) :: (2, 1) :: (2, 2) :: Nil
```

## Using Guard Expressions

We can use _guard expressions_ in `forM` expressions. For example, the program:

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2; if x < y)
    yield (x, y)
```

evaluates to the list:

```flix
(1, 2) :: Nil
```

## Working with Options and Results

We can also use `forM` to work with the `Option` data type. For example:

```flix
def divide(x: Int32, y: Int32): Option[Int32] = 
    if (y == 0) None else Some(x / y)

def f(): Option[Int32] = 
    forM (
        x <- divide(5, 2);
        y <- divide(x, 8);
        z <- divide(9, y)
    ) yield x + y + z
```

Here the function `f` returns `None` since `x = 5 / 2 = 2` and `2 / 8 = 0` hence
the last division fails. 

Similarly, we can use `forM` to work with the `Result[e, t]` data type. For
example:

```flix
use Sys.Console

def readLine(): Result[String, String] \ Console =
    Option.toOk("Unexpected end of input.", Console.readln())

def greet(): Result[String, Unit] \ Console =
    forM (
        fstName <- readLine();
        lstName <- readLine();
        ageLine <- readLine();
        ageNum  <- Int32.parse(10, ageLine)
    ) yield {
        Console.println("Hello ${lstName}, ${fstName}.");
        Console.println("You are ${ageNum} years old!")
    }

def main(): Unit \ Console =
    Console.println("Please enter your first name, last name, and age:");
    match greet() {
        case Ok(_)    => ()
        case Err(msg) => Console.eprintln(msg)
    }
```

Here `main` prompts the user to enter their first name, last name, and age, and
`greet` reads them. `Console.readln` returns an `Option[String]` which is `None`
at the end of input, so `readLine` turns it into a `Result[String, String]` with
`Option.toOk`. Thus the local variables `fstName`, `lstName`, and `ageLine` are
`String`s. We parse `ageLine` into an `Int32` using `Int32.parse`, which returns
a `Result[String, Int32]` value. If every operation is successful then we print
a greeting and return `Ok(())` (i.e., `Ok` of `Unit`). Otherwise, we return the
first `Err(msg)` value. Since `main` must return `Unit`, it matches on the
result and prints the error message, if any.

## Working with Other Monads

We can use `forM` with other types of `Monad`s, including `Chain` and `Nel`s
(non-empty lists). For example, we can write:

```flix
let l1 = Nel(1, 2 :: Nil);
let l2 = Nel(1, 2 :: Nil);
forM (x <- l1; y <- l2)
    yield (x, y)
```

which evaluates to the non-empty list:

```flix
Nel((1, 1), (1, 2) :: (2, 1) :: (2, 2) :: Nil)
```

> **Note:** We cannot use an `if`-guard with non-empty lists because such an
> `if`-guard requires an instance of the `MonadZero` trait which is not
> implemented by non-empty list (since such a list cannot be empty). 

## Desugaring

The `forM` expression is syntactic sugar for uses of `Monad.flatMap`,
`Applicative.point`, and `MonadZero.empty`. 

For example, the expression:

```flix
let l1 = 1 :: 2 :: Nil;
let l2 = 1 :: 2 :: Nil;
forM (x <- l1; y <- l2; if x < y)
    yield (x, y)
```

is de-sugared to:

```flix
Monad.flatMap(x -> 
    Monad.flatMap(y -> 
        if (x < y)
            Applicative.point((x, y))
        else 
            MonadZero.empty(), 
    l2), 
l1);
```
-->
