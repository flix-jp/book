# Visual Studio Code 拡張機能

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/vscode.html)を参照してください。

Flix には[フル機能の Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=flix.flix)が用意されています：

![Visual Studio Code1](images/vscode1.png)

Flix 拡張機能は本物の Flix コンパイラを使用しているため、（エラーメッセージなどの）すべての情報は、常に実際の Flix プログラミング言語と 1:1 で一致します。

また、Flix には "Flixify Dark" という（任意で使用できる）Visual Studio Code のカラーテーマも用意されています。

## 機能

* __セマンティックシンタックスハイライト__
    - *.flix ファイルのコードハイライト。[公式の vscode テーマ](https://marketplace.visualstudio.com/items?itemName=flix.flixify-dark)と組み合わせると最も効果的です。

* __診断__
    - コンパイラのエラーメッセージを表示します。

* __自動補完__
    - 入力中に自動補完を行います。
    - 自動補完はコンテキストを考慮します。
    - プログラムホールに対する型駆動の補完を行います。

* __スニペット__
    - よく使われるコード構文を自動補完します。

* __インレイヒント__
    - インラインで型情報を表示します。

* __型とエフェクトのホバー表示__
    - 任意の式にホバーすると、その型とエフェクトを確認できます。
    - 任意のローカル変数や仮引数にホバーすると、その型を確認できます。
    - 任意の関数にホバーすると、その型シグネチャとドキュメントを確認できます。

* __定義へジャンプ__
    - 任意の関数の定義へジャンプできます。
    - 任意のローカル変数や仮引数の定義へジャンプできます。
    - 任意の enum ケースの定義へジャンプできます。

* __参照の検索__
    - 関数へのすべての参照を検索できます。
    - ローカル変数や仮引数へのすべての参照を検索できます。
    - enum ケースへのすべての参照を検索できます。
    - トレイトのすべての実装を検索できます。

* __シンボル__
    - ドキュメント内のすべてのシンボルを一覧表示します。
    - ワークスペース内のすべてのシンボルを一覧表示します。

* __リネーム__
    - ローカル変数や仮引数をリネームできます。
    - 関数をリネームできます。

* __コードレンズ__
    - エディタ内から `main` を実行できます。
    - エディタ内からテストを実行できます。

* __ハイライト__
    - 意味的に関連するシンボルをハイライトします。

* __セマンティックトークン__
    - コンパイラが提供する追加のコードハイライトのヒントです。

## 既知の制限事項

- PowerShell で特殊文字を含むファイル名を使用すると、既知の問題が発生します。Flix のソースファイルには ASCII 文字のみの名前を付けることをおすすめします。

- この拡張機能は「ワークスペースモード(Workspace Mode)」で作業していること、つまり Flix のソースコードを含むフォルダを開いていることを前提としています。

- 起動時に、Flix コンパイラは Flix 標準ライブラリ全体をキャッシュに読み込む必要があるため、数秒かかります。

<!--
# Visual Studio Code Extension

Flix comes with [a fully-featured Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=flix.flix):

![Visual Studio Code1](images/vscode1.png)

The Flix extension uses the real Flix compiler hence all information (e.g. error
messages) are always 1:1 with the real Flix programming language.

Flix also comes with an (optional) Visual Studio Code color theme called "Flixify Dark".

## Features

* __Semantic Syntax Highlighting__
    - Code highlighting for *.flix files. This work best with the [official vscode theme](https://marketplace.visualstudio.com/items?itemName=flix.flixify-dark).

* __Diagnostics__
    - Compiler error messages. 

* __Auto-complete__
    - Auto-complete as you type.
    - Auto-completion is context aware.
    - Type-directed completion of program holes.

* __Snippets__
    - Auto-complete common code constructs.

* __Inlay Hints__
    - Shows inline type information.

* __Type and Effect Hovers__
    - Hover over any expression to see its type and effect.
    - Hover over any local variable or formal parameter to see its type.
    - Hover over any function to see its type signature and documentation.

* __Jump to Definition__
    - Jump to the definition of any function.
    - Jump to the definition of any local variable or formal parameter.
    - Jump to the definition of any enum case.

* __Find References__
    - Find all references to a function.
    - Find all references to a local variable or formal parameter.
    - Find all references to an enum case.
    - Find all implementations of a trait.

* __Symbols__
    - List all document symbols.
    - List all workspace symbols.

* __Rename__
    - Rename local variables or formal parameters.
    - Rename functions.

* __Code Lenses__
    - Run `main` from within the editor.
    - Run tests from within the editor.

* __Highlight__
    - Highlights semantically related symbols.

* __Semantic Tokens__
    - Additional code highlighting hints provided by the compiler.

## Known Limitations

- There is a known issue with PowerShell and using file names that contain
  special characters. We recommend that Flix source files are given only ASCII
  names. 

- The extension assumes that you are working in "Workspace Mode", i.e. you must
  have a folder open which contains your Flix source code. 

- Upon startup, the Flix compiler has to load the entire Flix standard library
  into its caches which takes a few seconds.
-->
