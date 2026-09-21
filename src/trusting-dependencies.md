# 依存関係の信頼

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/trusting-dependencies.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/trusting-dependencies.md)ください。

サプライチェーン攻撃(supply-chain attack)のリスクを軽減するため、Flix はすべてのパッケージの依存関係を、使用できる言語機能を制限する security context(セキュリティコンテキスト) の中でビルドします。

security context は次のとおりです：

| Security Context   | Java 相互運用 | 未検査キャスト | `IO` エフェクト |
|---------------------|---------------|-----------------|------------------|
| `paranoid`          | 禁止          | 禁止            | 禁止             |
| `plain`（デフォルト）| 禁止          | 禁止            | 許可             |
| `unrestricted`      | 許可          | 許可            | 許可             |

`plain` context でビルドされた依存関係は計算を行い、エフェクトを使用できますが、Flix の外側には手を伸ばせません。Java クラスの `import` も、Java のコンストラクタ・メソッド・フィールドの呼び出しも、`unsafe` も、未検査キャストも使えません。`paranoid` context でビルドされた依存関係は、`IO` エフェクトすら使用できません。

これが `plain` と `paranoid` を信頼して依存できる理由です。Java に手を伸ばせず、型システムを迂回することもできないパッケージは、その型とエフェクトが示すことしか行えません。呼び出す関数のシグネチャを読めば、それを呼び出すことで何が起こりうるかがエフェクトシステムからわかります。`paranoid` パッケージが純粋な値を返す場合、内部でどう書かれていようと、ファイルシステムに触れることも、ソケットを開くことも、時刻を読み取ることもできません。`unrestricted` context には、そのような保証がなく、シグネチャはそのコードが何をするかについて何も教えてくれません。

## security context の設定

各依存関係の security context は、マニフェストの中で設定できます：

```toml
[dependencies]
"github:flix/museum"              = { version = "2.1.0", mount = "museum", security = "plain" }
"github:magnus-madsen/helloworld" = { version = "1.3.0", mount = "helloworld", security = "unrestricted" }
```

security context を宣言していない依存関係は、`plain` context でビルドされます。

> **注意:** security context は依存関係に適用されます。私たち自身のコードは、依存しているパッケージに対して何を宣言していようと、常に unrestricted です。

## context がグラフ全体に広がる仕組み

security context は推移的に適用されます。ある依存関係の security context は、その依存関係自身が持つ依存関係にも適用されます。ただし、そのうちの 1 つがより制限の弱い security context を明示的に宣言している場合は例外です。複数の依存関係が同じパッケージを必要とする場合、そのパッケージには、要求された中で最も制限の強い security context が適用されます。

したがって、依存関係は他の何かに依存することでその context から逃れることはできません。`plain` context または `paranoid` context でビルドされたパッケージは、Maven 依存関係や JAR 依存関係を自身で持つことすらできません。Java ライブラリには `unrestricted` context が必要であり、それが見つかった場合 Flix はエラーを報告します。

## context の選び方

推奨されるのは、security context を**指定せず**、デフォルトの `plain` を使うことです。これが柔軟性と安全性の最も良いバランスを提供します。

> **警告:** `unrestricted` は、依存関係と、それが依存するすべてのものが*何でも*できてしまうため、可能な限り避けてください。`unrestricted` な依存関係を含むコードは、ビルドやコンパイルをするだけでもサプライチェーン攻撃にさらされる可能性があります。

## エフェクトを必要とするライブラリを書く

エフェクトを必要とする Flix ライブラリの作者である場合のベストプラクティスは、`IO` エフェクトを直接使うのではなく独自のカスタムエフェクトを導入し、ライブラリを 2 つのパッケージに分割することです：

| パッケージ                | 説明                                    | Security Context |
|---------------------------|-----------------------------------------|-------------------|
| `webserver-lib`           | エフェクトを使ったコア機能              | `plain`           |
| `webserver-lib-handlers`  | Java 相互運用や IO を行うハンドラ       | `unrestricted`    |

このアプローチには、次のような利点があります：

- ほとんどの機能が、信頼できる `plain` の security context にとどまります。
- 安全でないコードは `webserver-lib-handlers` に隔離されるため、レビューが容易になります。
- 提供されたハンドラを信頼できない場合、ユーザーは自分でハンドラを実装できます。

エフェクトとそのハンドラの定義方法については、[エフェクトとハンドラ](./effects-and-handlers.md) を参照してください。

<!--
# Trusting Dependencies

To reduce the risk of supply-chain attacks, Flix builds every package dependency in a
*security context* that limits which language features it may use.

The security contexts are:

| Security Context  | Java Interop | Unchecked Casts | The `IO` Effect |
|--------------------|--------------|-------------------|--------------------|
| `paranoid`         | Forbidden    | Forbidden         | Forbidden          |
| `plain` (default)  | Forbidden    | Forbidden         | Allowed            |
| `unrestricted`     | Allowed      | Allowed           | Allowed            |

A dependency built in the `plain` context can compute and it can use effects, but it
cannot reach outside Flix: no `import` of a Java class, no calls to Java constructors,
methods, or fields, no `unsafe`, and no unchecked casts. A dependency built in the
`paranoid` context cannot even use the `IO` effect.

This is what makes `plain` and `paranoid` safe to depend on: a package that cannot
reach Java, and cannot cast its way around the type system, can only do what its types
and effects say it does. We read the signature of a function we call, and the effect
system tells us what calling it can bring about — a `paranoid` package that returns a
pure value cannot touch the file system, open a socket, or read the clock, however it
is written inside. In the `unrestricted` context we have no such guarantee, and the
signatures tell us nothing about what the code may do.

## Setting the Security Context

We can set the security context of each dependency in the manifest:

```toml
[dependencies]
"github:flix/museum"              = { version = "2.1.0", mount = "museum", security = "plain" }
"github:magnus-madsen/helloworld" = { version = "1.3.0", mount = "helloworld", security = "unrestricted" }
```

A dependency that declares no security context is built in the `plain` context.

> **Note:** Security contexts apply to dependencies. Our own code is always
> unrestricted, whatever we declare for the packages we depend on.

## How Contexts Spread Through the Graph

Security contexts are transitive: the security context of a dependency also applies
to its own dependencies, unless one of them explicitly declares a lesser security
context. If several dependencies require the same package, the package inherits the
most restrictive security context requested.

A dependency can therefore not escape its context by depending on something else. A
package built in the `plain` or `paranoid` context may not even have Maven- or
JAR-dependencies of its own, since Java libraries require the `unrestricted`
context, and Flix reports an error if it finds one.

## Choosing a Context

The recommended approach is to **not** specify a security context, and thus default
to `plain`. It provides the best balance between flexibility and safety.

> **Warning:** Avoid `unrestricted` when possible, as it permits a dependency — and
> everything it depends on — to do *anything*. Even building or compiling code that
> includes `unrestricted` dependencies can by itself expose us to a supply-chain
> attack.

## Writing a Library that Needs Effects

If we are the author of a Flix library that requires effects, the best practice is
to introduce our own custom effects instead of using the `IO` effect directly, and
to split the library into two packages:

| Package                  | Description                           | Security Context |
|--------------------------|---------------------------------------|-------------------|
| `webserver-lib`          | Core functionality using effects      | `plain`           |
| `webserver-lib-handlers` | Handlers that perform Java interop/IO | `unrestricted`    |

This approach has several benefits:

- Most functionality remains in the trusted `plain` security context.
- Unsafe code is isolated in `webserver-lib-handlers` for easier review.
- Users can implement their own handlers if they do not trust the provided ones.

See [Effects and Handlers](./effects-and-handlers.md) for how to define an effect
and its handlers.
-->
