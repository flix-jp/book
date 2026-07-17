# コンパニオンモジュール

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/companion-modules.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/companion-modules.md)ください。

モジュールの内部では、そのモジュールと同じ名前を持つ enum、struct、エフェクト、またはトレイトを宣言できます。このような宣言を、そのモジュールの companion(コンパニオン) と呼びます。

例：

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }
}
```

ここでは、`Color` enum が `Color` モジュールの companion です。

companion の名前はモジュールからエクスポートされます。つまり、`Color` はモジュールと enum の両方を指すことができます。case は `Color.Red` としても `Color.Color.Red` としても参照できます。

companion は、そのモジュール内の他のどの宣言よりも前に置かなければなりません。そうでない場合、コンパイラはエラーを報告します。

## enum のコンパニオン

enum がモジュールの companion として宣言されている場合、その型と case はモジュール全体で自動的に利用可能になります：

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }

    pub def isWarm(c: Color): Bool = match c {
        case Red    => true
        case Green  => false
        case Blue   => false
    }
}
```

ここでは、companion である `Color` モジュールの内部で、`Color` 型と `Red`、`Green`、`Blue` の各 case がスコープに入っています。

## struct のコンパニオン

struct もモジュールの companion として宣言できます。struct のフィールドはそのコンパニオンモジュールの内部からしか見えないため、フィールドを読み書きする関数はすべてそこに置く必要があります。

例：

```flix
mod Point {
    pub struct Point[r] {
        x: Int32,
        mut y: Int32
    }

    pub def area(p: Point[r]): Int32 \ r = p->x * p->y
}
```

ここで `area` が `x` と `y` のフィールドにアクセスできるのは、それが `Point` のコンパニオンモジュールの内部にあるからです。フィールドの可視性の詳細については、[構造体](structs.md)を参照してください。

## エフェクトのコンパニオン

エフェクトもモジュールの companion として宣言できます。そのエフェクトにデフォルトハンドラがある場合、それは同じコンパニオンモジュールに置きます：

```flix
mod Fs.Glob {
    pub eff Glob {
        def glob(base: String, pattern: String): Result[IoError, List[String]]
    }

    // エフェクトのハンドラや補助関数はここに置きます。
}
```

## トレイトのコンパニオン

トレイトもモジュールの companion として宣言できます。通常、トレイトに関連する機能を格納する場所としてコンパニオンモジュールを使います：

```flix
mod Addable {
    pub trait Addable[t] {
        pub def add(x: t, y: t): t
    }

    pub def add3(x: t, y: t, z: t): t with Addable[t] = add(add(x, y), z)
}
```

`Addable` のメンバーにアクセスする際、Flix はトレイト宣言とそのコンパニオンモジュールの両方を自動的に探索します。その結果、`Addable.add` はトレイトのメンバーである `add` を指し、`Addable.add3` は `Addable` モジュール内の関数を指します。

注意すべき点として、トレイトのコンパニオンモジュールに定義された関数は、そのトレイトのインスタンスによって再定義することができません。したがって、後から再定義するつもりのないメンバーだけをコンパニオンモジュールに置くべきです。

## コンパニオンモジュール内のインスタンス

トレイトのインスタンスは、その型のコンパニオンモジュール内で宣言できます。例えば、`Size` enum に対する `Add`、`Sub`、`ToString` のインスタンスは、enum 自体と並べて配置します：

```flix
mod Fs.Size {
    pub enum Size(Int64) with Eq, Order, Hash

    instance Add[Size] {
        pub def add(x: Size, y: Size): Size =
            let Size(x1) = x;
            let Size(y1) = y;
            Size(x1 + y1)
    }

    pub def zero(): Size = Size(0i64)
}
```

トレイトが別の場所で定義されている場合、インスタンスを置く場所としてはここが推奨されます。

<!--
# Companion Modules

Inside a module we can declare an enum, struct, effect, or trait with the
same name as the module. Such a declaration is called the _companion_ of the
module.

For example:

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }
}
```

Here the `Color` enum is the companion of the `Color` module.

The companion's name is exported from the module. This means that `Color` can
refer to both the module and the enum. We can refer to a case as `Color.Red`
or as `Color.Color.Red`.

The companion must appear before any other declaration inside its module.
Otherwise the compiler raises an error.

## Enum Companions

When an enum is declared as the companion of its module, the type and its
cases are automatically available throughout the module:

```flix
mod Color {
    pub enum Color {
        case Red,
        case Green,
        case Blue
    }

    pub def isWarm(c: Color): Bool = match c {
        case Red    => true
        case Green  => false
        case Blue   => false
    }
}
```

Here the `Color` type and the `Red`, `Green`, and `Blue` cases are in scope
within the companion `Color` module.

## Struct Companions

A struct may also be declared as the companion of its module. The fields of a
struct are only visible from within its companion module, so any function that
reads or writes them must live there.

For example:

```flix
mod Point {
    pub struct Point[r] {
        x: Int32,
        mut y: Int32
    }

    pub def area(p: Point[r]): Int32 \ r = p->x * p->y
}
```

Here `area` can access the `x` and `y` fields because it lives inside the
companion module of `Point`. See [Structs](structs.md) for more on field
visibility.

## Effect Companions

An effect may be declared as the companion of its module. The default handler
for the effect, if any, lives in the same companion module:

```flix
mod Fs.Glob {
    pub eff Glob {
        def glob(base: String, pattern: String): Result[IoError, List[String]]
    }

    // Handlers and helpers for the effect go here.
}
```

## Trait Companions

A trait may also be declared as the companion of its module. We typically use
the companion module to store functionality that is related to the trait:

```flix
mod Addable {
    pub trait Addable[t] {
        pub def add(x: t, y: t): t
    }

    pub def add3(x: t, y: t, z: t): t with Addable[t] = add(add(x, y), z)
}
```

When accessing a member of `Addable`, Flix automatically looks in both the
trait declaration and its companion module. Consequently, `Addable.add` refers
to the trait member `add` whereas `Addable.add3` refers to the function inside
the `Addable` module.

We should be aware that functions defined in the companion module of a trait
cannot be redefined by instances of the associated trait. Thus we should only
put members into the companion module when we do not intend to redefine them
later.

## Instances in Companion Modules

A trait instance may be declared in the companion module of its type. For
example, instances of `Add`, `Sub`, and `ToString` for the `Size` enum are
placed alongside the enum itself:

```flix
mod Fs.Size {
    pub enum Size(Int64) with Eq, Order, Hash

    instance Add[Size] {
        pub def add(x: Size, y: Size): Size =
            let Size(x1) = x;
            let Size(y1) = y;
            Size(x1 + y1)
    }

    pub def zero(): Size = Size(0i64)
}
```

This is the recommended location for instances when the trait is defined
elsewhere.
-->
