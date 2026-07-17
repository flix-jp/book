# モジュール

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/modules.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/modules.md)ください。

Flix は、他の多くのプログラミング言語で知られているような階層的モジュール(Hierarchical module)をサポートしています。

## モジュールの宣言と使用

モジュールは、`mod` キーワードに続けてモジュールの名前空間と名前を書くことで宣言します。

例えば、次のようにモジュールを宣言できます：

```flix
mod Math {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
}
```

ここでは、`sum` という関数を内部に持つ `Math` というモジュールを宣言しました。モジュールの外側からは、完全修飾名(Fully-qualified name)を使って `sum` 関数を参照できます：

```flix
def main(): Unit \ IO = 
    let result = Math.sum(123, 456);
    println(result)
```

あるいは、`use` を使って `sum` 関数をローカルスコープに持ち込むこともできます：

```flix
def main(): Unit \ IO = 
    use Math.sum;
    let result = sum(123, 456);
    println(result)
```

## モジュール内の複数の宣言を使用する

モジュール内に複数の宣言がある場合：

```flix
mod Math {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
    pub def mul(x: Int32, y: Int32): Int32 = x * y
}
```

もちろん、それぞれの宣言を個別に `use` することもできます：

```flix
use Math.sum;
use Math.mul;

def main(): Unit \ IO =
    mul(42, 84) |> sum(21) |> println
```

しかし、複数の `use` を 1 つにまとめる、より短い書き方もあります：

```flix
use Math.{sum, mul};

def main(): Unit \ IO =
    mul(42, 84) |> sum(21) |> println
```

> **注意:** Flix はワイルドカードによる use をサポートしていません。これは、分かりにくいバグにつながる可能性があるためです。

## リネームによる名前衝突の回避

同じ名前を持つ宣言同士の名前衝突は、リネーム(Renaming)を使って回避できます。

例えば、次の 2 つのモジュールがあるとします：

```flix
mod A {
    pub def concat(x: String, y: String): String = x + y
}

mod B {
    pub def concat(xs: List[Int32], ys: List[Int32]): List[Int32] = xs ::: ys
}
```

このとき、それぞれの `concat` 関数を一意な名前で `use` できます。例えば：

```flix
use A.{concat => concatStrings}
use B.{concat => concatLists}

def main(): Unit \ IO =
    concatStrings("Hello", " World!") |> println
```

この機能は強力ですが、多くの場合は完全修飾名を使う方が適切かもしれません。

## モジュールと Enum

モジュールの内部で enum を定義できます。例えば：

```flix
mod Zoo {
    pub enum Animal {
        case Cat,
        case Dog,
        case Fox
    }
}
```

ここで `Zoo` モジュールは、`Cat`、`Dog`、`Fox` という 3 つのケースを持つ `Animal` という enum 型を含んでいます。

型とケースには、完全修飾名を使ってアクセスできます：

```flix
def says(a: Zoo.Animal): String = match a {
    case Zoo.Animal.Cat => "Meow"
    case Zoo.Animal.Dog => "Woof"
    case Zoo.Animal.Fox => "Roar"
}

def main(): Unit \ IO = 
    println("A cat says ${says(Zoo.Animal.Cat)}!")
```

あるいは、`Animal` 型とそのケースの両方を `use` することもできます：

```flix
use Zoo.Animal
use Zoo.Animal.Cat
use Zoo.Animal.Dog
use Zoo.Animal.Fox

def says(a: Animal): String = match a {
    case Animal.Cat => "Meow"
    case Animal.Dog => "Woof"
    case Animal.Fox => "Roar"
}

def main(): Unit \ IO = 
    println("A cat says ${says(Cat)}!")
```

`use Zoo.Animal` は `Animal` の*型*をスコープに持ち込むのに対し、`use Zoo.Animal.Cat` は `Cat` という*ケース*をスコープに持ち込むことに注意してください。

## モジュールとトレイト

モジュールの内部でトレイトを定義することもできます。その仕組みは、モジュール内の enum と同様です。

例えば、次のように書けます：

```flix
mod Zoo {
    pub trait Speakable[t] {
        pub def say(x: t): String
    }
}

enum Animal with ToString {
    case Cat,
    case Dog,
    case Fox
}

instance Zoo.Speakable[Animal] {
    pub def say(a: Animal): String = match a {
        case Cat => "Meow"
        case Dog => "Woof"
        case Fox => "Roar"
    }
}
```

完全修飾名を使えば次のように書けます：

```flix
def speak(x: t): Unit \ IO with Zoo.Speakable[t], ToString[t] = 
    println("A ${x} says ${Zoo.Speakable.say(x)}!")

def main(): Unit \ IO = 
    speak(Animal.Cat)
```

あるいは、`Zoo.Speakable` トレイトと `Zoo.Speakable.say` 関数を `use` することもできます：

```flix
use Zoo.Speakable
use Zoo.Speakable.say

def speak(x: t): Unit \ IO with Speakable[t], ToString[t] = 
    println("A ${x} says ${say(x)}!")
```

<!--
# Modules

Flix supports hierarchical modules as known from many other programming
languages.

## Declaring and Using Modules

We declare modules using the `mod` keyword followed by the namespace and name of
the module. 

For example, we can declare a module:

```flix
mod Math {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
}
```

Here we have declared a module called `Math` with a function called `sum` inside
it. We can refer to the `sum` function, from outside of its module, using its
fully-qualified name:

```flix
def main(): Unit \ IO = 
    let result = Math.sum(123, 456);
    println(result)
```

Alternatively, we can bring the `sum` function into local scope with `use`:

```flix
def main(): Unit \ IO = 
    use Math.sum;
    let result = sum(123, 456);
    println(result)
```

## Using Multiple Declarations from a Module

If we have multiple declarations in a module:

```flix
mod Math {
    pub def sum(x: Int32, y: Int32): Int32 = x + y
    pub def mul(x: Int32, y: Int32): Int32 = x * y
}
```

We can, of course, `use` each declaration:

```flix
use Math.sum;
use Math.mul;

def main(): Unit \ IO =
    mul(42, 84) |> sum(21) |> println
```

but a shorter way is to group the `use`s together into one:

```flix
use Math.{sum, mul};

def main(): Unit \ IO =
    mul(42, 84) |> sum(21) |> println
```

> **Note:** Flix does not support wildcard uses since they can lead to subtle
> bugs.

## Avoiding Name Clashes with Renaming

We can use renaming to avoid name clashes between identically named declarations.

For example, if we have two modules:

```flix
mod A {
    pub def concat(x: String, y: String): String = x + y
}

mod B {
    pub def concat(xs: List[Int32], ys: List[Int32]): List[Int32] = xs ::: ys
}
```

We can then `use` each `concat` function under a unique name. For example:

```flix
use A.{concat => concatStrings}
use B.{concat => concatLists}

def main(): Unit \ IO =
    concatStrings("Hello", " World!") |> println
```

While this feature is powerful, in many cases using a fully-qualified name might be
more appropriate.

## Modules and Enums

We can define an enum inside a module. For example:

```flix
mod Zoo {
    pub enum Animal {
        case Cat,
        case Dog,
        case Fox
    }
}
```

Here the `Zoo` module contains an enum type named `Animal` which has three
cases: `Cat`, `Dog`, and `Fox`. 

We can access the type and the cases using their fully-qualified names:

```flix
def says(a: Zoo.Animal): String = match a {
    case Zoo.Animal.Cat => "Meow"
    case Zoo.Animal.Dog => "Woof"
    case Zoo.Animal.Fox => "Roar"
}

def main(): Unit \ IO = 
    println("A cat says ${says(Zoo.Animal.Cat)}!")
```

Alternatively, we can `use` both the `Animal` type and its cases:

```flix
use Zoo.Animal
use Zoo.Animal.Cat
use Zoo.Animal.Dog
use Zoo.Animal.Fox

def says(a: Animal): String = match a {
    case Animal.Cat => "Meow"
    case Animal.Dog => "Woof"
    case Animal.Fox => "Roar"
}

def main(): Unit \ IO = 
    println("A cat says ${says(Cat)}!")
```

Note that `use Zoo.Animal` brings the `Animal` _type_ into scope, whereas `use
Zoo.Animal.Cat` brings the `Cat` _case_ into scope.

## Modules and Trait

We can also define a trait inside a module. The mechanism is similar to
enums inside modules. 

For example, we can write:

```flix
mod Zoo {
    pub trait Speakable[t] {
        pub def say(x: t): String
    }
}

enum Animal with ToString {
    case Cat,
    case Dog,
    case Fox
}

instance Zoo.Speakable[Animal] {
    pub def say(a: Animal): String = match a {
        case Cat => "Meow"
        case Dog => "Woof"
        case Fox => "Roar"
    }
}
```

We can use fully-qualified names to write:

```flix
def speak(x: t): Unit \ IO with Zoo.Speakable[t], ToString[t] = 
    println("A ${x} says ${Zoo.Speakable.say(x)}!")

def main(): Unit \ IO = 
    speak(Animal.Cat)
```

Or we can `use` the `Zoo.Speakable` trait and the `Zoo.Speakable.say`
function: 

```flix
use Zoo.Speakable
use Zoo.Speakable.say

def speak(x: t): Unit \ IO with Speakable[t], ToString[t] = 
    println("A ${x} says ${say(x)}!")
```
-->
