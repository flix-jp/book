# Env

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/env.html)を参照してください。

Flix は、環境変数、システムプロパティ、およびプラットフォーム情報にアクセスするためのライブラリエフェクト(Library effect)として `Env` を提供しています。`Env` エフェクトにはデフォルトハンドラがあるため、`main` の中で明示的に `runWithIO` を呼び出す必要はありません。主要なモジュールは `Sys.Env` です。

## Env エフェクト

`Env` エフェクトは、プログラムの環境を読み取るための操作を提供します：

```flix
pub eff Env {
    /// コマンドラインからプログラムに渡された引数を返します。
    def getArgs(): List[String]

    /// 現在のシステム環境の Map を返します。
    def getEnv(): Map[String, String]

    /// 指定された環境変数の値を返します。
    def getVar(name: String): Option[String]

    /// 名前で指定されたシステムプロパティを返します。
    def getProp(name: String): Option[String]

    /// オペレーティングシステム名を返します。
    def getOsName(): Option[String]

    /// オペレーティングシステムのアーキテクチャを返します。
    def getOsArch(): Option[String]

    /// オペレーティングシステムのバージョンを返します。
    def getOsVersion(): Option[String]

    /// ファイル区切り文字を返します。
    def getFileSeparator(): String

    /// パス区切り文字を返します。
    def getPathSeparator(): String

    /// システムの行区切り文字を返します。
    def getLineSeparator(): String

    /// ユーザーの現在の作業ディレクトリを返します。
    def getCurrentWorkingDirectory(): Option[String]

    /// デフォルトの一時ファイルのパスを返します。
    def getTemporaryDirectory(): Option[String]

    /// ユーザーのアカウント名を返します。
    def getUserName(): Option[String]

    /// ユーザーのホームディレクトリを返します。
    def getUserHomeDirectory(): Option[String]

    /// JVM が利用できる仮想プロセッサ数を返します。
    def getVirtualProcessors(): Int32
}
```

## ディレクトリと OS 情報

`Env` のもっとも簡単な使い方は、現在のディレクトリ、ホームディレクトリ、あるいはオペレーティングシステムを問い合わせることです：

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let home = Env.getUserHomeDirectory();
    println("Home: ${home}");
    let cwd = Env.getCurrentWorkingDirectory();
    println("CWD: ${cwd}");
    let os = Env.getOsName();
    println("OS: ${os}")
```

## 環境変数

単一の環境変数を読み取るには `getVar` を、すべての環境変数を Map として取得するには `getEnv` を使います：

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let path = Env.getVar("PATH");
    println("PATH: ${path}");
    let all = Env.getEnv();
    println("Total env vars: ${Map.size(all)}")
```

## システム情報

`Env` エフェクトは、OS のアーキテクチャや利用可能なプロセッサ数といったプラットフォームの詳細情報も公開しています：

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let name    = Env.getOsName();
    let arch    = Env.getOsArch();
    let version = Env.getOsVersion();
    let cpus    = Env.getVirtualProcessors();
    println("OS:   ${name}");
    println("Arch: ${arch}");
    println("Ver:  ${version}");
    println("CPUs: ${cpus}")
```

<!--
# Env

Flix provides `Env` as a library effect for accessing environment variables,
system properties, and platform information. The `Env` effect has a default
handler, so no explicit `runWithIO` call is needed in `main`. The key module is
`Sys.Env`.

## The Env Effect

The `Env` effect provides operations for reading the program's environment:

```flix
pub eff Env {
    /// Returns the arguments passed to the program via the command line.
    def getArgs(): List[String]

    /// Returns a map of the current system environment.
    def getEnv(): Map[String, String]

    /// Returns the value of the specified environment variable.
    def getVar(name: String): Option[String]

    /// Returns the system property by name.
    def getProp(name: String): Option[String]

    /// Returns the operating system name.
    def getOsName(): Option[String]

    /// Returns the operating system architecture.
    def getOsArch(): Option[String]

    /// Returns the operating system version.
    def getOsVersion(): Option[String]

    /// Returns the file separator.
    def getFileSeparator(): String

    /// Returns the path separator.
    def getPathSeparator(): String

    /// Returns the system line separator.
    def getLineSeparator(): String

    /// Returns the user's current working directory.
    def getCurrentWorkingDirectory(): Option[String]

    /// Returns the default temp file path.
    def getTemporaryDirectory(): Option[String]

    /// Returns the user's account name.
    def getUserName(): Option[String]

    /// Returns the user's home directory.
    def getUserHomeDirectory(): Option[String]

    /// Returns the number of virtual processors available to the JVM.
    def getVirtualProcessors(): Int32
}
```

## Directories and OS Info

The simplest use of `Env` is to query the current directory, home directory,
or operating system:

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let home = Env.getUserHomeDirectory();
    println("Home: ${home}");
    let cwd = Env.getCurrentWorkingDirectory();
    println("CWD: ${cwd}");
    let os = Env.getOsName();
    println("OS: ${os}")
```

## Environment Variables

Use `getVar` to read a single environment variable, or `getEnv` to retrieve all
of them as a map:

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let path = Env.getVar("PATH");
    println("PATH: ${path}");
    let all = Env.getEnv();
    println("Total env vars: ${Map.size(all)}")
```

## System Information

The `Env` effect also exposes platform details such as the OS architecture and
the number of available processors:

```flix
use Sys.Env

def main(): Unit \ { Env, IO } =
    let name    = Env.getOsName();
    let arch    = Env.getOsArch();
    let version = Env.getOsVersion();
    let cpus    = Env.getVirtualProcessors();
    println("OS:   ${name}");
    println("Arch: ${arch}");
    println("Ver:  ${version}");
    println("CPUs: ${cpus}")
```
-->
