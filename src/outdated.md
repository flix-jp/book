# 古くなったパッケージの確認

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/outdated.html)を参照してください。

`outdated` コマンドを使うと、Flix パッケージに利用可能な更新があるかどうかを確認できます。

例えば、`flix.toml` に以下の依存関係があるとします：

```toml
[dependencies]
"github:flix/museum"            = "1.0.0"
"github:flix/museum-giftshop"   = "1.0.0"
```

このとき `outdated` コマンドを実行すると、次のような出力が得られます：

```shell
Found `flix.toml`. Checking dependencies...
Resolving Flix dependencies...
  Cached `flix/museum.toml` (v1.0.0).
  Cached `flix/museum-giftshop.toml` (v1.0.0).  
  Cached `flix/museum-entrance.toml` (v1.0.0).  
  Cached `flix/museum-giftshop.toml` (v1.0.0).  
  Cached `flix/museum-restaurant.toml` (v1.0.0).
  Cached `flix/museum-clerk.toml` (v1.0.0).     
  Cached `flix/museum-clerk.toml` (v1.0.0).
Downloading Flix dependencies...
  Cached `flix/museum.fpkg` (v1.0.0).
  Cached `flix/museum-giftshop.fpkg` (v1.0.0).
  Cached `flix/museum-entrance.fpkg` (v1.0.0).
  Cached `flix/museum-giftshop.fpkg` (v1.0.0).
  Cached `flix/museum-restaurant.fpkg` (v1.0.0).
  Cached `flix/museum-clerk.fpkg` (v1.0.0).
  Cached `flix/museum-clerk.fpkg` (v1.0.0).
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.

package                 current    major    minor    patch
flix/museum             1.0.0               1.4.0         
flix/museum-giftshop    1.0.0               1.1.0         
```

`outdated` コマンドの出力から、使用中の 2 つのパッケージに更新が利用可能であることが分かります：

- `flix/museum` は `1.0.0` から `1.4.0` にアップグレードできます。
- `flix/museum-giftshop` は `1.0.0` から `1.1.0` にアップグレードできます。

パッケージをアップグレードしたい場合は、`flix.toml` を手動で変更する必要があります。

<!--
# Finding Outdated Packages

We can use the `outdated` command to check if any Flix packages have updates
available. 

For example, if we have the following dependency in `flix.toml`:

```toml
[dependencies]
"github:flix/museum"            = "1.0.0"
"github:flix/museum-giftshop"   = "1.0.0"
```

then we can run `outdated` command which will produce:

```shell
Found `flix.toml`. Checking dependencies...
Resolving Flix dependencies...
  Cached `flix/museum.toml` (v1.0.0).
  Cached `flix/museum-giftshop.toml` (v1.0.0).  
  Cached `flix/museum-entrance.toml` (v1.0.0).  
  Cached `flix/museum-giftshop.toml` (v1.0.0).  
  Cached `flix/museum-restaurant.toml` (v1.0.0).
  Cached `flix/museum-clerk.toml` (v1.0.0).     
  Cached `flix/museum-clerk.toml` (v1.0.0).
Downloading Flix dependencies...
  Cached `flix/museum.fpkg` (v1.0.0).
  Cached `flix/museum-giftshop.fpkg` (v1.0.0).
  Cached `flix/museum-entrance.fpkg` (v1.0.0).
  Cached `flix/museum-giftshop.fpkg` (v1.0.0).
  Cached `flix/museum-restaurant.fpkg` (v1.0.0).
  Cached `flix/museum-clerk.fpkg` (v1.0.0).
  Cached `flix/museum-clerk.fpkg` (v1.0.0).
Resolving Maven dependencies...
  Running Maven dependency resolver.
Downloading external jar dependencies...
Dependency resolution completed.

package                 current    major    minor    patch
flix/museum             1.0.0               1.4.0         
flix/museum-giftshop    1.0.0               1.1.0         
```

The `outdated` command tells us that we are using two packages which have
updates available:

- `flix/museum` can be upgraded from `1.0.0` to `1.4.0`.
- `flix/museum-giftshop` can be upgraded from `1.0.0` to `1.1.0`.

If we want to upgrade a package, we must manually modify `flix.toml`.
-->
