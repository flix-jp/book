# GitHub でパッケージを公開する

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/publish.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/publish.md)ください。

Flix のパッケージは GitHub 上で公開されます。

## パッケージを自動で公開する

以下の手順に従うことで、Flix は自動的にパッケージ化を行い、GitHub 上にアーティファクト(Artifact)を公開できます：

1. マニフェスト(Manifest)である `flix.toml` があることを確認します（なければ `init` で作成します）。
2. `flix.toml` の version フィールドが正しいことを確認します。
3. `flix.toml` の repository フィールドが正しいことを確認します。（例：`repository
   = "github:user/repo"`）
4. 対象のリポジトリの `Contents` に対して読み取り・書き込みのアクセス権を持つ GitHub トークンを持っていることを確認します。
    - 持っていない場合は、GitHub の `Settings > Developer settings >
   Personal access tokens` に移動して、新しいトークンを作成します。
5. `check` と `test` を実行して、すべてが問題ないことを確認します。
6. `release --github-token <TOKEN>` を実行します。次のように表示されるはずです：

```shell
Found `flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
Release github:user/repo v1.2.3? [y/N]: y
Building project...
Publishing new release...

 Successfully released v1.2.3
 https://github.com/user/repo/releases/tag/v1.2.3
```

> **ヒント:** GitHub 上で公開されているパッケージの例としては、[Museum Project](https://github.com/flix/museum) を参照してください。

> **ヒント:** 環境変数 `GITHUB_TOKEN` が利用可能な場合、Flix はそこから GitHub トークンを読み取ります。

> **ヒント:** ファイル `.GITHUB_TOKEN` が利用可能な場合、Flix はそこからも GitHub トークンを読み取ります。

> **注意:** 空の GitHub リポジトリに対してアーティファクトを公開することはできません。

> **警告:** トークンは必ず安全に保管してください！

## パッケージを手動で公開する

以下の手順に従うことで、パッケージを手動で公開することもできます：

1. マニフェストである `flix.toml` があることを確認します（なければ `init` で作成します）。
2. `flix.toml` の version フィールドが正しいことを確認します。
3. `check` と `test` を実行して、すべてが正しいことを確認します。
4. `build-pkg` を実行します。`artifact` ディレクトリにファイルが生成されていることを確認します。
5. GitHub 上のリポジトリに移動します：
    1. "Releases" をクリックします。
    2. "Draft new release" をクリックします。
    3. `v1.2.3` の形式でタグを入力します（つまり SemVer を使用します）。
    4. `artifact` ディレクトリにある `package.fpkg` と `flix.toml` をアップロードします。

> **警告:** パッケージファイル（`foo.fpkg`）とマニフェストファイル（`flix.toml`）の*両方*をアップロードする必要があります。

<!--
# Publishing a Package on GitHub

Flix packages are published on GitHub.

## Automatically Publishing a Package

Flix can automatically package and publish artifacts on GitHub by following these steps:

1. Check that you have a `flix.toml` manifest (if not create one with `init`).
2. Check that the version field in `flix.toml` is correct.
3. Check that the repository field in `flix.toml` is correct. (e.g. `repository
   = "github:user/repo"`)
4. Check that you have a GitHub token which has read and write access to
   `Contents` for the relevant repository.
    - If not go to GitHub and navigate to `Settings > Developer settings >
   Personal access tokens` and create a new token.
5. Run `check` and `test` to ensure that everything looks alright.
6. Run `release --github-token <TOKEN>`. You should see:

```shell
Found `flix.toml'. Checking dependencies...
Resolving Flix dependencies...
Downloading Flix dependencies...
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.
Release github:user/repo v1.2.3? [y/N]: y
Building project...
Publishing new release...

 Successfully released v1.2.3
 https://github.com/user/repo/releases/tag/v1.2.3
```

> **Tip:** See the [Museum Project](https://github.com/flix/museum) for an
> example of a package that has been published on GitHub.

> **Tip:** Flix will read the GitHub token from the environment variable
> `GITHUB_TOKEN`, if available. 

> **Tip:** Flix will also read the GitHub token from the file `.GITHUB_TOKEN`,
> if available. 

> **Note:** You cannot publish artifacts for empty GitHub repositories.

> **Warning:** Be sure to keep your token safe!

## Manually Publishing a Package

A package can also be manually published by following these steps:

1. Check that you have a `flix.toml` manifest (if not create one with `init`).
2. Check that the version field in `flix.toml` is correct.
3. Run `check` and `test` to ensure that everything looks correct.
4. Run `build-pkg`. Check that the `artifact` directory is populated.
5. Go to the repository on GitHub:
    1. Click "Releases".
    2. Click "Draft new release".
    3. Enter a tag of the form `v1.2.3` (i.e. use SemVer).
    4. Upload the `package.fpkg` and `flix.toml` from the `artifact` directory.

> **Warning:** You must upload _both_ the package file  (`foo.fpkg`) and the
> manifest file (`flix.toml`).
-->
