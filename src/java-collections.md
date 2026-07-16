# Java のコレクション

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/java-collections.html)を参照してください。

Flix は Java のコレクションとの相互変換をサポートしています。

以下では、次のインポートエイリアスを使用します：

```flix
import java.util.{List => JList}
import java.util.{LinkedList => JLinkedList}
import java.util.{ArrayList => JArrayList}
import java.util.{Set => JSet}
import java.util.{TreeSet => JTreeSet}
import java.util.{Map => JMap}
import java.util.{TreeMap => JTreeMap}
```

次の関数は [Adaptor](https://api.flix.dev/Adaptor.html) モジュールで利用できます：

## Flix から Java へ

次の関数は Flix のコレクションを Java のコレクションに*変換*します：

```flix
///
/// リスト
///
def toList(ma: m[a]): JList[a] \ IO + Aef[m] with Foldable[m]
def toArrayList(ma: m[a]): JArrayList[a] \ IO + Aef[m] with Foldable[m]
def toLinkedList(ma: m[a]): JLinkedList[a] \ IO + Aef[m] with Foldable[m]

///
/// セット
///
def toSet(ma: m[a]): JSet[a] \ IO + Aef[m] with Order[a], Foldable[m]
def toTreeSet(ma: m[a]): JTreeSet[a] \ IO + Aef[m] with Order[a], Foldable[m]

///
/// マップ
///
def toMap(m: Map[k, v]): JMap[k, v] \ IO with Order[k]
def toTreeMap(m: Map[k, v]): JTreeMap[k, v] \ IO with Order[k]
```

各関数は新しいコレクションを構築し、すべての要素をそこにコピーします。そのため、各操作には少なくとも線形時間がかかります。結果は通常の Java コレクションであり、変更することもできます。

## Java から Flix へ

次の関数は Java のコレクションを Flix のコレクションに*変換*します：

```flix
/// リスト
def fromList(l: JList[a]): List[a]

/// セット
def fromSet(l: JSet[a]): Set[a] with Order[a]

/// マップ
def fromMap(m: JMap[k, v]): Map[k, v] with Order[k]
```

各関数は Java のコレクションから新しい Flix のコレクションを構築します。そのため、各操作には少なくとも線形時間がかかります。なお、`Set` と `Map` については、Flix のコレクションは `a` に定義された `Order[a]` インスタンスを使用します。これは Java が使用する順序と同じであるとは限りません。

> **警告:** プリミティブ値を持つ Flix や Java のコレクションを変換する際には、特別な注意が必要です。特に、変換の前に値を手動でボックス化(boxing)またはアンボックス化(unboxing)しなければなりません。

<!--
# Java Collections

Flix has support for conversion from and to Java collections. 

In the following, we use the following import aliases:

```flix
import java.util.{List => JList}
import java.util.{LinkedList => JLinkedList}
import java.util.{ArrayList => JArrayList}
import java.util.{Set => JSet}
import java.util.{TreeSet => JTreeSet}
import java.util.{Map => JMap}
import java.util.{TreeMap => JTreeMap}
```

The following functions are available in the
[Adaptor](https://api.flix.dev/Adaptor.html) module: 

## Flix to Java

The following functions _convert_ Flix collections to Java collections:

```flix
///
/// Lists
///
def toList(ma: m[a]): JList[a] \ IO + Aef[m] with Foldable[m]
def toArrayList(ma: m[a]): JArrayList[a] \ IO + Aef[m] with Foldable[m]
def toLinkedList(ma: m[a]): JLinkedList[a] \ IO + Aef[m] with Foldable[m]

///
/// Sets
///
def toSet(ma: m[a]): JSet[a] \ IO + Aef[m] with Order[a], Foldable[m]
def toTreeSet(ma: m[a]): JTreeSet[a] \ IO + Aef[m] with Order[a], Foldable[m]

///
/// Maps
///
def toMap(m: Map[k, v]): JMap[k, v] \ IO with Order[k]
def toTreeMap(m: Map[k, v]): JTreeMap[k, v] \ IO with Order[k]
```

Each function constructs a new collection and copies all its elements into it.
Hence each operation takes at least linear time. The result is a normal Java
collection (which can be modified). 

## Java to Flix

The following functions _convert_ Java collections to Flix collections:

```flix
/// Lists
def fromList(l: JList[a]): List[a]

/// Sets
def fromSet(l: JSet[a]): Set[a] with Order[a]

/// Maps
def fromMap(m: JMap[k, v]): Map[k, v] with Order[k]
```

Each function constructs a new Flix collection from a Java Collection. Hence
each operation takes at least linear time. Note that for `Set` and `Map`, the
Flix collections use the `Order[a]` instance defined on `a`. This may not be the
same ordering as used by Java. 

> **Warning:** Converting Flix and/or Java collections with primitive values
> requires extra care. In particular, values must be manually boxed or unboxed
> before any conversion.
-->
