# 变量绑定与解构

## [为何要手动设置变量的可变性？](https://course.rs/basic/variable.html#%E4%B8%BA%E4%BD%95%E8%A6%81%E6%89%8B%E5%8A%A8%E8%AE%BE%E7%BD%AE%E5%8F%98%E9%87%8F%E7%9A%84%E5%8F%AF%E5%8F%98%E6%80%A7) <a href="#wei-he-yao-shou-dong-she-zhi-bian-liang-de-ke-bian-xing" id="wei-he-yao-shou-dong-she-zhi-bian-liang-de-ke-bian-xing"></a>

在其它大多数语言中，要么只支持声明可变的变量，要么只支持声明不可变的变量（例如函数式语言），前者为编程提供了灵活性，后者为编程提供了安全性，而 Rust 比较野，选择了两者我都要，既要灵活性又要安全性。

除了以上两个优点，还有一个很大的优点，那就是运行性能上的提升，因为将本身无需改变的变量声明为不可变在运行期会避免一些多余的 `runtime` 检查。

## [变量命名](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E5%91%BD%E5%90%8D) <a href="#bian-liang-ming-ming" id="bian-liang-ming-ming"></a>

在命名方面，和其它语言没有区别，不过当给变量命名时，需要遵循 [Rust 命名规范](https://course.rs/practice/naming.html)。Rust 语言有一些**关键字**（_keywords_），和其他语言一样，这些关键字都是被保留给 Rust 语言使用的，因此，它们不能被用作变量或函数的名称。在 [附录 A](https://course.rs/appendix/keywords.html) 中可找到关键字列表。

## [变量绑定](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E7%BB%91%E5%AE%9A) <a href="#bian-liang-bang-ding" id="bian-liang-bang-ding"></a>

在其它语言中，我们用 `var a = "hello world"` 的方式给 `a` 赋值，也就是把等式右边的 `"hello world"` 字符串赋值给变量 `a` ，而在 Rust 中，我们这样写： `let a = "hello world"` ，同时给这个过程起了另一个名字：**变量绑定**。

为何不用赋值而用绑定呢（其实你也可以称之为赋值，但是绑定的含义更清晰准确）？这里就涉及 Rust 最核心的原则——**所有权**，简单来讲，任何内存对象都是有主人的，而且一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人（聪明的读者应该能猜到，在这种情况下，该对象之前的主人就会丧失对该对象的所有权），像极了我们的现实世界，不是吗？

那为什么要引进“所有权”这个新的概念呢？请稍安勿躁，时机一旦成熟，我们就回来继续讨论这个话题。

## [变量可变性](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E5%8F%AF%E5%8F%98%E6%80%A7) <a href="#bian-liang-ke-bian-xing" id="bian-liang-ke-bian-xing"></a>

Rust 的变量在默认情况下是**不可变的**。前文提到，这是 Rust 团队为我们精心设计的语言特性之一，让我们编写的代码更安全，性能也更好。当然你可以通过 `mut` 关键字让变量变为**可变的**，让设计更灵活。

如果变量 `a` 不可变，那么一旦为它绑定值，就不能再修改 `a`。举个例子，在我们的工程目录下使用 `cargo new variables` 新建一个项目，叫做 _variables_ 。

然后在新建的 _variables_ 目录下，编辑 _src/main.rs_ ，改为下面代码：

```rust
// Some code
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

保存文件，再使用 `cargo run` 运行它，迎面而来的是一条错误提示：

```
// Some code
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error

```

具体的错误原因是 `cannot assign twice to immutable variable x`（无法对不可变的变量进行重复赋值），因为我们想为不可变的 `x` 变量再次赋值。

这种错误是为了避免无法预期的错误发生在我们的变量上：一个变量往往被多处代码所使用，其中一部分代码假定该变量的值永远不会改变，而另外一部分代码却无情的改变了这个值，在实际开发过程中，这个错误是很难被发现的，特别是在多线程编程中。

这种规则让我们的代码变得非常清晰，只有你想让你的变量改变时，它才能改变，这样就不会造成心智上的负担，也给别人阅读代码带来便利。

但是可变性也非常重要，否则我们就要像 ClojureScript 那样，每次要改变，就要重新生成一个对象，在拥有大量对象的场景，性能会变得非常低下，内存拷贝的成本异常的高。

在 Rust 中，可变性很简单，只要在变量名前加一个 `mut` 即可, 而且这种显式的声明方式还会给后来人传达这样的信息：嗯，这个变量在后面代码部分会发生改变。

为了让变量声明为可变,将 _src/main.rs_ 改为以下内容：

```rust
// Some code
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

```
// Some code
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6

```

选择可变还是不可变，更多的还是取决于你的使用场景，例如不可变可以带来安全性，但是丧失了灵活性和性能（如果你要改变，就要重新创建一个新的变量，这里涉及到内存对象的再分配）。而可变变量最大的好处就是使用上的灵活性和性能上的提升。

例如，在使用大型数据结构或者热点代码路径（被大量频繁调用）的情形下，在同一内存位置更新实例可能比复制并返回新分配的实例要更快。使用较小的数据结构时，通常创建新的实例并以更具函数式的风格来编写程序，可能会更容易理解，所以值得以较低的性能开销来确保代码清晰。

## [使用下划线开头忽略未使用的变量](https://course.rs/basic/variable.html#%E4%BD%BF%E7%94%A8%E4%B8%8B%E5%88%92%E7%BA%BF%E5%BC%80%E5%A4%B4%E5%BF%BD%E7%95%A5%E6%9C%AA%E4%BD%BF%E7%94%A8%E7%9A%84%E5%8F%98%E9%87%8F) <a href="#shi-yong-xia-hua-xian-kai-tou-hu-le-wei-shi-yong-de-bian-liang" id="shi-yong-xia-hua-xian-kai-tou-hu-le-wei-shi-yong-de-bian-liang"></a>

如果你创建了一个变量却不在任何地方使用它，Rust 通常会给你一个警告，因为这可能会是个 BUG。但是有时创建一个不会被使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时**你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头**：

```rust
// Some code
fn main() {
    let _x = 5;
    let y = 10;
}
```

可以看到，两个变量都是只有声明，没有使用，但是编译器却独独给出了 `y` 未被使用的警告，充分说明了 `_` 变量名前缀在这里发挥的作用。

值得注意的是，这里编译器还很善意的给出了提示（Rust 的编译器非常强大，这里的提示只是小意思）：将 `y` 修改 `_y` 即可。这里就不再给出代码，留给大家手动尝试并观察下运行结果。

## [变量解构](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E8%A7%A3%E6%9E%84) <a href="#bian-liang-jie-gou" id="bian-liang-jie-gou"></a>

`let` 表达式不仅仅用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容：

```rust
// Some code
fn main() {
    let (a, mut b): (bool,bool) = (true, false);
    // a = true,不可变; b = false，可变
    println!("a = {:?}, b = {:?}", a, b);

    b = true;
    assert_eq!(a, b);
}
```

### [解构式赋值](https://course.rs/basic/variable.html#%E8%A7%A3%E6%9E%84%E5%BC%8F%E8%B5%8B%E5%80%BC) <a href="#jie-gou-shi-fu-zhi" id="jie-gou-shi-fu-zhi"></a>

在 [Rust 1.59](https://course.rs/appendix/rust-versions/1.59.html) 版本后，我们可以在赋值语句的左式中使用元组、切片和结构体模式了。

```rust
// Some code
struct Struct {
    e: i32
}

fn main() {
    let (a, b, c, d, e);

    (a, b) = (1, 2);
    // _ 代表匹配一个值，但是我们不关心具体的值是什么，因此没有使用一个变量名而是使用了 _
    [c, .., d, _] = [1, 2, 3, 4, 5];
    Struct { e, .. } = Struct { e: 5 };

    assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
}
```

这种使用方式跟之前的 `let` 保持了一致性，但是 `let` 会重新绑定，而这里仅仅是对之前绑定的变量进行再赋值。

需要注意的是，使用 `+=` 的赋值语句还不支持解构式赋值。

## [变量和常量之间的差异](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E5%92%8C%E5%B8%B8%E9%87%8F%E4%B9%8B%E9%97%B4%E7%9A%84%E5%B7%AE%E5%BC%82) <a href="#bian-liang-he-chang-liang-zhi-jian-de-cha-yi" id="bian-liang-he-chang-liang-zhi-jian-de-cha-yi"></a>

变量的值不能更改可能让你想起其他另一个很多语言都有的编程概念：**常量**(_constant_)。与不可变变量一样，常量也是绑定到一个常量名且不允许更改的值，但是常量和变量之间存在一些差异：

* 常量不允许使用 `mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定它的值。
* 常量使用 `const` 关键字而不是 `let` 关键字来声明，并且值的类型**必须**标注。

我们将在下一节[数据类型](https://course.rs/basic/base-type/index.html)中介绍，因此现在暂时无需关心细节。

下面是一个常量声明的例子，其常量名为 `MAX_POINTS`，值设置为 `100,000`。（Rust 常量的命名约定是全部字母都使用大写，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性）：

```rust
// Some code
const MAX_POINTS: u32 = 100_000;
```

常量可以在任意作用域内声明，包括全局作用域，在声明的作用域内，常量在程序运行的整个过程中都有效。对于需要在多处代码共享一个不可变的值时非常有用，例如游戏中允许玩家赚取的最大点数或光速。

## [变量遮蔽(shadowing)](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E9%81%AE%E8%94%BDshadowing) <a href="#bian-liang-zhe-bi-shadowing" id="bian-liang-zhe-bi-shadowing"></a>

Rust 允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的，如下所示：

```rust
// Some code
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;

    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

这个程序首先将数值 `5` 绑定到 `x`，然后通过重复使用 `let x =` 来遮蔽之前的 `x`，并取原来的值加上 `1`，所以 `x` 的值变成了 `6`。第三个 `let` 语句同样遮蔽前面的 `x`，取之前的值并乘上 `2`，得到的 `x` 最终值为 `12`。当运行此程序，将输出以下内容：

```
// Some code
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
   ...
The value of x in the inner scope is: 12
The value of x is: 6

```

这和 `mut` 变量的使用是不同的，第二个 `let` 生成了完全不同的新变量，两个变量只是恰好拥有同样的名称，涉及一次内存对象的再分配 ，而 `mut` 声明的变量，可以修改同一个内存地址上的值，并不会发生内存对象的再分配，性能要更好。

变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字。

例如，假设有一个程序要统计一个空格字符串的空格数量：

```rust
// Some code
// 字符串类型
let spaces = "   ";
// usize数值类型
let spaces = spaces.len();

```

这种结构是允许的，因为第一个 `spaces` 变量是一个字符串类型，第二个 `spaces` 变量是一个全新的变量且和第一个具有相同的变量名，且是一个数值类型。所以变量遮蔽可以帮我们节省些脑细胞，不用去想如 `spaces_str` 和 `spaces_num` 此类的变量名；相反我们可以重复使用更简单的 `spaces` 变量名。如果你不用 `let` :

```rust
// Some code
let mut spaces = "   ";
spaces = spaces.len();

```

```
// Some code
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

error: aborting due to previous error

```

显然，Rust 对类型的要求很严格，不允许将整数类型 `usize` 赋值给字符串类型。`usize` 是一种 CPU 相关的整数类型，在[数值类型](https://course.rs/basic/base-type/numbers.html#%E6%95%B4%E6%95%B0%E7%B1%BB%E5%9E%8B)中有详细介绍。








