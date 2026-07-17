# ビルドとパッケージ管理

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/build-and-packages.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/build-and-packages.md)ください。

Flix にはビルドシステム(Build system)とパッケージマネージャ(Package manager)が付属しています。ビルドシステムを使うと、Flix プログラムを Java クラスの集まりにコンパイルしたり、fat JAR(ファット JAR)をビルドしたりすることが簡単にできます。パッケージマネージャを使うと、Flix パッケージを作成して GitHub に公開し、マニフェストファイル(Manifest file)を介してそれらに依存することができます。また、パッケージマネージャによって、Maven で公開されている Java の JAR アーティファクトに依存することも可能です。

Flix のビルドシステムは、以下のコマンドをサポートしています：

- `init`: カレントディレクトリに新しい Flix プロジェクトを作成します。
- `check`: 現在のプロジェクトにコンパイルエラーがないか検査します。
- `build`: 現在のプロジェクトをビルドします（つまり、Java バイトコードを出力します）。
- `build-jar`: 現在のプロジェクトから jar ファイルをビルドします。
- `build-fatjar`: すべての依存関係をバンドルした jar ファイルをビルドします。
- `build-pkg`: 現在のプロジェクトから fpkg ファイルをビルドします。
- `run`: 現在のプロジェクトの main を実行します。
- `test`: 現在のプロジェクトのすべてのテストを実行します。

すべてのコマンドは、コマンドライン、REPL、そして VSCode から実行できます。

`build-pkg` を除くすべてのコマンドは、マニフェストファイルがなくても動作します。Flix プロジェクトをビルド、パッケージ化、公開するには、`flix.toml` マニフェストが必要です。`init` コマンドは、`flix.toml` マニフェストがまだ存在しない場合、空のスケルトンを作成します。

## プロジェクト構造

Flix は、`*.flix`、`src/**/*.flix,`、`test/**/*.flix` のパスからソースファイルを探索します。

Flix は、`lib/**/*.fpkg` と `lib/**/*.jar` のパスから Flix パッケージと JAR を探索します。

<!--
# Build and Package Management

Flix comes with a build system and package manager. The build system makes it
simple to compile a Flix program to a collection of Java classes and to build a
fat JAR. The package manager makes it possible to create Flix packages, publish
them on GitHub, and depend on them via a manifest file. The package manager also
makes it possible to depend on Java JAR-artifacts published on Maven. 

The Flix build system supports the following commands:

- `init`: creates a new Flix project in the current directory.
- `check`: checks the current project for compiler errors.
- `build`: builds the current project (i.e. emits Java bytecode).
- `build-jar`: builds a jar-file from the current project. 
- `build-fatjar`: builds a jar-file with all dependencies bundled.
- `build-pkg`: builds a fpkg-file from the current project. 
- `run`: runs main in current project.  
- `test`: runs all tests in the current project.

All commands can be executed from the command line, from the REPL, and from
VSCode.

All commands, except `build-pkg` work without a manifest file. To build,
package, and publish a Flix project, a `flix.toml` manifest is required. The
`init` command will create an empty skeleton `flix.toml` manifest, if not
already present. 

## Project Structure

Flix scans for source files in the paths `*.flix`, `src/**/*.flix,`, and
`test/**/*.flix`.

Flix scans for Flix packages and JARs in the paths `lib/**/*.fpkg` and
`lib/**/*.jar`.
-->
