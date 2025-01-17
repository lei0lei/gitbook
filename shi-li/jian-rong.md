# 兼容

## [兼容性](https://rustwiki.org/zh-CN/rust-by-example/compatibility.html#%E5%85%BC%E5%AE%B9%E6%80%A7) <a href="#jian-rong-xing" id="jian-rong-xing"></a>

Rust 语言正在快速发展，因此尽管努力确保尽可能向前兼容，但仍可能出现某些兼容性问题。

* [原始标识符](https://rustwiki.org/zh-CN/rust-by-example/compatibility/raw_identifiers.html)

## [原始标志符](https://rustwiki.org/zh-CN/rust-by-example/compatibility/raw_identifiers.html#%E5%8E%9F%E5%A7%8B%E6%A0%87%E5%BF%97%E7%AC%A6) <a href="#yuan-shi-biao-zhi-fu" id="yuan-shi-biao-zhi-fu"></a>

与许多编程语言一样，Rust 拥有“关键字”的概念。 这些标识符对语言有特定意义，所以不能在变量名、函数名和其他位置使用它们。 原始标识符允许你使用通常不允许的关键字。 当 Rust 引入新关键字时，使用旧版 Rust 的库拥有与新版本中引入的关键字同名的变量或函数，这一点就特别有用。

举个例子，使用 2015 版 Rust 编译的 crate `foo`，它导出一个名为 `try` 的函数。 此关键字（_try_）在 2018 版本的新功能中保留下来，因此如果没有原始标识符，我们将无法命名该功能。

```rust
extern crate foo;

fn main() {
    foo::try();
}
```

将得到如下错误：

```
error: expected identifier, found keyword `try`
 --> src/main.rs:4:4
  |
4 | foo::try();
  |      ^^^ expected identifier, found keyword
```

使用原始标志符重写上述代码：

```rust
extern crate foo;

fn main() {
    foo::r#try();
}
```
