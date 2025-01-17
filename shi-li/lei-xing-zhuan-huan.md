# 类型转换

## [类型转换](https://rustwiki.org/zh-CN/rust-by-example/conversion.html#%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2) <a href="#lei-xing-zhuan-huan" id="lei-xing-zhuan-huan"></a>

Rust 使用 [trait](https://rustwiki.org/zh-CN/rust-by-example/trait.html) 解决类型之间的转换问题。最一般的转换会用到 [`From`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) 和 [`Into`](https://rustwiki.org/zh-CN/std/convert/trait.Into.html) 两个 trait。不过，即便常见的情况也可能会用到特别的 trait，尤其是从 `String` 转换到别的类型，以及把别的类型转换到 `String` 时。

## `From` 和 `Into` <a href="#from-he-into" id="from-he-into"></a>

[`From`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) 和 [`Into`](https://rustwiki.org/zh-CN/std/convert/trait.Into.html) 两个 trait 是内部相关联的，实际上这是它们实现的一部分。如果我们能够从类型 B 得到类型 A，那么很容易相信我们也能够把类型 B 转换为类型 A。

### [`From`](https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html#from) <a href="#from" id="from"></a>

[`From`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) trait 允许一种类型定义 “怎么根据另一种类型生成自己”，因此它提供了一种类型转换的简单机制。在标准库中有无数 `From` 的实现，规定原生类型及其他常见类型的转换功能。

比如，可以很容易地把 `str` 转换成 `String`：

```rust

#![allow(unused)]
fn main() {
let my_str = "hello";
let my_string = String::from(my_str);
}

```

也可以为我们自己的类型定义转换机制：

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let num = Number::from(30);
    println!("My number is {:?}", num);
}

```

## [`TryFrom` and `TryInto`](https://rustwiki.org/zh-CN/rust-by-example/conversion/try_from_try_into.html#tryfrom-and-tryinto)

类似于 [`From` 和 `Into`](https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html)，[`TryFrom`](https://rustwiki.org/zh-CN/std/convert/trait.TryFrom.html) 和 [`TryInto`](https://rustwiki.org/zh-CN/std/convert/trait.TryInto.html) 是类型转换的通用 trait。不同于 `From`/`Into` 的是，`TryFrom` 和 `TryInto` trait 用于易出错的转换，也正因如此，其返回值是 [`Result`](https://rustwiki.org/zh-CN/std/result/enum.Result.html) 型。

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(i32);

impl TryFrom<i32> for EvenNumber {
    type Error = ();

    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(())
        }
    }
}

fn main() {
    // TryFrom

    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(EvenNumber::try_from(5), Err(()));

    // TryInto

    let result: Result<EvenNumber, ()> = 8i32.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, ()> = 5i32.try_into();
    assert_eq!(result, Err(()));
}

```

## `ToString` 和 `FromStr`

### [`ToString`](https://rustwiki.org/zh-CN/rust-by-example/conversion/string.html#tostring) <a href="#tostring" id="tostring"></a>

要把任何类型转换成 `String`，只需要实现那个类型的 [`ToString`](https://rustwiki.org/zh-CN/std/string/trait.ToString.html) trait。然而不要直接这么做，您应该实现[`fmt::Display`](https://rustwiki.org/zh-CN/std/fmt/trait.Display.html) trait，它会自动提供 [`ToString`](https://rustwiki.org/zh-CN/std/string/trait.ToString.html)，并且还可以用来打印类型，就像 [`print!`](https://rustwiki.org/zh-CN/rust-by-example/hello/print.html) 一节中讨论的那样。

```rust
use std::fmt;

struct Circle {
    radius: i32
}

impl fmt::Display for Circle {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Circle of radius {}", self.radius)
    }
}

fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_string());
}

```

译注：一个实现 `ToString` 的例子

```rust
use std::string::ToString;

struct Circle {
    radius: i32
}

impl ToString for Circle {
    fn to_string(&self) -> String {
        format!("Circle of radius {:?}", self.radius)
    }
}

fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_string());
}

```

### [解析字符串](https://rustwiki.org/zh-CN/rust-by-example/conversion/string.html#%E8%A7%A3%E6%9E%90%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#jie-xi-zi-fu-chuan" id="jie-xi-zi-fu-chuan"></a>

我们经常需要把字符串转成数字。完成这项工作的标准手段是用 [`parse`](https://rustwiki.org/zh-CN/std/primitive.str.html#method.parse) 函数。我们得提供要转换到的类型，这可以通过不使用类型推断，或者用 “涡轮鱼” 语法（turbo fish，`<>`）实现。

只要对目标类型实现了 [`FromStr`](https://rustwiki.org/zh-CN/std/str/trait.FromStr.html) trait，就可以用 `parse` 把字符串转换成目标类型。 标准库中已经给无数种类型实现了 `FromStr`。如果要转换到用户定义类型，只要手动实现 `FromStr` 就行。

```rust
fn main() {
    let parsed: i32 = "5".parse().unwrap();
    let turbo_parsed = "10".parse::<i32>().unwrap();

    let sum = parsed + turbo_parsed;
    println!{"Sum: {:?}", sum};
}

```

\
[\
](https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html)\










