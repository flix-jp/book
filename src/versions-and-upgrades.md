# バージョンとアップグレード

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/versions-and-upgrades.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/versions-and-upgrades.md)ください。

マニフェストに書かれたバージョンは、ビルドに使用できる*最低*バージョンであり、実際に使われる正確なバージョンではありません。Flix は各パッケージについてビルドすべきバージョンを決定し、ダウンロードした内容を記録し、より新しいものがあれば教えてくれます。

## Flix の依存関係解決の仕組み

Flix はプロジェクトの依存関係を 4 つのステップで解決します：

1. Flix は `flix.toml` を読み込み、依存関係を通じて到達可能なすべての Flix パッケージのマニフェストを、要求されているすべてのバージョンについてダウンロードします。
2. Flix は各パッケージについて、ビルドすべき単一のバージョンを決定します。
3. Flix は、選択された各パッケージのパッケージファイルをダウンロードします。
4. Flix は各パッケージの Maven の依存関係を調べ、それらをダウンロードします。

[依存関係の使用](./using-dependencies.md) の例では、1 つの依存関係から 5 つのパッケージがダウンロードされました。これは `flix/museum` が次のような依存関係ツリーを持っているためです：

- `flix/museum` は以下に依存します：
    - `flix/museum-clerk`（v2.1.0）
    - `flix/museum-entrance` は以下に依存します：
        - `flix/museum-clerk`（v2.0.0）
    - `flix/museum-giftshop` は以下に依存します：
        - `flix/museum-clerk`（v2.0.0）
    - `flix/museum-restaurant` は以下に依存します：
        - `org.apache.commons:commons-lang3`

`flix/museum-clerk` のマニフェストは、要求されている両方のバージョンでダウンロードされますが、ビルドされるのはそのうちの 1 つだけです。

## Flix がバージョンを選ぶ仕組み

Flix は、依存関係グラフの中で何かが要求している最大のバージョンでパッケージをビルドします。これは、すべての依存先を満たす最小のバージョンでもあります。

言い換えると、依存関係グラフの 2 つの部分が同じパッケージの異なるバージョンを要求している場合、Flix はそのうちの新しい方を採用します。これがうまくいくのは、マニフェスト内のバージョンが下限だからです。より古いバージョンを要求する依存先は、メジャーバージョンさえ同じであれば、より新しいバージョンでビルドされても問題ありません。そのため、要求したものより古いバージョンのパッケージが使われることは決してなく、むしろ新しいバージョンが使われることがあります。

上の例では、`flix/museum` は `flix/museum-clerk` をバージョン `2.1.0` で要求していますが、`flix/museum-entrance` と `flix/museum-giftshop` はバージョン `2.0.0` で要求しています。Flix はこれを `2.1.0` でビルドし、次のように報告します：

```
  Raised `flix/museum-clerk` (v2.0.0 -> v2.1.0), required by `flix/museum` (v2.1.0).
```

**パッケージは 1 つのバージョンでのみビルドされます。** プロジェクトが同じパッケージの異なるメジャーバージョンに推移的に依存している場合、どの単一バージョンもすべての依存先を満たすことができないため、Flix はエラーを報告します。例えば、ある依存先が `flix/museum-clerk` をバージョン `1.1.0` で要求し、別の依存先が `2.1.0` で要求している場合：

```
Found incompatible versions of the same package in the dependency graph:
  The package 'github:flix/museum-clerk' is required at versions that do not share a major version:

    1.1.0 required by 'flix/museum-giftshop'
    2.1.0 required by 'flix/museum'

  A package is built at one version, which must satisfy every dependent: it must
  be at or above the version the dependent requires, and have the same major version.
  No version satisfies these, so one of the dependents must move across a major version.
```

この場合、どちらかの依存先がメジャーバージョンをまたいで移行する必要がありますが、それができるのはその作者だけです。

## Flix のバージョン

各パッケージは、その `flix` フィールドの中で、そのパッケージをビルドできる最も古い Flix のバージョンを宣言します。Flix は依存関係グラフ内のすべてのパッケージのこのフィールドを確認し、現在実行中のバージョンよりも新しい Flix のバージョンを要求するパッケージのビルドを拒否します：

```
The package 'github:flix/museum' 2.1.0 requires Flix version 0.76.2, but the current version is 0.76.1.
Please upgrade to Flix 0.76.2 or newer.
```

## ロックファイル <a name="the-lock-file"></a>

Flix はプロジェクトの依存関係を解決する際、`flix.toml` の隣に `packages.lock` ファイルを書き出します：

```toml
[lock]
version = 1

[packages."github:flix/museum-clerk"."1.0.0"]
toml    = "sha256:af31faa57878e01363946f2b82df193bd5cc939ce0c5235c6e125ab727bbe35d"
fpkg    = "sha256:ed9210a2088e9a31d4406d4e1306737c68fb3080f7d5a1c11e7316b780bb5704"

[packages."github:flix/museum-giftshop"."1.0.0"]
toml    = "sha256:d1e5db83f30efa1ef8b16911a6d7289d87cba33bb8ec4f70d4c4f0af763482e8"
fpkg    = "sha256:49349a6c68d3b4c0636e0dd0215e4d3099edc6ab06565960fd525575e5790bf2"
```

ロックファイルには、依存関係解決が読み込んだすべてのファイルのダイジェストが記録されます。依存関係グラフが要求するすべてのバージョンにおける各パッケージの `flix.toml`、そしてビルドされる各パッケージのパッケージファイルです。次回のビルド時、Flix は各ファイルをインストールする際にロックファイルと照合し、以前と同じ内容でなくなった依存関係のコンパイルを拒否します。

> **ヒント:** `packages.lock` ファイルはバージョン管理にコミットすべきです。

> **注意:** ロックファイルが記録するのは Flix パッケージのみです。Maven ライブラリや URL からダウンロードした JAR ファイルはこれには含まれません。

## 古くなったパッケージの確認

`outdated` コマンドを使うと、使用している Flix パッケージに新しいリリースがあるかどうかを確認できます。例えば次のような依存関係を持つプロジェクトでは：

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "1.0.0", mount = "giftshop" }
```

`outdated` コマンドは次のように報告します：

```
package                 declared    built    major    minor    patch
flix/museum-giftshop    1.0.0       1.0.0    2.0.0    1.1.0
```

パッケージ `flix/museum-giftshop` には 2 つの更新が利用可能です。メジャーバージョンを変えずに `1.0.0` から `1.1.0` にアップグレードするか、メジャーバージョンをまたいで `2.0.0` にアップグレードするかです。

この表には 2 つのバージョン列があります：

- `declared` は `flix.toml` に書かれているバージョンです。
- `built` は、実際にそのパッケージがビルドされているバージョンで、これはより大きい場合があります。宣言されたバージョンは使用できる最低バージョンであり、別の依存先がより新しいバージョンを要求しているかもしれないからです。

パッケージは、実際にビルドされているバージョンで比較されます。したがって、宣言しているバージョンが古くても、実際にビルドされているバージョンが最新リリースであれば、そのパッケージは最新の状態であり、一覧には表示されません。何も古くなっていない場合、Flix は次のように報告します：

```
All dependencies are up to date
```

パッケージをアップグレードするには、`flix.toml` を自分で変更します。Flix はマニフェストを自動的に編集してくれません。

> **注意:** 一覧に表示されるのは Flix パッケージのみです。Maven の依存関係はチェックされません。

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

The example in [Using Dependencies](./using-dependencies.md) downloads five
packages from a single dependency, because `flix/museum` has this dependency tree:

- `flix/museum` depends on:
    - `flix/museum-clerk` (v2.1.0)
    - `flix/museum-entrance` which depends on:
        - `flix/museum-clerk` (v2.0.0)
    - `flix/museum-giftshop` which depends on:
        - `flix/museum-clerk` (v2.0.0)
    - `flix/museum-restaurant` which depends on
        - `org.apache.commons:commons-lang3`

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
`2.1.0`, whereas `flix/museum-entrance` and `flix/museum-giftshop` require it at
`2.0.0`. Flix builds it at `2.1.0` and reports that it did so:

```
  Raised `flix/museum-clerk` (v2.0.0 -> v2.1.0), required by `flix/museum` (v2.1.0).
```

**A package is built at one version only.** If a project transitively depends on two
different major versions of the same package, then no single version satisfies every
dependent, and Flix reports an error. For example, if one dependent required
`flix/museum-clerk` at `1.1.0` while another required it at `2.1.0`:

```
Found incompatible versions of the same package in the dependency graph:
  The package 'github:flix/museum-clerk' is required at versions that do not share a major version:

    1.1.0 required by 'flix/museum-giftshop'
    2.1.0 required by 'flix/museum'

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
The package 'github:flix/museum' 2.1.0 requires Flix version 0.76.2, but the current version is 0.76.1.
Please upgrade to Flix 0.76.2 or newer.
```

## The Lock File

When Flix resolves the dependencies of a project, it writes a `packages.lock` file
next to `flix.toml`:

```toml
[lock]
version = 1

[packages."github:flix/museum-clerk"."1.0.0"]
toml    = "sha256:af31faa57878e01363946f2b82df193bd5cc939ce0c5235c6e125ab727bbe35d"
fpkg    = "sha256:ed9210a2088e9a31d4406d4e1306737c68fb3080f7d5a1c11e7316b780bb5704"

[packages."github:flix/museum-giftshop"."1.0.0"]
toml    = "sha256:d1e5db83f30efa1ef8b16911a6d7289d87cba33bb8ec4f70d4c4f0af763482e8"
fpkg    = "sha256:49349a6c68d3b4c0636e0dd0215e4d3099edc6ab06565960fd525575e5790bf2"
```

The lock file records the digest of every file that the resolution read: the
`flix.toml` of every package at every version that the dependency graph requires,
and the package file of every package that is built. On a later build, Flix verifies
each file against the lock file as it is installed, and refuses to compile a
dependency that is no longer the same.

> **Tip:** The `packages.lock` file should be committed to version control.

> **Note:** The lock file records Flix packages only. Maven libraries and JAR-files
> downloaded from a URL are not covered by it.

## Finding Outdated Packages

We can check whether any of our Flix packages have newer releases with the
`outdated` command. For a project that depends on:

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "1.0.0", mount = "giftshop" }
```

the `outdated` command reports:

```
package                 declared    built    major    minor    patch
flix/museum-giftshop    1.0.0       1.0.0    2.0.0    1.1.0
```

The package `flix/museum-giftshop` has two updates available: we can upgrade from
`1.0.0` to `1.1.0` without changing major version, or to `2.0.0` across one.

The table has two version columns:

- `declared` is the version written in `flix.toml`.
- `built` is the version the package is actually built at, which can be greater: a
  declared version is the least version we can build with, and another dependent
  may require a greater one.

A package is compared by the version it is built at. A dependency that is built at
its newest release is therefore up to date, and is not listed, even if the version
we declare is older. When nothing is outdated, Flix reports:

```
All dependencies are up to date
```

To upgrade a package, we change the version in `flix.toml` ourselves. Flix does not
edit the manifest for us.

> **Note:** Only Flix packages are listed. Maven dependencies are not checked.
-->
