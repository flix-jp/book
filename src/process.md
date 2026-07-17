# Process

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/process.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/process.md)ください。

Flix は、OS プロセスの起動と管理のためのライブラリエフェクト(library effect)として `Process` を提供しています。`Process` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。中心となるモジュールは `Sys.Process` です。

## Process エフェクト

`Process` エフェクトは、プロセスの起動、その入出力ストリームへのアクセス、そして終了の待機をサポートしています。

```flix
pub eff Process {
    /// コマンド `cmd` を、引数 `args`、作業ディレクトリ `cwd`、環境変数 `env` で実行します。
    def execWithCwdAndEnv(cmd: String, args: List[String],
        cwd: Option[String], env: Map[String, String]):
        Result[IoError, ProcessHandle]

    /// プロセス `ph` の終了値を返します。
    def exitValue(ph: ProcessHandle): Result[IoError, Int32]

    /// プロセス `ph` が生存しているかどうかを返します。
    def isAlive(ph: ProcessHandle): Result[IoError, Bool]

    /// プロセス `ph` の PID を返します。
    def pid(ph: ProcessHandle): Result[IoError, Int64]

    /// プロセス `ph` の標準入力ストリームを返します。
    def stdin(ph: ProcessHandle): Result[IoError, StdIn]

    /// プロセス `ph` の標準出力ストリームを返します。
    def stdout(ph: ProcessHandle): Result[IoError, StdOut]

    /// プロセス `ph` の標準エラーストリームを返します。
    def stderr(ph: ProcessHandle): Result[IoError, StdErr]

    /// プロセス `ph` を停止します。
    def stop(ph: ProcessHandle): Result[IoError, Unit]

    /// プロセス `ph` の終了を待機し、その終了値を返します。
    def waitFor(ph: ProcessHandle): Result[IoError, Int32]

    /// プロセス `ph` の終了を最大 `time`（単位は `tUnit`）だけ待機します。
    /// プロセスが終了した場合は `true` を、タイムアウトした場合は `false` を返します。
    def waitForTimeout(ph: ProcessHandle, time: Int64, tUnit: TimeUnit):
        Result[IoError, Bool]
}
```

## Process モジュール

`Process` モジュールは、`Process` エフェクトの上に構築された便利な関数を提供しています。

```flix
mod Process {
    /// コマンド `cmd` を引数 `args` で実行します。
    def exec(cmd: String, args: List[String]):
        Result[IoError, ProcessHandle] \ Process

    /// `cmd` を、引数 `args` と作業ディレクトリ `cwd` で実行します。
    def execWithCwd(cmd: String, args: List[String], cwd: Option[String]):
        Result[IoError, ProcessHandle] \ Process

    /// `cmd` を、引数 `args` と環境変数 `env` で実行します。
    def execWithEnv(cmd: String, args: List[String], env: Map[String, String]):
        Result[IoError, ProcessHandle] \ Process
}
```

## コマンドの実行

OS プロセスを起動する最も簡単な方法は `Process.exec` を使うことです。この関数はコマンドと引数のリストを受け取り、`Result[IoError, ProcessHandle]` を返します。

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.exec("java", "-version" :: Nil) {
        case Result.Ok(_)    => println("Process started successfully.")
        case Result.Err(err) => println("Unable to execute process: ${err}")
    }
```

## プロセス出力の読み取り

プロセスを起動した後、`Process.stdout` と `Process.stderr` でその出力ストリームにアクセスできます。返される `StdOut` 型と `StdErr` 型は `Readable` を実装しているため、そこからバイト列を読み取ることができます。

```flix
use Sys.Process

def main(): Unit \ { Process, IO } = region rc {
    match Process.exec("java", "-version" :: Nil) {
        case Result.Err(err) => println("exec failed: ${err}")
        case Result.Ok(ph)   =>
            match Process.stderr(ph) {
                case Result.Err(err) => println("stderr failed: ${err}")
                case Result.Ok(err)  =>
                    let buf = Array.repeat(rc, 1024, (0i8: Int8));
                    match Readable.read(buf, err) {
                        case Result.Err(e) => println("read failed: ${e}")
                        case Result.Ok(n)  => println("Read ${n} bytes from stderr.")
                    }
            }
    }
}
```

> **注意:** `java -version` は標準出力ではなく標準エラーに書き込みます。

## 終了の待機

プロセスが終了するまでブロックするには `Process.waitFor` を使います。この関数は終了コードを `Int32` として返します。

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.exec("java", "-version" :: Nil) {
        case Result.Err(err) => println("exec failed: ${err}")
        case Result.Ok(ph)   =>
            match Process.waitFor(ph) {
                case Result.Err(err) => println("waitFor failed: ${err}")
                case Result.Ok(code) => println("Process exited with code: ${code}")
            }
    }
```

## 作業ディレクトリと環境変数

`Process.execWithCwd` は、特定の作業ディレクトリを指定してプロセスを起動します。`Process.execWithEnv` は、追加の環境変数を渡します。

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.execWithCwd("java", "-version" :: Nil, Some("/tmp")) {
        case Result.Ok(_)    => println("execWithCwd succeeded.")
        case Result.Err(err) => println("execWithCwd failed: ${err}")
    };
    match Process.execWithEnv("java", "-version" :: Nil, Map#{"MY_VAR" => "hello"}) {
        case Result.Ok(_)    => println("execWithEnv succeeded.")
        case Result.Err(err) => println("execWithEnv failed: ${err}")
    }
```

<!--
# Process

Flix provides `Process` as a library effect for spawning and managing OS
processes. The `Process` effect has a default handler, so no explicit
`runWithIO` call is needed in `main`. The key module is `Sys.Process`.

## The Process Effect

The `Process` effect supports spawning processes, accessing their I/O streams,
and waiting for completion:

```flix
pub eff Process {
    /// Executes `cmd` with `args`, working directory `cwd`, and environment `env`.
    def execWithCwdAndEnv(cmd: String, args: List[String],
        cwd: Option[String], env: Map[String, String]):
        Result[IoError, ProcessHandle]

    /// Returns the exit value of the process `ph`.
    def exitValue(ph: ProcessHandle): Result[IoError, Int32]

    /// Returns whether the process `ph` is alive.
    def isAlive(ph: ProcessHandle): Result[IoError, Bool]

    /// Returns the PID of the process `ph`.
    def pid(ph: ProcessHandle): Result[IoError, Int64]

    /// Returns the stdin stream of the process `ph`.
    def stdin(ph: ProcessHandle): Result[IoError, StdIn]

    /// Returns the stdout stream of the process `ph`.
    def stdout(ph: ProcessHandle): Result[IoError, StdOut]

    /// Returns the stderr stream of the process `ph`.
    def stderr(ph: ProcessHandle): Result[IoError, StdErr]

    /// Stops the process `ph`.
    def stop(ph: ProcessHandle): Result[IoError, Unit]

    /// Waits for the process `ph` to finish and returns its exit value.
    def waitFor(ph: ProcessHandle): Result[IoError, Int32]

    /// Waits at most `time` (in the given `tUnit`) for the process `ph` to finish.
    /// Returns `true` if the process exited, `false` if the timeout elapsed.
    def waitForTimeout(ph: ProcessHandle, time: Int64, tUnit: TimeUnit):
        Result[IoError, Bool]
}
```

## The Process Module

The `S` module provides convenience functions built on the `Process`
effect:

```flix
mod Process {
    /// Executes the command `cmd` with the arguments `args`.
    def exec(cmd: String, args: List[String]):
        Result[IoError, ProcessHandle] \ Process

    /// Executes `cmd` with `args` and working directory `cwd`.
    def execWithCwd(cmd: String, args: List[String], cwd: Option[String]):
        Result[IoError, ProcessHandle] \ Process

    /// Executes `cmd` with `args` and environment variables `env`.
    def execWithEnv(cmd: String, args: List[String], env: Map[String, String]):
        Result[IoError, ProcessHandle] \ Process
}
```

## Executing a Command

The simplest way to spawn an OS process is with `Process.exec`. It takes a
command and a list of arguments, and returns a
`Result[IoError, ProcessHandle]`:

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.exec("java", "-version" :: Nil) {
        case Result.Ok(_)    => println("Process started successfully.")
        case Result.Err(err) => println("Unable to execute process: ${err}")
    }
```

## Reading Process Output

After spawning a process, you can access its output streams with
`Process.stdout` and `Process.stderr`. The returned `StdOut` and `StdErr` types
implement `Readable`, so you can read bytes from them:

```flix
use Sys.Process

def main(): Unit \ { Process, IO } = region rc {
    match Process.exec("java", "-version" :: Nil) {
        case Result.Err(err) => println("exec failed: ${err}")
        case Result.Ok(ph)   =>
            match Process.stderr(ph) {
                case Result.Err(err) => println("stderr failed: ${err}")
                case Result.Ok(err)  =>
                    let buf = Array.repeat(rc, 1024, (0i8: Int8));
                    match Readable.read(buf, err) {
                        case Result.Err(e) => println("read failed: ${e}")
                        case Result.Ok(n)  => println("Read ${n} bytes from stderr.")
                    }
            }
    }
}
```

> **Note:** `java -version` writes to stderr, not stdout.

## Waiting for Completion

Use `Process.waitFor` to block until a process exits. It returns the exit code
as an `Int32`:

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.exec("java", "-version" :: Nil) {
        case Result.Err(err) => println("exec failed: ${err}")
        case Result.Ok(ph)   =>
            match Process.waitFor(ph) {
                case Result.Err(err) => println("waitFor failed: ${err}")
                case Result.Ok(code) => println("Process exited with code: ${code}")
            }
    }
```

## Working Directory and Environment

`Process.execWithCwd` spawns a process with a specific working directory.
`Process.execWithEnv` passes additional environment variables:

```flix
use Sys.Process

def main(): Unit \ { Process, IO } =
    match Process.execWithCwd("java", "-version" :: Nil, Some("/tmp")) {
        case Result.Ok(_)    => println("execWithCwd succeeded.")
        case Result.Err(err) => println("execWithCwd failed: ${err}")
    };
    match Process.execWithEnv("java", "-version" :: Nil, Map#{"MY_VAR" => "hello"}) {
        case Result.Ok(_)    => println("execWithEnv succeeded.")
        case Result.Err(err) => println("execWithEnv failed: ${err}")
    }
```
-->
