# アーティファクトのビルド

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/building-artifacts.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/building-artifacts.md)ください。

プログラムが動くようになったら、出荷できるものをビルドできます。Java クラスファイル、JAR ファイル、スタンドアロンな fat JAR ファイル、あるいは Flix パッケージです。

このうち普段よく使うのは 2 つです。プログラムをエンドユーザーに配布する場合は fat JAR ファイルです。Java が動く環境であればどこでも単独で実行できるからです。他の Flix 開発者とライブラリを共有する場合は Flix パッケージです。

## プロジェクトのコンパイル

`build` コマンドを使うと、プロジェクトをコンパイルできます。`build` コマンドはコード生成を含めてプロジェクト全体をコンパイルしますが、ディスクには何も書き込みません。プロジェクト全体がコンパイルできることを確認するために使います。

## クラスファイルのビルド

`build-classes` コマンドを使うと、プロジェクトを Java クラスファイルにコンパイルできます。Flix はクラスファイルを `build/class` ディレクトリに書き出し、プロジェクトに `main` 関数があれば、`java` で実行できます：

```shell
$ java -cp build/class Main
Hello World!
```

クラスファイルは、出荷したいものとしてはあまり適していません。ディレクトリをまとめて保持しなければならず、すべての依存関係を自分でクラスパスに追加する必要もあります。プログラムを他の人に渡すには、代わりに fat JAR ファイルをビルドします。

> **注意:** プロジェクト自体、またはその依存関係のいずれかが JAR ファイルに依存している場合は、それらもクラスパスに追加する必要があります。

## JAR ファイルのビルド

`build-jar` コマンドを使うと、JAR ファイルをビルドできます。Flix は JAR ファイルを `artifact` ディレクトリに書き出し、プロジェクトのディレクトリ名にちなんで命名します。例えば `hello-world` というディレクトリのプロジェクトなら `artifact/hello-world.jar` になります。これは `java` で実行できます：

```shell
$ java -jar artifact/hello-world.jar
Hello World!
```

この JAR ファイルには、プロジェクトのクラスファイルと、プロジェクトに `resources` ディレクトリがあればその中のファイルが含まれます。

> **注意:** `build-jar` コマンドはプロジェクト自体をコンパイルします。事前に `build` や `build-classes` を実行する必要はありません。

このようにビルドされた JAR ファイルには、プロジェクトの依存関係は含まれ*ません*。プログラムが（[依存関係の使用](./using-dependencies.md) で説明するような）Java ライブラリなど何らかの依存関係を使っている場合、この JAR ファイルを実行すると JVM がそれを見つけられずに失敗します：

```shell
$ java -jar artifact/inventory.jar
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/commons/lang3/StringUtils
```

対応方法は 2 つあります。依存関係を自分でクラスパスに追加するか、JAR ファイルにバンドルするかです。

## fat JAR ファイルへの依存関係のバンドル

`build-fatjar` コマンドを使うと、すべての依存関係をバンドルした JAR ファイル、すなわち *fat* JAR ファイルをビルドできます。Flix は同じ場所にこれを書き出し、これでプログラムは単独で動作するようになります：

```shell
$ java -jar artifact/inventory.jar
!dlroW olleH
```

fat JAR ファイルには、プロジェクトのクラスファイル、`resources` ディレクトリ内のファイル、そして `lib` ディレクトリにあるすべての JAR ファイルの内容、つまり Flix と Maven の*すべての*依存関係が含まれます。

> **注意:** `build-jar` コマンドと `build-fatjar` コマンドは、同じ JAR ファイルに書き込みます。最後に実行したどちらかのコマンドの結果が残ります。

> **注意:** `build-fatjar` コマンドはプロジェクト自体をコンパイルします。事前に `build` や `build-classes` を実行する必要はありません。

## Flix パッケージのビルド

`build-pkg` コマンドを使うと、プロジェクトを Flix パッケージにまとめることができます。Flix はパッケージを `artifact` ディレクトリに書き出し、その隣にマニフェストをコピーします：

```
artifact
├── flix.toml
└── package.fpkg
```

この 2 つのファイルこそがリリースの中身であり、[パッケージの公開](./publishing-a-package.md) で説明する `release` コマンドがアップロードするものでもあります。どちらのファイルも固定された名前を持ちます。パッケージはプロジェクトの名前やビルドされる場所にかかわらず `package.fpkg` と呼ばれます。これにより、リリースのファイルは何も読み込まなくても、リポジトリとバージョンだけで参照できます。

> **注意:** JAR ファイルはこの規則に従いません。JAR ファイルは何もアドレスで取得しに来ないローカルなビルド成果物なので、`build-jar` と `build-fatjar` は引き続きプロジェクトのディレクトリ名にちなんで命名します。そのため、`hello-world` という名前のプロジェクトディレクトリでは、`artifact` ディレクトリに `package.fpkg`、`flix.toml`、`hello-world.jar` が入ることになります。

Flix パッケージは、本質的にはプロジェクトのソースコードの zip ファイルです。マニフェスト、`README.md`、`LICENSE.md`、そして `src` にある Flix ファイルが含まれます。テストやコンパイル済みコードは含まれません。Flix パッケージは、それに依存する側がソースコードからコンパイルします。マニフェストとともに、[パッケージの公開](./publishing-a-package.md) で説明するように GitHub 上で公開できます。

> **注意:** `build-pkg` コマンドはまずプロジェクトをチェックし、コンパイルできないソースコードからのパッケージのビルドを拒否します。

> **注意:** `build-pkg` コマンドにはマニフェストが必要です。マニフェストのないパッケージは公開できないためです。

## クリーンアップ

`clean` コマンドを使うと、`build` ディレクトリを削除できます。これにより、`build-classes` が書き出したクラスファイルと、`doc` が書き出したドキュメントが削除されます。

> **注意:** `clean` コマンドは `artifact` ディレクトリには手をつけません。自分でビルドした JAR ファイルやパッケージファイルを削除するのは自分自身の役目です。

<!--
# Building Artifacts

When the program works, we can build something we can ship: Java class files, a
JAR-file, a standalone fat JAR-file, or a Flix package.

Two of these are what we usually want: a fat JAR-file, if we distribute a program to
end users, since it runs on its own wherever Java runs, and a Flix package, if we
share a library with other Flix developers.

## Compiling the Project

We can compile the project with the `build` command. The `build` command compiles
the entire project, including code generation, but writes nothing to disk. We use
it to check that the whole project compiles.

## Building Class Files

We can compile the project to Java class files with the `build-classes` command.
Flix writes the class files to the `build/class` directory, and if the project has
a `main` function, we can run it with `java`:

```shell
$ java -cp build/class Main
Hello World!
```

Class files are rarely what we want to ship: we have to keep the directory together,
and we have to put every dependency on the class path ourselves. To hand the program
to someone else, we build a fat JAR-file instead.

> **Note:** If the project, or one of its dependencies, depends on JAR-files, then
> these must also be on the class path.

## Building a JAR-file

We can build a JAR-file with the `build-jar` command. Flix writes the JAR-file to the
`artifact` directory, named after the project directory — a project in a directory
called `hello-world` gives us `artifact/hello-world.jar` — and we can run it with
`java`:

```shell
$ java -jar artifact/hello-world.jar
Hello World!
```

The JAR-file holds the class files of the project together with the files in the
`resources` directory, if the project has one.

> **Note:** The `build-jar` command compiles the project itself. There is no need
> to run `build` or `build-classes` first.

A JAR-file built this way does *not* hold the dependencies of the project. If the
program uses one — say a Java library, as described in
[Using Dependencies](./using-dependencies.md) — then the JVM fails to find it when
we run the JAR-file:

```shell
$ java -jar artifact/inventory.jar
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/commons/lang3/StringUtils
```

We can either put the dependencies on the class path ourselves, or bundle them
into the JAR-file.

## Bundling Dependencies in a Fat JAR-file

We can build a JAR-file with all dependencies bundled — a *fat* JAR-file — with
the `build-fatjar` command. Flix writes it to the same place, and now the program
runs on its own:

```shell
$ java -jar artifact/inventory.jar
!dlroW olleH
```

The fat JAR-file holds the class files of the project, the files in the `resources`
directory, and the contents of every JAR-file in the `lib` directory, i.e. _all_
dependencies — both Flix and Maven.

> **Note:** The `build-jar` and `build-fatjar` commands write to the same
> JAR-file. Whichever we ran last is the one we have.

> **Note:** The `build-fatjar` command compiles the project itself. There is no
> need to run `build` or `build-classes` first.

## Building a Flix Package

We can bundle the project into a Flix package with the `build-pkg` command. Flix
writes the package to the `artifact` directory, and copies the manifest next to it:

```
artifact
├── flix.toml
└── package.fpkg
```

These two files are exactly what a release holds, and exactly what the `release`
command uploads, as described in [Publishing a Package](./publishing-a-package.md).
Both carry a fixed name: the package is called `package.fpkg` whatever the project is
called and wherever it is built, so that the files of a release can be addressed from
the repository and the version alone, without reading anything first.

> **Note:** JAR-files do not follow this rule: a JAR-file is a local build output that
> nothing fetches by address, so `build-jar` and `build-fatjar` keep naming it after
> the project directory. In a project directory called `hello-world`, the `artifact`
> directory therefore holds `package.fpkg`, `flix.toml`, and `hello-world.jar`.

A Flix package is essentially a zip-file of the source code of the project: it
holds the manifest, the `README.md`, the `LICENSE.md`, and the Flix files in `src`.
It holds no tests and no compiled code — a Flix package is compiled from source by
whoever depends on it. Together with its manifest, it can be published on GitHub,
as described in [Publishing a Package](./publishing-a-package.md).

> **Note:** The `build-pkg` command checks the project first, and refuses to build
> a package from source code that does not compile.

> **Note:** The `build-pkg` command requires a manifest, since a package without a
> manifest cannot be published.

## Cleaning Up

We can remove the `build` directory with the `clean` command. This deletes the
class files written by `build-classes` and the documentation written by `doc`.

> **Note:** The `clean` command leaves the `artifact` directory alone. We remove
> the JAR- and package-files we have built ourselves.
-->
