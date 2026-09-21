# チェック・実行・テスト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/checking-running-and-testing.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/checking-running-and-testing.md)ください。

プロジェクトに取り組んでいる間、主に使うコマンドは 3 つです。コード生成なしでプロジェクトをコンパイルしてエラーを報告する `check`、プロジェクトをコンパイルして `main` 関数を実行する `run`、そして `@Test` が付いたすべての関数を実行する `test` です。

## エラーのチェック

`check` コマンドを使うと、プロジェクトにコンパイルエラーがないかチェックできます：

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

Flix はすべてのコマンドを、プロジェクトの依存関係の解決から開始します。これがこの出力内容です。私たちのプロジェクトにはまだ依存関係がないため、ダウンロードするものはありません。以降の例では、これらの行は省略します。

プロジェクトがコンパイルできない場合、Flix は代わりにエラーを報告します。例えば、整数に対して `and` を使う関数を追加すると：

```flix
def isPositive(x: Int32): Bool = x and true
```

`check` は次のように報告します：

```
-- Type Error [E7796] -------------------------------------------- src/Main.flix

>> Unexpected type: expected 'Bool', found 'Int32'.

5 | def isPositive(x: Int32): Bool = x and true
                                     ^
                                     unexpected type
```

開発中は、`build` コマンドよりも `check` コマンドの方が望ましいです。`check` はコード生成を省略するため、大幅に高速だからです。

## プログラムの実行

`run` コマンドを使うと、プロジェクトをコンパイルして実行できます。事前に何かをビルドしておく必要はありません：

```
Hello World!
```

`run` コマンドは、プロジェクトの `main` 関数を実行します。`main` がどのようなものかについては、[main 関数](./main.md) を参照してください。

## テストの実行

`test` コマンドを使うと、テストを実行できます。Flix は `@Test` が付いたすべての関数を収集して実行し、結果のサマリーを表示します：

```
Running 1 tests...

   PASS  test01 1.3ms

Passed: 1, Failed: 0. Skipped: 0. Elapsed: 3.8ms.
```

テストの書き方については、[テストフレームワーク](./test-framework.md) を参照してください。

## ドキュメントの生成

`doc` コマンドを使うと、プロジェクトの API ドキュメントを生成できます。Flix はドキュメントを `build/doc` ディレクトリに書き出します：

```
build
└── doc
    ├── favicon.png
    ├── index.html
    ├── index.js
    └── styles.css
```

このドキュメントは、`build/doc/index.html` をブラウザで開くことで読めます。

## 統計情報の表示

`stat` コマンドを使うと、プロジェクトに関する統計情報を表示できます：

```
<unnamed> 0.1.0

2 files, 5 lines: 4 code, 1 comment, 0 blank.
0 modules, 2 defs: 0 pure, 2 effectful, 0 effect polymorphic.
0 types, 0 traits, 0 instances, 0 effects.
```

見出しの部分には、そのパッケージが公開されるリポジトリの名前（何も宣言されていない場合は `<unnamed>`）と、それに続くバージョンが表示されます。

プログラムが期待どおりに動くようになったら、いよいよ出荷できるものをビルドする準備が整います。

<!--
# Checking, Running, and Testing

While we work on a project we mostly use three commands: `check` to compile the
project and report errors without generating code, `run` to compile the project and
run its `main` function, and `test` to run every function marked with `@Test`.

## Checking for Errors

We can check the project for compiler errors with the `check` command:

```
Found 'flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
```

Flix begins every command by resolving the dependencies of the project, which is
what these lines report. Our project has no dependencies yet, so there is nothing
to download. We leave these lines out of the examples that follow.

If the project does not compile, Flix reports the errors instead. If we add a
function that uses `and` on an integer:

```flix
def isPositive(x: Int32): Bool = x and true
```

then `check` reports:

```
-- Type Error [E7796] -------------------------------------------- src/Main.flix

>> Unexpected type: expected 'Bool', found 'Int32'.

5 | def isPositive(x: Int32): Bool = x and true
                                     ^
                                     unexpected type
```

During development, the `check` command is preferable to the `build` command,
because `check` skips code generation and hence is significantly faster.

## Running the Program

We can compile and run the project with the `run` command. We do not have to build
anything first:

```
Hello World!
```

The `run` command runs the `main` function of the project. See
[The Main Function](./main.md) for what `main` may look like.

## Running the Tests

We can run the tests with the `test` command. Flix collects every function marked
with `@Test`, runs it, and prints a summary:

```
Running 1 tests...

   PASS  test01 1.3ms

Passed: 1, Failed: 0. Skipped: 0. Elapsed: 3.8ms.
```

See [Test Framework](./test-framework.md) for how to write tests.

## Generating Documentation

We can generate API documentation for the project with the `doc` command. Flix
writes the documentation to the `build/doc` directory:

```
build
└── doc
    ├── favicon.png
    ├── index.html
    ├── index.js
    └── styles.css
```

We read it by opening `build/doc/index.html` in a browser.

## Printing Statistics

We can print statistics about the project with the `stat` command:

```
<unnamed> 0.1.0

2 files, 5 lines: 4 code, 1 comment, 0 blank.
0 modules, 2 defs: 0 pure, 2 effectful, 0 effect polymorphic.
0 types, 0 traits, 0 instances, 0 effects.
```

The header names the package by the repository it is published from, or
`<unnamed>` when it declares none, followed by its version.

Once the program does what we want, we are ready to build something we can ship.
-->
