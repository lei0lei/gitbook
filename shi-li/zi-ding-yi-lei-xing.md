# 自定义类型

## 自定义类型 <a href="#zi-ding-yi-lei-xing" id="zi-ding-yi-lei-xing"></a>

Rust 自定义数据类型主要是通过下面这两个关键字来创建：

* `struct`： 定义一个结构体（structure）
* `enum`： 定义一个枚举类型（enumeration）

而常量（constant）可以通过 `const` 和 `static` 关键字来创建。

## 结构体 <a href="#jie-gou-ti" id="jie-gou-ti"></a>

结构体（structure，缩写成 struct）有 3 种类型，使用 `struct` 关键字来创建：

* 元组结构体（tuple struct），事实上就是具名元组而已。
* 经典的 [C 语言风格结构体](https://en.wikipedia.org/wiki/Struct_\(C_programming_language\))（C struct）。
* 单元结构体（unit struct），不带字段，在泛型中很有用。

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// 单元结构体
struct Unit;

// 元组结构体
struct Pair(i32, f32);

// 带有两个字段的结构体
struct Point {
    x: f32,
    y: f32,
}

// 结构体可以作为另一个结构体的字段
#[allow(dead_code)]
struct Rectangle {
    // 可以在空间中给定左上角和右下角在空间中的位置来指定矩形。
    top_left: Point,
    bottom_right: Point,
}

fn main() {
    // 使用简单的写法初始化字段，并创建结构体
    let name = String::from("Peter");
    let age = 27;
    let peter = Person { name, age };

    // 以 Debug 方式打印结构体
    println!("{:?}", peter);

    // 实例化结构体 `Point`
    let point: Point = Point { x: 10.3, y: 0.4 };

    // 访问 point 的字段
    println!("point coordinates: ({}, {})", point.x, point.y);

    // 使用结构体更新语法创建新的 point，
    // 这样可以用到之前的 point 的字段
    let bottom_right = Point { x: 5.2, ..point };

    // `bottom_right.y` 与 `point.y` 一样，因为这个字段就是从 `point` 中来的
    println!("second point: ({}, {})", bottom_right.x, bottom_right.y);

    // 使用 `let` 绑定来解构 point
    let Point { x: left_edge, y: top_edge } = point;

    let _rectangle = Rectangle {
        // 结构体的实例化也是一个表达式
        top_left: Point { x: left_edge, y: top_edge },
        bottom_right: bottom_right,
    };

    // 实例化一个单元结构体
    let _unit = Unit;

    // 实例化一个元组结构体
    let pair = Pair(1, 0.1);

    // 访问元组结构体的字段
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // 解构一个元组结构体
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);
}

```

#### [动手试一试:](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html#%E5%8A%A8%E6%89%8B%E8%AF%95%E4%B8%80%E8%AF%95) <a href="#dong-shou-shi-yi-shi" id="dong-shou-shi-yi-shi"></a>

1. 增加一个计算 `Rectangle` （长方形）面积的函数 `rect_area`（尝试使用嵌套的解构方式）。
2. 增加一个函数 `square`，接受的参数是一个 `Point` 和一个 `f32`，并返回一个 `Rectangle`（长方形），其左上角位于该点上，长和宽都对应于 `f32`。

#### [参见:](https://rustwiki.org/zh-CN/rust-by-example/custom_types/structs.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[属性](https://rustwiki.org/zh-CN/rust-by-example/attribute.html) 和 [解构](https://rustwiki.org/zh-CN/rust-by-example/flow_control/match/destructuring.html)

## 枚举 <a href="#mei-ju" id="mei-ju"></a>

`enum` 关键字允许创建一个从数个不同取值中选其一的枚举类型（enumeration）。任何一个在 `struct` 中合法的取值在 `enum` 中也合法。

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

// 创建一个 `enum`（枚举）来对 web 事件分类。注意变量名和类型共同指定了 `enum`
// 取值的种类：`PageLoad` 不等于 `PageUnload`，`KeyPress(char)` 不等于
// `Paste(String)`。各个取值不同，互相独立。
enum WebEvent {
    // 一个 `enum` 可以是单元结构体（称为 `unit-like` 或 `unit`），
    PageLoad,
    PageUnload,
    // 或者一个元组结构体，
    KeyPress(char),
    Paste(String),
    // 或者一个普通的结构体。
    Click { x: i64, y: i64 }
}

// 此函数将一个 `WebEvent` enum 作为参数，无返回值。
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // 从 `enum` 里解构出 `c`。
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // 把 `Click` 解构给 `x` and `y`。
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` 从一个字符串切片中创建一个具有所有权的 `String`。
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}

```

### [类型别名](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum.html#%E7%B1%BB%E5%9E%8B%E5%88%AB%E5%90%8D) <a href="#lei-xing-bie-ming" id="lei-xing-bie-ming"></a>

若使用类型别名，则可以通过其别名引用每个枚举变量。当枚举的名称太长或者太一般化，且你想要对其重命名，那么这对你会有所帮助。

```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

// 创建一个类型别名
type Operations = VeryVerboseEnumOfThingsToDoWithNumbers;

fn main() {
    // 我们可以通过别名引用每个枚举变量，避免使用又长又不方便的枚举名字
    let x = Operations::Add;
}

```

最常见的情况就是在 `impl` 块中使用 `Self` 别名。

```rust
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

impl VeryVerboseEnumOfThingsToDoWithNumbers {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            Self::Add => x + y,
            Self::Subtract => x - y,
        }
    }
}

```

该功能已在 Rust 中稳定下来， 可以阅读 [stabilization report](https://github.com/rust-lang/rust/pull/61682/#issuecomment-502472847) 来了解更多有关枚举和类型别名的知识。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`match`](https://rustwiki.org/zh-CN/rust-by-example/flow_control/match.html), [`fn`](https://rustwiki.org/zh-CN/rust-by-example/fn.html), 和 [`String`](https://rustwiki.org/zh-CN/rust-by-example/std/str.html), [“类型别名枚举变量” 的 RFC](https://rust-lang.github.io/rfcs/2338-type-alias-enum-variants.html)

### 使用 use <a href="#shi-yong-use" id="shi-yong-use"></a>

使用 `use` 声明的话，就可以不写出名称的完整路径了：

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

enum Status {
    Rich,
    Poor,
}

enum Work {
    Civilian,
    Soldier,
}

fn main() {
    // 显式地 `use` 各个名称使他们直接可用，而不需要指定它们来自 `Status`。
    use Status::{Poor, Rich};
    // 自动地 `use` `Work` 内部的各个名称。
    use Work::*;

    // `Poor` 等价于 `Status::Poor`。
    let status = Poor;
    // `Civilian` 等价于 `Work::Civilian`。
    let work = Civilian;

    match status {
        // 注意这里没有用完整路径，因为上面显式地使用了 `use`。
        Rich => println!("The rich have lots of money!"),
        Poor => println!("The poor have no money..."),
    }

    match work {
        // 再次注意到没有用完整路径。
        Civilian => println!("Civilians work!"),
        Soldier  => println!("Soldiers fight!"),
    }
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/enum_use.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`match`](https://rustwiki.org/zh-CN/rust-by-example/flow_control/match.html) 和 [`use`](https://rustwiki.org/zh-CN/rust-by-example/mod/use.html)

### [C 风格用法](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/c_like.html#c-%E9%A3%8E%E6%A0%BC%E7%94%A8%E6%B3%95) <a href="#c-feng-ge-yong-fa" id="c-feng-ge-yong-fa"></a>

`enum` 也可以像 C 语言风格的枚举类型那样使用。

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

// 拥有隐式辨别值（implicit discriminator，从 0 开始）的 enum
enum Number {
    Zero,
    One,
    Two,
}

// 拥有显式辨别值（explicit discriminator）的 enum
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}

fn main() {
    // `enum` 可以转成整型。
    println!("zero is {}", Number::Zero as i32);
    println!("one is {}", Number::One as i32);

    println!("roses are #{:06x}", Color::Red as i32);
    println!("violets are #{:06x}", Color::Blue as i32);
}

```

#### [参考：](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/c_like.html#%E5%8F%82%E8%80%83) <a href="#can-kao" id="can-kao"></a>

[类型转换](https://rustwiki.org/zh-CN/rust-by-example/types/cast.html)

### [测试实例：链表](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/testcase_linked_list.html#%E6%B5%8B%E8%AF%95%E5%AE%9E%E4%BE%8B%E9%93%BE%E8%A1%A8)

`enum` 的一个常见用法就是创建链表（linked-list）：

```rust
use List::*;

enum List {
    // Cons：元组结构体，包含链表的一个元素和一个指向下一节点的指针
    Cons(u32, Box<List>),
    // Nil：末结点，表明链表结束
    Nil,
}

// 可以为 enum 定义方法
impl List {
    // 创建一个空的 List 实例
    fn new() -> List {
        // `Nil` 为 `List` 类型（译注：因 `Nil` 的完整名称是 `List::Nil`）
        Nil
    }

    // 处理一个 List，在其头部插入新元素，并返回该 List
    fn prepend(self, elem: u32) -> List {
        // `Cons` 同样为 List 类型
        Cons(elem, Box::new(self))
    }

    // 返回 List 的长度
    fn len(&self) -> u32 {
        // 必须对 `self` 进行匹配（match），因为这个方法的行为取决于 `self` 的
        // 取值种类。
        // `self` 为 `&List` 类型，`*self` 为 `List` 类型，匹配一个具体的 `T`
        // 类型要好过匹配引用 `&T`。
        match *self {
            // 不能得到 tail 的所有权，因为 `self` 是借用的；
            // 因此使用一个对 tail 的引用
            Cons(_, ref tail) => 1 + tail.len(),
            // （递归的）基准情形（base case）：一个长度为 0 的空列表
            Nil => 0
        }
    }

    // 返回列表的字符串表示（该字符串是堆分配的）
    fn stringify(&self) -> String {
        match *self {
            Cons(head, ref tail) => {
                // `format!` 和 `print!` 类似，但返回的是一个堆分配的字符串，
                // 而不是打印结果到控制台上
                format!("{}, {}", head, tail.stringify())
            },
            Nil => {
                format!("Nil")
            },
        }
    }
}

fn main() {
    // 创建一个空链表
    let mut list = List::new();

    // 追加一些元素
    list = list.prepend(1);
    list = list.prepend(2);
    list = list.prepend(3);

    // 显示链表的最后状态
    println!("linked list has length: {}", list.len());
    println!("{}", list.stringify());
}
rust
```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum/testcase_linked_list.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`Box`](https://rustwiki.org/zh-CN/rust-by-example/std/box.html) 和 [方法](https://rustwiki.org/zh-CN/rust-by-example/fn/methods.html)\
[常量](https://rustwiki.org/zh-CN/rust-by-example/custom_types/constants.html#%E5%B8%B8%E9%87%8F)
-----------------------------------------------------------------------------------------------

Rust 有两种常量，可以在任意作用域声明，包括全局作用域。它们都需要显式的类型声明：

* `const`：不可改变的值（通常使用这种）。
* `static`：具有 [`'static`](https://rustwiki.org/zh-CN/rust-by-example/scope/lifetime/static_lifetime.html) 生命周期的，可以是可变的变量（译注：须使用 `static mut` 关键字）。

有个特例就是 `"string"` 字面量。它可以不经改动就被赋给一个 `static` 变量，因为它的类型标记：`&'static str` 就包含了所要求的生命周期 `'static`。其他的引用类型都必须特地声明，使之拥有 `'static` 生命周期。这两种引用类型的差异似乎也无关紧要，因为无论如何，`static` 变量都得显式地声明。

```rust
// 全局变量是在所有其他作用域之外声明的。
static LANGUAGE: &'static str = "Rust";
const  THRESHOLD: i32 = 10;

fn is_big(n: i32) -> bool {
    // 在一般函数中访问常量
    n > THRESHOLD
}

fn main() {
    let n = 16;

    // 在 main 函数（主函数）中访问常量
    println!("This is {}", LANGUAGE);
    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // 报错！不能修改一个 `const` 常量。
    THRESHOLD = 5;
    // 改正 ^ 注释掉此行
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/custom_types/constants.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`const`/`static` RFC](https://github.com/rust-lang/rfcs/blob/master/text/0246-const-vs-static.md), [`'static` 生命周期](https://rustwiki.org/zh-CN/rust-by-example/scope/lifetime/static_lifetime.html)











