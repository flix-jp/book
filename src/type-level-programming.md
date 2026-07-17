# 型レベルプログラミング

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/type-level-programming.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/type-level-programming.md)ください。

> **注意:** この機能は実験的です。プロダクションでは使用しないでください。

このセクションは、型レベルプログラミングと Phantom type(ファントム型)についての予備知識があることを前提としています。

## 型レベル真偽値

Flix 独自の機能のひとつに、_型レベル真偽値式(type-level Boolean formulas)_ のサポートがあります。これは、`true` と `false` が型であるだけでなく、`x and (not y)` のような論理式もまた型であるということを意味します。型レベル真偽値式は型カインド `Bool` を持ちます。2 つの型レベル真偽値式は、式が同値である（すなわち同じ真理値表を持つ）場合に等しくなります。例えば、`true` と `x or not x` という 2 つの型は _同じ型_ です。

型レベル真偽値式は、一般的な Refinement type(篩型)や Dependent type(依存型)ほどの表現力はありませんが、完全な型推論とパラメトリック多相をサポートしています。つまり、非常に扱いやすい（エルゴノミックな）機能です。

型レベル真偽値式を使うことで、プログラムの不変条件を静的に強制できます。

いくつかの例で説明します：

### 人間とヴァンパイア

```flix
///
/// ファントム型レベル真偽値を使って、人が生きているのか、
/// それとも不死者（すなわちヴァンパイア）なのかをモデル化できます。
///
enum Person[_isAlive: Bool] {
    /// 人は名前と年齢を持ち、レコードとしてモデル化されます。
    case P({name = String, age = Int32})
}

///
/// 真偽値 `true` を「生きている」、真偽値 `false` を「不死者
/// （すなわちヴァンパイア）」と解釈します。
///
type alias Alive  = true
type alias Undead = false

///
/// 生まれてきた人は生きています。
///
def born(name: String): Person[Alive] =
    Person.P({name = name, age = 0})

///
/// 生きている人が噛まれるとヴァンパイアになります。
///
/// すでに不死者（すなわちヴァンパイア）である人が再び噛まれることは
/// ないことを、型システムが強制している点に注目してください。
///
def bite(p: Person[Alive]): Person[Undead] = match p {
    /// 実装は重要ではありません。単に人を再構築するだけです。
    case Person.P(r) => Person.P(r)
}

///
/// 2 人は結婚できますが、それは両者とも生きているか、両者とも不死者である場合に限られます。
///
/// （教会はまだ人間とヴァンパイアの結婚を認めていません。）
///
/// 両方の引数が同じ型を持つことを型システムが強制している点に注目してください。
///
def marry(_p1: Person[isAlive], _p2: Person[isAlive]): Unit = ()

///
/// born のより洗練されたバージョンを実装できます。
///
/// 2 人の間に子どもが生まれた場合、どちらか一方がヴァンパイアであれば、その子どももヴァンパイアです。
///
/// ここでは、結果が生きているのか不死者なのかを計算するために、
/// 型レベル計算 `isAlive1 and isAlive2` を使っている点に注目してください。
///
def offspring(p1: Person[isAlive1], p2: Person[isAlive2]): Person[isAlive1 and isAlive2] =
    match (p1, p2) {
        case (Person.P(r1), Person.P(r2)) =>
            Person.P({name = "Spawn of ${r1#name} and ${r2#name}", age = 0})
}

///
/// 人は年を取ります——生きていても不死者であっても。
///
/// この関数は `isAlive` パラメータを保存する点に注目してください。つまり、
/// 生きている人は生きたままです。
///
def birthday(p: Person[isAlive]): Person[isAlive] = match p {
    case Person.P(r) => Person.P({name = r#name, age = r#age + 1})
}
```

型システムが特定の不変条件をどのように強制するかを説明しましょう。

例えば、人が二度噛まれることはないことを型システムが保証します：

```flix
let p = birthday(bite(born("Dracula")));
bite(p);
```

このプログラムをコンパイルすると、Flix コンパイラはコンパイルエラーを出力します：

```
❌ -- Type Error --------------------------------------------------

>> Expected argument of type 'Person[true]', but got 'Person[false]'.

69 |     bite(p);
              ^
              expected: 'Person[true]'

The function 'bite' expects its 1st argument to be of type 'Person[true]'.
```

ここで、`true` は `Alive`（生きている）を意味し、`false` は `Undead`（不死者）を意味することを思い出してください。

<!--
# Type-Level Programming

> **Note:** This feature is experimental. Do not use in production.

This section assumes prior familiarity with type-level programming and phantom
types.

## Type-Level Booleans

A unique Flix feature is its support for _type-level Boolean formulas_. This
means `true` and `false` are types, but also that formulas such as `x and (not
y)` are types. A type-level Boolean formula has kind `Bool`. Two type-level
Boolean formulas are equal if the formulas are equivalent (i.e. have the same
truth tables). For example, the two types `true` and `x or not x` are _the same
type_.

While type-level Boolean formulas are not as expressive as general refinement
types or dependent types they support complete type inference and parametric
polymorphism. This means that they are very ergonomic to work with.

We can use type-level Boolean formulas to statically enforce program invariants.

We illustrate with a few examples:

### Humans and Vampires

```flix
///
/// We can use a phantom type-level Boolean to model whether a person is alive
/// or is undead (i.e. a vampire).
///
enum Person[_isAlive: Bool] {
    /// A person has a name, an age, and is modelled as a record.
    case P({name = String, age = Int32})
}

///
/// We interpret the Boolean `true` is alive and the Boolean `false` as undead
/// (i.e. a vampire).
///
type alias Alive  = true
type alias Undead = false

///
/// A person who is born is alive.
///
def born(name: String): Person[Alive] =
    Person.P({name = name, age = 0})

///
/// A person who is alive and is bitten becomes a vampire.
///
/// Note that the type system enforces that an already undead (i.e. a vampire)
/// cannot be bitten again.
///
def bite(p: Person[Alive]): Person[Undead] = match p {
    /// The implementation is not important; it simply restructs the person.
    case Person.P(r) => Person.P(r)
}

///
/// Two persons can be married, but only if they are both alive or both undead.
///
/// (The church does not yet recognize human-vampire marriages.)
///
/// Note that the type system enforces that both arguments have the same type.
///
def marry(_p1: Person[isAlive], _p2: Person[isAlive]): Unit = ()

///
/// We can implement a more sophisticated version of born.
///
/// If two persons have a child then that child is a vampire if one of them is.
///
/// Note that here we use the type-level computation `isAlive1 and isAlive2`
/// to compute whether the result is alive or undead.
///
def offspring(p1: Person[isAlive1], p2: Person[isAlive2]): Person[isAlive1 and isAlive2] =
    match (p1, p2) {
        case (Person.P(r1), Person.P(r2)) =>
            Person.P({name = "Spawn of ${r1#name} and ${r2#name}", age = 0})
}

///
/// A person can age-- no matter if they are alive or undead.
///
/// Note that this function preserves the `isAlive` parameter. That is, if a
/// person is alive they stay alive.
///
def birthday(p: Person[isAlive]): Person[isAlive] = match p {
    case Person.P(r) => Person.P({name = r#name, age = r#age + 1})
}
```

We can now illustrate how the type system enforces certain invariants.

For example, the type system ensures that person cannot be bitten twice:

```flix
let p = birthday(bite(born("Dracula")));
bite(p);
```

If we compile this program then the Flix compiler emits a compiler error:

```
❌ -- Type Error --------------------------------------------------

>> Expected argument of type 'Person[true]', but got 'Person[false]'.

69 |     bite(p);
              ^
              expected: 'Person[true]'

The function 'bite' expects its 1st argument to be of type 'Person[true]'.
```

Here we have to recall that `true` means `Alive` and `false` means `Undead`.
-->
