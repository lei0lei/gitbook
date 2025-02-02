# 集合



## [常见集合](https://kaisery.github.io/trpl-zh-cn/ch08-00-common-collections.html#%E5%B8%B8%E8%A7%81%E9%9B%86%E5%90%88) <a href="#chang-jian-ji-he" id="chang-jian-ji-he"></a>

Rust 标准库中包含一系列被称为 **集合**（_collections_）的非常有用的数据结构。大部分其他数据类型都代表一个特定的值，不过集合可以包含多个值。不同于内建的数组和元组类型，<mark style="color:red;">这些集合指向的数据是储存在堆上的</mark>，这意味着数据的数量不必在编译时就已知，并且还可以随着程序的运行增长或缩小。每种集合都有着不同功能和成本，而根据当前情况选择合适的集合，这是一项应当逐渐掌握的技能。在这一章里，我们将详细的了解三个在 Rust 程序中被广泛使用的集合：

* _vector_ 允许我们一个挨着一个地储存一系列数量可变的值
* **字符串**（_string_）是字符的集合。我们之前见过 `String` 类型，不过在本章我们将深入了解。
* **哈希 map**（_hash map_）允许我们将值与一个特定的键（key）相关联。这是一个叫做 _map_ 的更通用的数据结构的特定实现。

对于标准库提供的其他类型的集合，请查看[文档](https://doc.rust-lang.org/std/collections/index.html)。

我们将讨论如何创建和更新 vector、字符串和哈希 map，以及它们有什么特别之处。

## [使用 Vector 储存列表](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E4%BD%BF%E7%94%A8-vector-%E5%82%A8%E5%AD%98%E5%88%97%E8%A1%A8) <a href="#shi-yong-vector-chu-cun-lie-biao" id="shi-yong-vector-chu-cun-lie-biao"></a>

我们要讲到的第一个类型是 `Vec<T>`，也被称为 _vector_。vector 允许我们在一个单独的数据结构中储存多于一个的值，它在内存中彼此相邻地排列所有的值。vector 只能储存相同类型的值。它们在拥有一系列项的场景下非常实用，例如文件中的文本行或是购物车中商品的价格。

### [新建 vector](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E6%96%B0%E5%BB%BA-vector) <a href="#xin-jian-vector" id="xin-jian-vector"></a>

为了创建一个新的空 vector，可以调用 `Vec::new` 函数，如示例 8-1 所示：

```rust
    let v: Vec<i32> = Vec::new();
```

示例 8-1：新建一个空的 vector 来储存 `i32` 类型的值

<mark style="color:red;">注意这里我们增加了一个类型注解</mark>。因为没有向这个 vector 中插入任何值，Rust 并不知道我们想要储存什么类型的元素。这是一个非常重要的点。vector 是用泛型实现的，第十章会涉及到如何对你自己的类型使用它们。现在，所有你需要知道的就是 `Vec<T>` 是一个由标准库提供的类型，它可以存放任何类型，而当 `Vec` 存放某个特定类型时，那个类型位于尖括号中。在示例 8-1 中，我们告诉 Rust `v` 这个 `Vec<T>` 将存放 `i32` 类型的元素。

<mark style="color:red;">通常，我们会用初始值来创建一个</mark> <mark style="color:red;"></mark><mark style="color:red;">`Vec<T>`</mark> <mark style="color:red;"></mark><mark style="color:red;">而 Rust 会推断出储存值的类型，所以很少会需要这些类型注解。</mark>为了方便 Rust 提供了 `vec!` 宏，这个宏会根据我们提供的值来创建一个新的 vector。示例 8-2 新建一个拥有值 `1`、`2` 和 `3` 的 `Vec<i32>`。推断为 `i32` 是因为这是默认整型类型，第三章的 [“数据类型”](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B) 讨论过：

```rust
    let v = vec![1, 2, 3];
```

示例 8-2：新建一个包含初值的 vector

因为我们提供了 `i32` 类型的初始值，Rust 可以推断出 `v` 的类型是 `Vec<i32>`，因此类型注解就不是必须的。接下来让我们看看如何修改一个 vector。

### [更新 vector](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E6%9B%B4%E6%96%B0-vector) <a href="#geng-xin-vector" id="geng-xin-vector"></a>

对于新建一个 vector 并向其增加元素，可以使用 `push` 方法，如示例 8-3 所示：

```rust
fn main() {
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
}

```

示例 8-3：使用 `push` 方法向 vector 增加值

如第三章中讨论的任何变量一样，如果想要能够改变它的值，必须使用 `mut` 关键字使其可变。放入其中的所有值都是 `i32` 类型的，而且 Rust 也根据数据做出如此判断，所以不需要 `Vec<i32>` 注解。

### [读取 vector 的元素](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E8%AF%BB%E5%8F%96-vector-%E7%9A%84%E5%85%83%E7%B4%A0) <a href="#du-qu-vector-de-yuan-su" id="du-qu-vector-de-yuan-su"></a>

有两种方法引用 vector 中储存的值：通过索引或使用 `get` 方法。在接下来的示例中，为了更加清楚的说明，我们已经标注了这些函数返回的值的类型。

示例 8-4 展示了访问 vector 中一个值的两种方式，索引语法或者 `get` 方法：

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
}

```

列表 8-4：使用索引语法或 `get` 方法来访问 vector 中的项

这里有几个细节需要注意。我们使用索引值 `2` 来获取第三个元素，因为索引是从数字 0 开始的。使用 `&` 和 `[]` 会得到一个索引位置元素的引用。<mark style="color:red;">当使用索引作为参数调用</mark> <mark style="color:red;"></mark><mark style="color:red;">`get`</mark> <mark style="color:red;"></mark><mark style="color:red;">方法时，会得到一个可以用于</mark> <mark style="color:red;"></mark><mark style="color:red;">`match`</mark> <mark style="color:red;"></mark><mark style="color:red;">的</mark> <mark style="color:red;"></mark><mark style="color:red;">`Option<&T>`</mark>。

Rust 提供了两种引用元素的方法的原因是当尝试使用现有元素范围之外的索引值时可以选择让程序如何运行。举个例子，让我们看看使用这个技术，尝试在当有一个 5 个元素的 vector 接着访问索引 100 位置的元素会发生什么，如示例 8-5 所示：

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
}

```

示例 8-5：尝试访问一个包含 5 个元素的 vector 的索引 100 处的元素

当运行这段代码，你会发现对于第一个 `[]` 方法，当引用一个不存在的元素时 Rust 会造成 panic。这个方法更适合当程序认为尝试访问超过 vector 结尾的元素是一个严重错误的情况，这时应该使程序崩溃。

当 `get` 方法被传递了一个数组外的索引时，它不会 panic 而是返回 `None`。当偶尔出现超过 vector 范围的访问属于正常情况的时候可以考虑使用它。接着你的代码可以有处理 `Some(&element)` 或 `None` 的逻辑，如第六章讨论的那样。例如，索引可能来源于用户输入的数字。如果它们不慎输入了一个过大的数字那么程序就会得到 `None` 值，你可以告诉用户当前 vector 元素的数量并再请求它们输入一个有效的值。这就比因为输入错误而使程序崩溃要友好的多！

一旦程序获取了一个有效的引用，借用检查器将会执行所有权和借用规则（第四章讲到）来确保 vector 内容的这个引用和任何其他引用保持有效。回忆一下不能在相同作用域中同时存在可变和不可变引用的规则。这个规则适用于示例 8-6，当我们获取了 vector 的第一个元素的不可变引用并尝试在 vector 末尾增加一个元素的时候，如果尝试在函数的后面引用这个元素是行不通的：

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
```

示例 8-6：在拥有 vector 中项的引用的同时向其增加一个元素

编译会给出这个错误：

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {first}");
  |                                      ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```

<mark style="color:red;">示例 8-6 中的代码看起来应该能够运行：为什么第一个元素的引用会关心 vector 结尾的变化？不能这么做的原因是由于 vector 的工作方式：在 vector 的结尾增加新元素时，在没有足够空间将所有元素依次相邻存放的情况下，可能会要求分配新内存并将老的元素拷贝到新的空间中。这时，第一个元素的引用就指向了被释放的内存。借用规则阻止程序陷入这种状况</mark>。

> 注意：关于 `Vec<T>` 类型的更多实现细节，请查看 [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/vec/vec.html)

### [遍历 vector 中的元素](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E9%81%8D%E5%8E%86-vector-%E4%B8%AD%E7%9A%84%E5%85%83%E7%B4%A0) <a href="#bian-li-vector-zhong-de-yuan-su" id="bian-li-vector-zhong-de-yuan-su"></a>

如果想要依次访问 vector 中的每一个元素，我们可以遍历其所有的元素而无需通过索引一次一个的访问。示例 8-7 展示了如何使用 `for` 循环来获取 `i32` 值的 vector 中的每一个元素的不可变引用并将其打印：

```rust
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
}

```

示例 8-7：通过 `for` 循环遍历 vector 的元素并打印

我们也可以遍历可变 vector 的每一个元素的可变引用以便能改变他们。示例 8-8 中的 `for` 循环会给每一个元素加 `50`：

```rust
fn main() {
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}

```

示例 8-8：遍历 vector 中元素的可变引用

为了修改可变引用所指向的值，在使用 `+=` 运算符之前必须使用解引用运算符（`*`）获取 `i` 中的值。第十五章的 [“通过解引用运算符追踪指针的值”](https://kaisery.github.io/trpl-zh-cn/ch15-02-deref.html#%E9%80%9A%E8%BF%87%E8%A7%A3%E5%BC%95%E7%94%A8%E8%BF%90%E7%AE%97%E7%AC%A6%E8%BF%BD%E8%B8%AA%E6%8C%87%E9%92%88%E7%9A%84%E5%80%BC) 部分会详细介绍解引用运算符。

因为借用检查器的规则，无论可变还是不可变地遍历一个 vector 都是安全。如果尝试在示例 8-7 和 示例 8-8 的 `for` 循环体内插入或删除项，都会得到一个类似示例 8-6 代码中类似的编译错误。`for` 循环中获取的 vector 引用阻止了同时对 vector 整体的修改。

### [使用枚举来储存多种类型](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E4%BD%BF%E7%94%A8%E6%9E%9A%E4%B8%BE%E6%9D%A5%E5%82%A8%E5%AD%98%E5%A4%9A%E7%A7%8D%E7%B1%BB%E5%9E%8B) <a href="#shi-yong-mei-ju-lai-chu-cun-duo-zhong-lei-xing" id="shi-yong-mei-ju-lai-chu-cun-duo-zhong-lei-xing"></a>

vector 只能储存相同类型的值。这是很不方便的；绝对会有需要储存一系列不同类型的值的用例。幸运的是，枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举！

例如，假如我们想要从电子表格的一行中获取值，而这一行的有些列包含数字，有些包含浮点值，还有些是字符串。我们可以定义一个枚举，其成员会存放这些不同类型的值，同时所有这些枚举成员都会被当作相同类型：那个枚举的类型。接着可以创建一个储存枚举值的 vector，这样最终就能够储存不同类型的值了。示例 8-9 展示了其用例：

```rust
fn main() {
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}

```

示例 8-9：定义一个枚举，以便能在 vector 中存放不同类型的数据

Rust 在编译时就必须准确的知道 vector 中类型的原因在于它需要知道储存每个元素到底需要多少内存。第二个好处是可以准确的知道这个 vector 中允许什么类型。如果 Rust 允许 vector 存放任意类型，那么当对 vector 元素执行操作时一个或多个类型的值就有可能会造成错误。使用枚举外加 `match` 意味着 Rust 能在编译时就保证总是会处理所有可能的情况，正如第六章讲到的那样。

<mark style="color:red;">如果在编写程序时不能确切无遗地知道运行时会储存进 vector 的所有类型，枚举技术就行不通了。相反，你可以使用 trait 对象</mark>，第十七章会讲到它。

现在我们了解了一些使用 vector 的最常见的方式，请一定去看看标准库中 `Vec` 定义的很多其他实用方法的 [API 文档](https://doc.rust-lang.org/std/vec/struct.Vec.html)。例如，除了 `push` 之外还有一个 `pop` 方法，它会移除并返回 vector 的最后一个元素。

### [丢弃 vector 时也会丢弃其所有元素](https://kaisery.github.io/trpl-zh-cn/ch08-01-vectors.html#%E4%B8%A2%E5%BC%83-vector-%E6%97%B6%E4%B9%9F%E4%BC%9A%E4%B8%A2%E5%BC%83%E5%85%B6%E6%89%80%E6%9C%89%E5%85%83%E7%B4%A0) <a href="#diu-qi-vector-shi-ye-hui-diu-qi-qi-suo-you-yuan-su" id="diu-qi-vector-shi-ye-hui-diu-qi-qi-suo-you-yuan-su"></a>

类似于任何其他的 `struct`，vector 在其离开作用域时会被释放，如示例 8-4 所标注的：

```rust
fn main() {
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
}

```

示例 8-10：展示 vector 和其元素于何处被丢弃

当 vector 被丢弃时，所有其内容也会被丢弃，这意味着这里它包含的整数将被清理。借用检查器确保了任何 vector 中内容的引用仅在 vector 本身有效时才可用。

让我们继续下一个集合类型：`String`！

## [使用字符串储存 UTF-8 编码的文本](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E4%BD%BF%E7%94%A8%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%82%A8%E5%AD%98-utf-8-%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC) <a href="#shi-yong-zi-fu-chuan-chu-cun-utf8-bian-ma-de-wen-ben" id="shi-yong-zi-fu-chuan-chu-cun-utf8-bian-ma-de-wen-ben"></a>

第四章已经讲过一些字符串的内容，不过现在让我们更深入地了解它。字符串是新晋 Rustacean 们通常会被困住的领域，这是由于三方面理由的结合：Rust 倾向于确保暴露出可能的错误，字符串是比很多程序员所想象的要更为复杂的数据结构，以及 UTF-8。所有这些要素结合起来对于来自其他语言背景的程序员就可能显得很困难了。

在集合章节中讨论字符串的原因是，字符串就是作为字节的集合外加一些方法实现的，当这些字节被解释为文本时，这些方法提供了实用的功能。在这一部分，我们会讲到 `String` 中那些任何集合类型都有的操作，比如创建、更新和读取。也会讨论 `String` 与其他集合不一样的地方，例如索引 `String` 是很复杂的，由于人和计算机理解 `String` 数据方式的不同。

### [什么是字符串？](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E4%BB%80%E4%B9%88%E6%98%AF%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#shen-me-shi-zi-fu-chuan" id="shen-me-shi-zi-fu-chuan"></a>

在开始深入这些方面之前，我们需要讨论一下术语 **字符串** 的具体意义。<mark style="color:red;">Rust 的核心语言中只有一种字符串类型：字符串</mark> slice `str`，它<mark style="color:red;">通常以被借用的形式出现</mark>，`&str`。第四章讲到了 **字符串 slices**：它们是一些对储存在别处的 UTF-8 编码字符串数据的引用。举例来说，由于字符串字面值被储存在程序的二进制输出中，因此字符串字面值也是字符串 slices。

### [新建字符串](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E6%96%B0%E5%BB%BA%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#xin-jian-zi-fu-chuan" id="xin-jian-zi-fu-chuan"></a>

很多 `Vec` 可用的操作在 `String` 中同样可用，事实上 `String` 被实现为一个带有一些额外保证、限制和功能的字节 vector 的封装。其中一个同样作用于 `Vec<T>` 和 `String` 函数的例子是用来新建一个实例的 `new` 函数，如示例 8-11 所示。

```rust
    let mut s = String::new();
```

示例 8-11：新建一个空的 `String`

这新建了一个叫做 `s` 的空的字符串，接着我们可以向其中装载数据。通常字符串会有初始数据，因为我们希望一开始就有这个字符串。为此，可以使用 `to_string` 方法，它能用于任何实现了 `Display` trait 的类型，比如字符串字面值。示例 8-12 展示了两个例子。

```rust
fn main() {
    let data = "initial contents";

    let s = data.to_string();

    // 该方法也可直接用于字符串字面值：
    let s = "initial contents".to_string();
}

```

示例 8-12：使用 `to_string` 方法从字符串字面值创建 `String`

这些代码会创建包含 `initial contents` 的字符串。

也可以使用 `String::from` 函数来从字符串字面值创建 `String`。示例 8-13 中的代码等同于使用 `to_string`。

```rust
    let s = String::from("initial contents");
```

示例 8-13：使用 `String::from` 函数从字符串字面值创建 `String`

因为字符串应用广泛，这里有很多不同的用于字符串的通用 API 可供选择。其中一些可能看起来多余，不过都有其用武之地！在这个例子中，`String::from` 和 `.to_string` 最终做了完全相同的工作，所以如何选择就是代码风格与可读性的问题了。

记住字符串是 UTF-8 编码的，所以可以包含任何可以正确编码的数据，如示例 8-14 所示。

```rust
fn main() {
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
}

```

示例 8-14：在字符串中储存不同语言的问候语

所有这些都是有效的 `String` 值。

### [更新字符串](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E6%9B%B4%E6%96%B0%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#geng-xin-zi-fu-chuan" id="geng-xin-zi-fu-chuan"></a>

`String` 的大小可以增加，其内容也可以改变，就像可以放入更多数据来改变 `Vec` 的内容一样。另外，可以方便的使用 `+` 运算符或 `format!` 宏来拼接 `String` 值。

#### [**使用 `push_str` 和 `push` 附加字符串**](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E4%BD%BF%E7%94%A8-push_str-%E5%92%8C-push-%E9%99%84%E5%8A%A0%E5%AD%97%E7%AC%A6%E4%B8%B2)

可以通过 `push_str` 方法来附加字符串 slice，从而使 `String` 变长，如示例 8-15 所示。

```rust
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar");
}

```

示例 8-15：使用 `push_str` 方法向 `String` 附加字符串 slice

执行这两行代码之后，`s` 将会包含 `foobar`。`push_str` 方法采用字符串 slice，因为我们并不需要获取参数的所有权。例如，示例 8-16 中我们希望在将 `s2` 的内容附加到 `s1` 之后还能使用它。

```rust
fn main() {
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
}

```

示例 8-16：将字符串 slice 的内容附加到 `String` 后使用它

如果 `push_str` 方法获取了 `s2` 的所有权，就不能在最后一行打印出其值了。好在代码如我们期望那样工作！

`push` 方法被定义为获取一个单独的字符作为参数，并附加到 `String` 中。示例 8-17 展示了使用 `push` 方法将字母 "l" 加入 `String` 的代码。

```rust
fn main() {
    let mut s = String::from("lo");
    s.push('l');
}

```

示例 8-17：使用 `push` 将一个字符加入 `String` 值中

执行这些代码之后，`s` 将会包含 “lol”。

#### [**使用 `+` 运算符或 `format!` 宏拼接字符串**](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E4%BD%BF%E7%94%A8--%E8%BF%90%E7%AE%97%E7%AC%A6%E6%88%96-format-%E5%AE%8F%E6%8B%BC%E6%8E%A5%E5%AD%97%E7%AC%A6%E4%B8%B2)

通常你会希望将两个已知的字符串合并在一起。一种办法是像这样使用 `+` 运算符，如示例 8-18 所示。

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用
}

```

示例 8-18：使用 `+` 运算符将两个 `String` 值合并到一个新的 `String` 值中

执行完这些代码之后，字符串 `s3` 将会包含 `Hello, world!`。<mark style="color:red;">`s1`</mark> <mark style="color:red;"></mark><mark style="color:red;">在相加后不再有效的原因，和使用</mark> <mark style="color:red;"></mark><mark style="color:red;">`s2`</mark> <mark style="color:red;"></mark><mark style="color:red;">的引用的原因，与使用</mark> <mark style="color:red;"></mark><mark style="color:red;">`+`</mark> <mark style="color:red;"></mark><mark style="color:red;">运算符时调用的函数签名有关。</mark><mark style="color:red;">`+`</mark> <mark style="color:red;"></mark><mark style="color:red;">运算符使用了</mark> <mark style="color:red;"></mark><mark style="color:red;">`add`</mark> <mark style="color:red;"></mark><mark style="color:red;">函数，这个函数签名看起来像这样：</mark>

```rust
fn add(self, s: &str) -> String {
```

在标准库中你会发现，`add` 的定义使用了泛型和关联类型。在这里我们替换为了具体类型，这也正是当使用 `String` 值调用这个方法会发生的。第十章会讨论泛型。这个签名提供了理解 `+` 运算那微妙部分的线索。

首先，`s2` 使用了 `&`，意味着我们使用第二个字符串的 **引用** 与第一个字符串相加。<mark style="color:red;">这是因为</mark> <mark style="color:red;"></mark><mark style="color:red;">`add`</mark> <mark style="color:red;"></mark><mark style="color:red;">函数的</mark> <mark style="color:red;"></mark><mark style="color:red;">`s`</mark> <mark style="color:red;"></mark><mark style="color:red;">参数：只能将</mark> <mark style="color:red;"></mark><mark style="color:red;">`&str`</mark> <mark style="color:red;"></mark><mark style="color:red;">和</mark> <mark style="color:red;"></mark><mark style="color:red;">`String`</mark> <mark style="color:red;"></mark><mark style="color:red;">相加，不能将两个</mark> <mark style="color:red;"></mark><mark style="color:red;">`String`</mark> <mark style="color:red;"></mark><mark style="color:red;">值相加</mark>。不过等一下 —— 正如 `add` 的第二个参数所指定的，`&s2` 的类型是 `&String` 而不是 `&str`。那么为什么示例 8-18 还能编译呢？

之所以能够在 `add` 调用中使用 `&s2` 是因为 <mark style="color:red;">`&String`</mark> <mark style="color:red;"></mark><mark style="color:red;">可以被</mark> <mark style="color:red;"></mark><mark style="color:red;">**强转**</mark><mark style="color:red;">（</mark>_<mark style="color:red;">coerced</mark>_<mark style="color:red;">）成</mark> <mark style="color:red;"></mark><mark style="color:red;">`&str`</mark>。当`add`函数被调用时，Rust 使用了一个被称为 **Deref 强制转换**（_deref coercion_）的技术，你可以将其理解为它把 `&s2` 变成了 `&s2[..]`。第十五章会更深入的讨论 Deref 强制转换。因为 `add` 没有获取参数的所有权，所以 `s2` 在这个操作后仍然是有效的 `String`。

其次，可以发现签名中 `add` 获取了 `self` 的所有权，因为 `self` **没有** 使用 `&`。这意味着示例 8-18 中的 `s1` 的所有权将被移动到 `add` 调用中，之后就不再有效。所以虽然 `let s3 = s1 + &s2;` 看起来就像它会复制两个字符串并创建一个新的字符串，而实际上这个语句会获取 `s1` 的所有权，附加上从 `s2` 中拷贝的内容，并返回结果的所有权。换句话说，它看起来好像生成了很多拷贝，不过实际上并没有：这个实现比拷贝要更高效。

如果想要级联多个字符串，`+` 的行为就显得笨重了：

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
}

```

这时 `s` 的内容会是 “tic-tac-toe”。在有这么多 `+` 和 `"` 字符的情况下，很难理解具体发生了什么。对于更为复杂的字符串链接，可以使用 `format!` 宏：

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
}

```

这些代码也会将 `s` 设置为 “tic-tac-toe”。`format!` 与 `println!` 的工作原理相同，不过不同于将输出打印到屏幕上，它返回一个带有结果内容的 `String`。这个版本就好理解的多，<mark style="color:red;">宏</mark> <mark style="color:red;"></mark><mark style="color:red;">`format!`</mark> <mark style="color:red;"></mark><mark style="color:red;">生成的代码使用引用所以不会获取任何参数的所有权。</mark>

### [索引字符串](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E7%B4%A2%E5%BC%95%E5%AD%97%E7%AC%A6%E4%B8%B2) <a href="#suo-yin-zi-fu-chuan" id="suo-yin-zi-fu-chuan"></a>

在很多语言中，通过索引来引用字符串中的单独字符是有效且常见的操作。然而在 Rust 中，如果你尝试使用索引语法访问 `String` 的一部分，会出现一个错误。考虑一下如示例 8-19 中所示的无效代码。

```rust
    let s1 = String::from("hello");
    let h = s1[0];
```

示例 8-19：尝试对字符串使用索引语法

这段代码会导致如下错误：

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`
  = help: the following other types implement trait `Index<Idx>`:
            <String as Index<RangeFrom<usize>>>
            <String as Index<RangeFull>>
            <String as Index<RangeInclusive<usize>>>
            <String as Index<RangeTo<usize>>>
            <String as Index<RangeToInclusive<usize>>>
            <String as Index<std::ops::Range<usize>>>
            <str as Index<I>>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error
```

错误和提示说明了全部问题：Rust 的字符串不支持索引。那么接下来的问题是，为什么不支持呢？为了回答这个问题，我们必须先聊一聊 Rust 是如何在内存中储存字符串的。

#### [**内部表现**](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E5%86%85%E9%83%A8%E8%A1%A8%E7%8E%B0)

<mark style="color:red;">`String`</mark> <mark style="color:red;"></mark><mark style="color:red;">是一个</mark> <mark style="color:red;"></mark><mark style="color:red;">`Vec<u8>`</mark> <mark style="color:red;"></mark><mark style="color:red;">的封装</mark>。让我们看看示例 8-14 中一些正确编码的字符串的例子。首先是这一个：

```rust
    let hello = String::from("Hola");
```

在这里，`len` 的值是 4，这意味着储存字符串 “Hola” 的 `Vec` 的长度是四个字节：这里每一个字母的 UTF-8 编码都占用一个字节。那下面这个例子又如何呢？（注意这个字符串中的首字母是西里尔字母的 Ze 而不是阿拉伯数字 3。）

```rust
    let hello = String::from("Здравствуйте");
```

当问及这个字符是多长的时候有人可能会说是 12。然而，Rust 的回答是 24。这是使用 UTF-8 编码 “Здравствуйте” 所需要的字节数，这是因为每个 Unicode 标量值需要两个字节存储。<mark style="color:red;">因此一个字符串字节值的索引并不总是对应一个有效的 Unicode 标量值</mark>。作为演示，考虑如下无效的 Rust 代码：

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

我们已经知道 `answer`的第一个字符不是数字`3`。当使用 UTF-8 编码时，第一个字节（西里尔字母的 Ze）`З` 的编码是 `208`，第二个是 `151`，所以 `answer` 实际上应该是 `208`，不过 `208` 自身并不是一个有效的字母。返回 `208` 可不是一个请求字符串第一个字母的人所希望看到的，不过它是 Rust 在字节索引 0 位置所能提供的唯一数据。<mark style="color:red;">用户通常不会想要一个字节值被返回。即使这个字符串只有拉丁字母，如果</mark> <mark style="color:red;"></mark><mark style="color:red;">`&"hello"[0]`</mark> <mark style="color:red;"></mark><mark style="color:red;">是返回字节值的有效代码，它也会返回</mark> <mark style="color:red;"></mark><mark style="color:red;">`104`</mark> <mark style="color:red;"></mark><mark style="color:red;">而不是</mark> <mark style="color:red;"></mark><mark style="color:red;">`h`</mark>。

为了避免返回意外的值并造成不能立刻发现的 bug，Rust 根本不会编译这些代码，并在开发过程中及早杜绝了误会的发生。

#### [**字节、标量值和字形簇！天呐！**](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E5%AD%97%E8%8A%82%E6%A0%87%E9%87%8F%E5%80%BC%E5%92%8C%E5%AD%97%E5%BD%A2%E7%B0%87%E5%A4%A9%E5%91%90)

这引起了关于 UTF-8 的另外一个问题：从 Rust 的角度来讲，事实上有三种相关方式可以理解字符串：字节、标量值和字形簇（最接近人们眼中 **字母** 的概念）。

比如这个用梵文书写的印度语单词 “नमस्ते”，最终它储存在 vector 中的 `u8` 值看起来像这样：

```
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

这里有 18 个字节，也就是计算机最终会储存的数据。如果从 Unicode 标量值的角度理解它们，也就像 Rust 的 `char` 类型那样，这些字节看起来像这样：

```
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char`，不过第四个和第六个都不是字母，它们是发音符号本身并没有任何意义。最后，如果以字形簇的角度理解，就会得到人们所说的构成这个单词的四个字母：

```
["न", "म", "स्", "ते"]
```

Rust 提供了多种不同的方式来解释计算机储存的原始字符串数据，这样程序就可以选择它需要的表现方式，而无所谓是何种人类语言。

最后一个 Rust 不允许使用索引获取 `String` 字符的原因是，<mark style="color:red;">索引操作预期总是需要常数时间（O(1)）</mark>。但是对于 `String` 不可能保证这样的性能，因为 Rust 必须从开头到索引位置遍历来确定有多少有效的字符。

### [字符串 slice](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E5%AD%97%E7%AC%A6%E4%B8%B2-slice) <a href="#zi-fu-chuan-slice" id="zi-fu-chuan-slice"></a>

索引字符串通常是一个坏点子，因为字符串索引应该返回的类型是不明确的：字节值、字符、字形簇或者字符串 slice。因此，如果你真的希望使用索引创建字符串 slice 时，Rust 会要求你更明确一些。为了更明确索引并表明你需要一个字符串 slice，相比使用 `[]` 和单个值的索引，可以使用 `[]` 和一个 range 来创建含特定字节的字符串 slice：

```rust
#![allow(unused)]
fn main() {
let hello = "Здравствуйте";

let s = &hello[0..4];
}

```

这里，`s` 会是一个 `&str`，它包含字符串的头四个字节。早些时候，我们提到了这些字母都是两个字节长的，所以这意味着 `s` 将会是 “Зд”。

如果获取 `&hello[0..1]` 会发生什么呢？答案是：Rust 在运行时会 panic，就跟访问 vector 中的无效索引时一样：

```console
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/main.rs:4:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

你应该小心谨慎地使用这个操作，因为这么做可能会使你的程序崩溃。

### [遍历字符串的方法](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E9%81%8D%E5%8E%86%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%9A%84%E6%96%B9%E6%B3%95) <a href="#bian-li-zi-fu-chuan-de-fang-fa" id="bian-li-zi-fu-chuan-de-fang-fa"></a>

操作字符串每一部分的最好的方法是明确表示需要字符还是字节。对于单独的 Unicode 标量值使用 <mark style="color:red;">`chars`</mark> <mark style="color:red;"></mark><mark style="color:red;">方法</mark>。对 “Зд” 调用 `chars` 方法会将其分开并返回两个 `char` 类型的值，接着就可以遍历其结果来访问每一个元素了：

```rust
#![allow(unused)]
fn main() {
for c in "Зд".chars() {
    println!("{c}");
}
}

```

这些代码会打印出如下内容：

```
З
д
```

另外 `bytes` 方法返回每一个原始字节，这可能会适合你的使用场景：

```rust
#![allow(unused)]
fn main() {
for b in "Зд".bytes() {
    println!("{b}");
}
}

```

这些代码会打印出组成 `String` 的 4 个字节：

```
208
151
208
180
```

不过请记住有效的 Unicode 标量值可能会由不止一个字节组成。

从字符串中获取如同天城文这样的字形簇是很复杂的，所以标准库并没有提供这个功能。[crates.io](https://crates.io/) 上有些提供这样功能的 crate。

### [字符串并不简单](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%B9%B6%E4%B8%8D%E7%AE%80%E5%8D%95) <a href="#zi-fu-chuan-bing-bu-jian-dan" id="zi-fu-chuan-bing-bu-jian-dan"></a>

总而言之，字符串还是很复杂的。不同的语言选择了不同的向程序员展示其复杂性的方式。Rust 选择了以准确的方式处理 `String` 数据作为所有 Rust 程序的默认行为，这意味着程序员们必须更多的思考如何预先处理 UTF-8 数据。这种权衡取舍相比其他语言更多的暴露出了字符串的复杂性，不过也使你在开发周期后期免于处理涉及非 ASCII 字符的错误。

好消息是标准库提供了很多围绕 `String` 和 `&str` 构建的功能，来帮助我们正确处理这些复杂场景。请务必查看这些使用方法的文档，例如 `contains` 来搜索一个字符串，和 `replace` 将字符串的一部分替换为另一个字符串。

称作 `String` 的类型是由标准库提供的，而没有写进核心语言部分，它是可增长的、可变的、有所有权的、UTF-8 编码的字符串类型。当 Rustacean 们谈到 Rust 的 “字符串”时，它们通常指的是 `String` 或字符串 slice `&str` 类型，而不特指其中某一个。虽然本部分内容大多是关于 `String` 的，不过这两个类型在 Rust 标准库中都被广泛使用，`String` 和字符串 slices 都是 UTF-8 编码的。

现在让我们转向一些不太复杂的集合：哈希 map！

## [使用 Hash Map 储存键值对](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E4%BD%BF%E7%94%A8-hash-map-%E5%82%A8%E5%AD%98%E9%94%AE%E5%80%BC%E5%AF%B9) <a href="#shi-yong-hashmap-chu-cun-jian-zhi-dui" id="shi-yong-hashmap-chu-cun-jian-zhi-dui"></a>

最后介绍的常用集合类型是 **哈希 map**（_hash map_）。`HashMap<K, V>` 类型储存了一个键类型 `K` 对应一个值类型 `V` 的映射。它通过一个 **哈希函数**（_hashing function_）来实现映射，决定如何将键和值放入内存中。很多编程语言支持这种数据结构，不过通常有不同的名字：哈希、map、对象、哈希表或者关联数组，仅举几例。

哈希 map 可以用于需要任何类型作为键来寻找数据的情况，而不是像 vector 那样通过索引。例如，在一个游戏中，你可以将每个团队的分数记录到哈希 map 中，其中键是队伍的名字而值是每个队伍的分数。给出一个队名，就能得到他们的得分。

本章我们会介绍哈希 map 的基本 API，不过还有更多吸引人的功能隐藏于标准库在 `HashMap<K, V>` 上定义的函数中。一如既往请查看标准库文档来了解更多信息。

### [新建一个哈希 map](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E6%96%B0%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%93%88%E5%B8%8C-map) <a href="#xin-jian-yi-ge-ha-xi-map" id="xin-jian-yi-ge-ha-xi-map"></a>

可以使用 `new` 创建一个空的 `HashMap`，并使用 `insert` 增加元素。在示例 8-20 中我们记录两支队伍的分数，分别是蓝队和黄队。蓝队开始有 10 分而黄队开始有 50 分：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}

```

示例 8-20：新建一个哈希 map 并插入一些键值对

注意必须首先 `use` 标准库中集合部分的 `HashMap`。在这三个常用集合中，`HashMap` 是最不常用的，所以并没有被 prelude 自动引用。标准库中对 `HashMap` 的支持也相对较少，例如，并没有内建的构建宏。

像 vector 一样，哈希 map 将它们的数据储存在堆上，这个 `HashMap` 的键类型是 `String` 而值类型是 `i32`。类似于 vector，<mark style="color:red;">哈希 map 是同质的：所有的键必须是相同类型，值也必须都是相同类型</mark>。

#### [访问哈希 map 中的值](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E8%AE%BF%E9%97%AE%E5%93%88%E5%B8%8C-map-%E4%B8%AD%E7%9A%84%E5%80%BC) <a href="#fang-wen-ha-xi-map-zhong-de-zhi" id="fang-wen-ha-xi-map-zhong-de-zhi"></a>

可以通过 `get` 方法并提供对应的键来从哈希 map 中获取值，如示例 8-21 所示：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
}

```

示例 8-21：访问哈希 map 中储存的蓝队分数

这里，`score` 是与蓝队分数相关的值，应为 `10`。<mark style="color:red;">`get`</mark> <mark style="color:red;"></mark><mark style="color:red;">方法返回</mark> <mark style="color:red;"></mark><mark style="color:red;">`Option<&V>`</mark>，如果某个键在哈希 map 中没有对应的值，`get` 会返回 `None`。程序中通过调用 <mark style="color:red;">`copied`</mark> <mark style="color:red;"></mark><mark style="color:red;">方法来获取一个</mark> <mark style="color:red;"></mark><mark style="color:red;">`Option<i32>`</mark> <mark style="color:red;"></mark><mark style="color:red;">而不是</mark> <mark style="color:red;"></mark><mark style="color:red;">`Option<&i32>`</mark>，接着调用 <mark style="color:red;">`unwrap_or`</mark> <mark style="color:red;"></mark><mark style="color:red;">在</mark> <mark style="color:red;"></mark><mark style="color:red;">`score`</mark> <mark style="color:red;"></mark><mark style="color:red;">中没有该键所对应的项时将其设置为零</mark>。

可以使用与 vector 类似的方式来遍历哈希 map 中的每一个键值对，也就是 `for` 循环：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
}

```

这会以任意顺序打印出每一个键值对：

```
Yellow: 50
Blue: 10
```

### [哈希 map 和所有权](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E5%93%88%E5%B8%8C-map-%E5%92%8C%E6%89%80%E6%9C%89%E6%9D%83) <a href="#ha-xi-map-he-suo-you-quan" id="ha-xi-map-he-suo-you-quan"></a>

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者，如示例 8-22 所示：

```rust
fn main() {
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // 这里 field_name 和 field_value 不再有效，
    // 尝试使用它们看看会出现什么编译错误！
}

```

示例 8-22：展示一旦键值对被插入后就为哈希 map 所拥有

当 `insert` 调用将 `field_name` 和 `field_value` 移动到哈希 map 中后，将不能使用这两个绑定。

如果将值的引用插入哈希 map，这些值本身将不会被移动进哈希 map。但是这些引用指向的值必须至少在哈希 map 有效时也是有效的。第十章 [“生命周期与引用有效性”](https://kaisery.github.io/trpl-zh-cn/ch10-03-lifetime-syntax.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E4%B8%8E%E5%BC%95%E7%94%A8%E6%9C%89%E6%95%88%E6%80%A7) 部分将会更多的讨论这个问题。

### [更新哈希 map](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E6%9B%B4%E6%96%B0%E5%93%88%E5%B8%8C-map) <a href="#geng-xin-ha-xi-map" id="geng-xin-ha-xi-map"></a>

尽管键值对的数量是可以增长的，每个唯一的键只能同时关联一个值（反之不一定成立：比如蓝队和黄队的 `scores` 哈希 map 中都可能存储有 10 这个值）。

当我们想要改变哈希 map 中的数据时，必须决定如何处理一个键已经有值了的情况。可以选择完全无视旧值并用新值代替旧值。可以选择保留旧值而忽略新值，并只在键 **没有** 对应值时增加新值。或者可以结合新旧两值。让我们看看这分别该如何处理！

#### [**覆盖一个值**](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E8%A6%86%E7%9B%96%E4%B8%80%E4%B8%AA%E5%80%BC)

如果我们插入了一个键值对，接着用相同的键插入一个不同的值，与这个键相关联的旧值将被替换。即便示例 8-23 中的代码调用了两次 `insert`，哈希 map 也只会包含一个键值对，因为两次都是对蓝队的键插入的值：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores);
}

```

示例 8-23：替换以特定键储存的值

这会打印出 `{"Blue": 25}`。原始的值 `10` 则被覆盖了。

#### [**只在键没有对应值时插入键值对**](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E5%8F%AA%E5%9C%A8%E9%94%AE%E6%B2%A1%E6%9C%89%E5%AF%B9%E5%BA%94%E5%80%BC%E6%97%B6%E6%8F%92%E5%85%A5%E9%94%AE%E5%80%BC%E5%AF%B9)

我们经常会检查某个特定的键是否已经存在于哈希 map 中并进行如下操作：如果哈希 map 中键已经存在则不做任何操作。如果不存在则连同值一块插入。

为此哈希 map 有一个特有的 API，叫做 <mark style="color:red;">`entry`</mark>，它获取我们想要检查的键作为参数。`entry` 函数的返回值是一个枚举，`Entry`，它代表了可能存在也可能不存在的值。比如说我们想要检查黄队的键是否关联了一个值。如果没有，就插入值 50，对于蓝队也是如此。使用 `entry` API 的代码看起来像示例 8-24 这样：

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
}

```

示例 8-24：使用 `entry` 方法只在键没有对应一个值时插入

`Entry` 的 `or_insert` 方法在键对应的值存在时就返回这个值的可变引用，如果不存在则将参数作为新值插入并返回新值的可变引用。这比编写自己的逻辑要简明的多，另外也与借用检查器结合得更好。

运行示例 8-24 的代码会打印出 `{"Yellow": 50, "Blue": 10}`。第一个 `entry` 调用会插入黄队的键和值 `50`，因为黄队并没有一个值。第二个 `entry` 调用不会改变哈希 map 因为蓝队已经有了值 `10`。

#### [**根据旧值更新一个值**](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E6%A0%B9%E6%8D%AE%E6%97%A7%E5%80%BC%E6%9B%B4%E6%96%B0%E4%B8%80%E4%B8%AA%E5%80%BC)

另一个常见的哈希 map 的应用场景是找到一个键对应的值并根据旧的值更新它。例如，示例 8-25 中的代码计数一些文本中每一个单词分别出现了多少次。我们使用哈希 map 以单词作为键并递增其值来记录我们遇到过几次这个单词。如果是第一次看到某个单词，就插入值 `0`。

```rust
fn main() {
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
}

```

示例 8-25：通过哈希 map 储存单词和计数来统计出现次数

这会打印出 `{"world": 2, "hello": 1, "wonderful": 1}`。你可能会看到相同的键值对以不同的顺序打印：回忆一下[“访问哈希 map 中的值”](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E8%AE%BF%E9%97%AE%E5%93%88%E5%B8%8C-map-%E4%B8%AD%E7%9A%84%E5%80%BC)部分中遍历哈希 map 会以任意顺序进行。

`split_whitespace` 方法返回一个由空格分隔 `text` 值子 slice 的迭代器。`or_insert` 方法返回这个键的值的一个可变引用（`&mut V`）。这里我们将这个可变引用储存在 `count` 变量中，所以为了赋值必须首先使用星号（`*`）解引用 `count`。这个可变引用在 `for` 循环的结尾离开作用域，这样所有这些改变都是安全的并符合借用规则。

### [哈希函数](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E5%93%88%E5%B8%8C%E5%87%BD%E6%95%B0) <a href="#ha-xi-han-shu" id="ha-xi-han-shu"></a>

`HashMap` 默认使用一种叫做 SipHash 的哈希函数，它可以抵御涉及哈希表（hash table）[1](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#siphash) 的拒绝服务（Denial of Service, DoS）攻击。然而这并不是可用的最快的算法，不过为了更高的安全性值得付出一些性能的代价。如果性能监测显示此哈希函数非常慢，以致于你无法接受，你可以指定一个不同的 _hasher_ 来切换为其它函数。hasher 是一个实现了 `BuildHasher` trait 的类型。第十章会讨论 trait 和如何实现它们。你并不需要从头开始实现你自己的 hasher；[crates.io](https://crates.io/) 有其他人分享的实现了许多常用哈希算法的 hasher 的库。

1&#x20;

[https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## [总结](https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html#%E6%80%BB%E7%BB%93) <a href="#zong-jie" id="zong-jie"></a>

vector、字符串和哈希 map 会在你的程序需要储存、访问和修改数据时帮助你。这里有一些你应该能够解决的练习问题：

* 给定一系列数字，使用 vector 并返回这个列表的中位数（排列数组后位于中间的值）和众数（mode，出现次数最多的值；这里哈希 map 会很有帮助）。
* 将字符串转换为 Pig Latin，也就是每一个单词的第一个辅音字母被移动到单词的结尾并增加 “ay”，所以 “first” 会变成 “irst-fay”。元音字母开头的单词则在结尾增加 “hay”（“apple” 会变成 “apple-hay”）。牢记 UTF-8 编码！
* 使用哈希 map 和 vector，创建一个文本接口来允许用户向公司的部门中增加员工的名字。例如，“Add Sally to Engineering” 或 “Add Amir to Sales”。接着让用户获取一个部门的所有员工的列表，或者公司每个部门的所有员工按照字典序排列的列表。

标准库 API 文档中描述的这些类型的方法将有助于你进行这些练习！

我们已经开始接触可能会有失败操作的复杂程序了，这也意味着接下来是一个了解错误处理的绝佳时机！



