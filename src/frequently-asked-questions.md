# よくある質問

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/frequently-asked-questions.html)を参照してください。

## Flix は定数をサポートしていますか？

はい、とも言えますし、いいえ、とも言えます。Flix はトップレベル定数(Top-level constant)をサポートしていません。ただし、引数を取らない純粋関数を宣言することはできます：

```flix
def pi(): Float64 = 3.14f64
```

Flix コンパイラは、このような定数をインライン化します。

一度だけ実行したい高価な計算がある場合は、必要な場所で計算し、明示的に引き回すようにしてください。Flix がトップレベル定数をサポートしていないのは、main より前にいかなるコードも実行されるべきではない、という原則に反するためです。

<!--
# Frequently Asked Questions

## Does Flix supports constants?

Yes and no. Flix does not support top-level constants. You can, however, declare
a pure function which takes zero arguments:

```flix
def pi(): Float64 = 3.14f64
```

The Flix compiler will inline such constants. 

If you have an expensive computation that you want to perform once, you should
compute it where needed and explicitly pass it around. Flix does not support
top-level constants because they violate the principle that no code should be
executed before main.
-->
