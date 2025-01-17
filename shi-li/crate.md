# crate

## [crate](https://rustwiki.org/zh-CN/rust-by-example/crates.html#crate) <a href="#crate" id="crate"></a>

crate（中文有 “包，包装箱” 之意）是 Rust 的编译单元。当调用 `rustc some_file.rs` 时，`some_file.rs` 被当作 **crate 文件**。如果 `some_file.rs` 里面含有 `mod` 声明，那么模块文件的内容将在编译之前被插入 crate 文件的相应声明处。换句话说，模块**不会**单独被编译，只有 crate 才会被编译。

crate 可以编译成二进制可执行文件（binary）或库文件（library）。默认情况下，`rustc` 将从 crate 产生二进制可执行文件。这种行为可以通过 `rustc` 的选项 `--crate-type` 重载。

## [库](https://rustwiki.org/zh-CN/rust-by-example/crates/lib.html#%E5%BA%93)

让我们创建一个库，然后看看如何把它链接到另一个 crate。

```
pub fn public_function() {
    println!("called rary's `public_function()`");
}

fn private_function() {
    println!("called rary's `private_function()`");
}

pub fn indirect_access() {
    print!("called rary's `indirect_access()`, that\n> ");

    private_function();
}

```

```bash
$ rustc --crate-type=lib rary.rs
$ ls lib*
library.rlib
```

默认情况下，库会使用 crate 文件的名字，前面加上 “lib” 前缀，但这个默认名称可以使用 [`crate_name` 属性](https://rustwiki.org/zh-CN/rust-by-example/attribute/crate.html) 覆盖。

## [使用库](https://rustwiki.org/zh-CN/rust-by-example/crates/using_lib.html#%E4%BD%BF%E7%94%A8%E5%BA%93)

\
要将一个 crate 链接到上节新建的库，可以使用 `rustc` 的 `--extern` 选项。然后将所有的物件导入到与库名相同的模块下。此模块的操作通常与任何其他模块相同。

```rust
// extern crate rary; // 在 Rust 2015 版或更早版本需要这个导入语句

fn main() {
    rary::public_function();

    // 报错！ `private_function` 是私有的
    //rary::private_function();

    rary::indirect_access();
}
```

```bash
# library.rlib 是已编译好的库的路径，这里假设它在同一目录下：
$ rustc executable.rs --extern rary=library.rlib --edition=2018 && ./executable 
called rary's `public_function()`
called rary's `indirect_access()`, that
> called rary's `private_function()`
```

\
\
