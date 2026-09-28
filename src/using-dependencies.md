# 依存関係の使用

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/using-dependencies.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/using-dependencies.md)ください。

Flix プロジェクトは 3 種類のものに依存できます。GitHub 上で公開されている Flix パッケージ、Maven 上で公開されている Java ライブラリ、そして URL からダウンロードする JAR ファイルです。この 3 つはいずれもマニフェストの中で宣言し、Flix がそれらをダウンロードします。

理想的には、Flix プロジェクトは Flix パッケージのみに依存するべきです。Flix パッケージはソースコードからコンパイルされ、できることを制限する security context(セキュリティコンテキスト) の中でビルドされ、私たちが書いているのと同じ言語で書かれています。Maven ライブラリ、そしてそれ以上に URL からの JAR ファイルは最後の手段です。まず Flix パッケージを探し、それが存在しない場合にのみ Java に頼るべきです。

## 依存関係を宣言する 2 つの方法

プロジェクトの依存関係とは、そのマニフェストが宣言しているものです。依存関係を変更する方法は 2 つあり、どちらも同じように有効です。`flix.toml` を自分で編集する方法と、代わりに編集してくれるコマンドを実行する方法です。

| タスク                     | `flix.toml` を編集する場合          | コマンドを実行する場合    |
|----------------------------|--------------------------------------|---------------------------|
| Flix パッケージを追加する  | `[dependencies]` にその項目を追加する | `install <owner>/<repo>` |
| バージョンを変更する       | その項目の `version` を変更する       | `upgrade <owner>/<repo>` |
| 削除する                   | その項目を削除する                   | `remove <owner>/<repo>`  |

各コマンドは、`install flix/museum-giftshop flix/museum-entrance` のように、一度に複数のパッケージを指定できます。その場合、変更はすべてのパッケージに適用されるか、まったく適用されないかのどちらかです。

どちらの方法でも同じマニフェストが得られるため、好きなように組み合わせて使えます。以下のセクションでは両方の方法を並べて示し、[2 つの方法の違い](#how-the-two-ways-differ) でその違いをまとめます。

## Flix パッケージの追加

Flix パッケージへの依存関係は、マニフェストの `[dependencies]` セクションの項目です：

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "2.0.2", mount = "giftshop" }
```

キーはそのパッケージが公開されている GitHub リポジトリです。依存関係には 2 つの要素を宣言します。要求するバージョンである `version` と、そのパッケージにアクセスするための名前である mount(マウント) です。依存関係には、[依存関係の信頼](./trusting-dependencies.md) で説明するように `security` context を宣言することもできます。

この項目は自分で書くこともできますし、`install` コマンドで Flix に書かせることもできます：

```
install flix/museum-giftshop
```

`install` コマンドは、本来であれば自分で行う選択を代わりに行ってくれます。バージョンを指定しない限り、`install flix/museum-giftshop@2.0.2` のように名前を付けない限りは、そのパッケージの最新リリースを宣言します。そのリポジトリ名が有効な mount であればその名前でパッケージをマウントし、そうでなければ mount 名を尋ねてきます：

```
github:flix/museum-giftshop is reached through a mount, as in 'use MuseumGiftshop::greet'.
Mount [MuseumGiftshop]: giftshop
```

好きな mount 名を入力するか、Enter を押して角括弧内のものを採用します。`--yes` オプションを付けると、`install` は尋ねずに角括弧内のものをそのまま採用します。依存関係の解決が終わると、`install` は何を宣言したかを報告します：

```
Added 'flix/museum-giftshop' v2.0.2, mounted at 'giftshop'.
```

どちらの方法でも、Flix はそのパッケージと、それが依存するすべてのものをダウンロードします。`install` はすぐにそれを行い、自分で書いた項目については、Flix は次に `check` のようなコマンドを実行したときにそれを行います：

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum-giftshop.toml` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.2)... OK.
Downloading Flix dependencies...
  Downloading `flix/museum-giftshop.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.2)... OK.
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

要求したパッケージは 1 つですが、2 つダウンロードされました。Flix パッケージは自身の依存関係を一緒に連れてくるためであり、`flix/museum-giftshop` は `flix/museum-clerk` に依存しています。Flix が各パッケージのどのバージョンを選ぶかについては、[バージョンとアップグレード](./versions-and-upgrades.md) で扱います。

## マウントしたパッケージの使用

Flix パッケージは、私たち自身の名前空間の一部にはなりません。パッケージのモジュールはそれ自身の名前空間に存在し、`use` の中で `::` の前に書く、そのパッケージの *mount* を通じてアクセスします：

```flix
use giftshop::Giftshop

def main(): Unit \ IO =
    Giftshop.buyGift()
```

mount はパッケージ自身ではなく私たち自身のマニフェストの中で宣言するため、2 つのプロジェクトが同じパッケージを別々の名前で参照することもできます。

> **注意:** mount は単純な名前でなければなりません。文字の後に文字・数字・アンダースコアが続く形式です。したがって、名前にハイフンを含むリポジトリには別の mount 名が必要です。だからこそ `github:flix/museum-giftshop` は `giftshop` としてマウントされ、`install` が mount 名を尋ねてくるのもそのためです。

> **注意:** `::` を書けるのは `use` の中だけです。式の中で `giftshop::Giftshop.buyGift()` と書いたり、型の中で `giftshop::Giftshop` と書いたりすることはできません。

> **注意:** パッケージの `pub` 宣言のみにアクセスできます。パッケージが `pub` と宣言していないモジュールは、そのパッケージ内でプライベートです。

## Flix パッケージの削除

Flix パッケージは、`[dependencies]` からその項目を削除するか、`remove` コマンドで削除できます：

```
remove flix/museum-giftshop
```

これは次のように報告します：

```
Removed 'flix/museum-giftshop' v2.0.2, which was mounted at 'giftshop'.
```

どちらの方法でも、削除できるのは自分のマニフェストが宣言しているパッケージだけです。ここでの `flix/museum-clerk` のように、他の依存関係を通じて到達しているパッケージは、その依存関係によって宣言されているため、その依存関係と一緒になくなります。

削除されたパッケージのために Flix がダウンロードしたファイルは、[lib ディレクトリ](#the-lib-directory) で説明するように、無視される状態のまま `lib` ディレクトリに残ります。

## 2 つの方法の違い <a name="how-the-two-ways-differ"></a>

どちらの方法も同じ依存関係を宣言します。違うのは、誰が選択を行うか、そしていつ変更が反映されるかです：

|                       | `flix.toml` を編集する場合                     | コマンドを実行する場合                          |
|-----------------------|--------------------------------------------------|--------------------------------------------------|
| バージョン            | 自分で選ぶ                                        | 最新リリース、または自分が指定したもの            |
| Mount                 | 自分で選ぶ                                        | リポジトリの名前、または自分が答えたもの          |
| Security context      | 自分で選ぶ                                        | なし（つまり `plain`）                            |
| 反映されるタイミング  | 次に `check` などのコマンドを実行したとき          | すぐに                                            |
| 解決に失敗した場合    | エラーが報告され、自分の編集内容はそのまま残る     | エラーが報告され、`flix.toml` は元に戻る          |
| 対象範囲              | Flix パッケージ、Maven、JAR 依存関係               | Flix パッケージのみ                               |

バージョン・Mount・Security context の行は `install` について説明しています。`upgrade` コマンドはバージョンのみを変更し、[バージョンとアップグレード](./versions-and-upgrades.md#upgrading-a-package) で説明するように、別のものを指定しない限りは同じメジャーバージョン内にとどまります。

[バージョンとアップグレード](./versions-and-upgrades.md) の `flix/museum` のように、`plain` 以外の security context で信頼させる必要があるパッケージは、そのためマニフェストを編集して宣言します。

## Maven 依存関係の追加

`[mvn-dependencies]` セクションに、Maven 上で公開されている Java ライブラリへの依存関係を追加できます：

```toml
[mvn-dependencies]
"org.apache.commons:commons-lang3" = "3.12.0"
```

> **注意:** Maven 依存関係や JAR 依存関係にはコマンドが存在しません。これらは `flix.toml` を編集して宣言します。

Flix はこのセクションを Maven の依存関係リゾルバに渡し、そのライブラリと、それが依存するライブラリを一緒にダウンロードします：

```
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (v3.12.0).
  Running Maven dependency resolver.
```

その後は、[Java との相互運用](./interoperability.md) で説明するように、他の Java ライブラリと同じようにそのライブラリを使用します：

```flix
import org.apache.commons.lang3.StringUtils

def main(): Unit \ IO =
    println(StringUtils.reverse("Hello World!"))
```

## URL からの JAR ファイルの追加

`[jar-dependencies]` セクションで、URL からダウンロードする JAR ファイルに依存できます。キーは JAR ファイルが保存されるファイル名で、`.jar` で終わる必要があります：

```toml
[jar-dependencies]
"commons-lang3.jar" = "url:https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
```

Flix は依存関係を解決する際に、これをダウンロードします：

```
Downloading external jar dependencies...
  Downloading `commons-lang3.jar` from `https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar`... OK.
```

> **警告:** JAR 依存関係は、その URL がどこを指していようと、そこから配信されるものがそのまま使われます。外部の JAR 依存関係はできるだけ避けるべきです。Flix パッケージの方が優れており、Maven ライブラリの方が URL よりも優れています。

## lib ディレクトリ <a name="the-lib-directory"></a>

Flix はダウンロードしたものを `lib` ディレクトリに配置します。Flix パッケージは `lib/github` の下に、Maven ライブラリは `lib/cache` に、URL からダウンロードした JAR ファイルは `lib/external` に置かれます。各 Flix パッケージは、そのリポジトリとバージョンごとに保持されます：

```
lib
└── github
    └── flix
        ├── museum-clerk
        │   └── 2.1.2
        │       ├── museum-clerk-2.1.2.fpkg
        │       └── museum-clerk-2.1.2.toml
        └── museum-giftshop
            └── 2.0.2
                ├── museum-giftshop-2.0.2.fpkg
                └── museum-giftshop-2.0.2.toml
```

`lib` ディレクトリは Flix によって管理されるため、コミットすべきではありません。また、このディレクトリはスキャンされません。Flix は `flix.toml` が宣言する依存関係のみを読み込むため、手動で `lib` に配置したパッケージや JAR ファイルは無視され、削除されたパッケージが残していったものも同様に無視されます。

<!--
# Using Dependencies

A Flix project can depend on three kinds of things: Flix packages published on
GitHub, Java libraries published on Maven, and JAR-files downloaded from a URL. We
declare all three in the manifest, and Flix downloads them for us.

Ideally a Flix project depends on Flix packages only. A Flix package is compiled from
source, is built in a security context that limits what it may do, and is written in
the language we are writing. A Maven library, and even more so a JAR-file from a URL,
is a last resort: we should look for a Flix package first, and reach for Java only
when there is none.

## Two Ways to Declare a Dependency

The dependencies of a project are the ones its manifest declares. We can change them
in two ways, and both are equally valid: we edit `flix.toml` ourselves, or we run a
command that edits it for us.

| Task               | By editing `flix.toml`             | By running               |
|--------------------|------------------------------------|--------------------------|
| Add a Flix package | add its entry to `[dependencies]`  | `install <owner>/<repo>` |
| Change its version | change the `version` of its entry  | `upgrade <owner>/<repo>` |
| Remove it          | delete its entry                   | `remove <owner>/<repo>`  |

Each command can name several packages at once, as in
`install flix/museum-giftshop flix/museum-entrance`, and changes either all of them
or none.

Both ways give the same manifest, and we can mix them as we like. The sections below
show them side by side, and [How the Two Ways Differ](#how-the-two-ways-differ) sums
up what sets them apart.

## Adding a Flix Package

A dependency on a Flix package is an entry in the `[dependencies]` section of the
manifest:

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "2.0.2", mount = "giftshop" }
```

The key is the GitHub repository the package is published from. A dependency
declares two things: the `version` we require, and the `mount`, the name we reach
the package under. A dependency may also declare a `security` context, as
described in [Trusting Dependencies](./trusting-dependencies.md).

We can write this entry ourselves, or have Flix write it for us with the `install`
command:

```
install flix/museum-giftshop
```

The `install` command makes the choices that we otherwise make ourselves. It
declares the newest release of the package, unless we name a version, as in
`install flix/museum-giftshop@2.0.2`. It mounts the package under the name of its
repository if that name is a valid mount, and asks us for one otherwise:

```
github:flix/museum-giftshop is reached through a mount, as in 'use MuseumGiftshop::greet'.
Mount [MuseumGiftshop]: giftshop
```

We type the mount we want, or press Enter to take the one in brackets. With the
`--yes` option, `install` takes the one in brackets without asking. Once the
dependencies are resolved, `install` reports what it declared:

```
Added 'flix/museum-giftshop' v2.0.2, mounted at 'giftshop'.
```

Either way, Flix downloads the package and everything it depends on: `install` does
so right away, and for an entry we wrote ourselves, Flix does so the next time we run
a command, such as `check`:

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum-giftshop.toml` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.2)... OK.
Downloading Flix dependencies...
  Downloading `flix/museum-giftshop.fpkg` (v2.0.2)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.2)... OK.
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

We asked for one package and got two: a Flix package brings its own dependencies
with it, and `flix/museum-giftshop` depends on `flix/museum-clerk`. Which version of
each package Flix picks is the subject of
[Versions and Upgrades](./versions-and-upgrades.md).

## Using a Mounted Package

A Flix package does not become part of our own namespace. Its modules live in a
namespace of their own, and we reach them through the package's *mount*, which we
write before `::` in a `use`:

```flix
use giftshop::Giftshop

def main(): Unit \ IO =
    Giftshop.buyGift()
```

The mount is declared in our own manifest, not by the package, so two projects may
reach the same package under different names.

> **Note:** A mount must be a simple name: a letter followed by letters, digits,
> and underscores. A repository whose name contains a hyphen therefore needs a
> different one, which is why `github:flix/museum-giftshop` is mounted as
> `giftshop`, and why `install` asks for its mount.

> **Note:** The `::` can be written in a `use` only. We cannot write
> `giftshop::Giftshop.buyGift()` in an expression, nor `giftshop::Giftshop` in a
> type.

> **Note:** Only the `pub` declarations of a package can be reached. A module that
> a package does not declare `pub` is private to it.

## Removing a Flix Package

We can remove a Flix package by deleting its entry from `[dependencies]`, or with
the `remove` command:

```
remove flix/museum-giftshop
```

which reports:

```
Removed 'flix/museum-giftshop' v2.0.2, which was mounted at 'giftshop'.
```

Either way, we can only remove a package that our manifest declares. A package that
is reached through another dependency, like `flix/museum-clerk` here, is declared by
that dependency, and goes away with it.

The files that Flix downloaded for a removed package stay in the `lib` directory,
where they are ignored, as described in [The lib Directory](#the-lib-directory).

## How the Two Ways Differ

Both ways declare the same dependency. They differ in who makes the choices, and in
when the change takes effect:

|                     | Editing `flix.toml`                          | Running a command                                      |
|---------------------|----------------------------------------------|--------------------------------------------------------|
| Version             | we choose it                                 | the newest release, or the one we name                 |
| Mount               | we choose it                                 | the name of the repository, or the one we answer with  |
| Security context    | we choose it                                 | none, which means `plain`                              |
| Takes effect        | the next time we run a command, e.g. `check` | right away                                             |
| If resolution fails | the error is reported, and our edit stays    | the error is reported, and `flix.toml` is put back     |
| Covers              | Flix packages, Maven, and JAR-dependencies   | Flix packages only                                     |

The version, mount, and security rows describe `install`. The `upgrade` command
changes only the version, and stays within the major version unless we name another,
as described in [Versions and Upgrades](./versions-and-upgrades.md#upgrading-a-package).

A package that must be trusted with a security context other than `plain`, such as
`flix/museum` in [Versions and Upgrades](./versions-and-upgrades.md), is therefore
declared by editing the manifest.

## Adding a Maven Dependency

We can add a dependency on a Java library published on Maven in the
`[mvn-dependencies]` section:

```toml
[mvn-dependencies]
"org.apache.commons:commons-lang3" = "3.12.0"
```

> **Note:** There is no command for Maven- or JAR-dependencies. We declare them by
> editing `flix.toml`.

Flix hands the section to a Maven dependency resolver, which downloads the library
together with the libraries it depends on:

```
Resolving Maven dependencies...
  Adding `org.apache.commons:commons-lang3' (v3.12.0).
  Running Maven dependency resolver.
```

We then use the library as we use any other Java library, as described in
[Interoperability with Java](./interoperability.md):

```flix
import org.apache.commons.lang3.StringUtils

def main(): Unit \ IO =
    println(StringUtils.reverse("Hello World!"))
```

## Adding a JAR-file from a URL

We can depend on a JAR-file that is downloaded from a URL in the
`[jar-dependencies]` section. The key is the file name the JAR-file is saved under,
which must end in `.jar`:

```toml
[jar-dependencies]
"commons-lang3.jar" = "url:https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
```

Flix downloads it while it resolves the dependencies:

```
Downloading external jar dependencies...
  Downloading `commons-lang3.jar` from `https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar`... OK.
```

> **Warning:** A JAR-dependency is whatever the URL serves, from wherever it points.
> We should avoid external JAR-dependencies: a Flix package is better, and a Maven
> library is better than a URL.

## The lib Directory

Flix places what it downloads in the `lib` directory: Flix packages under
`lib/github`, Maven libraries in `lib/cache`, and JAR-files downloaded from a URL
in `lib/external`. Every Flix package is kept under its repository and version:

```
lib
└── github
    └── flix
        ├── museum-clerk
        │   └── 2.1.2
        │       ├── museum-clerk-2.1.2.fpkg
        │       └── museum-clerk-2.1.2.toml
        └── museum-giftshop
            └── 2.0.2
                ├── museum-giftshop-2.0.2.fpkg
                └── museum-giftshop-2.0.2.toml
```

The `lib` directory is managed by Flix and should not be committed. It is not
scanned: Flix loads the dependencies that `flix.toml` declares, and nothing else,
so a package or JAR-file we place in `lib` by hand is ignored, and so is what a
removed package leaves behind.
-->
