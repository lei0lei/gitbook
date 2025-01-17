# helloworld

这是传统的 Hello World 程序的源码。

```
// 这是注释内容，将会被编译器忽略掉
// 可以单击那边的按钮 "Run" 来测试这段代码 ->
// 若想用键盘操作，可以使用快捷键 "Ctrl + Enter" 来运行

// 这段代码支持编辑，你可以自由地修改代码！
// 通过单击 "Reset" 按钮可以使代码恢复到初始状态 ->

// 这是主函数
fn main() {
    // 调用编译生成的可执行文件时，这里的语句将被运行。

    // 将文本打印到控制台
    println!("Hello World!");
}

```

`println!` 是一个[**宏**](https://rustwiki.org/zh-CN/rust-by-example/macros.html)（macros），可以将文本输出到控制台（console）。

使用 Rust 的编译器 `rustc` 可以从源程序生成可执行文件：

```bash
$ rustc hello.rs
```

使用 `rustc` 编译后将得到可执行文件 `hello`。

```bash
$ ./hello
Hello World!
```

#### [动手试一试](https://rustwiki.org/zh-CN/rust-by-example/hello.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

单击上面的 "Run" 按钮并观察输出结果。然后增加一行代码，再一次使用宏 `println!`，得到下面结果：

```
Hello World!
I'm a Rustacean!
```

## 注释 <a href="#zhu-shi" id="zhu-shi"></a>

注释对任何程序都不可缺少，同样 Rust 支持几种不同的注释方式。

* **普通注释**，其内容将被编译器忽略掉：
  * `// 单行注释，注释内容直到行尾。`
  * `/* 块注释，注释内容一直到结束分隔符。 */`
* **文档注释**，其内容将被解析成 HTML 帮助[文档](https://rustwiki.org/zh-CN/rust-by-example/meta/doc.html)：
  * `/// 为接下来的项生成帮助文档。`
  * `//! 为注释所属于的项（译注：如 crate、模块或函数）生成帮助文档。`

<pre class="language-rust"><code class="lang-rust">fn main() {
    // 这是行注释的例子
    // 注意有两个斜线在本行的开头
    // 在这里面的所有内容都不会被编译器读取

<strong>    // println!("Hello, world!");
</strong>
    // 请运行一下，你看到结果了吗？现在请将上述语句的两条斜线删掉，并重新运行。

    /*
     * 这是另外一种注释——块注释。一般而言，行注释是推荐的注释格式，
     * 不过块注释在临时注释大块代码特别有用。/* 块注释可以 /* 嵌套, */ */
     * 所以只需很少按键就可注释掉这些 main() 函数中的行。/*/*/* 自己试试！*/*/*/
     */

    /*
    注意，上面的例子中纵向都有 `*`，这只是一种风格，实际上这并不是必须的。
    */

    // 观察块注释是如何简单地对表达式进行修改的，行注释则不能这样。
    // 删除注释分隔符将会改变结果。
    let x = 5 + /* 90 + */ 5;
    println!("Is `x` 10 or 100? x = {}", x);
}

</code></pre>

\
格式化输出
-----

打印操作由 [`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/) 里面所定义的一系列[宏](https://rustwiki.org/zh-CN/rust-by-example/macros.html)来处理，包括：

* `format!`：将格式化文本写到[字符串](https://rustwiki.org/zh-CN/rust-by-example/std/str.html)。
* `print!`：与 `format!` 类似，但将文本输出到控制台（io::stdout）。
* `println!`: 与 `print!` 类似，但输出结果追加一个换行符。
* `eprint!`：与 `print!` 类似，但将文本输出到标准错误（io::stderr）。
* `eprintln!`：与 `eprint!` 类似，但输出结果追加一个换行符。

这些宏都以相同的做法解析文本。有个额外优点是格式化的正确性会在编译时检查。

```rust
fn main() {
    // 通常情况下，`{}` 会被任意变量内容所替换。
    // 变量内容会转化成字符串。
    println!("{} days", 31);

    // 不加后缀的话，31 就自动成为 i32 类型。
    // 你可以添加后缀来改变 31 的类型（例如使用 31i64 声明 31 为 i64 类型）。

    // 用变量替换字符串有多种写法。
    // 比如可以使用位置参数。
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");

    // 可以使用命名参数。
    println!("{subject} {verb} {object}",
             object="the lazy dog",
             subject="the quick brown fox",
             verb="jumps over");

    // 可以在 `:` 后面指定特殊的格式。
    println!("{} of {:b} people know binary, the other half don't", 1, 2);

    // 你可以按指定宽度来右对齐文本。
    // 下面语句输出 "     1"，5 个空格后面连着 1。
    println!("{number:>width$}", number=1, width=6);

    // 你可以在数字左边补 0。下面语句输出 "000001"。
    println!("{number:>0width$}", number=1, width=6);

    // println! 会检查使用到的参数数量是否正确。
    println!("My name is {0}, {1} {0}", "Bond");
    // 改正 ^ 补上漏掉的参数："James"

    // 创建一个包含单个 `i32` 的结构体（structure）。命名为 `Structure`。
    #[allow(dead_code)]
    struct Structure(i32);

    // 但是像结构体这样的自定义类型需要更复杂的方式来处理。
    // 下面语句无法运行。
    println!("This struct `{}` won't print...", Structure(3));
    // 改正 ^ 注释掉此行。
}
rururu
```

[`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/) 包含多种 [`trait`](https://rustwiki.org/zh-CN/rust-by-example/trait.html)（特质）来控制文字显示，其中重要的两种 trait 的基本形式如下：

* `fmt::Debug`：使用 `{:?}` 标记。格式化文本以供调试使用。
* `fmt::Display`：使用 `{}` 标记。以更优雅和友好的风格来格式化文本。

上例使用了 `fmt::Display`，因为标准库提供了那些类型的实现。若要打印自定义类型的文本，需要更多的步骤。

#### [动手试一试](https://rustwiki.org/zh-CN/rust-by-example/hello/print.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

* 改正上面代码中的两个错误（见 “改正”），使它可以没有错误地运行。
* 再用一个 `println!` 宏，通过控制显示的小数位数来打印：`Pi is roughly 3.142`（Pi 约等于 3.142）。为了达到练习目的，使用 `let pi = 3.141592` 作为 Pi 的近似值（提示：设置小数位的显示格式可以参考文档 [`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/)）。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/hello/print.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/), [`macros`](https://rustwiki.org/zh-CN/rust-by-example/macros.html), [`struct`](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html) 和 [`trait`](https://rustwiki.org/zh-CN/rust-by-example/trait.html)

## [调试（Debug）](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_debug.html#%E8%B0%83%E8%AF%95debug) <a href="#tiao-shi-debug" id="tiao-shi-debug"></a>

所有的类型，若想用 `std::fmt` 的格式化打印，都要求实现至少一个可打印的 `traits`。仅有一些类型提供了自动实现，比如 `std` 库中的类型。所有其他类型都**必须**手动实现。

`fmt::Debug` 这个 `trait` 使这项工作变得相当简单。所有类型都能推导（`derive`，即自动创建）`fmt::Debug` 的实现。但是 `fmt::Display` 需要手动实现。

```rust

#![allow(unused)]
fn main() {
// 这个结构体不能使用 `fmt::Display` 或 `fmt::Debug` 来进行打印。
struct UnPrintable(i32);

// `derive` 属性会自动创建所需的实现，使这个 `struct` 能使用 `fmt::Debug` 打印。
#[derive(Debug)]
struct DebugPrintable(i32);
}

```

所有 `std` 库类型都天生可以使用 `{:?}` 来打印：

```rust
// 推导 `Structure` 的 `fmt::Debug` 实现。
// `Structure` 是一个包含单个 `i32` 的结构体。
#[derive(Debug)]
struct Structure(i32);

// 将 `Structure` 放到结构体 `Deep` 中。然后使 `Deep` 也能够打印。
#[derive(Debug)]
struct Deep(Structure);

fn main() {
    // 使用 `{:?}` 打印和使用 `{}` 类似。
    println!("{:?} months in a year.", 12);
    println!("{1:?} {0:?} is the {actor:?} name.",
             "Slater",
             "Christian",
             actor="actor's");

    // `Structure` 也可以打印！
    println!("Now {:?} will print!", Structure(3));
    
    // 使用 `derive` 的一个问题是不能控制输出的形式。
    // 假如我只想展示一个 `7` 怎么办？
    println!("Now {:?} will print!", Deep(Structure(7)));
}

```

所以 `fmt::Debug` 确实使这些内容可以打印，但是牺牲了一些美感。Rust 也通过 `{:#?}` 提供了 “美化打印” 的功能：

```rust
#[derive(Debug)]
struct Person<'a> {
    name: &'a str,
    age: u8
}

fn main() {
    let name = "Peter";
    let age = 27;
    let peter = Person { name, age };

    // 美化打印
    println!("{:#?}", peter);
}

```

你可以通过手动实现 `fmt::Display` 来控制显示效果。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_debug.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[attributes](https://rustwiki.org/zh-CN/reference/attributes.html), [`derive`](https://rustwiki.org/zh-CN/rust-by-example/trait/derive.html), [`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/) 和 [`struct`](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html)

## [显示（Display）](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display.html#%E6%98%BE%E7%A4%BAdisplay) <a href="#xian-shi-display" id="xian-shi-display"></a>

`fmt::Debug` 通常看起来不太简洁，因此自定义输出的外观经常是更可取的。这需要通过手动实现 [`fmt::Display`](https://rustwiki.org/zh-CN/std/fmt/) 来做到。`fmt::Display` 采用 `{}` 标记。实现方式看起来像这样：

```rust

#![allow(unused)]
fn main() {
// （使用 `use`）导入 `fmt` 模块使 `fmt::Display` 可用
use std::fmt;

// 定义一个结构体，咱们会为它实现 `fmt::Display`。以下是个简单的元组结构体
// `Structure`，包含一个 `i32` 元素。
struct Structure(i32);

// 为了使用 `{}` 标记，必须手动为类型实现 `fmt::Display` trait。
impl fmt::Display for Structure {
    // 这个 trait 要求 `fmt` 使用与下面的函数完全一致的函数签名
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 仅将 self 的第一个元素写入到给定的输出流 `f`。返回 `fmt:Result`，此
        // 结果表明操作成功或失败。注意 `write!` 的用法和 `println!` 很相似。
        write!(f, "{}", self.0)
    }
}
}

```

`fmt::Display` 的效果可能比 `fmt::Debug` 简洁，但对于 `std` 库来说，这就有一个问题。模棱两可的类型该如何显示呢？举个例子，假设标准库对所有的 `Vec<T>` 都实现了同一种输出样式，那么它应该是哪种样式？下面两种中的一种吗？

* `Vec<path>`：`/:/etc:/home/username:/bin`（使用 `:` 分割）
* `Vec<number>`：`1,2,3`（使用 `,` 分割）

我们没有这样做，因为没有一种合适的样式适用于所有类型，标准库也并不擅自规定一种样式。对于 `Vec<T>` 或其他任意泛型容器（generic container），`fmt::Display` 都没有实现。因此在这些泛型的情况下要用 `fmt::Debug`。

这并不是一个问题，因为对于任何**非**泛型的**容器**类型， `fmt::Display` 都能够实现。

```rust
use std::fmt; // 导入 `fmt`

// 带有两个数字的结构体。推导出 `Debug`，以便与 `Display` 的输出进行比较。
#[derive(Debug)]
struct MinMax(i64, i64);

// 实现 `MinMax` 的 `Display`。
impl fmt::Display for MinMax {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 使用 `self.number` 来表示各个数据。
        write!(f, "({}, {})", self.0, self.1)
    }
}

// 为了比较，定义一个含有具名字段的结构体。
#[derive(Debug)]
struct Point2D {
    x: f64,
    y: f64,
}

// 类似地对 `Point2D` 实现 `Display`
impl fmt::Display for Point2D {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 自定义格式，使得仅显示 `x` 和 `y` 的值。
        write!(f, "x: {}, y: {}", self.x, self.y)
    }
}

fn main() {
    let minmax = MinMax(0, 14);

    println!("Compare structures:");
    println!("Display: {}", minmax);
    println!("Debug: {:?}", minmax);

    let big_range =   MinMax(-300, 300);
    let small_range = MinMax(-3, 3);

    println!("The big range is {big} and the small is {small}",
             small = small_range,
             big = big_range);

    let point = Point2D { x: 3.3, y: 7.2 };

    println!("Compare points:");
    println!("Display: {}", point);
    println!("Debug: {:?}", point);

    // 报错。`Debug` 和 `Display` 都被实现了，但 `{:b}` 需要 `fmt::Binary`
    // 得到实现。这语句不能运行。
    // println!("What does Point2D look like in binary: {:b}?", point);
}

```

`fmt::Display` 被实现了，而 `fmt::Binary` 没有，因此 `fmt::Binary` 不能使用。 `std::fmt` 有很多这样的 [`trait`](https://rustwiki.org/zh-CN/rust-by-example/trait.html)，它们都要求有各自的实现。这些内容将在后面的 [`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/) 章节中详细介绍。

#### [动手试一试](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

检验上面例子的输出，然后在示例程序中，仿照 `Point2D` 结构体增加一个复数结构体。 使用一样的方式打印，输出结果要求是这个样子：

```txt
Display: 3.3 + 7.2i
Debug: Complex { real: 3.3, imag: 7.2 }
```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`derive`](https://rustwiki.org/zh-CN/rust-by-example/trait/derive.html), [`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/), [macros](https://rustwiki.org/zh-CN/rust-by-example/macros.html), [`struct`](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html), [`trait`](https://rustwiki.org/zh-CN/rust-by-example/trait.html), 和 [use](https://rustwiki.org/zh-CN/rust-by-example/mod/use.html)

## [测试实例：List](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display/testcase_list.html#%E6%B5%8B%E8%AF%95%E5%AE%9E%E4%BE%8Blist) <a href="#ce-shi-shi-li-list" id="ce-shi-shi-li-list"></a>

对一个结构体实现 `fmt::Display`，其中的元素需要一个接一个地处理到，这可能会很麻烦。问题在于每个 `write!` 都要生成一个 `fmt::Result`。正确的实现需要处理**所有**的 Result。Rust 专门为解决这个问题提供了 `?` 操作符。

在 `write!` 上使用 `?` 会像是这样：

```rust
// 对 `write!` 进行尝试（try），观察是否出错。若发生错误，返回相应的错误。
// 否则（没有出错）继续执行后面的语句。
write!(f, "{}", value)?;
```

另外，你也可以使用 `try!` 宏，它和 `?` 是一样的。这种写法比较罗嗦，故不再推荐， 但在老一些的 Rust 代码中仍会看到。使用 `try!` 看起来像这样：

```rust
try!(write!(f, "{}", value));
```

有了 `?`，对一个 `Vec` 实现 `fmt::Display` 就很简单了：

```rust
use std::fmt; // 导入 `fmt` 模块。

// 定义一个包含单个 `Vec` 的结构体 `List`。
struct List(Vec<i32>);

impl fmt::Display for List {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 使用元组的下标获取值，并创建一个 `vec` 的引用。
        let vec = &self.0;

        write!(f, "[")?;

        // 使用 `v` 对 `vec` 进行迭代，并用 `count` 记录迭代次数。
        for (count, v) in vec.iter().enumerate() {
            // 对每个元素（第一个元素除外）加上逗号。
            // 使用 `?` 或 `try!` 来返回错误。
            if count != 0 { write!(f, ", ")?; }
            write!(f, "{}", v)?;
        }

        // 加上配对中括号，并返回一个 fmt::Result 值。
        write!(f, "]")
    }
}

fn main() {
    let v = List(vec![1, 2, 3]);
    println!("{}", v);
}

```

#### [动手试一试：](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display/testcase_list.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

更改程序使 vector 里面每个元素的下标也能够打印出来。新的结果如下：

```rust
[0: 1, 1: 2, 2: 3]
```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/hello/print/print_display/testcase_list.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`for`](https://rustwiki.org/zh-CN/rust-by-example/flow_control/for.html), [`ref`](https://rustwiki.org/zh-CN/rust-by-example/scope/borrow/ref.html), [`Result`](https://rustwiki.org/zh-CN/rust-by-example/std/result.html), [`struct`](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html), [`?`](https://rustwiki.org/zh-CN/rust-by-example/std/result/question_mark.html), 和 [`vec!`](https://rustwiki.org/zh-CN/rust-by-example/std/vec.html)

## [格式化](https://rustwiki.org/zh-CN/rust-by-example/hello/print/fmt.html#%E6%A0%BC%E5%BC%8F%E5%8C%96) <a href="#ge-shi-hua" id="ge-shi-hua"></a>

我们已经看到，格式化的方式是通过**格式字符串**来指定的：

* `format!("{}", foo)` -> `"3735928559"`
* `format!("0x{:X}", foo)` -> [`"0xDEADBEEF"`](https://en.wikipedia.org/wiki/Deadbeef#Magic_debug_values)
* `format!("0o{:o}", foo)` -> `"0o33653337357"`

根据使用的**参数类型**是 `X`、`o` 还是**未指定**，同样的变量（`foo`）能够格式化成不同的形式。

这个格式化的功能是通过 trait 实现的，每种参数类型都对应一种 trait。最常见的格式化 trait 就是 `Display`，它可以处理参数类型为未指定的情况，比如 `{}`。

```rust
use std::fmt::{self, Formatter, Display};

struct City {
    name: &'static str,
    // 纬度
    lat: f32,
    // 经度
    lon: f32,
}

impl Display for City {
    // `f` 是一个缓冲区（buffer），此方法必须将格式化后的字符串写入其中
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        let lat_c = if self.lat >= 0.0 { 'N' } else { 'S' };
        let lon_c = if self.lon >= 0.0 { 'E' } else { 'W' };

        // `write!` 和 `format!` 类似，但它会将格式化后的字符串写入
        // 一个缓冲区（即第一个参数f）中。
        write!(f, "{}: {:.3}°{} {:.3}°{}",
               self.name, self.lat.abs(), lat_c, self.lon.abs(), lon_c)
    }
}

#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

fn main() {
    for city in [
        City { name: "Dublin", lat: 53.347778, lon: -6.259722 },
        City { name: "Oslo", lat: 59.95, lon: 10.75 },
        City { name: "Vancouver", lat: 49.25, lon: -123.1 },
    ].iter() {
        println!("{}", *city);
    }
    for color in [
        Color { red: 128, green: 255, blue: 90 },
        Color { red: 0, green: 3, blue: 254 },
        Color { red: 0, green: 0, blue: 0 },
    ].iter() {
        // 在添加了针对 fmt::Display 的实现后，请改用 {} 检验效果。
        println!("{:?}", *color)
    }
}

```

在 [`fmt::fmt`](https://rustwiki.org/zh-CN/std/fmt/) 文档中可以查看[格式化 traits 一览表](https://rustwiki.org/zh-CN/std/fmt/#formatting-traits)和它们的参数类型。

#### [动手试一试](https://rustwiki.org/zh-CN/rust-by-example/hello/print/fmt.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

为上面的 `Color` 结构体实现 `fmt::Display`，应得到如下的输出结果：

```
RGB (128, 255, 90) 0x80FF5A
RGB (0, 3, 254) 0x0003FE
RGB (0, 0, 0) 0x000000
```

如果感到疑惑，可看下面两条提示：

* 你[可能需要多次列出每个颜色](https://rustwiki.org/zh-CN/std/fmt/#argument-types)，
* 你可以使用 `:02` [补零使位数为 2 位](https://rustwiki.org/zh-CN/std/fmt/#width)。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/hello/print/fmt.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`std::fmt`](https://rustwiki.org/zh-CN/std/fmt/)

\
