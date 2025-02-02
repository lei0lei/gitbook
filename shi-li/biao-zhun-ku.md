# 标准库more

## [标准库更多介绍](https://rustwiki.org/zh-CN/rust-by-example/std_misc.html#%E6%A0%87%E5%87%86%E5%BA%93%E6%9B%B4%E5%A4%9A%E4%BB%8B%E7%BB%8D) <a href="#biao-zhun-ku-geng-duo-jie-shao" id="biao-zhun-ku-geng-duo-jie-shao"></a>

标准库也提供了很多其他类型来支持某些功能，例如：

* 线程（Threads）
* 信道（Channels）
* 文件输入输出（File I/O）

这些内容在[原生类型](https://rustwiki.org/zh-CN/rust-by-example/primitives.html)之外进行了有效扩充。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/std_misc.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[原生类型](https://rustwiki.org/zh-CN/rust-by-example/primitives.html) 和 [标准库类型](https://rustwiki.org/zh-CN/std/)

## [线程](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads.html#%E7%BA%BF%E7%A8%8B) <a href="#xian-cheng" id="xian-cheng"></a>

Rust 通过 `spawn` 函数提供了创建本地操作系统（native OS）线程的机制，该函数的参数是一个通过值捕获变量的闭包（moving closure）。

```
use std::thread;

static NTHREADS: i32 = 10;

// 这是主（`main`）线程
fn main() {
    // 提供一个 vector 来存放所创建的子线程（children）。
    let mut children = vec![];

    for i in 0..NTHREADS {
        // 启动（spin up）另一个线程
        children.push(thread::spawn(move || {
            println!("this is thread number {}", i)
        }));
    }

    for child in children {
        // 等待线程结束。返回一个结果。
        let _ = child.join();
    }
}

```

这些线程由操作系统调度（schedule）。

### [测试实例：map-reduce](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads/testcase_mapreduce.html#%E6%B5%8B%E8%AF%95%E5%AE%9E%E4%BE%8Bmap-reduce) <a href="#ce-shi-shi-li-mapreduce" id="ce-shi-shi-li-mapreduce"></a>

Rust 使数据的并行化处理非常简单，在 Rust 中你无需面对并行处理的很多传统难题。

标准库提供了开箱即用的线程类型，把它和 Rust 的所有权概念与别名规则结合起来，可以自动地避免数据竞争（data race）。

当某状态对某线程是可见的，别名规则（即一个可变引用 XOR 一些只读引用。译注：XOR 是异或的意思，即「二者仅居其一」）就自动地避免了别的线程对它的操作。（当需要同步处理时，请使用 `Mutex` 或 `Channel` 这样的同步类型。）

在本例中，我们将会计算一堆数字中每一位的和。我们将把它们分成几块，放入不同的线程。每个线程会把自己那一块数字的每一位加起来，之后我们再把每个线程提供的结果再加起来。

注意到，虽然我们在线程之间传递了引用，但 Rust 理解我们是在传递只读的引用，因此不会发生数据竞争等不安全的事情。另外，因为我们把数据块 `move` 到了线程中，Rust 会保证数据存活至线程退出，因此不会产生悬挂指针。

```
use std::thread;

// 这是 `main` 线程
fn main() {

    // 这是我们要处理的数据。
    // 我们会通过线程实现 map-reduce 算法，从而计算每一位的和
    // 每个用空白符隔开的块都会分配给单独的线程来处理
    //
    // 试一试：插入空格，看看输出会怎样变化！
    let data = "86967897737416471853297327050364959
11861322575564723963297542624962850
70856234701860851907960690014725639
38397966707106094172783238747669219
52380795257888236525459303330302837
58495327135744041048897885734297812
69920216438980873548808413720956532
16278424637452589860345374828574668";

    // 创建一个向量，用于储存将要创建的子线程
    let mut children = vec![];

    /*************************************************************************
     * "Map" 阶段
     *
     * 把数据分段，并进行初始化处理
     ************************************************************************/

    // 把数据分段，每段将会单独计算
    // 每段都是完整数据的一个引用（&str）
    let chunked_data = data.split_whitespace();

    // 对分段的数据进行迭代。
    // .enumerate() 会把当前的迭代计数与被迭代的元素以元组 (index, element)
    // 的形式返回。接着立即使用 “解构赋值” 将该元组解构成两个变量，
    // `i` 和 `data_segment`。
    for (i, data_segment) in chunked_data.enumerate() {
        println!("data segment {} is \"{}\"", i, data_segment);

        // 用单独的线程处理每一段数据
        //
        // spawn() 返回新线程的句柄（handle），我们必须拥有句柄，
        // 才能获取线程的返回值。
        //
        // 'move || -> u32' 语法表示该闭包：
        // * 没有参数（'||'）
        // * 会获取所捕获变量的所有权（'move'）
        // * 返回无符号 32 位整数（'-> u32'）
        //
        // Rust 可以根据闭包的内容推断出 '-> u32'，所以我们可以不写它。
        //
        // 试一试：删除 'move'，看看会发生什么
        children.push(thread::spawn(move || -> u32 {
            // 计算该段的每一位的和：
            let result = data_segment
                        // 对该段中的字符进行迭代..
                        .chars()
                        // ..把字符转成数字..
                        .map(|c| c.to_digit(10).expect("should be a digit"))
                        // ..对返回的数字类型的迭代器求和
                        .sum();

            // println! 会锁住标准输出，这样各线程打印的内容不会交错在一起
            println!("processed segment {}, result={}", i, result);

            // 不需要 “return”，因为 Rust 是一种 “表达式语言”，每个代码块中
            // 最后求值的表达式就是代码块的值。
            result

        }));
    }


    /*************************************************************************
     * "Reduce" 阶段
     *
     * 收集中间结果，得出最终结果
     ************************************************************************/

    // 把每个线程产生的中间结果收入一个新的向量中
    let mut intermediate_sums = vec![];
    for child in children {
        // 收集每个子线程的返回值
        let intermediate_sum = child.join().unwrap();
        intermediate_sums.push(intermediate_sum);
    }

    // 把所有中间结果加起来，得到最终结果
    //
    // 我们用 “涡轮鱼” 写法 ::<> 来为 sum() 提供类型提示。
    //
    // 试一试：不使用涡轮鱼写法，而是显式地指定 intermediate_sums 的类型
    let final_result = intermediate_sums.iter().sum::<u32>();

    println!("Final sum result: {}", final_result);
}



```

#### [作业](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads/testcase_mapreduce.html#%E4%BD%9C%E4%B8%9A) <a href="#zuo-ye" id="zuo-ye"></a>

根据用户输入的数据来决定线程的数量是不明智的。如果用户输入的数据中有一大堆空格怎么办？我们**真的**想要创建 2000 个线程吗？

请修改程序，使得数据总是被分成有限数目的段，这个数目是由程序开头的静态常量决定的。

#### [参见:](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads/testcase_mapreduce.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

* [线程](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads.html)
* [向量](https://rustwiki.org/zh-CN/rust-by-example/std/vec.html)和[迭代器](https://rustwiki.org/zh-CN/rust-by-example/trait/iter.html)
* [闭包](https://rustwiki.org/zh-CN/rust-by-example/fn/closures.html)、[移动](https://rustwiki.org/zh-CN/rust-by-example/scope/move.html)语义和[`move`闭包](https://rustwiki.org/zh-CN/book/ch13-01-closures.html#closures-can-capture-their-environment)
* [解构](https://rustwiki.org/zh-CN/book/ch18-03-pattern-syntax.html#%E8%A7%A3%E6%9E%84%E5%B9%B6%E5%88%86%E8%A7%A3%E5%80%BC)赋值
* 使用[涡轮鱼写法](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#method.collect)帮助类型推断
* [unwrap vs. expect](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap.html)
* [枚举类型](https://rustwiki.org/zh-CN/book/ch03-05-control-flow.html#%E4%BD%BF%E7%94%A8-loop-%E9%87%8D%E5%A4%8D%E6%89%A7%E8%A1%8C%E4%BB%A3%E7%A0%81)

## [通道](https://rustwiki.org/zh-CN/rust-by-example/std_misc/channels.html#%E9%80%9A%E9%81%93) <a href="#tong-dao" id="tong-dao"></a>

Rust 为线程之间的通信提供了异步的通道（`channel`）。通道允许两个端点之间信息的单向流动：`Sender`（发送端） 和 `Receiver`（接收端）。

```
use std::sync::mpsc::{Sender, Receiver};
use std::sync::mpsc;
use std::thread;

static NTHREADS: i32 = 3;

fn main() {
    // 通道有两个端点：`Sender<T>` 和 `Receiver<T>`，其中 `T` 是要发送
    // 的消息的类型（类型标注是可选的）
    let (tx, rx): (Sender<i32>, Receiver<i32>) = mpsc::channel();

    for id in 0..NTHREADS {
        // sender 端可被复制
        let thread_tx = tx.clone();

        // 每个线程都将通过通道来发送它的 id
        thread::spawn(move || {
            // 被创建的线程取得 `thread_tx` 的所有权
            // 每个线程都把消息放在通道的消息队列中
            thread_tx.send(id).unwrap();

            // 发送是一个非阻塞（non-blocking）操作，线程将在发送完消息后
            // 会立即继续进行
            println!("thread {} finished", id);
        });
    }

    // 所有消息都在此处被收集
    let mut ids = Vec::with_capacity(NTHREADS as usize);
    for _ in 0..NTHREADS {
        // `recv` 方法从通道中拿到一个消息
        // 若无可用消息的话，`recv` 将阻止当前线程
        ids.push(rx.recv());
    }

    // 显示消息被发送的次序
    println!("{:?}", ids);
}

```

[路径](https://rustwiki.org/zh-CN/rust-by-example/std_misc/path.html#%E8%B7%AF%E5%BE%84)

`Path` 结构体代表了底层文件系统的文件路径。`Path` 分为两种：`posix::Path`，针对类 UNIX 系统；以及 `windows::Path`，针对 Windows。prelude 会选择并输出符合平台类型的 `Path` 种类。

> 译注：prelude 是 Rust 自动地在每个程序中导入的一些通用的东西，这样我们就不必每写 一个程序就手动导入一番。

`Path` 可从 `OsStr` 类型创建，并且它提供数种方法，用于获取路径指向的文件/目录的信息。

注意 `Path` 在内部并不是用 UTF-8 字符串表示的，而是存储为若干字节（`Vec<u8>`）的 vector。因此，将 `Path` 转化成 `&str` 并非零开销的（free），且可能失败（因此它返回一个 `Option`）。

```
use std::path::Path;fn main() {    // 从 `&'static str` 创建一个 `Path`    let path = Path::new(".");    // `display` 方法返回一个可显示（showable）的结构体    let display = path.display();    // `join` 使用操作系统特定的分隔符来合并路径到一个字节容器，并返回新的路径    let new_path = path.join("a").join("b");    // 将路径转换成一个字符串切片    match new_path.to_str() {        None => panic!("new path is not a valid UTF-8 sequence"),        Some(s) => println!("new path is {}", s),    }}
```

记得看看其他的 `Path` 方法（`posix::Path` 或 `windows::Path` 的），还有 `Metadata` 结构体类型。

#### [参见](https://rustwiki.org/zh-CN/rust-by-example/std_misc/path.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[OsStr](https://rustwiki.org/zh-CN/std/ffi/struct.OsStr.html) 和 [Metadata](https://rustwiki.org/zh-CN/std/fs/struct.Metadata.html)。[\
](https://rustwiki.org/zh-CN/rust-by-example/std_misc/threads/testcase_mapreduce.html)

## [文件输入输出（I/O）](https://rustwiki.org/zh-CN/rust-by-example/std_misc/file.html#%E6%96%87%E4%BB%B6%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BAio) <a href="#wen-jian-shu-ru-shu-chu-io" id="wen-jian-shu-ru-shu-chu-io"></a>

`File` 结构体表示一个被打开的文件（它包裹了一个文件描述符），并赋予了对所表示的文件的读写能力。

由于在进行文件 I/O（输入/输出）操作时可能出现各种错误，因此 `File` 的所有方法都返回 `io::Result<T>` 类型，它是 `Result<T, io::Error>` 的别名。

这使得所有 I/O 操作的失败都变成**显式的**。借助这点，程序员可以看到所有的失败路径，并被鼓励主动地处理这些情形。

### &#x20;[打开文件 `open`](https://rustwiki.org/zh-CN/rust-by-example/std_misc/file/open.html#%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6-open)

`open` 静态方法能够以只读模式（read-only mode）打开一个文件。

`File` 拥有资源，即文件描述符（file descriptor），它会在自身被 `drop` 时关闭文件。

```
use std::fs::File;
use std::io::prelude::*;
use std::path::Path;

fn main() {
    // 创建指向所需的文件的路径
    let path = Path::new("hello.txt");
    let display = path.display();

    // 以只读方式打开路径，返回 `io::Result<File>`
    let mut file = match File::open(&path) {
        // `io::Error` 的 `description` 方法返回一个描述错误的字符串。
        Err(why) => panic!("couldn't open {}: {:?}", display, why),
        Ok(file) => file,
    };

    // 读取文件内容到一个字符串，返回 `io::Result<usize>`
    let mut s = String::new();
    match file.read_to_string(&mut s) {
        Err(why) => panic!("couldn't read {}: {:?}", display, why),
        Ok(_) => print!("{} contains:\n{}", display, s),
    }

    // `file` 离开作用域，并且 `hello.txt` 文件将被关闭。
}

```

下面是所希望的成功的输出：

```bash
$ echo "Hello World!" > hello.txt
$ rustc open.rs && ./open
hello.txt contains:
Hello World!
```

（我们鼓励您在不同的失败条件下测试前面的例子：hello.txt 不存在，或 hello.txt 不可读，等等。）

### [创建文件 `create`](https://rustwiki.org/zh-CN/rust-by-example/std_misc/file/create.html#%E5%88%9B%E5%BB%BA%E6%96%87%E4%BB%B6-create) <a href="#chuang-jian-wen-jian-create" id="chuang-jian-wen-jian-create"></a>

`create` 静态方法以只写模式（write-only mode）打开一个文件。若文件已经存在，则旧内容将被销毁。否则，将创建一个新文件。

```rust
static LOREM_IPSUM: &'static str =
"Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
";

use std::io::prelude::*;
use std::fs::File;
use std::path::Path;

fn main() {
    let path = Path::new("out/lorem_ipsum.txt");
    let display = path.display();

    // 以只写模式打开文件，返回 `io::Result<File>`
    let mut file = match File::create(&path) {
        Err(why) => panic!("couldn't create {}: {:?}", display, why),
        Ok(file) => file,
    };

    // 将 `LOREM_IPSUM` 字符串写进 `file`，返回 `io::Result<()>`
    match file.write_all(LOREM_IPSUM.as_bytes()) {
        Err(why) => {
            panic!("couldn't write to {}: {:?}", display, why)
        },
        Ok(_) => println!("successfully wrote to {}", display),
    }
}
```

下面是预期成功的输出：

```bash
$ mkdir out
$ rustc create.rs && ./create
successfully wrote to out/lorem_ipsum.txt
$ cat out/lorem_ipsum.txt
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
```

（和前面例子一样，我们鼓励你在失败条件下测试这个例子。）

还有一个更通用的 `open_mode` 方法，这能够以其他方式来来打开文件，如：read+write（读 + 写），追加（append），等等。

### [读取行](https://rustwiki.org/zh-CN/rust-by-example/std_misc/file/read_lines.html#%E8%AF%BB%E5%8F%96%E8%A1%8C) <a href="#du-qu-hang" id="du-qu-hang"></a>

方法 `lines()` 在文件的行上返回一个迭代器。

`File::open` 需要一个泛型 `AsRef<Path>`。这正是 `read_lines()` 期望的输入。

```rust
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;

fn main() {
    // 在生成输出之前，文件主机必须存在于当前路径中
    if let Ok(lines) = read_lines("./hosts") {
        // 使用迭代器，返回一个（可选）字符串
        for line in lines {
            if let Ok(ip) = line {
                println!("{}", ip);
            }      
        }   
    }
}

// 输出包裹在 Result 中以允许匹配错误，
// 将迭代器返回给文件行的读取器（Reader）。
fn read_lines<P>(filename: P) -> io::Result<io::Lines<io::BufReader<File>>>
where P: AsRef<Path>, {
    let file = File::open(filename)?;
    Ok(io::BufReader::new(file).lines())
}

```

运行此程序将一行行将内容打印出来。

```bash
$ echo -e "127.0.0.1\n192.168.0.1\n" > hosts
$ rustc read_lines.rs && ./read_lines
127.0.0.1
192.168.0.1
```

这个过程比在内存中创建 `String` 更有效，特别是处理更大的文件。



## [子进程](https://rustwiki.org/zh-CN/rust-by-example/std_misc/process.html#%E5%AD%90%E8%BF%9B%E7%A8%8B) <a href="#zi-jin-cheng" id="zi-jin-cheng"></a>

`process::Output` 结构体表示已结束的子进程（child process）的输出，而 `process::Command` 结构体是一个进程创建者（process builder）。

```
use std::process::Command;

fn main() {
    let output = Command::new("rustc")
        .arg("--version")
        .output().unwrap_or_else(|e| {
            panic!("failed to execute process: {}", e)
    });

    if output.status.success() {
        let s = String::from_utf8_lossy(&output.stdout);

        print!("rustc succeeded and stdout was:\n{}", s);
    } else {
        let s = String::from_utf8_lossy(&output.stderr);

        print!("rustc failed and stderr was:\n{}", s);
    }
}

```

（再试试上面的例子，给 `rustc` 命令传入一个错误的 flag）

### [管道](https://rustwiki.org/zh-CN/rust-by-example/std_misc/process/pipe.html#%E7%AE%A1%E9%81%93) <a href="#guan-dao" id="guan-dao"></a>

`std::Child` 结构体代表了一个正在运行的子进程，它暴露了 `stdin`（标准输入），`stdout`（标准输出）和 `stderr`（标准错误）句柄，从而可以通过管道与所代表的进程交互。

```
use std::io::prelude::*;
use std::process::{Command, Stdio};

static PANGRAM: &'static str =
"the quick brown fox jumped over the lazy dog\n";

fn main() {
    // 启动 `wc` 命令
    let process = match Command::new("wc")
                                .stdin(Stdio::piped())
                                .stdout(Stdio::piped())
                                .spawn() {
        Err(why) => panic!("couldn't spawn wc: {:?}", why),
        Ok(process) => process,
    };

    // 将字符串写入 `wc` 的 `stdin`。
    //
    // `stdin` 拥有 `Option<ChildStdin>` 类型，不过我们已经知道这个实例不为空值，
    // 因而可以直接 `unwrap 它。
    match process.stdin.unwrap().write_all(PANGRAM.as_bytes()) {
        Err(why) => panic!("couldn't write to wc stdin: {:?}", why),
        Ok(_) => println!("sent pangram to wc"),
    }

    // 因为 `stdin` 在上面调用后就不再存活，所以它被 `drop` 了，管道也被关闭。
    //
    // 这点非常重要，因为否则 `wc` 就不会开始处理我们刚刚发送的输入。

    // `stdout` 字段也拥有 `Option<ChildStdout>` 类型，所以必需解包。
    let mut s = String::new();
    match process.stdout.unwrap().read_to_string(&mut s) {
        Err(why) => panic!("couldn't read wc stdout: {:?}", why),
        Ok(_) => print!("wc responded with:\n{}", s),
    }
}

```

[\
](https://rustwiki.org/zh-CN/rust-by-example/std_misc/process.html)

### [等待](https://rustwiki.org/zh-CN/rust-by-example/std_misc/process/wait.html#%E7%AD%89%E5%BE%85) <a href="#deng-dai" id="deng-dai"></a>

如果你想等待一个 `process::Child` 完成，就必须调用 `Child::wait`，这会返回一个 `process::ExitStatus`。

```rust
use std::process::Command;

fn main() {
    let mut child = Command::new("sleep").arg("5").spawn().unwrap();
    let _result = child.wait().unwrap();

    println!("reached end of main");
}
```

```bash
$ rustc wait.rs && ./wait
reached end of main
# `wait` keeps running for 5 seconds
# `sleep 5` command ends, and then our `wait` program finishes
```

## [文件系统操作](https://rustwiki.org/zh-CN/rust-by-example/std_misc/fs.html#%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%93%8D%E4%BD%9C) <a href="#wen-jian-xi-tong-cao-zuo" id="wen-jian-xi-tong-cao-zuo"></a>

`std::io::fs` 模块包含几个处理文件系统的函数。

```rust
use std::fs;
use std::fs::{File, OpenOptions};
use std::io;
use std::io::prelude::*;
use std::os::unix;
use std::path::Path;

// `% cat path` 的简单实现
fn cat(path: &Path) -> io::Result<String> {
    let mut f = File::open(path)?;
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}

// `% echo s > path` 的简单实现
fn echo(s: &str, path: &Path) -> io::Result<()> {
    let mut f = File::create(path)?;

    f.write_all(s.as_bytes())
}

// `% touch path` 的简单实现（忽略已存在的文件）
fn touch(path: &Path) -> io::Result<()> {
    match OpenOptions::new().create(true).write(true).open(path) {
        Ok(_) => Ok(()),
        Err(e) => Err(e),
    }
}

fn main() {
    println!("`mkdir a`");
    // 创建一个目录，返回 `io::Result<()>`
    match fs::create_dir("a") {
        Err(why) => println!("! {:?}", why.kind()),
        Ok(_) => {},
    }

    println!("`echo hello > a/b.txt`");
    // 前面的匹配可以用 `unwrap_or_else` 方法简化
    echo("hello", &Path::new("a/b.txt")).unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
    });

    println!("`mkdir -p a/c/d`");
    // 递归地创建一个目录，返回 `io::Result<()>`
    fs::create_dir_all("a/c/d").unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
    });

    println!("`touch a/c/e.txt`");
    touch(&Path::new("a/c/e.txt")).unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
    });

    println!("`ln -s ../b.txt a/c/b.txt`");
    // 创建一个符号链接，返回 `io::Resutl<()>`
    if cfg!(target_family = "unix") {
        unix::fs::symlink("../b.txt", "a/c/b.txt").unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
        });
    }

    println!("`cat a/c/b.txt`");
    match cat(&Path::new("a/c/b.txt")) {
        Err(why) => println!("! {:?}", why.kind()),
        Ok(s) => println!("> {}", s),
    }

    println!("`ls a`");
    // 读取目录的内容，返回 `io::Result<Vec<Path>>`
    match fs::read_dir("a") {
        Err(why) => println!("! {:?}", why.kind()),
        Ok(paths) => for path in paths {
            println!("> {:?}", path.unwrap().path());
        },
    }

    println!("`rm a/c/e.txt`");
    // 删除一个文件，返回 `io::Result<()>`
    fs::remove_file("a/c/e.txt").unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
    });

    println!("`rmdir a/c/d`");
    // 移除一个空目录，返回 `io::Result<()>`
    fs::remove_dir("a/c/d").unwrap_or_else(|why| {
        println!("! {:?}", why.kind());
    });
}
```

下面是所期望的成功的输出：

```bash
$ rustc fs.rs && ./fs
`mkdir a`
`echo hello > a/b.txt`
`mkdir -p a/c/d`
`touch a/c/e.txt`
`ln -s ../b.txt a/c/b.txt`
`cat a/c/b.txt`
> hello
`ls a`
> "a/b.txt"
> "a/c"
`rm a/c/e.txt`
`rmdir a/c/d`
```

且 `a` 目录的最终状态为：

```
$ tree a
a
|-- b.txt
`-- c
    `-- b.txt -> ../b.txt

1 directory, 2 files
```

另一种定义 `cat` 函数的方式是使用 `?` 标记：

```rust
fn cat(path: &Path) -> io::Result<String> {
    let mut f = File::open(path)?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/std_misc/fs.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`cfg!`](https://rustwiki.org/zh-CN/rust-by-example/attribute/cfg.html)

## [程序参数](https://rustwiki.org/zh-CN/rust-by-example/std_misc/arg.html#%E7%A8%8B%E5%BA%8F%E5%8F%82%E6%95%B0) <a href="#cheng-xu-can-shu" id="cheng-xu-can-shu"></a>

### [标准库](https://rustwiki.org/zh-CN/rust-by-example/std_misc/arg.html#%E6%A0%87%E5%87%86%E5%BA%93) <a href="#biao-zhun-ku" id="biao-zhun-ku"></a>

命令行参数可使用 `std::env::args` 进行接收，这将返回一个迭代器，该迭代器会对每个参数举出一个字符串。

```
use std::env;fn main() {    let args: Vec<String> = env::args().collect();    // 第一个参数是调用本程序的路径    println!("My path is {}.", args[0]);    // 其余的参数是被传递给程序的命令行参数。    // 请这样调用程序：    //   $ ./args arg1 arg2    println!("I got {:?} arguments: {:?}.", args.len() - 1, &args[1..]);}
```

```bash
$ ./args 1 2 3
My path is ./args.
I got 3 arguments: ["1", "2", "3"].
```

### [crate](https://rustwiki.org/zh-CN/rust-by-example/std_misc/arg.html#crate) <a href="#crate" id="crate"></a>

另外，也有很多 crate 提供了编写命令行应用的额外功能。[Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/app.html#ex-clap-basic) 展示了使用最流行的命令行参数 crate，即 `clap` 的最佳实践。

### [参数解析](https://rustwiki.org/zh-CN/rust-by-example/std_misc/arg/matching.html#%E5%8F%82%E6%95%B0%E8%A7%A3%E6%9E%90) <a href="#can-shu-jie-xi" id="can-shu-jie-xi"></a>

可以用模式匹配来解析简单的参数：

```
use std::env;

fn increase(number: i32) {
    println!("{}", number + 1);
}

fn decrease(number: i32) {
    println!("{}", number - 1);
}

fn help() {
    println!("usage:
match_args <string>
    Check whether given string is the answer.
match_args {{increase|decrease}} <integer>
    Increase or decrease given integer by one.");
}

fn main() {
    let args: Vec<String> = env::args().collect();

    match args.len() {
        // 没有传入参数
        1 => {
            println!("My name is 'match_args'. Try passing some arguments!");
        },
        // 一个传入参数
        2 => {
            match args[1].parse() {
                Ok(42) => println!("This is the answer!"),
                _ => println!("This is not the answer."),
            }
        },
        // 传入一条命令和一个参数
        3 => {
            let cmd = &args[1];
            let num = &args[2];
            // 解析数字
            let number: i32 = match num.parse() {
                Ok(n) => {
                    n
                },
                Err(_) => {
                    println!("error: second argument not an integer");
                    help();
                    return;
                },
            };
            // 解析命令
            match &cmd[..] {
                "increase" => increase(number),
                "decrease" => decrease(number),
                _ => {
                    println!("error: invalid command");
                    help();
                },
            }
        },
        // 所有其他情况
        _ => {
            // 显示帮助信息
            help();
        }
    }
}

```

```bash
$ ./match_args Rust
This is not the answer.
$ ./match_args 42
This is the answer!
$ ./match_args do something
error: second argument not an integer
usage:
match_args <string>
    Check whether given string is the answer.
match_args {increase|decrease} <integer>
    Increase or decrease given integer by one.
$ ./match_args do 42
error: invalid command
usage:
match_args <string>
    Check whether given string is the answer.
match_args {increase|decrease} <integer>
    Increase or decrease given integer by one.
$ ./match_args increase 42
43
```

## [外部语言函数接口](https://rustwiki.org/zh-CN/rust-by-example/std_misc/ffi.html#%E5%A4%96%E9%83%A8%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0%E6%8E%A5%E5%8F%A3) <a href="#wai-bu-yu-yan-han-shu-jie-kou" id="wai-bu-yu-yan-han-shu-jie-kou"></a>

Rust 提供了到 C 语言库的外部语言函数接口（Foreign Function Interface，FFI）。外部语言函数必须在一个 `extern` 代码块中声明，且该代码块要带有一个包含库名称的 `#[link]` 属性。

```rust
use std::fmt;

// 这个 extern 代码块链接到 libm 库
#[link(name = "m")]
extern {
    // 这个外部函数用于计算单精度复数的平方根
    fn csqrtf(z: Complex) -> Complex;

    // 这个用来计算单精度复数的复变余弦
    fn ccosf(z: Complex) -> Complex;
}

// 由于调用其他语言的函数被认为是不安全的，我们通常会给它们写一层安全的封装
fn cos(z: Complex) -> Complex {
    unsafe { ccosf(z) }
}

fn main() {
    // z = -1 + 0i
    let z = Complex { re: -1., im: 0. };

    // 调用外部语言函数是不安全操作
    let z_sqrt = unsafe { csqrtf(z) };

    println!("the square root of {:?} is {:?}", z, z_sqrt);

    // 调用不安全操作的安全的 API 封装
    println!("cos({:?}) = {:?}", z, cos(z));
}

// 单精度复数的最简实现
#[repr(C)]
#[derive(Clone, Copy)]
struct Complex {
    re: f32,
    im: f32,
}

impl fmt::Debug for Complex {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        if self.im < 0. {
            write!(f, "{}-{}i", self.re, -self.im)
        } else {
            write!(f, "{}+{}i", self.re, self.im)
        }
    }
}
```



