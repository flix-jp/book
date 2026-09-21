# パッケージの公開

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/publishing-a-package.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/publishing-a-package.md)ください。

Flix パッケージは GitHub 上で公開されます。1 つのリリースは、`build-pkg` が `artifact` ディレクトリに残す 2 つのファイル、すなわちパッケージ自体である `package.fpkg` と、それを説明するマニフェストである `flix.toml` を保持します。そのパッケージに依存する側は、この両方をダウンロードします。

## マニフェストの準備

GitHub 上で公開されるパッケージは、公開元のリポジトリを宣言しなければなりません：

```toml
[package]
version    = "2.1.0"
flix       = "0.76.2"
repository = "github:flix/museum"
```

Flix はパッケージのビルドとアップロードを代行してくれますが、マニフェストの内容が意図どおりかどうかまでは判断できません。開発者である私たちは、リリースする前に次の 3 点を確認しなければなりません：

1. `version` フィールドが、リリースしようとしているバージョンであり、以前に同じバージョンをリリースしていないこと。バージョンは一度公開されると変更できません。
2. `repository` フィールドが、公開先のリポジトリを表していること。これは、依存する側がそのパッケージに依存するために書く名前になります。
3. `check` と `test` の両方が通ること。`release` コマンドはパッケージをビルドするため、プロジェクトがコンパイルできる必要がありますが、テストは代わりに実行してくれません。

## `release` コマンドによる公開

`release` コマンドを使うと、Flix がパッケージ化とリリースの公開を代行してくれます。このコマンドには、対象リポジトリの `Contents` に対する読み取り・書き込みアクセス権を持つ GitHub トークンが必要です。これは GitHub の `Settings > Developer settings > Personal access tokens` から作成します。

その後、`release --github-token <TOKEN>` を実行します：

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
Release github:user/repo v1.2.3? [y/N]: y
Building project...
Publishing a new release...

Successfully released v1.2.3
https://github.com/user/repo/releases/tag/v1.2.3
```

Flix はパッケージをビルドし、`v1.2.3` というタグのリリースを作成し、そのリリースにパッケージの 2 つのファイル `package.fpkg` と `flix.toml` をアップロードします。

> **ヒント:** Flix は、次の 3 か所を順番に GitHub トークンとして探します。`--github-token` オプション、プロジェクトディレクトリ内の `.GITHUB_TOKEN` ファイル、そして環境変数 `GITHUB_TOKEN` です。

> **ヒント:** `--yes` オプションを使うと、確認プロンプトに自動で答えられます。スクリプトや CI からリリースする場合に便利です。

> **警告:** トークンは必ず安全に保管してください。生成される `.gitignore` は `.GITHUB_TOKEN` を除外しているため、誤ってコミットしてしまうことはありません。

> **注意:** 空の GitHub リポジトリに対してリリースを公開することはできません。

> **ヒント:** GitHub 上で公開されているパッケージの例としては、[Museum Project](https://github.com/flix/museum) を参照してください。

## パッケージのバージョン付け

マニフェスト内の 2 つのバージョン番号は、どちらも [SemVer](https://semver.org/) に従いますが、それぞれ異なる意味を持ちます。

`version` フィールドはパッケージ自体のバージョンです。どの部分をインクリメントするかによって、依存する側にどのような変更を期待させるかが決まります：

- **パッチ**リリース（`1.2.3` から `1.2.4`）は、何かを修正するもので API には手を加えません。
- **マイナー**リリース（`1.2.3` から `1.3.0`）は、既存の API を壊さずに追加を行うものです。
- **メジャー**リリース（`1.2.3` から `2.0.0`）は、依存する側が頼っているかもしれない何かを変更または削除するものです。

この区別は単なる慣習にとどまりません。依存関係解決の基準そのものです。Flix はパッケージを 1 つのバージョンでビルドし、そのバージョンはすべての依存する側が要求するバージョン以上であり、かつメジャーバージョンが一致していなければなりません。したがって、パッチリリースやマイナーリリースは依存する側が何も考えずに取り込めるものですが、メジャーリリースは、それぞれが意図的に踏み出す必要があるステップです。詳しくは [バージョンとアップグレード](./versions-and-upgrades.md) を参照してください。これはまた、`outdated` の 3 つの列が区別しているものでもあります。

`flix` フィールドは、そのパッケージをビルドできる最も古い Flix コンパイラのバージョンです。Flix はこれを、現在実行中のコンパイラと比較し、より新しいバージョンを要求するパッケージのビルドを拒否します。そのため、このフィールドを引き上げると、まだアップグレードしていないすべての依存先を締め出すことになります。パッケージがより新しいコンパイラが提供する何かを必要とする場合にこのフィールドを引き上げますが、できればパッチリリースではなく、マイナーリリースやメジャーリリースの際に行うべきです。

> **注意:** Flix 自体はメジャーバージョン `0` です。そのため、Flix 自身のマイナーリリースが言語を変更することがあります。`flix` フィールドはあくまで下限であり、Flix は、より新しいコンパイラであれば、より古いバージョンを要求するパッケージをビルドできると仮定します。

## パッケージが約束すること

マニフェストの 2 つのフィールドは、そのパッケージに依存する側への約束です：

- `version` フィールドは、パッケージが公開されているタグと一致していなければなりません。マニフェストが別のバージョンを宣言している `v1.2.3` のリリースは、解決される際に拒否され、その作者だけがこれを修正できます。
- `flix` フィールドは、そのパッケージをビルドできる最も古い Flix のバージョンでなければなりません。必要以上に新しいバージョンを宣言しているパッケージは、理由もなく依存先を締め出してしまいます。逆に必要より古いバージョンを宣言していると、依存先の環境ではコンパイルできません。

公開されたパッケージは、それに依存する側によってソースコードからコンパイルされます。そのため、`pub` 宣言のみが到達可能です。依存する側が使用することを意図しているすべてのモジュールは、`pub` と宣言しなければなりません。

<!--
# Publishing a Package

Flix packages are published on GitHub. A release holds two files, which are the two
files that `build-pkg` leaves in the `artifact` directory: `package.fpkg`, the package
itself, and `flix.toml`, the manifest that describes it. Whoever depends on the package
downloads both.

## Preparing the Manifest

A package that is published on GitHub must declare the repository it is published
from:

```toml
[package]
version    = "2.1.0"
flix       = "0.76.2"
repository = "github:flix/museum"
```

Flix builds and uploads the package for us, but it cannot tell whether the manifest
says what we meant. We — as the developer — must check three things before we release:

1. That the `version` field is the version we mean to release, and that we have not
   released it before. A version is published once and cannot be changed afterwards.
2. That the `repository` field names the repository we release to. It is the name our
   dependents write to depend on the package.
3. That `check` and `test` both pass. The `release` command builds the package, which
   means the project has to compile, but it does not run the tests for us.

## Publishing with the `release` Command

Flix can package and publish a release for us with the `release` command. The
command needs a GitHub token that has read and write access to `Contents` for the
repository, which we create under `Settings > Developer settings > Personal access
tokens` on GitHub.

We can then run `release --github-token <TOKEN>`:

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
Release github:user/repo v1.2.3? [y/N]: y
Building project...
Publishing a new release...

Successfully released v1.2.3
https://github.com/user/repo/releases/tag/v1.2.3
```

Flix builds the package, creates a release tagged `v1.2.3`, and uploads the two files
of the release to it: `package.fpkg` and `flix.toml`.

> **Tip:** Flix looks for the GitHub token in three places, in order: the
> `--github-token` option, a `.GITHUB_TOKEN` file in the project directory, and the
> `GITHUB_TOKEN` environment variable.

> **Tip:** The `--yes` option answers the confirmation prompt for us, which is what
> we want when we release from a script or from CI.

> **Warning:** Be sure to keep the token safe. The generated `.gitignore` excludes
> `.GITHUB_TOKEN` so that we do not commit it by accident.

> **Note:** We cannot publish a release for an empty GitHub repository.

> **Tip:** See the [Museum Project](https://github.com/flix/museum) for an example of
> a package that has been published on GitHub.

## Versioning a Package

Two version numbers in the manifest follow [SemVer](https://semver.org/), and they say
different things.

The `version` field is the version of the package itself. Which part we increment tells
our dependents what kind of change to expect:

- A **patch** release, `1.2.3` to `1.2.4`, fixes something and leaves the API alone.
- A **minor** release, `1.2.3` to `1.3.0`, adds to the API without breaking what was
  already there.
- A **major** release, `1.2.3` to `2.0.0`, changes or removes something that dependents
  may rely on.

Here the distinction is not only a convention: it is what dependency resolution runs on.
Flix builds a package at one version, which must be at or above what every dependent
requires and have the same major version. A patch or a minor release is therefore
something our dependents can take without thinking, whereas a major release is a step
each of them has to make deliberately, as described in
[Versions and Upgrades](./versions-and-upgrades.md). That is also what the three columns
of `outdated` separate.

The `flix` field is the oldest version of the Flix compiler that can build the package.
Flix compares it against the compiler that is running, and refuses to build a package
that wants a newer one, so raising this field shuts out every dependent that has not
upgraded yet. We raise it when the package needs something a newer compiler provides,
and preferably in a minor or a major release rather than in a patch.

> **Note:** Flix itself is at major version `0`, so its own minor releases may change
> the language. The `flix` field is a lower bound only: Flix assumes that a newer
> compiler can build a package that asks for an older one.

## What a Package Promises

Two fields of the manifest are promises to whoever depends on the package:

- The `version` field must match the tag the package is released under. A release of
  `v1.2.3` whose manifest declares another version is rejected when it is resolved,
  and only its author can fix it.
- The `flix` field must be the oldest version of Flix that can build the package. A
  package that declares a newer version than it needs excludes dependents for no
  reason; one that declares an older version than it needs does not compile for them.

A published package is compiled from source by whoever depends on it, so only its
`pub` declarations can be reached. Every module our dependents are meant to use must
be declared `pub`.
-->
