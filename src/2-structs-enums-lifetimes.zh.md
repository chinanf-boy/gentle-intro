# 结构{structs},枚举{enums}和匹配{match}

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Rust 喜欢 move 它, move 它](#rust-%E5%96%9C%E6%AC%A2-move-%E5%AE%83-move-%E5%AE%83)
- [变量的范围](#%E5%8F%98%E9%87%8F%E7%9A%84%E8%8C%83%E5%9B%B4)
- [元组](#%E5%85%83%E7%BB%84)
- [结构{Structs}](#%E7%BB%93%E6%9E%84structs)
- [生命周期{Lifetimes}开始咬人啦](#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9Flifetimes%E5%BC%80%E5%A7%8B%E5%92%AC%E4%BA%BA%E5%95%A6)
- [特点{Traits}](#%E7%89%B9%E7%82%B9traits)
- [示例: 遍历浮点范围的迭代器](#%E7%A4%BA%E4%BE%8B-%E9%81%8D%E5%8E%86%E6%B5%AE%E7%82%B9%E8%8C%83%E5%9B%B4%E7%9A%84%E8%BF%AD%E4%BB%A3%E5%99%A8)
- [泛型函数](#%E6%B3%9B%E5%9E%8B%E5%87%BD%E6%95%B0)
- [简单的枚举](#%E7%AE%80%E5%8D%95%E7%9A%84%E6%9E%9A%E4%B8%BE)
- [枚举的全部荣耀](#%E6%9E%9A%E4%B8%BE%E7%9A%84%E5%85%A8%E9%83%A8%E8%8D%A3%E8%80%80)
- [关于匹配的 更多](#%E5%85%B3%E4%BA%8E%E5%8C%B9%E9%85%8D%E7%9A%84-%E6%9B%B4%E5%A4%9A)
- [闭包{Closures}](#%E9%97%AD%E5%8C%85closures)
- [三种迭代器](#%E4%B8%89%E7%A7%8D%E8%BF%AD%E4%BB%A3%E5%99%A8)
- [具有动态数据的结构](#%E5%85%B7%E6%9C%89%E5%8A%A8%E6%80%81%E6%95%B0%E6%8D%AE%E7%9A%84%E7%BB%93%E6%9E%84)
- [泛型结构](#%E6%B3%9B%E5%9E%8B%E7%BB%93%E6%9E%84)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Rust 喜欢 move 它, move 它

我想稍微回退一下，给你看一些惊奇的东西:

```rust
// move1.rs
fn main() {
    let s1 = "hello dolly".to_string();
    let s2 = s1;
    println!("s1 {}", s1);
}
```

我们得到以下错误:

```
error[E0382]: use of moved value: `s1`
 --> move1.rs:5:22
  |
4 |     let s2 = s1;
  |         -- value moved here
5 |     println!("s1 {}", s1);
  |                      ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`,
  which does not implement the `Copy` trait
```

Rust 与其他语言有不同的行为。 其他语言的变量总是会`引用{references}` (如 Java 或 Python)，`s2`成为对引用到`s1`的字符串对象的又一个引用。 在 C ++ 中，`s1`是一种`值{value}`，它会 _复制_ 到`s2`。 但 Rust 会移动该值。 它没有看到`字符串{strings}`是具有 可复制性的 ("没有实现 Copy trait"，也就是相应的复制方法，它并没有)。

我们不会看到像数字这样的"原始"类型不能复制，因为它们只是数值; 他们被允许复制，因为他们复制成本堪称便宜。 但，`String`是已经分配了包含"Hello dolly"的内存，而要复制这内容，将涉及分配更多内存还要复制`字符{char}`。Rust 才不会静悄悄地做这样的事情。

考虑一个`String`，它包含"Moby-Dick"的全文。 它不是一个很大的结构，只有文本的内存地址，以及它的大小以及分配块的大小。要复制这`String`会是昂贵的，因为该内存分配在堆上，和复制品也需要自己的内容分配区。

```
    String
    | addr | ---------> Call me Ishmael.....
    | size |                    |
    | cap  |                    |
                                |
    &str                        |
    | addr | -------------------|
    | size |

    f64
    | 8 bytes |
```

第二个值是一个字符串切片 (`&str`)，它与第一个字符串指向相同的内存，再加个大小 - 它仅仅只是(地址)名字。便宜复制!

第三个值是一个`f64`- 只有 8 个字节。 它不涉及任何其他内存，所以它的复制和移动一样便宜。

`复制{Copy}`值只能通过它们在内存中的表示来定义，而当 Rust 拷贝时，它只是在其他地方复制这些字节。类似地, 一个没有`复制{Copy}`的`值{value}`也是 _只是移动了{moved}_。 与 C ++ 不同的是, Rust 在复制和移动方面没有自作聪明。

> 译者: 对 具有引用的变量 隐形`移动{move}`该变量, 在 Rust 是错误的。

用函数调用重写，将显示完全相同的错误:

```rust
// move2.rs

fn dump(s: String) {
    println!("{}", s);
}

fn main() {
    let s1 = "hello dolly".to_string();
    dump(s1);
    println!("s1 {}", s1); // <---error: 'value used here after move'
}
```

在这里，你有一个选择。 您可以传递对该字符串的`引用{&}`，或者使用它的`clone`方法来明确拷贝。一般来说，第一种是更好的方法。

```rust
fn dump(s: &String) {
    println!("{}", s);
}

fn main() {
    let s1 = "hello dolly".to_string();
    dump(&s1);
    println!("s1 {}", s1);
}
```

错误消失。 但你很少看到一个简朴`String`像这样的引用，因为传递一个字符串文字是非常丑陋的， _还要_ 涉及创建一个临时字符串。

```rust
    dump(&"hello world".to_string());
```

因此，声明该函数的最佳方式是:

```rust
fn dump(s: &str) {
    println!("{}", s);
}
```

那么， `dump(&s1)` 和 `dump("hello world")`这两种情况都会 好好工作。 (这里就是`Deref`起的作用, Rust 会为你转换`&String`至`&str`。 )

总而言之，`非复制{non-Copy}`的分配工作，会将 值 从一个位置`移动{move}`到另一个位置。不然的话，Rust 将被迫 _隐式_ 做一个`副本{copy}`，并打破 Rust 本身 _明确分配_ 的承诺。

## 变量的范围

所以，经验法则是更愿意保留对原始数据的`引用{&}` - 以此来`"借用{borrow}"`它。

但，一个引用必须 _不能_ 长命过`拥有人{owner}`!

首先, Rust 是一个 _块范围的{block-scoped}_ 语言。 变量仅在其代码块持续时间内存在:

```rust
{
    let a = 10;
    let b = "hello";
    {
        let c = "hello".to_string();
        // a,b 和 c 有
    }
    //  c 没有了
    // a,b  有
    for i in 0..a {
        let b = &b[1..];
        // 原来的 b 不再可见 - 它被罩住了。
    }
    //  b 没有了
    // i 没有了
}
```

循环变量 (如`i`) 有点不同，它们只在循环代码块中可见。 创建一个使用相同名称的新变量并不是一个错误 (`'覆盖'`) ，但它可能会造成混淆。

当一个变量'超出范围'，那么它会 _扔掉了{dropped}_。 任何使用的内存都会被回收，而该变量的其他 _资源{resources}_ 将返回给系统 - 例如，扔掉一个`文件{File}`，就等于关闭它。 这是一件好事。不用的资源在不需要时立即回收。

(另一个 Rust 的特色问题是，变量看起来可能在范围内，但其值已经是`移动{move}`了的。 )

这里有一个`rs1`，其引用到`tmp`值, 而引用只在其区块`{}`内存在:

```rust
01 // ref1.rs
02 fn main() {
03    let s1 = "hello dolly".to_string();
04    let mut rs1 = &s1;
05    {
06        let tmp = "hello world".to_string();
07        rs1 = &tmp; // <==
08    }
09    println!("ref {}", rs1);
10 }
```

我们先`借用{borrow}`了`s1`值，然后再借用`tmp`值。但`tmp`在(**05~08**)区块之外就被扔掉了!

```
error: `tmp` does not live long enough
  --> ref1.rs:8:5
   |
7  |         rs1 = &tmp;
   |                --- borrow occurs here
8  |     }
   |     ^ `tmp` dropped here while still borrowed
9  |     println!("ref {}", rs1);
10 | }
   | - borrowed value needs to live until here
```

`tmp`哪里去了? 走了,死了,回到了天空中的堆中，故名: _扔掉了{dropeed}_。 Rust 把你从 **C** 的可怕的'悬挂指针'问题中拯救出来，问题具体就是:一个指向陈旧数据的引用。

> 在 区块中, `rs1`-指向-> `&tmp`, 但在区块结束后, tmp 整个都被 _扔掉了{dropped}_ , 这个时候 `rs1` 就变成一个指向陈旧(已扔掉)数据的引用。

## 元组

有时，从函数返回多个值，会非常有用。元组就是一个方便的解决方案:

```rust
// tuple1.rs

fn add_mul(x: f64, y: f64) -> (f64,f64) {
    (x + y, x * y)
}

fn main() {
    let t = add_mul(2.0,10.0);

    // 可以 调试打印
    println!("t {:?}", t);

    // 可以 给出值'索引'
    println!("add {} mul {}", t.0,t.1);

    // 可以 _提取_ 值
    let (add,mul) = t;
    println!("add {} mul {}", add,mul);
}
// t (12, 20)
// add 12 mul 20
// add 12 mul 20
```

元组能包含 _不同_ 类型，这也是它与数组的主要区别。

```rust
let tuple = ("hello", 5, 'c');

assert_eq!(tuple.0, "hello");
assert_eq!(tuple.1, 5);
assert_eq!(tuple.2, 'c');
```

下面出现在一些`迭代器{Iterator}`方法。 `enumerate`就像同名的 Python 生成器(generator) 一样:

```rust
    for t in ["zero","one","two"].iter().enumerate() {
        print!(" {} {};",t.0,t.1);
    }
    //  0 zero; 1 one; 2 two;
```

`zip`会将两个迭代器，组合成一个 _包含来自两者的值的元组_ 的迭代器:

```rust
    let names = ["ten","hundred","thousand"];
    let nums = [10,100,1000];
    for p in names.iter().zip(nums.iter()) {
        print!(" {} {};", p.0,p.1);
    }
    //  ten 10; hundred 100; thousand 1000;
```

## 结构{Structs}

元组很方便，但是要追踪每个部分的含义，`t.1`的这种写法不够直接与明了。

Rust _结构_ 就不同，它包含命名 _字段{fields}_ :

```rust
// struct1.rs

struct Person {
    first_name: String,
    last_name: String
}

fn main() {
    let p = Person {
        first_name: "John".to_string(),
        last_name: "Smith".to_string()
    };
    println!("person {} {}", p.first_name,p.last_name);
}
```

虽然，不应该假定任何特定的内存布局，但是结构体的值将在内存中相邻放置，因为编译器是要高效，而不是节省大小的手段，来组织内存，哦，还有存在填充的可能。

初始化这个结构有点笨拙，所以我们想要把构造一个`Person`，融入其自身的函数。通过把它放进`impl`块, 这初始函数可以做成`Person`的一个 _关联函数_ :

```rust
// struct2.rs

struct Person {
    first_name: String,
    last_name: String
}

impl Person {

    fn new(first: &str, name: &str) -> Person {
        Person {
            first_name: first.to_string(),
            last_name: name.to_string()
        }
    }

}

fn main() {
    let p = Person::new("John","Smith");
    println!("person {} {}", p.first_name,p.last_name);
}
```

这个`new`名字，没有什么魔力或其他东西，随你喜欢。要注意的是，它使用类似 C ++ 进行访问 - 使用双冒号的符号`::`。

下面是个`Person` _方法_，需要一个 _自我引用{reference self}_ 参数:

```rust
impl Person {
    ...

    fn full_name(&self) -> String {
        format!("{} {}", self.first_name, self.last_name)
    }

}
...
    println!("fullname {}", p.full_name());
// fullname John Smith
```

明确使用该`self`，并作为`引用`传递。 (你可以把`&self`想成`self: &Person`简写。 )

还有，关键字`Self`(自身：注意首大写)指的是结构类型 - 你可以在脑海中用`Person`替换掉`Self`:

```rust
    fn copy(&self) -> Self {
        Self::new(&self.first_name,&self.last_name)
    }
```

方法可以允许修改数据, 用到 _可变的自我{mutable self}_ 参数:

```rust
    fn set_first_name(&mut self, name: &str) {
        self.first_name = name.to_string();
    }
```

当使用简单的`self`参数时，数据会 _移动{move}_：

```rust
    fn to_tuple(self) -> (String,String) {
        (self.first_name, self.last_name)
    }
```

(试试使用`&self`- 结构不会在没有过争斗的情况下，放开数据!)

注意，`v.to_tuple()`被调用之后，`v`已经移动并且不再可用。

总结:

- 没有`self`相关参数: 您可以将函数与结构关联，如`new`"构造函数"。
- `&self`参数: 可以使用结构体的值，但不能改变它们。
- `&mut self`参数: 可以修改这些值。
- `self`参数: 将消耗值，因它移动了。

如果您尝试对`Person`执行一个调试打印，你会得到一个信息错误:

```
error[E0277]: the trait bound `Person: std::fmt::Debug` is not satisfied
  --> struct2.rs:23:21
   |
23 |     println!("{:?}", p);
   |                     ^ the trait `std::fmt::Debug` is not implemented for `Person`
   |
   = note: `Person` cannot be formatted using `:?`; if it is defined in your crate,
    add `#[derive(Debug)]` or manually implement it
   = note: required by `std::fmt::Debug::fmt`
```

编译器提供建议，所以我们放了`#[derive(Debug)]`在`Person`前面，现在有实用的输出:

```
Person { first_name: "John", last_name: "Smith" }
```

该 _指示{directive}_ 注释会让编译器对`Person`，**生成**一个 `Debug` 实现, 简单且有效。对于你的结构来说，这是一个很好的事情，简单加上一句注释，它们就可以打印出来。

> 译者:该指令注释，是有关 Rust 宏方面的知识，若想了解[更多](https://kaisery.github.io/trpl-zh-cn/ch19-06-macros.html#a%E5%AE%8F)

这是最后的小程序:

```rust
// struct4.rs
use std::fmt;

#[derive(Debug)]
struct Person {
    first_name: String,
    last_name: String
}

impl Person {

    fn new(first: &str, name: &str) -> Person {
        Person {
            first_name: first.to_string(),
            last_name: name.to_string()
        }
    }

    fn full_name(&self) -> String {
        format!("{} {}",self.first_name, self.last_name)
    }

    fn set_first_name(&mut self, name: &str) {
        self.first_name = name.to_string();
    }

    fn to_tuple(self) -> (String,String) {
        (self.first_name, self.last_name)
    }
}

fn main() {
    let mut p = Person::new("John","Smith");

    println!("{:?}", p);

    p.set_first_name("Jane");

    println!("{:?}", p);

    println!("{:?}", p.to_tuple());
    // p has now moved.

}
// Person { first_name: "John", last_name: "Smith" }
// Person { first_name: "Jane", last_name: "Smith" }
// ("Jane", "Smith")
```

## 生命周期{Lifetimes}开始咬人啦

通常，结构体包含值，但通常它们还需要包含`引用{&}`。 假设我们想在一个结构中放置一个`字符串切片{&str}`，而不是一个字符串值。

```rust
// life1.rs

#[derive(Debug)]
struct A {
    s: &str
}

fn main() {
    let a = A { s: "hello dammit" };

    println!("{:?}", a);
}
```

```
error[E0106]: missing lifetime specifier
 --> life1.rs:5:8
  |
5 |     s: &str
  |        ^ expected lifetime parameter
```

为了理解编译器的投诉，你必须从 Rust 的角度看问题。

如果不知道一个‘引用’的生命周期，是不允许你存储它。 所有`引用{&}`都是从某个值那里`借用{borrowed}`的，而且所有的`值`都是有`生命周期{lifetimes}`的。`引用的生命周期不能长于该值的生命周期`。Rust 不能允许这种 `引用可能突然失效` 的情况。

> 译者: 这时，你可以停一停了，好好想想上面这段话的含义，且自行概略如下问题的答案。
> 问：值 与 引用 的关系？

现在,字符串切片是从 _字符串常量_ 借用的，像"hello"或是`String`值。 字符串常量在整个程序期间都存在，也称为"静态{static}"生命周期。

所以，下面写法是有效的 - 我们向 Rust 保证字符串切片，总是指向这`静态{static}`字符串:

```rust
// life2.rs

#[derive(Debug)]
struct A {
    s: &'static str
}

fn main() {
    let a = A { s: "hello dammit" };

    println!("{:?}", a);
}
// A { s: "hello dammit" }
```

确实，这不是最 _漂亮_ 符号，但有时丑，是精确的必要代价。

这也可以用来指明，从函数返回的字符串切片:

```rust
fn how(i: u32) -> &'static str {
    match i {
    0 => "none",
    1 => "one",
    _ => "many"
    }
}
```

这是静态字符串的特殊情况，但应该严格对待。

不过嘛，我们也可以指定`引用{&}`的生命周期，与结构本身 _至少一样长_ 。

```rust
// life3.rs

#[derive(Debug)]
struct A <'a> { // 注意写法
    s: &'a str
}

fn main() {
    let s = "I'm a little string".to_string(); // string
    let a = A { s: &s }; // <== 结构

    println!("{:?}", a);
}
```

`生命周期{Lifetimes}`通常被称为'a','b'等，不过您也可以写'我{me}'，随你喜欢，自己知道且简洁就好。

之后看看`main`函数的内容，我们的`a`结构和`s`字符串受到严格的合同约束: `a`借用了`s`，并且不能长命过`s`。

接下来，用这个 `A` 结构体定义，我们想写一个函数，它返回一个`A`值:

```rust
fn makes_a() -> A {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

但 `A` 需要一个生命周期 - "要预期的生命周期参数{expected lifetime parameter}":

```
  = help: this function's return type contains a borrowed value,
   but there is no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

`rustc`提供建议，所以我们遵循它:

```rust
fn makes_a() -> A<'static> {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

而现在的错误是

```
8 |      A { s: &string }
  |              ^^^^^^ does not live long enough
9 | }
  | - borrowed value only lives until here
```

这是无法安全工作的，因为`string`将在函数结束时被删除，并且引用不可以长命过`string`。

您可以将生命周期参数，视为一个值类型的一部分，会有所帮助。

有时候，结构中包含一个值 _和_ 从该值借用的引用，看，似乎是个好主意。 但，这基本上是不可能的，因为结构必须是 _可移动的_，而任何移动都将使引用无效。其实也没有必要这样做 - 例如，如果你的结构有一个字符串字段-string，并且还想要提供切片，那么，它完全可以保留索引，再加个方法，来生成实际的切片。

## 特点{Traits}

> 译者: Traits 的 中文意思名字有好几个，但，本质是: 定义结构的一系列行为/方法。

请注意 Rust 不会拼写`struct` _类_。 关键字`类`在其他语言中是如此超载，意味着，它有效地击毙了原真的想法。

让我们这样说吧: Rust 结构不能 _继承_ 来自其他结构; 他们都是独特的类型。 没有 _sub-typing{子类型}_ 。他们都是愚蠢的数据.

所以，一个类型之间的关系又应该怎样 _做_ 呢? 这正是 _Traits_ 的作用。

`rustc`经常谈到`实现{implementing} X 的特点{trait}`，所以现在恰是讨论 _Traits_ 的时候了。

这里有一个定义 _Traits_ 的例子, 帮特定类型去 _实现_ 它。

```rust
// trait1.rs

trait Show {
    fn show(&self) -> String;
}

impl Show for i32 {
    fn show(&self) -> String {
        format!("four-byte signed {}", self)
    }
}

impl Show for f64 {
    fn show(&self) -> String {
        format!("eight-byte float {}", self)
    }
}

fn main() {
    let answer = 42;
    let maybe_pi = 3.14;
    let s1 = answer.show();
    let s2 = maybe_pi.show();
    println!("show {}", s1);
    println!("show {}", s2);
}
// show four-byte signed 42
// show eight-byte float 3.14
```

它太酷了; 我们增加了`i32`和`f64`两者泛型的 _一种新方法_ !

熟悉 Rust ，就要学习标准库的基本 trait (他们倾向于成群结队)。

非常普遍的有`Debug`。 我们给`Person`一个方便的默认实现，`#[derive(Debug)]`，但，假如我们想要一个完整的`Person`-Debug 实现:

```rust
use std::fmt;

impl fmt::Debug for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.full_name())
    }
}
...
    println!("{:?}", p);
    // John Smith
```

`write!`是一个非常有用的宏 - 内部的`f`是实现了`Write`的东西。 (这也适用于`File`- 甚至是一个`String`. )

而，`显示{Display}`控制如何使用"{}"打印值，当然也要有对应的实现，就像`Debug`一样。 作为一个有用的副作用，任何实现了`Display`的，其`ToString`也自动可用。 所以，如果我们实现了`Person`的`Display`, `p.to_string()`也可用了。

`Clone`定义了`clone`方法，可简单用"#[deriv(Clone)]"进行定义，如果要所有的字段都实现`Clone`的话。

## 示例: 遍历浮点范围的迭代器

之前，我们已经遇到范围表达 (`0..n`) ，但它们不适用于浮点值。 ( _强行_ 去做，最终你会得到一个无趣的 1.0。 )

回想一下，迭代器的非正式定义; 它是一个带有结构体，具有一个可能会返回`Some`或`None`的`next`方法。 在这个过程中，迭代器本身被修改，它保持迭代的状态 (如 next 索引等等)。 迭代的数据通常不会改变, (但，可以参阅`Vec::drain`，对于修改其数据的有趣迭代器)。

这里是正式的定义: [迭代器(Iterator) trait](https://doc.rust-lang.org/std/iter/trait.Iterator.html).

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    ...
}
```

我们在这里，看到了`Iterator` trait 的[关联类型{associated type}](https://doc.rust-lang.org/stable/book/associated-types.html)。这个 trait 必须与任意类型合作，所以你必须以某种方式指定返回类型。 方法`next`可以在不使用特定类型的情况下编写 - 而是通过`Self`引用该类型参数的`Item`。

`f64`的迭代器 trait ，是写入`Iterator<Item=f64>`，它可以理解为："迭代器的关联类型 Item 设置为 f64"。

至于，`...`表达语句指的是`Iterator`所 _提供的方法_ 。 你只需要定义`Item`和`next`，那该表达语句就可为你所用。

```rust
// trait3.rs

struct FRange {
    val: f64,
    end: f64,
    incr: f64
}

fn range(x1: f64, x2: f64, skip: f64) -> FRange {
    FRange {val: x1, end: x2, incr: skip}
}

impl Iterator for FRange {
    type Item = f64;

    fn next(&mut self) -> Option<Self::Item> {
        let res = self.val;
        if res >= self.end {
            None
        } else {
            self.val += self.incr;
            Some(res)
        }
    }
}


fn main() {
    for x in range(0.0, 1.0, 0.1) {
        println!("{} ", x);
    }
}
```

而相当凌乱的结果是

```
0
0.1
0.2
0.30000000000000004
0.4
0.5
0.6
0.7
0.7999999999999999
0.8999999999999999
0.9999999999999999
```

这是因为 0.1 不能精确表示为一个浮点数，所以需要一些格式化帮助。 更换成`println!`：

```rust
println!("{:.1} ", x);
```

我们得到更干净的输出 (这个[格式](https://doc.rust-lang.org/std/fmt/index.html)的意思是'小数点后一位小数'。 ) 所有默认的迭代器方法都是可用，所以，我们可以将这些值收集到一个`向量{Vec}`中，通过 `map`方法来使用它们。等等。

```rust
    let v: Vec<f64> = range(0.0, 1.0, 0.1).map(|x| x.sin()).collect();
```

## 泛型函数

我们需要一个函数，来抛出实现了`Debug`的任何值。 以下是对泛型函数的第一次尝试，我们可以在其中传递一个 _任何_ 值类型的引用。`T`是一个类型参数，需要在函数名称后面声明:

```rust
fn dump<T> (value: &T) {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
```

但是, Rust 显然对这种泛型类型`T`一无所知:

```
error[E0277]: the trait bound `T: std::fmt::Debug` is not satisfied
...
   = help: the trait `std::fmt::Debug` is not implemented for `T`
   = help: consider adding a `where T: std::fmt::Debug` bound
```

为了这个工作, 需要告知 Rust ，这个`T`要实现`Debug`了的!

```rust
fn dump<T> (value: &T)
where T: std::fmt::Debug {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
// value is 42
```

Rust 泛型函数需要 _Traits bounds_ 类型 - 我们在这里说，"T 是实现了 Debug 的任意类型"。 `rustc`是非常有用的，并且确切地说明需要提供什么界限(bound)。

> 译者: Traits bounds (特征界限)，本质上说: 参数的类型，约束 在，要是实现了对应的 Trait。

现在，Rust 知道这个`T`的 特征界限，它可以给你敏锐的编译器消息:

```rust
struct Foo {
    name: String
}

let foo = Foo{name: "hello".to_string()};

dump(&foo)
```

错误是："`Foo` 没有实现 `std::fmt::Debug` trait"。

函数在动态语言中已经是泛型的，因为值会带有它们的实际类型，并且类型检查会在运行时发生 - 或者惨败。 对于较大的程序，我们确实想在编译时想知道问题! 这些语言的程序员不应平静地坐在编译器的错误之中，而必须处理程序运行时，才会出现的问题。 墨菲定律，告诉我们这些问题往往会发生在 最不方便/灾难性 的时刻。

平方数的操作函数是泛型的: `x * x`要适用整数，浮点数和任意知道关于乘法运算符`*`的类型。 但是，其类型界限又是什么?

```rust
// gen1.rs

fn sqr<T> (x: T) -> T {
    x * x
}

fn main() {
    let res = sqr(10.0);
    println!("res {}",res);
}
```

第一个问题是 Rust 不知道`T`可以做乘法:

```
error[E0369]: binary operation `*` cannot be applied to type `T`
 --> gen1.rs:4:5
  |
4 |     x * x
  |     ^
  |
note: an implementation of `std::ops::Mul` might be missing for `T`
 --> gen1.rs:4:5
  |
4 |     x * x
  |     ^
```

遵循编译器的建议，让我们使用[这个 Traits](https://doc.rust-lang.org/std/ops/trait.Mul.html)限制该类型参数，这个 Traits 用来实现乘法运算符`*`:

```rust
fn sqr<T> (x: T) -> T
where T: std::ops::Mul {
    x * x
}
```

仍，不起作用:

```
rror[E0308]: mismatched types
 --> gen2.rs:6:5
  |
6 |     x * x
  |     ^^^ expected type parameter, found associated type
  |
  = note: expected type `T`
  = note:    found type `<T as std::ops::Mul>::Output`
```

`rustc`是说有关`x * x`的类型，是`T::Output`关联类型，而不是`T`。 实际上，`x * x`与`x`类型没有道理是相同的，例如，两个向量的积是一个标量。

```rust
fn sqr<T> (x: T) -> T::Output
where T: std::ops::Mul {
    x * x
}
```

现在的错误是:

```
error[E0382]: use of moved value: `x`
 --> gen2.rs:6:7
  |
6 |     x * x
  |     - ^ value used here after move
  |     |
  |     value moved here
  |
  = note: move occurs because `x` has type `T`, which does not implement the `Copy` trait
```

所以，我们需要进一步限制类型!

```rust
fn sqr<T> (x: T) -> T::Output
where T: std::ops::Mul + Copy {
    x * x
}
```

(终于) 起作用了。要冷静地倾听编译器，每次都会让你更接近原力点，... 终会流畅编译。

确实, 在 C ++ 中，_是_ 更简单一点:

```cpp
template <typename T>
T sqr(x: T) {
    return x * x;
}
```

但， (说实话) C ++ 在这里采用了牛仔策略。C ++ 的`模板{template}`错误很不好，因为，编译器都知道的所有， (最终) 是某些操作符或方法没有被定义。 C ++ 委员会知道这是一个问题，所以他们正在努力让[concepts](<https://en.wikipedia.org/wiki/Concepts_(C%2B%2B)>)工作起来，这与 Rust 中的`trait约束类型`参数非常相似。

Rust 泛型函数，一开始可能看起来有点难接受，但是，显式，就是明确定义，就能确切地知道可以安全地提供哪种值。

这些函数是 _单态{monomorphic}_ 调用的，与 _多态{polymorphic}_ 合作。 函数的主体都会为每个 唯一类型 分别编译的。通过多态函数，相同的机器代码可以与每种匹配类型一起工作， 动态地 _调度{dispatching}_ 正确的方法。

`Monomorphic`生成更快的代码，专用于特定类型，并且，常是 _内联{inlined}_ 起来。所以，当`sqr(x)`被看到，它会被有效地用`x * x`取代。 缺点是，大的泛型函数为每一种可能导致的类型，产生大量的代码，引起 _代码膨胀_。但与往常一样，总是有折衷的方式; 有经验的人学会为工作，做出正确的选择。

## 简单的枚举

`枚举{enums}`类型具有一些确定的值。 例如，一个方向只有四个可能的值。(上下左右)

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right
}
...
    // `start` is type `Direction`
    let start = Direction::Left;
```

可以在枚举上定义方法，就像结构一样。 该`match`表达语句是处理`enum`值的基本方式。

```rust
impl Direction {
    fn as_str(&self) -> &'static str {
        match *self { // *self 有 Direction 类型
            Direction::Up => "Up",
            Direction::Down => "Down",
            Direction::Left => "Left",
            Direction::Right => "Right"
        }
    }
}
```

标点符号很重要。 注意`match`后面的`self`之前的`*`。 很容易忘记，因为 Rust 经常会推断它 (我们说`self.first_name`，而不是`(*self).first_name`)。 但是，`匹配{matching}`是更精确的工作。若将它排除在外，会产生一大堆消息，这些消息可归结为这种类型的不匹配:

```
   = note: expected type `&Direction`
   = note:    found type `Direction`
```

这是因为`self`有`&Direction`类型，所以我们必须投入`*` _遵循_ 该值。

像结构一样，枚举可以实现 traits，我们的朋友`#[derive(Debug)]`，可以添加到`Direction`:

```rust
        println!("start {:?}",start);
        // start Left
```

所以，`as_str`方法并不是真的必要，因为我们总是可以从`Debug`得到名字。 (但`as_str`是 _不分配{not allocate}_ ，这可能很重要。)

你不应该在这里，假设任何特定的顺序 - 这里没有默许的"起始"整数值。

这里有一个方法，来定义每个`方向`值的'后继者'。 非常方便的*通配符用法*，将枚举名称暂时放入方法上下文中：

```rust
    fn next(&self) -> Direction {
        use Direction::*; // <===
        match *self {
            Up => Right,
            Right => Down,
            Down => Left,
            Left => Up
        }
    }
    ...

    let mut d = start;
    for _ in 0..8 {
        println!("d {:?}", d);
        d = d.next();
    }
    // d Left
    // d Up
    // d Right
    // d Down
    // d Left
    // d Up
    // d Right
    // d Down
```

结果就是，这个特定的,任意的顺序中，各个方向一直循环。 它 (事实上)是非常简单的状态机器。

这些枚举值，无法比较:

```
assert_eq!(start, Direction::Left);

error[E0369]: binary operation `==` cannot be applied to type `Direction`
  --> enum1.rs:42:5
   |
42 |     assert_eq!(start, Direction::Left);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
note: an implementation of `std::cmp::PartialEq` might be missing for `Direction`
  --> enum1.rs:42:5
```

解决办法就是，在`enum Direction`前面加上`#[derive(Debug,PartialEq)]`。

这是一个重点 - Rust 用户定义的类型一开始就是这么新鲜和朴素。

你通过实现共同的 traits ，给予他们合理的默认行为。这也适用于结构 - 如果你要求 Rust 为一个结构体 _derive_ `PartialEq`，它会做出同样合理的事情，但要，假设所有的字段都实现它，并构建了一个对照结果。 如果不是这样，或者你想重新定义相等性质，那么你可以明确地自定义`PartialEq`。

Rust 也有'C 风格的枚举':

```rust
// enum2.rs

enum Speed {
    Slow = 10,
    Medium = 20,
    Fast = 50
}

fn main() {
    let s = Speed::Slow;
    let speed = s as u32;
    println!("speed {}", speed);
}
```

它们用一个整数值进行初始化，并可以通过类型转换(as)，将其转换为整数。

你只需要给名字一个值，然，每次自动增加一个值:

```rust
enum Difficulty {
    Easy = 1,
    Medium,  // is 2
    Hard   // is 3
}
```

顺便说一下，枚举内字段的'名字'一词太模糊了，就像一直在说'物质'。 这里的合适名词，是 _变种{variant}_ - `Speed`枚举有`Slow`，`Medium`和`Fast`的变种。

这些枚举 _确_ 有一个自然的顺序，但你必须问得好。在`enum Speed`前面放置`#[derive(PartialEq,PartialOrd)]`之后，`Speed::Fast > Speed::Slow`和`Speed::Medium != Speed::Slow`才是对的。

## 枚举的全部荣耀

完全形式的 rust 类似于类固醇上的 C 联盟，like a Ferrari compared to a Fiat Uno。考虑以 类型-安全的方式 存储不同值的问题。

```rust
// enum3.rs

#[derive(Debug)]
enum Value {
    Number(f64),
    Str(String),
    Bool(bool)
}

fn main() {
    use Value::*;
    let n = Number(2.3);
    let s = Str("hello".to_string());
    let b = Bool(true);

    println!("n {:?} s {:?} b {:?}", n,s,b);
}
// n Number(2.3) s Str("hello") b Bool(true)
```

同样，这个枚举只能包含这些值的 _一个_ ;其大小将是 最大变体 的大小。

到目前为止，并不是真正的超级跑车，虽然枚举知道如何打印出来是很酷的。 但，他们也知道它们包含的 _哪一种_ 值，和 _还有_ `match`的超级力量:

```rust
fn eat_and_dump(v: Value) {
    use Value::*;
    match v {
        Number(n) => println!("number is {}", n),
        Str(s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}
....
eat_and_dump(n);
eat_and_dump(s);
eat_and_dump(b);
//number is 2.3
//string is 'hello'
//boolean is true
```

(而这就是`Option`和`Result`的本质 - 都是枚举。)

我们喜欢这个`eat_and_dump`函数，但我们希望将该值作为引用传递，因为当前`移动{move}`了，并且该值被'吃掉'了:

```rust
fn dump(v: &Value) {
    use Value::*;
    match *v {  // type of *v is Value
        Number(n) => println!("number is {}", n),
        Str(s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}

error[E0507]: cannot move out of borrowed content
  --> enum3.rs:12:11
   |
12 |     match *v {
   |           ^^ cannot move out of borrowed content
13 |     Number(n) => println!("number is {}",n),
14 |     Str(s) => println!("string is '{}'",s),
   |         - hint: to prevent move, use `ref s` or `ref mut s`
```

这次, 你无法处理借用引用。 Rust 不会让你 _提取_ 包含在原始值中的字符串。 它没有抱怨`Number`，因为它很高兴复制`f64`，但是`String`是没有实现`Copy`的。

我之前提到过，`match`对精确 _类型_ 是挑剔的，在这里，我们按照提示进行操作(加 `ref`); 现在，我们只是借用对包含字符串的引用。

> 译者: [据我了解](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=7d311a4df2f4f8face8be0c7425ae1f7)，现 rustc 编译器已不再提示这个示例的错误，因它自行修正了此错误。2019.2.24

```rust
fn dump(v: &Value) {
    use Value::*;
    match *v {
        Number(n) => println!("number is {}", n),
        Str(ref s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}
    ....

    dump(&s);
    // string is 'hello'
```

在我们继续前进之前，感受下 Rust 编译成功的欣快感，也让我们暂停一下。`rustc`在生成足够上下文，以供人类使用非常优秀的 _修正_ 错误，却不一定要 _理解_ 错误。现在我们来理解下。

这个问题是 `match` 的正确性，以及 借用检查者阻止任何违反规则的企图的结合。 其中一条规则是你不能抽出所属某种拥有类型的值。 C ++ 的一些知识在这里是一个障碍，因为 C ++ 会用复制它的方式绕过这个问题，甚至还 _说得通_ 。

如果你尝试从一个 Vec 中抽出一个字符串，你会得到完全相同的错误，也就是`*v.get(0).unwrap()` (因为索引返回的是引用，所以使用`*`。 )，而它不会让你这样做。 (有时在这种情况下，`clone`并不是一个坏的解决方案。)

(顺便一提，正是出于这个原因，`v[0]`不适用于像字符串这样的非可复制值。 你必须借用`&v[0]`或使用 `v[0].clone()` 复制来达到目的)

至于`match`，你可以看到`Str(s)=>`，其作为`Str(s: String)=>`的简称。 局部变量(通常称为一个 _绑定_ 值 ) 被创建。 当你吃掉一个值，并提取其内容时，通常推断的类型是可行，但我们真正需要的是`s: &String`，而`ref`暗示，可以确保这一点: 我们只是想借用该字符串。

在这里，我们确实想提取该字符串，并且不关心之后的枚举值。 `_`像往常一样会匹配任何东西。

```rust
impl Value {
    fn to_str(self) -> Option<String> {
        match self {
        Value::Str(s) => Some(s),
        _ => None
        }
    }
}
    ...
    println!("s? {:?}", s.to_str());
    // s? Some("hello")
    // println!("{:?}", s) // error! s has moved...
```

函数命名很重要 - 这叫做`to_str`，而不是`as_str`。 你可以编写一个方法，借用该字符串，作为(as)一个`Option<&String>` (这个引用需要与 枚举变量 具有相同的生命周期。 ) ，这样，你就不能命名为`to_str`。

你也可以写`to_str` - 它完全等价的:

```rust
    fn to_str(self) -> Option<String> {
        if let Value::Str(s) = self {
            Some(s)
        } else {
            None
        }
    }
```

## 关于匹配的 更多

回想一下，元组的值可以用'()'来提取:

```rust
    let t = (10,"hello".to_string());
    ...
    let (n,s) = t;
    // t 已 移动了. 不再存在
    // n 是 i32, s 是 String
```

这是一个 _解构{destructuring}_ 特例; 我们有一些数据，希望将其分开来 (像这里) ，或只是借用它的值。无论哪种方式，我们都可以得到结构的各个部分。

语法与在`match`中使用的相似。 在这里，我们明确地借用了这些值。

```rust
    let (ref n,ref s) = t;
    // n 和 s 从 t 那里借用. t 还存在!
    // n 是 &i32, s 是 &String
```

解构与结构一起工作:

```rust
    struct Point {
        x: f32,
        y: f32
    }

    let p = Point{x:1.0,y:2.0};
    ...
    let Point{x,y} = p;
    // p 还在, 直到 x 和 y 已复制
    // x 和 y 都是 f32
```

下面时间，看看`match`的新模式。前两种模式与`let`解构相同 - 它只匹配第一个元素为零的元组，和一个 _任何_ 字符串; 第二个模式增加了一个`if`，所以它只匹配`(1， "hello")`。 最后，只是一个匹配 _任何_ 的 变量。但，如果`match`要应用一个表达式，而你不希望将变量绑定到该表达式，那会被忽略的`_`就会很有用，这是一个`match`结尾的常用方法。

```rust
fn match_tuple(t: (i32,String)) {
    let text = match t {
        (0, s) => format!("zero {}", s),
        (1, ref s) if s == "hello" => format!("hello one!"),
        tt => format!("no match {:?}", tt),
        // 或 使用  _ => format!("no match")
        // 若你对变量不感兴趣。
     };
    println!("{}", text);
}
```

为什么该函数不匹配`match_tuple((1,"hello"))`? 匹配是一个精确的工作，而编译器会抱怨:

```
  = note: expected type `std::string::String`
  = note:    found type `&'static str`
```

我们为什么需要`ref s`? 如果你有一个需要借用的*if-守卫*，这时存在个稍微隐晦的问题 (查找 E0008 错误)，因为如果 if-守卫 是在不同的上下文中发生，就会发生移动。这是隐晦漏洞的示例情况。

> 译者 TODO: 添加 E0008 错误的中文翻译

如果类型 _是_ `&str`，那么我们直接匹配它:

```rust
    match (42,"answer") {
        (42,"answer") => println!("yes"),
        _ => println!("no")
    };
```

`match`用到`if let`的情况。这有个很酷的例子，因为如果我们得到一个`Some`，我们可以匹配里面的，只从元组中提取字符串。 所以在这里没有必要嵌套`if let`表达式。我们用`_`，因为我们对元组的第一部分不感兴趣。

```rust
    let ot = Some((2, "hello".to_string());

    if let Some((_,ref s)) = ot {
        assert_eq!(s, "hello");
    }
    // 我们只是借用该字符串, 而不是 '不可挽回地破坏结构'
```

使用`parse`时会出现一个有趣的问题 (或任何需要从上下文中，计算出其返回类型 的函数)

```rust
    if let Ok(n) = "42".parse() {
        ...
    }
```

那么，这`n`是什么类型的? 不管怎样，你必须提供一个提示 - 什么样的整数?它是否是一个整数?

```rust
    if let Ok(n) = "42".parse::<i32>() {
        ...
    }
```

这种不太优雅的语法被称为"涡轮运算符{turbofish operator}".

如果你有正在返回`Result`的一个函数，那么问号运算符提供了一个更加优雅的解决方案:

```rust
    let n: i32 = "42".parse()?;
```

但是，解析错误需要转换为`Result`的错误变种，这是我们稍后讨论时要讨论的话题-[6.错误处理](./6-error-handling.zh.html).

## 闭包{Closures}

Rust 的很多力量来源于 _闭包_。 它们最简单的形式就像快捷函数一样:

```rust
    let f = |x| x * x;

    let res = f(10);

    println!("res {}", res);
    // res 100
```

在这个例子中没有明确的类型 - 一切都是从整数常量 10 ，开始推导出来的。

如果我们运行，会收到`f`具有不同类型的错误 - Rust 已经决定`f`必须在整数类型上调用:

```
    let res = f(10);

    let resf = f(1.2);
  |
8 |     let resf = f(1.2);
  |                  ^^^ expected integral variable, found floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`

```

所以，第一次调用修复了参数的类型`x`。这相当于这个函数:

```rust
    fn f (x: i32) -> i32 {
        x * x
    }
```

但，函数和闭包之间存在很大差异，具体 _体现_ 在明确类型的需要。 这里，我们先执行一个线性函数:

```rust
    let m = 2.0;
    let c = 1.0;

    let lin = |x| m*x + c;

    println!("res {} {}", lin(1.0), lin(2.0));
    // res 3 5
```

你不能用明确的`fn`形式 - 因它不知道闭包范围内的变量。闭包函数是从其上下文 _借用了_ `m`和`c`。

现在，这`lin`是什么类型? 只有`rustc`知道。 在引擎盖下，闭包是一个 _结构_ ，且是可调用的 ('实现调用操作符') 。它的行为就好像这样写出来的:

```rust
struct MyAnonymousClosure1<'a> {
    m: &'a f64,
    c: &'a f64
}

impl <'a>MyAnonymousClosure1<'a> {
    fn call(&self, x: f64) -> f64 {
        self.m * x  + self.c
    }
}
```

当然，编译器就出来做事了，把简单的闭包语法变成完整的代码! 你需要知道的是，闭包为一个 _结构_ 和它 _借用_ 来自其环境的值。因此它有一个 _lifetime_。

所有闭包都是独特的类型，但它们有共同的 traits。 所以即使我们不知道确切的类型，我们知道泛型约束:

```rust
fn apply<F>(x: f64, f: F) -> f64
where F: Fn(f64)->f64  {
    f(x)
}
...
    let res1 = apply(3.0,lin);
    let res2 = apply(3.14, |x| x.sin());
```

子曰: `apply`为`T`这样的 _任何_ 且具备`Fn(f64) -> f64`的类型工作 - 也就是说，这是一个需要`f64`并返回`f64`的函数。

运行`apply(3.0,lin)`后，试图访问`lin`会给出一个有趣的错误:

```
    let l = lin;
error[E0382]: use of moved value: `lin`
  --> closure2.rs:22:9
   |
16 |     let res = apply(3.0,lin);
   |                         --- value moved here
...
22 |     let l = lin;
   |         ^ value used here after move
   |
   = note: move occurs because `lin` has type
    `[closure@closure2.rs:12:15: 12:26 m:&f64, c:&f64]`,
     which does not implement the `Copy` trait

```

就是这样，`apply`吃了我们的闭包函数。 还有，这个结构的实际类型，`rustc`会弥补实现它。 始终，将闭包视为结构是有帮助的。

调用一个闭包就是一个 _方法调用_: 三种函数 trait 对应于三种方法:

- `Fn` 结构传递为`&self`
- `FnMut` 结构传递为`&mut self`
- `FnOnce` 结构传递为`self`

所以，闭包可能会改变它的 _来自上层_ 引用:

```rust
    fn mutate<F>(mut f: F)
    where F: FnMut() {
        f()
    }
    let mut s = "world";
    mutate(|| s = "hello");
    assert_eq!(s, "hello");
```

注意`mut`-`f`需要可变来工作.

但是，你无法逃避借用规则。考虑一下:

```rust
let mut s = "world";

// 闭包搞了个 s 的 可变借用
let mut changer = || s = "world";

changer();
// 再搞个 s 不可变借用
assert_eq!(s, "world");
```

无法完成! 错误是：在 assert 声明中，我们不能借用`s`，因为它之前作为可变借用，已经被闭包`changer`搞走了。 只要闭包存在，其他代码就不能访问`s`，所以解决方案是通过将闭包放在一个 有限的范围 内，来控制这个生命周期:

```rust
let mut s = "world";
{
    let mut changer = || s = "world";
    changer();
}
assert_eq!(s, "world");
```

在这一点上，如果你习惯了 JavaScript 或 Lua 等语言，你可能会感到 Rust 闭包的复杂性，而不是在这些语言中的直截了当。 这正是 Rust 承诺不作出任何分配的必要成本。 在 JavaScript 中，等效的`mutate(function() {s = "hello";})`，将始终，导致动态分配闭包。

有时，你不希望闭包借用这些变量，而是 _移动_ 他们。

```rust
    let name = "dolly".to_string();
    let age = 42;

    let c = move || {
        println!("name {} age {}", name,age);
    };

    c();

    println!("name {}",name);
```

最后的错误`println`是: "使用了移动值: `name`"，所以这里有一个解决方案 - 如果我们 _想保持_ `name`活着 - 就将 复制的副本 移入`闭包{move}`中:

```rust
    let cname = name.to_string();
    let c = move || {
        println!("name {} age {}",cname,age);
    };
```

为什么需要移动的闭包? 因为我们可能需要在 原始上下文不再存在 的地方调用它们。 经典案例是创建一个 _thread{线程}_。 移动的闭包不借用，就没有生命周期。

> 移动后, 线程中, 所使用的变量, 就会与 原上下文 没有关系了。

迭代器方法中，主要使用闭包。 回想一下，我们定义的遍历一系列浮点数的`range`迭代器。使用闭包对此 (或任何其他迭代器) 进行操作都很简单:

```rust
    let sine: Vec<f64> = range(0.0,1.0,0.1).map(|x| x.sin()).collect();
```

`map`没有在 Vec 上定义 (尽管，很容易创建一个这样的 trait)，因为那样的话， _每次_ map 都将创建一个新的 Vec。就这样，选择很明显了。

这个`sum`，不存在创建临时对象:

```rust
 let sum: f64 = range(0.0,1.0,0.1).map(|x| x.sin()).sum();
```

它 (事实上) 会像明确的循环一样快! 如果 Rust 闭包与 Javascript 闭包一样"没有摩擦火花"，那么这种性能保证就不可能。

`filter`是另一种有用的迭代器方法 - 它只允许，通过匹配条件的值:

```rust
    let tuples = [(10,"ten"),(20,"twenty"),(30,"thirty"),(40,"forty")];
    let iter = tuples.iter().filter(|t| t.0 > 20).map(|t| t.1);

    for name in iter {
        println!("{} ", name);
    }
    // thirty
    // forty
```

## 三种迭代器

三种类型 (再次) 对应于三种基本参数类型。

假设我们有一个`String`值的 Vec 。以下是明确的迭代器类型，和 _隐式{implicitly}_，以及迭代器返回的实际类型。

```rust
for s in vec.iter() {...} // &String
for s in vec.iter_mut() {...} // &mut String
for s in vec.into_iter() {...} // String

// 隐式!
for s in &vec {...} // &String
for s in &mut vec {...} // &mut String
for s in vec {...} // String
```

就我个人而言,我更喜欢明确，但，了解这两种形式及其含义是非常重要的。

`into_iter` _消耗_ Vec ，并提取它的字符串，所以之后 Vec 不再可用 - 它已被移动。 这是 Pythonistas 过去常说的一个确定问题`for s in vec`!

所以，隐含的形式`for s in &vec`通常才是你想要的，就像`&T`在向函数传递参数时，是一个很好的默认值。

理解这三种类型是如何工作是很重要的，因为 Rust 严重依赖于类型推导 - 在闭包参数中，你不会经常看到明确的类型。 这是一件好事， 因为如果所有这些类型都明确的话， 它的 _写法_ 会很嘈杂。 当然，这个紧凑的代码的代价，是你需要知道隐式类型究竟是什么!

`map`取得迭代器返回的任何值，并将其转换为其他值，但是`filter`需要的是一个该值的 _引用_。 在这种正在使用`iter`的情况下，迭代器 item 的类型是`&String`。 注意`filter`接收的是这种类型的引用.

```rust
for n in vec.iter().map(|x: &String| x.len()) {...} // n 是 usize
....
}

for s in vec.iter().filter(|x: &&String| x.len() > 2) { // s 是 &String
...
}
```

在调用方法(如:`x.len()`)时， Rust 会自动 _解引用_，所以问题不明显。 但`|x:&& String|`x =="one"|将 _不会_ 工作, 因为操作符号对 类型匹配 更加严格。 `rustc`会抱怨`&&String`和`&str`没有这样进行比较的。 所以你需要明确的 解引用 ，让`&&String`变成能 _完成_ 比较 的`&String`。

```rust
for s in vec.iter().filter(|x: &&String| *x == "one") {...}
// 等价的隐式写法:
for s in vec.iter().filter(|x| *x == "one") {...}
```

如果省略显式类型，则可以修改参数，使`s`的类型就是现在的`&String`:

```rust
for s in vec.iter().filter(|&x| x == "one")
```

看你如何看待它。

## 具有动态数据的结构

一个最强大的技术是 _一个包含对自身引用的结构_。

这里是一个 _二叉树_ 的基本构建块，用 C 语言 表示 (每个人最喜欢的老亲戚都喜欢使用没有保护的电动工具。 )

```rust
    struct Node {
        const char *payload;
        struct Node *left;
        struct Node *right;
    };
```

你不能 _直接{directly}_ 在 Rust 这样做 - 包含`Node`字段，因为`Node`的大小取决于`Node`的大小... 它无法计算。 所以我们使用指针指向`Node`结构，因为指针的大小总是已知的。

如果`left`不是`NULL`，那`Node`将有一个`left`字段，其指向另一个节点，一直无限下去。

Rust 不会`NULL` (至少不 _安全_) ， 所以这显然是一份`Option`的工作。 但你，不能只是把一个`Node`放在`Option`里面，因为我们不知道`Node`的大小 (等等)。 这又是`Box`的工作，因为它分配了包含一个指向数据的指针，并且一直具有固定大小。

所以这里是 Rust 的等价物，使用`type`创建一个别名:

```rust
type NodeBox = Option<Box<Node>>;

#[derive(Debug)]
struct Node {
    payload: String,
    left: NodeBox,
    right: NodeBox
}
```

( Rust 以这种方式解决问题 - 不需要前瞻性声明。 )

下面，第一个测试程序:

```rust
impl Node {
    fn new(s: &str) -> Node {
        Node{payload: s.to_string(), left: None, right: None}
    }

    fn boxer(node: Node) -> NodeBox {
        Some(Box::new(node))
    }

    fn set_left(&mut self, node: Node) {
        self.left = Self::boxer(node);
    }

    fn set_right(&mut self, node: Node) {
        self.right = Self::boxer(node);
    }

}


fn main() {
    let mut root = Node::new("root");
    root.set_left(Node::new("left"));
    root.set_right(Node::new("right"));

    println!("arr {:#?}", root);
}
```

由于"{:#?}" ('#'表示'扩开') ,输出结果非常漂亮.

```
root Node {
    payload: "root",
    left: Some(
        Node {
            payload: "left",
            left: None,
            right: None
        }
    ),
    right: Some(
        Node {
            payload: "right",
            left: None,
            right: None
        }
    )
}
```

现在, `root`变量若被丢弃会发生什么 ? 所有字段都被删除; 如果树的"分支"被丢弃，就会扔掉 _它们_ 的字段等等。 `Box::new`可能是最接近`new`关键字的呢，但我们没有必要`delete`要么`free`。

我们现在必须为这棵树制定一个用法。请注意，可以指定字符串 顺序: 'bar'&lt;'foo'，'abba'>'aardvark'; 所谓的"字母顺序"。 (严格来说，这是词汇顺序，因为人类语言非常多样化，并且有着奇怪的规则。)

这是一个按字符串的顺序，插入节点的方法。我们将新数据与当前节点进行比较 - 如果较少，则尝试插入左侧，否则尝试插入右侧。 左边可能没有节点，那么就`set_left`等等。

```rust
    fn insert(&mut self, data: &str) {
        if data < &self.payload {
            match self.left {
                Some(ref mut n) => n.insert(data),
                None => self.set_left(Self::new(data)),
            }
        } else {
            match self.right {
                Some(ref mut n) => n.insert(data),
                None => self.set_right(Self::new(data)),
            }
        }
    }

    ...
    fn main() {
        let mut root = Node::new("root");
        root.insert("one");
        root.insert("two");
        root.insert("four");

        println!("root {:#?}", root);
    }
```

注意`match`- 我们会提供一个可变的引用给到 box，如果`Option`是`Some`的话，并应用`insert`方法。 否则，我们需要为左侧创建一个新的`Node`等等。 `Box`是一个 _聪明_ 指针; 请注意，不需要"拆箱{unboxing}"来呼叫`Node`方法!

这里是输出树:

```
root Node {
    payload: "root",
    left: Some(
        Node {
            payload: "one",
            left: Some(
                Node {
                    payload: "four",
                    left: None,
                    right: None
                }
            ),
            right: None
        }
    ),
    right: Some(
        Node {
            payload: "two",
            left: None,
            right: None
        }
    )
}
```

比，其他字符串'小于'的字符串放在左侧,，则放在右侧。

参观时间。 这是 _按顺序遍历_ - 我们访问左边，在节点上做点什么，然后访问右边。

```rust
    fn visit(&self) {
        if let Some(ref left) = self.left {
            left.visit();
        }
        println!("'{}'", self.payload);
        if let Some(ref right) = self.right {
            right.visit();
        }
    }
    ...
    ...
    root.visit();
    // 'four'
    // 'one'
    // 'root'
    // 'two'
```

所以，我们按顺序访问这些字符串! 请注意重复出现的`ref` - `if let`使用与`match`完全相同的规则。

## 泛型结构

考虑前面的二叉树的例子。 这将是 _严重刺激_ ，不得不重写它, 当为了所有可能的 payload 类型。 所以，我们的泛型`Node`与它的类型参数`T`.

```rust
type NodeBox<T> = Option<Box<Node<T>>>;

#[derive(Debug)]
struct Node<T> {
    payload: T,
    left: NodeBox<T>,
    right: NodeBox<T>
}
```

该实现显示了语言之间的差异。 payload 的基本操作是比较，所以 T 必须与之相当`<` ，等等， 实现 `PartialOrd`。 必须在`impl`其中声明类型参数:

```rust
impl <T: PartialOrd> Node<T> {
    fn new(s: T) -> Node<T> {
        Node{payload: s, left: None, right: None}
    }

    fn boxer(node: Node<T>) -> NodeBox<T> {
        Some(Box::new(node))
    }

    fn set_left(&mut self, node: Node<T>) {
        self.left = Self::boxer(node);
    }

    fn set_right(&mut self, node: Node<T>) {
        self.right = Self::boxer(node);
    }

    fn insert(&mut self, data: T) {
        if data < self.payload {
            match self.left {
                Some(ref mut n) => n.insert(data),
                None => self.set_left(Self::new(data)),
            }
        } else {
            match self.right {
                Some(ref mut n) => n.insert(data),
                None => self.set_right(Self::new(data)),
            }
        }
    }
}


fn main() {
    let mut root = Node::new("root".to_string());
    root.insert("one".to_string());
    root.insert("two".to_string());
    root.insert("four".to_string());

    println!("root {:#?}", root);
}
```

所以，泛型结构要像 C ++ 一样，需要在 `<>` 中指定泛型类型参数(们)。 Rust 通常很聪明，可以从上下文中得出这个类型参数 - 它知道它有一个`Node<T>`，还知道它的`insert`方法需要`T`参数。 `insert` 的第一次运行，会把`T`钉成为`String`。如果有任何进一步的运行不一致，它会投诉。

但是，你确实需要适当地限制这种类型!
