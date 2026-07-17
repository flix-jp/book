# モジュールの宣言

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/declaring-modules.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/declaring-modules.md)ください。

すでに見てきたように、モジュールは `mod` キーワードを使って宣言できます：

```flix
mod Museum {
    // ... メンバー ...
}
```

モジュールは、他のモジュールの中に入れ子にすることができます：

```flix
mod Museum {
    mod Entrance {
        pub def buyTicket(): Unit \ IO = 
            println("Museum.Entrance.buyTicket() was called.")
    }

    mod Restaurant {
        pub def buyMeal(): Unit \ IO = 
            println("Museum.Restaurant.buyMeal() was called.")
    }

    mod Giftshop {
        pub def buyGift(): Unit \ IO = 
            println("Museum.Giftshop.buyGift() was called.")
    }
}
```

これらのメソッドは、次のように呼び出せます：

```flix
def main(): Unit \ IO = 
    Museum.Entrance.buyTicket();
    Museum.Restaurant.buyMeal();
    Museum.Giftshop.buyGift()
```

あるいは、次のように呼び出すこともできます：

```flix
use Museum.Entrance.buyTicket;
use Museum.Restaurant.buyMeal;
use Museum.Giftshop.buyGift;
def main(): Unit \ IO = 
    buyTicket();
    buyMeal();
    buyGift()
```

## アクセシビリティ

モジュール `A` の中で宣言されたモジュールメンバー `m` は、次のいずれかの場合に別のモジュール `B` からアクセスできます：

- メンバー `m` が公開（`pub`）として宣言されている。
- モジュール `B` が `A` のサブモジュール(sub-module)である。

例えば、次のコードは許可されます：

```flix
mod A {
    mod B {
       pub def g(): Unit \ IO = A.f() // OK
    }

    def f(): Unit \ IO = println("A.f() was called.")
}
```

ここで `f` はモジュール `A` に対してプライベートです。しかし、`B` は `A` のサブモジュールであるため、`B` の内部から `f` にアクセスできます。一方、次のコードは許可され*ません*：

```flix
mod A {
    mod B {
       def g(): Unit \ IO = println("A.B.g() was called.")
    }

    pub def f(): Unit \ IO = A.B.g() // NOT OK
}
```

なぜなら、`g` は `B` に対してプライベートだからです。

<!--
# Declaring Modules

As we have already seen, modules can be declared using the `mod` keyword:

```flix
mod Museum {
    // ... members ...
}
```

We can nest modules inside other modules:

```flix
mod Museum {
    mod Entrance {
        pub def buyTicket(): Unit \ IO = 
            println("Museum.Entrance.buyTicket() was called.")
    }

    mod Restaurant {
        pub def buyMeal(): Unit \ IO = 
            println("Museum.Restaurant.buyMeal() was called.")
    }

    mod Giftshop {
        pub def buyGift(): Unit \ IO = 
            println("Museum.Giftshop.buyGift() was called.")
    }
}
```

We can call these methods as follows:

```flix
def main(): Unit \ IO = 
    Museum.Entrance.buyTicket();
    Museum.Restaurant.buyMeal();
    Museum.Giftshop.buyGift()
```

Or alternatively as follows:

```flix
use Museum.Entrance.buyTicket;
use Museum.Restaurant.buyMeal;
use Museum.Giftshop.buyGift;
def main(): Unit \ IO = 
    buyTicket();
    buyMeal();
    buyGift()
```

## Accessibility

A module member `m` declared in module `A` is accessible from another module `B`
if:

- the member `m` is declared as public (`pub`).
- the module `B` is a sub-module of `A`.

For example, the following is allowed:

```flix
mod A {
    mod B {
       pub def g(): Unit \ IO = A.f() // OK
    }

    def f(): Unit \ IO = println("A.f() was called.")
}
```

Here `f` is private to the module A. However, since `B` is a sub-module of `A`
can access `f` from inside `B`. On the other hand, the following is _not_
allowed:

```flix
mod A {
    mod B {
       def g(): Unit \ IO = println("A.B.g() was called.")
    }

    pub def f(): Unit \ IO = A.B.g() // NOT OK
}
```

because `g` is private to `B`.
-->
