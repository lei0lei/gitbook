# 变量绑定

## 变量绑定 <a href="#bian-liang-bang-ding" id="bian-liang-bang-ding"></a>

Rust 通过静态类型确保类型安全。变量绑定可以在声明时说明类型，不过在多数情况下，编译器能够从上下文推导出变量的类型，从而大大减少了类型说明的工作。

使用 `let` 绑定操作可以将值（比如字面量）绑定（bind）到变量。

```rust
fn main() {
    let an_integer = 1u32;
    let a_boolean = true;
    let unit = ();

    // 将 `an_integer` 复制到 `copied_integer`
    let copied_integer = an_integer;

    println!("An integer: {:?}", copied_integer);
    println!("A boolean: {:?}", a_boolean);
    println!("Meet the unit value: {:?}", unit);

    // 编译器会对未使用的变量绑定产生警告；可以给变量名加上下划线前缀来消除警告。
    let _unused_variable = 3u32;

    let noisy_unused_variable = 2u32;
    // 改正 ^ 在变量名前加上下划线以消除警告
}

```

## 可变变量

变量绑定默认是不可变的（immutable），但加上 `mut` 修饰语后变量就可以改变。

```rust
fn main() {
    let _immutable_binding = 1;
    let mut mutable_binding = 1;

    println!("Before mutation: {}", mutable_binding);

    // 正确代码
    mutable_binding += 1;

    println!("After mutation: {}", mutable_binding);

    // 错误！
    _immutable_binding += 1;
    // 改正 ^ 将此行注释掉
}

```

编译器会给出关于变量可变性的详细诊断信息。

## 作用域和遮蔽

变量绑定有一个作用域（scope），它被限定只在一个**代码块**（block）中生存（live）。 代码块是一个被 `{}` 包围的语句集合。另外也允许[变量遮蔽](https://en.wikipedia.org/wiki/Variable_shadowing)（variable shadowing）。

```rust
fn main() {
    // 此绑定生存于 main 函数中
    let long_lived_binding = 1;

    // 这是一个代码块，比 main 函数拥有更小的作用域
    {
        // 此绑定只存在于本代码块
        let short_lived_binding = 2;

        println!("inner short: {}", short_lived_binding);

        // 此绑定*遮蔽*了外面的绑定
        let long_lived_binding = 5_f32;

        println!("inner long: {}", long_lived_binding);
    }
    // 代码块结束

    // 报错！`short_lived_binding` 在此作用域上不存在
    println!("outer short: {}", short_lived_binding);
    // 改正 ^ 注释掉这行

    println!("outer long: {}", long_lived_binding);

    // 此绑定同样*遮蔽*了前面的绑定
    let long_lived_binding = 'a';

    println!("outer long: {}", long_lived_binding);
}

```

## 变量先声明

可以先声明（declare）变量绑定，后面才将它们初始化（initialize）。但是这种做法很少用，因为这样可能导致使用未初始化的变量。

```rust
fn main() {
    // 声明一个变量绑定
    let a_binding;

    {
        let x = 2;

        // 初始化一个绑定
        a_binding = x * x;
    }

    println!("a binding: {}", a_binding);

    let another_binding;

    // 报错！使用了未初始化的绑定
    println!("another binding: {}", another_binding);
    // 改正 ^ 注释掉此行

    another_binding = 1;

    println!("another binding: {}", another_binding);
}

```

编译器禁止使用未经初始化的变量，因为这会产生未定义行为（undefined behavior）。\
\


## 冻结

当数据被相同的名称不变地绑定时，它还会**冻结**（freeze）。在不可变绑定超出作用域之前，无法修改已冻结的数据：

```rust
fn main() {
    let mut _mutable_integer = 7i32;

    {
        // 被不可变的 `_mutable_integer` 遮蔽
        let _mutable_integer = _mutable_integer;

        // 报错！`_mutable_integer` 在本作用域被冻结
        _mutable_integer = 50;
        // 改正 ^ 注释掉上面这行

        // `_mutable_integer` 离开作用域
    }

    // 正常运行！ `_mutable_integer` 在这个作用域没有冻结
    _mutable_integer = 3;
}

```

\
\
