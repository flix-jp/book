# モジュールの使用

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/using-modules.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/using-modules.md)ください。

すでに見てきたように、`use` 構文はモジュールのメンバーをローカルスコープに持ち込みます。

例えば、次のプログラムがあるとします：

```flix
mod Paint {
    pub enum Color {
        case Red, Green, Blue
    }

    pub type alias Hue = Color

    pub def isWarm(c: Color): Bool =
        match c {
            case Color.Red    => true
            case Color.Green  => false
            case Color.Blue   => false
        }
}
```

このとき、以下の `use` はすべて意味を持ちます：

```flix
use Paint.Color
use Paint.Color.{Red, Green, Blue}
use Paint.Hue
use Paint.isWarm
```

## さまざまな種類の use

Flix は、以下を含む複数の種類の use をサポートしています：

- 名前の修飾付き use：`use Paint.Color`
- 複数の名前の修飾付き use：`use Paint.Color.{Red, Green, Blue}`
- リネームを伴う修飾付き use：`use Paint.{Color => Shade}`
- 複数のリネームを伴う修飾付き use：`use Paint.Color.{Red => R, Green => G, Blue => B}`

リネームは、1 つの名前だけをリネームする場合でも、常に波括弧の中に書きます。

> **注意:** Flix はワイルドカードによる use をサポートしていません。

> **注意:** use は修飾されていなければなりません。`use Paint` と書くことはできませんし、書く必要もありません。`Paint` や `List` のようなトップレベルのモジュールは常にスコープ内にあります。

## パッケージの use

`use` は、プロジェクトが依存している Flix パッケージのモジュールに到達するためにも使えます。`flix.toml` の中でそのパッケージに付けられた名前である mount を書き、続けて `::` を書きます：

```flix
use game::Board                          // パッケージのトップレベルモジュール
use game::Game.Rules                     // ネストされたモジュール
use game::Game.Rules.players             // 関数
use game::{Board, Game}                  // 複数のモジュール
use game::Game.Rules.{players => count}  // リネームを伴う場合
```

`::` はパッケージとその中のモジュールパスを区切るもので、モジュールパス自体はこれまでどおり `.` で区切ります。つまり `use game::Game.Rules` と書き、`use game::Game::Rules` とは書きません。`::` の前後に空白を入れてはならず、1 つの use の中で使えるのは 1 回だけです。

パッケージを指定しない `use` はこれまでどおりの動作をします。`use Chain.Empty` は標準ライブラリに到達し、`use Paint.Color` は自分自身のモジュールに到達します。

> **注意:** `::` を書けるのは `use` の中だけです。式の中で `game::Board.place()` と書いたり、型の中で `game::Board` と書いたりすることはできません。

パッケージがどのようにマウントされるかについては、[依存関係の使用](./using-dependencies.md) を参照してください。

## use はどこに書けるのか？

Flix では、次の 3 つの場所で use を使えます：

- ファイルの先頭
- モジュールの内部
- 関数の内部

ファイルの先頭、またはモジュールの内部に書く use は、そのファイルまたはモジュールの宣言より前に書かなければなりません。

例えば：

```flix
mod A {
    use Chain.Empty
    use Chain.One
    use Chain.Chain
    use Int32.max

    pub def maxValue(c: Chain[Int32]): Int32 = 
        match c {
            case Empty       => 0
            case One(x)      => x
            case Chain(x, y) => max(maxValue(x), maxValue(y))
        }
}
```

これは次のように書くこともできます：

```flix
mod A {
    pub def maxValue(c: Chain[Int32]): Int32 = 
        use Chain.Empty;
        use Chain.One;
        use Chain.Chain;
        use Int32.max;
        match c {
            case Empty       => 0
            case One(x)      => x
            case Chain(x, y) => max(maxValue(x), maxValue(y))
        }
}
```

式の内部ではセミコロンが使われている点に注意してください。

## デフォルトの use

Flix では、いくつかの組み込みコンストラクタが常にスコープ内にあります：

- `List.Nil` と `List.Cons`
- `Option.None` と `Option.Some`
- `Result.Ok` と `Result.Err`

<!--
# Using Modules

As we have already seen, the `use` construct brings members of a module into local scope.

For example, given the program:

```flix
mod Paint {
    pub enum Color {
        case Red, Green, Blue
    }

    pub type alias Hue = Color

    pub def isWarm(c: Color): Bool =
        match c {
            case Color.Red    => true
            case Color.Green  => false
            case Color.Blue   => false
        }
}
```

All of the following `use`s are meaningful:

```flix
use Paint.Color
use Paint.Color.{Red, Green, Blue}
use Paint.Hue
use Paint.isWarm
```

## All Kinds of Uses

Flix supports several kinds of uses, including:

- A qualified use of a name: `use Paint.Color`.
- A qualified use of multiple names: `use Paint.Color.{Red, Green, Blue}`.
- A qualified use with rename: `use Paint.{Color => Shade}`.
- A qualified use with multiple renames: `use Paint.Color.{Red => R, Green => G, Blue => B}`.

A rename is always written inside braces, even when we rename a single name.

> **Note:** Flix does not support wildcard uses.

> **Note:** A use must be qualified. We cannot write `use Paint`, and we do not
> have to: a top-level module, such as `Paint` or `List`, is always in scope.

## Uses of a Package

A `use` can also reach a module of a Flix package that the project depends on. We
write the *mount* of the package, the name it is declared under in `flix.toml`,
followed by `::`:

```flix
use game::Board                          // a top-level module of the package
use game::Game.Rules                     // a nested module
use game::Game.Rules.players             // a function
use game::{Board, Game}                  // several modules
use game::Game.Rules.{players => count}  // with a rename
```

The `::` separates the package from the module path inside it, and the module
path is separated by `.` as always: we write `use game::Game.Rules`, never
`use game::Game::Rules`. There must be no whitespace around the `::`, and it can
occur at most once in a use.

A `use` that names no package is unaffected: `use Chain.Empty` reaches the
standard library, and `use Paint.Color` reaches our own module, exactly as
before.

> **Note:** The `::` can be written in a `use` only. We cannot write
> `game::Board.place()` in an expression, nor `game::Board` in a type.

See [Using Dependencies](./using-dependencies.md) for how a package is mounted.

## Where can Uses Occur?

Flix supports uses in three places:

- At the top of a file.
- Inside modules.
- Inside functions.

A use at the top of a file, or inside a module, must be written before the
declarations of that file or module.

For example:

```flix
mod A {
    use Chain.Empty
    use Chain.One
    use Chain.Chain
    use Int32.max

    pub def maxValue(c: Chain[Int32]): Int32 = 
        match c {
            case Empty       => 0
            case One(x)      => x
            case Chain(x, y) => max(maxValue(x), maxValue(y))
        }
}
```

which can also be written as:

```flix
mod A {
    pub def maxValue(c: Chain[Int32]): Int32 = 
        use Chain.Empty;
        use Chain.One;
        use Chain.Chain;
        use Int32.max;
        match c {
            case Empty       => 0
            case One(x)      => x
            case Chain(x, y) => max(maxValue(x), maxValue(y))
        }
}
```

Note the use of semicolons when inside an expression. 

## Default Uses

In Flix, a few built-in constructors are always in scope:

- `List.Nil` and `List.Cons`.
- `Option.None` and `Option.Some`.
- `Result.Ok` and `Result.Err`.
-->
