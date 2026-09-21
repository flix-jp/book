# プロジェクトとパッケージ管理

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/build-and-packages.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/build-and-packages.md)ください。

Flix にはビルドシステム(Build system)とパッケージマネージャ(Package manager)が付属しています。

ビルドシステムは、プロジェクトを Java クラスファイル、JAR ファイル、あるいは単一のスタンドアロンな fat JAR ファイルにコンパイルします。パッケージマネージャは、プロジェクトが依存する Flix パッケージや Java ライブラリをダウンロードし、プロジェクトを GitHub 上に他の人が依存できるパッケージとして公開します。

Flix プロジェクトは、`flix.toml` マニフェスト、`src` 内の Flix ソースコード、`test` 内のテストからなるディレクトリです。このようなプロジェクトは `init` コマンドで作成し、`check`・`run`・`test` コマンドで作業を進め、`build-jar` コマンドや `release` コマンドで出荷します。

コマンドはコマンドラインから実行します。Flix をどのように起動するかは、[はじめに](./getting-started.md) で説明したように、インストール方法によって異なります。ほとんどのコマンドは REPL からも実行でき、REPL ではコロンを付けて `:check` や `:test` のように書きます。VSCode では何かを実行する必要はありません。プロジェクトは入力するたびにチェックされ、`main` の上や各テストの上に `▶ Run` と `▶ Run Tests` が表示されます。

> **ヒント:** 作業中は REPL を使うことを推奨します。REPL はプロジェクトをメモリ上に保持し、変更された部分のみを再コンパイルするため、REPL での `:check` や `:test` は、毎回ゼロから実行されるシェルからの同じコマンドの再実行よりもはるかに高速です。

## コマンド一覧

| コマンド          | 説明                                                     |
|-------------------|----------------------------------------------------------|
| `init`            | カレントディレクトリに新しいプロジェクトを作成します。    |
| `check`           | プロジェクトにコンパイルエラーがないか検査します。         |
| `run`             | プロジェクトの `main` を実行します。                       |
| `test`            | プロジェクトのすべてのテストを実行します。                 |
| `doc`             | プロジェクトの API ドキュメントを生成します。               |
| `stat`            | プロジェクトに関する統計情報を表示します。                 |
| `build`           | プロジェクト全体をコンパイルします。                       |
| `build-classes`   | プロジェクトを Java クラスファイルにコンパイルします。      |
| `build-jar`       | プロジェクトから JAR ファイルをビルドします。               |
| `build-fatjar`    | すべての依存関係をバンドルした JAR ファイルをビルドします。 |
| `build-pkg`       | プロジェクトから Flix パッケージ（fpkg ファイル）をビルドします。 |
| `clean`           | `build` ディレクトリを削除します。                         |
| `outdated`        | より新しいバージョンが利用可能な依存関係を表示します。       |
| `release`         | プロジェクトの新しいバージョンを GitHub にリリースします。   |

プロジェクトの依存関係は、[依存関係の使用](./using-dependencies.md) で説明するように、マニフェストの中で宣言します。すべての依存関係は、[依存関係の信頼](./trusting-dependencies.md) で説明するように、その依存関係が実行できることを制限する **security context（セキュリティコンテキスト）** の中でビルドされます。

> **注意:** ほとんどのコマンドは、マニフェストが存在しないディレクトリでも動作します。その場合 Flix は `*.flix`、`src/**`、`test/**` からソースファイルを読み込み、依存関係の解決は行いません。`build-pkg`・`clean`・`release` の各コマンドにはマニフェストが必要です。

<!--
# Projects and Packages

Flix comes with a build system and a package manager.

The build system compiles a project to Java class files, to a JAR-file, or to a
single standalone fat JAR-file. The package manager downloads the Flix packages
and Java libraries that a project depends on, and publishes a project on GitHub
as a package that others can depend on.

A Flix project is a directory with a `flix.toml` manifest, Flix source code in
`src`, and tests in `test`. We create such a project with the `init` command, we
work on it with the `check`, `run`, and `test` commands, and we ship it with the
`build-jar` and `release` commands.

We run the commands from the command line. How we invoke Flix depends on how we
installed it, as described in [Getting Started](./getting-started.md). Most
commands can also be run from the REPL, where they are written with a colon, e.g.
`:check` and `:test`. In VSCode we do not have to run anything: the project is
checked as we type, and `▶ Run` and `▶ Run Tests` appear above `main` and above
every test.

> **Tip:** We should prefer the REPL while we work. The REPL keeps the project in
> memory and recompiles only what has changed, so `:check` and `:test` in the REPL
> are much faster than re-running the same commands from the shell, where every run
> starts from scratch.

## The Commands at a Glance

| Command         | Description                                             |
|-----------------|-----------------------------------------------------------|
| `init`          | creates a new project in the current directory.         |
| `check`         | checks the project for compiler errors.                 |
| `run`           | runs `main` in the project.                             |
| `test`          | runs all tests in the project.                          |
| `doc`           | generates API documentation for the project.            |
| `stat`          | prints statistics about the project.                    |
| `build`         | compiles the entire project.                            |
| `build-classes` | compiles the project to Java class files.               |
| `build-jar`     | builds a JAR-file from the project.                     |
| `build-fatjar`  | builds a JAR-file with all dependencies bundled.        |
| `build-pkg`     | builds a Flix package (an fpkg-file) from the project.  |
| `clean`         | removes the `build` directory.                          |
| `outdated`      | shows dependencies which have newer versions available. |
| `release`       | releases a new version of the project to GitHub.        |

We declare the dependencies of a project in its manifest, as described in
[Using Dependencies](./using-dependencies.md). Every dependency is built in a
*security context* that limits what it is allowed to do, as described in
[Trusting Dependencies](./trusting-dependencies.md).

> **Note:** Most commands also work in a directory that has no manifest. Flix
> then loads source files from `*.flix`, `src/**`, and `test/**`, and has no
> dependencies to resolve. The `build-pkg`, `clean`, and `release` commands
> require a manifest.
-->
