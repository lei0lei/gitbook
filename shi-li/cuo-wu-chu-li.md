# 错误处理

## [错误处理](https://rustwiki.org/zh-CN/rust-by-example/error.html#%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86) <a href="#cuo-wu-chu-li" id="cuo-wu-chu-li"></a>

错误处理（error handling）是处理可能发生的失败情况的过程。例如读取一个文件时失败了，如果继续使用这个**无效的**输入，那显然是有问题的。注意到并且显式地处理这种错误可以避免程序的其他部分产生潜在的问题。

在 Rust 中有多种处理错误的方式，在接下来的小节中会一一介绍。它们多少有些区别，使用场景也不尽相同。总的来说：

* 显式的 `panic` 主要用于测试，以及处理不可恢复的错误。在原型开发中这很有用，比如 用来测试还没有实现的函数，不过这时使用 `unimplemented` 更能表达意图。另外在 测试中，`panic` 是一种显式地失败（fail）的好方法。
* `Option` 类型是为了值是可选的、或者缺少值并不是错误的情况准备的。比如说寻找 父目录时，`/` 和 `C:` 这样的目录就没有父目录，这应当并不是一个错误。当处理 `Option` 时，`unwrap` 可用于原型开发，也可以用于能够确定 `Option` 中一定有值 的情形。然而 `expect` 更有用，因为它允许你指定一条错误信息，以免万一还是出现 了错误。
* 当错误有可能发生，且应当由调用者处理时，使用 `Result`。你也可以 `unwrap` 然后 使用 `expect`，但是除了在测试或者原型开发中，请不要这样做。

有关错误处理的更多内容，可参考[官方文档](https://rustwiki.org/zh-CN/book/ch09-00-error-handling.html)的错误处理的章节。

## [`panic`](https://rustwiki.org/zh-CN/rust-by-example/error/panic.html#panic) <a href="#panic" id="panic"></a>

我们将要看到的最简单的错误处理机制就是 `panic`。它会打印一个错误消息，开始回退（unwind）任务，且通常会退出程序。这里我们显式地在错误条件下调用 `panic`：

```
fn give_princess(gift: &str) {
    // 公主讨厌蛇，所以如果公主表示厌恶的话我们要停止！
    if gift == "snake" { panic!("AAAaaaaa!!!!"); }

    println!("I love {}s!!!!!", gift);
}

fn main() {
    give_princess("teddy bear");
    give_princess("snake");
}

```

## [`Option` 和 `unwrap`](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap.html#option-%E5%92%8C-unwrap) <a href="#option-he-unwrap" id="option-he-unwrap"></a>

上个例子展示了如何主动地引入程序失败（program failure）。当公主收到蛇这件不合适的礼物时，我们就让程序 `panic`。但是，如果公主期待收到礼物，却没收到呢？这同样是一件糟糕的事情，所以我们要想办法来解决这个问题！

我们**可以**检查空字符串（`""`），就像处理蛇那样。但既然我们在用 Rust，不如让编译器辨别没有礼物的情况。

在标准库（`std`）中有个叫做 `Option<T>`（option 中文意思是 “选项”）的枚举类型，用于有 “不存在” 的可能性的情况。它表现为以下两个 “option”（选项）中的一个：

* `Some(T)`：找到一个属于 `T` 类型的元素
* `None`：找不到相应元素

这些选项可以通过 `match` 显式地处理，或使用 `unwrap` 隐式地处理。隐式处理要么返回 `Some` 内部的元素，要么就 `panic`。

请注意，手动使用 [expect](https://rustwiki.org/zh-CN/std/option/enum.Option.html#method.expect) 方法自定义 `panic` 信息是可能的，但相比显式处理，`unwrap` 的输出仍显得不太有意义。在下面例子中，显式处理将举出更受控制的结果，同时如果需要的话，仍然可以使程序 `panic`。

```
// 平民（commoner）们见多识广，收到什么礼物都能应对。
// 所有礼物都显式地使用 `match` 来处理。
fn give_commoner(gift: Option<&str>) {
    // 指出每种情况下的做法。
    match gift {
        Some("snake") => println!("Yuck! I'm throwing that snake in a fire."),
        Some(inner)   => println!("{}? How nice.", inner),
        None          => println!("No gift? Oh well."),
    }
}

// 养在深闺人未识的公主见到蛇就会 `panic`（恐慌）。
// 这里所有的礼物都使用 `unwrap` 隐式地处理。
fn give_princess(gift: Option<&str>) {
    // `unwrap` 在接收到 `None` 时将返回 `panic`。
    let inside = gift.unwrap();
    if inside == "snake" { panic!("AAAaaaaa!!!!"); }

    println!("I love {}s!!!!!", inside);
}

fn main() {
    let food  = Some("chicken");
    let snake = Some("snake");
    let void  = None;

    give_commoner(food);
    give_commoner(snake);
    give_commoner(void);

    let bird = Some("robin");
    let nothing = None;

    give_princess(bird);
    give_princess(nothing);
}

```

### [使用 `?` 解开 `Option`](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap/question_mark.html#%E4%BD%BF%E7%94%A8--%E8%A7%A3%E5%BC%80-option)

你可以使用 `match` 语句来解开 `Option`，但使用 `?` 运算符通常会更容易。如果 `x` 是 `Option`，那么若 `x` 是 `Some` ，对`x?`表达式求值将返回底层值，否则无论函数是否正在执行都将终止且返回 `None`。

```
fn next_birthday(current_age: Option<u8>) -> Option<String> {
    // 如果 `current_age` 是 `None`，这将返回 `None`。
    // 如果 `current_age` 是 `Some`，内部的 `u8` 将赋值给 `next_age`。
    let next_age: u8 = current_age?;
    Some(format!("Next year I will be {}", next_age))
}

```

你可以将多个 `?` 链接在一起，以使代码更具可读性。

```
struct Person {
    job: Option<Job>,
}

#[derive(Clone, Copy)]
struct Job {
    phone_number: Option<PhoneNumber>,
}

#[derive(Clone, Copy)]
struct PhoneNumber {
    area_code: Option<u8>,
    number: u32,
}

impl Person {

    // 获取此人的工作电话号码的区号（如果存在的话）。
    fn work_phone_area_code(&self) -> Option<u8> {
        // 没有`？`运算符的话，这将需要很多的嵌套的 `match` 语句。
        // 这将需要更多代码——尝试自己编写一下，看看哪个更容易。
        self.job?.phone_number?.area_code
    }
}

fn main() {
    let p = Person {
        job: Some(Job {
            phone_number: Some(PhoneNumber {
                area_code: Some(61),
                number: 439222222,
            }),
        }),
    };

    assert_eq!(p.work_phone_area_code(), Some(61));
}

```

### [组合算子：`map`](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap/map.html#%E7%BB%84%E5%90%88%E7%AE%97%E5%AD%90map)

`match` 是处理 `Option` 的一个可用的方法，但你会发现大量使用它会很繁琐，特别是当操作只对一种输入是有效的时。这时，可以使用[组合算子](https://rustwiki.org/zh-CN/reference/glossary.html#%E7%BB%84%E5%90%88%E7%AE%97%E5%AD%90)（combinator），以模块化的风格来管理控制流。

`Option` 有一个内置方法 `map()`，这个组合算子可用于 `Some -> Some` 和 `None -> None` 这样的简单映射。多个不同的 `map()` 调用可以串起来，这样更加灵活。

在下面例子中，`process()` 轻松取代了前面的所有函数，且更加紧凑。

```
#![allow(dead_code)]

#[derive(Debug)] enum Food { Apple, Carrot, Potato }

#[derive(Debug)] struct Peeled(Food);
#[derive(Debug)] struct Chopped(Food);
#[derive(Debug)] struct Cooked(Food);

// 削皮。如果没有食物，就返回 `None`。否则返回削好皮的食物。
fn peel(food: Option<Food>) -> Option<Peeled> {
    match food {
        Some(food) => Some(Peeled(food)),
        None       => None,
    }
}

// 切食物。如果没有食物，就返回 `None`。否则返回切好的食物。
fn chop(peeled: Option<Peeled>) -> Option<Chopped> {
    match peeled {
        Some(Peeled(food)) => Some(Chopped(food)),
        None               => None,
    }
}

// 烹饪食物。这里，我们使用 `map()` 来替代 `match` 以处理各种情况。
fn cook(chopped: Option<Chopped>) -> Option<Cooked> {
    chopped.map(|Chopped(food)| Cooked(food))
}

// 这个函数会完成削皮切块烹饪一条龙。我们把 `map()` 串起来，以简化代码。
fn process(food: Option<Food>) -> Option<Cooked> {
    food.map(|f| Peeled(f))
        .map(|Peeled(f)| Chopped(f))
        .map(|Chopped(f)| Cooked(f))
}

// 在尝试吃食物之前确认食物是否存在是非常重要的！
fn eat(food: Option<Cooked>) {
    match food {
        Some(food) => println!("Mmm. I love {:?}", food),
        None       => println!("Oh no! It wasn't edible."),
    }
}

fn main() {
    let apple = Some(Food::Apple);
    let carrot = Some(Food::Carrot);
    let potato = None;

    let cooked_apple = cook(chop(peel(apple)));
    let cooked_carrot = cook(chop(peel(carrot)));

    // 现在让我们试试看起来更简单的 `process()`。
    let cooked_potato = process(potato);

    eat(cooked_apple);
    eat(cooked_carrot);
    eat(cooked_potato);
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap/map.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[闭包](https://rustwiki.org/zh-CN/rust-by-example/fn/closures.html), [`Option`](https://rustwiki.org/zh-CN/std/option/enum.Option.html), 和 [`Option::map()`](https://rustwiki.org/zh-CN/std/option/enum.Option.html#method.map)\
\


### [组合算子：`and_then`](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap/and_then.html#%E7%BB%84%E5%90%88%E7%AE%97%E5%AD%90and_then)

`map()` 以链式调用的方式来简化 `match` 语句。然而，如果以返回类型是 `Option<T>` 的函数作为 `map()` 的参数，会导致出现嵌套形式 `Option<Option<T>>`。这样多层串联调用就会变得混乱。所以有必要引入 `and_then()`，在某些语言中它叫做 flatmap。

`and_then()` 使用被 `Option` 包裹的值来调用其输入函数并返回结果。 如果 `Option` 是 `None`，那么它返回 `None`。

在下面例子中，`cookable_v2()` 会产生一个 `Option<Food>`。如果在这里使用 `map()` 而不是 `and_then()` 将会得到 `Option<Option<Food>>`，这对 `eat()` 来说是一个无效类型。

```
#![allow(dead_code)]

#[derive(Debug)] enum Food { CordonBleu, Steak, Sushi }
#[derive(Debug)] enum Day { Monday, Tuesday, Wednesday }

// 我们没有制作寿司所需的原材料（ingredient）（有其他的原材料）。
fn have_ingredients(food: Food) -> Option<Food> {
    match food {
        Food::Sushi => None,
        _           => Some(food),
    }
}

// 我们拥有全部食物的食谱，除了法国蓝带猪排（Cordon Bleu）的。
fn have_recipe(food: Food) -> Option<Food> {
    match food {
        Food::CordonBleu => None,
        _                => Some(food),
    }
}


// 要做一份好菜，我们需要原材料和食谱。
// 我们可以借助一系列 `match` 来表达这个逻辑：
fn cookable_v1(food: Food) -> Option<Food> {
    match have_ingredients(food) {
        None       => None,
        Some(food) => match have_recipe(food) {
            None       => None,
            Some(food) => Some(food),
        },
    }
}

// 也可以使用 `and_then()` 把上面的逻辑改写得更紧凑：
fn cookable_v2(food: Food) -> Option<Food> {
    have_ingredients(food).and_then(have_recipe)
}

fn eat(food: Food, day: Day) {
    match cookable_v2(food) {
        Some(food) => println!("Yay! On {:?} we get to eat {:?}.", day, food),
        None       => println!("Oh no. We don't get to eat on {:?}?", day),
    }
}

fn main() {
    let (cordon_bleu, steak, sushi) = (Food::CordonBleu, Food::Steak, Food::Sushi);

    eat(cordon_bleu, Day::Monday);
    eat(steak, Day::Tuesday);
    eat(sushi, Day::Wednesday);
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/error/option_unwrap/and_then.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[闭包](https://rustwiki.org/zh-CN/rust-by-example/fn/closures.html)，[`Option::map()`](https://rustwiki.org/zh-CN/std/option/enum.Option.html#method.map), 和 [`Option::and_then()`](https://rustwiki.org/zh-CN/std/option/enum.Option.html#method.and_then)

## [结果 `Result`](https://rustwiki.org/zh-CN/rust-by-example/error/result.html#%E7%BB%93%E6%9E%9C-result) <a href="#jie-guo-result" id="jie-guo-result"></a>

[`Result`](https://rustwiki.org/zh-CN/std/result/enum.Result.html) 是 [`Option`](https://rustwiki.org/zh-CN/std/option/enum.Option.html) 类型的更丰富的版本，描述的是可能的**错误**而不是可能的**不存在**。

也就是说，`Result<T，E>` 可以有两个结果的其中一个：

* `Ok<T>`：找到 `T` 元素
* `Err<E>`：找到 `E` 元素，`E` 即表示错误的类型。

按照约定，预期结果是 “Ok”，而意外结果是 “Err”。

`Result` 有很多类似 `Option` 的方法。例如 `unwrap()`，它要么举出元素 `T`，要么就 `panic`。 对于事件的处理，`Result` 和 `Option` 有很多相同的组合算子。

在使用 Rust 时，你可能会遇到返回 `Result` 类型的方法，例如 [`parse()`](https://rustwiki.org/zh-CN/std/primitive.str.html#method.parse) 方法。它并不是总能把字符串解析成指定的类型，所以 `parse()` 返回一个 `Result` 表示可能的失败。

我们来看看当 `parse()` 字符串成功和失败时会发生什么：

```
fn multiply(first_number_str: &str, second_number_str: &str) -> i32 {
    // 我们试着用 `unwrap()` 把数字放出来。它会咬我们一口吗？
    let first_number = first_number_str.parse::<i32>().unwrap();
    let second_number = second_number_str.parse::<i32>().unwrap();
    first_number * second_number
}

fn main() {
    let twenty = multiply("10", "2");
    println!("double is {}", twenty);

    let tt = multiply("t", "2");
    println!("double is {}", tt);
}

```

在失败的情况下，`parse()` 产生一个错误，留给 `unwrap()` 来解包并产生 `panic`。另外，`panic` 会退出我们的程序，并提供一个让人很不爽的错误消息。

为了改善错误消息的质量，我们应该更具体地了解返回类型并考虑显式地处理错误。

### [`Result` 的 `map`](https://rustwiki.org/zh-CN/rust-by-example/error/result/result_map.html#result-%E7%9A%84-map) <a href="#result-de-map" id="result-de-map"></a>

上一节的 `multiply` 函数的 panic 设计不是健壮的（robust）。一般地，我们希望把错误返回给调用者，这样它可以决定回应错误的正确方式。

首先，我们需要了解需要处理的错误类型是什么。为了确定 `Err` 的类型，我们可以用 [`parse()`](https://rustwiki.org/zh-CN/std/primitive.str.html#method.parse) 来试验。Rust 已经为 [`i32`](https://rustwiki.org/zh-CN/std/primitive.i32.html) 类型使用 [`FromStr`](https://rustwiki.org/zh-CN/std/str/trait.FromStr.html) trait 实现了 `parse()`。结果表明，这里的 `Err` 类型被指定为 [`ParseIntError`](https://rustwiki.org/zh-CN/std/num/struct.ParseIntError.html)。

> 译注：原文没有具体讲如何确定 `Err` 的类型。由于目前用于获取类型的函数仍然是不 稳定的，我们可以用间接的方法。使用下面的代码：
>
> ```
> fn main () {    let i: () = "t".parse::<i32>();}
> ```
>
> 由于不可能把 `Result` 类型赋给单元类型变量 `i`，编译器会提示我们：
>
> ```
> note: expected type `()`
>          found type `std::result::Result<i32, std::num::ParseIntError>`
> ```
>
> 这样就知道了 `parse<i32>` 函数的返回类型详情。

在下面的例子中，使用简单的 `match` 语句导致了更加繁琐的代码。

```
use std::num::ParseIntError;

// 修改了上一节中的返回类型，现在使用模式匹配而不是 `unwrap()`。
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    match first_number_str.parse::<i32>() {
        Ok(first_number)  => {
            match second_number_str.parse::<i32>() {
                Ok(second_number)  => {
                    Ok(first_number * second_number)
                },
                Err(e) => Err(e),
            }
        },
        Err(e) => Err(e),
    }
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    // 这种情形下仍然会给出正确的答案。
    let twenty = multiply("10", "2");
    print(twenty);

    // 这种情况下就会提供一条更有用的错误信息。
    let tt = multiply("t", "2");
    print(tt);
}
use std::num::ParseIntError;

// 就像 `Option` 那样，我们可以使用 `map()` 之类的组合算子。
// 除去写法外，这个函数与上面那个完全一致，它的作用是：
// 如果值是合法的，计算其乘积，否则返回错误。
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    first_number_str.parse::<i32>().and_then(|first_number| {
        second_number_str.parse::<i32>().map(|second_number| first_number * second_number)
    })
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    // 这种情况下仍然会给出正确的答案。
    let twenty = multiply("10", "2");
    print(twenty);

    // 这种情况下就会提供一条更有用的错误信息。
    let tt = multiply("t", "2");
    print(tt);
}

```

幸运的是，`Option` 的 `map`、`and_then`、以及很多其他组合算子也为 `Result` 实现了。官方文档的 [`Result`](https://rustwiki.org/zh-CN/std/result/enum.Result.html) 一节包含完整的方法列表。

```
use std::num::ParseIntError;

// 就像 `Option` 那样，我们可以使用 `map()` 之类的组合算子。
// 除去写法外，这个函数与上面那个完全一致，它的作用是：
// 如果值是合法的，计算其乘积，否则返回错误。
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    first_number_str.parse::<i32>().and_then(|first_number| {
        second_number_str.parse::<i32>().map(|second_number| first_number * second_number)
    })
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    // 这种情况下仍然会给出正确的答案。
    let twenty = multiply("10", "2");
    print(twenty);

    // 这种情况下就会提供一条更有用的错误信息。
    let tt = multiply("t", "2");
    print(tt);
}

```

### [给 `Result` 取别名](https://rustwiki.org/zh-CN/rust-by-example/error/result/result_alias.html#%E7%BB%99-result-%E5%8F%96%E5%88%AB%E5%90%8D) <a href="#gei-result-qu-bie-ming" id="gei-result-qu-bie-ming"></a>

当我们要重用某个 `Result` 类型时，该怎么办呢？回忆一下，Rust 允许我们创建[别名](https://rustwiki.org/zh-CN/rust-by-example/types/alias.html)。若某个 `Result` 有可能被重用，我们可以方便地给它取一个别名。

在模块的层面上创建别名特别有帮助。同一模块中的错误常常会有相同的 `Err` 类型，所以单个别名就能简便地定义**所有**相关的 `Result`。这太有用了，以至于标准库也提供了一个别名： [`io::Result`](https://rustwiki.org/zh-CN/std/io/type.Result.html)！

下面给出一个简短的示例来展示语法：

```
use std::num::ParseIntError;// 为带有错误类型 `ParseIntError` 的 `Result` 定义一个泛型别名。type AliasedResult<T> = Result<T, ParseIntError>;// 使用上面定义过的别名来表示上一节中的 `Result<i32,ParseIntError>` 类型。fn multiply(first_number_str: &str, second_number_str: &str) -> AliasedResult<i32> {    first_number_str.parse::<i32>().and_then(|first_number| {        second_number_str.parse::<i32>().map(|second_number| first_number * second_number)    })}// 在这里使用别名又让我们节省了一些代码量。fn print(result: AliasedResult<i32>) {    match result {        Ok(n)  => println!("n is {}", n),        Err(e) => println!("Error: {}", e),    }}fn main() {    print(multiply("10", "2"));    print(multiply("t", "2"));}
```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/error/result/result_alias.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`io::Result`](https://rustwiki.org/zh-CN/std/io/type.Result.html)

### [提前返回](https://rustwiki.org/zh-CN/rust-by-example/error/result/early_returns.html#%E6%8F%90%E5%89%8D%E8%BF%94%E5%9B%9E) <a href="#ti-qian-fan-hui" id="ti-qian-fan-hui"></a>

在上一个例子中，我们显式地使用组合算子处理了错误。另一种处理错误的方式是使用 `match` 语句和**提前返回**（early return）的结合。

这也就是说，如果发生错误，我们可以停止函数的执行然后返回错误。对有些人来说，这样的代码更好写，更易读。这次我们使用提前返回改写之前的例子：

```
use std::num::ParseIntError;

fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    let first_number = match first_number_str.parse::<i32>() {
        Ok(first_number)  => first_number,
        Err(e) => return Err(e),
    };

    let second_number = match second_number_str.parse::<i32>() {
        Ok(second_number)  => second_number,
        Err(e) => return Err(e),
    };

    Ok(first_number * second_number)
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    print(multiply("10", "2"));
    print(multiply("t", "2"));
}

```

到此为止，我们已经学会了如何使用组合算子和提前返回显式地处理错误。我们一般是想要避免 panic 的，但显式地处理所有错误确实显得过于繁琐。

在下一部分，我们将看到，当只是需要 `unwrap` 并且不产生 `panic` 时，可以使用 `?` 来达到同样的效果。

### [引入 `?`](https://rustwiki.org/zh-CN/rust-by-example/error/result/enter_question_mark.html#%E5%BC%95%E5%85%A5-)

有时我们只是想 `unwrap` 且避免产生 `panic`。到现在为止，对 `unwrap` 的错误处理都在强迫我们一层层地嵌套，然而我们只是想把里面的变量拿出来。`?` 正是为这种情况准备的。

当找到一个 `Err` 时，可以采取两种行动：

1. `panic!`，不过我们已经决定要尽可能避免 panic 了。
2. 返回它，因为 `Err` 就意味着它已经不能被处理了。

`?` **几乎**[1](https://rustwiki.org/zh-CN/rust-by-example/error/result/enter_question_mark.html#%E2%80%A0) 就等于一个会返回 `Err` 而不是 `panic` 的 `unwrap`。我们来看看怎样简化之前使用组合算子的例子：

```
use std::num::ParseIntError;

fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    let first_number = first_number_str.parse::<i32>()?;
    let second_number = second_number_str.parse::<i32>()?;

    Ok(first_number * second_number)
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    print(multiply("10", "2"));
    print(multiply("t", "2"));
}

```

#### [`try!` 宏](https://rustwiki.org/zh-CN/rust-by-example/error/result/enter_question_mark.html#try-%E5%AE%8F) <a href="#try-hong" id="try-hong"></a>

在 `?` 出现以前，相同的功能是使用 `try!` 宏完成的。现在我们推荐使用 `?` 运算符，但是在老代码中仍然会看到 `try!`。如果使用 `try!` 的话，上一个例子中的 `multiply` 函数看起来会像是这样：

```
use std::num::ParseIntError;

fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    let first_number = try!(first_number_str.parse::<i32>());
    let second_number = try!(second_number_str.parse::<i32>());

    Ok(first_number * second_number)
}

fn print(result: Result<i32, ParseIntError>) {
    match result {
        Ok(n)  => println!("n is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    print(multiply("10", "2"));
    print(multiply("t", "2"));
}

```

1&#x20;

更多细节请看[`?` 的更多用法](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/reenter_question_mark.html)。\


## [处理多种错误类型](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types.html#%E5%A4%84%E7%90%86%E5%A4%9A%E7%A7%8D%E9%94%99%E8%AF%AF%E7%B1%BB%E5%9E%8B)

前面出现的例子都是很方便的情况；都是 `Result` 和其他 `Result` 交互，还有 `Option` 和其他 `Option` 交互。

有时 `Option` 需要和 `Result` 进行交互，或是 `Result<T, Error1>` 需要和 `Result<T, Error2>` 进行交互。在这类情况下，我们想要以一种方式来管理不同的错误类型，使得它们可组合且易于交互。

在下面代码中，`unwrap` 的两个实例生成了不同的错误类型。`Vec::first` 返回一个 `Option`，而 `parse::<i32>` 返回一个 `Result<i32, ParseIntError>`：

```
fn double_first(vec: Vec<&str>) -> i32 {
    let first = vec.first().unwrap(); // 生成错误 1
    2 * first.parse::<i32>().unwrap() // 生成错误 2
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];
    
    println!("The first doubled is {}", double_first(numbers));
    
    println!("The first doubled is {}", double_first(empty));
    // 错误1：输入 vector 为空
    
    println!("The first doubled is {}", double_first(strings));
    // 错误2：此元素不能解析成数字
}

```

在下面几节中，我们会看到处理这类问题的几种策略。

### [从 `Option` 中取出 `Result`](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/option_result.html#%E4%BB%8E-option-%E4%B8%AD%E5%8F%96%E5%87%BA-result) <a href="#cong-option-zhong-qu-chu-result" id="cong-option-zhong-qu-chu-result"></a>

处理混合错误类型的最基本的手段就是让它们互相包含。

```
use std::num::ParseIntError;

fn double_first(vec: Vec<&str>) -> Option<Result<i32, ParseIntError>> {
    vec.first().map(|first| {
        first.parse::<i32>().map(|n| 2 * n)
    })
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    println!("The first doubled is {:?}", double_first(numbers));

    println!("The first doubled is {:?}", double_first(empty));
    // Error 1: the input vector is empty

    println!("The first doubled is {:?}", double_first(strings));
    // Error 2: the element doesn't parse to a number
}

```

有时候我们不想再处理错误（比如使用 [`?`](https://rustwiki.org/zh-CN/rust-by-example/error/result/enter_question_mark.html) 的时候），但如果 `Option` 是 `None` 则继续处理错误。一些组合算子可以让我们轻松地交换 `Result` 和 `Option`。

```
use std::num::ParseIntError;

fn double_first(vec: Vec<&str>) -> Result<Option<i32>, ParseIntError> {
    let opt = vec.first().map(|first| {
        first.parse::<i32>().map(|n| 2 * n)
    });

    opt.map_or(Ok(None), |r| r.map(Some))
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    println!("The first doubled is {:?}", double_first(numbers));
    println!("The first doubled is {:?}", double_first(empty));
    println!("The first doubled is {:?}", double_first(strings));
}

```

### &#x20;[定义一个错误类型](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/define_error_type.html#%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%AA%E9%94%99%E8%AF%AF%E7%B1%BB%E5%9E%8B)

有时候把所有不同的错误都视为一种错误类型会简化代码。我们将用一个自定义错误类型来演示这一点。

Rust 允许我们定义自己的错误类型。一般来说，一个 “好的” 错误类型应当：

* 用同一个类型代表了多种错误
* 向用户提供了清楚的错误信息
* 能够容易地与其他类型比较
  * 好的例子：`Err(EmptyVec)`
  * 坏的例子：`Err("Please use a vector with at least one element".to_owned())`
* 能够容纳错误的具体信息
  * 好的例子：`Err(BadChar(c, position))`
  * 坏的例子：`Err("+ cannot be used here".to_owned())`
* 能够与其他错误很好地整合

```
use std::error;
use std::fmt;

type Result<T> = std::result::Result<T, DoubleError>;

#[derive(Debug, Clone)]
// 定义我们的错误类型，这种类型可以根据错误处理的实际情况定制。
// 我们可以完全自定义错误类型，也可以在类型中完全采用底层的错误实现，
// 也可以介于二者之间。
struct DoubleError;

// 错误的生成与它如何显示是完全没关系的。没有必要担心复杂的逻辑会导致混乱的显示。
//
// 注意我们没有储存关于错误的任何额外信息，也就是说，如果不修改我们的错误类型定义的话，
// 就无法指明是哪个字符串解析失败了。
impl fmt::Display for DoubleError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}

// 为 `DoubleError` 实现 `Error` trait，这样其他错误可以包裹这个错误类型。
impl error::Error for DoubleError {
    fn source(&self) -> Option<&(dyn error::Error + 'static)> {
        // 泛型错误，没有记录其内部原因。
        None
    }
}

fn double_first(vec: Vec<&str>) -> Result<i32> {
    vec.first()
       // 把错误换成我们的新类型。
       .ok_or(DoubleError)
       .and_then(|s| {
            s.parse::<i32>()
                // 这里也换成新类型。
                .map_err(|_| DoubleError)
                .map(|i| 2 * i)
        })
}

fn print(result: Result<i32>) {
    match result {
        Ok(n)  => println!("The first doubled is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}

```

### [把错误 “装箱”](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/boxing_errors.html#%E6%8A%8A%E9%94%99%E8%AF%AF-%E8%A3%85%E7%AE%B1) <a href="#ba-cuo-wu-zhuang-xiang" id="ba-cuo-wu-zhuang-xiang"></a>

如果又想写简单的代码，又想保存原始错误信息，一个方法是把它们[装箱](https://rustwiki.org/zh-CN/std/boxed/struct.Box.html)（`Box`）。这样做的坏处就是，被包装的错误类型只能在运行时了解，而不能被[静态地判别](https://rustwiki.org/zh-CN/book/ch17-02-trait-objects.html#trait-%E5%AF%B9%E8%B1%A1%E6%89%A7%E8%A1%8C%E5%8A%A8%E6%80%81%E5%88%86%E5%8F%91)。

对任何实现了 `Error` trait 的类型，标准库的 `Box` 通过 [`From`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) 为它们提供了到 `Box<Error>` 的转换。

```
use std::error;
use std::fmt;

// 为 `Box<error::Error>` 取别名。
type Result<T> = std::result::Result<T, Box<dyn error::Error>>;

#[derive(Debug, Clone)]
struct EmptyVec;

impl fmt::Display for EmptyVec {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}

impl error::Error for EmptyVec {
    fn description(&self) -> &str {
        "invalid first item to double"
    }

    fn cause(&self) -> Option<&dyn error::Error> {
        // 泛型错误。没有记录其内部原因。
        None
    }
}

fn double_first(vec: Vec<&str>) -> Result<i32> {
    vec.first()
       .ok_or_else(|| EmptyVec.into())  // 装箱
       .and_then(|s| {
            s.parse::<i32>()
                .map_err(|e| e.into())  // 装箱
                .map(|i| 2 * i)
        })
}

fn print(result: Result<i32>) {
    match result {
        Ok(n)  => println!("The first doubled is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}

```

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/boxing_errors.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[动态分发](https://rustwiki.org/zh-CN/book/ch17-02-trait-objects.html#trait-%E5%AF%B9%E8%B1%A1%E6%89%A7%E8%A1%8C%E5%8A%A8%E6%80%81%E5%88%86%E5%8F%91) and [`Error` trait](https://rustwiki.org/zh-CN/std/error/trait.Error.html)

### [`?` 的其他用法](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/reenter_question_mark.html#-%E7%9A%84%E5%85%B6%E4%BB%96%E7%94%A8%E6%B3%95) <a href="#de-qi-ta-yong-fa" id="de-qi-ta-yong-fa"></a>

注意在上一个例子中，我们调用 `parse` 后总是立即将错误从标准库的错误 `map`（映射）到装箱错误。

```rust
.and_then(|s| s.parse::<i32>()
    .map_err(|e| e.into())
```

因为这个操作很简单常见，如果有省略写法就好了。遗憾的是 `and_then` 不够灵活，所以实现不了这样的写法。不过，我们可以使用 `?` 来代替它。

`?` 之前被解释为要么 `unwrap`，要么 `return Err(err)`，这只是在大多数情况下是正确的。`?` 实际上是指 `unwrap` 或 `return Err(From::from(err))`。由于 `From::from` 是不同类型之间的转换工具，也就是说，如果在错误可转换成返回类型地方使用 `?`，它将自动转换成返回类型。

我们在这里使用 `?` 重写之前的例子。重写后，只要为我们的错误类型实现 `From::from`，就可以不再使用 `map_err`。

```
use std::error;
use std::fmt;

// 为 `Box<error::Error>` 取别名。
type Result<T> = std::result::Result<T, Box<dyn error::Error>>;

#[derive(Debug)]
struct EmptyVec;

impl fmt::Display for EmptyVec {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}

impl error::Error for EmptyVec {}

// 这里的结构和之前一样，但是这次没有把所有的 `Result` 和 `Option` 串起来，
// 而是使用 `?` 立即得到内部值。
fn double_first(vec: Vec<&str>) -> Result<i32> {
    let first = vec.first().ok_or(EmptyVec)?;
    let parsed = first.parse::<i32>()?;
    Ok(2 * parsed)
}

fn print(result: Result<i32>) {
    match result {
        Ok(n)  => println!("The first doubled is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}

```

这段代码已经相当清晰了。与原来的 `panic` 相比，除了返回类型是 `Result` 之外，它就像是把所有的 `unwrap` 调用都换成 `?` 一样。因此必须在顶层解构它们。

#### [参见：](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/reenter_question_mark.html#%E5%8F%82%E8%A7%81) <a href="#can-jian" id="can-jian"></a>

[`From::from`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) 和 [`?`](https://rustwiki.org/zh-CN/reference/expressions/operator-expr.html#%E9%97%AE%E5%8F%B7%E6%93%8D%E4%BD%9C%E7%AC%A6)

### [包裹错误](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/wrap_error.html#%E5%8C%85%E8%A3%B9%E9%94%99%E8%AF%AF)

把错误装箱这种做法也可以改成把它包裹到你自己的错误类型中。

```
use std::error;
use std::num::ParseIntError;
use std::fmt;

type Result<T> = std::result::Result<T, DoubleError>;

#[derive(Debug)]
enum DoubleError {
    EmptyVec,
    // 在这个错误类型中，我们采用 `parse` 的错误类型中 `Err` 部分的实现。
    // 若想提供更多信息，则该类型中还需要加入更多数据。
    Parse(ParseIntError),
}

impl fmt::Display for DoubleError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match *self {
            DoubleError::EmptyVec =>
                write!(f, "please use a vector with at least one element"),
            // 这是一个封装（wrapper），它采用内部各类型对 `fmt` 的实现。
            DoubleError::Parse(ref e) => e.fmt(f),
        }
    }
}

impl error::Error for DoubleError {
    fn source(&self) -> Option<&(dyn error::Error + 'static)> {
        match *self {
            DoubleError::EmptyVec => None,
            // 原因采取内部对错误类型的实现。它隐式地转换成了 trait 对象 `&error:Error`。
            // 这可以工作，因为内部的类型已经实现了 `Error` trait。
            DoubleError::Parse(ref e) => Some(e),
        }
    }
}

// 实现从 `ParseIntError` 到 `DoubleError` 的转换。
// 在使用 `?` 时，或者一个 `ParseIntError` 需要转换成 `DoubleError` 时，它会被自动调用。
impl From<ParseIntError> for DoubleError {
    fn from(err: ParseIntError) -> DoubleError {
        DoubleError::Parse(err)
    }
}

fn double_first(vec: Vec<&str>) -> Result<i32> {
    let first = vec.first().ok_or(DoubleError::EmptyVec)?;
    let parsed = first.parse::<i32>()?;

    Ok(2 * parsed)
}

fn print(result: Result<i32>) {
    match result {
        Ok(n)  => println!("The first doubled is {}", n),
        Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}

```

这种做法会在错误处理中增加一些模板化的代码，而且也不是所有的应用都需要这样做。一些库可以帮你处理模板化代码的问题。

#### [See also:](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types/wrap_error.html#see-also) <a href="#see-also" id="see-also"></a>

[`From::from`](https://rustwiki.org/zh-CN/std/convert/trait.From.html) and [`枚举类型`](https://rustwiki.org/zh-CN/rust-by-example/custom_types/enum.html)\


## [遍历 `Result`](https://rustwiki.org/zh-CN/rust-by-example/error/iter_result.html#%E9%81%8D%E5%8E%86-result) <a href="#bian-li-result" id="bian-li-result"></a>

`Iter::map` 操作可能失败，比如：

```
fn main() {
    let strings = vec!["tofu", "93", "18"];
    let numbers: Vec<_> = strings
        .into_iter()
        .map(|s| s.parse::<i32>())
        .collect();
    println!("Results: {:?}", numbers);
}

```

我们来看一些处理这种问题的策略：

### [使用 `filter_map()` 忽略失败的项](https://rustwiki.org/zh-CN/rust-by-example/error/iter_result.html#%E4%BD%BF%E7%94%A8-filter_map-%E5%BF%BD%E7%95%A5%E5%A4%B1%E8%B4%A5%E7%9A%84%E9%A1%B9) <a href="#shi-yong-filtermap-hu-lve-shi-bai-de-xiang" id="shi-yong-filtermap-hu-lve-shi-bai-de-xiang"></a>

`filter_map` 会调用一个函数，过滤掉为 `None` 的所有结果。

```
fn main() {
    let strings = vec!["tofu", "93", "18"];
    let numbers: Vec<_> = strings
        .into_iter()
        .filter_map(|s| s.parse::<i32>().ok())
        .collect();
    println!("Results: {:?}", numbers);
}

```

### [使用 `collect()` 使整个操作失败](https://rustwiki.org/zh-CN/rust-by-example/error/iter_result.html#%E4%BD%BF%E7%94%A8-collect-%E4%BD%BF%E6%95%B4%E4%B8%AA%E6%93%8D%E4%BD%9C%E5%A4%B1%E8%B4%A5) <a href="#shi-yong-collect-shi-zheng-ge-cao-zuo-shi-bai" id="shi-yong-collect-shi-zheng-ge-cao-zuo-shi-bai"></a>

`Result` 实现了 `FromIter`，因此结果的向量（`Vec<Result<T, E>>`）可以被转换成结果包裹着向量（`Result<Vec<T>, E>`）。一旦找到一个 `Result::Err` ，遍历就被终止。

```
fn main() {
    let strings = vec!["tofu", "93", "18"];
    let numbers: Result<Vec<_>, _> = strings
        .into_iter()
        .map(|s| s.parse::<i32>())
        .collect();
    println!("Results: {:?}", numbers);
}

```

同样的技巧可以对 `Option` 使用。

### [使用 `Partition()` 收集所有合法的值与错误](https://rustwiki.org/zh-CN/rust-by-example/error/iter_result.html#%E4%BD%BF%E7%94%A8-partition-%E6%94%B6%E9%9B%86%E6%89%80%E6%9C%89%E5%90%88%E6%B3%95%E7%9A%84%E5%80%BC%E4%B8%8E%E9%94%99%E8%AF%AF) <a href="#shi-yong-partition-shou-ji-suo-you-he-fa-de-zhi-yu-cuo-wu" id="shi-yong-partition-shou-ji-suo-you-he-fa-de-zhi-yu-cuo-wu"></a>

```
fn main() {
    let strings = vec!["tofu", "93", "18"];
    let (numbers, errors): (Vec<_>, Vec<_>) = strings
        .into_iter()
        .map(|s| s.parse::<i32>())
        .partition(Result::is_ok);
    println!("Numbers: {:?}", numbers);
    println!("Errors: {:?}", errors);
}

```

当你看着这些结果时，你会发现所有东西还在 `Result` 中保存着。要取出它们，需要一些模板化的代码。

```
fn main() {
    let strings = vec!["tofu", "93", "18"];
    let (numbers, errors): (Vec<_>, Vec<_>) = strings
        .into_iter()
        .map(|s| s.parse::<i32>())
        .partition(Result::is_ok);
    let numbers: Vec<_> = numbers.into_iter().map(Result::unwrap).collect();
    let errors: Vec<_> = errors.into_iter().map(Result::unwrap_err).collect();
    println!("Numbers: {:?}", numbers);
    println!("Errors: {:?}", errors);
}

```

