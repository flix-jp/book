# エフェクトのロック

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/locking-effects.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/locking-effects.md)ください。

> **注意:** エフェクトのロック(Effect locking)は実験的な機能です。

`plain` または `paranoid` の security context でビルドされた依存関係は、[依存関係の信頼](./trusting-dependencies.md)で説明したとおり、その型とエフェクトのシグネチャが示すことしか行えません。しかし、シグネチャは固定されたものではありません。パッケージの新しいバージョンが、私たちが呼び出している関数にエフェクトを追加するかもしれません。そのような変更を検出するには、アップグレードのたびに、すべての依存関係のすべての公開シグネチャをもう一度読み直す必要があります。

エフェクトのロックは、これを私たちの代わりに行ってくれます。`eff-lock` コマンドは、プロジェクトが宣言しているすべての依存関係における、すべての公開関数の型とエフェクトのシグネチャを `effects.lock` ファイルに記録し、`eff-check` コマンドは、それらのうちどれが変更されたかを教えてくれます。

## シグネチャのロック

次のように依存関係を持つプロジェクトの場合：

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "2.0.2", mount = "giftshop" }
"github:flix/museum-clerk"    = { version = "2.1.2", mount = "clerk" }
```

`eff-lock` コマンドはプロジェクトを型検査し、次のように報告します：

```
Locked 3 signatures of 2 packages.
```

このコマンドは、`flix.toml` の隣に `effects.lock` ファイルを書き出します：

```toml
[lock]
version = 1

[packages."github:flix/museum-clerk".defs]
"Clerk.welcomeVisitor" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"
"Clerk.work" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"

[packages."github:flix/museum-giftshop".defs]
"Giftshop.buyGift" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"
```

ロックファイルには、パッケージごとに1つのセクションがあり、その中には、パッケージが宣言している公開関数ごとに、そのパッケージ内での名前と、その関数の型・エフェクトのシグネチャのハッシュが記された行が並びます。トレイトが宣言するシグネチャは、専用の `sigs` テーブルに記録されます。
ここに登場する3つの関数はすべて `Unit -> Unit \ IO` というシグネチャを持つため、すべて同じハッシュになっています。

このロックが対象とするのは、マニフェストが宣言している依存関係であり、それらがさらに依存しているパッケージは対象外です。宣言していないパッケージは、呼び出すこともできないパッケージだからです。

> **ヒント:** `effects.lock` ファイルは、`packages.lock` と同様にバージョン管理にコミットすべきです。

## シグネチャの確認

`eff-check` コマンドはプロジェクトを型検査し、ロックされているすべての関数のシグネチャを、ロックファイル内のハッシュと比較します。何も変わっていない場合は、次のように報告します：

```
Checked 3 signatures of 2 packages. Nothing has changed.
```

私たちがロックした時点の `flix/museum-giftshop` のリリースでは `buyGift` が純粋関数として宣言されていたが、アップグレードした先のリリースでは `IO` エフェクトを伴って宣言されているとしましょう。この場合、`eff-check` コマンドは次のように報告します：

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@  WARNING! YOU MAY BE SUBJECT TO A SUPPLY CHAIN ATTACK!  @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
            ~~ Signatures have changed! ~~

These declarations are not the ones that were locked. A dependency may now
be able to do things it could not do when you locked it.

  github:flix/museum-giftshop:
    Giftshop.buyGift
      def buyGift(): Unit \ IO

Do not build or run this project until you know why. Find out who published
the version you are building, and what changed in it.

Record the new signatures with flix eff-lock only once you are certain the
change is one the author made, and one you want.
```

シグネチャの変更は、必ずしも攻撃を意味するわけではありません。作者が意図的にその変更をリリースし、私たちもそれを望んでいる場合もあります。しかし、それが気づかれずに通り過ぎることがあってはなりません。シグネチャがなぜ変わったのかを把握し、その変更を受け入れたなら、もう一度 `eff-lock` を実行して新しいシグネチャを記録します。

> **ヒント:** `eff-check` コマンドは、シグネチャが変更されている場合は終了コード `1` で、そうでない場合は `0` で終了するため、CI ビルドを失敗させるために使えます。

## パッケージを1つずつロックする

どちらのコマンドも、パッケージを指定できます：

```
eff-lock flix/museum-giftshop
```

このコマンドは次のように報告します：

```
Locked 1 signature of 'github:flix/museum-giftshop'.
```

そして、ロックファイル内の `flix/museum-giftshop` のセクションだけを置き換え、他のすべてのセクションはそのままにします。同様に、`eff-check flix/museum-giftshop` はそのパッケージだけを確認します。これにより、あるパッケージの変更は受け入れて記録しつつ、別のパッケージの変更はまだレビュー中のまま、という状態にできます。

## 2つのロックファイル

プロジェクトには2つのロックファイルがあり、それぞれ異なるものから守ってくれます。

`packages.lock` ファイルは、プロジェクトが解決した各バージョンのファイルのダイジェストを記録するため、一度見たバージョンは常に同じバイト列であることが保証されます。アップグレードは新しいバージョンを解決してそれを記録します。それがアップグレードの役目だからです。`effects.lock` ファイルは、依存関係が何をし得るかを記録するものであり、`eff-lock` だけがこれを書き込むため、アップグレードを経ても生き残ります。前者は、私たちがダウンロードしたものとはもはや異なるファイルを持つバージョンを検出します。後者は、私たちが受け入れたものより多くのことを行うバージョンを検出します。

## ロックと security context

エフェクトのロックは、security context が与える保証に依存しています。シグネチャが関数の行い得ることを表すのは、そのパッケージが `plain` または `paranoid` context でビルドされている場合に限られます。`unrestricted` context でビルドされたパッケージも他と同様にロックされますが、そのシグネチャは、そのパッケージが何を行い得るかについて何も教えてくれないため、ロックが変化していないからといって安心はできません。

ロックが最も有益な情報をもたらすのは、[エフェクトを必要とするライブラリの書き方](./trusting-dependencies.md#writing-a-library-that-needs-effects)で推奨しているように、ライブラリが自身のエフェクトを宣言している場合です。`IO` ではなく、使用しているエフェクトを名指しするシグネチャは、行っていることが変化したときにシグネチャ自体も変化します。

<!--
# Locking Effects

> **Note:** Effect locking is an experimental feature.

A dependency built in the `plain` or `paranoid` security context can only do what its
type and effect signatures say it does, as described in
[Trusting Dependencies](./trusting-dependencies.md). But signatures are not fixed. A
new version of a package may add an effect to a function we call. To catch such a
change, we would have to read every public signature of every dependency again after
every upgrade.

Effect locking does this for us. The `eff-lock` command records the type and effect
signature of every public function of every dependency the project declares in an
`effects.lock` file, and the `eff-check` command tells us which of them have changed
since.

## Locking the Signatures

For a project that depends on:

```toml
[dependencies]
"github:flix/museum-giftshop" = { version = "2.0.2", mount = "giftshop" }
"github:flix/museum-clerk"    = { version = "2.1.2", mount = "clerk" }
```

the `eff-lock` command type checks the project and reports:

```
Locked 3 signatures of 2 packages.
```

It writes an `effects.lock` file next to `flix.toml`:

```toml
[lock]
version = 1

[packages."github:flix/museum-clerk".defs]
"Clerk.welcomeVisitor" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"
"Clerk.work" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"

[packages."github:flix/museum-giftshop".defs]
"Giftshop.buyGift" = "sha256:527d67f451945e784feca592a8e17b91f2a5152039171ce326efbdd200a01015"
```

The lock file has a section for each package, and in it a line for each public
function the package declares, by its name inside the package, with a hash of the
function's type and effect signature. The signatures that a trait declares go in a
`sigs` table of their own.
The three functions here all have the signature `Unit -> Unit \ IO`, which is why
they all have the same hash.

The lock covers the dependencies the manifest declares, and not the packages they
depend on: a package we do not declare is one we cannot call.

> **Tip:** The `effects.lock` file should be committed to version control, like
> `packages.lock`.

## Checking the Signatures

The `eff-check` command type checks the project and compares the signature of every
locked function with the hash in the lock file. When nothing has changed, it reports:

```
Checked 3 signatures of 2 packages. Nothing has changed.
```

Suppose that the release of `flix/museum-giftshop` we locked declared `buyGift` as a
pure function, and that we upgrade to a release that declares it with the `IO`
effect. The `eff-check` command then reports:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@  WARNING! YOU MAY BE SUBJECT TO A SUPPLY CHAIN ATTACK!  @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
            ~~ Signatures have changed! ~~

These declarations are not the ones that were locked. A dependency may now
be able to do things it could not do when you locked it.

  github:flix/museum-giftshop:
    Giftshop.buyGift
      def buyGift(): Unit \ IO

Do not build or run this project until you know why. Find out who published
the version you are building, and what changed in it.

Record the new signatures with flix eff-lock only once you are certain the
change is one the author made, and one you want.
```

A changed signature is not necessarily an attack. The author may have released the
change on purpose, and we may want it. But it should never pass unnoticed. Once we
know why a signature changed, and accept the change, we run `eff-lock` again to record
the new signatures.

> **Tip:** The `eff-check` command exits with status `1` when a signature has
> changed, and with `0` otherwise, so it can fail a CI build.

## Locking One Package at a Time

Both commands can name a package:

```
eff-lock flix/museum-giftshop
```

The command reports:

```
Locked 1 signature of 'github:flix/museum-giftshop'.
```

and replaces the section of `flix/museum-giftshop` in the lock file, leaving every
other section as it was. Likewise, `eff-check flix/museum-giftshop` checks that
package alone. This lets us accept the change in one package, and record it, while the
change in another is still under review.

## The Two Lock Files

A project has two lock files, and they guard against different things.

The `packages.lock` file records the digest of the files of each version the project
has resolved, so that a version, once seen, is always the same bytes. An upgrade
resolves new versions and records them: that is what an upgrade is for. The
`effects.lock` file records what the dependencies may do, and only `eff-lock` writes
it, so it survives an upgrade. The first catches a version whose files are no longer
the ones we downloaded. The second catches a version that does more than the one we
took on.

## Locks and Security Contexts

Effect locking rests on the guarantee of the security context. A signature says what
a function can do only when its package is built in the `plain` or `paranoid`
context. A package built in the `unrestricted` context is locked like any other, but
its signatures tell us nothing about what it may do, so an unchanged lock on it is no
assurance.

The lock is most informative when a library declares its own effects, as recommended
in [Writing a Library that Needs Effects](./trusting-dependencies.md#writing-a-library-that-needs-effects).
A function whose signature names the effects it uses, rather than `IO`, has a
signature that changes when what it does changes.
-->
