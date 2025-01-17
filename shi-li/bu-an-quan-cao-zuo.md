# 不安全操作

## [不安全操作](https://rustwiki.org/zh-CN/rust-by-example/unsafe.html#%E4%B8%8D%E5%AE%89%E5%85%A8%E6%93%8D%E4%BD%9C) <a href="#bu-an-quan-cao-zuo" id="bu-an-quan-cao-zuo"></a>

在本章一开始，我们借用[官方文档](https://rustwiki.org/zh-CN/rust-by-example/unsafe)的一句话，“在整个代码库（code base，指构建一个软件系统所使用的全部代码）中，要尽可能减少不安全代码的量”。记住这句话，接着我们进入学习！在 Rust 中，不安全代码块用于避开编译器的保护策略；具体地说，不安全代码块主要用于四件事情：

* 解引用裸指针
* 通过 FFI 调用函数（这已经在[之前的章节](https://rustwiki.org/zh-CN/rust-by-example/std_misc/ffi.html)介绍过了）
* 调用不安全的函数
* 内联汇编（inline assembly）

#### [原始指针](https://rustwiki.org/zh-CN/rust-by-example/unsafe.html#%E5%8E%9F%E5%A7%8B%E6%8C%87%E9%92%88) <a href="#yuan-shi-zhi-zhen" id="yuan-shi-zhi-zhen"></a>

原始指针（raw pointer，裸指针）`*` 和引用 `&T` 有类似的功能，但引用总是安全的，因为借用检查器保证了它指向一个有效的数据。解引用一个裸指针只能通过不安全代码块执行。

```
fn main() {
    let raw_p: *const u32 = &10;

    unsafe {
        assert!(*raw_p == 10);
    }
}

```

#### [调用不安全函数](https://rustwiki.org/zh-CN/rust-by-example/unsafe.html#%E8%B0%83%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E5%87%BD%E6%95%B0) <a href="#tiao-yong-bu-an-quan-han-shu" id="tiao-yong-bu-an-quan-han-shu"></a>

一些函数可以声明为不安全的（`unsafe`），这意味着在使用它时保证正确性不再是编译器的责任，而是程序员的。一个例子就是 [`std::slice::from_raw_parts`](https://rustwiki.org/zh-CN/std/slice/fn.from_raw_parts.html)，向它传入指向第一个元素的指针和长度参数，它会创建一个切片。

```
use std::slice;

fn main() {
    let some_vector = vec![1, 2, 3, 4];

    let pointer = some_vector.as_ptr();
    let length = some_vector.len();

    unsafe {
        let my_slice: &[u32] = slice::from_raw_parts(pointer, length);
        
        assert_eq!(some_vector.as_slice(), my_slice);
    }
}

```

`slice::from_raw_parts` 假设传入的指针指向有效的内存，且被指向的内存具有正确的数据类型，我们**必须**满足这一假设，否则程序的行为是未定义的（undefined），于是我们就不能预测会发生些什么了。

[\
](https://rustwiki.org/zh-CN/rust-by-example/compatibility.html)
