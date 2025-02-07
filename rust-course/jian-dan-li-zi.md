# 简单例子

让我们一起动手完成一个项目，来快速上手 Rust！本章将介绍 Rust 中一些常用概念，并通过真实的程序来展示如何运用它们。你将会学到 `let`、`match`、方法（method）、关联函数（associated function）、外部 crate 等知识！后续章节会深入探讨这些概念的细节。在这一章，我们将练习基础内容。

我们会实现一个经典的新手编程问题：猜猜看游戏。它是这么工作的：程序将会随机生成一个 1 到 100 之间的随机整数。接着它会请玩家猜一个数并输入，然后提示猜测是大了还是小了。如果猜对了，它会打印祝贺信息并退出。



### 准备一个新项目 <a href="#zhun-bei-yi-ge-xin-xiang-mu" id="zhun-bei-yi-ge-xin-xiang-mu"></a>

```sh
// Some code
cargo new guessing_game
cd guessing_game
```

```rust
// src/main.rs
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

为了获取用户输入并打印结果作为输出，我们需要将 `io`输入/输出库引入当前作用域。`io` 库来自于标准库，也被称为 `std`：

```
// Some code
use std::io;
```

默认情况下，Rust 设定了若干个会自动导入到每个程序作用域中的标准库内容，这组内容被称为 _预导入（<mark style="color:red;">**preclude**</mark>）_ 内容。你可以在[标准库文档](https://doc.rust-lang.org/std/prelude/index.html)中查看预导入的所有内容。

如果你需要的类型不在预导入内容中，就必须使用 `use` 语句显式地将其引入作用域。`std::io` 库提供很多有用的功能，包括接收用户输入的功能。

#### 使用变量储存值 <a href="#shi-yong-bian-liang-chu-cun-zhi" id="shi-yong-bian-liang-chu-cun-zhi"></a>

接下来，创建一个 **变量**（_variable_）来储存用户输入，像这样：

```rust
// Some code
let mut guess = String::new();
```

在 Rust 中，<mark style="color:red;">变量默认是不可变的</mark>，这意味着一旦我们给变量赋值，这个值就不再可以修改了。

下面的例子展示了如何在变量名前使用 `mut` 来使一个变量可变：

```rust
// Some code
let apples = 5; // 不可变
let mut bananas = 5; // 可变
```

{% hint style="info" %}
注意：`//` 语法开始一个注释，持续到行尾。Rust 忽略注释中的所有内容
{% endhint %}

回到猜猜看程序中。现在我们知道了 `let mut guess` 会引入一个叫做 `guess` 的可变变量。等号（`=`）告诉 Rust 我们现在想将某个值绑定在变量上。等号的右边是 `guess` 所绑定的值，它是 `String::new` 的结果，这个函数会返回一个 `String` 的新实例。[`String`](https://doc.rust-lang.org/std/string/struct.String.html) 是一个标准库提供的字符串类型，它是 UTF-8 编码的可增长文本块。

`::new` 那一行的 `::` 语法表明 `new` 是 `String` 类型的一个 <mark style="color:red;">**关联函数**</mark>（_associated function_）。关联函数是针对类型实现的，在这个例子中是 `String`，而不是 `String` 的某个特定实例。一些语言中把它称为 **静态方法**（_static method_）。

`new` 函数创建了一个新的空字符串，你会发现很多类型上有 `new` 函数，因为它是创建类型实例的惯用函数名。

总的来说，`let mut guess = String::new();` 这一行创建了一个可变变量，当前它绑定到一个新的 `String` 空实例上。

#### 接收用户输入 <a href="#jie-shou-yong-hu-shu-ru" id="jie-shou-yong-hu-shu-ru"></a>

回忆一下，我们在程序的第一行使用 `use std::io;` 从标准库中引入了输入/输出功能。现在调用 `io` 库中的函数 `stdin`：

```rust
// Some code
    io::stdin()
        .read_line(&mut guess)
```

如果程序的开头没有使用 `use std::io;` 引入 `io` 库，我们仍可以通过把函数调用写成 `std::io::stdin` 来使用函数。`stdin` 函数返回一个 [`std::io::Stdin`](https://doc.rust-lang.org/std/io/struct.Stdin.html) 的实例，这代表终端标准输入句柄的类型。

代码的下一部分，`.read_line(&mut guess)`，调用 [`read_line`](https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line) 方法从标准输入句柄获取用户输入。我们还将 `&mut guess` 作为参数传递给 `read_line()` 函数，让其将用户输入储存到这个字符串中。<mark style="color:blue;">`read_line`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">的工作是，无论用户在标准输入中键入什么内容，都将其追加（不会覆盖其原有内容）到一个字符串中，因此它需要字符串作为参数。这个字符串参数应该是可变的，以便</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`read_line`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">将用户输入附加上去。</mark>

`&` 表示这个参数是一个 <mark style="color:red;">**引用**</mark>（_reference_），它允许多处代码访问同一处数据，而无需在内存中多次拷贝。引用是一个复杂的特性，Rust 的一个主要优势就是安全而简单的操纵引用。完成当前程序并不需要了解如此多细节。现在，我们只需知道它像变量一样，默认是不可变的。因此，需要写成 `&mut guess` 来使其可变，而不是 `&guess`。（第四章会更全面的解释引用。）

#### 使用 `Result` 类型来处理潜在的错误 <a href="#shi-yong-result-lei-xing-lai-chu-li-qian-zai-de-cuo-wu" id="shi-yong-result-lei-xing-lai-chu-li-qian-zai-de-cuo-wu"></a>

我们还没有完全分析完这行代码。虽然我们已经讲到了第三行代码，但要注意：它仍是逻辑行（虽然换行了但仍是语句）的一部分。后一部分是这个方法（method）：

```rust
// Some code
.expect("Failed to read line");
```

我们也可以将代码这样写：

```rust
// Some code
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

不过，过长的代码行难以阅读，所以最好拆开来写。通常来说，当使用 <mark style="color:red;">`.method_name()`</mark> <mark style="color:red;"></mark><mark style="color:red;">语法调用方法时引入换行符和空格将长的代码行拆开是明智的</mark>。现在来看看这行代码干了什么。

之前提到了 `read_line` 会将用户输入附加到传递给它的字符串中，不过它也会返回一个类型为 `Result` 的值。 [`Result`](https://doc.rust-lang.org/std/result/enum.Result.html) 是一种[_枚举类型_](https://kaisery.github.io/trpl-zh-cn/ch06-00-enums.html)，通常也写作 _enum_。枚举类型变量的值可以是多种可能状态中的一个。我们把每种可能的状态称为一种 _枚举成员（variant）_。

这里的 `Result` 类型将用来编码错误处理的信息。

`Result` 的成员是 `Ok` 和 `Err`，`Ok` 成员表示操作成功，内部包含成功时产生的值。`Err` 成员则意味着操作失败，并且包含失败的前因后果。

这些 `Result` 类型的作用是编码错误处理信息。`Result` 类型的值，像其他类型一样，拥有定义于其上的方法。<mark style="color:blue;">`Result`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">的实例拥有</mark> [<mark style="color:blue;">`expect`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">方法</mark>](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect)。如果 `io::Result` 实例的值是 `Err`，`expect` 会导致程序崩溃，并显示当做参数传递给 `expect` 的信息。如果 `read_line` 方法返回 `Err`，则可能是来源于底层操作系统错误的结果。如果 `Result` 实例的值是 `Ok`，`expect` 会获取 `Ok` 中的值并原样返回。在本例中，这个值是用户输入到标准输入中的字节数。

#### 使用 `println!` 占位符打印值 <a href="#shi-yong-println-zhan-wei-fu-da-yin-zhi" id="shi-yong-println-zhan-wei-fu-da-yin-zhi"></a>

除了位于结尾的右花括号，目前为止就只有这一行代码值得讨论一下了，就是这一行：

```rust
// Some code
println!("You guessed: {guess}");
```

这行代码现在打印了存储用户输入的字符串。里面的 `{}` 是预留在特定位置的占位符：把 `{}` 想象成小蟹钳，可以夹住合适的值。当打印变量的值时，变量名可以写进大括号中。当打印表达式的执行结果时，格式化字符串（format string）中大括号中留空，格式化字符串后跟逗号分隔的需要打印的表达式列表，其顺序与每一个空大括号占位符的顺序一致。在一个 `println!` 调用中打印变量和表达式的值看起来像这样：

```rust
// Some code
#![allow(unused)]
fn main() {
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
}

```

这行代码会打印出 `x = 5 and y + 2 = 12`。

### 生成一个秘密数字 <a href="#sheng-cheng-yi-ge-mi-mi-shu-zi" id="sheng-cheng-yi-ge-mi-mi-shu-zi"></a>

接下来，需要生成一个秘密数字，好让用户来猜。秘密数字应该每次都不同，这样重复玩才不会乏味；范围应该在 1 到 100 之间，这样才不会太困难。Rust 标准库中尚未包含随机数功能。然而，Rust 团队还是提供了一个包含上述功能的 [`rand` crate](https://crates.io/crates/rand)。

#### 使用 crate 来增加更多功能 <a href="#shi-yong-crate-lai-zeng-jia-geng-duo-gong-neng" id="shi-yong-crate-lai-zeng-jia-geng-duo-gong-neng"></a>

记住，_crate_ 是一个 Rust 代码包。我们正在构建的项目是一个 _二进制 crate_，它生成一个可执行文件。 `rand` crate 是一个 _库 crate_，<mark style="color:blue;">库 crate 可以包含任意能被其他程序使用的代码，但是不能自执行。</mark>

Cargo 对外部 crate 的运用是其真正的亮点所在。在我们使用 `rand` 编写代码之前，需要修改 _Cargo.toml_ 文件，引入一个 `rand` 依赖。现在打开这个文件并将下面这一行添加到 `[dependencies]` 片段标题之下。在当前版本下，请确保按照我们这里的方式指定 `rand`，否则本教程中的示例代码可能无法工作。

文件名：Cargo.toml

```toml
// Some code
[dependencies]
rand = "0.8.5"
```

在 _Cargo.toml_ 文件中，标题以及之后的内容属同一个片段，直到遇到下一个标题才开始新的片段。`[dependencies]` 片段告诉 Cargo 本项目依赖了哪些外部 crate 及其版本。本例中，我们使用语义化版本 `0.8.5` 来指定 `rand` crate。Cargo 理解 [语义化版本（Semantic Versioning）](http://semver.org/)（有时也称为 _SemVer_），这是一种定义版本号的标准。`0.8.5` 事实上是 `^0.8.5` 的简写，它表示任何至少是 `0.8.5` 但小于 `0.9.0` 的版本。

现在我们有了一个外部依赖，Cargo 从 _registry_ 上获取所有包的最新版本信息，这是一份来自 [Crates.io](https://crates.io/) 的数据拷贝。Crates.io 是 Rust 生态环境中的开发者们向他人贡献 Rust 开源项目的地方。

在更新完 registry 后，Cargo 检查 `[dependencies]` 片段并下载列表中包含但还未下载的 crates。本例中，虽然只声明了 `rand` 一个依赖，然而 Cargo 还是额外获取了 `rand` 所需要的其他 crates，因为 `rand` 依赖它们来正常工作。下载完成后，Rust 编译依赖，然后使用这些依赖编译项目。

如果不做任何修改，立刻再次运行 `cargo build`，则不会看到任何除了 `Finished` 行之外的输出。Cargo 知道它已经下载并编译了依赖，同时 _Cargo.toml_ 文件也没有变动。Cargo 还知道代码也没有任何修改，所以它不会重新编译代码。因为无事可做，它简单的退出了。

_**Cargo.lock**_**&#x20;文件确保构建是可重现的**

Cargo 有一个机制来确保任何人在任何时候重新构建代码，都会产生相同的结果：Cargo 只会使用你指定的依赖版本，除非你又手动指定了别的。例如，如果下周 `rand` crate 的 `0.8.6` 版本出来了，它修复了一个重要的 bug，同时也含有一个会破坏代码运行的缺陷。为了处理这个问题，Rust 在你第一次运行 `cargo build` 时建立了 _Cargo.lock_ 文件，我们现在可以&#x5728;_&#x67;uessing\_game_ 目录找到它。

当第一次构建项目时，Cargo 计算出所有符合要求的依赖版本并写入 _Cargo.lock_ 文件。当将来构建项目时，Cargo 会发现 _Cargo.lock_ 已存在并使用其中指定的版本，而不是再次计算所有的版本。这使得你拥有了一个自动化的可重现的构建。换句话说，项目会持续使用 `0.8.5` 直到你显式升级，多亏有了 _Cargo.lock_ 文件。由于 _Cargo.lock_ 文件对于“可重复构建”非常重要，因此它通常会和项目中的其余代码一样纳入到版本控制系统中。

**更新 crate 到一个新版本**

当你 **确实** 需要升级 crate 时，Cargo 提供了这样一个命令，`update`，它会忽略 _Cargo.lock_ 文件，并计算出所有符合 _Cargo.toml_ 声明的最新版本。Cargo 接下来会把这些版本写入 _Cargo.lock_ 文件。不过，Cargo 默认只会寻找大于 `0.8.5` 而小于 `0.9.0` 的版本。

#### 生成一个随机数 <a href="#sheng-cheng-yi-ge-sui-ji-shu" id="sheng-cheng-yi-ge-sui-ji-shu"></a>

让我们开始使用 `rand` 来生成一个猜猜看随机数。下一步是更新 _src/main.rs_

```rust
// Some code
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}

```

首先，我们新增了一行 `use rand::Rng;`。`Rng` 是一个 <mark style="color:red;">trait</mark>，它定义了随机数生成器应实现的方法，想使用这些方法的话，此 trait 必须在作用域中。

接下来，我们在中间还新增加了两行。第一行调用了 `rand::thread_rng` 函数提供实际使用的随机数生成器：它位于当前执行线程的本地环境中，并从操作系统获取 seed。接着调用随机数生成器的 `gen_range` 方法。这个方法由 `use rand::Rng` 语句引入到作用域的 `Rng` trait 定义。`gen_range` 方法获取一个范围表达式（range expression）作为参数，并生成一个在此范围之间的随机数。这里使用的这类范围表达式使用了 `start..=end` 这样的形式，也就是说包含了上下端点，所以需要指定 `1..=100` 来请求一个 1 和 100 之间的数。

{% hint style="info" %}
注意：你不可能凭空就知道应该 use 哪个 trait 以及该从 crate 中调用哪个方法，因此每个 crate 有使用说明文档。Cargo 有一个很棒的功能是：运行 `cargo doc --open` 命令来构建所有本地依赖提供的文档，并在浏览器中打开。例如，假设你对 `rand` crate 中的其他功能感兴趣，你可以运行 `cargo doc --open` 并点击左侧导航栏中的 `rand`。
{% endhint %}

### 比较猜测的数字和秘密数字 <a href="#bi-jiao-cai-ce-de-shu-zi-he-mi-mi-shu-zi" id="bi-jiao-cai-ce-de-shu-zi-he-mi-mi-shu-zi"></a>

现在有了用户输入和一个随机数，我们可以比较它们。这个步骤如示例 2-4 所示。注意这段代码还不能通过编译，我们稍后会解释。

```rust
// Some code
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

接着，底部的五行新代码使用了 `Ordering` 类型，`cmp` 方法用来比较两个值并可以在任何可比较的值上调用。它获取一个被比较值的引用：这里是把 `guess` 与 `secret_number` 做比较。然后它会返回一个刚才通过 `use` 引入作用域的 `Ordering` 枚举的成员。使用一个 [`match`](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html) 表达式，根据对 `guess` 和 `secret_number` 调用 `cmp` 返回的 `Ordering` 成员来决定接下来做什么。

一个 `match` 表达式由 **分支（arms）** 构成。一个分支包含一个 **模式**（_pattern_）和表达式开头的值与分支模式相匹配时应该执行的代码。Rust 获取提供给 `match` 的值并挨个检查每个分支的模式。`match` 结构和模式是 Rust 中强大的功能，它体现了代码可能遇到的多种情形，并帮助你确保没有遗漏处理。这些功能将分别在第六章和第十八章详细介绍。

让我们看看使用 `match` 表达式的例子。假设用户猜了 50，这时随机生成的秘密数字是 38。

比较 50 与 38 时，因为 50 比 38 要大，`cmp` 方法会返回 `Ordering::Greater`。`Ordering::Greater` 是 `match` 表达式得到的值。它检查第一个分支的模式，`Ordering::Less` 与 `Ordering::Greater`并不匹配，所以它忽略了这个分支的代码并来到下一个分支。下一个分支的模式是 `Ordering::Greater`，**正确** 匹配！这个分支关联的代码被执行，在屏幕打印出 `Too big!`。`match` 表达式会在第一次成功匹配后终止，因为该场景下没有检查最后一个分支的必要。

然而，示例的代码并不能编译，可以尝试一下：

错误的核心表明这里有 **不匹配的类型**（_mismatched types_）。Rust 有一个静态强类型系统，同时也有类型推断。当我们写出 `let guess = String::new()` 时，Rust 推断出 `guess` 应该是 `String` 类型，并不需要我们写出类型。另一方面，`secret_number`，是数字类型。几个数字类型拥有 1 到 100 之间的值：32 位数字 `i32`；32 位无符号数字 `u32`；64 位数字 `i64` 等等。Rust 默认使用 `i32`，所以它是 `secret_number` 的类型，除非增加类型信息，或任何能让 Rust 推断出不同数值类型的信息。这里错误的原因在于 Rust 不会比较字符串类型和数字类型。

所以我们必须把从输入中读取到的 `String` 转换为一个真正的数字类型，才好与秘密数字进行比较。这可以通过在 `main` 函数体中增加如下代码来实现：

```rust
// Some code

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
```

这里创建了一个叫做 `guess` 的变量。不过等等，不是已经有了一个叫做 `guess` 的变量了吗？确实如此，不过 Rust 允许用一个新值来 **隐藏** （_Shadowing_） `guess` 之前的值。这个功能常用在需要转换值类型之类的场景。它允许我们复用 `guess` 变量的名字，而不是被迫创建两个不同变量，诸如 `guess_str` 和 `guess` 之类。[第三章](https://kaisery.github.io/trpl-zh-cn/ch03-01-variables-and-mutability.html#shadowing)会介绍 shadowing 的更多细节，现在只需知道这个功能经常用于将一个类型的值转换为另一个类型的值。

我们将这个新变量绑定到 `guess.trim().parse()` 表达式上。表达式中的 `guess` 指的是包含输入的字符串类型 `guess` 变量。`String` 实例的 `trim` 方法会去除字符串开头和结尾的空白字符，我们必须执行此方法才能将字符串与 `u32` 比较，因为 `u32` 只能包含数值型数据。用户必须输入 enter 键才能让 `read_line` 返回并输入他们的猜想，这将会在字符串中增加一个换行（newline）符。例如，用户输入 5 并按下 enter（在 Windows 上，按下 enter 键会得到一个回车符和一个换行符，`\r`），`guess` 看起来像这样：`5` 或者 `5\r`。 代表 “换行”，回车键； 代表 “回车”，回车键。`trim` 方法会消除  或者 `\r`，只留下 `5.`

[字符串的 `parse` 方法](https://doc.rust-lang.org/std/primitive.str.html#method.parse) 将字符串转换成其他类型。这里用它来把字符串转换为数值。我们需要告诉 Rust 具体的数字类型，这里通过 `let guess: u32` 指定。<mark style="color:blue;">`guess`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">后面的冒号（</mark><mark style="color:blue;">`:`</mark><mark style="color:blue;">）告诉 Rust 我们指定了变量的类型。</mark>Rust 有一些内建的数字类型；`u32` 是一个无符号的 32 位整型。对于不大的正整数来说，它是不错的默认类型，[第三章](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#integer-types)还会讲到其他数字类型。

另外，程序中的 `u32` 注解以及与 `secret_number` 的比较，意味着 Rust 会推断出 `secret_number` 也是 `u32` 类型。现在可以使用相同类型比较两个值了！

`parse` 方法只有在字符逻辑上可以转换为数字的时候才能工作所以非常容易出错。例如，字符串中包含 `A👍%`，就无法将其转换为一个数字。因此，`parse` 方法返回一个 `Result` 类型。像之前 [“使用 `Result` 类型来处理潜在的错误”](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E4%BD%BF%E7%94%A8-result-%E7%B1%BB%E5%9E%8B%E6%9D%A5%E5%A4%84%E7%90%86%E6%BD%9C%E5%9C%A8%E7%9A%84%E9%94%99%E8%AF%AF) 讨论的 `read_line` 方法那样，再次按部就班的用 `expect` 方法处理即可。如果 `parse` 不能从字符串生成一个数字，返回一个 `Result` 的 `Err` 成员时，`expect` 会使游戏崩溃并打印附带的信息。如果 `parse` 成功地将字符串转换为一个数字，它会返回 `Result` 的 `Ok` 成员，然后 `expect` 会返回 `Ok` 值中的数字

### 使用循环来允许多次猜测 <a href="#shi-yong-xun-huan-lai-yun-xu-duo-ci-cai-ce" id="shi-yong-xun-huan-lai-yun-xu-duo-ci-cai-ce"></a>

`loop` 关键字创建了一个无限循环。我们会增加循环来给用户更多机会猜数字：

```rust
    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```

如上所示，我们将提示用户猜测之后的所有内容移动到了循环中。<mark style="color:blue;">确保 loop 循环中的代码多缩进四个空格</mark>，再次运行程序。注意这里有一个新问题，因为程序忠实地执行了我们的要求：永远地请求另一个猜测，用户好像无法退出啊！

用户总能使用 ctrl-c 终止程序。不过还有另一个方法跳出无限循环，就是 [“比较猜测与秘密数字”](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E6%AF%94%E8%BE%83%E7%8C%9C%E6%B5%8B%E7%9A%84%E6%95%B0%E5%AD%97%E5%92%8C%E7%A7%98%E5%AF%86%E6%95%B0%E5%AD%97) 部分提到的 `parse`：如果用户输入的答案不是一个数字，程序会崩溃。我们可以利用这一点来退出.

#### 猜测正确后退出 <a href="#cai-ce-zheng-que-hou-tui-chu" id="cai-ce-zheng-que-hou-tui-chu"></a>

让我们增加一个 `break` 语句，在用户猜对时退出游戏：

```rust
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

#### 处理无效输入 <a href="#chu-li-wu-xiao-shu-ru" id="chu-li-wu-xiao-shu-ru"></a>

为了进一步改善游戏性，不要在用户输入非数字时崩溃，需要忽略非数字，让用户可以继续猜测。可以通过修改 `guess` 将 `String` 转化为 `u32` 那部分代码来实现，如下所示：

```rust
        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");
```

我们将 `expect` 调用换成 `match` 语句，以从遇到错误就崩溃转换为处理错误。须知 <mark style="color:blue;">`parse`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">返回一个</mark> <mark style="color:blue;"></mark><mark style="color:blue;">`Result`</mark> <mark style="color:blue;"></mark><mark style="color:blue;">类型，</mark>而 `Result` 是一个拥有 `Ok` 或 `Err` 成员的枚举。这里使用的 `match` 表达式，和之前处理 `cmp` 方法返回 `Ordering` 时用的一样。

如果 `parse` 能够成功的将字符串转换为一个数字，它会返回一个包含结果数字的 `Ok`。这个 `Ok` 值与 `match` 第一个分支的模式相匹配，该分支对应的动作返回 `Ok` 值中的数字 `num`，最后如愿变成新创建的 `guess` 变量。

如果 `parse` **不**能将字符串转换为一个数字，它会返回一个包含更多错误信息的 `Err`。`Err` 值不能匹配第一个 `match` 分支的 `Ok(num)` 模式，但是会匹配第二个分支的 `Err(_)` 模式：`_` 是一个通配符值，本例中用来匹配所有 `Err` 值，不管其中有何种信息。所以程序会执行第二个分支的动作，`continue` 意味着进入 `loop` 的下一次循环，请求另一个猜测。这样程序就有效的忽略了 `parse` 可能遇到的所有错误！

















