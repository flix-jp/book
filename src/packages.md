# パッケージ管理

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/packages.html)を参照してください。

自明でない Flix プロジェクトには、必ず `flix.toml` という manifest(マニフェスト)を用意すべきです。マニフェストには、プロジェクトとその依存関係に関する情報が記述されます。

最小構成のマニフェストは次のような形式です：

```toml
[package]
name        = "hello-library"
description = "A simple library"
version     = "0.1.0"
flix        = "0.35.0"
license     = "Apache-2.0"
authors     = ["John Doe <john@example.com>"]
```

> **注意:** `flix` フィールドはまだ使用されていませんが、将来的に使用される予定です。

## Flix の依存関係を追加する

マニフェストには、他の Flix パッケージへの依存関係を追加できます：

```toml
[dependencies]
"github:flix/museum"              = "1.4.0"
"github:magnus-madsen/helloworld" = "1.3.0"
```

> **注意:** Flix では、バージョン番号は SemVer に従う必要があります。

## Maven の依存関係を追加する

マニフェストには、Maven パッケージへの依存関係を追加することもできます：

```toml
[mvn-dependencies]
"org.junit.jupiter:junit-jupiter-api" = "5.9.2"
```

## 依存関係解決を理解する

Flix の依存関係解決(dependency resolution)は、次のように動作します：

1. Flix は `flix.toml` を読み込み、Flix パッケージの依存関係の推移的な集合を計算します。
2. Flix はこれらの Flix パッケージをすべてダウンロードします。
3. Flix は各パッケージを調べて Maven の依存関係を特定し、それらをダウンロードします。

例で説明しましょう。次のような依存関係を持つ Flix パッケージがあるとします：

```toml
[dependencies]
"github:flix/museum"              = "1.4.0"
```

Flix を実行すると、次の出力が得られます：

```
Found `flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v1.4.0)... OK.
  Downloading `flix/museum-entrance.toml` (v1.2.0)... OK.
  Downloading `flix/museum-giftshop.toml` (v1.1.0)... OK.
  Downloading `flix/museum-restaurant.toml` (v1.1.0)... OK.
  Downloading `flix/museum-clerk.toml` (v1.1.0)... OK.
  Cached `flix/museum-clerk.toml` (v1.1.0).
Downloading Flix dependencies...
  Downloading `flix/museum.fpkg` (v1.4.0)... OK.
  Downloading `flix/museum-entrance.fpkg` (v1.2.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v1.1.0)... OK.
  Downloading `flix/museum-restaurant.fpkg` (v1.1.0)... OK.
  Downloading `flix/museum-clerk.fpkg` (v1.1.0)... OK.
  Cached `flix/museum-clerk.fpkg` (v1.1.0).
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (3.12.0).
  Running Maven dependency resolver.
Dependency resolution completed.
```

これは、`flix/museum` が次のような依存関係ツリーを持っているためです：

- `flix/museum` は以下に依存します：
    - `flix/museum-entrance` は以下に依存します：
        - `flix/museum-clerk`
    - `flix/museum-giftshop` は以下に依存します：
        - `flix/museum-clerk`
    - `flix/museum-restaurant` は以下に依存します：
        - `org.apache.commons:commons-lang3`

## セキュリティ

サプライチェーン攻撃(supply-chain attack)のリスクを軽減するため、すべての依存関係には **security context(セキュリティコンテキスト)** が設定されます。これは、明示的に設定しなかった場合でも同様です。security context は、依存関係が使用できる言語機能を制御します。より広い security context を許可すると使える機能は増えますが、その分サプライチェーン攻撃のリスクも高まります。

security context は次のように定義されています：

| Security Context | Java 相互運用 | 未検査キャスト  | `IO` エフェクト |
|------------------|--------------|-----------------|-----------------|
| `paranoid`       | 禁止         | 禁止            | 禁止            |
| `plain`（デフォルト）| 禁止      | 禁止            | 許可            |
| `unrestricted`   | 許可         | 許可            | 許可            |

各依存関係の security context は、マニフェスト内で次のように設定できます：
```toml
[dependencies]
"github:flix/museum"              = { version = "1.4.0", security = "plain" }
"github:magnus-madsen/helloworld" = { version = "1.3.0", security = "unrestricted" }
```

security context は推移的に適用されます。つまり、ある依存関係の security context は、その推移的依存関係にも適用されます。ただし、依存関係がより制限の強い security context を明示的に宣言している場合は例外です。複数の依存関係が同じライブラリを必要とする場合、そのライブラリには、要求された中で最も制限の強い security context が適用されます。

推奨されるのは、security context を**指定せず**、デフォルトの `plain` を使うことです。これが柔軟性と安全性の最も良いバランスを提供します。`unrestricted` は、（推移的な）依存関係が*何でも*できてしまうため、可能な限り避けるべきです。`unrestricted` な依存関係を含むコードは、ビルドやコンパイルをするだけでもサプライチェーン攻撃にさらされる可能性があります。

エフェクトを必要とする Flix ライブラリの作者である場合のベストプラクティスは、`IO` エフェクトを直接使うのではなく独自のカスタムエフェクトを導入し、ライブラリを 2 つのパッケージに分割することです：

| パッケージ                | 説明                                    | Security Context |
|--------------------------|---------------------------------------|------------------|
| `webserver-lib`          | エフェクトを使ったコア機能              | `plain`          |
| `webserver-lib-handlers` | Java 相互運用や IO を行うハンドラ       | `unrestricted`   |

このアプローチには、次のような利点があります：
- ほとんどの機能が、信頼できる `plain` の security context にとどまります。
- 安全でないコードは `webserver-lib-handlers` に隔離されるため、レビューが容易になります。
- 提供されたハンドラを信頼できない場合、ユーザーは自分でハンドラを実装できます。

<!--
# Package Management

Every non-trivial Flix project should have a `flix.toml` manifest. The manifest
contains information about the project and its dependencies.

A minimal manifest is of the form:

```toml
[package]
name        = "hello-library"
description = "A simple library"
version     = "0.1.0"
flix        = "0.35.0"
license     = "Apache-2.0"
authors     = ["John Doe <john@example.com>"]
```

> **Note:** The `flix` field is not yet used, but it will be used in the future.

## Adding Flix Dependencies

We can add dependencies on other Flix packages to the manifest:

```toml
[dependencies]
"github:flix/museum"              = "1.4.0"
"github:magnus-madsen/helloworld" = "1.3.0"
```

> **Note:** Flix requires version numbers to follow SemVer.

## Adding Maven Dependencies

We can also add dependencies on Maven packages to the manifest:

```toml
[mvn-dependencies]
"org.junit.jupiter:junit-jupiter-api" = "5.9.2"
```

## Understanding Dependency Resolution

Flix dependency resolution works as follows:

1. Flix reads `flix.toml` and computes the transitive set of Flix package
   dependencies.
2. Flix downloads all of these Flix packages.
3. Flix inspects each package for its Maven dependencies and downloads these.

We illustrate with an example. Assume we have a Flix package with:

```toml
[dependencies]
"github:flix/museum"              = "1.4.0"
```

Running Flix produces:

```
Found `flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v1.4.0)... OK.
  Downloading `flix/museum-entrance.toml` (v1.2.0)... OK.
  Downloading `flix/museum-giftshop.toml` (v1.1.0)... OK.
  Downloading `flix/museum-restaurant.toml` (v1.1.0)... OK.
  Downloading `flix/museum-clerk.toml` (v1.1.0)... OK.
  Cached `flix/museum-clerk.toml` (v1.1.0).
Downloading Flix dependencies...
  Downloading `flix/museum.fpkg` (v1.4.0)... OK.
  Downloading `flix/museum-entrance.fpkg` (v1.2.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v1.1.0)... OK.
  Downloading `flix/museum-restaurant.fpkg` (v1.1.0)... OK.
  Downloading `flix/museum-clerk.fpkg` (v1.1.0)... OK.
  Cached `flix/museum-clerk.fpkg` (v1.1.0).
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (3.12.0).
  Running Maven dependency resolver.
Dependency resolution completed.
```

This happens because `flix/museum` has the following dependency tree:

- `flix/museum` depends on:
    - `flix/museum-entrance` which depends on:
        - `flix/museum-clerk`
    - `flix/museum-giftshop` which depends on:
        - `flix/museum-clerk`
    - `flix/museum-restaurant` which depends on
        - `org.apache.commons:commons-lang3`

## Security
To reduce the risk of supply-chain attacks, every dependency has a **security
context** — even if you don't set one explicitly. Security contexts control
which language features a dependency may use. Broader security contexts enable
more features but also increase the risk of supply-chain attacks.

The security contexts are defined as follows:

| Security Context | Java Interop | Unchecked Casts | The `IO` Effect |
|------------------|--------------|-----------------|-----------------|
| `paranoid`       | Forbidden    | Forbidden       | Forbidden       |
| `plain` (default)| Forbidden    | Forbidden       | Allowed         |
| `unrestricted`   | Allowed      | Allowed         | Allowed         |

You can set the security context of each dependency in the manifest like so:
```toml
[dependencies]
"github:flix/museum"              = { version = "1.4.0", security = "plain" }
"github:magnus-madsen/helloworld" = { version = "1.3.0", security = "unrestricted" }
```

Security contexts are transitive: a dependency's security context also applies
to its transitive dependencies, unless a dependency explicitly declares a lesser
security context. If multiple dependencies require the same library, the library
inherits the most restrictive security context requested.

The recommended approach is to **not** specify a security context, thus
defaulting to `plain`. It provides the best balance between flexibility and
safety. You should avoid `unrestricted` when possible, as it permits
(transitive) dependencies to do *anything*. Even building or compiling code that
includes `unrestricted` dependencies can by itself expose you to a supply-chain
attack.

If you are the author of a Flix library that requires effects, the best
practice is to introduce your own custom effects instead of using the `IO`
effect directly, and to split the library into two packages:

| Package                  | Description                           | Security Context |
|--------------------------|---------------------------------------|------------------|
| `webserver-lib`          | Core functionality using effects      | `plain`          |
| `webserver-lib-handlers` | Handlers that perform Java interop/IO | `unrestricted`   |

This approach provides several benefits:
- Most functionality remains in the trusted `plain` security context.
- Unsafe code is isolated in `webserver-lib-handlers` for easier review.
- Users can implement their own handlers if they don't trust the provided ones.
-->
