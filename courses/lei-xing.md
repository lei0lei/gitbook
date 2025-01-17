# 类型

## [类型转换](https://course.rs/advance/into-types/converse.html#%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2) <a href="#lei-xing-zhuan-huan" id="lei-xing-zhuan-huan"></a>

Rust 是类型安全的语言，因此在 Rust 中做类型转换不是一件简单的事，这一章节我们将对 Rust 中的类型转换进行详尽讲解。

> 高能预警：本章节有些难，可以考虑学了进阶后回头再看

### [`as`转换](https://course.rs/advance/into-types/converse.html#as%E8%BD%AC%E6%8D%A2) <a href="#as-zhuan-huan" id="as-zhuan-huan"></a>

先来看一段代码：

```rust
fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  if a < b {
    println!("Ten is less than one hundred.");
  }
}

```

能跟着这本书一直学习到这里，说明你对 Rust 已经有了一定的理解，那么一眼就能看出这段代码注定会报错，因为 `a` 和 `b` 拥有不同的类型，Rust 不允许两种不同的类型进行比较。

解决办法很简单，只要把 `b` 转换成 `i32` 类型即可，Rust 中内置了一些基本类型之间的转换，这里使用 `as` 操作符来完成： `if a < (b as i32) {...}`。那么为什么不把 `a` 转换成 `u16` 类型呢？

因为每个类型能表达的数据范围不同，如果把范围较大的类型转换成较小的类型，会造成错误，因此我们需要把范围较小的类型转换成较大的类型，来避免这些问题的发生。

> 使用类型转换需要小心，因为如果执行以下操作 `300_i32 as i8`，你将获得 `44` 这个值，而不是 `300`，因为 `i8` 类型能表达的的最大值为 `2^7 - 1`，使用以下代码可以查看 `i8` 的最大值：

```rust

#![allow(unused)]
fn main() {
let a = i8::MAX;
println!("{}",a);
}

```

下面列出了常用的转换形式：

```rust
fn main() {
   let a = 3.1 as i8;
   let b = 100_i8 as i32;
   let c = 'a' as u8; // 将字符'a'转换为整数，97

   println!("{},{},{}",a,b,c)
}

```

[**内存地址转换为指针**](https://course.rs/advance/into-types/converse.html#%E5%86%85%E5%AD%98%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2%E4%B8%BA%E6%8C%87%E9%92%88)

```rust

#![allow(unused)]
fn main() {
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize; // 将p1内存地址转换为一个整数
let second_address = first_address + 4; // 4 == std::mem::size_of::<i32>()，i32类型占用4个字节，因此将内存地址 + 4
let p2 = second_address as *mut i32; // 访问该地址指向的下一个整数p2
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
}

```

[**强制类型转换的边角知识**](https://course.rs/advance/into-types/converse.html#%E5%BC%BA%E5%88%B6%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E7%9A%84%E8%BE%B9%E8%A7%92%E7%9F%A5%E8%AF%86)

1. 转换不具有传递性 就算 `e as U1 as U2` 是合法的，也不能说明 `e as U2` 是合法的（`e` 不能直接转换成 `U2`）。

### [TryInto 转换](https://course.rs/advance/into-types/converse.html#tryinto-%E8%BD%AC%E6%8D%A2) <a href="#tryinto-zhuan-huan" id="tryinto-zhuan-huan"></a>

在一些场景中，使用 `as` 关键字会有比较大的限制。如果你想要在类型转换上拥有完全的控制而不依赖内置的转换，例如处理转换错误，那么可以使用 `TryInto` ：

```rust
use std::convert::TryInto;

fn main() {
   let a: u8 = 10;
   let b: u16 = 1500;

   let b_: u8 = b.try_into().unwrap();

   if a < b_ {
     println!("Ten is less than one hundred.");
   }
}

```

上面代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能有些同学会为此困惑，主要原因在于**如果你要使用一个特征的方法，那么你需要引入该特征到当前的作用域中**，我们在上面用到了 `try_into` 方法，因此需要引入对应的特征。但是 Rust 又提供了一个非常便利的办法，把最常用的标准库中的特征通过[`std::prelude`](https://course.rs/appendix/prelude.html)模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`，你可以尝试删除第一行的代码 `use ...`，看看是否会报错。

`try_into` 会尝试进行一次转换，并返回一个 `Result`，此时就可以对其进行相应的错误处理。由于我们的例子只是为了快速测试，因此使用了 `unwrap` 方法，该方法在发现错误时，会直接调用 `panic` 导致程序的崩溃退出，在实际项目中，请不要这么使用，具体见[panic](https://course.rs/basic/result-error/panic.html#%E8%B0%83%E7%94%A8-panic)部分。

最主要的是 `try_into` 转换会捕获大类型向小类型转换时导致的溢出错误：

```rust
fn main() {
    let b: i16 = 1500;

    let b_: u8 = match b.try_into() {
        Ok(b1) => b1,
        Err(e) => {
            println!("{:?}", e.to_string());
            0
        }
    };
}

```

运行后输出如下 `"out of range integral type conversion attempted"`，在这里我们程序捕获了错误，编译器告诉我们类型范围超出的转换是不被允许的，因为我们试图把 `1500_i16` 转换为 `u8` 类型，后者明显不足以承载这么大的值。

### [通用类型转换](https://course.rs/advance/into-types/converse.html#%E9%80%9A%E7%94%A8%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2) <a href="#tong-yong-lei-xing-zhuan-huan" id="tong-yong-lei-xing-zhuan-huan"></a>

虽然 `as` 和 `TryInto` 很强大，但是只能应用在数值类型上，可是 Rust 有如此多的类型，想要为这些类型实现转换，我们需要另谋出路，先来看看在一个笨办法，将一个结构体转换为另外一个结构体：

```rust

#![allow(unused)]
fn main() {
struct Foo {
    x: u32,
    y: u16,
}

struct Bar {
    a: u32,
    b: u16,
}

fn reinterpret(foo: Foo) -> Bar {
    let Foo { x, y } = foo;
    Bar { a: x, b: y }
}
}

```

简单粗暴，但是从另外一个角度来看，也挺啰嗦的，好在 Rust 为我们提供了更通用的方式来完成这个目的。

[**强制类型转换**](https://course.rs/advance/into-types/converse.html#%E5%BC%BA%E5%88%B6%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)

在某些情况下，类型是可以进行隐式强制转换的，虽然这些转换弱化了 Rust 的类型系统，但是它们的存在是为了让 Rust 在大多数场景可以工作(说白了，帮助用户省事)，而不是报各种类型上的编译错误。

首先，在匹配特征时，不会做任何强制转换(除了方法)。一个类型 `T` 可以强制转换为 `U`，不代表 `impl T` 可以强制转换为 `impl U`，例如下面的代码就无法通过编译检查：

```rust
trait Trait {}

fn foo<X: Trait>(t: X) {}

impl<'a> Trait for &'a i32 {}

fn main() {
    let t: &mut i32 = &mut 0;
    foo(t);
}

```

报错如下：

```console
error[E0277]: the trait bound `&mut i32: Trait` is not satisfied
--> src/main.rs:9:9
|
9 |     foo(t);
|         ^ the trait `Trait` is not implemented for `&mut i32`
|
= help: the following implementations were found:
        <&'a i32 as Trait>
= note: `Trait` is implemented for `&i32`, but not for `&mut i32`
```

`&i32` 实现了特征 `Trait`， `&mut i32` 可以转换为 `&i32`，但是 `&mut i32` 依然无法作为 `Trait` 来使用。

[**点操作符**](https://course.rs/advance/into-types/converse.html#%E7%82%B9%E6%93%8D%E4%BD%9C%E7%AC%A6)

方法调用的点操作符看起来简单，实际上非常不简单，它在调用时，会发生很多魔法般的类型转换，例如：自动引用、自动解引用，强制类型转换直到类型能匹配等。

假设有一个方法 `foo`，它有一个接收器(接收器就是 `self`、`&self`、`&mut self` 参数)。如果调用 `value.foo()`，编译器在调用 `foo` 之前，需要决定到底使用哪个 `Self` 类型来调用。现在假设 `value` 拥有类型 `T`。

再进一步，我们使用[完全限定语法](https://course.rs/basic/trait/advance-trait.html#%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E8%AF%AD%E6%B3%95)来进行准确的函数调用:

1. 首先，编译器检查它是否可以直接调用 `T::foo(value)`，称之为**值方法调用**
2. 如果上一步调用无法完成(例如方法类型错误或者特征没有针对 `Self` 进行实现，上文提到过特征不能进行强制转换)，那么编译器会尝试增加自动引用，例如会尝试以下调用： `<&T>::foo(value)` 和 `<&mut T>::foo(value)`，称之为**引用方法调用**
3. 若上面两个方法依然不工作，编译器会试着解引用 `T` ，然后再进行尝试。这里使用了 `Deref` 特征 —— 若 `T: Deref<Target = U>` (`T` 可以被解引用为 `U`)，那么编译器会使用 `U` 类型进行尝试，称之为**解引用方法调用**
4. 若 `T` 不能被解引用，且 `T` 是一个定长类型(在编译器类型长度是已知的)，那么编译器也会尝试将 `T` 从定长类型转为不定长类型，例如将 `[i32; 2]` 转为 `[i32]`
5. 若还是不行，那...没有那了，最后编译器大喊一声：汝欺我甚，不干了！

下面我们来用一个例子来解释上面的方法查找算法:

```rust

#![allow(unused)]
fn main() {
let array: Rc<Box<[T; 3]>> = ...;
let first_entry = array[0];
}

```

`array` 数组的底层数据隐藏在了重重封锁之后，那么编译器如何使用 `array[0]` 这种数组原生访问语法通过重重封锁，准确的访问到数组中的第一个元素？

1. 首先， `array[0]` 只是[`Index`](https://doc.rust-lang.org/std/ops/trait.Index.html)特征的语法糖：编译器会将 `array[0]` 转换为 `array.index(0)` 调用，当然在调用之前，编译器会先检查 `array` 是否实现了 `Index` 特征。
2. 接着，编译器检查 `Rc<Box<[T; 3]>>` 是否有实现 `Index` 特征，结果是否，不仅如此，`&Rc<Box<[T; 3]>>` 与 `&mut Rc<Box<[T; 3]>>` 也没有实现。
3. 上面的都不能工作，编译器开始对 `Rc<Box<[T; 3]>>` 进行解引用，把它转变成 `Box<[T; 3]>`
4. 此时继续对 `Box<[T; 3]>` 进行上面的操作 ：`Box<[T; 3]>`， `&Box<[T; 3]>`，和 `&mut Box<[T; 3]>` 都没有实现 `Index` 特征，所以编译器开始对 `Box<[T; 3]>` 进行解引用，然后我们得到了 `[T; 3]`
5. `[T; 3]` 以及它的各种引用都没有实现 `Index` 索引(是不是很反直觉:D，在直觉中，数组都可以通过索引访问，实际上只有数组切片才可以!)，它也不能再进行解引用，因此编译器只能祭出最后的大杀器：将定长转为不定长，因此 `[T; 3]` 被转换成 `[T]`，也就是数组切片，它实现了 `Index` 特征，因此最终我们可以通过 `index` 方法访问到对应的元素。

过程看起来很复杂，但是也还好，挺好理解，如果你现在不能彻底理解，也不要紧，等以后对 Rust 理解更深了，同时需要深入理解类型转换时，再来细细品读本章。

再来看看以下更复杂的例子：

```rust

#![allow(unused)]
fn main() {
fn do_stuff<T: Clone>(value: &T) {
    let cloned = value.clone();
}
}

```

上面例子中 `cloned` 的类型是什么？首先编译器检查能不能进行**值方法调用**， `value` 的类型是 `&T`，同时 `clone` 方法的签名也是 `&T` ： `fn clone(&T) -> T`，因此可以进行值方法调用，再加上编译器知道了 `T` 实现了 `Clone`，因此 `cloned` 的类型是 `T`。

如果 `T: Clone` 的特征约束被移除呢？

```rust

#![allow(unused)]
fn main() {
fn do_stuff<T>(value: &T) {
    let cloned = value.clone();
}
}

```

首先，从直觉上来说，该方法会报错，因为 `T` 没有实现 `Clone` 特征，但是真实情况是什么呢？

我们先来推导一番。 首先通过值方法调用就不再可行，因为 `T` 没有实现 `Clone` 特征，也就无法调用 `T` 的 `clone` 方法。接着编译器尝试**引用方法调用**，此时 `T` 变成 `&T`，在这种情况下， `clone` 方法的签名如下： `fn clone(&&T) -> &T`，接着我们现在对 `value` 进行了引用。 编译器发现 `&T` 实现了 `Clone` 类型(所有的引用类型都可以被复制，因为其实就是复制一份地址)，因此可以推出 `cloned` 也是 `&T` 类型。

最终，我们复制出一份引用指针，这很合理，因为值类型 `T` 没有实现 `Clone`，只能去复制一个指针了。

下面的例子也是自动引用生效的地方：

```rust

#![allow(unused)]
fn main() {
#[derive(Clone)]
struct Container<T>(Arc<T>);

fn clone_containers<T>(foo: &Container<i32>, bar: &Container<T>) {
    let foo_cloned = foo.clone();
    let bar_cloned = bar.clone();
}
}

```

推断下上面的 `foo_cloned` 和 `bar_cloned` 是什么类型？提示: 关键在 `Container` 的泛型参数，一个是 `i32` 的具体类型，一个是泛型类型，其中 `i32` 实现了 `Clone`，但是 `T` 并没有。

首先要复习一下复杂类型派生 `Clone` 的规则：一个复杂类型能否派生 `Clone`，需要它内部的所有子类型都能进行 `Clone`。因此 `Container<T>(Arc<T>)` 是否实现 `Clone` 的关键在于 `T` 类型是否实现了 `Clone` 特征。

上面代码中，`Container<i32>` 实现了 `Clone` 特征，因此编译器可以直接进行值方法调用，此时相当于直接调用 `foo.clone`，其中 `clone` 的函数签名是 `fn clone(&T) -> T`，由此可以看出 `foo_cloned` 的类型是 `Container<i32>`。

然而，`bar_cloned` 的类型却是 `&Container<T>`，这个不合理啊，明明我们为 `Container<T>` 派生了 `Clone` 特征，因此它也应该是 `Container<T>` 类型才对。万事皆有因，我们先来看下 `derive` 宏最终生成的代码大概是啥样的：

```rust

#![allow(unused)]
fn main() {
impl<T> Clone for Container<T> where T: Clone {
    fn clone(&self) -> Self {
        Self(Arc::clone(&self.0))
    }
}
}

```

从上面代码可以看出，派生 `Clone` 能实现的根本是 `T` 实现了[`Clone`特征](https://doc.rust-lang.org/std/clone/trait.Clone.html#derivable)：`where T: Clone`， 因此 `Container<T>` 就没有实现 `Clone` 特征。

编译器接着会去尝试引用方法调用，此时 `&Container<T>` 引用实现了 `Clone`，最终可以得出 `bar_cloned` 的类型是 `&Container<T>`。

当然，也可以为 `Container<T>` 手动实现 `Clone` 特征：

```rust

#![allow(unused)]
fn main() {
impl<T> Clone for Container<T> {
    fn clone(&self) -> Self {
        Self(Arc::clone(&self.0))
    }
}
}

```

此时，编译器首次尝试值方法调用即可通过，因此 `bar_cloned` 的类型变成 `Container<T>`。

这一块儿内容真的挺复杂，每一个坚持看完的读者都是真正的勇士，我也是：为了写好这块儿内容，作者足足花了 **4** 个小时！

[**变形记(Transmutes)**](https://course.rs/advance/into-types/converse.html#%E5%8F%98%E5%BD%A2%E8%AE%B0transmutes)

前方危险，敬请绕行！

类型系统，你让开！我要自己转换这些类型，不成功便成仁！虽然本书大多是关于安全的内容，我还是希望你能仔细考虑避免使用本章讲到的内容。这是你在 Rust 中所能做到的真真正正、彻彻底底、最最可怕的非安全行为，在这里，所有的保护机制都形同虚设。

先让你看看深渊长什么样，开开眼，然后你再决定是否深入： `mem::transmute<T, U>` 将类型 `T` 直接转成类型 `U`，唯一的要求就是，这两个类型占用同样大小的字节数！我的天，这也算限制？这简直就是无底线的转换好吧？看看会导致什么问题：

1. 首先也是最重要的，转换后创建一个任意类型的实例会造成无法想象的混乱，而且根本无法预测。不要把 `3` 转换成 `bool` 类型，就算你根本不会去使用该 `bool` 类型，也不要去这样转换
2. 变形后会有一个重载的返回类型，即使你没有指定返回类型，为了满足类型推导的需求，依然会产生千奇百怪的类型
3. 将 `&` 变形为 `&mut` 是未定义的行为
   * 这种转换永远都是未定义的
   * 不，你不能这么做
   * 不要多想，你没有那种幸运
4. 变形为一个未指定生命周期的引用会导致[无界生命周期](https://course.rs/advance/lifetime/advance.html)
5. 在复合类型之间互相变换时，你需要保证它们的排列布局是一模一样的！一旦不一样，那么字段就会得到不可预期的值，这也是未定义的行为，至于你会不会因此愤怒， **WHO CARES** ，你都用了变形了，老兄！

对于第 5 条，你该如何知道内存的排列布局是一样的呢？对于 `repr(C)` 类型和 `repr(transparent)` 类型来说，它们的布局是有着精确定义的。但是对于你自己的"普通却自信"的 Rust 类型 `repr(Rust)` 来说，它可不是有着精确定义的。甚至同一个泛型类型的不同实例都可以有不同的内存布局。 `Vec<i32>` 和 `Vec<u32>` 它们的字段可能有着相同的顺序，也可能没有。对于数据排列布局来说，**什么能保证，什么不能保证**目前还在 Rust 开发组的[工作任务](https://rust-lang.github.io/unsafe-code-guidelines/layout.html)中呢。

你以为你之前凝视的是深渊吗？不，你凝视的只是深渊的大门。 `mem::transmute_copy<T, U>` 才是真正的深渊，它比之前的还要更加危险和不安全。它从 `T` 类型中拷贝出 `U` 类型所需的字节数，然后转换成 `U`。 `mem::transmute` 尚有大小检查，能保证两个数据的内存大小一致，现在这哥们干脆连这个也丢了，只不过 `U` 的尺寸若是比 `T` 大，会是一个未定义行为。

当然，你也可以通过裸指针转换和 `unions` (todo!)获得所有的这些功能，但是你将无法获得任何编译提示或者检查。裸指针转换和 `unions` 也不是魔法，无法逃避上面说的规则。

`transmute` 虽然危险，但作为一本工具书，知识当然要全面，下面列举两个有用的 `transmute` 应用场景 :)。

* 将裸指针变成函数指针：

```rust

#![allow(unused)]
fn main() {
fn foo() -> i32 {
    0
}

let pointer = foo as *const ();
let function = unsafe { 
    // 将裸指针转换为函数指针
    std::mem::transmute::<*const (), fn() -> i32>(pointer) 
};
assert_eq!(function(), 0);
}

```

* 延长生命周期，或者缩短一个静态生命周期寿命：

```rust

#![allow(unused)]
fn main() {
struct R<'a>(&'a i32);

// 将 'b 生命周期延长至 'static 生命周期
unsafe fn extend_lifetime<'b>(r: R<'b>) -> R<'static> {
    std::mem::transmute::<R<'b>, R<'static>>(r)
}

// 将 'static 生命周期缩短至 'c 生命周期
unsafe fn shorten_invariant_lifetime<'b, 'c>(r: &'b mut R<'static>) -> &'b mut R<'c> {
    std::mem::transmute::<&'b mut R<'static>, &'b mut R<'c>>(r)
}
}

```

以上例子非常先进！但是是非常不安全的 Rust 行为！

## [newtype](https://course.rs/advance/into-types/custom-type.html#newtype) <a href="#newtype" id="newtype"></a>

何为 `newtype`？简单来说，就是使用[元组结构体](https://course.rs/basic/compound-type/struct.html#%E5%85%83%E7%BB%84%E7%BB%93%E6%9E%84%E4%BD%93tuple-struct)的方式将已有的类型包裹起来：`struct Meters(u32);`，那么此处 `Meters` 就是一个 `newtype`。

为何需要 `newtype`？Rust 这多如繁星的 Old 类型满足不了我们吗？这是因为：

* 自定义类型可以让我们给出更有意义和可读性的类型名，例如与其使用 `u32` 作为距离的单位类型，我们可以使用 `Meters`，它的可读性要好得多
* 对于某些场景，只有 `newtype` 可以很好地解决
* 隐藏内部类型的细节

一箩筐的理由～～ 让我们先从第二点讲起。

[**为外部类型实现外部特征**](https://course.rs/advance/into-types/custom-type.html#%E4%B8%BA%E5%A4%96%E9%83%A8%E7%B1%BB%E5%9E%8B%E5%AE%9E%E7%8E%B0%E5%A4%96%E9%83%A8%E7%89%B9%E5%BE%81)

在之前的章节中，我们有讲过，如果在外部类型上实现外部特征必须使用 `newtype` 的方式，否则你就得遵循孤儿规则：要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` 必须至少有一个在当前的作用范围内。

例如，如果想使用 `println!("{}", v)` 的方式去格式化输出一个动态数组 `Vec`，以期给用户提供更加清晰可读的内容，那么就需要为 `Vec` 实现 `Display` 特征，但是这里有一个问题： `Vec` 类型定义在标准库中，`Display` 亦然，这时就可以祭出大杀器 `newtype` 来解决：

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}

```

如上所示，使用元组结构体语法 `struct Wrapper(Vec<String>)` 创建了一个 `newtype` Wrapper，然后为它实现 `Display` 特征，最终实现了对 `Vec` 动态数组的格式化输出。

[**更好的可读性及类型异化**](https://course.rs/advance/into-types/custom-type.html#%E6%9B%B4%E5%A5%BD%E7%9A%84%E5%8F%AF%E8%AF%BB%E6%80%A7%E5%8F%8A%E7%B1%BB%E5%9E%8B%E5%BC%82%E5%8C%96)

首先，更好的可读性不等于更少的代码（如果你学过 Scala，相信会深有体会），其次下面的例子只是一个示例，未必能体现出更好的可读性：

```rust
use std::ops::Add;
use std::fmt;

struct Meters(u32);
impl fmt::Display for Meters {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "目标地点距离你{}米", self.0)
    }
}

impl Add for Meters {
    type Output = Self;

    fn add(self, other: Meters) -> Self {
        Self(self.0 + other.0)
    }
}
fn main() {
    let d = calculate_distance(Meters(10), Meters(20));
    println!("{}", d);
}

fn calculate_distance(d1: Meters, d2: Meters) -> Meters {
    d1 + d2
}

```

上面代码创建了一个 `newtype` Meters，为其实现 `Display` 和 `Add` 特征，接着对两个距离进行求和计算，最终打印出该距离：

```console
目标地点距离你30米
```

事实上，除了可读性外，还有一个极大的优点：如果给 `calculate_distance` 传一个其它的类型，例如 `struct MilliMeters(u32);`，该代码将无法编译。尽管 `Meters` 和 `MilliMeters` 都是对 `u32` 类型的简单包装，但是**它们是不同的类型**！

[**隐藏内部类型的细节**](https://course.rs/advance/into-types/custom-type.html#%E9%9A%90%E8%97%8F%E5%86%85%E9%83%A8%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%BB%86%E8%8A%82)

众所周知，Rust 的类型有很多自定义的方法，假如我们把某个类型传给了用户，但是又不想用户调用这些方法，就可以使用 `newtype`：

```rust
struct Meters(u32);

fn main() {
    let i: u32 = 2;
    assert_eq!(i.pow(2), 4);

    let n = Meters(i);
    // 下面的代码将报错，因为`Meters`类型上没有`pow`方法
    // assert_eq!(n.pow(2), 4);
}

```

不过需要偷偷告诉你的是，这种方式实际上是掩耳盗铃，因为用户依然可以通过 `n.0.pow(2)` 的方式来调用内部类型的方法 :)

### [类型别名(Type Alias)](https://course.rs/advance/into-types/custom-type.html#%E7%B1%BB%E5%9E%8B%E5%88%AB%E5%90%8Dtype-alias) <a href="#lei-xing-bie-ming-typealias" id="lei-xing-bie-ming-typealias"></a>

除了使用 `newtype`，我们还可以使用一个更传统的方式来创建新类型：类型别名

```rust
type Meters = u32
```

嗯，不得不说，类型别名的方式看起来比 `newtype` 顺眼的多，而且跟其它语言的使用方式几乎一致，但是： **类型别名并不是一个独立的全新的类型，而是某一个类型的别名**，因此编译器依然会把 `Meters` 当 `u32` 来使用：

```rust

#![allow(unused)]
fn main() {
type Meters = u32;

let x: u32 = 5;
let y: Meters = 5;

println!("x + y = {}", x + y);
}

```

上面的代码将顺利编译通过，但是如果你使用 `newtype` 模式，该代码将无情报错，简单做个总结：

* 类型别名仅仅是别名，只是为了让可读性更好，并不是全新的类型，`newtype` 才是！
* 类型别名无法实&#x73B0;_&#x4E3A;外部类型实现外部特&#x5F81;_&#x7B49;功能，而 `newtype` 可以

类型别名除了让类型可读性更好，还能**减少模版代码的使用**：

```rust

#![allow(unused)]
fn main() {
let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
    // --snip--
}

fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
    // --snip--
}
}

```

`f` 是一个令人眼花缭乱的类型 `Box<dyn Fn() + Send + 'static>`，如果仔细看，会发现其实只有一个 `Send` 特征不认识，`Send` 是什么在这里不重要，你只需理解，`f` 就是一个 `Box<dyn T>` 类型的特征对象，实现了 `Fn()` 和 `Send` 特征，同时生命周期为 `'static`。

因为 `f` 的类型贼长，导致了后面我们在使用它时，到处都充斥这些不太优美的类型标注，好在类型别名可解君忧：

```rust

#![allow(unused)]
fn main() {
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
}

```

Bang！是不是？！立刻大幅简化了我们的使用。喝着奶茶、哼着歌、我写起代码撩起妹，何其快哉！

在标准库中，类型别名应用最广的就是简化 `Result<T, E>` 枚举。

例如在 `std::io` 库中，它定义了自己的 `Error` 类型：`std::io::Error`，那么如果要使用该 `Result` 就要用这样的语法：`std::result::Result<T, std::io::Error>;`，想象一下代码中充斥着这样的东东是一种什么感受？颤抖吧。。。

由于使用 `std::io` 库时，它的所有错误类型都是 `std::io::Error`，那么我们完全可以把该错误对用户隐藏起来，只在内部使用即可，因此就可以使用类型别名来简化实现：

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

Bingo，这样一来，其它库只需要使用 `std::io::Result<T>` 即可替代冗长的 `std::result::Result<T, std::io::Error>` 类型。

更香的是，由于它只是别名，因此我们可以用它来调用真实类型的所有方法，甚至包括 `?` 符号！

### [!永不返回类型](https://course.rs/advance/into-types/custom-type.html#%E6%B0%B8%E4%B8%8D%E8%BF%94%E5%9B%9E%E7%B1%BB%E5%9E%8B) <a href="#yong-bu-fan-hui-lei-xing" id="yong-bu-fan-hui-lei-xing"></a>

在[函数](https://course.rs/basic/base-type/function.html#%E6%B0%B8%E4%B8%8D%E8%BF%94%E5%9B%9E%E7%9A%84%E5%87%BD%E6%95%B0)那章，曾经介绍过 `!` 类型：`!` 用来说明一个函数永不返回任何值，当时可能体会不深，没事，在学习了更多手法后，保证你有全新的体验：

```rust
fn main() {
    let i = 2;
    let v = match i {
       0..=3 => i,
       _ => println!("不合规定的值:{}", i)
    };
}

```

上面函数，会报出一个编译错误:

```console
error[E0308]: `match` arms have incompatible types // match的分支类型不同
 --> src/main.rs:5:13
  |
3 |       let v = match i {
  |  _____________-
4 | |        0..3 => i,
  | |                - this is found to be of type `{integer}` // 该分支返回整数类型
5 | |        _ => println!("不合规定的值:{}", i)
  | |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected integer, found `()` // 该分支返回()单元类型
6 | |     };
  | |_____- `match` arms have incompatible types
```

原因很简单: 要赋值给 `v`，就必须保证 `match` 的各个分支返回的值是同一个类型，但是上面一个分支返回数值、另一个分支返回元类型 `()`，自然会出错。

既然 `println` 不行，那再试试 `panic`

```rust
fn main() {
    let i = 2;
    let v = match i {
       0..=3 => i,
       _ => panic!("不合规定的值:{}", i)
    };
}

```

神奇的事发生了，此处 `panic` 竟然通过了编译。难道这两个宏拥有不同的返回类型？

你猜的没错：`panic` 的返回值是 `!`，代表它决不会返回任何值，既然没有任何返回值，那自然不会存在分支类型不匹配的情况。

## [Sized 和不定长类型 DST](https://course.rs/advance/into-types/sized.html#sized-%E5%92%8C%E4%B8%8D%E5%AE%9A%E9%95%BF%E7%B1%BB%E5%9E%8B-dst) <a href="#sized-he-bu-ding-chang-lei-xing-dst" id="sized-he-bu-ding-chang-lei-xing-dst"></a>

在 Rust 中类型有多种抽象的分类方式，例如本书之前章节的：基本类型、集合类型、复合类型等。再比如说，如果从编译器何时能获知类型大小的角度出发，可以分成两类:

* 定长类型( sized )，这些类型的大小在编译时是已知的
* 不定长类型( unsized )，与定长类型相反，它的大小只有到了程序运行时才能动态获知，这种类型又被称之为 DST

首先，我们来深入看看何为 DST。

### [动态大小类型 DST](https://course.rs/advance/into-types/sized.html#%E5%8A%A8%E6%80%81%E5%A4%A7%E5%B0%8F%E7%B1%BB%E5%9E%8B-dst) <a href="#dong-tai-da-xiao-lei-xing-dst" id="dong-tai-da-xiao-lei-xing-dst"></a>

读者大大们之前学过的几乎所有类型，都是固定大小的类型，包括集合 `Vec`、`String` 和 `HashMap` 等，而动态大小类型刚好与之相反：**编译器无法在编译期得知该类型值的大小，只有到了程序运行时，才能动态获知**。对于动态类型，我们使用 `DST`(dynamically sized types)或者 `unsized` 类型来称呼它。

上述的这些集合虽然底层数据可动态变化，感觉像是动态大小的类型。但是实际上，**这些底层数据只是保存在堆上，在栈中还存有一个引用类型**，该引用包含了集合的内存地址、元素数目、分配空间信息，通过这些信息，编译器对于该集合的实际大小了若指掌，最最重要的是：**栈上的引用类型是固定大小的**，因此它们依然是固定大小的类型。

**正因为编译器无法在编译期获知类型大小，若你试图在代码中直接使用 DST 类型，将无法通过编译。**

现在给你一个挑战：想出几个 DST 类型。俺厚黑地说一句，估计大部分人都想不出这样的一个类型，就连我，如果不是查询着资料在写，估计一时半会儿也想不到一个。

先来看一个最直白的：

[**试图创建动态大小的数组**](https://course.rs/advance/into-types/sized.html#%E8%AF%95%E5%9B%BE%E5%88%9B%E5%BB%BA%E5%8A%A8%E6%80%81%E5%A4%A7%E5%B0%8F%E7%9A%84%E6%95%B0%E7%BB%84)

```rust

#![allow(unused)]
fn main() {
fn my_function(n: usize) {
    let array = [123; n];
}
}

```

以上代码就会报错(错误输出的内容并不是因为 DST，但根本原因是类似的)，因为 `n` 在编译期无法得知，而数组类型的一个组成部分就是长度，长度变为动态的，自然类型就变成了 unsized 。

[**切片**](https://course.rs/advance/into-types/sized.html#%E5%88%87%E7%89%87)

切片也是一个典型的 DST 类型，具体详情参见另一篇文章: [易混淆的切片和切片引用](https://course.rs/difficulties/slice.html)。

[**str**](https://course.rs/advance/into-types/sized.html#str)

考虑一下这个类型：`str`，感觉有点眼生？是的，它既不是 `String` 动态字符串，也不是 `&str` 字符串切片，而是一个 `str`。它是一个动态类型，同时还是 `String` 和 `&str` 的底层数据类型。 由于 `str` 是动态类型，因此它的大小直到运行期才知道，下面的代码会因此报错：

```rust

#![allow(unused)]
fn main() {
// error
let s1: str = "Hello there!";
let s2: str = "How's it going?";

// ok
let s3: &str = "on?"
}

```

Rust 需要明确地知道一个特定类型的值占据了多少内存空间，同时该类型的所有值都必须使用相同大小的内存。如果 Rust 允许我们使用这种动态类型，那么这两个 `str` 值就需要占用同样大小的内存，这显然是不现实的: `s1` 占用了 12 字节，`s2` 占用了 15 字节，总不至于为了满足同样的内存大小，用空白字符去填补字符串吧？

所以，我们只有一条路走，那就是给它们一个固定大小的类型：`&str`。那么为何字符串切片 `&str` 就是固定大小呢？因为它的引用存储在栈上，具有固定大小(类似指针)，同时它指向的数据存储在堆中，也是已知的大小，再加上 `&str` 引用中包含有堆上数据内存地址、长度等信息，因此最终可以得出字符串切片是固定大小类型的结论。

与 `&str` 类似，`String` 字符串也是固定大小的类型。

正是因为 `&str` 的引用有了底层堆数据的明确信息，它才是固定大小类型。假设如果它没有这些信息呢？那它也将变成一个动态类型。因此，将动态数据固定化的秘诀就是**使用引用指向这些动态数据，然后在引用中存储相关的内存位置、长度等信息**。

[**特征对象**](https://course.rs/advance/into-types/sized.html#%E7%89%B9%E5%BE%81%E5%AF%B9%E8%B1%A1)

```rust

#![allow(unused)]
fn main() {
fn foobar_1(thing: &dyn MyThing) {}     // OK
fn foobar_2(thing: Box<dyn MyThing>) {} // OK
fn foobar_3(thing: MyThing) {}          // ERROR!
}

```

如上所示，只能通过引用或 `Box` 的方式来使用特征对象，直接使用将报错！

[**总结：只能间接使用的 DST**](https://course.rs/advance/into-types/sized.html#%E6%80%BB%E7%BB%93%E5%8F%AA%E8%83%BD%E9%97%B4%E6%8E%A5%E4%BD%BF%E7%94%A8%E7%9A%84-dst)

Rust 中常见的 `DST` 类型有: `str`、`[T]`、`dyn Trait`，**它们都无法单独被使用，必须要通过引用或者 `Box` 来间接使用** 。

我们之前已经见过，使用 `Box` 将一个没有固定大小的特征变成一个有固定大小的特征对象，那能否故技重施，将 `str` 封装成一个固定大小类型？留个悬念先，我们来看看 `Sized` 特征。

### [Sized 特征](https://course.rs/advance/into-types/sized.html#sized-%E7%89%B9%E5%BE%81) <a href="#sized-te-zheng" id="sized-te-zheng"></a>

既然动态类型的问题这么大，那么在使用泛型时，Rust 如何保证我们的泛型参数是固定大小的类型呢？例如以下泛型函数：

```rust

#![allow(unused)]
fn main() {
fn generic<T>(t: T) {
    // --snip--
}
}

```

该函数很简单，就一个泛型参数 T，那么如何保证 `T` 是固定大小的类型？仔细回想下，貌似在之前的课程章节中，我们也没有做过任何事情去做相关的限制，那 `T` 怎么就成了固定大小的类型了？奥秘在于编译器自动帮我们加上了 `Sized` 特征约束：

```rust

#![allow(unused)]
fn main() {
fn generic<T: Sized>(t: T) {
    // --snip--
}
}

```

在上面，Rust 自动添加的特征约束 `T: Sized`，表示泛型函数只能用于一切实现了 `Sized` 特征的类型上，而**所有在编译时就能知道其大小的类型，都会自动实现 `Sized` 特征**，例如。。。。也没啥好例如的，你能想到的几乎所有类型都实现了 `Sized` 特征，除了上面那个坑坑的 `str`，哦，还有特征。

**每一个特征都是一个可以通过名称来引用的动态大小类型**。因此如果想把特征作为具体的类型来传递给函数，你必须将其转换成一个特征对象：诸如 `&dyn Trait` 或者 `Box<dyn Trait>` (还有 `Rc<dyn Trait>`)这些引用类型。

现在还有一个问题：假如想在泛型函数中使用动态数据类型怎么办？可以使用 `?Sized` 特征(不得不说这个命名方式很 Rusty，竟然有点幽默)：

```rust

#![allow(unused)]
fn main() {
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
}

```

`?Sized` 特征用于表明类型 `T` 既有可能是固定大小的类型，也可能是动态大小的类型。还有一点要注意的是，函数参数类型从 `T` 变成了 `&T`，因为 `T` 可能是动态大小的，因此需要用一个固定大小的指针(引用)来包裹它。

### [`Box<str>`](https://course.rs/advance/into-types/sized.html#boxstr) <a href="#boxstr" id="boxstr"></a>

在结束前，再来看看之前遗留的问题：使用 `Box` 可以将一个动态大小的特征变成一个具有固定大小的特征对象，能否故技重施，将 `str` 封装成一个固定大小类型？

先回想下，章节前面的内容介绍过该如何把一个动态大小类型转换成固定大小的类型： **使用引用指向这些动态数据，然后在引用中存储相关的内存位置、长度等信息**。

好的，根据这个，我们来一起推测。首先，`Box<str>` 使用了一个引用来指向 `str`，嗯，满足了第一个条件。但是第二个条件呢？`Box` 中有该 `str` 的长度信息吗？显然是 `No`。那为什么特征就可以变成特征对象？其实这个还蛮复杂的，简单来说，对于特征对象，编译器无需知道它具体是什么类型，只要知道它能调用哪几个方法即可，因此编译器帮我们实现了剩下的一切。

来验证下我们的推测：

```rust
fn main() {
    let s1: Box<str> = Box::new("Hello there!" as str);
}

```

报错如下：

```
error[E0277]: the size for values of type `str` cannot be known at compilation time
 --> src/main.rs:2:24
  |
2 |     let s1: Box<str> = Box::new("Hello there!" as str);
  |                        ^^^^^^^^ doesn't have a size known at compile-time
  |
  = help: the trait `Sized` is not implemented for `str`
  = note: all function arguments must have a statically known size
```

提示得很清晰，不知道 `str` 的大小，因此无法使用这种语法进行 `Box` 进装，但是你可以这么做:

```rust
let s1: Box<str> = "Hello there!".into();
```

主动转换成 `str` 的方式不可行，但是可以让编译器来帮我们完成，只要告诉它我们需要的类型即可。

## [整数转换为枚举](https://course.rs/advance/into-types/enum-int.html#%E6%95%B4%E6%95%B0%E8%BD%AC%E6%8D%A2%E4%B8%BA%E6%9E%9A%E4%B8%BE) <a href="#zheng-shu-zhuan-huan-wei-mei-ju" id="zheng-shu-zhuan-huan-wei-mei-ju"></a>

在 Rust 中，从枚举到整数的转换很容易，但是反过来，就没那么容易，甚至部分实现还挺邪恶, 例如使用`transmute`。

### [一个真实场景的需求](https://course.rs/advance/into-types/enum-int.html#%E4%B8%80%E4%B8%AA%E7%9C%9F%E5%AE%9E%E5%9C%BA%E6%99%AF%E7%9A%84%E9%9C%80%E6%B1%82) <a href="#yi-ge-zhen-shi-chang-jing-de-xu-qiu" id="yi-ge-zhen-shi-chang-jing-de-xu-qiu"></a>

在实际场景中，从枚举到整数的转换有时还是非常需要的，例如你有一个枚举类型，然后需要从外面传入一个整数，用于控制后续的流程走向，此时就需要用整数去匹配相应的枚举(你也可以用整数匹配整数-, -，看看会不会被喷)。

既然有了需求，剩下的就是看看该如何实现，这篇文章的水远比你想象的要深，且看八仙过海各显神通。

### [C 语言的实现](https://course.rs/advance/into-types/enum-int.html#c-%E8%AF%AD%E8%A8%80%E7%9A%84%E5%AE%9E%E7%8E%B0) <a href="#c-yu-yan-de-shi-xian" id="c-yu-yan-de-shi-xian"></a>

对于 C 语言来说，万物皆邪恶，因此我们不讨论安全，只看实现，不得不说很简洁：

```C
#include <stdio.h>

enum atomic_number {
    HYDROGEN = 1,
    HELIUM = 2,
    // ...
    IRON = 26,
};

int main(void)
{
    enum atomic_number element = 26;

    if (element == IRON) {
        printf("Beware of Rust!\n");
    }

    return 0;
}

```

但是在 Rust 中，以下代码：

```rust
enum MyEnum {
    A = 1,
    B,
    C,
}

fn main() {
    // 将枚举转换成整数，顺利通过
    let x = MyEnum::C as i32;

    // 将整数转换为枚举，失败
    match x {
        MyEnum::A => {}
        MyEnum::B => {}
        MyEnum::C => {}
        _ => {}
    }
}

```

就会报错: `MyEnum::A => {} mismatched types, expected i32, found enum MyEnum`。

### [使用三方库](https://course.rs/advance/into-types/enum-int.html#%E4%BD%BF%E7%94%A8%E4%B8%89%E6%96%B9%E5%BA%93) <a href="#shi-yong-san-fang-ku" id="shi-yong-san-fang-ku"></a>

首先可以想到的肯定是三方库，毕竟 Rust 的生态目前已经发展的很不错，类似的需求总是有的，这里我们先使用`num-traits`和`num-derive`来试试。

在`Cargo.toml`中引入：

```toml
[dependencies]
num-traits = "0.2.14"
num-derive = "0.3.3"
```

代码如下:

```rust
use num_derive::FromPrimitive;
use num_traits::FromPrimitive;

#[derive(FromPrimitive)]
enum MyEnum {
    A = 1,
    B,
    C,
}

fn main() {
    let x = 2;

    match FromPrimitive::from_i32(x) {
        Some(MyEnum::A) => println!("Got A"),
        Some(MyEnum::B) => println!("Got B"),
        Some(MyEnum::C) => println!("Got C"),
        None            => println!("Couldn't convert {}", x),
    }
}

```

除了上面的库，还可以使用一个较新的库: [`num_enums`](https://github.com/illicitonion/num_enum)。

### [TryFrom + 宏](https://course.rs/advance/into-types/enum-int.html#tryfrom--%E5%AE%8F) <a href="#tryfrom-hong" id="tryfrom-hong"></a>

在 Rust 1.34 后，可以实现`TryFrom`特征来做转换:

```rust

#![allow(unused)]
fn main() {
use std::convert::TryFrom;

impl TryFrom<i32> for MyEnum {
    type Error = ();

    fn try_from(v: i32) -> Result<Self, Self::Error> {
        match v {
            x if x == MyEnum::A as i32 => Ok(MyEnum::A),
            x if x == MyEnum::B as i32 => Ok(MyEnum::B),
            x if x == MyEnum::C as i32 => Ok(MyEnum::C),
            _ => Err(()),
        }
    }
}
}

```

以上代码定义了从`i32`到`MyEnum`的转换，接着就可以使用`TryInto`来实现转换：

```rust
use std::convert::TryInto;

fn main() {
    let x = MyEnum::C as i32;

    match x.try_into() {
        Ok(MyEnum::A) => println!("a"),
        Ok(MyEnum::B) => println!("b"),
        Ok(MyEnum::C) => println!("c"),
        Err(_) => eprintln!("unknown number"),
    }
}

```

但是上面的代码有个问题，你需要为每个枚举成员都实现一个转换分支，非常麻烦。好在可以使用宏来简化，自动根据枚举的定义来实现`TryFrom`特征:

```rust

#![allow(unused)]
fn main() {
#[macro_export]
macro_rules! back_to_enum {
    ($(#[$meta:meta])* $vis:vis enum $name:ident {
        $($(#[$vmeta:meta])* $vname:ident $(= $val:expr)?,)*
    }) => {
        $(#[$meta])*
        $vis enum $name {
            $($(#[$vmeta])* $vname $(= $val)?,)*
        }

        impl std::convert::TryFrom<i32> for $name {
            type Error = ();

            fn try_from(v: i32) -> Result<Self, Self::Error> {
                match v {
                    $(x if x == $name::$vname as i32 => Ok($name::$vname),)*
                    _ => Err(()),
                }
            }
        }
    }
}

back_to_enum! {
    enum MyEnum {
        A = 1,
        B,
        C,
    }
}
}

```

### [邪恶之王 std::mem::transmute](https://course.rs/advance/into-types/enum-int.html#%E9%82%AA%E6%81%B6%E4%B9%8B%E7%8E%8B-stdmemtransmute) <a href="#xieezhi-wang-stdmemtransmute" id="xieezhi-wang-stdmemtransmute"></a>

**这个方法原则上并不推荐，但是有其存在的意义，如果要使用，你需要清晰的知道自己为什么使用**。

在之前的类型转换章节，我们提到过非常邪恶的[`transmute`转换](https://course.rs/advance/into-types/converse.html#%E5%8F%98%E5%BD%A2%E8%AE%B0transmutes)，其实，当你知道数值一定不会超过枚举的范围时(例如枚举成员对应 1，2，3，传入的整数也在这个范围内)，就可以使用这个方法完成变形。

> 最好使用#\[repr(..)]来控制底层类型的大小，免得本来需要 i32，结果传入 i64，最终内存无法对齐，产生奇怪的结果

```rust
#[repr(i32)]
enum MyEnum {
    A = 1, B, C
}

fn main() {
    let x = MyEnum::C;
    let y = x as i32;
    let z: MyEnum = unsafe { std::mem::transmute(y) };

    // match the enum that came from an int
    match z {
        MyEnum::A => { println!("Found A"); }
        MyEnum::B => { println!("Found B"); }
        MyEnum::C => { println!("Found C"); }
    }
}

```

既然是邪恶之王，当然得有真本事，无需标准库、也无需 unstable 的 Rust 版本，我们就完成了转换！awesome!??

### [总结](https://course.rs/advance/into-types/enum-int.html#%E6%80%BB%E7%BB%93) <a href="#zong-jie" id="zong-jie"></a>

本文列举了常用(其实差不多也是全部了，还有一个 unstable 特性没提到)的从整数转换为枚举的方式，推荐度按照出现的先后顺序递减。

但是推荐度最低，不代表它就没有出场的机会，只要使用边界清晰，一样可以大放光彩，例如最后的`transmute`函数.
