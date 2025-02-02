# 函数

## [函数](https://rustwiki.org/zh-CN/rust-by-example/fn.html#%E5%87%BD%E6%95%B0) <a href="#han-shu" id="han-shu"></a>

函数（function）使用 `fn` 关键字来声明。函数的参数需要标注类型，就和变量一样，如果函数返回一个值，返回类型必须在箭头 `->` 之后指定。

函数最后的表达式将作为返回值。也可以在函数内使用 `return` 语句来提前返一个值，甚至可以在循环或 `if` 内部使用。

让我们使用函数来重写 FizzBuzz 程序吧！

```
// 和 C/C++ 不一样，Rust 的函数定义位置是没有限制的
fn main() {
    // 我们可以在这里使用函数，后面再定义它
    fizzbuzz_to(100);
}

// 一个返回布尔值的函数
fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
    // 边界情况，提前返回
    if rhs == 0 {
        return false;
    }

    // 这是一个表达式，可以不用 `return` 关键字
    lhs % rhs == 0
}

// 一个 “不” 返回值的函数。实际上会返回一个单元类型 `()`。
fn fizzbuzz(n: u32) -> () {
    if is_divisible_by(n, 15) {
        println!("fizzbuzz");
    } else if is_divisible_by(n, 3) {
        println!("fizz");
    } else if is_divisible_by(n, 5) {
        println!("buzz");
    } else {
        println!("{}", n);
    }
}

// 当函数返回 `()` 时，函数签名可以省略返回类型
fn fizzbuzz_to(n: u32) {
    for n in 1..=n {
        fizzbuzz(n);
    }
}

```

## [方法](https://rustwiki.org/zh-CN/rust-by-example/fn/methods.html#%E6%96%B9%E6%B3%95)

方法（method）是依附于对象的函数。这些方法通过关键字 `self` 来访问对象中的数据和其他。方法在 `impl` 代码块中定义。

```
struct Point {
    x: f64,
    y: f64,
}

// 实现的代码块，`Point` 的所有方法都在这里给出
impl Point {
    // 这是一个静态方法（static method）
    // 静态方法不需要被实例调用
    // 这类方法一般用作构造器（constructor）
    fn origin() -> Point {
        Point { x: 0.0, y: 0.0 }
    }

    // 另外一个静态方法，需要两个参数：
    fn new(x: f64, y: f64) -> Point {
        Point { x: x, y: y }
    }
}

struct Rectangle {
    p1: Point,
    p2: Point,
}

impl Rectangle {
    // 这是一个实例方法（instance method）
    // `&self` 是 `self: &Self` 的语法糖（sugar），其中 `Self` 是方法调用者的
    // 类型。在这个例子中 `Self` = `Rectangle`
    fn area(&self) -> f64 {
        // `self` 通过点运算符来访问结构体字段
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        // `abs` 是一个 `f64` 类型的方法，返回调用者的绝对值
        ((x1 - x2) * (y1 - y2)).abs()
    }

    fn perimeter(&self) -> f64 {
        let Point { x: x1, y: y1 } = self.p1;
        let Point { x: x2, y: y2 } = self.p2;

        2.0 * ((x1 - x2).abs() + (y1 - y2).abs())
    }

    // 这个方法要求调用者是可变的
    // `&mut self` 为 `self: &mut Self` 的语法糖
    fn translate(&mut self, x: f64, y: f64) {
        self.p1.x += x;
        self.p2.x += x;

        self.p1.y += y;
        self.p2.y += y;
    }
}

// `Pair` 拥有资源：两个堆分配的整型
struct Pair(Box<i32>, Box<i32>);

impl Pair {
    // 这个方法会 “消耗” 调用者的资源
    // `self` 为 `self: Self` 的语法糖
    fn destroy(self) {
        // 解构 `self`
        let Pair(first, second) = self;

        println!("Destroying Pair({}, {})", first, second);

        // `first` 和 `second` 离开作用域后释放
    }
}

fn main() {
    let rectangle = Rectangle {
        // 静态方法使用双冒号调用
        p1: Point::origin(),
        p2: Point::new(3.0, 4.0),
    };

    // 实例方法通过点运算符来调用
    // 注意第一个参数 `&self` 是隐式传递的，亦即：
    // `rectangle.perimeter()` === `Rectangle::perimeter(&rectangle)`
    println!("Rectangle perimeter: {}", rectangle.perimeter());
    println!("Rectangle area: {}", rectangle.area());

    let mut square = Rectangle {
        p1: Point::origin(),
        p2: Point::new(1.0, 1.0),
    };

    // 报错！ `rectangle` 是不可变的，但这方法需要一个可变对象
    //rectangle.translate(1.0, 0.0);
    // 试一试 ^ 去掉此行的注释

    // 正常运行！可变对象可以调用可变方法
    square.translate(1.0, 1.0);

    let pair = Pair(Box::new(1), Box::new(2));

    pair.destroy();

    // 报错！前面的 `destroy` 调用 “消耗了” `pair`
    //pair.destroy();
    // 试一试 ^ 将此行注释去掉
}

```

## [闭包](https://rustwiki.org/zh-CN/rust-by-example/fn/closures.html#%E9%97%AD%E5%8C%85) <a href="#bi-bao" id="bi-bao"></a>

Rust 中的闭包（closure），也叫做 lambda 表达式或者 lambda，是一类能够捕获周围作用域中变量的函数。例如，一个可以捕获 x 变量的闭包如下：

```Rust
|val| val + x
```

它们的语法和能力使它们在临时（on the fly）使用时相当方便。调用一个闭包和调用一个函数完全相同，不过调用闭包时，输入和返回类型两者都**可以**自动推导，而输入变量名**必须**指明。

其他的特点包括：

* 声明时使用 `||` 替代 `()` 将输入参数括起来。
* 函数体定界符（`{}`）对于单个表达式是可选的，其他情况必须加上。
* 有能力捕获外部环境的变量。

```
fn main() {
    // 通过闭包和函数分别实现自增。
    // 译注：下面这行是使用函数的实现
    fn  function            (i: i32) -> i32 { i + 1 }

    // 闭包是匿名的，这里我们将它们绑定到引用。
    // 类型标注和函数的一样，不过类型标注和使用 `{}` 来围住函数体都是可选的。
    // 这些匿名函数（nameless function）被赋值给合适地命名的变量。
    let closure_annotated = |i: i32| -> i32 { i + 1 };
    let closure_inferred  = |i     |          i + 1  ;

    // 译注：将闭包绑定到引用的说法可能不准。
    // 据[语言参考](https://doc.rust-lang.org/beta/reference/types.html#closure-types)
    // 闭包表达式产生的类型就是 “闭包类型”，不属于引用类型，而且确实无法对上面两个
    // `closure_xxx` 变量解引用。

    let i = 1;
    // 调用函数和闭包。
    println!("function: {}", function(i));
    println!("closure_annotated: {}", closure_annotated(i));
    println!("closure_inferred: {}", closure_inferred(i));

    // 没有参数的闭包，返回一个 `i32` 类型。
    // 返回类型是自动推导的。
    let one = || 1;
    println!("closure returning one: {}", one());
}

```

### &#x20;[捕获](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/capture.html#%E6%8D%95%E8%8E%B7)

闭包本质上很灵活，能做功能要求的事情，使闭包在没有类型标注的情况下运行。这使得捕获（capture）能够灵活地适应用例，既可移动（move），又可借用（borrow）。闭包可以通过以下方式捕获变量：

* 通过引用：`&T`
* 通过可变引用：`&mut T`
* 通过值：`T`

闭包优先通过引用来捕获变量，并且仅在需要时使用其他方式。

<pre><code>fn main() {
    use std::mem;

    let color = String::from("green");

    // 这个闭包打印 `color`。它会立即借用（通过引用，`&#x26;`）`color` 并将该借用和
    // 闭包本身存储到 `print` 变量中。`color` 会一直保持被借用状态直到
    // `print` 离开作用域。
    //
    // `println!` 只需传引用就能使用，而这个闭包捕获的也是变量的引用，因此无需
    // 进一步处理就可以使用 `println!`。
<strong>    let print = || println!("`color`: {}", color);
</strong>
    // 使用借用来调用闭包 `color`。
    print();

    // `color` 可再次被不可变借用，因为闭包只持有一个指向 `color` 的不可变引用。
    let _reborrow = &#x26;color;
    print();

    // 在最后使用 `print` 之后，移动或重新借用都是允许的。
    let _color_moved = color;

    let mut count = 0;
    // 这个闭包使 `count` 值增加。要做到这点，它需要得到 `&#x26;mut count` 或者
    // `count` 本身，但 `&#x26;mut count` 的要求没那么严格，所以我们采取这种方式。
    // 该闭包立即借用 `count`。
    //
    // `inc` 前面需要加上 `mut`，因为闭包里存储着一个 `&#x26;mut` 变量。调用闭包时，
    // 该变量的变化就意味着闭包内部发生了变化。因此闭包需要是可变的。
    let mut inc = || {
        count += 1;
        println!("`count`: {}", count);
    };

    // 使用可变借用调用闭包
    inc();

    // 因为之后调用闭包，所以仍然可变借用 `count`
    // 试图重新借用将导致错误
    // let _reborrow = &#x26;count;
    // ^ 试一试：将此行注释去掉。
    inc();

    // 闭包不再借用 `&#x26;mut count`，因此可以正确地重新借用
    let _count_reborrowed = &#x26;mut count;

    // 不可复制类型（non-copy type）。
    let movable = Box::new(3);

    // `mem::drop` 要求 `T` 类型本身，所以闭包将会捕获变量的值。这种情况下，
    // 可复制类型将会复制给闭包，从而原始值不受影响。不可复制类型必须移动
    // （move）到闭包中，因而 `movable` 变量在这里立即移动到了闭包中。
    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(movable);
    };

    // `consume` 消耗了该变量，所以该闭包只能调用一次。
    consume();
    //consume();
    // ^ 试一试：将此行注释去掉。
}

</code></pre>

在竖线 `|` 之前使用 `move` 会强制闭包取得被捕获变量的所有权：

```
fn main() {
    // `Vec` 在语义上是不可复制的。
    let haystack = vec![1, 2, 3];

    let contains = move |needle| haystack.contains(needle);

    println!("{}", contains(&1));
    println!("{}", contains(&4));

    //println!("There're {} elements in vec", haystack.len());
    // ^ 取消上面一行的注释将导致编译时错误，因为借用检查不允许在变量被移动走
    // 之后继续使用它。

    // 在闭包的签名中删除 `move` 会导致闭包以不可变方式借用 `haystack`，因此之后
    // `haystack` 仍然可用，取消上面的注释也不会导致错误。
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/capture.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

### [`Box`](https://rustwiki.org/zh-CN/rust-by-example/std/box.html) 和 [`std::mem::drop`](https://rustwiki.org/zh-CN/std/mem/fn.drop.html)[ ](https://rustwiki.org/zh-CN/rust-by-example/flow_control/while_let.html)[作为输入参数](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/input_parameters.html#%E4%BD%9C%E4%B8%BA%E8%BE%93%E5%85%A5%E5%8F%82%E6%95%B0)

虽然 Rust 无需类型说明就能在大多数时候完成变量捕获，但在编写函数时，这种模糊写法是不允许的。当以闭包作为输入参数时，必须指出闭包的完整类型，它是通过使用以下 `trait` 中的一种来指定的。其受限制程度按以下顺序递减：

* `Fn`：表示捕获方式为通过引用（`&T`）的闭包
* `FnMut`：表示捕获方式为通过可变引用（`&mut T`）的闭包
* `FnOnce`：表示捕获方式为通过值（`T`）的闭包

> 译注：顺序之所以是这样，是因为 `&T` 只是获取了不可变的引用，`&mut T` 则可以改变变量，`T` 则是拿到了变量的所有权而非借用。

对闭包所要捕获的每个变量，编译器都将以限制最少的方式来捕获。

> 译注：这句可能说得不对，事实上是在满足使用需求的前提下尽量以限制最多的方式捕获。

例如用一个类型说明为 `FnOnce` 的闭包作为参数。这说明闭包可能采取 `&T`，`&mut T` 或 `T` 中的一种捕获方式，但编译器最终是根据所捕获变量在闭包里的使用情况决定捕获方式。

这是因为如果能以移动的方式捕获变量，则闭包也有能力使用其他方式借用变量。注意反过来就不再成立：如果参数的类型说明是 `Fn`，那么不允许该闭包通过 `&mut T` 或 `T` 捕获变量。

在下面的例子中，试着分别用一用 `Fn`、`FnMut` 和 `FnOnce`，看看会发生什么：

```
// 该函数将闭包作为参数并调用它。
fn apply<F>(f: F) where
    // 闭包没有输入值和返回值。
    F: FnOnce() {
    // ^ 试一试：将 `FnOnce` 换成 `Fn` 或 `FnMut`。

    f();
}

// 输入闭包，返回一个 `i32` 整型的函数。
fn apply_to_3<F>(f: F) -> i32 where
    // 闭包处理一个 `i32` 整型并返回一个 `i32` 整型。
    F: Fn(i32) -> i32 {

    f(3)
}

fn main() {
    use std::mem;

    let greeting = "hello";
    // 不可复制的类型。
    // `to_owned` 从借用的数据创建有所有权的数据。
    let mut farewell = "goodbye".to_owned();

    // 捕获 2 个变量：通过引用捕获 `greeting`，通过值捕获 `farewell`。
    let diary = || {
        // `greeting` 通过引用捕获，故需要闭包是 `Fn`。
        println!("I said {}.", greeting);

        // 下文改变了 `farewell` ，因而要求闭包通过可变引用来捕获它。
        // 现在需要 `FnMut`。
        farewell.push_str("!!!");
        println!("Then I screamed {}.", farewell);
        println!("Now I can sleep. zzzzz");

        // 手动调用 drop 又要求闭包通过值获取 `farewell`。
        // 现在需要 `FnOnce`。
        mem::drop(farewell);
    };

    // 以闭包作为参数，调用函数 `apply`。
    apply(diary);

    // 闭包 `double` 满足 `apply_to_3` 的 trait 约束。
    let double = |x| 2 * x;

    println!("3 doubled: {}", apply_to_3(double));
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/input_parameters.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`std::mem::drop`](https://rustwiki.org/zh-CN/std/mem/fn.drop.html), [`Fn`](https://rustwiki.org/zh-CN/std/ops/trait.Fn.html), [`FnMut`](https://rustwiki.org/zh-CN/std/ops/trait.FnMut.html), 和 [`FnOnce`](https://rustwiki.org/zh-CN/std/ops/trait.FnOnce.html)

### [类型匿名](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/anonymity.html#%E7%B1%BB%E5%9E%8B%E5%8C%BF%E5%90%8D) <a href="#lei-xing-ni-ming" id="lei-xing-ni-ming"></a>

闭包从周围的作用域中捕获变量是简单明了的。这样会有某些后果吗？确实有。观察一下使用闭包作为函数参数，这要求闭包是[泛型](https://rustwiki.org/zh-CN/rust-by-example/generics.html)的，闭包定义的方式决定了这是必要的。

```rust

#![allow(unused)]
fn main() {
// `F` 必须是泛型的。
fn apply<F>(f: F) where
    F: FnOnce() {
    f();
}
}

```

当闭包被定义，编译器会隐式地创建一个匿名类型的结构体，用以储存闭包捕获的变量，同时为这个未知类型的结构体实现函数功能，通过 `Fn`、`FnMut` 或 `FnOnce` 三种 `trait` 中的一种。

若使用闭包作为函数参数，由于这个结构体的类型未知，任何的用法都要求是泛型的。然而，使用未限定类型的参数 `<T>` 过于不明确，并且是不允许的。事实上，指明为该结构体实现的是 `Fn`、`FnMut`、或 `FnOnce` 中的哪种 `trait`，对于约束该结构体的类型而言就已经足够了。

```
// `F` 必须为一个没有输入参数和返回值的闭包实现 `Fn`，这和对 `print` 的
// 要求恰好一样。
fn apply<F>(f: F) where
    F: Fn() {
    f();
}

fn main() {
    let x = 7;

    // 捕获 `x` 到匿名类型中，并为它实现 `Fn`。
    // 将闭包存储到 `print` 中。
    let print = || println!("{}", x);

    apply(print);
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/anonymity.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[详尽分析](https://huonw.github.io/blog/2015/05/finding-closure-in-rust/), [`Fn`](https://rustwiki.org/zh-CN/std/ops/trait.Fn.html), [`FnMut`](https://rustwiki.org/zh-CN/std/ops/trait.FnMut.html), 和 [`FnOnce`](https://rustwiki.org/zh-CN/std/ops/trait.FnOnce.html)

### [输入函数](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/input_functions.html#%E8%BE%93%E5%85%A5%E5%87%BD%E6%95%B0) <a href="#shu-ru-han-shu" id="shu-ru-han-shu"></a>

既然闭包可以作为参数，你很可能想知道函数是否也可以呢。确实可以！如果你声明一个接受闭包作为参数的函数，那么任何满足该闭包的 trait 约束的函数都可以作为其参数。

```
// 定义一个函数，可以接受一个由 `Fn` 限定的泛型 `F` 参数并调用它。
fn call_me<F: Fn()>(f: F) {
    f()
}

// 定义一个满足 `Fn` 约束的封装函数（wrapper function）。
fn function() {
    println!("I'm a function!");
}

fn main() {
    // 定义一个满足 `Fn` 约束的闭包。
    let closure = || println!("I'm a closure!");
    
    call_me(closure);
    call_me(function);
}

```

多说一句，`Fn`、`FnMut` 和 `FnOnce` 这些 `trait` 明确了闭包如何从周围的作用域中捕获变量。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/input_functions.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`Fn`](https://rustwiki.org/zh-CN/std/ops/trait.Fn.html), [`FnMut`](https://rustwiki.org/zh-CN/std/ops/trait.FnMut.html), 和 [`FnOnce`](https://rustwiki.org/zh-CN/std/ops/trait.FnOnce.html)

### &#x20;[作为输出参数](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/output_parameters.html#%E4%BD%9C%E4%B8%BA%E8%BE%93%E5%87%BA%E5%8F%82%E6%95%B0)

闭包作为输入参数是可能的，所以返回闭包作为输出参数（output parameter）也应该是可能的。然而返回闭包类型会有问题，因为目前 Rust 只支持返回具体（非泛型）的类型。按照定义，匿名的闭包的类型是未知的，所以只有使用`impl Trait`才能返回一个闭包。

返回闭包的有效特征是：

* `Fn`
* `FnMut`
* `FnOnce`

除此之外，还必须使用 `move` 关键字，它表明所有的捕获都是通过值进行的。这是必须的，因为在函数退出时，任何通过引用的捕获都被丢弃，在闭包中留下无效的引用。

```
fn create_fn() -> impl Fn() {
    let text = "Fn".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnmut() -> impl FnMut() {
    let text = "FnMut".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "FnOnce".to_owned();
    
    move || println!("This is a: {}", text)
}

fn main() {
    let fn_plain = create_fn();
    let mut fn_mut = create_fnmut();
    let fn_once = create_fnonce();

    fn_plain();
    fn_mut();
    fn_once();
}


```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/output_parameters.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`Fn`](https://rustwiki.org/zh-CN/std/ops/trait.Fn.html), [`FnMut`](https://rustwiki.org/zh-CN/std/ops/trait.FnMut.html), [泛型](https://rustwiki.org/zh-CN/rust-by-example/generics.html) 和 [impl Trait](https://rustwiki.org/zh-CN/rust-by-example/trait/impl_trait.html).

#### [Iterator::any](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/closure_examples/iter_any.html#iteratorany) <a href="#std-zhong-de-li-zi" id="std-zhong-de-li-zi"></a>

`Iterator::any` 是一个函数，若传给它一个迭代器（iterator），当其中任一元素满足谓词（predicate）时它将返回 `true`，否则返回 `false`（译注：谓词是闭包规定的， `true`/`false` 是闭包作用在元素上的返回值）。它的签名如下：

```rust
pub trait Iterator {
    // 被迭代的类型。
    type Item;

    // `any` 接受 `&mut self` 参数（译注：回想一下，这是 `self: &mut Self` 的简写）
    // 表明函数的调用者可以被借用和修改，但不会被消耗。
    fn any<F>(&mut self, f: F) -> bool where
        // `FnMut` 表示被捕获的变量最多只能被修改，而不能被消耗。
        // `Self::Item` 表明变量是通过值传递给闭包（译注：是迭代器对应的元素的类型）
        F: FnMut(Self::Item) -> bool {}
}

```

```
fn main() {
    let vec1 = vec![1, 2, 3];
    let vec2 = vec![4, 5, 6];

    // 对 vec 的 `iter()` 举出 `&i32`。（通过用 `&x` 匹配）把它解构成 `i32`。
    // 译注：注意 `any` 方法会自动地把 `vec.iter()` 举出的迭代器的元素一个个地
    // 传给闭包。因此闭包接收到的参数是 `&i32` 类型的。
    println!("2 in vec1: {}", vec1.iter()     .any(|&x| x == 2));
    // 对 vec 的 `into_iter()` 举出 `i32` 类型。无需解构。
    println!("2 in vec2: {}", vec2.into_iter().any(| x| x == 2));

    let array1 = [1, 2, 3];
    let array2 = [4, 5, 6];

    // 对数组的 `iter()` 举出 `&i32`。
    println!("2 in array1: {}", array1.iter()     .any(|&x| x == 2));
    // 对数组的 `into_iter()` 举出 `i32`。
    println!("2 in array2: {}", array2.into_iter().any(|x| x == 2));
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/closure_examples/iter_any.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`std::iter::Iterator::any`](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#method.any)[`std` 中的例子](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/closure_examples.html#std-%E4%B8%AD%E7%9A%84%E4%BE%8B%E5%AD%90)

本小节列出几个标准库中使用闭包的例子。

#### [Iterator::find](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/closure_examples/iter_find.html#iteratorfind) <a href="#iteratorfind" id="iteratorfind"></a>

`Iterator::find` 是一个函数，在传给它一个迭代器时，将用 `Option` 类型返回第一个满足谓词的元素。它的签名如下：

```rust
pub trait Iterator {
    // 被迭代的类型。
    type Item;

    // `find` 接受 `&mut self` 参数，表明函数的调用者可以被借用和修改，
    // 但不会被消耗。
    fn find<P>(&mut self, predicate: P) -> Option<Self::Item> where
        // `FnMut` 表示被捕获的变量最多只能被修改，而不能被消耗。
        // `&Self::Item` 指明了被捕获变量的类型（译注：是对迭代器元素的引用类型）
        P: FnMut(&Self::Item) -> bool {}
}
```

```
fn main() {
    let vec1 = vec![1, 2, 3];
    let vec2 = vec![4, 5, 6];

    // 对 vec1 的 `iter()` 举出 `&i32` 类型。
    let mut iter = vec1.iter();
    // 对 vec2 的 `into_iter()` 举出 `i32` 类型。
    let mut into_iter = vec2.into_iter();

    // 对迭代器举出的元素的引用是 `&&i32` 类型。解构成 `i32` 类型。
    // 译注：注意 `find` 方法会把迭代器元素的引用传给闭包。迭代器元素自身
    // 是 `&i32` 类型，所以传给闭包的是 `&&i32` 类型。
    println!("Find 2 in vec1: {:?}", iter     .find(|&&x| x == 2));
    // 对迭代器举出的元素的引用是 `&i32` 类型。解构成 `i32` 类型。
    println!("Find 2 in vec2: {:?}", into_iter.find(| &x| x == 2));

    let array1 = [1, 2, 3];
    let array2 = [4, 5, 6];

    // 对数组的 `iter()` 举出 `&i32`。
    println!("Find 2 in array1: {:?}", array1.iter()     .find(|&&x| x == 2));
    // 对数组的 `into_iter()` 通常举出 `&i32``。
    println!("Find 2 in array2: {:?}", array2.into_iter().find(|&x| x == 2));
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/fn/closures/closure_examples/iter_find.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`std::iter::Iterator::find`](https://rustwiki.org/zh-CN/std/iter/trait.Iterator.html#method.find)

## [高阶函数](https://rustwiki.org/zh-CN/rust-by-example/fn/hof.html#%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0) <a href="#gao-jie-han-shu" id="gao-jie-han-shu"></a>

Rust 提供了高阶函数（Higher Order Function, HOF），指那些输入一个或多个函数，并且/或者产生一个更有用的函数的函数。HOF 和惰性迭代器（lazy iterator）给 Rust 带来了函数式（functional）编程的风格。

```
fn is_odd(n: u32) -> bool {
    n % 2 == 1
}

fn main() {
    println!("Find the sum of all the squared odd numbers under 1000");
    let upper = 1000;

    // 命令式（imperative）的写法
    // 声明累加器变量
    let mut acc = 0;
    // 迭代：0，1, 2, ... 到无穷大
    for n in 0.. {
        // 数字的平方
        let n_squared = n * n;

        if n_squared >= upper {
            // 若大于上限则退出循环
            break;
        } else if is_odd(n_squared) {
            // 如果是奇数就计数
            acc += n_squared;
        }
    }
    println!("imperative style: {}", acc);

    // 函数式的写法
    let sum_of_squared_odd_numbers: u32 =
        (0..).map(|n| n * n)             // 所有自然数取平方
             .take_while(|&n| n < upper) // 取小于上限的
             .filter(|&n| is_odd(n))     // 取奇数
             .fold(0, |sum, i| sum + i); // 最后加起来
    println!("functional style: {}", sum_of_squared_odd_numbers);
}

```

[Option](https://rustwiki.org/zh-CN/core/option/enum.Option.html) 和 [迭代器](https://rustwiki.org/zh-CN/core/iter/trait.Iterator.html) 都实现了不少高阶函数。

## [发散函数](https://rustwiki.org/zh-CN/rust-by-example/fn/diverging.html#%E5%8F%91%E6%95%A3%E5%87%BD%E6%95%B0)

发散函数（diverging function）绝不会返回。 它们使用 `!` 标记，这是一个空类型。

```rust

#![allow(unused)]
fn main() {
fn foo() -> ! {
    panic!("This call never returns.");
}
}

```

和所有其他类型相反，这个类型无法实例化，因为此类型可能具有的所有可能值的集合为空。 注意，它与 `()` 类型不同，后者只有一个可能的值。

如下面例子，虽然返回值中没有信息，但此函数会照常返回。

```rust
fn some_fn() {
    ()
}

fn main() {
    let a: () = some_fn();
    println!("This function returns and you can see this line.")
}

```

下面这个函数相反，这个函数永远不会将控制内容返回给调用者。

```rust
#![feature(never_type)]

fn main() {
    let x: ! = panic!("This call never returns.");
    println!("You will never see this line!");
}
```

虽然这看起来像是一个抽象的概念，但实际上这非常有用且方便。这种类型的主要优点是它可以被转换为任何其他类型，从而可以在需要精确类型的地方使用，例如在 `match` 匹配分支。 这允许我们编写如下代码：

```rust
fn main() {
    fn sum_odd_numbers(up_to: u32) -> u32 {
        let mut acc = 0;
        for i in 0..up_to {
            // 注意这个 match 表达式的返回值必须为 u32，
            // 因为 “addition” 变量是这个类型。
            let addition: u32 = match i%2 == 1 {
                // “i” 变量的类型为 u32，这毫无问题。
                true => i,
                // 另一方面，“continue” 表达式不返回 u32，但它仍然没有问题，
                // 因为它永远不会返回，因此不会违反匹配表达式的类型要求。
                false => continue,
            };
            acc += addition;
        }
        acc
    }
    println!("Sum of odd numbers up to 9 (excluding): {}", sum_odd_numbers(9));
}

```

这也是永远循环（如 `loop {}`）的函数（如网络服务器）或终止进程的函数（如 `exit()`）的返回类型。\














