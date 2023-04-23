# 入门

## 安装

linux安装使用命令:

```sh
// Some code
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

确保系统中存在链接器，一些常见的rust包依赖于C代码，因此需要一个C编译器,GCC是一个好的选择，ubuntu可以使用build-essential.

检查是否正确安装可以使用:

```sh
// Some code
rustc --version
```

更新可以使用:

```sh
// Some code
rustup update
```

卸载可以使用:

```sh
// Some code
rustup self uninstall
```

## hello world

```rust
// main.rs
fn main() {
    println!("Hello, world!");
}
```

在linux上运行:

```sh
// Some code
rustc main.rs
./main
```

hello world代码定义了一个`main`函数，`main`是rust程序的入口函数。

函数体应该包含在`{}`中,rust缩进使用4个空格。`println!`调用了一个rust宏，如果调用的是函数应该是`println`,宏与函数并不总是遵循相同的规则。

接下来看一下编译运行的步骤：

运行rust程序前必须进行编译:

```sh
// Some code
rustc main.rs
```

之后如上图会输出一个可执行文件，不同的系统会出现不同的文件，windows下会额外有一个main.pdb的文件包含了调试信息。

rust是一种预编译静态类型(AOT)语言。随着项目的增长，我们可能要管理项目的方方面面使代码易于分享，下面我们介绍一个叫cargo的工具。

## Cargo

cargo是rust的构建系统和包管理器，大多数人使用cargo来管理rust项目，因为可以处理很多任务，比如构建代码，下载依赖库并进行编译。

在编写复杂程序时使用cargo来启动项目将使得添加依赖项更加容易。

```sh
// Some code
cargo --version
```

### 使用cargo创建项目

```sh
// Some code
cargo new hello_cargo
cd hello_cargo
```

第一行命令新建了名为 _`hello_cargo`_ 的目录和项目。我们将项目命名为 _hello\_cargo_，同时 Cargo 在一个同名目录中创建项目文件。

进入 _hello\_cargo_ 目录并列出文件。将会看到 Cargo 生成了两个文件和一个目录：一个 _`Cargo.toml`_ 文件，一个 _`src`_ 目录，以及位于 _`src`_ 目录中的 _`main.rs`_ 文件。

这也会在 _hello\_cargo_ 目录初始化了一个 git 仓库，以及一个 _.gitignore_ 文件。如果在一个已经存在的 git 仓库中运行 `cargo new`，则这些 git 相关文件则不会生成；可以通过运行 `cargo new --vcs=git` 来覆盖这些行为。

{% hint style="info" %}
注意：Git 是一个常用的版本控制系统（version control system，VCS）。可以通过 `--vcs` 参数使 `cargo new` 切换到其它版本控制系统（VCS），或者不使用 VCS。运行 `cargo new --help` 参看可用的选项。
{% endhint %}

```toml
// Cargo.toml 
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

这个文件使用 [_TOML_](https://toml.io/) (_Tom's Obvious, Minimal Language_) 格式，这是 Cargo 配置文件的格式。

第一行，`[package]`，是一个片段（section）标题，表明下面的语句用来配置一个包。随着我们在这个文件增加更多的信息，还将增加其他片段（section）。

接下来的三行设置了 Cargo 编译程序所需的配置：项目的名称、项目的版本以及要使用的 Rust 版本。[附录 E](https://kaisery.github.io/trpl-zh-cn/appendix-05-editions.html)

最后一行，`[dependencies]`，是罗列项目依赖的片段的开始。在 Rust 中，代码包被称为 _crates_。这个项目并不需要其他的 crate，不过在第二章的第一个项目会用到依赖，那时会用得上这个片段。

现在打开 _src/main.rs_ 看看：

{% code overflow="wrap" %}
```rust
// Some code
fn main() {
    println!("Hello, world!");
}

```
{% endcode %}

Cargo 为你生成了一个 “Hello, world!” 程序，正如我们之前编写的示例 1-1！目前为止，我们的项目与 Cargo 生成项目的区别是 Cargo 将代码放在 _src_ 目录，同时项目根目录包含一个 _Cargo.toml_ 配置文件。

Cargo 期望源文件存放在 _src_ 目录中。项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件。使用 Cargo 帮助你保持项目干净整洁，一切井井有条。

如果没有使用 Cargo 开始项目，比如我们创建的 Hello,world! 项目，可以将其转化为一个 Cargo 项目。将代码放入 _src_ 目录，并创建一个合适的 _Cargo.toml_ 文件。

### 构建并运行cargo项目

现在让我们看看通过 Cargo 构建和运行 “Hello, world!” 程序有什么不同！在 _hello\_cargo_ 目录下，输入下面的命令来构建项目：

```
// Some code
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

这个命令会创建一个可执行文件 _target/debug/hello\_cargo_ （在 Windows 上是 _target\debug\hello\_cargo.exe_），而不是放在目前目录下。由于默认的构建方法是调试构建（debug build），Cargo 会将可执行文件放在名为 _debug_ 的目录中。可以通过这个命令运行可执行文件：

```
// Some code
$ ./target/debug/hello_cargo # 或者在 Windows 下为 .\target\debug\hello_cargo.exe
Hello, world!
```

如果一切顺利，终端上应该会打印出 `Hello, world!`。首次运行 `cargo build` 时，也会使 Cargo 在项目根目录创建一个新文件：_Cargo.lock_。这个文件记录项目依赖的实际版本。这个项目并没有依赖，所以其内容比较少。你自己永远也不需要碰这个文件，让 Cargo 处理它就行了。

我们刚刚使用 `cargo build` 构建了项目，并使用 `./target/debug/hello_cargo` 运行了程序，也可以使用 `cargo run` 在一个命令中同时编译并运行生成的可执行文件：

```
// Some code
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

比起要记得运行 `cargo build` 之后再用可执行文件的完整路径来运行程序，使用 `cargo run` 可以实现完全相同的效果，而且要方便得多，所以大多数开发者会使用 `cargo run`。

注意这一次并没有出现表明 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现文件并没有被改变，所以它并没有重新编译，而是直接运行了可执行文件。如果修改了源文件的话，Cargo 会在运行之前重新构建项目，并会出现像这样的输出：

```
// Some code
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo 还提供了一个叫 `cargo check` 的命令。该命令快速检查代码确保其可以编译，但并不产生可执行文件：

```
// Some code
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

为什么你会不需要可执行文件呢？通常 `cargo check` 要比 `cargo build` 快得多，因为它省略了生成可执行文件的步骤。如果你在编写代码时持续的进行检查，`cargo check` 可以让你快速了解现在的代码能不能正常通过编译！为此很多 Rustaceans 编写代码时定期运行 `cargo check` 确保它们可以编译。当准备好使用可执行文件时才运行 `cargo build`。

### 发布

当项目最终准备好发布时，可以使用 `cargo build --release` 来优化编译项目。这会在 _target/release_ 而不是 _target/debug_ 下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。这也就是为什么会有两种不同的配置：一种是为了开发，你需要经常快速重新构建；另一种是为用户构建最终程序，它们不会经常重新构建，并且希望程序运行得越快越好。如果你在测试代码的运行时间，请确保运行 `cargo build --release` 并使用 _target/release_ 下的可执行文件进行测试。



