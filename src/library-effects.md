# ライブラリエフェクト

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/library-effects.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/library-effects.md)ください。

Flix は、一般的な I/O 操作のための組み込みライブラリエフェクト(Library effect)をいくつか提供しています。これらのエフェクトはすべてデフォルトハンドラを持っているため、`main` で明示的に `runWithIO` を呼び出す必要はありません。

| エフェクト                                           | 説明                                                                                             |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| [`Assert`](./assert.md)                              | 実行時アサーション（`assertTrue`、`assertEq` など）。ハンドラの設定が可能です。                  |
| [`Logger`](./logger.md)                              | 5 段階の重大度レベルによる構造化ログ。フィルタリングと収集に対応しています。                     |
| [`Math.Random`](./random.md)                         | 擬似乱数の生成。シードによる決定的な動作も選択できます。                                         |
| [`Fs.FileSystem`](./filesystem.md) <br> [`Fs.FileRead`](./filesystem.md) <br> [`Fs.FileWrite`](./filesystem.md) <br> [`Fs.FileStat`](./filesystem.md) | ファイル I/O、メタデータ、ディレクトリ、およびミドルウェア（chroot、アトミックな書き込み、インメモリファイルシステムなど）。 |
| [`Net.Http`](./http-and-https.md) <br> [`Net.Https`](./http-and-https.md) | フルーエント API による HTTP リクエストの送信と、ミドルウェア（リトライ、レート制限、サーキットブレーカー）。 |
| [`Sys.Console`](./console.md)                        | ターミナル I/O：入力の読み取り、stdout/stderr への出力、プロンプト、メニュー。                   |
| [`Sys.Env`](./env.md)                                | 環境変数、システムプロパティ、プラットフォーム情報へのアクセス。                                 |
| [`Sys.Exit`](./exit.md)                              | 指定した終了コードによるプログラムの終了。                                                       |
| [`Sys.Process`](./process.md)                        | OS プロセスの起動と管理。                                                                        |
| [`Time.Clock`](./clock.md)                           | さまざまな単位での現在時刻（実時間）の取得。                                                     |
| [`Time.Sleep`](./sleep.md)                           | 現在のスレッドの一時停止。合成可能なミドルウェア（ジッター、上限、ログ出力）に対応しています。   |

<!--
# Library Effects

Flix provides several built-in library effects for common I/O operations. These
effects all have default handlers, so no explicit `runWithIO` is needed in
`main`.

| Effect                                              | Description                                                                                    |
|------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [`Assert`](./assert.md)                              | Runtime assertions (`assertTrue`, `assertEq`, etc.) with configurable handlers.                |
| [`Logger`](./logger.md)                              | Structured logging at five severity levels with filtering and collection.                      |
| [`Math.Random`](./random.md)                         | Generating pseudorandom numbers, with optional seeded determinism.                             |
| [`Fs.FileSystem`](./filesystem.md) <br> [`Fs.FileRead`](./filesystem.md) <br> [`Fs.FileWrite`](./filesystem.md) <br> [`Fs.FileStat`](./filesystem.md) | File I/O, metadata, directories, and middleware (chroot, atomic writes, in-memory FS, etc.).   |
| [`Net.Http`](./http-and-https.md) <br> [`Net.Https`](./http-and-https.md) | Sending HTTP requests with a fluent API, middleware (retries, rate limiting, circuit breakers). |
| [`Sys.Console`](./console.md)                        | Terminal I/O: reading input, printing to stdout/stderr, prompts, and menus.                    |
| [`Sys.Env`](./env.md)                                | Accessing environment variables, system properties, and platform information.                  |
| [`Sys.Exit`](./exit.md)                              | Terminating the program with a specific exit code.                                             |
| [`Sys.Process`](./process.md)                        | Spawning and managing OS processes.                                                            |
| [`Time.Clock`](./clock.md)                           | Querying the current wall-clock time in various units.                                         |
| [`Time.Sleep`](./sleep.md)                           | Pausing the current thread with composable middleware (jitter, caps, logging).                  |
-->
