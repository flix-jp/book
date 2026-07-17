# テストフレームワーク

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/test-framework.html)を参照するか、[翻訳にご協力](https://github.com/flix-jp/book/edit/master/src/test-framework.md)ください。

Flix には組み込みのテストフレームワークが付属しています。テストは、`@Test` アノテーション(Annotation)が付けられた Flix の関数です。テスト関数は引数を取らず、`Unit` を返す必要があります。

`Assert` モジュールは、テストのためのアサーション(Assertion)関数を提供しています。よく使われるものを以下に示します：

| 関数                                           | 目的                                 |
|------------------------------------------------|--------------------------------------|
| `Assert.assertEq(expected = value, actual)`    | 値同士が等しいことをアサートする     |
| `Assert.assertNeq(unexpected = value, actual)` | 値同士が等しくないことをアサートする |
| `Assert.assertTrue(cond)`                      | 条件が真であることをアサートする     |
| `Assert.assertFalse(cond)`                     | 条件が偽であることをアサートする     |
| `Assert.assertSome(opt)`                       | Option が Some であることをアサートする |
| `Assert.assertNone(opt)`                       | Option が None であることをアサートする |
| `Assert.assertOk(res)`                         | Result が Ok であることをアサートする |
| `Assert.assertErr(res)`                        | Result が Err であることをアサートする |
| `Assert.assertEmpty(coll)`                     | コレクションが空であることをアサートする |
| `Assert.assertMemberOf(x, coll)`               | 要素がコレクションに含まれることをアサートする |
| `Assert.fail(msg)`                             | メッセージ付きで無条件に失敗する     |
| `Assert.success(msg)`                          | メッセージ付きで無条件に成功する     |

`assertEq` 関数と `assertNeq` 関数には、ラベル付き引数(Labelled argument) `expected` / `unexpected` が必要です。

以下に例を示します：

```flix
use Assert.{assertEq, assertTrue, assertFalse, assertOk, assertErr}

def add(x: Int32, y: Int32): Int32 = x + y

def isEven(x: Int32): Bool = Int32.modulo(x, 2) == 0

def safeDivide(x: Int32, y: Int32): Result[String, Int32] =
    if (y == 0) Err("Division by zero") else Ok(x / y)

@Test
def testAdd01(): Unit \ Assert =
    assertEq(expected = 5, add(2, 3))

@Test
def testIsEven01(): Unit \ Assert =
    assertTrue(isEven(4))

@Test
def testIsEven02(): Unit \ Assert =
    assertFalse(isEven(3))

@Test
def testSafeDivide01(): Unit \ Assert =
    assertOk(safeDivide(10, 2))

@Test
def testSafeDivide02(): Unit \ Assert =
    assertErr(safeDivide(10, 0))
```

テストを実行すると（例えば `flix test` で）、次のような結果が得られます：

```
Running 5 tests...

   PASS  testAdd01 1,4ms
   PASS  testIsEven01 312,5us
   PASS  testIsEven02 229,8us
   PASS  testSafeDivide01 366,0us
   PASS  testSafeDivide02 299,7us

Passed: 5, Failed: 0. Skipped: 0. Elapsed: 3,8ms.
```

## カスタムメッセージ付きのアサーション

ほとんどのアサーションには、カスタムエラーメッセージを指定できる `WithMsg` バリアントがあります。

```flix
use Assert.{assertEqWithMsg, assertTrueWithMsg, assertFalseWithMsg}

@Test
def testAdd01(): Unit \ Assert =
    assertEqWithMsg(expected = 5, add(2, 3), "addition should work")

@Test
def testIsEven01(): Unit \ Assert =
    assertTrueWithMsg(isEven(4), "4 should be even")

@Test
def testIsEven02(): Unit \ Assert =
    assertFalseWithMsg(isEven(3), "3 should be odd")
```

## `@Test` 関数のシグネチャ

`@Test` が付けられた関数は、次のいずれかのシグネチャを持たなければなりません：

```flix
@Test
def test01(): Unit = ...
def test02(): Unit \ Assert = ...
def test03(): Unit \ Assert + IO = ...
```

さらに、`@Test` 関数は、`@DefaultHandler` が存在する任意の代数エフェクトを使用できます。

<!--
# Test Framework

Flix comes with a built-in test framework. A test is a Flix function marked with
the `@Test` annotation. A test function must take no arguments and return
`Unit`.

The `Assert` module provides assertion functions for testing. Here are the most commonly used:

| Function                                       | Purpose                              |
|------------------------------------------------|--------------------------------------|
| `Assert.assertEq(expected = value, actual)`    | Assert equality between values       |
| `Assert.assertNeq(unexpected = value, actual)` | Assert inequality between values     |
| `Assert.assertTrue(cond)`                      | Assert condition is true             |
| `Assert.assertFalse(cond)`                     | Assert condition is false            |
| `Assert.assertSome(opt)`                       | Assert Option is Some                |
| `Assert.assertNone(opt)`                       | Assert Option is None                |
| `Assert.assertOk(res)`                         | Assert Result is Ok                  |
| `Assert.assertErr(res)`                        | Assert Result is Err                 |
| `Assert.assertEmpty(coll)`                     | Assert collection is empty           |
| `Assert.assertMemberOf(x, coll)`               | Assert element is in collection      |
| `Assert.fail(msg)`                             | Unconditionally fail with message    |
| `Assert.success(msg)`                          | Unconditionally succeed with message |

The `assertEq` and `assertNeq` functions require a labelled argument `expected` / `unexpected`.

Here is an example:

```flix
use Assert.{assertEq, assertTrue, assertFalse, assertOk, assertErr}

def add(x: Int32, y: Int32): Int32 = x + y

def isEven(x: Int32): Bool = Int32.modulo(x, 2) == 0

def safeDivide(x: Int32, y: Int32): Result[String, Int32] =
    if (y == 0) Err("Division by zero") else Ok(x / y)

@Test
def testAdd01(): Unit \ Assert =
    assertEq(expected = 5, add(2, 3))

@Test
def testIsEven01(): Unit \ Assert =
    assertTrue(isEven(4))

@Test
def testIsEven02(): Unit \ Assert =
    assertFalse(isEven(3))

@Test
def testSafeDivide01(): Unit \ Assert =
    assertOk(safeDivide(10, 2))

@Test
def testSafeDivide02(): Unit \ Assert =
    assertErr(safeDivide(10, 0))
```

Running the tests (e.g. with `flix test`) yields:

```
Running 5 tests...

   PASS  testAdd01 1,4ms
   PASS  testIsEven01 312,5us
   PASS  testIsEven02 229,8us
   PASS  testSafeDivide01 366,0us
   PASS  testSafeDivide02 299,7us

Passed: 5, Failed: 0. Skipped: 0. Elapsed: 3,8ms.
```

## Assertions with Custom Messages

Most assertions have `WithMsg` variants for custom error messages.

```flix
use Assert.{assertEqWithMsg, assertTrueWithMsg, assertFalseWithMsg}

@Test
def testAdd01(): Unit \ Assert =
    assertEqWithMsg(expected = 5, add(2, 3), "addition should work")

@Test
def testIsEven01(): Unit \ Assert =
    assertTrueWithMsg(isEven(4), "4 should be even")

@Test
def testIsEven02(): Unit \ Assert =
    assertFalseWithMsg(isEven(3), "3 should be odd")
```

## `@Test` Function Signatures

A function marked with `@Test` must have one of the following signatures:

```flix
@Test
def test01(): Unit = ...
def test02(): Unit \ Assert = ...
def test03(): Unit \ Assert + IO = ...
```

In addition, a `@Test` function may use any algebraic effect for which there is
a `@DefaultHandler`.
-->
