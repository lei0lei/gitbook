# 多线程并发编程

## [多线程并发编程](https://course.rs/advance/concurrency-with-threads/intro.html#%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B) <a href="#duo-xian-cheng-bing-fa-bian-cheng" id="duo-xian-cheng-bing-fa-bian-cheng"></a>

安全和高效的处理并发是 Rust 语言的主要目标之一。随着现代处理器的核心数不断增加，并发和并行已经成为日常编程不可或缺的一部分，甚至于 Go 语言已经将并发简化到一个 `go` 关键字就可以。

可惜的是，在 Rust 中由于语言设计理念、安全、性能的多方面考虑，并没有采用 Go 语言大道至简的方式，而是选择了多线程与 `async/await` 相结合，优点是可控性更强、性能更高，缺点是复杂度并不低，当然这也是系统级语言的应有选择：**使用复杂度换取可控性和性能**。

不过，大家也不用担心，本书的目标就是降低 Rust 使用门槛，这个门槛自然也包括如何在 Rust 中进行异步并发编程，我们将从多线程以及 `async/await` 两个方面去深入浅出地讲解，首先，从本章的多线程开始。

## [并发和并行](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E5%B9%B6%E5%8F%91%E5%92%8C%E5%B9%B6%E8%A1%8C) <a href="#bing-fa-he-bing-hang" id="bing-fa-he-bing-hang"></a>

> 并发是同一时间应对多件事情的能力 - [Rob Pike](https://en.wikipedia.org/wiki/Rob\_Pike)

并行和并发其实并不难，但是也给一些用户造成了困扰，因此我们专门开辟一个章节，用于讲清楚这两者的区别。

`Erlang` 之父 [`Joe Armstrong`](https://en.wikipedia.org/wiki/Joe\_Armstrong\_\(programmer\))（伟大的异步编程先驱，开创一个时代的殿堂级计算机科学家，我还犹记得当年刚学到 `Erlang` 时的震撼，respect！）用一张 5 岁小孩都能看懂的图片解释了并发与并行的区别：

![](https://pic1.zhimg.com/80/f37dd89173715d0e21546ea171c8a915\_1440w.png)

上图很直观的体现了：

* **并发(Concurrent)** 是多个队列使用同一个咖啡机，然后两个队列轮换着使用（未必是 1:1 轮换，也可能是其它轮换规则），最终每个人都能接到咖啡
* **并行(Parallel)** 是每个队列都拥有一个咖啡机，最终也是每个人都能接到咖啡，但是效率更高，因为同时可以有两个人在接咖啡

当然，我们还可以对比下串行：只有一个队列且仅使用一台咖啡机，前面哪个人接咖啡时突然发呆了几分钟，后面的人就只能等他结束才能继续接。可能有读者有疑问了，从图片来看，并发也存在这个问题啊，前面的人发呆了几分钟不接咖啡怎么办？很简单，另外一个队列的人把他推开就行了，自己队友不能在背后开枪，但是其它队的可以:)

在正式开始之前，先给出一个结论：**并发和并行都是对“多任务”处理的描述，其中并发是轮流处理，而并行是同时处理**。

### [CPU 多核](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#cpu-%E5%A4%9A%E6%A0%B8) <a href="#cpu-duo-he" id="cpu-duo-he"></a>

现在的个人计算机动辄拥有十来个核心（M1 Max/Intel 12 代），如果使用串行的方式那真是太低效了，因此我们把各种任务简单分成多个队列，每个队列都交给一个 CPU 核心去执行，当某个 CPU 核心没有任务时，它还能去其它核心的队列中偷任务（真·老黄牛），这样就实现了并行化处理。

[**单核心并发**](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E5%8D%95%E6%A0%B8%E5%BF%83%E5%B9%B6%E5%8F%91)

那问题来了，在早期只有一个 CPU 核心时，我们的任务是怎么处理的呢？其实聪明的读者应该已经想到，是的，并发解君愁。当然，这里还得提到操作系统的多线程，正是操作系统多线程 + CPU 核心，才实现了现代化的多任务操作系统。

在 OS 级别，多线程负责管理我们的任务队列，你可以简单认为一个线程管理着一个任务队列，然后线程之间还能根据空闲度进行任务调度。我们的程序只会跟 OS 线程打交道，并不关心 CPU 到底有多少个核心，真正关心的只是 OS，当线程把任务交给 CPU 核心去执行时，如果只有一个 CPU 核心，那么它就只能同时处理一个任务。

相信大家都看出来了：**CPU 核心**对应的是上图的咖啡机，而**多个线程的任务队列**就对应的多个排队的队列，由于终受限于 CPU 核心数，每个队列每次只会有一个任务被处理。

和排队一样，假如某个任务执行时间过长，就会导致用户界面的假死（相信使用 Windows 的同学或多或少都碰到过假死的问题）， 那么就需要 CPU 的任务调度了（真实 CPU 的调度很复杂，我们这里做了简化），有一个调度器会按照某些条件从队列中选择任务进行执行，并且当一个任务执行时间过长时，会强行切换该任务到后台中（或者放入任务队列，真实情况很复杂！），去执行新的任务。

不断这样的快速任务切换，对用户而言就实现了表面上的多任务同时处理，但是实际上最终也只有一个 CPU 核心在不停的工作。

因此并发的关键在于：**快速轮换处理不同的任务**，给用户带来所有任务同时在运行的假象。

[**多核心并行**](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E5%A4%9A%E6%A0%B8%E5%BF%83%E5%B9%B6%E8%A1%8C)

当 CPU 核心增多到 `N` 时，那么同一时间就能有 `N` 个任务被处理，那么我们的并行度就是 `N`，相应的处理效率也变成了单核心的 `N` 倍（实际情况并没有这么高）。

[**多核心并发**](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E5%A4%9A%E6%A0%B8%E5%BF%83%E5%B9%B6%E5%8F%91)

当核心增多到 `N` 时，操作系统同时在进行的任务肯定远不止 `N` 个，这些任务将被放入 `M` 个线程队列中，接着交给 `N` 个 CPU 核心去执行，最后实现了 `M:N` 的处理模型，在这种情况下，**并发与并行是同时在发生的，所有用户任务从表面来看都在并发的运行，但实际上，同一时刻只有 `N` 个任务能被同时并行的处理**。

看到这里，相信大家已经明白两者的区别，那么我们下面给出一个正式的定义（该定义摘选自<<并发的艺术>>）。

### [正式的定义](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E6%AD%A3%E5%BC%8F%E7%9A%84%E5%AE%9A%E4%B9%89) <a href="#zheng-shi-de-ding-yi" id="zheng-shi-de-ding-yi"></a>

如果某个系统支持两个或者多个动作的**同时存在**，那么这个系统就是一个并发系统。如果某个系统支持两个或者多个动作**同时执行**，那么这个系统就是一个并行系统。并发系统与并行系统这两个定义之间的关键差异在于 **“存在”** 这个词。

在并发程序中可以同时拥有两个或者多个线程。这意味着，如果程序在单核处理器上运行，那么这两个线程将交替地换入或者换出内存。这些线程是 **同时“存在”** 的——每个线程都处于执行过程中的某个状态。如果程序能够并行执行，那么就一定是运行在多核处理器上。此时，程序中的每个线程都将分配到一个独立的处理器核上，因此可以同时运行。

相信你已经能够得出结论——**“并行”概念是“并发”概念的一个子集**。也就是说，你可以编写一个拥有多个线程或者进程的并发程序，但如果没有多核处理器来执行这个程序，那么就不能以并行方式来运行代码。因此，凡是在求解单个问题时涉及多个执行流程的编程模式或者执行行为，都属于并发编程的范畴。

### [编程语言的并发模型](https://course.rs/advance/concurrency-with-threads/concurrency-parallelism.html#%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80%E7%9A%84%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B) <a href="#bian-cheng-yu-yan-de-bing-fa-mo-xing" id="bian-cheng-yu-yan-de-bing-fa-mo-xing"></a>

如果大家学过其它语言的多线程，可能就知道不同语言对于线程的实现可能大相径庭：

* 由于操作系统提供了创建线程的 API，因此部分语言会直接调用该 API 来创建线程，因此最终程序内的线程数和该程序占用的操作系统线程数相等，一般称之为**1:1 线程模型**，例如 Rust。
* 还有些语言在内部实现了自己的线程模型（绿色线程、协程），程序内部的 M 个线程最后会以某种映射方式使用 N 个操作系统线程去运行，因此称之为**M:N 线程模型**，其中 M 和 N 并没有特定的彼此限制关系。一个典型的代表就是 Go 语言。
* 还有些语言使用了 Actor 模型，基于消息传递进行并发，例如 Erlang 语言。

总之，每一种模型都有其优缺点及选择上的权衡，而 Rust 在设计时考虑的权衡就是运行时(Runtime)。出于 Rust 的系统级使用场景，且要保证调用 C 时的极致性能，它最终选择了尽量小的运行时实现。

> 运行时是那些会被打包到所有程序可执行文件中的 Rust 代码，根据每个语言的设计权衡，运行时虽然有大有小（例如 Go 语言由于实现了协程和 GC，运行时相对就会更大一些），但是除了汇编之外，每个语言都拥有它。小运行时的其中一个好处在于最终编译出的可执行文件会相对较小，同时也让该语言更容易被其它语言引入使用。

而绿色线程/协程的实现会显著增大运行时的大小，因此 Rust 只在标准库中提供了 `1:1` 的线程模型，如果你愿意牺牲一些性能来换取更精确的线程控制以及更小的线程上下文切换成本，那么可以选择 Rust 中的 `M:N` 模型，这些模型由三方库提供了实现，例如大名鼎鼎的 `tokio`。

在了解了并发和并行后，我们可以正式开始 Rust 的多线程之旅。



## [使用线程](https://course.rs/advance/concurrency-with-threads/thread.html#%E4%BD%BF%E7%94%A8%E7%BA%BF%E7%A8%8B) <a href="#shi-yong-xian-cheng" id="shi-yong-xian-cheng"></a>

放在十年前，多线程编程可能还是一个少数人才掌握的核心概念，但是在今天，随着编程语言的不断发展，多线程、多协程、Actor 等并发编程方式已经深入人心，同时多线程编程的门槛也在不断降低，本章节我们来看看在 Rust 中该如何使用多线程。

### [多线程编程的风险](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%BC%96%E7%A8%8B%E7%9A%84%E9%A3%8E%E9%99%A9) <a href="#duo-xian-cheng-bian-cheng-de-feng-xian" id="duo-xian-cheng-bian-cheng-de-feng-xian"></a>

由于多线程的代码是同时运行的，因此我们无法保证线程间的执行顺序，这会导致一些问题：

* 竞态条件(race conditions)，多个线程以非一致性的顺序同时访问数据资源
* 死锁(deadlocks)，两个线程都想使用某个资源，但是又都在等待对方释放资源后才能使用，结果最终都无法继续执行
* 一些因为多线程导致的很隐晦的 BUG，难以复现和解决

虽然 Rust 已经通过各种机制减少了上述情况的发生，但是依然无法完全避免上述情况，因此我们在编程时需要格外的小心，同时本书也会列出多线程编程时常见的陷阱，让你提前规避可能的风险。

### [创建线程](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B) <a href="#chuang-jian-xian-cheng" id="chuang-jian-xian-cheng"></a>

使用 `thread::spawn` 可以创建线程：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}

```

有几点值得注意：

* 线程内部的代码使用闭包来执行
* `main` 线程一旦结束，程序就立刻结束，因此需要保持它的存活，直到其它子线程完成自己的任务
* `thread::sleep` 会让当前线程休眠指定的时间，随后其它线程会被调度运行（上一节并发与并行中有简单介绍过），因此就算你的电脑只有一个 CPU 核心，该程序也会表现的如同多 CPU 核心一般，这就是并发！

来看看输出：

```console
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 4 from the main thread!
hi number 5 from the spawned thread!
```

如果多运行几次，你会发现好像每次输出会不太一样，因为：虽说线程往往是轮流执行的，但是这一点无法被保证！线程调度的方式往往取决于你使用的操作系统。总之，**千万不要依赖线程的执行顺序**。

### [等待子线程的结束](https://course.rs/advance/concurrency-with-threads/thread.html#%E7%AD%89%E5%BE%85%E5%AD%90%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%BB%93%E6%9D%9F) <a href="#deng-dai-zi-xian-cheng-de-jie-shu" id="deng-dai-zi-xian-cheng-de-jie-shu"></a>

上面的代码你不但可能无法让子线程从 1 顺序打印到 10，而且可能打印的数字会变少，因为主线程会提前结束，导致子线程也随之结束，更过分的是，如果当前系统繁忙，甚至该子线程还没被创建，主线程就已经结束了！

因此我们需要一个方法，让主线程安全、可靠地等所有子线程完成任务后，再 kill self：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..5 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}

```

通过调用 `handle.join`，可以让当前线程阻塞，直到它等待的子线程的结束，在上面代码中，由于 `main` 线程会被阻塞，因此它直到子线程结束后才会输出自己的 `1..5`：

```console
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

以上输出清晰的展示了线程阻塞的作用，如果你将 `handle.join` 放置在 `main` 线程中的 `for` 循环后面，那就是另外一个结果：两个线程交替输出。

### [在线程闭包中使用 move](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%9C%A8%E7%BA%BF%E7%A8%8B%E9%97%AD%E5%8C%85%E4%B8%AD%E4%BD%BF%E7%94%A8-move) <a href="#zai-xian-cheng-bi-bao-zhong-shi-yong-move" id="zai-xian-cheng-bi-bao-zhong-shi-yong-move"></a>

在[闭包](https://course.rs/advance/functional-programing/closure.html#move-%E5%92%8C-fn)章节中，有讲过 `move` 关键字在闭包中的使用可以让该闭包拿走环境中某个值的所有权，同样地，你可以使用 `move` 来将所有权从一个线程转移到另外一个线程。

首先，来看看在一个线程中直接使用另一个线程中的数据会如何：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}

```

以上代码在子线程的闭包中捕获了环境中的 `v` 变量，来看看结果：

```console
error[E0373]: closure may outlive the current function, but it borrows `v`, which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
note: function requires argument type to outlive `'static`
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here's a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

其实代码本身并没有什么问题，问题在于 Rust 无法确定新的线程会活多久（多个线程的结束顺序并不是固定的），所以也无法确定新线程所引用的 `v` 是否在使用过程中一直合法：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}

```

大家要记住，线程的启动时间点和结束时间点是不确定的，因此存在一种可能，当主线程执行完， `v` 被释放掉时，新的线程很可能还没有结束甚至还没有被创建成功，此时新线程对 `v` 的引用立刻就不再合法！

好在报错里进行了提示：``to force the closure to take ownership of v (and any other referenced variables), use the `move` keyword``，让我们使用 `move` 关键字拿走 `v` 的所有权即可：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();

    // 下面代码会报错borrow of moved value: `v`
    // println!("{:?}",v);
}

```

如上所示，很简单的代码，而且 Rust 的所有权机制保证了数据使用上的安全：`v` 的所有权被转移给新的线程后，`main` 线程将无法继续使用：最后一行代码将报错。

### [线程是如何结束的](https://course.rs/advance/concurrency-with-threads/thread.html#%E7%BA%BF%E7%A8%8B%E6%98%AF%E5%A6%82%E4%BD%95%E7%BB%93%E6%9D%9F%E7%9A%84) <a href="#xian-cheng-shi-ru-he-jie-shu-de" id="xian-cheng-shi-ru-he-jie-shu-de"></a>

之前我们提到 `main` 线程是程序的主线程，一旦结束，则程序随之结束，同时各个子线程也将被强行终止。那么有一个问题，如果父线程不是 `main` 线程，那么父线程的结束会导致什么？自生自灭还是被干掉？

在系统编程中，操作系统提供了直接杀死线程的接口，简单粗暴，但是 Rust 并没有提供这样的接口，原因在于，粗暴地终止一个线程可能会导致资源没有释放、状态混乱等不可预期的结果，一向以安全自称的 Rust，自然不会砸自己的饭碗。

那么 Rust 中线程是如何结束的呢？答案很简单：线程的代码执行完，线程就会自动结束。但是如果线程中的代码不会执行完呢？那么情况可以分为两种进行讨论：

* 线程的任务是一个循环 IO 读取，任务流程类似：IO 阻塞，等待读取新的数据 -> 读到数据，处理完成 -> 继续阻塞等待 ··· -> 收到 socket 关闭的信号 -> 结束线程，在此过程中，绝大部分时间线程都处于阻塞的状态，因此虽然看上去是循环，CPU 占用其实很小，也是网络服务中最最常见的模型
* 线程的任务是一个循环，里面没有任何阻塞，包括休眠这种操作也没有，此时 CPU 很不幸的会被跑满，而且你如果没有设置终止条件，该线程将持续跑满一个 CPU 核心，并且不会被终止，直到 `main` 线程的结束

第一情况很常见，我们来模拟看看第二种情况：

```rust
use std::thread;
use std::time::Duration;
fn main() {
    // 创建一个线程A
    let new_thread = thread::spawn(move || {
        // 再创建一个线程B
        thread::spawn(move || {
            loop {
                println!("I am a new thread.");
            }
        })
    });

    // 等待新创建的线程执行完成
    new_thread.join().unwrap();
    println!("Child thread is finish!");

    // 睡眠一段时间，看子线程创建的子线程是否还在运行
    thread::sleep(Duration::from_millis(100));
}

```

以上代码中，`main` 线程创建了一个新的线程 `A`，同时该新线程又创建了一个新的线程 `B`，可以看到 `A` 线程在创建完 `B` 线程后就立即结束了，而 `B` 线程则在不停地循环输出。

从之前的线程结束规则，我们可以猜测程序将这样执行：`A` 线程结束后，由它创建的 `B` 线程仍在疯狂输出，直到 `main` 线程在 100 毫秒后结束。如果你把该时间增加到几十秒，就可以看到你的 CPU 核心 100% 的盛况了-,-

### [多线程的性能](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E6%80%A7%E8%83%BD) <a href="#duo-xian-cheng-de-xing-neng" id="duo-xian-cheng-de-xing-neng"></a>

下面我们从多个方面来看看多线程的性能大概是怎么样的。

[**创建线程的性能**](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B%E7%9A%84%E6%80%A7%E8%83%BD)

据不精确估算，创建一个线程大概需要 0.24 毫秒，随着线程的变多，这个值会变得更大，因此线程的创建耗时并不是不可忽略的，只有当真的需要处理一个值得用线程去处理的任务时，才使用线程，一些鸡毛蒜皮的任务，就无需创建线程了。

[**创建多少线程合适**](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%88%9B%E5%BB%BA%E5%A4%9A%E5%B0%91%E7%BA%BF%E7%A8%8B%E5%90%88%E9%80%82)

因为 CPU 的核心数限制，当任务是 CPU 密集型时，就算线程数超过了 CPU 核心数，也并不能帮你获得更好的性能，因为每个线程的任务都可以轻松让 CPU 的某个核心跑满，既然如此，让线程数等于 CPU 核心数是最好的。

但是当你的任务大部分时间都处于阻塞状态时，就可以考虑增多线程数量，这样当某个线程处于阻塞状态时，会被切走，进而运行其它的线程，典型就是网络 IO 操作，我们可以为每一个进来的用户连接创建一个线程去处理，该连接绝大部分时间都是处于 IO 读取阻塞状态，因此有限的 CPU 核心完全可以处理成百上千的用户连接线程，但是事实上，对于这种网络 IO 情况，一般都不再使用多线程的方式了，毕竟操作系统的线程数是有限的，意味着并发数也很容易达到上限，而且过多的线程也会导致线程上下文切换的代价过大，使用 `async/await` 的 `M:N` 并发模型，就没有这个烦恼。

[**多线程的开销**](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%BC%80%E9%94%80)

下面的代码是一个无锁实现(CAS)的 `Hashmap` 在多线程下的使用：

```rust

#![allow(unused)]
fn main() {
for i in 0..num_threads {
    let ht = Arc::clone(&ht);

    let handle = thread::spawn(move || {
        for j in 0..adds_per_thread {
            let key = thread_rng().gen::<u32>();
            let value = thread_rng().gen::<u32>();
            ht.set_item(key, value);
        }
    });

    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}
}

```

按理来说，既然是无锁实现了，那么锁的开销应该几乎没有，性能会随着线程数的增加接近线性增长，但是真的是这样吗？

下图是该代码在 `48` 核机器上的运行结果：

![](https://pic3.zhimg.com/80/v2-af225672de09c0e377023f5f39dd87eb\_1440w.png)

从图上可以明显的看出：吞吐并不是线性增长，尤其从 `16` 核开始，甚至开始肉眼可见的下降，这是为什么呢？

限于书本的篇幅有限，我们只能给出大概的原因：

* 虽然是无锁，但是内部是 CAS 实现，大量线程的同时访问，会让 CAS 重试次数大幅增加
* 线程过多时，CPU 缓存的命中率会显著下降，同时多个线程竞争一个 CPU Cache-line 的情况也会经常发生
* 大量读写可能会让内存带宽也成为瓶颈
* 读和写不一样，无锁数据结构的读往往可以很好地线性增长，但是写不行，因为写竞争太大

总之，多线程的开销往往是在锁、数据竞争、缓存失效上，这些限制了现代化软件系统随着 CPU 核心的增多性能也线性增加的野心。

### [线程屏障(Barrier)](https://course.rs/advance/concurrency-with-threads/thread.html#%E7%BA%BF%E7%A8%8B%E5%B1%8F%E9%9A%9Cbarrier) <a href="#xian-cheng-ping-zhang-barrier" id="xian-cheng-ping-zhang-barrier"></a>

在 Rust 中，可以使用 `Barrier` 让多个线程都执行到某个点后，才继续一起往后执行：

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn main() {
    let mut handles = Vec::with_capacity(6);
    let barrier = Arc::new(Barrier::new(6));

    for _ in 0..6 {
        let b = barrier.clone();
        handles.push(thread::spawn(move|| {
            println!("before wait");
            b.wait();
            println!("after wait");
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }
}

```

上面代码，我们在线程打印出 `before wait` 后增加了一个屏障，目的就是等所有的线程都打印出**before wait**后，各个线程再继续执行：

```console
before wait
before wait
before wait
before wait
before wait
before wait
after wait
after wait
after wait
after wait
after wait
after wait
```

### [线程局部变量(Thread Local Variable)](https://course.rs/advance/concurrency-with-threads/thread.html#%E7%BA%BF%E7%A8%8B%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8Fthread-local-variable) <a href="#xian-cheng-ju-bu-bian-liang-threadlocalvariable" id="xian-cheng-ju-bu-bian-liang-threadlocalvariable"></a>

对于多线程编程，线程局部变量在一些场景下非常有用，而 Rust 通过标准库和三方库对此进行了支持。

[**标准库 thread\_local**](https://course.rs/advance/concurrency-with-threads/thread.html#%E6%A0%87%E5%87%86%E5%BA%93-thread\_local)

使用 `thread_local` 宏可以初始化线程局部变量，然后在线程内部使用该变量的 `with` 方法获取变量值：

```rust

#![allow(unused)]
fn main() {
use std::cell::RefCell;
use std::thread;

thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

FOO.with(|f| {
    assert_eq!(*f.borrow(), 1);
    *f.borrow_mut() = 2;
});

// 每个线程开始时都会拿到线程局部变量的FOO的初始值
let t = thread::spawn(move|| {
    FOO.with(|f| {
        assert_eq!(*f.borrow(), 1);
        *f.borrow_mut() = 3;
    });
});

// 等待线程完成
t.join().unwrap();

// 尽管子线程中修改为了3，我们在这里依然拥有main线程中的局部值：2
FOO.with(|f| {
    assert_eq!(*f.borrow(), 2);
});
}

```

上面代码中，`FOO` 即是我们创建的**线程局部变量**，每个新的线程访问它时，都会使用它的初始值作为开始，各个线程中的 `FOO` 值彼此互不干扰。注意 `FOO` 使用 `static` 声明为生命周期为 `'static` 的静态变量。

可以注意到，线程中对 `FOO` 的使用是通过借用的方式，但是若我们需要每个线程独自获取它的拷贝，最后进行汇总，就有些强人所难了。

你还可以在结构体中使用线程局部变量：

```rust
use std::cell::RefCell;

struct Foo;
impl Foo {
    thread_local! {
        static FOO: RefCell<usize> = RefCell::new(0);
    }
}

fn main() {
    Foo::FOO.with(|x| println!("{:?}", x));
}

```

或者通过引用的方式使用它:

```rust

#![allow(unused)]
fn main() {
use std::cell::RefCell;
use std::thread::LocalKey;

thread_local! {
    static FOO: RefCell<usize> = RefCell::new(0);
}
struct Bar {
    foo: &'static LocalKey<RefCell<usize>>,
}
impl Bar {
    fn constructor() -> Self {
        Self {
            foo: &FOO,
        }
    }
}
}

```

[**三方库 thread-local**](https://course.rs/advance/concurrency-with-threads/thread.html#%E4%B8%89%E6%96%B9%E5%BA%93-thread-local)

除了标准库外，一位大神还开发了 [thread-local](https://github.com/Amanieu/thread\_local-rs) 库，它允许每个线程持有值的独立拷贝：

```rust

#![allow(unused)]
fn main() {
use thread_local::ThreadLocal;
use std::sync::Arc;
use std::cell::Cell;
use std::thread;

let tls = Arc::new(ThreadLocal::new());

// 创建多个线程
for _ in 0..5 {
    let tls2 = tls.clone();
    thread::spawn(move || {
        // 将计数器加1
        let cell = tls2.get_or(|| Cell::new(0));
        cell.set(cell.get() + 1);
    }).join().unwrap();
}

// 一旦所有子线程结束，收集它们的线程局部变量中的计数器值，然后进行求和
let tls = Arc::try_unwrap(tls).unwrap();
let total = tls.into_iter().fold(0, |x, y| x + y.get());

// 和为5
assert_eq!(total, 5);
}

```

该库不仅仅使用了值的拷贝，而且还能自动把多个拷贝汇总到一个迭代器中，最后进行求和，非常好用。

### [用条件控制线程的挂起和执行](https://course.rs/advance/concurrency-with-threads/thread.html#%E7%94%A8%E6%9D%A1%E4%BB%B6%E6%8E%A7%E5%88%B6%E7%BA%BF%E7%A8%8B%E7%9A%84%E6%8C%82%E8%B5%B7%E5%92%8C%E6%89%A7%E8%A1%8C) <a href="#yong-tiao-jian-kong-zhi-xian-cheng-de-gua-qi-he-zhi-hang" id="yong-tiao-jian-kong-zhi-xian-cheng-de-gua-qi-he-zhi-hang"></a>

条件变量(Condition Variables)经常和 `Mutex` 一起使用，可以让线程挂起，直到某个条件发生后再继续执行：

```rust
use std::thread;
use std::sync::{Arc, Mutex, Condvar};

fn main() {
    let pair = Arc::new((Mutex::new(false), Condvar::new()));
    let pair2 = pair.clone();

    thread::spawn(move|| {
        let (lock, cvar) = &*pair2;
        let mut started = lock.lock().unwrap();
        println!("changing started");
        *started = true;
        cvar.notify_one();
    });

    let (lock, cvar) = &*pair;
    let mut started = lock.lock().unwrap();
    while !*started {
        started = cvar.wait(started).unwrap();
    }

    println!("started changed");
}

```

上述代码流程如下：

1. `main` 线程首先进入 `while` 循环，调用 `wait` 方法挂起等待子线程的通知，并释放了锁 `started`
2. 子线程获取到锁，并将其修改为 `true`，然后调用条件变量的 `notify_one` 方法来通知主线程继续执行

### [只被调用一次的函数](https://course.rs/advance/concurrency-with-threads/thread.html#%E5%8F%AA%E8%A2%AB%E8%B0%83%E7%94%A8%E4%B8%80%E6%AC%A1%E7%9A%84%E5%87%BD%E6%95%B0) <a href="#zhi-bei-tiao-yong-yi-ci-de-han-shu" id="zhi-bei-tiao-yong-yi-ci-de-han-shu"></a>

有时，我们会需要某个函数在多线程环境下只被调用一次，例如初始化全局变量，无论是哪个线程先调用函数来初始化，都会保证全局变量只会被初始化一次，随后的其它线程调用就会忽略该函数：

```rust
use std::thread;
use std::sync::Once;

static mut VAL: usize = 0;
static INIT: Once = Once::new();

fn main() {
    let handle1 = thread::spawn(move || {
        INIT.call_once(|| {
            unsafe {
                VAL = 1;
            }
        });
    });

    let handle2 = thread::spawn(move || {
        INIT.call_once(|| {
            unsafe {
                VAL = 2;
            }
        });
    });

    handle1.join().unwrap();
    handle2.join().unwrap();

    println!("{}", unsafe { VAL });
}

```

代码运行的结果取决于哪个线程先调用 `INIT.call_once` （虽然代码具有先后顺序，但是线程的初始化顺序并无法被保证！因为线程初始化是异步的，且耗时较久），若 `handle1` 先，则输出 `1`，否则输出 `2`。

**call\_once 方法**

执行初始化过程一次，并且只执行一次。

如果当前有另一个初始化过程正在运行，线程将阻止该方法被调用。

当这个函数返回时，保证一些初始化已经运行并完成，它还保证由执行的闭包所执行的任何内存写入都能被其他线程在这时可靠地观察到。

### [总结](https://course.rs/advance/concurrency-with-threads/thread.html#%E6%80%BB%E7%BB%93) <a href="#zong-jie" id="zong-jie"></a>

[Rust 的线程模型](https://course.rs/advance/concurrency-with-threads/intro.html)是 `1:1` 模型，因为 Rust 要保持尽量小的运行时。

我们可以使用 `thread::spawn` 来创建线程，创建出的多个线程之间并不存在执行顺序关系，因此代码逻辑千万不要依赖于线程间的执行顺序。

`main` 线程若是结束，则所有子线程都将被终止，如果希望等待子线程结束后，再结束 `main` 线程，你需要使用创建线程时返回的句柄的 `join` 方法。

在线程中无法直接借用外部环境中的变量值，因为新线程的启动时间点和结束时间点是不确定的，所以 Rust 无法保证该线程中借用的变量在使用过程中依然是合法的。你可以使用 `move` 关键字将变量的所有权转移给新的线程，来解决此问题。

父线程结束后，子线程仍在持续运行，直到子线程的代码运行完成或者 `main` 线程的结束。

\






















