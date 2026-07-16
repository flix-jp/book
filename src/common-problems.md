# よくある問題

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/common-problems.html)を参照してください。

- [ToString is not defined on 'a'](#tostring-is-not-defined-on-a)
- [レコードと複雑なインスタンス](#レコードと複雑なインスタンス)
- [Expected kind 'Bool or Effect' here, but kind 'Type' is used](#expected-kind-bool-or-effect-here-but-kind-type-is-used)

## ToString is not defined on 'a'

次のプログラムを考えます：

```flix
def main(): Unit \ IO =
    let l = Nil;
    println(l)
```

Flix コンパイラは次のように報告します：

```
❌ -- Type Error ---------------------

>> ToString is not defined on a. [...]

3 |     println(l)
        ^^^^^^^^^^
        missing ToString instance
```

問題は、空のリストが任意の `a` に対する多相型 `List[a]` を持つことです。このため、Flix は適切な `ToString` トレイトのインスタンスを選択できません。

解決策は、空のリストの型を指定することです。例えば、次のように書けます：

```flix
def main(): Unit \ IO =
    let l: List[Int32] = Nil;
    println(l)
```

これで問題は解決します。具体的な型 `List[Int32]` に対しては、Flix が `ToString` トレイトのインスタンスを見つけられるからです。

## レコードと複雑なインスタンス

次のプログラムを考えます：

```flix
instance Eq[{fstName = String, lstName = String}]
```

Flix コンパイラは次のように報告します：

```
❌ -- Instance Error --------------------------------------------------

>> Complex instance type '{ fstName = String, lstName = String }' in 'Eq'.

1 | instance Eq[{fstName = String, lstName = String}]
             ^^
             complex instance type
```

これは、少なくとも現時点では、レコード（や Datalog スキーマの行）に対してトレイトインスタンスを定義できないためです。この制限は将来変わるかもしれません。それまでは、レコードを代数的データ型で包む必要があります。例えば：

```flix
enum Person({fstName = String, lstName = String})
```

このようにすれば、`Person` 型に対して `Eq` を実装できます：

```flix
instance Eq[Person] {
    pub def eq(x: Person, y: Person): Bool =
        let Person(r1) = x;
        let Person(r2) = y;
        r1#fstName == r2#fstName and r1#lstName == r2#lstName
}
```

## Expected kind 'Bool or Effect' here, but kind 'Type' is used

次のプログラムを考えます：

```flix
enum A[a, b, ef] {
    case A(a -> b \ ef)
}
```

Flix コンパイラは次のように報告します：

```
❌ -- Kind Error -----------------------------------------------

>> Expected kind 'Bool or Effect' here, but kind 'Type' is used.

2 |     case A(a -> b \ ef)
                        ^^
                        unexpected kind.

Expected kind: Bool or Effect
Actual kind:   Type
```

これは、Flix が注釈のない型変数はすべてカインド `Type` を持つと仮定するためです。しかし上記の例では、`a` と `b` はカインド `Type` を持つべきですが、`ef` はカインド `Bool` を持つべきです。次のように明示的に指定できます：

```flix
enum A[a: Type, b: Type, ef: Bool] {
    case A(a -> b \ ef)
}
```

<!--
# Common Problems

- [ToString is not defined on 'a'](#tostring-is-not-defined-on-a)
- [Records and Complex Instances](#records-and-complex-instances)
- [Expected kind 'Bool or Effect' here, but kind 'Type' is used](#expected-kind-bool-or-effect-here-but-kind-type-is-used)

## ToString is not defined on 'a'

Given the program:

```flix
def main(): Unit \ IO =
    let l = Nil;
    println(l)
```

The Flix compiler reports:

```
❌ -- Type Error ---------------------

>> ToString is not defined on a. [...]

3 |     println(l)
        ^^^^^^^^^^
        missing ToString instance
```

The issue is that the empty list has the polymorphic type: `List[a]` for any
`a`. This means that Flix cannot select the appropriate `ToString` trait
instance.

The solution is to specify the type of the empty list. For example, we can write:

```flix
def main(): Unit \ IO =
    let l: List[Int32] = Nil;
    println(l)
```

which solves the problem because Flix can find an instance of `ToString` trait
for the concrete type `List[Int32]`.

## Records and Complex Instances

Given the program:

```flix
instance Eq[{fstName = String, lstName = String}]
```

The Flix compiler reports:

```
❌ -- Instance Error --------------------------------------------------

>> Complex instance type '{ fstName = String, lstName = String }' in 'Eq'.

1 | instance Eq[{fstName = String, lstName = String}]
             ^^
             complex instance type
```

This is because, at least for the moment, it is not possible to define
trait instances on records (or Datalog schema rows). This may change in the
future. Until then, it is necessary to wrap the record in an algebraic data
type. For example:

```flix
enum Person({fstName = String, lstName = String})
```

and then we can implement `Eq` for the `Person` type:

```flix
instance Eq[Person] {
    pub def eq(x: Person, y: Person): Bool =
        let Person(r1) = x;
        let Person(r2) = y;
        r1#fstName == r2#fstName and r1#lstName == r2#lstName
}
```

## Expected kind 'Bool or Effect' here, but kind 'Type' is used

Given the program:

```flix
enum A[a, b, ef] {
    case A(a -> b \ ef)
}
```

The Flix compiler reports:

```
❌ -- Kind Error -----------------------------------------------

>> Expected kind 'Bool or Effect' here, but kind 'Type' is used.

2 |     case A(a -> b \ ef)
                        ^^
                        unexpected kind.

Expected kind: Bool or Effect
Actual kind:   Type
```

This is because Flix assumes every un-annotated type variable to have kind
`Type`. However, in the above case `a` and `b` should have kind `Type`, but `ef`
should have kind `Bool`. We can make this explicit like so:

```flix
enum A[a: Type, b: Type, ef: Bool] {
    case A(a -> b \ ef)
}
```
-->
