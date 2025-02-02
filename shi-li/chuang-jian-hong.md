# 创建宏

## [使用 `macro_rules!` 来创建宏](https://rustwiki.org/zh-CN/rust-by-example/macros.html#%E4%BD%BF%E7%94%A8-macro_rules-%E6%9D%A5%E5%88%9B%E5%BB%BA%E5%AE%8F) <a href="#shi-yong-macrorules-lai-chuang-jian-hong" id="shi-yong-macrorules-lai-chuang-jian-hong"></a>

Rust 提供了一个强大的宏系统，可进行元编程（metaprogramming）。你已经在前面的章节中看到，宏看起来和函数很像，只不过名称末尾有一个感叹号 `!` 。宏并不产生函数调用，而是展开成源码，并和程序的其余部分一起被编译。Rust 又有一点和 C 以及其他语言都不同，那就是 Rust 的宏会展开为抽象语法树（AST，abstract syntax tree），而不是像字符串预处理那样直接替换成代码，这样就不会产生无法预料的优先权错误。

宏是通过 `macro_rules!` 宏来创建的。

```
// 这是一个简单的宏，名为 `say_hello`。
macro_rules! say_hello {
    // `()` 表示此宏不接受任何参数。
    () => (
        // 此宏将会展开成这个代码块里面的内容。
        println!("Hello!");
    )
}

fn main() {
    // 这个调用将会展开成 `println("Hello");`!
    say_hello!()
}

```

为什么宏是有用的？

1. 不写重复代码（DRY，Don't repeat yourself.）。很多时候你需要在一些地方针对不同 的类型实现类似的功能，这时常常可以使用宏来避免重复代码（稍后详述）。
2. 领域专用语言（DSL，domain-specific language）。宏允许你为特定的目的创造特定的 语法（稍后详述）。
3. 可变接口（variadic interface）。有时你需要能够接受不定数目参数的接口，比如 `println!`，根据格式化字符串的不同，它需要接受任意多的参数（稍后详述）。

## [语法](https://rustwiki.org/zh-CN/rust-by-example/macros/syntax.html#%E8%AF%AD%E6%B3%95) <a href="#yu-fa" id="yu-fa"></a>

在下面的小节中，我们将展示如何在 Rust 中定义宏。基本的概念有三个：

* [模式与指示符](https://rustwiki.org/zh-CN/rust-by-example/macros/designators.html)
* [重载](https://rustwiki.org/zh-CN/rust-by-example/macros/overload.html)
* [重复](https://rustwiki.org/zh-CN/rust-by-example/macros/repeat.html)

### &#x20;[指示符](https://rustwiki.org/zh-CN/rust-by-example/macros/designators.html#%E6%8C%87%E7%A4%BA%E7%AC%A6)

宏的参数使用一个美元符号 `$` 作为前缀，并使用一个**指示符**（designator）来注明类型：

```
macro_rules! create_function {
    // 此宏接受一个 `ident` 指示符表示的参数，并创建一个名为 `$func_name` 的函数。
    // `ident` 指示符用于变量名或函数名
    ($func_name:ident) => (
        fn $func_name() {
            // `stringify!` 宏把 `ident` 转换成字符串。
            println!("You called {:?}()",
                     stringify!($func_name))
        }
    )
}

// 借助上述宏来创建名为 `foo` 和 `bar` 的函数。
create_function!(foo);
create_function!(bar);

macro_rules! print_result {
    // 此宏接受一个 `expr` 类型的表达式，并将它作为字符串，连同其结果一起
    // 打印出来。
    // `expr` 指示符表示表达式。
    ($expression:expr) => (
        // `stringify!` 把表达式*原样*转换成一个字符串。
        println!("{:?} = {:?}",
                 stringify!($expression),
                 $expression)
    )
}

fn main() {
    foo();
    bar();

    print_result!(1u32 + 1);

    // 回想一下，代码块也是表达式！
    print_result!({
        let x = 1u32;

        x * x + 2 * x - 1
    });
}

```

这里列出全部指示符：

* `block`
* `expr` 用于表达式
* `ident` 用于变量名或函数名
* `item`
* `literal` 用于字面常量
* `pat` (**模式** _pattern_)
* `path`
* `stmt` (**语句** _statement_)
* `tt` (**标记树** _token tree_)
* `ty` (**类型** _type_)
* `vis` (_可见性描述符_)

完整列表详见 [Rust Reference](https://doc.rust-lang.org/reference/macros-by-example.html)。

### [重载](https://rustwiki.org/zh-CN/rust-by-example/macros/overload.html#%E9%87%8D%E8%BD%BD) <a href="#zhong-zai" id="zhong-zai"></a>

宏可以重载，从而接受不同的参数组合。在这方面，`macro_rules!` 的作用类似于匹配（match）代码块：

```
// 根据你调用它的方式，`test!` 将以不同的方式来比较 `$left` 和 `$right`。
macro_rules! test {
    // 参数不需要使用逗号隔开。
    // 参数可以任意组合！
    ($left:expr; and $right:expr) => (
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    );
    // ^ 每个分支都必须以分号结束。
    ($left:expr; or $right:expr) => (
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    );
}

fn main() {
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}

```

### [重复](https://rustwiki.org/zh-CN/rust-by-example/macros/repeat.html#%E9%87%8D%E5%A4%8D)

宏在参数列表中可以使用 `+` 来表示一个参数可能出现一次或多次，使用 `*` 来表示该参数可能出现零次或多次。

在下面例子中，把模式这样： `$(...),+` 包围起来，就可以匹配一个或多个用逗号隔开的表达式。另外注意到，宏定义的最后一个分支可以不用分号作为结束。

```
// `min!` 将求出任意数量的参数的最小值。
macro_rules! find_min {
    // 基本情形：
    ($x:expr) => ($x);
    // `$x` 后面跟着至少一个 `$y,`
    ($x:expr, $($y:expr),+) => (
        // 对 `$x` 后面的 `$y` 们调用 `find_min!` 
        std::cmp::min($x, find_min!($($y),+))
    )
}

fn main() {
    println!("{}", find_min!(1u32));
    println!("{}", find_min!(1u32 + 2 , 2u32));
    println!("{}", find_min!(5u32, 2u32 * 3, 4u32));
}

```

[DRY (不写重复代码)](https://rustwiki.org/zh-CN/rust-by-example/macros/dry.html#dry-%E4%B8%8D%E5%86%99%E9%87%8D%E5%A4%8D%E4%BB%A3%E7%A0%81)\



通过提取函数或测试集的公共部分，宏可以让你写出 DRY 的代码（DRY 是 Don't Repeat Yourself 的缩写，意思为 “不要写重复代码”）。这里给出一个例子，对 `Vec<T>` 实现并测试了关于 `+=`、`*=` 和 `-=` 等运算符。

```
use std::ops::{Add, Mul, Sub};

macro_rules! assert_equal_len {
    // `tt`（token tree，标记树）指示符表示运算符和标记。
    ($a:ident, $b: ident, $func:ident, $op:tt) => (
        assert!($a.len() == $b.len(),
                "{:?}: dimension mismatch: {:?} {:?} {:?}",
                stringify!($func),
                ($a.len(),),
                stringify!($op),
                ($b.len(),));
    )
}

macro_rules! op {
    ($func:ident, $bound:ident, $op:tt, $method:ident) => (
        fn $func<T: $bound<T, Output=T> + Copy>(xs: &mut Vec<T>, ys: &Vec<T>) {
            assert_equal_len!(xs, ys, $func, $op);

            for (x, y) in xs.iter_mut().zip(ys.iter()) {
                *x = $bound::$method(*x, *y);
                // *x = x.$method(*y);
            }
        }
    )
}

// 实现 `add_assign`、`mul_assign` 和 `sub_assign` 等函数。
op!(add_assign, Add, +=, add);
op!(mul_assign, Mul, *=, mul);
op!(sub_assign, Sub, -=, sub);

mod test {
    use std::iter;
    macro_rules! test {
        ($func: ident, $x:expr, $y:expr, $z:expr) => {
            #[test]
            fn $func() {
                for size in 0usize..10 {
                    let mut x: Vec<_> = iter::repeat($x).take(size).collect();
                    let y: Vec<_> = iter::repeat($y).take(size).collect();
                    let z: Vec<_> = iter::repeat($z).take(size).collect();

                    super::$func(&mut x, &y);

                    assert_eq!(x, z);
                }
            }
        }
    }

    // 测试 `add_assign`、`mul_assign` 和 `sub_assign`
    test!(add_assign, 1u32, 2u32, 3u32);
    test!(mul_assign, 2u32, 3u32, 6u32);
    test!(sub_assign, 3u32, 2u32, 1u32);
}


```

```bash
$ rustc --test dry.rs && ./dry
running 3 tests
test test::mul_assign ... ok
test test::add_assign ... ok
test test::sub_assign ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured
```

## [DSL（领域专用语言）](https://rustwiki.org/zh-CN/rust-by-example/macros/dsl.html#dsl%E9%A2%86%E5%9F%9F%E4%B8%93%E7%94%A8%E8%AF%AD%E8%A8%80)

DSL 是 Rust 的宏中集成的微型 “语言”。这种语言是完全合法的，因为宏系统会把它转换成普通的 Rust 语法树，它只不过看起来像是另一种语言而已。这就允许你为一些特定功能创造一套简洁直观的语法（当然是有限制的）。

比如说我想要定义一套小的计算器 API，可以传给它表达式，它会把结果打印到控制台上。

```
macro_rules! calculate {
    (eval $e:expr) => {{
        {
            let val: usize = $e; // 强制类型为整型
            println!("{} = {}", stringify!{$e}, val);
        }
    }};
}

fn main() {
    calculate! {
        eval 1 + 2 // 看到了吧，`eval` 可并不是 Rust 的关键字！
    }

    calculate! {
        eval (1 + 2) * (3 / 4)
    }
}

```

输出:

```txt
1 + 2 = 3
(1 + 2) * (3 / 4) = 0
```

这个例子非常简单，但是已经有很多利用宏开发的复杂接口了，比如 [`lazy_static`](https://crates.io/crates/lazy_static) 和 [`clap`](https://crates.io/crates/clap)。\


## [可变参数接口](https://rustwiki.org/zh-CN/rust-by-example/macros/variadics.html#%E5%8F%AF%E5%8F%98%E5%8F%82%E6%95%B0%E6%8E%A5%E5%8F%A3) <a href="#ke-bian-can-shu-jie-kou" id="ke-bian-can-shu-jie-kou"></a>

可变参数接口可以接受任意数目的参数。比如说 `println` 就可以，其参数的数目是由格式化字符串指定的。

我们可以把之前的 `calculate!` 宏改写成可变参数接口：

```
macro_rules! calculate {
    // 单个 `eval` 的模式
    (eval $e:expr) => {{
        {
            let val: usize = $e; // Force types to be integers
            println!("{} = {}", stringify!{$e}, val);
        }
    }};

    // 递归地拆解多重的 `eval`
    (eval $e:expr, $(eval $es:expr),+) => {{
        calculate! { eval $e }
        calculate! { $(eval $es),+ }
    }};
}

fn main() {
    calculate! { // 妈妈快看，可变参数的 `calculate!`！
        eval 1 + 2,
        eval 3 + 4,
        eval (2 * 3) + 1
    }
}

```

输出：

```txt
1 + 2 = 3
3 + 4 = 7
(2 * 3) + 1 = 7
```
