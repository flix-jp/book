# プロジェクトの作成

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/creating-a-project.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/creating-a-project.md)ください。

Flix プロジェクトは、`flix.toml` マニフェスト、`src` 内の Flix ソースコード、`test` 内のテストを含むディレクトリです。

このようなプロジェクトは `init` コマンドで作成できます。このコマンドはデフォルトのプロジェクト構造を作成します：

```
.
├── .github
│   └── workflows
│       └── build-and-test.yaml
├── .gitignore
├── flix.toml
├── LICENSE.md
├── README.md
├── src
│   └── Main.flix
└── test
    └── TestMain.flix

4 directories, 7 files
```

実際に作業するファイルは `flix.toml`、`src/Main.flix`、`test/TestMain.flix` の 3 つです。残りは編集することを前提とした出発点で、プレースホルダーのテキストが入った `README.md` と `LICENSE.md`、`.gitignore`、そして GitHub Actions のワークフローです。

> **ヒント:** `init` コマンドは、まだ存在しないファイルのみを作成します。すでにファイルがあるディレクトリで実行しても安全で、既存のプロジェクトに対して不足しているものを追加するためにも実行できます。

## マニフェスト

マニフェストはプロジェクトを記述するものです。`init` が書き出すマニフェストは最小構成です：

```toml
[package]
version = "0.1.0"
flix    = "0.76.2"

# repository = "github:<owner>/hello-world"
```

`version` はプロジェクトのバージョンです。`flix` フィールドは、そのプロジェクトをビルドできる最も古い Flix のバージョンで、`init` は現在実行している Flix のバージョンを設定します。`repository` フィールドは、[パッケージの公開](./publishing-a-package.md) で説明するように、プロジェクトを公開する際の GitHub リポジトリを表しますが、`init` はこれをコメントアウトした状態にします。

マニフェストはプロジェクトの成長に合わせて拡張されます。他のパッケージに依存する場合は、[依存関係の使用](./using-dependencies.md) で説明するように `[dependencies]` セクションを追加します。

> **注意:** Flix では、バージョン番号は [SemVer](https://semver.org/) に従う必要があります。

> **注意:** パッケージは、それが公開されるリポジトリによって名前が決まります。それ以外の要素は一切関係しません。

## Flix がソースコードを探す場所

Flix は `*.flix`、`src/**/*.flix`、`test/**/*.flix` のパスからソースファイルを探索します。`src` と `test` の中のファイルは好きなように整理でき、Flix はどの深さにあっても見つけます。

生成される `src/Main.flix` にはプログラムのエントリーポイントが入っています：

```flix
// The main entry point.
def main(): Unit \ IO =
    println("Hello World!")
```

そして、生成される `test/TestMain.flix` には 1 つのテストが入っています：

```flix
@Test
def test01(): Unit \ Assert = Assert.assertEq(expected = 2, 1 + 1)
```

プロジェクトには `resources` ディレクトリを置くこともできます。Flix はその中身をコンパイルしませんが、[アーティファクトのビルド](./building-artifacts.md) で説明するように、ビルドする JAR ファイルにバンドルします。

## コミットすべきもの

Flix は 3 つのディレクトリを生成します。`build` にはクラスファイルと生成されたドキュメントが、`artifact` にはビルドした JAR ファイルやパッケージファイルが、`lib` には Flix がダウンロードした依存関係が入ります。これらのディレクトリは通常バージョン管理に含める*べきではなく*、生成される `.gitignore` はすでにこれらを除外しています：

```
*.fpkg
*.jar
.GITHUB_TOKEN
artifact/
build/
lib/
crash_report_*.txt
```

Flix は `flix.toml` の隣に `packages.lock` ファイルも書き出します。ロックファイルには、各依存関係がダウンロードされた時点でのバージョン情報が記録されており、`.gitignore` がこれを除外していない理由の通り、このファイルは**コミットすべき**ものです。[バージョンとアップグレード](./versions-and-upgrades.md#the-lock-file) を参照してください。

> **警告:** `.GITHUB_TOKEN` ファイルは、Flix が GitHub トークンを探す場所の 1 つです。コミットされたトークンは、そのリポジトリを読み取れる誰にでも使われてしまい、直ちに失効させる必要が生じるため、このファイルは除外されています。

## プッシュのたびにチェックとテストを行う

生成される `.github/workflows/build-and-test.yaml` は、プッシュやプルリクエストのたびにプロジェクトをチェック・テストする GitHub Actions のワークフローです。`flix.toml` の `flix` フィールドから Flix のバージョンを読み取り、そのバージョンの Flix をダウンロードして、`check` に続けて `test` を実行します。

<!--
# Creating a Project

A Flix project is a directory that contains a `flix.toml` manifest, Flix source
code in `src`, and tests in `test`.

We can create such a project with the `init` command. The command creates the
default project structure:

```
.
├── .github
│   └── workflows
│       └── build-and-test.yaml
├── .gitignore
├── flix.toml
├── LICENSE.md
├── README.md
├── src
│   └── Main.flix
└── test
    └── TestMain.flix

4 directories, 7 files
```

The three files we work in are `flix.toml`, `src/Main.flix`, and
`test/TestMain.flix`. The rest is a starting point we are meant to edit: a
`README.md` and a `LICENSE.md` with placeholder text, a `.gitignore`, and a
GitHub Actions workflow.

> **Tip:** The `init` command only creates the files that are not already there.
> It is safe to run in a directory that already has files, and we can run it in an
> existing project to add what is missing.

## The Manifest

The manifest describes the project. The one that `init` writes is minimal:

```toml
[package]
version = "0.1.0"
flix    = "0.76.2"

# repository = "github:<owner>/hello-world"
```

The `version` is the version of the project. The `flix` field is the oldest version
of Flix that can build the project, which `init` sets to the version of Flix we are
running. The `repository` field, which `init` leaves commented out, is the GitHub
repository we publish the project from, as described in
[Publishing a Package](./publishing-a-package.md).

The manifest grows with the project: we add a `[dependencies]` section when we
depend on other packages, as described in
[Using Dependencies](./using-dependencies.md).

> **Note:** Flix requires version numbers to follow [SemVer](https://semver.org/).

> **Note:** A package is named by the repository it is published from, and by
> nothing else.

## Where Flix Looks for Source Code

Flix scans for source files in the paths `*.flix`, `src/**/*.flix`, and
`test/**/*.flix`. We are free to organize the files inside `src` and `test` as we
like: Flix finds them at any depth.

The generated `src/Main.flix` holds the entry point of the program:

```flix
// The main entry point.
def main(): Unit \ IO =
    println("Hello World!")
```

and the generated `test/TestMain.flix` holds a single test:

```flix
@Test
def test01(): Unit \ Assert = Assert.assertEq(expected = 2, 1 + 1)
```

A project may also have a `resources` directory. Flix does not compile what is in
it, but bundles it into the JAR-files we build, as described in
[Building Artifacts](./building-artifacts.md).

## What to Commit

Flix generates three directories: `build` holds class files and generated
documentation, `artifact` holds the JAR- and package-files we build, and `lib`
holds the dependencies Flix has downloaded. These directories should typically
_not_ be checked into version control, and the generated `.gitignore` already
excludes them:

```
*.fpkg
*.jar
.GITHUB_TOKEN
artifact/
build/
lib/
crash_report_*.txt
```

Flix also writes a `packages.lock` file next to `flix.toml`. The lock file records
what every dependency was when it was downloaded, and it *should* be committed,
which is why `.gitignore` does not exclude it. See
[Versions and Upgrades](./versions-and-upgrades.md#the-lock-file).

> **Warning:** A `.GITHUB_TOKEN` file is one of the places Flix looks for a GitHub
> token. It is excluded because a committed token can be used by anyone who can read
> the repository, and would have to be revoked at once.

## Checking and Testing on Every Push

The generated `.github/workflows/build-and-test.yaml` is a GitHub Actions workflow
that checks and tests the project on every push and pull request. It reads the
version of Flix from the `flix` field of `flix.toml`, downloads that version of
Flix, and runs `check` followed by `test`.
-->
