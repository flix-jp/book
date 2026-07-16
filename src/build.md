# ビルド管理

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/build.html)を参照してください。

ここでは、ビルドコマンドについて説明します。各コマンドは、コマンドライン、REPL、そして VSCode から実行できます。

## 新しいプロジェクトの作成

`init` コマンドを使うと、ディレクトリの中に新しいプロジェクトを作成できます。

これにより、デフォルトの Flix プロジェクト構造が作成されます：

```
.
├── flix.toml
├── LICENSE.md
├── README.md
├── src
│   └── Main.flix
└── test
    └── TestMain.flix

2 directories, 6 files
```

特に重要なファイルは `flix.toml`、`src/Main.flix`、`test/TestMain.flix` です。

マニフェスト(Manifest)ファイルである `flix.toml` については、次のセクションで説明します。

> **ヒント:** `init` コマンドは安全に使用できます。まだ存在しないファイルのみを作成します。

## プロジェクトのチェック

`check` コマンドを使うと、プロジェクトにコンパイルエラーがないかチェックできます。開発中は、`build` コマンドよりも `check` コマンドの方が望ましいです。`check` コマンドはコード生成を省略するため、大幅に高速だからです。

## プロジェクトのビルド

`build` コマンドを使うと、プロジェクトをコンパイルできます。`build` コマンドを実行すると、プロジェクト全体がコンパイルされ、バイトコード、すなわちコンパイル済みの Java クラスが `build` ディレクトリに出力されます。

Flix には `clean` コマンドはありません。`build` ディレクトリを削除することが同じ役割を果たします。

## JAR ファイルのビルド

`build-jar` コマンドを使うと、プロジェクトを JAR ファイルにコンパイルできます。`build-jar` コマンドは `artifact/project.jar` ファイルを出力します。`main` 関数がある場合は、次のように実行できます：

```bash
$ java -jar artifact/project.jar
```

この JAR ファイルには、`build` ディレクトリ内のすべてのクラスファイルが含まれます。プロジェクト自体、またはその依存関係のいずれかが JAR ファイルに依存している場合、ビルドされた JAR は外部の JAR に依存することがあります。

> **注意:** `build-jar` は自動的に `build` コマンドを呼び出します。

## fat JAR ファイルのビルド（すべての依存関係をバンドルする）

`build-fatjar` コマンドを使うと、プロジェクトを単一のスタンドアロンな fat JAR(ファット JAR)ファイルにコンパイルできます。`build-fatjar` コマンドは、Flix と Maven の*すべて*の依存関係が 1 つの JAR ファイルにバンドルされた `artifact/project.jar` ファイルを出力します。

この JAR ファイルには、`build` ディレクトリ内のすべてのクラスファイルに加えて、`lib` ディレクトリで見つかったすべての Maven 依存関係から抽出されたすべてのクラスファイルが含まれます。

> **注意:** `build-fatjar` は自動的に `build` コマンドを呼び出します。

## Flix プロジェクトのビルド

`build-pkg` コマンドを使うと、プロジェクトを Flix パッケージファイル(fpkg)にまとめることができます。`build-pkg` コマンドを実行すると、`artifact/project.fpkg` ファイルが出力されます。

Flix パッケージファイル(fpkg)は、本質的にはプロジェクトのソースコードの zip ファイルです。Flix パッケージは、その `flix.toml` マニフェストとともに GitHub 上で公開できます。

## プロジェクトの実行

プロジェクトを実行するために JAR ファイルをビルドする必要はありません。`run` コマンドを使うだけで、コンパイルしてメインエントリーポイントを実行できます。

## プロジェクトのテスト

`test` コマンドを使うと、プロジェクト内のすべてのテストケースを実行できます。Flix は `@Test` でマークされたすべての関数を収集して実行し、結果のサマリーを表示します：

```
Running 1 tests...

   PASS  test01 1,1ms

Passed: 1, Failed: 0. Skipped: 0. Elapsed: 3,9ms.
```

<!--
# Build Management

We now discuss the build commands. Each command can be executed from the command
line, from the REPL, and from VSCode. 

## Creating a New Project

We can create a new project, inside a directory, with the `init` command. 

This will create the default Flix project structure:

```
.
├── flix.toml
├── LICENSE.md
├── README.md
├── src
│   └── Main.flix
└── test
    └── TestMain.flix

2 directories, 6 files
```

The most relevant files are `flix.toml`, `src/Main.flix` and
`test/TestMain.flix`.

The `flix.toml` manifest file is discussed in the next section.

> **Tip:** The `init` command is safe to use; it will only create files that do
> not already exist. 

## Checking a Project

We can check a project for compiler errors with the `check` command. During
development, the `check` command is preferable to the `build` command because it
skips code generation (and hence is significantly faster). 

## Building a Project

We can compile a project with the `build` command. Running the `build` command
will compile the entire project and emit bytecode, i.e. compiled Java classes,
to the `build` directory.

Flix has no `clean` command. Deleting the `build` directory serves the same
purpose.

## Building a JAR-file

We can compile a project to a JAR-file with the `build-jar` command. The
`build-jar` command emits a `artifact/project.jar` file. If there is `main`
function, we can run it: 

```bash
$ java -jar artifact/project.jar
```

The JAR-file contains all class files from the `build` directory. The built JAR
may depend on external JARs, if the project, or one of its dependencies, depends
on JAR-files.

> **Note:** `build-jar` automatically invokes the `build` command.

## Building a fat JAR-file (bundling all dependencies)

We can compile a project to a single standalone fat JAR-file with the
`build-fatjar` command. The `build-fatjar` command emits a
`artifact/project.jar` file where _all_ dependencies — both Flix and Maven — are
bundled into one single JAR-file. 

The JAR-file contains all class files from the `build` directory together with
all class files extract from all Maven dependencies found in the `lib`
directory. 

> **Note:** `build-fatjar` automatically invokes the `build` command.

## Building a Flix Project

We can bundle a project into a Flix package file (fpkg) with the `build-pkg`
command. Running the `build-pkg` command emits a `artifact/project.fpkg` file.

A Flix package file (fpkg) is essentially zip-file of the project source code. A
Flix package, together with its `flix.toml` manifest, can be published on
GitHub.

## Running a Project

We do not have to build a JAR-file to run a project, we can simply use the `run`
command which will compile and run the main entry point.

## Testing a Project

We can use the `test` command to run all test cases in a project. Flix will
collect all functions marked with `@Test`, execute them, and print a summary of
the results:

```
Running 1 tests...

   PASS  test01 1,1ms

Passed: 1, Failed: 0. Skipped: 0. Elapsed: 3,9ms.
```
-->
