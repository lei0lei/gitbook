# 类型系统

## 类型系统 <a href="#lei-xing-xi-tong" id="lei-xing-xi-tong"></a>

Rust 提供了多种机制，用于改变或定义原生类型和用户定义类型。接下来会讲到：

* 原生类型的[类型转换](https://rustwiki.org/zh-CN/rust-by-example/types/cast.html)（cast）。
* 指定[字面量](https://rustwiki.org/zh-CN/rust-by-example/types/literals.html)的类型。
* 使用[类型推断](https://rustwiki.org/zh-CN/rust-by-example/types/inference.html)（type inference）。
* 给类型[取别名](https://rustwiki.org/zh-CN/rust-by-example/types/alias.html)（alias）。

## 类型转换 <a href="#lei-xing-zhuan-huan" id="lei-xing-zhuan-huan"></a>

Rust 不提供原生类型之间的隐式类型转换（coercion），但可以使用 `as` 关键字进行显式类型转换（casting）。

整型之间的转换大体遵循 C 语言的惯例，除了 C 会产生未定义行为的情形。在 Rust 中所有整型转换都是定义良好的。

```rust
// 不显示类型转换产生的溢出警告。
#![allow(overflowing_literals)]

fn main() {
    let decimal = 65.4321_f32;

    // 错误！不提供隐式转换
    let integer: u8 = decimal;
    // 改正 ^ 注释掉这一行

    // 可以显式转换
    let integer = decimal as u8;
    let character = integer as char;

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // 当把任何类型转换为无符号类型 T 时，会不断加上或减去 (std::T::MAX + 1)
    // 直到值位于新类型 T 的范围内。

    // 1000 已经在 u16 的范围内
    println!("1000 as a u16 is: {}", 1000 as u16);

    // 1000 - 256 - 256 - 256 = 232
    // 事实上的处理方式是：从最低有效位（LSB，least significant bits）开始保留
    // 8 位，然后剩余位置，直到最高有效位（MSB，most significant bit）都被抛弃。
    // 译注：MSB 就是二进制的最高位，LSB 就是二进制的最低位，按日常书写习惯就是
    // 最左边一位和最右边一位。
    println!("1000 as a u8 is : {}", 1000 as u8);
    // -1 + 256 = 255
    println!("  -1 as a u8 is : {}", (-1i8) as u8);

    // 对正数，这就和取模一样。
    println!("1000 mod 256 is : {}", 1000 % 256);

    // 当转换到有符号类型时，（位操作的）结果就和 “先转换到对应的无符号类型，
    // 如果 MSB 是 1，则该值为负” 是一样的。

    // 当然如果数值已经在目标类型的范围内，就直接把它放进去。
    println!(" 128 as a i16 is: {}", 128 as i16);
    // 128 转成 u8 还是 128，但转到 i8 相当于给 128 取八位的二进制补码，其值是：
    println!(" 128 as a i8 is : {}", 128 as i8);

    // 重复之前的例子
    // 1000 as u8 -> 232
    println!("1000 as a u8 is : {}", 1000 as u8);
    // 232 的二进制补码是 -24
    println!(" 232 as a i8 is : {}", 232 as i8);
}

```

* `fun(&foo)` 用**传引用**（pass by reference）的方式把变量传给函数，而非传值（pass by value，写法是 `fun(foo)`）。更多细节请看[借用](https://rustwiki.org/zh-CN/rust-by-example/scope/borrow.html)。
* `std::mem::size_of_val` 是一个函数，这里使用其**完整路径**（full path）调用。代码可以分成一些叫做**模块**（module）的逻辑单元。在本例中，`size_of_val` 函数是在 `mem` 模块中定义的，而 `mem` 模块又是在 `std` **crate** 中定义的。更多细节请看[模块](https://rustwiki.org/zh-CN/rust-by-example/mod.html)和[crate](https://rustwiki.org/zh-CN/rust-by-example/crates.html).

上面的代码使用了一些还没有讨论过的概念。心急的读者可以看看下面的简短解释：

```
fn main() {    // 带后缀的字面量，其类型在初始化时已经知道了。    let x = 1u8;    let y = 2u32;    let z = 3f32;    // 无后缀的字面量，其类型取决于如何使用它们。    let i = 1;    let f = 1.0;    // `size_of_val` 返回一个变量所占的字节数    println!("size of `x` in bytes: {}", std::mem::size_of_val(&x));    println!("size of `y`https://rustwiki.org/zh-CN/rust-by-example/types/literals.html#%E5%AD%97%E9%9D%A2%E9%87%8F in bytes: {}", std::mem::size_of_val(&y));    println!("size of `z` in bytes: {}", std::mem::size_of_val(&z));    println!("size of `i` in bytes: {}", std::mem::size_of_val(&i));    println!("size of `f` in bytes: {}", std::mem::size_of_val(&f));}
```

无后缀的数值字面量，其类型取决于怎样使用它们。如果没有限制，编译器会对整数使用 `i32`，对浮点数使用 `f64`。

## [字面量](https://rustwiki.org/zh-CN/rust-by-example/types/literals.html#%E5%AD%97%E9%9D%A2%E9%87%8F) <a href="#zi-mian-liang" id="zi-mian-liang"></a>

对数值字面量，只要把类型作为后缀加上去，就完成了类型说明。比如指定字面量 `42` 的类型是 `i32`，只需要写 `42i32`。

无后缀的数值字面量，其类型取决于怎样使用它们。如果没有限制，编译器会对整数使用 `i32`，对浮点数使用 `f64`。

```rust
fn main() {
    // 带后缀的字面量，其类型在初始化时已经知道了。
    let x = 1u8;
    let y = 2u32;
    let z = 3f32;

    // 无后缀的字面量，其类型取决于如何使用它们。
    let i = 1;
    let f = 1.0;

    // `size_of_val` 返回一个变量所占的字节数
    println!("size of `x` in bytes: {}", std::mem::size_of_val(&x));
    println!("size of `y`https://rustwiki.org/zh-CN/rust-by-example/types/literals.html#%E5%AD%97%E9%9D%A2%E9%87%8F in bytes: {}", std::mem::size_of_val(&y));
    println!("size of `z` in bytes: {}", std::mem::size_of_val(&z));
    println!("size of `i` in bytes: {}", std::mem::size_of_val(&i));
    println!("size of `f` in bytes: {}", std::mem::size_of_val(&f));
}

```

上面的代码使用了一些还没有讨论过的概念。心急的读者可以看看下面的简短解释：

* `fun(&foo)` 用**传引用**（pass by reference）的方式把变量传给函数，而非传值（pass by value，写法是 `fun(foo)`）。更多细节请看[借用](https://rustwiki.org/zh-CN/rust-by-example/scope/borrow.html)。
* `std::mem::size_of_val` 是一个函数，这里使用其**完整路径**（full path）调用。代码可以分成一些叫做**模块**（module）的逻辑单元。在本例中，`size_of_val` 函数是在 `mem` 模块中定义的，而 `mem` 模块又是在 `std` **crate** 中定义的。更多细节请看[模块](https://rustwiki.org/zh-CN/rust-by-example/mod.html)和[crate](https://rustwiki.org/zh-CN/rust-by-example/crates.html).

## 类型推断

Rust 的类型推断引擎是很聪明的，它不只是在初始化时看看[右值](https://en.wikipedia.org/wiki/Value_\(computer_science\)#lrvalue)（r-value）的类型而已，它还会考察变量之后会怎样使用，借此推断类型。这是一个类型推导的进阶例子：

```rust
fn main() {
    // 因为有类型说明，编译器知道 `elem` 的类型是 u8。
    let elem = 5u8;

    // 创建一个空向量（vector，即不定长的，可以增长的数组）。
    let mut vec = Vec::new();
    // 现在编译器还不知道 `vec` 的具体类型，只知道它是某种东西构成的向量（`Vec<_>`）
    
    // 在向量中插入 `elem`。
    vec.push(elem);
    // 啊哈！现在编译器知道 `vec` 是 u8 的向量了（`Vec<u8>`）。
    // 试一试 ^ 注释掉 `vec.push(elem)` 这一行。

    println!("{:?}", vec);
}

```

没有必要写类型说明，编译器和程序员皆大欢喜！

## 别名

可以用 `type` 语句给已有的类型取个新的名字。类型的名字必须遵循驼峰命名法（像是 `CamelCase` 这样），否则编译器将给出警告。原生类型是例外，比如： `usize`、`f32`，等等。

```rust
// `NanoSecond` 是 `u64` 的新名字。
type NanoSecond = u64;
type Inch = u64;

// 通过这个属性屏蔽警告。
#[allow(non_camel_case_types)]
type u64_t = u64;
// 试一试 ^ 移除上面那个属性

fn main() {
    // `NanoSecond` = `Inch` = `u64_t` = `u64`.
    let nanoseconds: NanoSecond = 5 as u64_t;
    let inches: Inch = 2 as u64_t;

    // 注意类型别名*并不能*提供额外的类型安全，因为别名*并不是*新的类型。
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}

```

别名的主要用途是避免写出冗长的模板化代码（boilerplate code）。如 `IoResult<T>` 是 `Result<T, IoError>` 类型的别名。

#### [参见:](https://rustwiki.org/zh-CN/rust-by-example/types/alias.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[属性](https://rustwiki.org/zh-CN/rust-by-example/attribute.html)\
[\
](https://rustwiki.org/zh-CN/rust-by-example/types.html)























