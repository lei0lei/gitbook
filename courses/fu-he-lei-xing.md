# 复合类型

本章的重点在复合类型上，顾名思义，复合类型是由其它类型组合而成的，最典型的就是结构体 `struct` 和枚举 `enum`。例如平面上的一个点 `point(x, y)`，它由两个数值类型的值 `x` 和 `y` 组合而来。我们无法单独去维护这两个数值，因为单独一个 `x` 或者 `y` 是含义不完整的，无法标识平面上的一个点，应该把它们看作一个整体去理解和处理。

来看一段代码，它使用我们之前学过的内容来构建文件操作：

```rust
// Some code
#![allow(unused_variables)]
type File = String;

fn open(f: &mut File) -> bool {
    true
}
fn close(f: &mut File) -> bool {
    true
}

#[allow(dead_code)]
fn read(f: &mut File, save_to: &mut Vec<u8>) -> ! {
    unimplemented!()
}

fn main() {
    let mut f1 = File::from("f1.txt");
    open(&mut f1);
    //read(&mut f1, &mut vec![]);
    close(&mut f1);
}
```

接下来我们的学习非常类似原型设计：有的方法只提供 API 接口，但是不提供具体实现。此外，有的变量在声明之后并未使用，因此在这个阶段我们需要排除一些编译器噪音（Rust 在编译的时候会扫描代码，变量声明后未使用会以 `warning` 警告的形式进行提示），引入 `#![allow(unused_variables)]` 属性标记，该标记会告诉编译器忽略未使用的变量，不要抛出 `warning` 警告，具体的常见编译器属性你可以在这里查阅：[编译器属性标记](https://course.rs/profiling/compiler/attributes.html)。

`read` 函数也非常有趣，它返回一个 `!` 类型，这个表明该函数是一个发散函数，不会返回任何值，包括 `()`。`unimplemented!()` 告诉编译器该函数尚未实现，`unimplemented!()` 标记通常意味着我们期望快速完成主要代码，回头再通过搜索这些标记来完成次要代码，类似的标记还有 `todo!()`，当代码执行到这种未实现的地方时，程序会直接报错。你可以反注释 `read(&mut f1, &mut vec![]);` 这行，然后再观察下结果。

同时，从代码设计角度来看，关于文件操作的类型和函数应该组织在一起，散落得到处都是，是难以管理和使用的。而且通过 `open(&mut f1)` 进行调用，也远没有使用 `f1.open()` 来调用好，这就体现出了只使用基本类型的局限性：**无法从更高的抽象层次去简化代码**。

接下来，我们将引入一个高级数据结构 —— 结构体 `struct`，来看看复合类型是怎样更好的解决这类问题。 开始之前，先来看看 Rust 的重点也是难点：字符串 `String` 和 `&str`。

## [字符串](https://course.rs/basic/compound-type/string-slice.html#%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#zi-fu-chuan" id="zi-fu-chuan"></a>

首先来看段很简单的代码：

```rust
// Some code
fn main() {
  let my_name = "Pascal";
  greet(my_name);
}

fn greet(name: String) {
  println!("Hello, {}!", name);
}
```

`greet` 函数接受一个字符串类型的 `name` 参数，然后打印到终端控制台中，非常好理解，你们猜猜，这段代码能否通过编译？

```
// Some code
error[E0308]: mismatched types
 --> src/main.rs:3:11
  |
3 |     greet(my_name);
  |           ^^^^^^^
  |           |
  |           expected struct `std::string::String`, found `&str`
  |           help: try using a conversion method: `my_name.to_string()`

error: aborting due to previous error

```

果然报错了，编译器提示 `greet` 函数需要一个 `String` 类型的字符串，却传入了一个 `&str` 类型的字符串。

### [切片(slice)](https://course.rs/basic/compound-type/string-slice.html#%E5%88%87%E7%89%87slice) <a href="#qie-pian-slice" id="qie-pian-slice"></a>

切片并不是 Rust 独有的概念，在 Go 语言中就非常流行，它允许你引用集合中部分连续的元素序列，而不是引用整个集合。

对于字符串而言，切片就是对 `String` 类型中某一部分的引用，它看起来像这样：

```rust
// Some code
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];

```

`hello` 没有引用整个 `String s`，而是引用了 `s` 的一部分内容，通过 `[0..5]` 的方式来指定。

这就是创建切片的语法，使用方括号包括的一个序列：**\[开始索引..终止索引]**，其中开始索引是切片中第一个元素的索引位置，而终止索引是最后一个元素后面的索引位置。换句话说，这是一个 `右半开区间`（或称为左闭右开区间）——指的是在区间的左端点是包含在内的，而右端点是不包含在内的。在切片数据结构内部会保存开始的位置和切片的长度，其中长度是通过 `终止索引` - `开始索引` 的方式计算得来的。

对于 `let world = &s[6..11];` 来说，`world` 是一个切片，该切片的指针指向 `s` 的第 7 个字节(索引从 0 开始, 6 是第 7 个字节)，且该切片的长度是 `5` 个字节。

<figure><img src="https://pic1.zhimg.com/80/v2-69da917741b2c610732d8526a9cc86f5_1440w.jpg" alt="" width="375"><figcaption></figcaption></figure>

在使用 Rust 的 `..` [range 序列](https://course.rs/basic/base-type/numbers.html#%E5%BA%8F%E5%88%97range)语法时，如果你想从索引 0 开始，可以使用如下的方式，这两个是等效的：

```rust
// Some code
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];

```

同样的，如果你的切片想要包含 `String` 的最后一个字节，则可以这样使用：

```rust
// Some code
let s = String::from("hello");

let len = s.len();

let slice = &s[4..len];
let slice = &s[4..];

```

你也可以截取完整的 `String` 切片：

```rust
// Some code
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];

```

在对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置，也就是 UTF-8 字符的边界，例如中文在 UTF-8 中占用三个字节，下面的代码就会崩溃：

```rust
 let s = "中国人"; let a = &s[0..2]; println!("{}",a);
```

因为我们只取 `s` 字符串的前两个字节，但是本例中每个汉字占用三个字节，因此没有落在边界处，也就是连 `中` 字都取不完整，此时程序会直接崩溃退出，如果改成 `&s[0..3]`，则可以正常通过编译。 因此，当你需要对字符串做切片索引操作时，需要格外小心这一点，关于该如何操作 UTF-8 字符串，参见[这里](https://course.rs/basic/compound-type/string-slice.html#%E6%93%8D%E4%BD%9C-utf-8-%E5%AD%97%E7%AC%A6%E4%B8%B2)。

字符串切片的类型标识是 `&str`，因此我们可以这样声明一个函数，输入 `String` 类型，返回它的切片：`fn first_word(s: &String) -> &str` 。

有了切片就可以写出这样的代码：

```rust
// Some code
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
fn first_word(s: &String) -> &str {
    &s[..1]
}
```

编译器报错如下：

```
// Some code
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here

```

回忆一下借用的规则：当我们已经有了可变借用时，就无法再拥有不可变的借用。因为 `clear` 需要清空改变 `String`，因此它需要一个可变借用（利用 VSCode 可以看到该方法的声明是 `pub fn clear(&mut self)` ，参数是对自身的可变借用 ）；而之后的 `println!` 又使用了不可变借用，也就是在 `s.clear()` 处可变借用与不可变借用试图同时生效，因此编译无法通过。

从上述代码可以看出，Rust 不仅让我们的 `API` 更加容易使用，而且也在编译期就消除了大量错误！

[**其它切片**](https://course.rs/basic/compound-type/string-slice.html#%E5%85%B6%E5%AE%83%E5%88%87%E7%89%87)

因为切片是对集合的部分引用，因此不仅仅字符串有切片，其它集合类型也有，例如数组：

```rust
// Some code
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);

```

该数组切片的类型是 `&[i32]`，数组切片和字符串切片的工作方式是一样的，例如持有一个引用指向原始数组的某个元素和长度。

### [什么是字符串?](https://course.rs/basic/compound-type/string-slice.html#%E4%BB%80%E4%B9%88%E6%98%AF%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#shen-me-shi-zi-fu-chuan" id="shen-me-shi-zi-fu-chuan"></a>

### [String 与 \&str 的转换](https://course.rs/basic/compound-type/string-slice.html#string-%E4%B8%8E-str-%E7%9A%84%E8%BD%AC%E6%8D%A2) <a href="#string-yu-str-de-zhuan-huan" id="string-yu-str-de-zhuan-huan"></a>

### [字符串索引](https://course.rs/basic/compound-type/string-slice.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B4%A2%E5%BC%95) <a href="#zi-fu-chuan-suo-yin" id="zi-fu-chuan-suo-yin"></a>

### [字符串切片](https://course.rs/basic/compound-type/string-slice.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%88%87%E7%89%87) <a href="#zi-fu-chuan-qie-pian" id="zi-fu-chuan-qie-pian"></a>

### [操作字符串](https://course.rs/basic/compound-type/string-slice.html#%E6%93%8D%E4%BD%9C%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#cao-zuo-zi-fu-chuan" id="cao-zuo-zi-fu-chuan"></a>

### [字符串转义](https://course.rs/basic/compound-type/string-slice.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E8%BD%AC%E4%B9%89) <a href="#zi-fu-chuan-zhuan-yi" id="zi-fu-chuan-zhuan-yi"></a>

### [操作 UTF-8 字符串](https://course.rs/basic/compound-type/string-slice.html#%E6%93%8D%E4%BD%9C-utf-8-%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#cao-zuo-utf8-zi-fu-chuan" id="cao-zuo-utf8-zi-fu-chuan"></a>

### [字符串深度剖析](https://course.rs/basic/compound-type/string-slice.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%B7%B1%E5%BA%A6%E5%89%96%E6%9E%90) <a href="#zi-fu-chuan-shen-du-pou-xi" id="zi-fu-chuan-shen-du-pou-xi"></a>



## [元组](https://course.rs/basic/compound-type/tuple.html#%E5%85%83%E7%BB%84) <a href="#yuan-zu" id="yuan-zu"></a>



## [结构体](https://course.rs/basic/compound-type/struct.html#%E7%BB%93%E6%9E%84%E4%BD%93) <a href="#jie-gou-ti" id="jie-gou-ti"></a>



## [枚举](https://course.rs/basic/compound-type/enum.html#%E6%9E%9A%E4%B8%BE) <a href="#mei-ju" id="mei-ju"></a>





## [数组](https://course.rs/basic/compound-type/array.html#%E6%95%B0%E7%BB%84) <a href="#shu-zu" id="shu-zu"></a>
