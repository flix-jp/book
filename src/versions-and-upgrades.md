# バージョンとアップグレード

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/versions-and-upgrades.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/versions-and-upgrades.md)ください。

マニフェストに書かれたバージョンは、ビルドに使用できる*最低*バージョンであり、実際に使われる正確なバージョンではありません。Flix は各パッケージについてビルドすべきバージョンを決定し、ダウンロードした内容を記録し、より新しいものがあれば教えてくれます。

## Flix の依存関係解決の仕組み

Flix はプロジェクトの依存関係を 4 つのステップで解決します：

1. Flix は `flix.toml` を読み込み、依存関係を通じて到達可能なすべての Flix パッケージのマニフェストを、要求されているすべてのバージョンについてダウンロードします。
2. Flix は各パッケージについて、ビルドすべき単一のバージョンを決定します。
3. Flix は、選択された各パッケージのパッケージファイルをダウンロードします。
4. Flix は各パッケージの Maven の依存関係を調べ、それらをダウンロードします。

これらのステップが実際に働く様子は、`flix/museum` に依存するプロジェクトで確認できます：

```toml
[dependencies]
"github:flix/museum" = { version = "4.0.0", mount = "museum", security = "unrestricted" }
```

このパッケージは、自身の依存関係の 1 つを通じて Java に手を伸ばすため、[依存関係の信頼](./trusting-dependencies.md) で説明するように `unrestricted` と宣言しなければなりません。次のような依存関係ツリーを持っています：

- `flix/museum` は以下に依存します：
    - `flix/museum-clerk`（v2.1.3）
    - `flix/museum-entrance` は以下に依存します：
        - `flix/museum-clerk`（v2.1.2）
    - `flix/museum-giftshop` は以下に依存します：
        - `flix/museum-clerk`（v2.1.2）
    - `flix/museum-restaurant` は以下に依存します：
        - `org.apache.commons:commons-lang3`

そして、1 つの依存関係から 5 つのパッケージが得られます：

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v4.0.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.3)... OK.
  Downloading `flix/museum-entrance.toml` (v2.0.2)... OK.
  Downloading `flix/museum-giftshop.toml` (v2.0.2)... OK.
  Downloading `flix/museum-restaurant.toml` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.2)... OK.
  Cached `flix/museum-clerk.toml` (v2.1.2).
  Raised `flix/museum-clerk` (v2.1.2 -> v2.1.3), required by `flix/museum` (v4.0.0).
Downloading Flix dependencies...
  Downloading `flix/museum-restaurant.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-entrance.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum.fpkg` (v4.0.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.3)... OK.
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (v3.12.0).
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

`flix/museum-clerk` のマニフェストは、要求されている両方のバージョンでダウンロードされますが、ビルドされるのはそのうちの 1 つだけです。

## Flix がバージョンを選ぶ仕組み

Flix は、依存関係グラフの中で何かが要求している最大のバージョンでパッケージをビルドします。これは、すべての依存先を満たす最小のバージョンでもあります。

言い換えると、依存関係グラフの 2 つの部分が同じパッケージの異なるバージョンを要求している場合、Flix はそのうちの新しい方を採用します。これがうまくいくのは、マニフェスト内のバージョンが下限だからです。より古いバージョンを要求する依存先は、メジャーバージョンさえ同じであれば、より新しいバージョンでビルドされても問題ありません。そのため、要求したものより古いバージョンのパッケージが使われることは決してなく、むしろ新しいバージョンが使われることがあります。

上の例では、`flix/museum` は `flix/museum-clerk` をバージョン `2.1.3` で要求していますが、`flix/museum-entrance` と `flix/museum-giftshop` はバージョン `2.1.2` で要求しています。Flix はこれを `2.1.3` でビルドし、次のように報告します：

```
  Raised `flix/museum-clerk` (v2.1.2 -> v2.1.3), required by `flix/museum` (v4.0.0).
```

**パッケージは 1 つのバージョンでのみビルドされます。** プロジェクトが同じパッケージの異なるメジャーバージョンに推移的に依存している場合、どの単一バージョンもすべての依存先を満たすことができないため、Flix はエラーを報告します。例えば、`flix/museum-giftshop` が `flix/museum-clerk` をバージョン `1.1.0` で要求し、他の依存先がそれぞれ `2.1.2` と `2.1.3` で要求している場合：

```
Found incompatible versions of the same package in the dependency graph:
  The package 'github:flix/museum-clerk' is required at versions that do not share a major version:

    1.1.0 required by 'flix/museum-giftshop'
    2.1.2 required by 'flix/museum-entrance'
    2.1.3 required by 'flix/museum'

  A package is built at one version, which must satisfy every dependent: it must
  be at or above the version the dependent requires, and have the same major version.
  No version satisfies these, so one of the dependents must move across a major version.
```

この場合、どちらかの依存先がメジャーバージョンをまたいで移行する必要がありますが、それができるのはその作者だけです。

## Flix のバージョン

各パッケージは、その `flix` フィールドの中で、そのパッケージをビルドできる最も古い Flix のバージョンを宣言します。Flix は依存関係グラフ内のすべてのパッケージのこのフィールドを確認し、現在実行中のバージョンよりも新しい Flix のバージョンを要求するパッケージのビルドを拒否します：

```
The package 'github:flix/museum' 4.0.0 requires Flix version 0.76.2, but the current version is 0.76.1.
Please upgrade to Flix 0.76.2 or newer.
```

## ロックファイル <a name="the-lock-file"></a>

Flix はプロジェクトの依存関係を解決する際、`flix.toml` の隣に `packages.lock` ファイルを書き出します。上の例では、次のように始まります：

```toml
[lock]
version = 1

[packages."github:flix/museum"."4.0.0"]
toml    = "sha256:5be4c6f17203a0c62294612f2877f52c181e2029153d88c8c65789f64bdf1c5f"
fpkg    = "sha256:c6d22cf0cdb1a45208b175636830f8364308db9f3f7259a2e4960dfb536d1e3c"

[packages."github:flix/museum-clerk"."2.1.2"]
toml    = "sha256:8dc56b5b992d348ce09ace7b4faad00eb81b8803a02ffa2f18051bda93d800de"

[packages."github:flix/museum-clerk"."2.1.3"]
toml    = "sha256:6f34418fffc6c8d5614f32c1f904dbed385bc00e9ac7692e9cbe277fed54f66d"
fpkg    = "sha256:f7817d79093be36bcf9bf1ed66fa920d1e6eab6b1ac3759c1cc6581568b6ff8b"
```

ロックファイルには、依存関係解決が読み込んだすべてのファイルのダイジェストが記録されます。依存関係グラフが要求するすべてのバージョンにおける各パッケージの `flix.toml`、そしてビルドされる各パッケージのパッケージファイルです。ここでは、`flix/museum-clerk` のマニフェストは両方のバージョンで記録されますが、そのパッケージファイルはビルドされるバージョンである `2.1.3` のものだけが記録されます。次回のビルド時、Flix は各ファイルをインストールする際にロックファイルと照合し、以前と同じ内容でなくなった依存関係のコンパイルを拒否します。

> **ヒント:** `packages.lock` ファイルはバージョン管理にコミットすべきです。

> **注意:** ロックファイルが記録するのは Flix パッケージのみです。Maven ライブラリや URL からダウンロードした JAR ファイルはこれには含まれません。

## 古くなったパッケージの確認 <a name="finding-outdated-packages"></a>

`outdated` コマンドを使うと、使用している Flix パッケージに新しいリリースがあるかどうかを確認できます。例えば次のような依存関係を持つプロジェクトでは：

```toml
[dependencies]
"github:flix/museum"          = { version = "3.0.1", mount = "museum", security = "unrestricted" }
"github:flix/museum-giftshop" = { version = "2.0.1", mount = "giftshop" }
```

`outdated` コマンドは次のように報告します：

```
package                 declared    built    major    minor    patch
flix/museum             3.0.1       3.0.1    4.0.0             3.0.2
flix/museum-giftshop    2.0.1       2.0.2
```

パッケージ `flix/museum` には 2 つの更新が利用可能です。メジャーバージョンを変えずに `3.0.1` から `3.0.2` にアップグレードするか、メジャーバージョンをまたいで `4.0.0` にアップグレードするかです。

この表には 2 つのバージョン列があります：

- `declared` は `flix.toml` に書かれているバージョンです。
- `built` は、実際にそのパッケージがビルドされているバージョンで、これはより大きい場合があります。宣言されたバージョンは使用できる最低バージョンであり、別の依存先がより新しいバージョンを要求しているかもしれないからです。

パッケージは、実際にビルドされているバージョンで比較されます。ここでは、`flix/museum` は `flix/museum-giftshop` をその最新リリースである `2.0.2` で要求しているため、それ以上新しいバージョンに移行する必要はありません。それでも一覧に表示されるのは、実際にビルドされているバージョンよりも古いバージョンを宣言しているためです。代わりに `2.0.2` を宣言することもできます。何も古くなっていない場合、Flix は次のように報告します：

```
All dependencies are up to date
```

> **ヒント:** `outdated` コマンドは、依存関係を一覧に表示する場合は終了ステータス `1` を返し、そうでない場合は `0` を返します。そのため、CI ビルドを失敗させるために利用できます。

> **注意:** 一覧に表示されるのは Flix パッケージのみです。Maven の依存関係はチェックされません。

## パッケージのアップグレード <a name="upgrading-a-package"></a>

パッケージをアップグレードするには、`flix.toml` 内のその項目の `version` を変更するか、`upgrade` コマンドを使います：

```
upgrade flix/museum
```

`upgrade` コマンドは、現在宣言しているものと同じメジャーバージョンを持つ最新リリースを宣言し、より新しいメジャーバージョンがある場合はそれを教えてくれます：

```
Upgraded 'flix/museum' v3.0.1 -> v3.0.2.

A newer major release is available, ask for it by name:
  flix upgrade flix/museum@4.0.0
```

新しいメジャーバージョンはコードを壊す可能性があるため、`upgrade` はそのバージョンを名指ししたときにのみそのバージョンへ移行します：

```
upgrade flix/museum@4.0.0
```

名指ししたバージョンはそのまま採用されるため、古いリリースを名指しして戻ることもできます。`upgrade` コマンドが変更するのはバージョンのみで、mount と security context は宣言したままになります。

複数のパッケージを名指しすることも、まったく名指ししないこともできます。何も名指ししない場合、`upgrade` はマニフェストが宣言しているすべての Flix パッケージを、それぞれのメジャーバージョン内でアップグレードし、変更したものだけを報告します。[古くなったパッケージの確認](#finding-outdated-packages) のプロジェクトでは、次のように報告されます：

```
Upgraded 'flix/museum' v3.0.1 -> v3.0.2.
Upgraded 'flix/museum-giftshop' v2.0.1 -> v2.0.2.

A newer major release is available, ask for it by name:
  flix upgrade flix/museum@4.0.0
```

1 回のコマンドでアップグレードされるパッケージは、まとめて変更されるか、まったく変更されないかのどちらかです。あるパッケージの新しいメジャーバージョンが別のパッケージの新しいメジャーバージョンを要求しているために、2 つのパッケージを同時に新しいメジャーバージョンへ移行させなければならない場合は、`flix.toml` の 1 回の編集で両方を変更するか、1 回の `upgrade` コマンドで両方を名指しします。

<!--
# Versions and Upgrades

A version in a manifest is the *least* version we can build with, not the exact
version we get. Flix works out which version of every package to build, records
what it downloaded, and tells us when there is something newer.

## How Flix Resolves Dependencies

Flix resolves the dependencies of a project in four steps:

1. Flix reads `flix.toml` and downloads the manifest of every Flix package that can
   be reached through the dependencies, at every version they are required at.
2. Flix works out which single version of each package to build.
3. Flix downloads the package file of each selected package.
4. Flix inspects each package for its Maven dependencies and downloads these.

We can see these steps at work in a project that depends on `flix/museum`:

```toml
[dependencies]
"github:flix/museum" = { version = "4.0.0", mount = "museum", security = "unrestricted" }
```

The package reaches Java through one of its own dependencies, so it must be declared
`unrestricted`, as described in [Trusting Dependencies](./trusting-dependencies.md).
It has this dependency tree:

- `flix/museum` depends on:
    - `flix/museum-clerk` (v2.1.3)
    - `flix/museum-entrance` which depends on:
        - `flix/museum-clerk` (v2.1.2)
    - `flix/museum-giftshop` which depends on:
        - `flix/museum-clerk` (v2.1.2)
    - `flix/museum-restaurant` which depends on
        - `org.apache.commons:commons-lang3`

and so a single dependency gives us five packages:

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v4.0.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.3)... OK.
  Downloading `flix/museum-entrance.toml` (v2.0.2)... OK.
  Downloading `flix/museum-giftshop.toml` (v2.0.2)... OK.
  Downloading `flix/museum-restaurant.toml` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.2)... OK.
  Cached `flix/museum-clerk.toml` (v2.1.2).
  Raised `flix/museum-clerk` (v2.1.2 -> v2.1.3), required by `flix/museum` (v4.0.0).
Downloading Flix dependencies...
  Downloading `flix/museum-restaurant.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-entrance.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum.fpkg` (v4.0.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.3)... OK.
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (v3.12.0).
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

The manifest of `flix/museum-clerk` is downloaded at both of the versions it is
required at, but only one of them is built.

## How Flix Selects a Version

Flix builds a package at the greatest version that anything in the dependency graph
requires, which is the least version that satisfies every dependent.

In other words, when two parts of the dependency graph ask for different versions of
the same package, Flix takes the newer of them. This works because a version in a
manifest is a lower bound: a dependent that asks for an older version is happy to be
built against a newer one, as long as the major version is the same. We therefore
never get an older version of a package than we asked for, and we may well get a
newer one.

In the example above, `flix/museum` requires `flix/museum-clerk` at version
`2.1.3`, whereas `flix/museum-entrance` and `flix/museum-giftshop` require it at
`2.1.2`. Flix builds it at `2.1.3` and reports that it did so:

```
  Raised `flix/museum-clerk` (v2.1.2 -> v2.1.3), required by `flix/museum` (v4.0.0).
```

**A package is built at one version only.** If a project transitively depends on two
different major versions of the same package, then no single version satisfies every
dependent, and Flix reports an error. For example, if `flix/museum-giftshop` required
`flix/museum-clerk` at `1.1.0`, while the others require it at `2.1.2` and `2.1.3`:

```
Found incompatible versions of the same package in the dependency graph:
  The package 'github:flix/museum-clerk' is required at versions that do not share a major version:

    1.1.0 required by 'flix/museum-giftshop'
    2.1.2 required by 'flix/museum-entrance'
    2.1.3 required by 'flix/museum'

  A package is built at one version, which must satisfy every dependent: it must
  be at or above the version the dependent requires, and have the same major version.
  No version satisfies these, so one of the dependents must move across a major version.
```

One of the dependents then has to move across a major version, which is something
only its author can do.

## The Version of Flix

Every package declares, in its `flix` field, the oldest version of Flix that can
build it. Flix checks the field of every package in the dependency graph, and
refuses to build a package that requires a newer version of Flix than the one we
are running:

```
The package 'github:flix/museum' 4.0.0 requires Flix version 0.76.2, but the current version is 0.76.1.
Please upgrade to Flix 0.76.2 or newer.
```

## The Lock File

When Flix resolves the dependencies of a project, it writes a `packages.lock` file
next to `flix.toml`. For the example above, it begins:

```toml
[lock]
version = 1

[packages."github:flix/museum"."4.0.0"]
toml    = "sha256:5be4c6f17203a0c62294612f2877f52c181e2029153d88c8c65789f64bdf1c5f"
fpkg    = "sha256:c6d22cf0cdb1a45208b175636830f8364308db9f3f7259a2e4960dfb536d1e3c"

[packages."github:flix/museum-clerk"."2.1.2"]
toml    = "sha256:8dc56b5b992d348ce09ace7b4faad00eb81b8803a02ffa2f18051bda93d800de"

[packages."github:flix/museum-clerk"."2.1.3"]
toml    = "sha256:6f34418fffc6c8d5614f32c1f904dbed385bc00e9ac7692e9cbe277fed54f66d"
fpkg    = "sha256:f7817d79093be36bcf9bf1ed66fa920d1e6eab6b1ac3759c1cc6581568b6ff8b"
```

The lock file records the digest of every file that the resolution read: the
`flix.toml` of every package at every version that the dependency graph requires,
and the package file of every package that is built. Here, the manifest of
`flix/museum-clerk` is recorded at both versions, but its package file only at
`2.1.3`, the version that is built. On a later build, Flix verifies each file against
the lock file as it is installed, and refuses to compile a dependency that is no
longer the same.

> **Tip:** The `packages.lock` file should be committed to version control.

> **Note:** The lock file records Flix packages only. Maven libraries and JAR-files
> downloaded from a URL are not covered by it.

## Finding Outdated Packages

We can check whether any of our Flix packages have newer releases with the
`outdated` command. For a project that depends on:

```toml
[dependencies]
"github:flix/museum"          = { version = "3.0.1", mount = "museum", security = "unrestricted" }
"github:flix/museum-giftshop" = { version = "2.0.1", mount = "giftshop" }
```

the `outdated` command reports:

```
package                 declared    built    major    minor    patch
flix/museum             3.0.1       3.0.1    4.0.0             3.0.2
flix/museum-giftshop    2.0.1       2.0.2
```

The package `flix/museum` has two updates available: we can upgrade from `3.0.1` to
`3.0.2` without changing major version, or to `4.0.0` across one.

The table has two version columns:

- `declared` is the version written in `flix.toml`.
- `built` is the version the package is actually built at, which can be greater: a
  declared version is the least version we can build with, and another dependent
  may require a greater one.

A package is compared by the version it is built at. Here, `flix/museum` requires
`flix/museum-giftshop` at `2.0.2`, its newest release, so there is nothing newer to
move to. It is listed all the same, because we declare an older version than the one
it is built at: we can declare `2.0.2` instead. When nothing is listed, Flix reports:

```
All dependencies are up to date
```

> **Tip:** The `outdated` command exits with status `1` when it lists a dependency,
> and with `0` otherwise, so it can fail a CI build.

> **Note:** Only Flix packages are listed. Maven dependencies are not checked.

## Upgrading a Package

We can upgrade a package by changing the `version` of its entry in `flix.toml`, or
with the `upgrade` command:

```
upgrade flix/museum
```

The `upgrade` command declares the newest release that has the same major version as
the one we declare, and tells us if there is a newer major version:

```
Upgraded 'flix/museum' v3.0.1 -> v3.0.2.

A newer major release is available, ask for it by name:
  flix upgrade flix/museum@4.0.0
```

A new major version may break our code, so `upgrade` only moves to one when we name
it:

```
upgrade flix/museum@4.0.0
```

A version that we name is taken as it is, so we can also name an older release to
move back to it. The `upgrade` command changes only the version: the mount and the
security context stay as we declared them.

We can name several packages, or none at all. With none, `upgrade` upgrades every
Flix package that the manifest declares, each within its major version, and reports
only the ones it changed. For the project in
[Finding Outdated Packages](#finding-outdated-packages), it reports:

```
Upgraded 'flix/museum' v3.0.1 -> v3.0.2.
Upgraded 'flix/museum-giftshop' v2.0.1 -> v2.0.2.

A newer major release is available, ask for it by name:
  flix upgrade flix/museum@4.0.0
```

The packages that one command upgrades are changed together or not at all. When two
packages must move to a new major version at the same time, because the new major
version of one requires the new major version of the other, we change both in one
edit of `flix.toml`, or name both in one `upgrade` command.
-->
