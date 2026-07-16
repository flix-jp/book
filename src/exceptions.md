# 例外

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/exceptions.html)を参照してください。

Flix では、`try-catch` 構文を使って Java の例外(Exception)を捕捉できます。この構文は Java のものと似ていますが、文法が少し異なります。

例えば：

```flix
import java.io.BufferedReader
import java.io.File
import java.io.FileReader
import java.io.FileNotFoundException
import java.io.IOException

def main(): Unit \ IO = 
    let f = new File("foo.txt");
    try {
        let r = new BufferedReader(new FileReader(f));
        let l = r.readLine();
        println("The first line of the file is: ${l}");
        r.close()
    } catch {
        case _: FileNotFoundException => 
            println("The file does not exist!")
        case ex: IOException => 
            println("The file could not be read!");
            println("The error message was: ${ex.getMessage()}")
    }
```

ここでは、`new FileReader()`、`r.readLine()`、`r.close()` の呼び出しが `IOException` を投げる可能性があります。これらの例外を捕捉するために `try-catch` ブロックを使っています。また、`FileNotFoundException` 例外に対しては専用の case を追加しています。

> **注意:** Flix のプログラムでは例外を使うべきではありません。これは悪いスタイルとみなされています。代わりに、プログラムでは `Result[e, t]` 型を使うべきです。`try-catch` 構文は、Flix と Java のコードの境界でのみ使用してください。

> **注意:** Flix は（まだ）`finally` ブロックをサポートしていません。

<!--
# Exceptions

In Flix, we can catch Java exceptions using the `try-catch` construct. The
construct is similar to the one in Java, but the syntax is slightly different. 

For example: 

```flix
import java.io.BufferedReader
import java.io.File
import java.io.FileReader
import java.io.FileNotFoundException
import java.io.IOException

def main(): Unit \ IO = 
    let f = new File("foo.txt");
    try {
        let r = new BufferedReader(new FileReader(f));
        let l = r.readLine();
        println("The first line of the file is: ${l}");
        r.close()
    } catch {
        case _: FileNotFoundException => 
            println("The file does not exist!")
        case ex: IOException => 
            println("The file could not be read!");
            println("The error message was: ${ex.getMessage()}")
    }
```

Here the calls `new FileReader()`, `r.readLine()`, and `r.close()` can throw
`IOException`s. We use a `try-catch` block to catch these exceptions. We add a
special case for the `FileNotFoundException` exception. 

> **Note:** Flix programs should not use exceptions: it is considered bad style.
> Instead, programs should use the `Result[e, t]` type. The `try-catch`
> construct should only be used on the boundary between Flix and Java code. 

> **Note:** Flix does not (yet) support a `finally` block.
-->
