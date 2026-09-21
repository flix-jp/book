# 依存関係の使用

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/using-dependencies.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/using-dependencies.md)ください。

Flix プロジェクトは 3 種類のものに依存できます。GitHub 上で公開されている Flix パッケージ、Maven 上で公開されている Java ライブラリ、そして URL からダウンロードする JAR ファイルです。この 3 つはいずれもマニフェストの中で宣言し、Flix がそれらをダウンロードします。

理想的には、Flix プロジェクトは Flix パッケージのみに依存するべきです。Flix パッケージはソースコードからコンパイルされ、できることを制限する security context(セキュリティコンテキスト) の中でビルドされ、私たちが書いているのと同じ言語で書かれています。Maven ライブラリ、そしてそれ以上に URL からの JAR ファイルは最後の手段です。まず Flix パッケージを探し、それが存在しない場合にのみ Java に頼るべきです。

## Flix パッケージの追加

マニフェストの `[dependencies]` セクションに、Flix パッケージへの依存関係を追加できます：

```toml
[dependencies]
"github:flix/museum" = { version = "2.1.0", mount = "museum" }
```

キーはそのパッケージが公開されている GitHub リポジトリです。依存関係には 2 つの要素を宣言します。要求するバージョンである `version` と、そのパッケージにアクセスするための名前である mount(マウント) です。依存関係には、[依存関係の信頼](./trusting-dependencies.md) で説明するように `security` context を宣言することもできます。

次にコマンドを実行すると、Flix はそのパッケージと、それが依存するすべてのものをダウンロードします：

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v2.1.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.0)... OK.
  Downloading `flix/museum-entrance.toml` (v2.0.0)... OK.
  Downloading `flix/museum-giftshop.toml` (v2.0.0)... OK.
  Downloading `flix/museum-restaurant.toml` (v2.0.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.0.0)... OK.
  Cached `flix/museum-clerk.toml` (v2.0.0).
  Raised `flix/museum-clerk` (v2.0.0 -> v2.1.0), required by `flix/museum` (v2.1.0).
Downloading Flix dependencies...
  Downloading `flix/museum.fpkg` (v2.1.0)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.0)... OK.
  Downloading `flix/museum-entrance.fpkg` (v2.0.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v2.0.0)... OK.
  Downloading `flix/museum-restaurant.fpkg` (v2.0.0)... OK.
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

要求したパッケージは 1 つですが、5 つダウンロードされました。Flix パッケージは自身の依存関係を一緒に連れてくるためです。Flix がどのバージョンを選ぶか、そして `flix/museum-clerk` のマニフェストがなぜ 2 回ダウンロードされるのかについては、[バージョンとアップグレード](./versions-and-upgrades.md) で扱います。

## マウントしたパッケージの使用

Flix パッケージは、私たち自身の名前空間の一部にはなりません。パッケージのモジュールはそれ自身の名前空間に存在し、`use` の中で `::` の前に書く、そのパッケージの *mount* を通じてアクセスします：

```flix
use museum::Museum

def main(): Unit \ IO =
    Museum.visitMuseum()
```

mount はそれぞれのマニフェストの中で自分で選ぶため、2 つのプロジェクトが同じパッケージを別々の名前で参照することもできます。

> **注意:** mount は単純な名前でなければなりません。文字の後に文字・数字・アンダースコアが続く形式です。したがって、名前にハイフンを含むリポジトリには別の mount 名が必要です。だからこそ `github:flix/museum-clerk` は `clerk` としてマウントされます。

> **注意:** `::` を書けるのは `use` の中だけです。式の中で `museum::Museum.visitMuseum()` と書いたり、型の中で `museum::Museum` と書いたりすることはできません。

> **注意:** パッケージの `pub` 宣言のみにアクセスできます。パッケージが `pub` と宣言していないモジュールは、そのパッケージ内でプライベートです。

## Maven 依存関係の追加

`[mvn-dependencies]` セクションに、Maven 上で公開されている Java ライブラリへの依存関係を追加できます：

```toml
[mvn-dependencies]
"org.apache.commons:commons-lang3" = "3.12.0"
```

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

## lib ディレクトリ

Flix はダウンロードしたものを `lib` ディレクトリに配置します。Flix パッケージは `lib/github` の下に、Maven ライブラリは `lib/cache` に、URL からダウンロードした JAR ファイルは `lib/external` に置かれます。各 Flix パッケージは、そのリポジトリとバージョンごとに保持されます：

```
lib
└── github
    └── flix
        └── museum
            └── 2.1.0
                ├── museum-2.1.0.fpkg
                └── museum-2.1.0.toml
```

`lib` ディレクトリは Flix によって管理されるため、コミットすべきではありません。また、このディレクトリはスキャンされません。Flix は `flix.toml` が宣言する依存関係のみを読み込むため、手動で `lib` に配置したパッケージや JAR ファイルは無視されます。

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

## Adding a Flix Package

We can add a dependency on a Flix package in the `[dependencies]` section of the
manifest:

```toml
[dependencies]
"github:flix/museum" = { version = "2.1.0", mount = "museum" }
```

The key is the GitHub repository the package is published from. A dependency
declares two things: the `version` we require, and the `mount`, the name we reach
the package under. A dependency may also declare a `security` context, as
described in [Trusting Dependencies](./trusting-dependencies.md).

The next time we run a command, Flix downloads the package and everything it
depends on:

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
  Downloading `flix/museum.toml` (v2.1.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.1.0)... OK.
  Downloading `flix/museum-entrance.toml` (v2.0.0)... OK.
  Downloading `flix/museum-giftshop.toml` (v2.0.0)... OK.
  Downloading `flix/museum-restaurant.toml` (v2.0.0)... OK.
  Downloading `flix/museum-clerk.toml` (v2.0.0)... OK.
  Cached `flix/museum-clerk.toml` (v2.0.0).
  Raised `flix/museum-clerk` (v2.0.0 -> v2.1.0), required by `flix/museum` (v2.1.0).
Downloading Flix dependencies...
  Downloading `flix/museum.fpkg` (v2.1.0)... OK.
  Downloading `flix/museum-clerk.fpkg` (v2.1.0)... OK.
  Downloading `flix/museum-entrance.fpkg` (v2.0.0)... OK.
  Downloading `flix/museum-giftshop.fpkg` (v2.0.0)... OK.
  Downloading `flix/museum-restaurant.fpkg` (v2.0.0)... OK.
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

We asked for one package and got five: a Flix package brings its own dependencies
with it. Which versions Flix picks, and why the manifest of `flix/museum-clerk` is
downloaded twice, is the subject of
[Versions and Upgrades](./versions-and-upgrades.md).

## Using a Mounted Package

A Flix package does not become part of our own namespace. Its modules live in a
namespace of their own, and we reach them through the package's *mount*, which we
write before `::` in a `use`:

```flix
use museum::Museum

def main(): Unit \ IO =
    Museum.visitMuseum()
```

We choose the mount ourselves, in our own manifest, so two projects may reach the
same package under different names.

> **Note:** A mount must be a simple name: a letter followed by letters, digits,
> and underscores. A repository whose name contains a hyphen therefore needs a
> different one, which is why `github:flix/museum-clerk` is mounted as `clerk`.

> **Note:** The `::` can be written in a `use` only. We cannot write
> `museum::Museum.visitMuseum()` in an expression, nor `museum::Museum` in a type.

> **Note:** Only the `pub` declarations of a package can be reached. A module that
> a package does not declare `pub` is private to it.

## Adding a Maven Dependency

We can add a dependency on a Java library published on Maven in the
`[mvn-dependencies]` section:

```toml
[mvn-dependencies]
"org.apache.commons:commons-lang3" = "3.12.0"
```

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
        └── museum
            └── 2.1.0
                ├── museum-2.1.0.fpkg
                └── museum-2.1.0.toml
```

The `lib` directory is managed by Flix and should not be committed. It is not
scanned: Flix loads the dependencies that `flix.toml` declares, and nothing else,
so a package or JAR-file we place in `lib` by hand is ignored.
-->
