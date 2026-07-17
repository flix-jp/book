# FileSystem

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/filesystem.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/filesystem.md)ください。

Flix は、ファイルシステム操作のための一連のエフェクトを提供しています。主要なモジュールは次のとおりです：

- `Fs.FileSystem` — 統合された `FileSystem` エフェクト（全 29 操作）
- `Fs.FileRead` — `FileRead` エフェクト（read、readLines、readBytes）
- `Fs.FileWrite` — `FileWrite` エフェクト（write、append、delete、copy、move、mkdir など）
- `Fs.FileStat` — `FileStat` エフェクト（存在確認、種別判定、パーミッション、タイムスタンプ、サイズ）
- `Fs.DirList` — `DirList` エフェクト（ディレクトリ内容の一覧取得）
- `Fs.Glob` — `Glob` エフェクト（パターンによるファイル検索）
- `Fs.Size` — ファイルサイズを扱うためのユーティリティ

これらのエフェクトはすべてデフォルトハンドラを持つため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。

さらに細粒度のリーフエフェクト(Leaf effect)（例：`FileExists`、`ReadFile`、`WriteFile`）も存在します。これらはデフォルトハンドラを持ちませんが、`runWith` ハンドラを使って親エフェクトへと実行を委ねることができます。詳細は[エフェクト階層](#the-effect-hierarchy)を参照してください。

## ファイルの読み込み

`FileRead.read` を使うと、ファイル全体を文字列として読み込むことができます：

```flix
use Fs.FileRead

def main(): Unit \ { FileRead, IO } =
    match FileRead.read("example.txt") {
        case Ok(content) => println(content)
        case Err(err)    => println("Error: ${err}")
    }
```

すべてのファイルシステム操作は `Result[IoError, ...]` を返します。`IoError` 型は `ErrorKind` とメッセージ文字列のペアです。`ErrorKind` enum は、何が問題だったのかを教えてくれます：

| ErrorKind               | 説明                                             |
|--------------------------|--------------------------------------------------|
| `NotFound`               | ファイルまたはディレクトリが見つかりませんでした。 |
| `AlreadyExists`          | ファイルまたはディレクトリがすでに存在します。     |
| `PermissionDenied`       | アクセスが拒否されました（ミドルウェアでも使用されます）。 |
| `InvalidPath`            | パスの形式が不正です。                             |
| ...                      | その他。                                          |

> **注意:** シグネチャに `IO` エフェクトが現れるのは、`println` を使っているためです。

## ファイルの書き込み

`FileWrite.write` を使うと、文字列をファイルに書き込むことができます：

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.write(str = "Hello, Flix!", "greeting.txt") {
        case Ok(_)    => println("File written successfully.")
        case Err(err) => println("Error: ${err}")
    }
```

## 行単位の読み書き

`readLines` と `writeLines` を使うと、ファイルを行単位で扱うことができます：

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    match FileWrite.writeLines(lines = List#{"Line 1", "Line 2", "Line 3"}, "data.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileRead.readLines("data.txt") {
                case Ok(lines) =>
                    foreach (line <- lines) {
                        println(line)
                    }
                case Err(err) => println("Read error: ${err}")
            }
    }
```

> **注意:** 読み込みと書き込みの両方を行うため、エフェクト集合には `FileRead`、`FileWrite`、`IO` が含まれます。

## バイト単位の読み書き

バイナリデータには `readBytes` と `writeBytes` を使うことができます：

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    let data = Vector#{72i8, 101i8, 108i8, 108i8, 111i8};
    match FileWrite.writeBytes(data, "binary.dat") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileRead.readBytes("binary.dat") {
                case Ok(bytes) =>
                    println("Read ${Vector.length(bytes)} bytes.");
                    println("As string: ${String.fromBytes(bytes)}")
                case Err(err) => println("Read error: ${err}")
            }
    }
```

## ファイルへの追記

`append` を使うと、既存のファイルを上書きせずにテキストを追記できます。ファイルが存在しない場合は新規に作成されます：

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    match FileWrite.write(str = "Line 1\n", "log.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileWrite.append(str = "Line 2\n", "log.txt") {
                case Err(err) => println("Append error: ${err}")
                case Ok(_) =>
                    match FileRead.read("log.txt") {
                        case Ok(content) => println(content)
                        case Err(err)    => println("Read error: ${err}")
                    }
            }
    }
```

`appendLines` や `appendBytes` といったバリエーションもあります。

## ディレクトリの一覧取得

`DirList.list` を使うと、ディレクトリ内のすべてのファイルとディレクトリの名前を取得できます：

```flix
use Fs.DirList

def main(): Unit \ { DirList, IO } =
    match DirList.list(".") {
        case Ok(entries) =>
            foreach (entry <- entries) {
                println(entry)
            }
        case Err(err) => println("Error: ${err}")
    }
```

## Glob によるファイル検索

`Glob.glob` を使うと、基点となるディレクトリ以下からグロブパターンにマッチするファイルを検索できます：

```flix
use Fs.Glob

def main(): Unit \ { Glob, IO } =
    match Glob.glob(".", "*.flix") {
        case Ok(files) =>
            foreach (file <- files) {
                println(file)
            }
        case Err(err) => println("Error: ${err}")
    }
```

## ファイルのメタデータ

`FileStat` エフェクトを使うと、ファイルのメタデータ（存在の有無、種別、サイズ、パーミッション、タイムスタンプ）を調べることができます：

```flix
use Fs.FileStat
use Fs.FileWrite

def main(): Unit \ { FileStat, FileWrite, IO } =
    let file = "example.txt";
    match FileWrite.write(str = "Hello!", file) {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileStat.exists(file) {
                case Ok(b)    => println("Exists: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.isRegularFile(file) {
                case Ok(b)    => println("Is regular file: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.isDirectory(file) {
                case Ok(b)    => println("Is directory: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.size(file) {
                case Ok(s)    => println("Size: ${s}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.modificationTime(file) {
                case Ok(t)    => println("Modification time: ${t}ms")
                case Err(err) => println("Error: ${err}")
            }
    }
```

`FileStat` エフェクトは、4 つのサブエフェクトを組み合わせたものです：

| サブエフェクト   | 操作                                                   |
|------------------|--------------------------------------------------------|
| `FileTest`       | `exists`、`isDirectory`、`isRegularFile`、`isSymbolicLink` |
| `FilePermission` | `isReadable`、`isWritable`、`isExecutable`             |
| `FileTime`       | `accessTime`、`creationTime`、`modificationTime`       |
| `FileSize`       | `size`                                                 |

## コピー・移動・削除

`FileWrite` エフェクトを使って、ファイルのコピー、移動、削除を行うこともできます：

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.write(str = "Hello!", "original.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            // オプションなしでコピー。
            match FileWrite.copy(src = "original.txt", "copy.txt") {
                case Ok(_)    => println("Copied.")
                case Err(err) => println("Copy error: ${err}")
            };
            // オプションなしで移動（リネーム）。
            match FileWrite.move(src = "copy.txt", "renamed.txt") {
                case Ok(_)    => println("Moved.")
                case Err(err) => println("Move error: ${err}")
            };
            // 削除。
            match FileWrite.delete("renamed.txt") {
                case Ok(_)    => println("Deleted.")
                case Err(err) => println("Delete error: ${err}")
            }
    }
```

`copy` と `move` は、オプション集合を受け取る `copyWith` と `moveWith` を簡便に使うためのラッパーです：

- `CopyOption.CopyAttributes` — ファイル属性を保持します
- `CopyOption.ReplaceExisting` — コピー先が存在する場合は上書きします
- `MoveOption.AtomicMove` — アトミックなリネームを行います
- `MoveOption.ReplaceExisting` — 移動先が存在する場合は上書きします

## ディレクトリの作成

単一のディレクトリを作成するには `mkDir` を、ディレクトリとその親ディレクトリすべてを作成するには `mkDirs` を、一時ディレクトリを作成するには `mkTempDir` を使うことができます：

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.mkDirs("a/b/c") {
        case Ok(_)    => println("Created a/b/c.")
        case Err(err) => println("Error: ${err}")
    };
    match FileWrite.mkTempDir("flix-") {
        case Ok(path) => println("Temp dir: ${path}")
        case Err(err) => println("Error: ${err}")
    }
```

## FileSystem エフェクト

`FileSystem` エフェクトは、すべてのファイルシステム操作を単一のエフェクトにまとめたものです。`FileStat`、`FileRead`、`FileWrite`、`DirList`、`Glob` のすべての操作を含みます。複数のカテゴリの操作をまとめて使いたい場合には `FileSystem` を使うとよいでしょう：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.write(str = "Hello!", "greeting.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileSystem.read("greeting.txt") {
                case Ok(content) => println("Read: ${content}")
                case Err(err)    => println("Read error: ${err}")
            }
    }
```

## ミドルウェア

ミドルウェア(Middleware)とは、ファイルシステム操作をインターセプトするエフェクトハンドラのことです。`run { ... } with FileSystem.<middleware>`（または対応するサブエフェクトのモジュール）を使って適用し、複数の `with` 節を積み重ねることで合成できます。

### ベースディレクトリ

`withBaseDir` は、相対パスをベースディレクトリを基準に解決します。絶対パスはそのまま通過します：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.mkDirs("/tmp/flix-basedir") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "Hello", "greeting.txt") {
                    case Err(err) => println("Write error: ${err}")
                    case Ok(_) =>
                        match FileSystem.read("greeting.txt") {
                            case Ok(content) => println("Read: ${content}")
                            case Err(err)    => println("Read error: ${err}")
                        }
                }
            } with FileSystem.withBaseDir("/tmp/flix-basedir")
    }
```

### Chroot

`withChroot` は、すべての操作をディレクトリのサブツリー内に制限します。chroot の外側のパスを対象とする操作は `PermissionDenied` エラーで失敗します：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.mkDirs("/tmp/flix-chroot") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "Hello", "/tmp/flix-chroot/data.txt") {
                    case Ok(_)    => println("Write inside chroot succeeded")
                    case Err(err) => println("Error: ${err}")
                };
                match FileSystem.read("/etc/hostname") {
                    case Ok(_)    => println("Unexpected: read outside chroot succeeded")
                    case Err(err) => println("Read outside chroot blocked: ${err}")
                }
            } with FileSystem.withChroot("/tmp/flix-chroot")
    }
```

### ロギング

`withLogging` は、各ファイルシステム操作を `Logger` エフェクト経由でログに記録します。`main` の型シグネチャに `Logger` が現れる点に注意してください：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "Hello, Flix!", "greeting.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.read("greeting.txt") {
                    case Ok(content) => println(content)
                    case Err(err)    => println("Read error: ${err}")
                }
        }
    } with FileSystem.withLogging
```

### 読み取り専用

`withReadOnly` は、すべての書き込み操作を `PermissionDenied` エラーでブロックします。読み込みおよびメタデータ取得（stat）の操作は通常どおり通過します：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "This will fail", "blocked.txt") {
            case Ok(_)    => println("Unexpected: write succeeded")
            case Err(err) => println("Write blocked: ${err}")
        };
        match FileSystem.exists("blocked.txt") {
            case Ok(b)    => println("Exists: ${b}")
            case Err(err) => println("Error: ${err}")
        }
    } with FileSystem.withReadOnly
```

### ドライラン

`withDryRun` は、書き込み操作を実際には実行せず、`Logger` エフェクト経由でログに記録します。読み込み操作は通常どおり実行されます：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "This won't be written", "phantom.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.exists("phantom.txt") {
                    case Ok(b)    => println("Exists: ${b}")
                    case Err(err) => println("Error: ${err}")
                }
        }
    } with FileSystem.withDryRun
```

### アトミック書き込み

`withAtomicWrite` は、まずデータを一時ファイルに書き込み、その後アトミックなリネームによって目的の場所へ配置します。これにより、失敗時に書き込みが中途半端な状態になることを防ぎます。影響を受けるのは `write`、`writeLines`、`writeBytes` のみで、追記やその他の操作はそのまま通過します：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Atomic content", "output.txt") {
            case Ok(_)    => println("Atomic write succeeded.")
            case Err(err) => println("Write error: ${err}")
        }
    } with FileSystem.withAtomicWrite
```

### バックアップ

`withBackup` は、既存のファイルを上書きする前にバックアップコピーを作成します。破壊的な操作（`write`、`writeLines`、`writeBytes`、`truncate`、`delete`、`copyWith`、`moveWith`）を行うたびに、既存のファイルが `file + suffix` へコピーされます：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.write(str = "Original content", "data.txt") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "New content", "data.txt") {
                    case Ok(_)    => println("Write succeeded; backup saved to data.txt.bak")
                    case Err(err) => println("Write error: ${err}")
                }
            } with FileSystem.withBackup(".bak")
    }
```

### 親ディレクトリの自動作成

`withMkParentDirs` は、書き込みおよび追記の操作を行う前に、親ディレクトリを自動的に作成します。親ディレクトリがすでに存在する場合は何もしません：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Hello", "deep/nested/path/greeting.txt") {
            case Ok(_)    => println("Write succeeded (parents created).")
            case Err(err) => println("Write error: ${err}")
        }
    } with FileSystem.withMkParentDirs
```

### 競合チェック

`withConflictCheck` は、ファイルの変更時刻を追跡し、前回の操作以降にファイルが外部から変更されていた場合に書き込みを拒否します。これにより、外部プロセスとの書き込み同士の競合を検出できます：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "First write", "shared.txt") {
            case Err(err) => println("Error: ${err}")
            case Ok(_) =>
                match FileSystem.write(str = "Second write", "shared.txt") {
                    case Ok(_)    => println("No conflict detected.")
                    case Err(err) => println("Conflict: ${err}")
                }
        }
    } with FileSystem.withConflictCheck
```

### 転送量の制限

`withTransferLimit` は、ペイロードが最大サイズを超える読み込みまたは書き込みの操作を拒否します：

```flix
use Fs.FileSystem
use Fs.Size

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Small", "ok.txt") {
            case Ok(_)    => println("Small write succeeded.")
            case Err(err) => println("Error: ${err}")
        }
    } with FileSystem.withTransferLimit(Size.megaBytes(10))
```

### アクセス制御

Flix は、アクセス可能なパスを制限するためのミドルウェアを提供しています。次のものが利用できます：

- `withAllowList(dirs)` — 列挙したディレクトリ内のパスのみを許可します
- `withDenyList(dirs)` — 列挙したディレクトリ内のパスをブロックします
- `withAllowGlob(patterns)` — 少なくとも 1 つのパターンにマッチするパスのみを許可します
- `withDenyGlob(patterns)` — いずれかのパターンにマッチするパスをブロックします

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.read("/tmp/safe/data.txt") {
            case Ok(content) => println(content)
            case Err(err)    => println("Error: ${err}")
        }
    } with FileSystem.withAllowList(Nel.of("/tmp/safe"))
```

### インメモリファイルシステム

`withInMemoryFS` ハンドラは、実際のファイルシステムを完全にメモリ上の実装で置き換えます。ファイルシステムは空の状態から始まり、書き込まれていないファイルを読み込むと `NotFound` が返ります。実際のファイルシステムへのアクセスは一切発生しません：

```flix
use Fs.FileSystem
use Time.Clock

def main(): Unit \ { Clock, IO } =
    run {
        let result = forM (
            _       <- FileSystem.mkDirs("/data");
            _       <- FileSystem.write(str = "Hello", "/data/hello.txt");
            _       <- FileSystem.write(str = "World", "/data/world.txt");
            entries <- FileSystem.list("/data");
            content <- FileSystem.read("/data/hello.txt");
            _       <- FileSystem.delete("/data/hello.txt");
            exists  <- FileSystem.exists("/data/hello.txt")
        ) yield (entries, content, exists);
        match result {
            case Err(err) => println("Error: ${err}")
            case Ok((entries, content, exists)) =>
                println("Files in /data:");
                foreach (entry <- entries) {
                    println("  ${entry}")
                };
                println("Content: ${content}");
                println("Exists after delete: ${exists}")
        }
    } with FileSystem.withInMemoryFS
```

`withInMemoryFS` は（ファイルのタイムスタンプのために）`Clock` エフェクトを必要としますが、`FileSystem` を完全に処理するため、エフェクトシグネチャからは `FileSystem` が取り除かれる点に注意してください。

### メモリオーバーレイ

`withMemoryOverlay` ハンドラは、実際のファイルシステムの上に、メモリ上の書き込み可能なストアを重ねます。書き込みはメモリ上に記録され、以降の読み込みでは書き込まれたデータが見えますが、実際のファイルシステムが変更されることはありません。オーバーレイに存在しないファイルの読み込みは、実際のファイルシステムへとフォールスルーします：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        // この書き込みはメモリ上に記録され、ディスクには書き込まれません。
        match FileSystem.write(str = "In-memory only", "virtual.txt") {
            case Err(err) => println("Error: ${err}")
            case Ok(_) =>
                match FileSystem.read("virtual.txt") {
                    case Ok(content) => println("Read from overlay: ${content}")
                    case Err(err)    => println("Error: ${err}")
                }
        }
    } with FileSystem.withMemoryOverlay
```

## ミドルウェアの合成

ミドルウェアは、`with` 節を積み重ねることで合成できます。最も内側のハンドラ（最初に書かれたもの）が元の操作をインターセプトし、続いてひとつ外側のハンドラへと委譲します。次の例では、ベースディレクトリ、親ディレクトリの自動作成、バックアップ、アトミック書き込み、競合チェック、ロギングを積み重ねています：

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "Hello, Flix!", "data/greeting.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.read("data/greeting.txt") {
                    case Ok(content) => println("Read: ${content}")
                    case Err(err)    => println("Read error: ${err}")
                }
        }
    } with FileSystem.withBaseDir("/tmp/flix-example")
      with FileSystem.withMkParentDirs
      with FileSystem.withConflictCheck
      with FileSystem.withBackup(".bak")
      with FileSystem.withAtomicWrite
      with FileSystem.withLogging
```

`FileSystem` と `Logger` の両エフェクトはデフォルトハンドラを持つため、`main` の型シグネチャに現れると自動的に処理されます。

> **注意:** `with` 節の順序は重要です。最も外側のハンドラ（最後に書かれたもの）は、すべての内側のハンドラを包み込みます。上の例では `withLogging` が最も外側にあるため、競合チェックによるリトライやアトミック書き込みの一時ファイルも含めて、*すべての*ファイルシステム操作を観測します。ミドルウェアを合成する際には、どの層がどの操作を観測すべきかを考えてください。

## ミドルウェア一覧

次の表は、どのミドルウェアがどのエフェクトで利用できるかを示しています（表全体を見るには右へスクロールしてください）：

| ミドルウェア           | FileTest | FilePermission | FileTime | FileStat | FileRead | DirList | Glob | FileWrite | FileSystem |
|------------------------|:--------:|:--------------:|:--------:|:--------:|:--------:|:-------:|:----:|:---------:|:----------:|
| `withLogging`          | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withBaseDir`          | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withChroot`           | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withAllowList`        | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withDenyList`         | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withAllowGlob`        | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withDenyGlob`         | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withFollowLinks`      | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withTransferLimit`    |          |                |          |          | x        |         |      | x         | x          |
| `withChecksum`         |          |                |          |          | x        |         |      | x         | x          |
| `withDryRun`           |          |                |          |          |          |         |      | x         | x          |
| `withReadOnly`         |          |                |          |          |          |         |      | x         | x          |
| `withAtomicWrite`      |          |                |          |          |          |         |      | x         | x          |
| `withBackup`           |          |                |          |          |          |         |      | x         | x          |
| `withConflictCheck`    |          |                |          |          |          |         |      | x         | x          |
| `withMkParentDirs`     |          |                |          |          |          |         |      | x         | x          |
| `withSizeRotation`     |          |                |          |          |          |         |      | x         | x          |
| `withMemoryOverlay`    |          |                |          |          |          |         |      |           | x          |
| `withInMemoryFS`       |          |                |          |          |          |         |      |           | x          |

## エフェクト階層

Flix のファイルシステムエフェクトは、エフェクト階層(Effect hierarchy)を成しています。最上位には全 29 操作を備えた `FileSystem` があります。その下には関連する操作をグループ化した中間のエフェクトがあり、最下層には個々の操作に対応するリーフエフェクトがあります：

```text
FileSystem                          (29 ops — unified root)
├── FileStat                        (11 ops)
│   ├── FileTest                    (4 ops: exists, isDirectory, isRegularFile, isSymbolicLink)
│   ├── FilePermission              (3 ops: isReadable, isWritable, isExecutable)
│   ├── FileTime                    (3 ops: accessTime, creationTime, modificationTime)
│   └── FileSize                    (1 op: size)
├── FileRead                        (3 ops: read, readLines, readBytes)
├── DirList                         (1 op: list)
├── Glob                            (1 op: glob)
└── FileWrite                       (13 ops: write, append, delete, copy, move, mkdir, etc.)
```

階層のどのレベルでも利用できます。例えば、`exists` だけが必要なら `FileExists` のようなリーフエフェクトを、ファイルの読み込みが必要なら `FileRead` を、すべてが必要なら `FileSystem` を使うことができます。

リーフエフェクトは、`runWith` ハンドラを使って親エフェクトへと実行を委ねることができます。例えば、`FileExists` を `FileTest` へ、`ReadFile` を `FileRead` へと委ねることができます：

```flix
use Fs.FileExists
use Fs.FileRead
use Fs.FileTest
use Fs.ReadFile

def main(): Unit \ { FileRead, FileTest, IO } =
    run {
        safeRead("example.txt")
    } with FileExists.runWithFileTest
      with ReadFile.runWithFileRead

def safeRead(file: String): Unit \ { FileExists, ReadFile, IO } =
    match FileExists.exists(file) {
        case Err(err)  => println("Error: ${err}")
        case Ok(false) => println("File does not exist")
        case Ok(true)  =>
            match ReadFile.read(file) {
                case Ok(content) => println(content)
                case Err(err)    => println("Read error: ${err}")
            }
    }
```

<!--
# FileSystem

Flix provides a family of effects for filesystem operations. The key modules
are:

- `Fs.FileSystem` — the unified `FileSystem` effect (all 29 operations)
- `Fs.FileRead` — the `FileRead` effect (read, readLines, readBytes)
- `Fs.FileWrite` — the `FileWrite` effect (write, append, delete, copy, move, mkdir, etc.)
- `Fs.FileStat` — the `FileStat` effect (exists, type tests, permissions, timestamps, size)
- `Fs.DirList` — the `DirList` effect (listing directory contents)
- `Fs.Glob` — the `Glob` effect (finding files by pattern)
- `Fs.Size` — utilities for working with file sizes

All effects have default handlers, so no explicit `runWithIO` call is needed in
`main`.

There are also more fine-grained leaf effects (e.g. `FileExists`,
`ReadFile`, `WriteFile`) that do not have default handlers but can be run into
their parent effects using `runWith` handlers. See [The Effect
Hierarchy](#the-effect-hierarchy) for details.

## Reading a File

We can use `FileRead.read` to read an entire file as a string:

```flix
use Fs.FileRead

def main(): Unit \ { FileRead, IO } =
    match FileRead.read("example.txt") {
        case Ok(content) => println(content)
        case Err(err)    => println("Error: ${err}")
    }
```

All filesystem operations return `Result[IoError, ...]`. The `IoError` type is
a pair of an `ErrorKind` and a message string. The `ErrorKind` enum tells us
what went wrong:

| ErrorKind               | Description                                      |
|--------------------------|--------------------------------------------------|
| `NotFound`               | The file or directory was not found.             |
| `AlreadyExists`          | The file or directory already exists.             |
| `PermissionDenied`       | Access was denied (also used by middleware).      |
| `InvalidPath`            | The path is malformed.                           |
| ...                      | and others.                                      |

> **Note:** The `IO` effect appears in the signature because of `println`.

## Writing a File

We can use `FileWrite.write` to write a string to a file:

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.write(str = "Hello, Flix!", "greeting.txt") {
        case Ok(_)    => println("File written successfully.")
        case Err(err) => println("Error: ${err}")
    }
```

## Reading and Writing Lines

We can use `readLines` and `writeLines` to work with files line by line:

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    match FileWrite.writeLines(lines = List#{"Line 1", "Line 2", "Line 3"}, "data.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileRead.readLines("data.txt") {
                case Ok(lines) =>
                    foreach (line <- lines) {
                        println(line)
                    }
                case Err(err) => println("Read error: ${err}")
            }
    }
```

> **Note:** Since we both read and write, the effect set includes `FileRead`,
> `FileWrite`, and `IO`.

## Reading and Writing Bytes

We can use `readBytes` and `writeBytes` for binary data:

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    let data = Vector#{72i8, 101i8, 108i8, 108i8, 111i8};
    match FileWrite.writeBytes(data, "binary.dat") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileRead.readBytes("binary.dat") {
                case Ok(bytes) =>
                    println("Read ${Vector.length(bytes)} bytes.");
                    println("As string: ${String.fromBytes(bytes)}")
                case Err(err) => println("Read error: ${err}")
            }
    }
```

## Appending to a File

We can use `append` to add text to an existing file without overwriting it. The
file is created if it does not exist:

```flix
use Fs.FileRead
use Fs.FileWrite

def main(): Unit \ { FileRead, FileWrite, IO } =
    match FileWrite.write(str = "Line 1\n", "log.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileWrite.append(str = "Line 2\n", "log.txt") {
                case Err(err) => println("Append error: ${err}")
                case Ok(_) =>
                    match FileRead.read("log.txt") {
                        case Ok(content) => println(content)
                        case Err(err)    => println("Read error: ${err}")
                    }
            }
    }
```

There are also `appendLines` and `appendBytes` variants.

## Listing a Directory

We can use `DirList.list` to get the names of all files and directories in a
directory:

```flix
use Fs.DirList

def main(): Unit \ { DirList, IO } =
    match DirList.list(".") {
        case Ok(entries) =>
            foreach (entry <- entries) {
                println(entry)
            }
        case Err(err) => println("Error: ${err}")
    }
```

## Finding Files with Glob

We can use `Glob.glob` to find files matching a glob pattern under a base
directory:

```flix
use Fs.Glob

def main(): Unit \ { Glob, IO } =
    match Glob.glob(".", "*.flix") {
        case Ok(files) =>
            foreach (file <- files) {
                println(file)
            }
        case Err(err) => println("Error: ${err}")
    }
```

## File Metadata

We can use the `FileStat` effect to inspect file metadata: existence, type,
size, permissions, and timestamps:

```flix
use Fs.FileStat
use Fs.FileWrite

def main(): Unit \ { FileStat, FileWrite, IO } =
    let file = "example.txt";
    match FileWrite.write(str = "Hello!", file) {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileStat.exists(file) {
                case Ok(b)    => println("Exists: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.isRegularFile(file) {
                case Ok(b)    => println("Is regular file: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.isDirectory(file) {
                case Ok(b)    => println("Is directory: ${b}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.size(file) {
                case Ok(s)    => println("Size: ${s}")
                case Err(err) => println("Error: ${err}")
            };
            match FileStat.modificationTime(file) {
                case Ok(t)    => println("Modification time: ${t}ms")
                case Err(err) => println("Error: ${err}")
            }
    }
```

The `FileStat` effect combines four sub-effects:

| Sub-effect       | Operations                                             |
|------------------|--------------------------------------------------------|
| `FileTest`       | `exists`, `isDirectory`, `isRegularFile`, `isSymbolicLink` |
| `FilePermission` | `isReadable`, `isWritable`, `isExecutable`             |
| `FileTime`       | `accessTime`, `creationTime`, `modificationTime`       |
| `FileSize`       | `size`                                                 |

## Copying, Moving, and Deleting

We can also use the `FileWrite` effect to copy, move, and delete files:

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.write(str = "Hello!", "original.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            // Copy with no options.
            match FileWrite.copy(src = "original.txt", "copy.txt") {
                case Ok(_)    => println("Copied.")
                case Err(err) => println("Copy error: ${err}")
            };
            // Move (rename) with no options.
            match FileWrite.move(src = "copy.txt", "renamed.txt") {
                case Ok(_)    => println("Moved.")
                case Err(err) => println("Move error: ${err}")
            };
            // Delete.
            match FileWrite.delete("renamed.txt") {
                case Ok(_)    => println("Deleted.")
                case Err(err) => println("Delete error: ${err}")
            }
    }
```

The `copy` and `move` functions are convenience wrappers around `copyWith` and
`moveWith`, which accept option sets:

- `CopyOption.CopyAttributes` — preserve file attributes
- `CopyOption.ReplaceExisting` — overwrite the destination if it exists
- `MoveOption.AtomicMove` — perform an atomic rename
- `MoveOption.ReplaceExisting` — overwrite the destination if it exists

## Creating Directories

We can use `mkDir` to create a single directory, `mkDirs` to create a directory
and all its parents, and `mkTempDir` to create a temporary directory:

```flix
use Fs.FileWrite

def main(): Unit \ { FileWrite, IO } =
    match FileWrite.mkDirs("a/b/c") {
        case Ok(_)    => println("Created a/b/c.")
        case Err(err) => println("Error: ${err}")
    };
    match FileWrite.mkTempDir("flix-") {
        case Ok(path) => println("Temp dir: ${path}")
        case Err(err) => println("Error: ${err}")
    }
```

## The FileSystem Effect

The `FileSystem` effect combines all filesystem operations into a single
effect. It includes all operations from `FileStat`, `FileRead`, `FileWrite`,
`DirList`, and `Glob`. We can use `FileSystem` when we need multiple categories
of operations together:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.write(str = "Hello!", "greeting.txt") {
        case Err(err) => println("Write error: ${err}")
        case Ok(_) =>
            match FileSystem.read("greeting.txt") {
                case Ok(content) => println("Read: ${content}")
                case Err(err)    => println("Read error: ${err}")
            }
    }
```

## Middleware

Middleware are effect handlers that intercept filesystem operations. We apply
them using `run { ... } with FileSystem.<middleware>` (or the corresponding
sub-effect module) and compose them by stacking multiple `with` clauses.

### Base Directory

`withBaseDir` resolves relative paths against a base directory. Absolute paths
pass through unchanged:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.mkDirs("/tmp/flix-basedir") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "Hello", "greeting.txt") {
                    case Err(err) => println("Write error: ${err}")
                    case Ok(_) =>
                        match FileSystem.read("greeting.txt") {
                            case Ok(content) => println("Read: ${content}")
                            case Err(err)    => println("Read error: ${err}")
                        }
                }
            } with FileSystem.withBaseDir("/tmp/flix-basedir")
    }
```

### Chroot

`withChroot` restricts all operations to a directory subtree. Operations
targeting paths outside the chroot fail with a `PermissionDenied` error:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.mkDirs("/tmp/flix-chroot") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "Hello", "/tmp/flix-chroot/data.txt") {
                    case Ok(_)    => println("Write inside chroot succeeded")
                    case Err(err) => println("Error: ${err}")
                };
                match FileSystem.read("/etc/hostname") {
                    case Ok(_)    => println("Unexpected: read outside chroot succeeded")
                    case Err(err) => println("Read outside chroot blocked: ${err}")
                }
            } with FileSystem.withChroot("/tmp/flix-chroot")
    }
```

### Logging

`withLogging` logs each filesystem operation via the `Logger` effect. Note
that `Logger` appears in the type signature of `main`:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "Hello, Flix!", "greeting.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.read("greeting.txt") {
                    case Ok(content) => println(content)
                    case Err(err)    => println("Read error: ${err}")
                }
        }
    } with FileSystem.withLogging
```

### Read-Only

`withReadOnly` blocks all write operations with a `PermissionDenied` error.
Read and stat operations pass through normally:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "This will fail", "blocked.txt") {
            case Ok(_)    => println("Unexpected: write succeeded")
            case Err(err) => println("Write blocked: ${err}")
        };
        match FileSystem.exists("blocked.txt") {
            case Ok(b)    => println("Exists: ${b}")
            case Err(err) => println("Error: ${err}")
        }
    } with FileSystem.withReadOnly
```

### Dry Run

`withDryRun` logs write operations via the `Logger` effect without performing
them. Read operations still execute normally:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "This won't be written", "phantom.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.exists("phantom.txt") {
                    case Ok(b)    => println("Exists: ${b}")
                    case Err(err) => println("Error: ${err}")
                }
        }
    } with FileSystem.withDryRun
```

### Atomic Write

`withAtomicWrite` writes data to a temporary file first, then atomically
renames it into place. This prevents partial writes on failure. Only `write`,
`writeLines`, and `writeBytes` are affected — appends and other operations
pass through unchanged:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Atomic content", "output.txt") {
            case Ok(_)    => println("Atomic write succeeded.")
            case Err(err) => println("Write error: ${err}")
        }
    } with FileSystem.withAtomicWrite
```

### Backup

`withBackup` creates a backup copy of existing files before overwriting them.
Before each destructive operation (`write`, `writeLines`, `writeBytes`,
`truncate`, `delete`, `copyWith`, `moveWith`), the existing file is copied to
`file + suffix`:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    match FileSystem.write(str = "Original content", "data.txt") {
        case Err(err) => println("Setup error: ${err}")
        case Ok(_) =>
            run {
                match FileSystem.write(str = "New content", "data.txt") {
                    case Ok(_)    => println("Write succeeded; backup saved to data.txt.bak")
                    case Err(err) => println("Write error: ${err}")
                }
            } with FileSystem.withBackup(".bak")
    }
```

### Create Parent Directories

`withMkParentDirs` automatically creates parent directories before write
and append operations. If the parent directory already exists, this is a
no-op:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Hello", "deep/nested/path/greeting.txt") {
            case Ok(_)    => println("Write succeeded (parents created).")
            case Err(err) => println("Write error: ${err}")
        }
    } with FileSystem.withMkParentDirs
```

### Conflict Check

`withConflictCheck` tracks file modification times and rejects writes when the
file has been modified externally since the last operation. This catches
write-write conflicts from external processes:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "First write", "shared.txt") {
            case Err(err) => println("Error: ${err}")
            case Ok(_) =>
                match FileSystem.write(str = "Second write", "shared.txt") {
                    case Ok(_)    => println("No conflict detected.")
                    case Err(err) => println("Conflict: ${err}")
                }
        }
    } with FileSystem.withConflictCheck
```

### Transfer Limit

`withTransferLimit` rejects read or write operations where the payload exceeds
a maximum size:

```flix
use Fs.FileSystem
use Fs.Size

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.write(str = "Small", "ok.txt") {
            case Ok(_)    => println("Small write succeeded.")
            case Err(err) => println("Error: ${err}")
        }
    } with FileSystem.withTransferLimit(Size.megaBytes(10))
```

### Access Control

Flix provides middleware for restricting which paths can be accessed. We can use:

- `withAllowList(dirs)` — only paths within the listed directories are allowed
- `withDenyList(dirs)` — paths within the listed directories are blocked
- `withAllowGlob(patterns)` — only paths matching at least one pattern are allowed
- `withDenyGlob(patterns)` — paths matching any pattern are blocked

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        match FileSystem.read("/tmp/safe/data.txt") {
            case Ok(content) => println(content)
            case Err(err)    => println("Error: ${err}")
        }
    } with FileSystem.withAllowList(Nel.of("/tmp/safe"))
```

### In-Memory Filesystem

The `withInMemoryFS` handler replaces the real filesystem with a fully
in-memory implementation. The filesystem starts empty; reads of non-written
files return `NotFound`. No real filesystem access occurs:

```flix
use Fs.FileSystem
use Time.Clock

def main(): Unit \ { Clock, IO } =
    run {
        let result = forM (
            _       <- FileSystem.mkDirs("/data");
            _       <- FileSystem.write(str = "Hello", "/data/hello.txt");
            _       <- FileSystem.write(str = "World", "/data/world.txt");
            entries <- FileSystem.list("/data");
            content <- FileSystem.read("/data/hello.txt");
            _       <- FileSystem.delete("/data/hello.txt");
            exists  <- FileSystem.exists("/data/hello.txt")
        ) yield (entries, content, exists);
        match result {
            case Err(err) => println("Error: ${err}")
            case Ok((entries, content, exists)) =>
                println("Files in /data:");
                foreach (entry <- entries) {
                    println("  ${entry}")
                };
                println("Content: ${content}");
                println("Exists after delete: ${exists}")
        }
    } with FileSystem.withInMemoryFS
```

Note that `withInMemoryFS` requires the `Clock` effect (for file timestamps)
but removes `FileSystem` from the effect signature since it fully handles it.

### Memory Overlay

The `withMemoryOverlay` handler layers an in-memory writable store on top of
the real filesystem. Writes are captured in memory and subsequent reads see the
written data, but the real filesystem is never modified. Reads of files not in the
overlay fall through to the real filesystem:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, IO } =
    run {
        // This write is captured in memory, not written to disk.
        match FileSystem.write(str = "In-memory only", "virtual.txt") {
            case Err(err) => println("Error: ${err}")
            case Ok(_) =>
                match FileSystem.read("virtual.txt") {
                    case Ok(content) => println("Read from overlay: ${content}")
                    case Err(err)    => println("Error: ${err}")
                }
        }
    } with FileSystem.withMemoryOverlay
```

## Composing Middleware

We can compose middleware by stacking `with` clauses. The innermost handler
(listed first) intercepts the original operation, and then delegates to the
next outer handler. Here is an example that stacks base directory, parent
directory creation, backup, atomic writes, conflict checking, and logging:

```flix
use Fs.FileSystem

def main(): Unit \ { FileSystem, Logger, IO } =
    run {
        match FileSystem.write(str = "Hello, Flix!", "data/greeting.txt") {
            case Err(err) => println("Write error: ${err}")
            case Ok(_) =>
                match FileSystem.read("data/greeting.txt") {
                    case Ok(content) => println("Read: ${content}")
                    case Err(err)    => println("Read error: ${err}")
                }
        }
    } with FileSystem.withBaseDir("/tmp/flix-example")
      with FileSystem.withMkParentDirs
      with FileSystem.withConflictCheck
      with FileSystem.withBackup(".bak")
      with FileSystem.withAtomicWrite
      with FileSystem.withLogging
```

The `FileSystem` and `Logger` effects both have default handlers, so they are
handled automatically when they appear in the type signature of `main`.

> **Note:** The order of `with` clauses matters. The outermost handler (listed
> last) wraps all inner handlers. In the example above, `withLogging` is
> outermost, so it sees *every* filesystem operation — including retries from
> conflict checks and temporary files from atomic writes. When composing
> middleware, think about which layer should observe which operations.

## Middleware Summary

The following table shows which middleware are available on which effects
(scroll right to see the full table):

| Middleware             | FileTest | FilePermission | FileTime | FileStat | FileRead | DirList | Glob | FileWrite | FileSystem |
|------------------------|:--------:|:--------------:|:--------:|:--------:|:--------:|:-------:|:----:|:---------:|:----------:|
| `withLogging`          | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withBaseDir`          | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withChroot`           | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withAllowList`        | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withDenyList`         | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withAllowGlob`        | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withDenyGlob`         | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withFollowLinks`      | x        | x              | x        | x        | x        | x       | x    | x         | x          |
| `withTransferLimit`    |          |                |          |          | x        |         |      | x         | x          |
| `withChecksum`         |          |                |          |          | x        |         |      | x         | x          |
| `withDryRun`           |          |                |          |          |          |         |      | x         | x          |
| `withReadOnly`         |          |                |          |          |          |         |      | x         | x          |
| `withAtomicWrite`      |          |                |          |          |          |         |      | x         | x          |
| `withBackup`           |          |                |          |          |          |         |      | x         | x          |
| `withConflictCheck`    |          |                |          |          |          |         |      | x         | x          |
| `withMkParentDirs`     |          |                |          |          |          |         |      | x         | x          |
| `withSizeRotation`     |          |                |          |          |          |         |      | x         | x          |
| `withMemoryOverlay`    |          |                |          |          |          |         |      |           | x          |
| `withInMemoryFS`       |          |                |          |          |          |         |      |           | x          |

## The Effect Hierarchy

The Flix filesystem effects form a hierarchy. At the top is `FileSystem` with
all 29 operations. Below it are intermediate effects that group related
operations, and at the bottom are leaf effects for individual operations:

```text
FileSystem                          (29 ops — unified root)
├── FileStat                        (11 ops)
│   ├── FileTest                    (4 ops: exists, isDirectory, isRegularFile, isSymbolicLink)
│   ├── FilePermission              (3 ops: isReadable, isWritable, isExecutable)
│   ├── FileTime                    (3 ops: accessTime, creationTime, modificationTime)
│   └── FileSize                    (1 op: size)
├── FileRead                        (3 ops: read, readLines, readBytes)
├── DirList                         (1 op: list)
├── Glob                            (1 op: glob)
└── FileWrite                       (13 ops: write, append, delete, copy, move, mkdir, etc.)
```

We can use any level of the hierarchy. For example, we can use a leaf effect
like `FileExists` when we only need `exists`, `FileRead` when we need to read
files, or `FileSystem` when we need everything.

We can run leaf effects into their parent effects using `runWith` handlers.
For example, we can run `FileExists` into `FileTest` and `ReadFile` into
`FileRead`:

```flix
use Fs.FileExists
use Fs.FileRead
use Fs.FileTest
use Fs.ReadFile

def main(): Unit \ { FileRead, FileTest, IO } =
    run {
        safeRead("example.txt")
    } with FileExists.runWithFileTest
      with ReadFile.runWithFileRead

def safeRead(file: String): Unit \ { FileExists, ReadFile, IO } =
    match FileExists.exists(file) {
        case Err(err)  => println("Error: ${err}")
        case Ok(false) => println("File does not exist")
        case Ok(true)  =>
            match ReadFile.read(file) {
                case Ok(content) => println(content)
                case Err(err)    => println("Read error: ${err}")
            }
    }
```
-->
