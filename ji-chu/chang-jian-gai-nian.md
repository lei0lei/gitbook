# 常见概念

{% hint style="info" %}
Rust 语言有一组保留的 **关键字**（_keywords_），就像大部分语言一样，它们只能由语言本身使用。记住，你不能使用这些关键字作为变量或函数的名称。大部分关键字有特殊的意义，你将在 Rust 程序中使用它们完成各种任务；一些关键字目前没有相应的功能，是为将来可能添加的功能保留的。可以在[附录 A](https://kaisery.github.io/trpl-zh-cn/appendix-01-keywords.html) 中找到关键字的列表。
{% endhint %}

## 变量和可变性 <a href="#bian-liang-he-ke-bian-xing" id="bian-liang-he-ke-bian-xing"></a>

<mark style="color:red;">变量默认是不可改变的（immutable）</mark>。这是 Rust 提供给你的众多优势之一，让你得以充分利用 Rust 提供的安全性和简单并发性来编写代码。不过，你仍然可以使用可变变量。让我们探讨一下 Rust 为何及如何鼓励你利用不可变性，以及何时你会选择不使用不可变性。

当变量不可变时，一旦值被绑定一个名称上，你就不能改变这个值。为了对此进行说明，使用 `cargo new variables` 命令在 _projects_ 目录生成一个叫做 _variables_ 的新项目。

接着，在新建的 _variables_ 目录，打开 _src/main.rs_ 并将代码替换为如下代码，这些代码还不能编译，我们会首次检查到不可变错误（immutability error）

```rust
// Some code
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}

```

保存并使用 `cargo run` 运行程序。应该会看到一条与不可变性有关的错误信息

在尝试改变预设为不可变的值时，产生编译时错误是很重要的，因为这种情况可能导致 bug。如果一部分代码假设一个值永远也不会改变，而另一部分代码改变了这个值，第一部分代码就有可能以不可预料的方式运行。不得不承认这种 bug 的起因难以跟踪，尤其是第二部分代码只是 **有时** 会改变值。

Rust 编译器保证，如果声明一个值不会变，它就真的不会变，所以你不必自己跟踪它。这意味着你的代码更易于推导。

不过可变性也是非常有用的，可以用来更方便地编写代码。尽管变量默认是不可变的，你仍然可以<mark style="color:red;">在变量名前添加</mark> <mark style="color:red;"></mark><mark style="color:red;">`mut`</mark> <mark style="color:red;"></mark><mark style="color:red;">来使其可变</mark>。`mut` 也向读者表明了其他代码将会改变这个变量值的意图。

```rust
// Some code
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

### 常量 <a href="#chang-liang" id="chang-liang"></a>

类似于不可变变量，_常量 (constants)_ 是绑定到一个名称的不允许改变的值，不过常量与变量还是有一些区别。

首先，<mark style="color:red;">不允许对常量使用</mark> <mark style="color:red;"></mark><mark style="color:red;">`mut`</mark>。常量不光默认不可变，它总是不可变。<mark style="color:red;">声明常量使用</mark> <mark style="color:red;"></mark><mark style="color:red;">`const`</mark> <mark style="color:red;"></mark><mark style="color:red;">关键字</mark>而不是 `let`，并且 _必须_ 注明<mark style="color:red;">值的类型</mark>。在下一部分，[“数据类型”](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B) 中会介绍类型和类型注解，现在无需关心这些细节，记住总是标注类型即可。

常量可以在任何作用域中声明，包括全局作用域，这在一个值需要被很多部分的代码用到时很有用。

最后一个区别是，<mark style="color:red;">常量只能被设置为常量表达式</mark>，而不可以是其他任何只能在运行时计算出的值。

下面是一个声明常量的例子：

```rust
// Some code
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量的名称是 `THREE_HOURS_IN_SECONDS`，它的值被设置为 60（一分钟内的秒数）乘以 60（一小时内的分钟数）再乘以 3（我们在这个程序中要计算的小时数）的结果。<mark style="color:red;">Rust 对常量的命名约定是在单词之间使用全大写加下划线</mark>。编译器能够在编译时计算一组有限的操作，这使我们可以选择以更容易理解和验证的方式写出此值，而不是将此常量设置为值 10,800。有关声明常量时可以使用哪些操作的详细信息，请参阅 [Rust Reference 的常量求值部分](https://doc.rust-lang.org/reference/const_eval.html)。

<mark style="color:red;">在声明它的作用域之中，常量在整个程序生命周期中都有效</mark>，此属性使得常量可以作为多处代码使用的全局范围的值，例如一个游戏中所有玩家可以获取的最高分或者光速。

将遍布于应用程序中的硬编码值声明为常量，能帮助后来的代码维护人员了解值的意图。如果将来需要修改硬编码值，也只需修改汇聚于一处的硬编码值。

### 隐藏 <a href="#yin-cang" id="yin-cang"></a>

正如在[第二章](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E6%AF%94%E8%BE%83%E7%8C%9C%E6%B5%8B%E7%9A%84%E6%95%B0%E5%AD%97%E5%92%8C%E7%A7%98%E5%AF%86%E6%95%B0%E5%AD%97)猜数字游戏中所讲，我们可以定义一个与之前变量同名的新变量。Rustacean 们称之为第一个变量被第二个 **隐藏（Shadowing）** 了，这意味着当您使用变量的名称时，编译器将看到第二个变量。实际上，第二个变量“遮蔽”了第一个变量，此时任何使用该变量名的行为中都会视为是在使用第二个变量，直到第二个变量自己也被隐藏或第二个变量的作用域结束。可以用相同变量名称来隐藏一个变量，以及重复使用 `let` 关键字来多次隐藏，如下所示：

```rust
// Some code
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```

这个程序首先将 `x` 绑定到值 `5` 上。接着通过 `let x =` 创建了一个新变量 `x`，获取初始值并加 `1`，这样 `x` 的值就变成 `6` 了。然后，在使用花括号创建的内部作用域内，第三个 `let` 语句也隐藏了 `x` 并创建了一个新的变量，将之前的值乘以 `2`，`x` 得到的值是 `12`。当该作用域结束时，内部 shadowing 的作用域也结束了，`x` 又返回到 `6`。

隐藏与将变量标记为 `mut` 是有区别的。当不小心尝试对变量重新赋值时，如果没有使用 `let` 关键字，就会导致编译时错误。通过使用 `let`，我们可以用这个值进行一些计算，不过计算完之后变量仍然是不可变的。

`mut` 与隐藏的另一个区别是，当再次使用 `let` 时，实际上创建了一个新变量，我们可以改变值的类型，并且复用这个名字。例如，假设程序请求用户输入空格字符来说明希望在文本之间显示多少个空格，接下来我们想将输入存储成数字（多少个空格）：

```rust
// Some code
    let spaces = "   ";
    let spaces = spaces.len();
```

## 数据类型 <a href="#shu-ju-lei-xing" id="shu-ju-lei-xing"></a>

在 Rust 中，每一个值都属于某一个 <mark style="color:red;">**数据类型**</mark>（_data type_），这告诉 Rust 它被指定为何种数据，以便明确数据处理方式。我们将看到两类数据类型子集：标量（scalar）和复合（compound）。

记住，Rust 是 <mark style="color:red;">**静态类型**</mark>（_statically typed_）语言，也就是说在编译时就必须知道所有变量的类型。根据值及其使用方式，编译器通常可以推断出我们想要用的类型。当多种类型均有可能时，比如第二章的 [“比较猜测的数字和秘密数字”](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number) 使用 `parse` 将 `String` 转换为数字时，<mark style="color:red;">必须增加类型注解</mark>，像这样：

```rust
// Some code
let guess: u32 = "42".parse().expect("Not a number!");
```

### 标量类型 <a href="#biao-liang-lei-xing" id="biao-liang-lei-xing"></a>

**标量**（_scalar_）类型代表一个单独的值。Rust 有<mark style="color:blue;">四种基本的标量类型</mark>：整型、浮点型、布尔类型和字符类型。

#### **整型**

**整数** 是一个没有小数部分的数字。我们在第二章使用过 `u32` 整数类型。该类型声明表明，它关联的值应该是一个占据 32 比特位的无符号整数（有符号整数类型以 `i` 开头而不是 `u`）。表格 3-1 展示了 Rust 内建的整数类型。我们可以使用其中的任一个来声明一个整数值的类型。

| 长度   | 有符号   | 无符号   |
| ---- | ----- | ----- |
| 8b   | i8    | u8    |
| 16b  | i16   | u16   |
| 32b  | i32   | u32   |
| 64b  | i64   | u64   |
| 128b | i128  | u128  |
| arch | isize | usize |

每一个变体都可以是有符号或无符号的，并有一个明确的大小。**有符号** 和 **无符号** 代表数字能否为负值，换句话说，这个数字是否有可能是负数（有符号数），或者永远为正而不需要符号（无符号数）。这有点像在纸上书写数字：当需要考虑符号的时候，数字以加号或减号作为前缀；然而，可以安全地假设为正数时，加号前缀通常省略。

`isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的，32 位架构上它们是 32 位的。

可以使用表格 3-2 中的任何一种形式编写数字字面值。请注意可以是多种数字类型的数字字面值允许使用类型后缀，例如 `57u8` 来指定类型，同时也<mark style="color:blue;">允许使用</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`_`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">做为分隔符以方便读数</mark>，例如`1_000`，它的值与你指定的 `1000` 相同。

| 数字字面值 | 示例           |   |
| ----- | ------------ | - |
| 十进制   | 98\_222      |   |
| 十六进制  | 0xff         |   |
| 八进制   | 0o77         |   |
| 二进制   | 0b1111\_0000 |   |
| 单字节字符 | b'A'         |   |

那么该使用哪种类型的数字呢？如果拿不定主意，Rust 的默认类型通常是个不错的起点，数字类型默认是 `i32`。`isize` 或 `usize` 主要作为某些集合的索引。

{% hint style="info" %}


比方说有一个 `u8` ，它可以存放从零到 `255` 的值。那么当你将其修改为 `256` 时会发生什么呢？这被称为 “整型溢出”（“integer overflow” ），这会导致以下两种行为之一的发生。当在 debug 模式编译时，Rust 检查这类问题并使程序 _panic_，这个术语被 Rust 用来表明程序因错误而退出。第九章 [“`panic!` 与不可恢复的错误”](https://kaisery.github.io/trpl-zh-cn/ch09-01-unrecoverable-errors-with-panic.html) 部分会详细介绍 panic。

使用 `--release` flag 在 release 模式中构建时，Rust **不会**检测会导致 panic 的整型溢出。相反发生整型溢出时，Rust 会进行一种被称为二进制补码 wrapping（_two’s complement wrapping_）的操作。简而言之，比此类型能容纳最大值还大的值会回绕到最小值，值 `256` 变成 `0`，值 `257` 变成 `1`，依此类推。程序不会 panic，不过变量可能也不会是你所期望的值。依赖整型溢出 wrapping 的行为被认为是一种错误。

<mark style="color:red;">为了显式地处理溢出的可能性，可以使用这几类标准库提供的原始数字类型方法</mark>：

* 所有模式下都可以使用 `wrapping_*` 方法进行 wrapping，如 `wrapping_add`
* 如果 `checked_*` 方法出现溢出，则返回 `None`值
* 用 `overflowing_*` 方法返回值和一个布尔值，表示是否出现溢出
* 用 `saturating_*` 方法在值的最小值或最大值处进行饱和处理
{% endhint %}

#### **浮点型**

Rust 也有两个原生的 **浮点数**（_floating-point numbers_）类型，它们是带小数点的数字。Rust 的浮点数类型是 `f32` 和 `f64`，分别占 32 位和 64 位。默认类型是 `f64`，因为在现代 CPU 中，它与 `f32` 速度几乎一样，不过精度更高。所有的浮点型都是有符号的。

这是一个展示浮点数的实例：

```rust
// Some code
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

浮点数采用 IEEE-754 标准表示。`f32` 是单精度浮点数，`f64` 是双精度浮点数。

#### **数值运算**

Rust 中的所有数字类型都支持基本数学运算：<mark style="color:red;">加法、减法、乘法、除法和取余</mark>。整数除法会向下舍入到最接近的整数。下面的代码展示了如何在 `let` 语句中使用它们：

```rust
// Some code
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // 结果为 -1

    // remainder
    let remainder = 43 % 5;
}
```

#### **布尔型**

正如其他大部分编程语言一样，Rust 中的布尔类型有两个可能的值：`true` 和 `false`。Rust 中的布尔类型使用 `bool` 表示。例如：

```rust
// Some code
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

使用布尔值的主要场景是条件表达式，例如 `if` 表达式。在 [“控制流”（“Control Flow”）](https://kaisery.github.io/trpl-zh-cn/ch03-05-control-flow.html#%E6%8E%A7%E5%88%B6%E6%B5%81) 部分将介绍 `if` 表达式在 Rust 中如何工作。

#### **字符类型**

Rust 的 `char` 类型是语言中最原生的字母类型。下面是一些声明 `char` 值的例子：

```rust
// Some code
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

注意，我们用<mark style="color:red;">单引号声明</mark> <mark style="color:red;"></mark><mark style="color:red;">`char`</mark> <mark style="color:red;"></mark><mark style="color:red;">字面量</mark>，而与之相反的是，使用<mark style="color:red;">双引号声明字符串字面量</mark>。Rust 的 `char` 类型的大小为四个字节 (four bytes)，并代表了一个 Unicode 标量值（Unicode Scalar Value），这意味着它可以比 ASCII 表示更多内容。在 Rust 中，带变音符号的字母（Accented letters），中文、日文、韩文等字符，emoji（绘文字）以及零长度的空白字符都是有效的 `char` 值。Unicode 标量值包含从 `U+0000` 到 `U+D7FF` 和 `U+E000` 到 `U+10FFFF` 在内的值。不过，“字符” 并不是一个 Unicode 中的概念，所以人直觉上的 “字符” 可能与 Rust 中的 `char` 并不符合。

### 复合类型 <a href="#fu-he-lei-xing" id="fu-he-lei-xing"></a>

**复合类型**（_Compound types_）可以将多个值组合成一个类型。Rust 有两个原生的复合类型：<mark style="color:red;">元组</mark>（tuple）和<mark style="color:red;">数组</mark>（array）。

#### **元组类型**

元组是一个将多个其他类型的值组合进一个复合类型的主要方式。<mark style="color:red;">元组长度固定：一旦声明，其长度不会增大或缩小。</mark>

我们使用包含在圆括号中的逗号分隔的值列表来创建一个元组。元组中的每一个位置都有一个类型，而且这些不同值的类型也不必是相同的。这个例子中使用了可选的类型注解：

```rust
// Some code
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

`tup` 变量绑定到整个元组上，因为元组是一个单独的复合元素。为了从元组中获取单个值，可以使用模式匹配（pattern matching）来解构（destructure）元组值，像这样：

```rust
// Some code
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {y}");
}
```

程序首先创建了一个元组并绑定到 `tup` 变量上。接着使用了 `let` 和一个模式将 `tup` 分成了三个不同的变量，`x`、`y` 和 `z`。这叫做 <mark style="color:red;">**解构**</mark>（_destructuring_），因为它将一个元组拆成了三个部分。最后，程序打印出了 `y` 的值，也就是 `6.4`。

我们也可以使用点号（`.`）后跟<mark style="color:red;">值的索引</mark>来直接访问它们。例如：

```rust
// Some code
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

这个程序创建了一个元组，`x`，然后使用其各自的索引访问元组中的每个元素。跟大多数编程语言一样，元组的第一个索引值是 0。

<mark style="color:blue;">不带任何值的元组有个特殊的名称，叫做</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**单元（unit）**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">元组</mark>。这种值以及对应的类型都写作 `()`，表示空值或空的返回类型。如果<mark style="color:blue;">表达式不返回任何其他值，则会隐式返回单元值</mark>。

#### **数组类型**

另一个包含多个值的方式是 **数组**（_array_）。与元组不同，数组中的每个元素的类型必须相同。Rust 中的数组与一些其他语言中的数组不同，Rust 中的<mark style="color:red;">数组长度是固定的</mark>。

我们将数组的值写成在方括号内，用逗号分隔：

```rust
// Some code
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

当你想要<mark style="color:red;">在栈（stack）而不是在堆（heap）上为数据分配空间</mark>（，或者是想要确保总是有固定数量的元素时，数组非常有用。但是数组并不如 vector 类型灵活。vector 类型是标准库提供的一个 **允许** 增长和缩小长度的类似数组的集合类型。当不确定是应该使用数组还是 vector 的时候，那么很可能应该使用 vector。

然而，当你确定元素个数不会改变时，数组会更有用。例如，当你在一个程序中使用月份名字时，你更应趋向于使用数组而不是 vector，因为你确定只会有 12 个元素。

```rust
// Some code
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

可以像这样编写数组的类型：在方括号中包含每个元素的类型，后跟分号，再后跟数组元素的数量。

```rust
// Some code
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

这里，`i32` 是每个元素的类型。分号之后，数字 `5` 表明该数组包含五个元素。

你还可以通过在方括号中指定初始值加分号再加元素个数的方式来创建一个每个元素都为相同值的数组：

```rust
// Some code
let a = [3; 5];
```

变量名为 `a` 的数组将包含 `5` 个元素，这些元素的值最初都将被设置为 `3`。这种写法与 `let a = [3, 3, 3, 3, 3];` 效果相同，但更简洁。

#### **访问数组元素**

数组是可以在栈 (stack) 上分配的已知固定大小的单个内存块。可以使用索引来访问数组的元素，像这样：

```rust
// Some code
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

在这个例子中，叫做 `first` 的变量的值是 `1`，因为它是数组索引 `[0]` 的值。变量 `second` 将会是数组索引 `[1]` 的值 `2`。

#### **无效的数组元素访问**

```rust
// Some code
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!("The value of the element at index {index} is: {element}");
}
```

程序在索引操作中使用一个无效的值时导致 **运行时** 错误。程序带着错误信息退出，并且没有执行最后的 `println!` 语句。当尝试用索引访问一个元素时，Rust 会检查指定的索引是否小于数组的长度。如果索引超出了数组长度，Rust 会 _panic_，这是 Rust 术语，它用于程序因为错误而退出的情况。这种检查必须在运行时进行，特别是在这种情况下，因为编译器不可能知道用户在以后运行代码时将输入什么值。

这是第一个在实战中遇到的 Rust 安全原则的例子。在很多底层语言中，并没有进行这类检查，这样当提供了一个不正确的索引时，就会访问无效的内存。通过立即退出而不是允许内存访问并继续执行，Rust 让你避开此类错误。

## 函数 <a href="#han-shu" id="han-shu"></a>

函数在 Rust 代码中非常普遍。你已经见过语言中最重要的函数之一：`main` 函数，它是很多程序的入口点。你也见过 `fn` 关键字，它用来声明新函数。

Rust 代码中的函数和变量名使用 _snake case_ 规范风格。在 snake case 中，所有字母都是小写并使用下划线分隔单词。这是一个包含函数定义示例的程序：

```rust
// Some code
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

我们在 Rust 中通过输入 `fn` 后面跟着函数名和一对圆括号来定义函数。大括号告诉编译器哪里是函数体的开始和结尾。

可以使用函数名后跟圆括号来调用我们定义过的任意函数。因为程序中已定义 `another_function` 函数，所以可以在 `main` 函数中调用它。注意，源码中 `another_function` 定义在 `main` 函数 **之后**；也可以定义在之前。Rust 不关心函数定义所在的位置，只要函数被调用时出现在调用之处可见的作用域内就行。

### 参数 <a href="#can-shu" id="can-shu"></a>

我们可以定义为拥有 **参数**（_parameters_）的函数，参数是特殊变量，是函数签名的一部分。当函数拥有参数（形参）时，可以为这些参数提供具体的值（实参）。技术上讲，这些具体值被称为参数（_arguments_），但是在日常交流中，人们倾向于不区分使用 _parameter_ 和 _argument_ 来表示函数定义中的变量或调用函数时传入的具体值。

在这版 `another_function` 中，我们增加了一个参数：

```rust
// Some code
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}
```

`another_function` 的声明中有一个命名为 `x` 的参数。`x` 的类型被指定为 `i32`。当我们将 `5` 传给 `another_function` 时，`println!` 宏会把 `5` 放在格式字符串中包含 `x` 的那对花括号的位置。

在函数签名中，<mark style="color:red;">**必须**</mark> <mark style="color:red;"></mark><mark style="color:red;">声明每个参数的类型</mark>。这是 Rust 设计中一个经过慎重考虑的决定：要求在函数定义中提供类型注解，意味着编译器再也不需要你在代码的其他地方注明类型来指出你的意图。而且，在知道函数需要什么类型后，编译器就能够给出更有用的错误消息。

当定义多个参数时，使用逗号分隔，像这样：

```rust
// Some code
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```

这个例子创建了一个名为 `print_labeled_measurement` 的函数，它有两个参数。第一个参数名为 `value`，类型是 `i32`。第二个参数是 `unit_label` ，类型是 `char`。然后，该函数打印包含 `value` 和 `unit_label` 的文本。

### 语句和表达式 <a href="#yu-ju-he-biao-da-shi" id="yu-ju-he-biao-da-shi"></a>

函数体由一系列的语句和一个可选的结尾表达式构成。目前为止，我们提到的函数还不包含结尾表达式，不过你已经见过作为语句一部分的表达式。因为 Rust 是一门基于表达式（expression-based）的语言，这是一个需要理解的（不同于其他语言）重要区别。其他语言并没有这样的区别，所以让我们看看语句与表达式有什么区别以及这些区别是如何影响函数体的。

**语句**（_Statements_）是<mark style="color:red;">执行一些操作但不返回值的指令</mark>。 **表达式**（_Expressions_）<mark style="color:red;">计算并产生一个值</mark>。让我们看一些例子。

函数定义也是语句，上面整个例子本身就是一个语句。

语句不返回值。因此，不能把 `let` 语句赋值给另一个变量，比如下面的例子尝试做的，会产生一个错误：

```
// Some code
fn main() {
    let x = (let y = 6);
}
```

`let y = 6` 语句并不返回值，所以没有可以绑定到 `x` 上的值。这与其他语言不同，例如 C 和 Ruby，它们的赋值语句会返回所赋的值。在这些语言中，可以这么写 `x = y = 6`，这样 `x` 和 `y` 的值都是 `6`；Rust 中不能这样写。

表达式会计算出一个值，并且你将编写的大部分 Rust 代码是由表达式组成的。考虑一个数学运算，比如 `5 + 6`，这是一个表达式并计算出值 `11`。表达式可以是语句的一部分：在示例 3-1 中，语句 `let y = 6;` 中的 `6` 是一个表达式，它计算出的值是 `6`。函数调用是一个表达式。宏调用是一个表达式。用大括号创建的一个新的块作用域也是一个表达式，例如：

```rust
// Some code
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

这个表达式：

```
// Some code
{
    let x = 3;
    x + 1
}
```

是一个代码块，它的值是 `4`。这个值作为 `let` 语句的一部分被绑定到 `y` 上。注意 `x+1` 这一行在结尾没有分号，与你见过的大部分代码行不同。<mark style="color:red;">表达式的结尾没有分号。如果在表达式的结尾加上分号，它就变成了语句，而语句不会返回值</mark>。在接下来探索具有返回值的函数和表达式时要谨记这一点。

### 具有返回值的函数 <a href="#ju-you-fan-hui-zhi-de-han-shu" id="ju-you-fan-hui-zhi-de-han-shu"></a>

函数可以向调用它的代码返回值。<mark style="color:red;">我们并不对返回值命名，但要在箭头（</mark><mark style="color:red;">`->`</mark><mark style="color:red;">）后声明它的类型</mark>。在 Rust 中，函数的返回值等同于函数体最后一个表达式的值。使用 `return` 关键字和指定值，可从函数中提前返回；但大部分函数隐式的返回最后的表达式。这是一个有返回值的函数的例子：

```
// Some code
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

在 `five` 函数中没有函数调用、宏、甚至没有 `let` 语句 —— 只有数字 `5`。这在 Rust 中是一个完全有效的函数。注意，也指定了函数返回值的类型，就是 `-> i32`。

`five` 函数的返回值是 `5`，所以返回值类型是 `i32`。让我们仔细检查一下这段代码。有两个重要的部分：首先，`let x = five();` 这一行表明我们使用函数的返回值初始化一个变量。因为 `five` 函数返回 `5.`

其次，`five` 函数没有参数并定义了返回值类型，不过函数体只有单单一个 `5` 也没有分号，因为这是一个表达式，我们想要返回它的值。

让我们看看另一个例子：

```rust
// Some code
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

运行代码会打印出 `The value of x is: 6`。但如果在包含 `x + 1` 的行尾加上一个分号，把它从表达式变成语句，我们将看到一个错误。

## 注释 <a href="#zhu-shi" id="zhu-shi"></a>

所有程序员都力求使其代码易于理解，不过有时还需要提供额外的解释。在这种情况下，程序员在源码中留下 **注释**（_comments_），编译器会忽略它们，不过阅读代码的人可能觉得有用。

在 Rust 中，惯用的注释样式是以两个斜杠开始注释，并持续到本行的结尾。<mark style="color:red;">对于超过一行的注释，需要在每一行前都加上</mark> <mark style="color:red;"></mark><mark style="color:red;">`//.`</mark>

## 控制流 <a href="#kong-zhi-liu" id="kong-zhi-liu"></a>

### `if` 表达式 <a href="#if-biao-da-shi" id="if-biao-da-shi"></a>

`if` 表达式允许根据条件执行不同的代码分支。你提供一个条件并表示 “如果条件满足，运行这段代码；如果条件不满足，不运行这段代码。”

```rust
// Some code
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

所有的 <mark style="color:red;">`if`</mark> <mark style="color:red;"></mark><mark style="color:red;">表达式</mark>都以 `if` 关键字开头，其后跟一个条件。在这个例子中，条件检查变量 `number` 的值是否小于 5。在条件为 `true` 时希望执行的代码块位于紧跟条件之后的大括号中。`if` 表达式中与条件关联的代码块有时被叫做 _arms_，就像第二章 [“比较猜测的数字和秘密数字”](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E6%AF%94%E8%BE%83%E7%8C%9C%E6%B5%8B%E7%9A%84%E6%95%B0%E5%AD%97%E5%92%8C%E7%A7%98%E5%AF%86%E6%95%B0%E5%AD%97) 部分中讨论到的 `match` 表达式中的分支一样。

也可以包含一个可选的 `else` 表达式来提供一个在条件为 `false` 时应当执行的代码块，这里我们就这么做了。如果不提供 `else` 表达式并且条件为 `false` 时，程序会直接忽略 `if` 代码块并继续执行下面的代码。

另外值得注意的是代码中的条件 **必须** 是 `bool` 值。如果条件不是 `bool` 值，我们将得到一个错误。

#### **使用 `else if` 处理多重条件**

可以将 `else if` 表达式与 `if` 和 `else` 组合来实现多重条件。例如：

```rust
// Some code
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

当执行这个程序时，它按顺序检查每个 `if` 表达式并执行第一个条件为 `true` 的代码块。注意即使 6 可以被 2 整除，也不会输出 `number is divisible by 2`，更不会输出 `else` 块中的 `number is not divisible by 4, 3, or 2`。原因是 Rust 只会执行第一个条件为 `true` 的代码块，并且一旦它找到一个以后，甚至都不会检查剩下的条件了。

使用过多的 `else if` 表达式会使代码显得杂乱无章，所以如果有多于一个 `else if` 表达式，最好重构代码

#### **在 `let` 语句中使用 `if`**

因为 `if` 是一个表达式，我们可以在 `let` 语句的右侧使用它，

```rust
// Some code
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

`number` 变量将会绑定到表示 `if` 表达式结果的值上.

记住，<mark style="color:red;">代码块的值是其最后一个表达式的值，而数字本身就是一个表达式</mark>。在这个例子中，整个 `if` 表达式的值取决于哪个代码块被执行。这意味着 `if` 的每个分支的可能的返回值都必须是相同类型；在示例 3-2 中，`if` 分支和 `else` 分支的结果都是 `i32` 整型。如果它们的类型不匹配，如下面这个例子，则会出现一个错误：

```
// Some code
fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {number}");
}
```

`if` 代码块中的表达式返回一个整数，而 `else` 代码块中的表达式返回一个字符串。这不可行，因为变量必须只有一个类型。Rust 需要在编译时就确切的知道 `number` 变量的类型，这样它就可以在编译时验证在每处使用的 `number` 变量的类型是有效的。如果`number`的类型仅在运行时确定，则 Rust 无法做到这一点；且编译器必须跟踪每一个变量的多种假设类型，那么它就会变得更加复杂，对代码的保证也会减少。

### 使用循环重复执行 <a href="#shi-yong-xun-huan-zhong-fu-zhi-hang" id="shi-yong-xun-huan-zhong-fu-zhi-hang"></a>

多次执行同一段代码是很常用的，Rust 为此提供了多种 **循环**（_loops_）。一个循环执行循环体中的代码直到结尾并紧接着回到开头继续执行。为了实验一下循环，让我们新建一个叫做 _loops_ 的项目。

Rust 有三种循环：`loop`、`while` 和 `for`。我们每一个都试试。

#### **使用 `loop` 重复执行代码**

`loop` 关键字告诉 Rust 一遍又一遍地执行一段代码直到你明确要求停止。

```rust
// Some code
fn main() {
    loop {
        println!("again!");
    }
}
```

当运行这个程序时，我们会看到连续的反复打印 `again!`，直到我们手动停止程序。

Rust 提供了一种从代码中跳出循环的方法。可以使用 `break` 关键字来告诉程序何时停止循环。

#### **从循环返回值**

`loop` 的一个用例是重试可能会失败的操作，比如检查线程是否完成了任务。然而你可能会需要将操作的结果传递给其它的代码。如果将返回值加入你用来停止循环的 `break` 表达式，它会被停止的循环返回：

```rust
// Some code
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

在循环之前，我们声明了一个名为 `counter` 的变量并初始化为 `0`。接着声明了一个名为 `result` 来存放循环的返回值。在循环的每一次迭代中，我们将 `counter` 变量加 `1`，接着检查计数是否等于 `10`。当相等时，使用 `break` 关键字返回值 `counter * 2`。循环之后，我们通过分号结束赋值给 `result` 的语句。最后打印出 `result` 的值，也就是 `20`。

#### **循环标签：在多个循环之间消除歧义**

如果存在嵌套循环，`break` 和 `continue` 应用于此时最内层的循环。你可以选择在一个循环上指定一个 **循环标签**（_loop label_），然后将标签与 `break` 或 `continue` 一起使用，使这些关键字应用于已标记的循环而不是最内层的循环。下面是一个包含两个嵌套循环的示例

```rust
// Some code
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

外层循环有一个标签 `counting_up`，它将从 0 数到 2。没有标签的内部循环从 10 向下数到 9。第一个没有指定标签的 `break` 将只退出内层循环。`break 'counting_up;` 语句将退出外层循环。

#### **`while` 条件循环**

在程序中计算循环的条件也很常见。当条件为 `true`，执行循环。当条件不再为 `true`，调用 `break` 停止循环。这个循环类型可以通过组合 `loop`、`if`、`else` 和 `break` 来实现；如果你喜欢的话，现在就可以在程序中试试。

然而，这个模式太常用了，Rust 为此内置了一个语言结构，它被称为 `while` 循环。示例 3-3 使用了 `while`：程序循环三次，每次数字都减一。接着，在循环结束后，打印出另一个信息并退出。

```rust
// Some code
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

这种结构消除了很多使用 `loop`、`if`、`else` 和 `break` 时所必须的嵌套，这样更加清晰。当条件为 `true` 就执行，否则退出循环。

#### **使用 `for` 遍历集合**

可以使用 `while` 结构来遍历集合中的元素，比如数组

```rust
// Some code
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}
```

这里，代码对数组中的元素进行计数。它从索引 `0` 开始，并接着循环直到遇到数组的最后一个索引（这时，`index < 5` 不再为真）。

数组中的所有五个元素都如期被打印出来。尽管 `index` 在某一时刻会到达值 `5`，不过循环在其尝试从数组获取第六个值（会越界）之前就停止了。

但这个过程很容易出错；如果索引长度或测试条件不正确会导致程序 panic。例如，如果将 `a` 数组的定义改为包含 4 个元素而忘记了更新条件 `while index < 4`，则代码会 panic。这也使程序更慢，因为编译器增加了运行时代码来对每次循环进行条件检查，以确定在循环的每次迭代中索引是否在数组的边界内。

作为更简洁的替代方案，可以使用 `for` 循环来对一个集合的每个元素执行一些代码。

```rust
// Some code
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```

当运行这段代码时，将看到与示例 3-4 一样的输出。更为重要的是，我们增强了代码安全性，并消除了可能由于超出数组的结尾或遍历长度不够而缺少一些元素而导致的 bug。

例如，在示例 3-4 的代码中，如果你将 `a` 数组的定义改为有四个元素，但忘记将条件更新为 `while index < 4`，代码将会 panic。使用 `for` 循环的话，就不需要惦记着在改变数组元素个数时修改其他的代码了。

`for` 循环的安全性和简洁性使得它成为 Rust 中使用最多的循环结构。即使是在想要循环执行代码特定次数时，例如示例 3-3 中使用 `while` 循环的倒计时例子，大部分 Rustacean 也会使用 `for` 循环。这么做的方式是使用 `Range`，它是标准库提供的类型，用来生成从一个数字开始到另一个数字之前结束的所有数字的序列。

下面是一个使用 `for` 循环来倒计时的例子，它还使用了一个我们还未讲到的方法，`rev`，用来反转 range：

```rust
// Some code
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```





