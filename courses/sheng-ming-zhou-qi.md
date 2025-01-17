# 生命周期

## [深入生命周期](https://course.rs/advance/lifetime/advance.html#%E6%B7%B1%E5%85%A5%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F) <a href="#shen-ru-sheng-ming-zhou-qi" id="shen-ru-sheng-ming-zhou-qi"></a>

其实关于生命周期的常用特性，在上一节中，我们已经概括得差不多了，本章主要讲解生命周期的一些高级或者不为人知的特性。对于新手，完全可以跳过本节内容，进行下一章节的学习。

### [不太聪明的生命周期检查](https://course.rs/advance/lifetime/advance.html#%E4%B8%8D%E5%A4%AA%E8%81%AA%E6%98%8E%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%A3%80%E6%9F%A5) <a href="#bu-tai-cong-ming-de-sheng-ming-zhou-qi-jian-cha" id="bu-tai-cong-ming-de-sheng-ming-zhou-qi-jian-cha"></a>

在 Rust 语言学习中，一个很重要的部分就是阅读一些你可能不经常遇到，但是一旦遇到就难以理解的代码，这些代码往往最令人头疼的就是生命周期，这里我们就来看看一些本以为可以编译，但是却因为生命周期系统不够聪明导致编译失败的代码。

[**例子 1**](https://course.rs/advance/lifetime/advance.html#%E4%BE%8B%E5%AD%90-1)

```rust
#[derive(Debug)]
struct Foo;

impl Foo {
    fn mutate_and_share(&mut self) -> &Self {
        &*self
    }
    fn share(&self) {}
}

fn main() {
    let mut foo = Foo;
    let loan = foo.mutate_and_share();
    foo.share();
    println!("{:?}", loan);
}

```

上面的代码中，`foo.mutate_and_share()` 虽然借用了 `&mut self`，但是它最终返回的是一个 `&self`，然后赋值给 `loan`，因此理论上来说它最终是进行了不可变借用，同时 `foo.share` 也进行了不可变借用，那么根据 Rust 的借用规则：多个不可变借用可以同时存在，因此该代码应该编译通过。

事实上，运行代码后，你将看到一个错误：

```console
error[E0502]: cannot borrow `foo` as immutable because it is also borrowed as mutable
  --> src/main.rs:12:5
   |
11 |     let loan = foo.mutate_and_share();
   |                ---------------------- mutable borrow occurs here
12 |     foo.share();
   |     ^^^^^^^^^^^ immutable borrow occurs here
13 |     println!("{:?}", loan);
   |                      ---- mutable borrow later used here
```

编译器的提示在这里其实有些难以理解，因为可变借用仅在 `mutate_and_share` 方法内部有效，出了该方法后，就只有返回的不可变借用，因此，按理来说可变借用不应该在 `main` 的作用范围内存在。

对于这个反直觉的事情，让我们用生命周期来解释下，可能你就很好理解了：

```rust
struct Foo;

impl Foo {
    fn mutate_and_share<'a>(&'a mut self) -> &'a Self {
        &'a *self
    }
    fn share<'a>(&'a self) {}
}

fn main() {
    'b: {
        let mut foo: Foo = Foo;
        'c: {
            let loan: &'c Foo = Foo::mutate_and_share::<'c>(&'c mut foo);
            'd: {
                Foo::share::<'d>(&'d foo);
            }
            println!("{:?}", loan);
        }
    }
}

```

以上是模拟了编译器的生命周期标注后的代码，可以注意到 `&mut foo` 和 `loan` 的生命周期都是 `'c`。

还记得生命周期消除规则中的第三条吗？因为该规则，导致了 `mutate_and_share` 方法中，参数 `&mut self` 和返回值 `&self` 的生命周期是相同的，因此，若返回值的生命周期在 `main` 函数有效，那 `&mut self` 的借用也是在 `main` 函数有效。

这就解释了可变借用为啥会在 `main` 函数作用域内有效，最终导致 `foo.share()` 无法再进行不可变借用。

总结下：`&mut self` 借用的生命周期和 `loan` 的生命周期相同，将持续到 `println` 结束。而在此期间 `foo.share()` 又进行了一次不可变 `&foo` 借用，违背了可变借用与不可变借用不能同时存在的规则，最终导致了编译错误。

上述代码实际上完全是正确的，但是因为生命周期系统的“粗糙实现”，导致了编译错误，目前来说，遇到这种生命周期系统不够聪明导致的编译错误，我们也没有太好的办法，只能修改代码去满足它的需求，并期待以后它会更聪明。

[**例子 2**](https://course.rs/advance/lifetime/advance.html#%E4%BE%8B%E5%AD%90-2)

再来看一个例子：

```rust

#![allow(unused)]
fn main() {
fn f<'a, T>(x: *const T) -> &'a T {
    unsafe {
        &*x
    }
}
}

```

这段代码不能通过编译的原因是编译器未能精确地判断出某个可变借用不再需要，反而谨慎的给该借用安排了一个很大的作用域，结果导致后续的借用失败：

```console
error[E0499]: cannot borrow `*map` as mutable more than once at a time
  --> src/main.rs:13:17
   |
5  |       fn get_default<'m, K, V>(map: &'m mut HashMap<K, V>, key: K) -> &'m mut V
   |                      -- lifetime `'m` defined here
...
10 |           match map.get_mut(&key) {
   |           -     ----------------- first mutable borrow occurs here
   |  _________|
   | |
11 | |             Some(value) => value,
12 | |             None => {
13 | |                 map.insert(key.clone(), V::default());
   | |                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ second mutable borrow occurs here
14 | |                 map.get_mut(&key).unwrap()
15 | |             }
16 | |         }
   | |_________- returning this value requires that `*map` is borrowed for `'m`
```

分析代码可知在 `match map.get_mut(&key)` 方法调用完成后，对 `map` 的可变借用就可以结束了。但从报错看来，编译器不太聪明，它认为该借用会持续到整个 `match` 语句块的结束(第 16 行处)，这便造成了后续借用的失败。

类似的例子还有很多，由于篇幅有限，就不在这里一一列举，如果大家想要阅读更多的类似代码，可以看看[<\<Rust 代码鉴赏>>](https://github.com/sunface/rust-codes)一书。

### [无界生命周期](https://course.rs/advance/lifetime/advance.html#%E6%97%A0%E7%95%8C%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F) <a href="#wu-jie-sheng-ming-zhou-qi" id="wu-jie-sheng-ming-zhou-qi"></a>

不安全代码(`unsafe`)经常会凭空产生引用或生命周期，这些生命周期被称为是 **无界(unbound)** 的。

无界生命周期往往是在解引用一个裸指针(裸指针 raw pointer)时产生的，换句话说，它是凭空产生的，因为输入参数根本就没有这个生命周期：

```rust
fn f<'a, T>(x: *const T) -> &'a T {    unsafe {        &*x    }}
```

上述代码中，参数 `x` 是一个裸指针，它并没有任何生命周期，然后通过 `unsafe` 操作后，它被进行了解引用，变成了一个 Rust 的标准引用类型，该类型必须要有生命周期，也就是 `'a`。

可以看出 `'a` 是凭空产生的，因此它是无界生命周期。这种生命周期由于没有受到任何约束，因此它想要多大就多大，这实际上比 `'static` 要强大。例如 `&'static &'a T` 是无效类型，但是无界生命周期 `&'unbounded &'a T` 会被视为 `&'a &'a T` 从而通过编译检查，因为它可大可小，就像孙猴子的金箍棒一般。

我们在实际应用中，要尽量避免这种无界生命周期。最简单的避免无界生命周期的方式就是在函数声明中运用生命周期消除规则。**若一个输出生命周期被消除了，那么必定因为有一个输入生命周期与之对应**。

### [生命周期约束 HRTB](https://course.rs/advance/lifetime/advance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%BA%A6%E6%9D%9F-hrtb) <a href="#sheng-ming-zhou-qi-yue-shu-hrtb" id="sheng-ming-zhou-qi-yue-shu-hrtb"></a>

生命周期约束跟特征约束类似，都是通过形如 `'a: 'b` 的语法，来说明两个生命周期的长短关系。

[**'a: 'b**](https://course.rs/advance/lifetime/advance.html#a-b)

假设有两个引用 `&'a i32` 和 `&'b i32`，它们的生命周期分别是 `'a` 和 `'b`，若 `'a` >= `'b`，则可以定义 `'a:'b`，表示 `'a` 至少要活得跟 `'b` 一样久。

```rust

#![allow(unused)]
fn main() {
struct DoubleRef<'a,'b:'a, T> {
    r: &'a T,
    s: &'b T
}
}

```

例如上述代码定义一个结构体，它拥有两个引用字段，类型都是泛型 `T`，每个引用都拥有自己的生命周期，由于我们使用了生命周期约束 `'b: 'a`，因此 `'b` 必须活得比 `'a` 久，也就是结构体中的 `s` 字段引用的值必须要比 `r` 字段引用的值活得要久。

[**T: 'a**](https://course.rs/advance/lifetime/advance.html#t-a)

表示类型 `T` 必须比 `'a` 活得要久：

```rust

#![allow(unused)]
fn main() {
struct Ref<'a, T: 'a> {
    r: &'a T
}
}

```

因为结构体字段 `r` 引用了 `T`，因此 `r` 的生命周期 `'a` 必须要比 `T` 的生命周期更短(被引用者的生命周期必须要比引用长)。

在 Rust 1.30 版本之前，该写法是必须的，但是从 1.31 版本开始，编译器可以自动推导 `T: 'a` 类型的约束，因此我们只需这样写即可：

```rust

#![allow(unused)]
fn main() {
struct Ref<'a, T> {
    r: &'a T
}
}

```

来看一个使用了生命周期约束的综合例子：

```rust

#![allow(unused)]
fn main() {
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
}

```

上面的例子中必须添加约束 `'a: 'b` 后，才能成功编译，因为 `self.part` 的生命周期与 `self`的生命周期一致，将 `&'a` 类型的生命周期强行转换为 `&'b` 类型，会报错，只有在 `'a` >= `'b` 的情况下，`'a` 才能转换成 `'b`。

### [闭包函数的消除规则](https://course.rs/advance/lifetime/advance.html#%E9%97%AD%E5%8C%85%E5%87%BD%E6%95%B0%E7%9A%84%E6%B6%88%E9%99%A4%E8%A7%84%E5%88%99) <a href="#bi-bao-han-shu-de-xiao-chu-gui-ze" id="bi-bao-han-shu-de-xiao-chu-gui-ze"></a>

先来看一段简单的代码：

```rust

#![allow(unused)]
fn main() {
fn fn_elision(x: &i32) -> &i32 { x }
let closure_slision = |x: &i32| -> &i32 { x };
}

```

乍一看，这段代码比古天乐还平平无奇，能有什么问题呢？来，拄拐走两圈试试：

```console
error: lifetime may not live long enough
  --> src/main.rs:39:39
   |
39 |     let closure = |x: &i32| -> &i32 { x }; // fails
   |                       -        -      ^ returning this value requires that `'1` must outlive `'2`
   |                       |        |
   |                       |        let's call the lifetime of this reference `'2`
   |                       let's call the lifetime of this reference `'1`
```

咦？竟然报错了，明明两个一模一样功能的函数，一个正常编译，一个却报错，错误原因是编译器无法推测返回的引用和传入的引用谁活得更久！

真的是非常奇怪的错误，学过上一节的读者应该都记得这样一条生命周期消除规则：**如果函数参数中只有一个引用类型，那该引用的生命周期会被自动分配给所有的返回引用**。我们当前的情况完美符合， `function` 函数的顺利编译通过，就充分说明了问题。

先给出一个结论：**这个问题，可能很难被解决，建议大家遇到后，还是老老实实用正常的函数，不要秀闭包了**。

对于函数的生命周期而言，它的消除规则之所以能生效是因为它的生命周期完全体现在签名的引用类型上，在函数体中无需任何体现：

```rust
fn fn_elision(x: &i32) -> &i32 {..}
```

因此编译器可以做各种编译优化，也很容易根据参数和返回值进行生命周期的分析，最终得出消除规则。

可是闭包，并没有函数那么简单，它的生命周期分散在参数和闭包函数体中(主要是它没有确切的返回值签名)：

```rust
let closure_slision = |x: &i32| -> &i32 { x };
```

编译器就必须深入到闭包函数体中，去分析和推测生命周期，复杂度因此极具提升：试想一下，编译器该如何从复杂的上下文中分析出参数引用的生命周期和闭包体中生命周期的关系？

由于上述原因(当然，实际情况复杂的多)，Rust 语言开发者目前其实是有意针对函数和闭包实现了两种不同的生命周期消除规则。

> **用 `Fn` 特征解决闭包生命周期**
>
> 之前我们提到了很难解决，但是并没有完全堵死(论文字的艺术- , -) 这不 @Ykong1337 同学就带了一个解决方法，为他点赞!
>
> ```rust
> fn main() {
>    let closure_slision = fun(|x: &i32| -> &i32 { x });
>    assert_eq!(*closure_slision(&45), 45);
>    // Passed !
> }
>
> fn fun<T, F: Fn(&T) -> &T>(f: F) -> F {
>    f
> }
>
> ```

### [NLL (Non-Lexical Lifetime)](https://course.rs/advance/lifetime/advance.html#nll-non-lexical-lifetime) <a href="#nll-non-lexical-lifetime" id="nll-non-lexical-lifetime"></a>

之前我们在[引用与借用](https://course.rs/basic/ownership/borrowing.html#NLL)那一章其实有讲到过这个概念，简单来说就是：**引用的生命周期正常来说应该从借用开始一直持续到作用域结束**，但是这种规则会让多引用共存的情况变得更复杂：

```rust
fn main() {
   let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    // 新编译器中，r1,r2作用域在这里结束

    let r3 = &mut s;
    println!("{}", r3);
}

```

按照上述规则，这段代码将会报错，因为 `r1` 和 `r2` 的不可变引用将持续到 `main` 函数结束，而在此范围内，我们又借用了 `r3` 的可变引用，这违反了借用的规则：要么多个不可变借用，要么一个可变借用。

好在，该规则从 1.31 版本引入 `NLL` 后，就变成了：**引用的生命周期从借用处开始，一直持续到最后一次使用的地方**。

按照最新的规则，我们再来分析一下上面的代码。`r1` 和 `r2` 不可变借用在 `println!` 后就不再使用，因此生命周期也随之结束，那么 `r3` 的借用就不再违反借用的规则，皆大欢喜。

再来看一段关于 `NLL` 的代码解释：

```rust

#![allow(unused)]
fn main() {
let mut u = 0i32;
let mut v = 1i32;
let mut w = 2i32;

// lifetime of `a` = α ∪ β ∪ γ
let mut a = &mut u;     // --+ α. lifetime of `&mut u`  --+ lexical "lifetime" of `&mut u`,`&mut u`, `&mut w` and `a`
use(a);                 //   |                            |
*a = 3; // <-----------------+                            |
...                     //                                |
a = &mut v;             // --+ β. lifetime of `&mut v`    |
use(a);                 //   |                            |
*a = 4; // <-----------------+                            |
...                     //                                |
a = &mut w;             // --+ γ. lifetime of `&mut w`    |
use(a);                 //   |                            |
*a = 5; // <-----------------+ <--------------------------+
}

```

这段代码一目了然，`a` 有三段生命周期：`α`，`β`，`γ`，每一段生命周期都随着当前值的最后一次使用而结束。

在实际项目中，`NLL` 规则可以大幅减少引用冲突的情况，极大的便利了用户，因此广受欢迎，最终该规则甚至演化成一个独立的项目，未来可能会进一步简化我们的使用，`Polonius`：

* [项目地址](https://github.com/rust-lang/polonius)
* [具体介绍](http://smallcultfollowing.com/babysteps/blog/2018/04/27/an-alias-based-formulation-of-the-borrow-checker/)

## [Reborrow 再借用](https://course.rs/advance/lifetime/advance.html#reborrow-%E5%86%8D%E5%80%9F%E7%94%A8) <a href="#reborrow-zai-jie-yong" id="reborrow-zai-jie-yong"></a>

学完 `NLL` 后，我们就有了一定的基础，可以继续学习关于借用和生命周期的一个高级内容：**再借用**。

先来看一段代码：

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn move_to(&mut self, x: i32, y: i32) {
        self.x = x;
        self.y = y;
    }
}

fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    let rr: &Point = &*r;

    println!("{:?}", rr);
    r.move_to(10, 10);
    println!("{:?}", r);
}

```

以上代码，大家可能会觉得可变引用 `r` 和不可变引用 `rr` 同时存在会报错吧？但是事实上并不会，原因在于 `rr` 是对 `r` 的再借用。

对于再借用而言，`rr` 再借用时不会破坏借用规则，但是你不能在它的生命周期内再使用原来的借用 `r`，来看看对上段代码的分析：

```rust
fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    // reborrow! 此时对`r`的再借用不会导致跟上面的借用冲突
    let rr: &Point = &*r;

    // 再借用`rr`最后一次使用发生在这里，在它的生命周期中，我们并没有使用原来的借用`r`，因此不会报错
    println!("{:?}", rr);

    // 再借用结束后，才去使用原来的借用`r`
    r.move_to(10, 10);
    println!("{:?}", r);
}

```

再来看一个例子：

```rust

#![allow(unused)]
fn main() {
use std::vec::Vec;
fn read_length(strings: &mut Vec<String>) -> usize {
   strings.len()
}
}

```

如上所示，函数体内对参数的二次借用也是典型的 `Reborrow` 场景。

那么下面让我们来做件坏事，破坏这条规则，使其报错：

```rust
fn main() {
    let mut p = Point { x: 0, y: 0 };
    let r = &mut p;
    let rr: &Point = &*r;

    r.move_to(10, 10);

    println!("{:?}", rr);

    println!("{:?}", r);
}

```

果然，破坏永远比重建简单 :) 只需要在 `rr` 再借用的生命周期内使用一次原来的借用 `r` 即可！

### [生命周期消除规则补充](https://course.rs/advance/lifetime/advance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%B6%88%E9%99%A4%E8%A7%84%E5%88%99%E8%A1%A5%E5%85%85) <a href="#sheng-ming-zhou-qi-xiao-chu-gui-ze-bu-chong" id="sheng-ming-zhou-qi-xiao-chu-gui-ze-bu-chong"></a>

在上一节中，我们介绍了三大基础生命周期消除规则，实际上，随着 Rust 的版本进化，该规则也在不断演进，这里再介绍几个常见的消除规则：

[**impl 块消除**](https://course.rs/advance/lifetime/advance.html#impl-%E5%9D%97%E6%B6%88%E9%99%A4)

```rust

#![allow(unused)]
fn main() {
impl<'a> Reader for BufReader<'a> {
    // methods go here
    // impl内部实际上没有用到'a
}
}

```

如果你以前写的`impl`块长上面这样，同时在 `impl` 内部的方法中，根本就没有用到 `'a`，那就可以写成下面的代码形式。

```rust

#![allow(unused)]
fn main() {
impl Reader for BufReader<'_> {
    // methods go here
}
}

```

`'_` 生命周期表示 `BufReader` 有一个不使用的生命周期，我们可以忽略它，无需为它创建一个名称。

歪个楼，有读者估计会发问：既然用不到 `'a`，为何还要写出来？如果你仔细回忆下上一节的内容，里面有一句专门用粗体标注的文字：**生命周期参数也是类型的一部分**，因此 `BufReader<'a>` 是一个完整的类型，在实现它的时候，你不能把 `'a` 给丢了！

[**生命周期约束消除**](https://course.rs/advance/lifetime/advance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%BA%A6%E6%9D%9F%E6%B6%88%E9%99%A4)

```rust

#![allow(unused)]
fn main() {
// Rust 2015
struct Ref<'a, T: 'a> {
    field: &'a T
}

// Rust 2018
struct Ref<'a, T> {
    field: &'a T
}
}

```

在本节的生命周期约束中，也提到过，新版本 Rust 中，上面情况中的 `T: 'a` 可以被消除掉，当然，你也可以显式的声明，但是会影响代码可读性。关于类似的场景，Rust 团队计划在未来提供更多的消除规则，但是，你懂的，计划未来就等于未知。

### [一个复杂的例子](https://course.rs/advance/lifetime/advance.html#%E4%B8%80%E4%B8%AA%E5%A4%8D%E6%9D%82%E7%9A%84%E4%BE%8B%E5%AD%90) <a href="#yi-ge-fu-za-de-li-zi" id="yi-ge-fu-za-de-li-zi"></a>

下面是一个关于生命周期声明过大的例子，会较为复杂，希望大家能细细阅读，它能帮你对生命周期的理解更加深入。

```rust
struct Interface<'a> {
    manager: &'a mut Manager<'a>
}

impl<'a> Interface<'a> {
    pub fn noop(self) {
        println!("interface consumed");
    }
}

struct Manager<'a> {
    text: &'a str
}

struct List<'a> {
    manager: Manager<'a>,
}

impl<'a> List<'a> {
    pub fn get_interface(&'a mut self) -> Interface {
        Interface {
            manager: &mut self.manager
        }
    }
}

fn main() {
    let mut list = List {
        manager: Manager {
            text: "hello"
        }
    };

    list.get_interface().noop();

    println!("Interface should be dropped here and the borrow released");

    // 下面的调用会失败，因为同时有不可变/可变借用
    // 但是Interface在之前调用完成后就应该被释放了
    use_list(&list);
}

fn use_list(list: &List) {
    println!("{}", list.manager.text);
}

```

运行后报错：

```console
error[E0502]: cannot borrow `list` as immutable because it is also borrowed as mutable // `list`无法被借用，因为已经被可变借用
  --> src/main.rs:40:14
   |
34 |     list.get_interface().noop();
   |     ---- mutable borrow occurs here // 可变借用发生在这里
...
40 |     use_list(&list);
   |              ^^^^^
   |              |
   |              immutable borrow occurs here // 新的不可变借用发生在这
   |              mutable borrow later used here // 可变借用在这里结束
```

这段代码看上去并不复杂，实际上难度挺高的，首先在直觉上，`list.get_interface()` 借用的可变引用，按理来说应该在这行代码结束后，就归还了，但是为什么还能持续到 `use_list(&list)` 后面呢？

这是因为我们在 `get_interface` 方法中声明的 `lifetime` 有问题，该方法的参数的生命周期是 `'a`，而 `List` 的生命周期也是 `'a`，说明该方法至少活得跟 `List` 一样久，再回到 `main` 函数中，`list` 可以活到 `main` 函数的结束，因此 `list.get_interface()` 借用的可变引用也会活到 `main` 函数的结束，在此期间，自然无法再进行借用了。

要解决这个问题，我们需要为 `get_interface` 方法的参数给予一个不同于 `List<'a>` 的生命周期 `'b`，最终代码如下：

```rust
struct Interface<'b, 'a: 'b> {
    manager: &'b mut Manager<'a>
}

impl<'b, 'a: 'b> Interface<'b, 'a> {
    pub fn noop(self) {
        println!("interface consumed");
    }
}

struct Manager<'a> {
    text: &'a str
}

struct List<'a> {
    manager: Manager<'a>,
}

impl<'a> List<'a> {
    pub fn get_interface<'b>(&'b mut self) -> Interface<'b, 'a>
    where 'a: 'b {
        Interface {
            manager: &mut self.manager
        }
    }
}

fn main() {

    let mut list = List {
        manager: Manager {
            text: "hello"
        }
    };

    list.get_interface().noop();

    println!("Interface should be dropped here and the borrow released");

    // 下面的调用可以通过，因为Interface的生命周期不需要跟list一样长
    use_list(&list);
}

fn use_list(list: &List) {
    println!("{}", list.manager.text);
}

```

至此，生命周期终于完结，两章超级长的内容，可以满足几乎所有对生命周期的学习目标。学完生命周期，意味着你正式入门了 Rust，只要再掌握几个常用概念，就可以上手写项目了。

## [&'static 和 T: 'static](https://course.rs/advance/lifetime/static.html#static-%E5%92%8C-t-static) <a href="#static-he-tstatic" id="static-he-tstatic"></a>

Rust 的难点之一就在于它有不少容易混淆的概念，例如 `&str` 、`str` 与 `String`， 再比如本文标题那两位。不过与字符串也有不同，这两位对于普通用户来说往往是无需进行区分的，但是当大家想要深入学习或使用 Rust 时，它们就会成为成功路上的拦路虎了。

与生命周期的其它章节不同，本文短小精悍，阅读过程可谓相当轻松愉快，话不多说，let's go。

`'static` 在 Rust 中是相当常见的，例如字符串字面值就具有 `'static` 生命周期:

```rust
fn main() {
  let mark_twain: &str = "Samuel Clemens";
  print_author(mark_twain);
}
fn print_author(author: &'static str) {
  println!("{}", author);
}

```

除此之外，特征对象的生命周期也是 `'static`，例如[这里](https://course.rs/compiler/fight-with-compiler/lifetime/closure-with-static.html#%E7%89%B9%E5%BE%81%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)所提到的。

除了 `&'static` 的用法外，我们在另外一种场景中也可以见到 `'static` 的使用:

```rust
use std::fmt::Display;
fn main() {
    let mark_twain = "Samuel Clemens";
    print(&mark_twain);
}

fn print<T: Display + 'static>(message: &T) {
    println!("{}", message);
}

```

在这里，很明显 `'static` 是作为生命周期约束来使用了。 **那么问题来了， `&'static` 和 `T: 'static` 的用法到底有何区别？**

### [`&'static`](https://course.rs/advance/lifetime/static.html#static) <a href="#static" id="static"></a>

`&'static` 对于生命周期有着非常强的要求：一个引用必须要活得跟剩下的程序一样久，才能被标注为 `&'static`。

对于字符串字面量来说，它直接被打包到二进制文件中，永远不会被 `drop`，因此它能跟程序活得一样久，自然它的生命周期是 `'static`。

但是，**`&'static` 生命周期针对的仅仅是引用，而不是持有该引用的变量，对于变量来说，还是要遵循相应的作用域规则** :

```rust
use std::{slice::from_raw_parts, str::from_utf8_unchecked};

fn get_memory_location() -> (usize, usize) {
  // “Hello World” 是字符串字面量，因此它的生命周期是 `'static`.
  // 但持有它的变量 `string` 的生命周期就不一样了，它完全取决于变量作用域，对于该例子来说，也就是当前的函数范围
  let string = "Hello World!";
  let pointer = string.as_ptr() as usize;
  let length = string.len();
  (pointer, length)
  // `string` 在这里被 drop 释放
  // 虽然变量被释放，无法再被访问，但是数据依然还会继续存活
}

fn get_str_at_location(pointer: usize, length: usize) -> &'static str {
  // 使用裸指针需要 `unsafe{}` 语句块
  unsafe { from_utf8_unchecked(from_raw_parts(pointer as *const u8, length)) }
}

fn main() {
  let (pointer, length) = get_memory_location();
  let message = get_str_at_location(pointer, length);
  println!(
    "The {} bytes at 0x{:X} stored: {}",
    length, pointer, message
  );
  // 如果大家想知道为何处理裸指针需要 `unsafe`，可以试着反注释以下代码
  // let message = get_str_at_location(1000, 10);
}

```

上面代码有两点值得注意：

* `&'static` 的引用确实可以和程序活得一样久，因为我们通过 `get_str_at_location` 函数直接取到了对应的字符串
* 持有 `&'static` 引用的变量，它的生命周期受到作用域的限制，大家务必不要搞混了

### [`T: 'static`](https://course.rs/advance/lifetime/static.html#t-static) <a href="#t-static" id="t-static"></a>

相比起来，这种形式的约束就有些复杂了。

首先，在以下两种情况下，`T: 'static` 与 `&'static` 有相同的约束：`T` 必须活得和程序一样久。

```rust
use std::fmt::Debug;

fn print_it<T: Debug + 'static>( input: T) {
    println!( "'static value passed in is: {:?}", input );
}

fn print_it1( input: impl Debug + 'static ) {
    println!( "'static value passed in is: {:?}", input );
}



fn main() {
    let i = 5;

    print_it(&i);
    print_it1(&i);
}

```

以上代码会报错，原因很简单: `&i` 的生命周期无法满足 `'static` 的约束，如果大家将 `i` 修改为常量，那自然一切 OK。

见证奇迹的时候，请不要眨眼，现在我们来稍微修改下 `print_it` 函数:

```rust
use std::fmt::Debug;

fn print_it<T: Debug + 'static>( input: &T) {
    println!( "'static value passed in is: {:?}", input );
}

fn main() {
    let i = 5;

    print_it(&i);
}

```

这段代码竟然不报错了！原因在于我们约束的是 `T`，但是使用的却是它的引用 `&T`，换而言之，我们根本没有直接使用 `T`，因此编译器就没有去检查 `T` 的生命周期约束！它只要确保 `&T` 的生命周期符合规则即可，在上面代码中，它自然是符合的。

再来看一个例子:

```rust
use std::fmt::Display;

fn main() {
  let r1;
  let r2;
  {
    static STATIC_EXAMPLE: i32 = 42;
    r1 = &STATIC_EXAMPLE;
    let x = "&'static str";
    r2 = x;
    // r1 和 r2 持有的数据都是 'static 的，因此在花括号结束后，并不会被释放
  }

  println!("&'static i32: {}", r1); // -> 42
  println!("&'static str: {}", r2); // -> &'static str

  let r3: &str;

  {
    let s1 = "String".to_string();

    // s1 虽然没有 'static 生命周期，但是它依然可以满足 T: 'static 的约束
    // 充分说明这个约束是多么的弱。。
    static_bound(&s1);

    // s1 是 String 类型，没有 'static 的生命周期，因此下面代码会报错
    r3 = &s1;

    // s1 在这里被 drop
  }
  println!("{}", r3);
}

fn static_bound<T: Display + 'static>(t: &T) {
  println!("{}", t);
}

```

### [static 到底针对谁？](https://course.rs/advance/lifetime/static.html#static-%E5%88%B0%E5%BA%95%E9%92%88%E5%AF%B9%E8%B0%81) <a href="#static-dao-di-zhen-dui-shui" id="static-dao-di-zhen-dui-shui"></a>

大家有没有想过，到底是 `&'static` 这个引用还是该引用指向的数据活得跟程序一样久呢？

**答案是引用指向的数据**，而引用本身是要遵循其作用域范围的，我们来简单验证下：

```rust
fn main() {
    {
        let static_string = "I'm in read-only memory";
        println!("static_string: {}", static_string);

        // 当 `static_string` 超出作用域时，该引用不能再被使用，但是数据依然会存在于 binary 所占用的内存中
    }

    println!("static_string reference remains alive: {}", static_string);
}

```

以上代码不出所料会报错，原因在于虽然字符串字面量 "I'm in read-only memory" 的生命周期是 `'static`，但是持有它的引用并不是，它的作用域在内部花括号 `}` 处就结束了。

### [课后练习](https://course.rs/advance/lifetime/static.html#%E8%AF%BE%E5%90%8E%E7%BB%83%E4%B9%A0) <a href="#ke-hou-lian-xi" id="ke-hou-lian-xi"></a>

> [Rust By Practice](https://zh.practice.rs/lifetime/static.html)，支持代码在线编辑和运行，并提供详细的习题解答。（本节暂无习题解答）

### [总结](https://course.rs/advance/lifetime/static.html#%E6%80%BB%E7%BB%93) <a href="#zong-jie" id="zong-jie"></a>

总之， `&'static` 和 `T: 'static` 大体上相似，相比起来，后者的使用形式会更加复杂一些。

至此，相信大家对于 `'static` 和 `T: 'static` 也有了清晰的理解，那么我们应该如何使用它们呢？

作为经验之谈，可以这么来:

* 如果你需要添加 `&'static` 来让代码工作，那很可能是设计上出问题了
* 如果你希望满足和取悦编译器，那就使用 `T: 'static`，很多时候它都能解决问题

> 一个小知识，在 Rust 标准库中，有 48 处用到了 &'static ，112 处用到了 `T: 'static` ，看来取悦编译器不仅仅是菜鸟需要的，高手也经常用到 :)





