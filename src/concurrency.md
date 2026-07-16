# 構造化並行性

> 💡 **お知らせ**: このドキュメントはAIによって翻訳されています。表現に違和感がある場合は、[原文（英語）](https://doc.flix.dev/concurrency.html)を参照してください。

Flix は、Go と Rust に着想を得た、チャネルとプロセスによる CSP スタイルの並行処理をサポートしています。

## プロセスの生成

`spawn` キーワードを使って、プロセス(Process)を生成できます：

```flix
def main(): Unit \ IO = region rc {
    spawn println("Hello from thread") @ rc;
    println("Hello from main")
}
```

生成されたプロセスは、必ずリージョンに関連付けられます。リージョンは、それに関連付けられたすべてのプロセスが完了するまで終了しません：

```flix
def main(): Unit \ IO =
    region r1 {
        region r2 {
            spawn println("Hello from r1") @ r1;
            spawn println("Hello from r2") @ r2
        };
        println("r2 is now complete")
    };
    println("r1 is now complete")
```

これはつまり、Flix が*構造化並行性*をサポートしているということです。生成されたプロセスには、明確に定義された開始点と終了点があります。

## チャネルによる通信

プロセス間で通信するには、チャネルを使用します。*チャネル(Channel)*を使うと、2つ以上のプロセスが互いに不変（イミュータブル）なメッセージを送り合うことで、データを交換できます。

チャネルには、*バッファ付き(Buffered)*と*バッファなし(Unbuffered)*の2つの種類があります。チャネルは必ずリージョンに関連付けられます。

バッファ付きチャネルは、作成時に設定されるサイズを持ち、その数だけメッセージを保持できます。満杯のバッファ付きチャネルにプロセスがメッセージを入れようとすると、そのプロセスは空きができるまでブロックされます。逆に、空のチャネルからプロセスがメッセージを取り出そうとすると、そのプロセスはチャネルにメッセージが入れられるまでブロックされます。

バッファなしチャネルは、サイズ0のバッファ付きチャネルのように動作します。取り出し（get）と書き込み（put）が成立するためには、送信側から受信側へメッセージが渡されるまで、両方のプロセスがランデブー（ブロック）しなければなりません。

チャネルを介してメッセージを送受信する例を示します：

```flix
def main(): Int32 \ {Chan, NonDet, IO} = region rc {
    let (tx, rx) = Channel.unbuffered();
    spawn Channel.send(42, tx) @ rc;
    Channel.recv(rx)
}
```

ここで `main` 関数は、`Sender` チャネル `tx` と `Receiver` チャネル `rx` を返すバッファなしチャネルを作成し、`send` 関数を spawn して、チャネルからのメッセージを待ちます。

この例が示すように、チャネルは *Sender(センダー)* と *Receiver(レシーバー)* という2つのエンドポイントで構成されます。予想される通り、メッセージの送信は `Sender` からのみ、受信は `Receiver` からのみ行えます。

## チャネルに対する select

`select` 式を使うと、複数のチャネルの集まりからメッセージを受信できます。例えば：

```flix
def meow(tx: Sender[String]): Unit \ Chan =
    Channel.send("Meow!", tx)

def woof(tx: Sender[String]): Unit \ Chan =
    Channel.send("Woof!", tx)

def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let (tx1, rx1) = Channel.buffered(1);
    let (tx2, rx2) = Channel.buffered(1);
    spawn meow(tx1) @ rc;
    spawn woof(tx2) @ rc;
    select {
        case m <- recv(rx1) => m
        case m <- recv(rx2) => m
    } |> println
}
```

生産者・消費者（producer-consumer）やロードバランサーなど、多くの重要な並行処理パターンを `select` 式で表現できます。

### デフォルトケース付きの select

場合によっては、メッセージが届くまでブロックして、永遠に待ち続けるかもしれない状況を避けたいことがあります。そのような場合は、メッセージがすぐに利用できないときに代わりのアクションを取りたくなります。これは、以下に示すように*デフォルトケース(Default case)*で実現できます：

```flix
def main(): String \ {Chan, NonDet} = region rc {
    let (_, rx1) = Channel.buffered(1);
    let (_, rx2) = Channel.buffered(1);
    select {
        case _ <- recv(rx1) => "one"
        case _ <- recv(rx2) => "two"
        case _             => "default"
    }
}
```

ここでは、`r1` にも `r2` にもメッセージが送信されることはありません。`select` 式はすべてのケースを試し、どのチャネルも準備できていなければ、直ちにデフォルトケースを選択します。したがって、デフォルトケースを使うことで、`select` 式が永遠にブロックすることを防げます。

### タイムアウト付きの select

デフォルトケースの代わりに、*ティッカー(Ticker)*と*タイマー(Timer)*を使って、`select` 式の中であらかじめ定められた時間だけ待つこともできます。

例えば、次のプログラムには、チャネルにメッセージを送るまでに1分かかる遅い関数がありますが、`select` 式は `Channel.timeout` を利用して、`5` 秒だけ待って諦めるようになっています：

```flix
def slow(tx: Sender[String]): Unit \ {Chan, NonDet, IO} =
    let delay = Channel.timeout(60, Time.TimeUnit.Seconds);
    Channel.recv(delay);
    Channel.send("I am very slow", tx)

def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let (tx, rx) = Channel.buffered(1);
    spawn slow(tx) @ rc;
    let timeout = Channel.timeout(5, Time.TimeUnit.Seconds);
    select {
        case m <- recv(rx)       => m
        case _ <- recv(timeout)  => "timeout"
    } |> println
}
```

このプログラムは、5秒後に文字列 `"timeout"` を出力します。

### チャネルのエフェクト

お気づきかもしれませんが、チャネルを使うと `Chan` と `NonDet` というエフェクトが現れます。

チャネルに対するあらゆる操作は `Chan` エフェクトを持ちます。このエフェクトは、プログラムがチャネルのグローバルな状態を変更または参照していることを表します。

チャネルに対する `recv` 操作は `NonDet` エフェクトを持ちます。これは、受け取る値が一般には非決定的であり、スレッドスケジューラの選択に依存するためです。2つのスレッドが同時にチャネルへ値を送信する準備ができていることがあり、どちらが先に送信できるかはスケジューラ次第です。

<!--
# Structured Concurrency

Flix supports CSP-style concurrency with channels and
processes inspired by Go and Rust.

## Spawning Processes

We can spawn a process with the `spawn` keyword:

```flix
def main(): Unit \ IO = region rc {
    spawn println("Hello from thread") @ rc;
    println("Hello from main")
}
```

Spawned processes are always associated with a region; the region
will not exit until all the processes associated with it have completed:

```flix
def main(): Unit \ IO =
    region r1 {
        region r2 {
            spawn println("Hello from r1") @ r1;
            spawn println("Hello from r2") @ r2
        };
        println("r2 is now complete")
    };
    println("r1 is now complete")
```

This means that Flix supports _structured concurrency_; spawned
processes have clearly defined entry and exit points.

## Communicating with Channels

To communicate between processes we use channels.
A _channel_ allows two or more processes to exchange
data by sending immutable messages to each other.

A channel comes in one of two variants: _buffered_ or
_unbuffered_. Channels are always associated with a region.

A buffered channel has a size, set at creation time,
and can hold that many messages.
If a process attempts to put a message into a
buffered channel that is full, then the process is
blocked until space becomes available.
If, on the other hand, a process attempts to get a
message from an empty channel, the process is blocked
until a message is put into the channel.

An unbuffered channel works like a buffered channel
of size zero; for a get and a put to happen both
processes must rendezvous (block) until the message
is passed from sender to receiver.

Here is an example of sending and receiving a message
over a channel:

```flix
def main(): Int32 \ {Chan, NonDet, IO} = region rc {
    let (tx, rx) = Channel.unbuffered();
    spawn Channel.send(42, tx) @ rc;
    Channel.recv(rx)
}
```

Here the `main` function creates an unbuffered
channel which returns `Sender` `tx` and a `Receiver` `rx` channels,
spawns the `send` function, and waits
for a message from the channel.

As the example shows, a channel consists of two end points:
the _Sender_ and the _Receiver_. As one would expect,
messages can only be send using the `Sender`, and only
received using the `Receiver`.

## Selecting on Channels

We can use the `select` expression to receive a
message from a collection of channels.
For example:

```flix
def meow(tx: Sender[String]): Unit \ Chan =
    Channel.send("Meow!", tx)

def woof(tx: Sender[String]): Unit \ Chan =
    Channel.send("Woof!", tx)

def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let (tx1, rx1) = Channel.buffered(1);
    let (tx2, rx2) = Channel.buffered(1);
    spawn meow(tx1) @ rc;
    spawn woof(tx2) @ rc;
    select {
        case m <- recv(rx1) => m
        case m <- recv(rx2) => m
    } |> println
}
```

Many important concurrency patterns such as
producer-consumer and load balancers can be expressed
using the `select` expression.

### Selecting with Default

In some cases, we do not want to block until a
message arrives, potentially waiting forever.
Instead, we want to take some alternative action if
no message is readily available.
We can achieve this with a _default case_ as shown
below:

```flix
def main(): String \ {Chan, NonDet} = region rc {
    let (_, rx1) = Channel.buffered(1);
    let (_, rx2) = Channel.buffered(1);
    select {
        case _ <- recv(rx1) => "one"
        case _ <- recv(rx2) => "two"
        case _             => "default"
    }
}
```

Here a message is never sent to `r1` nor `r2`.
The `select` expression tries all cases, and if no
channel is ready, it immediately selects the default
case.
Hence using a default case prevents the `select`
expression from blocking forever.

### Selecting with Timeouts

As an alternative to a default case, we can use
_tickers_ and _timers_ to wait for pre-defined
periods of time inside a `select` expression.

For example, here is a program that has a slow
function that takes a minute to send a message on
a channel, but the `select` expression relies on
`Channel.timeout` to only wait `5` seconds before
giving up:

```flix
def slow(tx: Sender[String]): Unit \ {Chan, NonDet, IO} =
    let delay = Channel.timeout(60, Time.TimeUnit.Seconds);
    Channel.recv(delay);
    Channel.send("I am very slow", tx)

def main(): Unit \ {Chan, NonDet, IO} = region rc {
    let (tx, rx) = Channel.buffered(1);
    spawn slow(tx) @ rc;
    let timeout = Channel.timeout(5, Time.TimeUnit.Seconds);
    select {
        case m <- recv(rx)       => m
        case _ <- recv(timeout)  => "timeout"
    } |> println
}
```

This program prints the string `"timeout"` after five
seconds.

### The Effects of Channels

As you might have noticed, the effects `Chan` and `NonDet`
shows up when using channels. 

Any operation on channels has the `Chan` effect. This effect
says that the program is either modifying or accessing the global
state of channels.

The `recv` operation on a channel has the `NonDet` effect. This
is because the value you receive will, in the general case, be
non-deterministic, depending on the choices of the thread scheduler.
Two threads might be ready to send a value on a channel at the same
time, and it is up to the scheduler which one gets to send first. 
-->
