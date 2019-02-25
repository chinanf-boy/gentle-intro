# 基础

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [你好,世界!](#%E4%BD%A0%E5%A5%BD%E4%B8%96%E7%95%8C)
- [循环和条件语句](#%E5%BE%AA%E7%8E%AF%E5%92%8C%E6%9D%A1%E4%BB%B6%E8%AF%AD%E5%8F%A5)
- [开始堆积木吧](#%E5%BC%80%E5%A7%8B%E5%A0%86%E7%A7%AF%E6%9C%A8%E5%90%A7)
- [函数类型是明确的](#%E5%87%BD%E6%95%B0%E7%B1%BB%E5%9E%8B%E6%98%AF%E6%98%8E%E7%A1%AE%E7%9A%84)
- [学习在哪里找到绳子](#%E5%AD%A6%E4%B9%A0%E5%9C%A8%E5%93%AA%E9%87%8C%E6%89%BE%E5%88%B0%E7%BB%B3%E5%AD%90)
- [数组和切片](#%E6%95%B0%E7%BB%84%E5%92%8C%E5%88%87%E7%89%87)
- [切和割](#%E5%88%87%E5%92%8C%E5%89%B2)
- [可选（Option）值](#%E5%8F%AF%E9%80%89option%E5%80%BC)
- [向量](#%E5%90%91%E9%87%8F)
- [迭代器](#%E8%BF%AD%E4%BB%A3%E5%99%A8)
- [更多关于向量](#%E6%9B%B4%E5%A4%9A%E5%85%B3%E4%BA%8E%E5%90%91%E9%87%8F)
- [字符串](#%E5%AD%97%E7%AC%A6%E4%B8%B2)
- [插曲: 获取命令行参数](#%E6%8F%92%E6%9B%B2-%E8%8E%B7%E5%8F%96%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0)
- [匹配](#%E5%8C%B9%E9%85%8D)
- [读取文件](#%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 你好,世界!

自从第一个 C 语言版本诞生，"hello world"的最初目的是测试编译器并运行一个实际的程序。

```rust
// hello.rs
fn main() {
    println!("Hello, World!");
}
```

```
$ rustc hello.rs
$ ./hello
Hello, World!
```

Rust 是一种带分号的花括号语言, C ++ 风格注释和一个`main`函数 一 目前来说，非常熟悉吧。 `感叹号{!}`表明这是一个 _宏_ 调用。 对于 C ++ 程序员来说，这可能是一个退步，因为它们使用了非常愚蠢的 C 宏 - 但我可以确保这些宏能够更强大和更理智。

对于其他任何人来说，会是"现在好了，我不得不记得说，砰!"。 但是，编译器很强的，知道吧;如果你忽略了那个惊叹号，你会得到:

```
error[E0425]: unresolved name `println`
    --> hello2.rs:2:5
    |
2 |     println("Hello, World!");
    |     ^^^^^^^ did you mean the macro `println!`?
```

学习一门语言意味着要熟悉它的错误。 试着把编译器当做是一个严格但友好的帮手，而不是一台对你 _大喊大叫{shouting}_ 的电脑，因为你在最开始时，就会看到很多红墨迹。对于编程人员来说，你的编译器提前指出你的错误比程序在用户面前炸毁要好得多。

下一步是介绍一个 _变量{variable}_:

```rust
// let1.rs
fn main() {
    let answer = 42;
    println!("Hello {}", answer);
}
```

拼写错误是 _编译{compile}_ 错误，而不是类似 Python 或 JavaScript 等动态语言的运行时错误。 这将为您节省很多压力!如果我写了'answr'而不是'answer'，编译器实际上会有关于它的 _不错提示_ :

```
    4 |     println!("Hello {}", answr);
      |                         ^^^^^ did you mean `answer`?
```

`println!`宏需要一个[格式字符串{format string}](https://doc.rust-lang.org/std/fmt/index.html)和一些 值 ;它与 Python 3 使用的格式非常相似。

另一个非常有用的宏是`assert_eq!`。 这是在 Rust 中进行测试的主力;您 _断言{assert}_ 两件事必须相等，如果不是，就会 _panic{恐慌}_，相当于程序崩溃。

```rust
// let2.rs
fn main() {
    let answer = 42;
    assert_eq!(answer,42);
}
```

本来是不会产生任何输出。但一旦改 42 为 40:

```
thread 'main' panicked at
'assertion failed: `(left == right)` (left: `42`, right: `40`)',
let2.rs:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

这是我们在 rust 中的第一个 _运行时错误_ 。

## 循环和条件语句

任何有趣的事情都可以会做不止一次:

```rust
// for1.rs
fn main() {
    for i in 0..5 {
        println!("Hello {}", i);
    }
}
```

这 _范围{range}_ 并不包括 **5**，所以`i`的范围从**0 到 4**。这在将数组等内容从 0 开始进行 _索引{indexes}_ 的语言中很方便。

有趣的事情也必须要 _有条件地{conditionally}_ 做:

```rust
// for2.rs
fn main() {
    for i in 0..5 {
        if i % 2 == 0 {
            println!("even {}", i);
        } else {
            println!("odd {}", i);
        }
    }
}
```

```
even 0
odd 1
even 2
odd 3
even 4
```

`i % 2`为 0，如果`i`能被 2 整除; Rust 使用 C 风格操作符。 条件周围没有括号，这就像 Go 语言。但是在条件后面必须要跟使用花括号的代码块。

同样的事情，更有趣的写法方式:

```rust
// for3.rs
fn main() {
    for i in 0..5 {
        let even_odd = if i % 2 == 0 {"even"} else {"odd"};
        println!("{} {}", even_odd, i);
    }
}
```

传统上，编程语言有 _声明{statements}_ (比如`if`) 和 _表达式{expressions}_ (比如`1 + i`) 。 在 rust 里，几乎所有的东西都有一个值并且可以成为表达式。 不再需要超丑的 C '三元操作符'`i % 2 == 0?"even": "odd"`.

⚠️ 请注意，这些代码块中没有任何分号(像`{"even"} else {"odd"}`这样的)。

## 开始堆积木吧

计算机非常擅长算术。 这里第一次尝试添加从 0 到 4 的所有数字:

```rust
// add1.rs
fn main() {
    let sum = 0;
    for i in 0..5 {
        sum += i;
    }
    println!("sum is {}", sum);
}
```

但它没有编译成功:

```
error[E0384]: re-assignment of immutable variable `sum`
    --> add1.rs:5:9
3 |     let sum = 0;
    |         --- first assignment to `sum`
4 |     for i in 0..5 {
5 |         sum += i;
    |         ^^^^^^^^ re-assignment of immutable variable
```

`不可变{Immutable}`? 一个变量不能 _变{vary}_? 默认的，`let`声明时，变量只能赋值。添加魔法`mut` (_请_ 让变量可变) 完成表演:

```rust
// add2.rs
fn main() {
    let mut sum = 0;
    for i in 0..5 {
        sum += i;
    }
    println!("sum is {}", sum);
}
```

其他语言使用人员，可能会感到费解，因，在他们看来，默认情况下变量就可以被重写。 变量的产生是，在运行时被分配了一个计算值 - 这不是一个 _常数 constant}_ 。 在数学中也有同样的说法，就像我们说'让 n 是 S 中最大的数'。

声明变量默认 _只读_ ，是有原因的。 在更大的程序中，很难跟踪正在写入的代码。 所以 Rust 是为了能够明确地表现出，像可变性 ('能写入') 的东西。 Rust 语言中有很多聪明之处，但它不会隐藏任何东西。

Rust 既是静态类型又是强类型的，它们通常是混淆的，但请考虑 C (静态但弱类型) 和 Python (动态但强类型)。 在静态类型中，类型在编译时是已知的，而动态类型仅在运行时知道。

然而，此刻，感觉 Rust 把这些类型 _藏{hiding}_ 了起来。究竟`i`是什么类型? 编译器可以从 0 开始, _类型推断_ 并提出`i32` (四字节有符号整数)。

让我们做一个改变`0`到`0.0`. 然后我们得到错误:

```
error[E0277]: the trait bound `{float}: std::ops::AddAssign<{integer}>` is not satisfied
    --> add3.rs:5:9
    |
5 |         sum += i;
    |         ^^^^^^^^ the trait `std::ops::AddAssign<{integer}>` is not implemented for `{float}`
    |
```

好了，蜜月结束了: 这意味着什么? 每个操作符 (像 `+=` ) 对应一个 _特性{trait}_ ，而这是一个抽象的接口，必须为每种具体的类型实现。 稍后我们将详细地处理 trait，但是这里您需要知道的是，`附加赋值{AddAssign}`是实现`+=`运算符的 trait 名称，错误是说浮点数没有实现整数的`+=`运算符。 (运算符 trait 的完整列表[在这里](https://doc.rust-lang.org/std/ops/index.html))

同样，Rust 喜欢张扬, 它不会默默地把那个整数转换成浮点数。

我们必须显式地将该值类型 _转换_ 为浮点数.

```rust
// add3.rs
fn main() {
    let mut sum = 0.0;
    for i in 0..5 {
        sum += i as f64;
    }
    println!("sum is {}", sum);
}
```

## 函数类型是明确的

_函数{Functions}_ 是一个，编译器不容有失的类型之处。

这实际上是一个深思熟虑的决定，因像 Haskell ，该语言拥有强大的类型推断，几乎没有显式的类型名称。这 Haskell 风格，确实是函数+显式类型签名的好方法。而这也是 rust 需要的。

这是一个简单的用户定义函数:

```rust
// fun1.rs

fn sqr(x: f64) -> f64 {
    return x * x;
}

fn main() {
    let res = sqr(2.0);
    println!("square is {}", res);
}
```

Rust 回到了一个传统的参数声明，其中类型跟在名称后面。如同在 Pascal 等 Algol 派生语言。

再次，若没有整数到浮点数的转换 - 如果你用'2'直接代替`2.0`，那么我们
会得到一个明确的错误：

```
8 |     let res = sqr(2);
    |                   ^ expected f64, found integral variable
    |
```

你很少会看到函数使用`return`声明。 更多时候，它会像这样:

```rust
fn sqr(x: f64) -> f64 {
    x * x
}
```

这是因为函数的主体（`{}`内部）具有 _最后值表达式_ ，就像 if-as-an-expression.

由于分号是由人的手指半自动插入的，因此您可以添加它
在 _最后值表达式_ ，并得到以下错误：

```
    |
3 | fn sqr(x: f64) -> f64 {
    |                       ^ expected f64, found ()
    |
    = note: expected type `f64`
    = note:    found type `()`
help: consider removing this semicolon:
    --> fun2.rs:4:8
    |
4 |     x * x;
    |       ^
```

这`()`类型是空的类型，没有什么结果，`无效{void}`，0，空，什么都没有的意思。 Rust 的一切都有个值，但有时它就是为空。编译器察觉这是个常见的错误，并能实实在在地帮助到你，(每个在 C++编译器上花过时间的人都知道，这可是个 _要死要死的情况_ )。

> 也就是说, 如果你要返回, 就不能加 `分号{;}`

没 return 表达风格的几个例子:

```rust
// 返回，一个浮点数的绝对值函数
fn abs(x: f64) -> f64 {
    if x > 0.0 {
        x
    } else {
        -x
    }
}

// 确保，该数字，定然在给予的范围内
fn clamp(x: f64, x1: f64, x2: f64) -> f64 {
    if x < x1 {
        x1
    } else if x > x2 {
        x2
    } else {
        x
    }
}
```

使用`return`不是错误的，但没有它，代码就会更干净。 但是对于从一个函数 _提前回来_ ， 你仍会用到`return`。

一些操作可以被优雅地表达 _递归_:

```rust
fn factorial(n: u64) -> u64 {
    if n == 0 {
        1
    } else {
        n * factorial(n-1)
    }
}
```

起初这可能有些奇怪，然后最好用铅笔和纸制作一些例子。然而，通常这样做不是最 _高效_ 的方式。

值也可以通过 _引用_ 方式传递。 一个引用是由`&`创建，还有用`*` _解引用_ 。

```rust
fn by_ref(x: &i32) -> i32{
    *x + 1
}

fn main() {
    let i = 10;
    let res1 = by_ref(&i);
    let res2 = by_ref(&41);
    println!("{} {}", res1,res2);
}
// 11 42
```

如果你想要一个函数来修改它的一个参数呢? 那么请输入 _可变引用_:

```rust
// fun4.rs

fn modifies(x: &mut f64) {
    *x = 1.0;
}

fn main() {
    let mut res = 0.0;
    modifies(&mut res);
    println!("res is {}", res);
}
```

这比 C ++ 更像 C ++ 。 你必须明确地传递参数 (加上`&`) 和明确 用`*` _解引用_ 。 然后键入`mut`, 因为它不是默认可变的。 (我一直觉得与 C 相比, C++ 引用太容易错过。 )

基本上, Rust 是引入一些 _摩擦{friction}_ 这里。并不是那么巧妙地推动函数直接返回值。 幸运的是, rust 有强力的方式表达"操作成功,结果在这里"。 所以`mut`不需要那么频繁。 当我们有一个大对象并且不想复制它时，传递引用就很重要了。

变量后加上类型的样式，同样适用于`let`，当你真的想改变变量的类型:

```rust
let bigint: i64 = 0;
```

## 学习在哪里找到绳子

现在是开始使用文档的时候了。 这已安装在您的机器上，您可以使用`rustup doc --std`在浏览器中打开它。

注意顶部的 _搜索_ ，因为这将是你的朋友;它完全离线运行。

假设我们想知道数学函数在哪里，所以搜索"cos"。 前两个，显示它为单精度和双精度浮点数字的定义。 它定义在 _值本身{value itself}_ 之上，作为一种方法，像这样:

```rust
let pi: f64 = 3.1416;
let x = pi/2.0;
let cosine = x.cos();
```

结果近乎于零; 我们显然需要一个更权威的'pi'!

(为什么我们需要一个明确的`f64`类型? 因为没有它，该*3.1416*常数可以是`f32`或`f64`类型，而这些都是非常不同的。)

让我引用一个`cos`例子，但写一个完整的程序(`assert_eq!`的表亲戚`assert!`;表达式必须正确)。

```rust
fn main() {
    let x = 2.0 * std::f64::consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

`std::f64::consts::PI`是一口饭! `::`与在 c++中有同样的意思，(通常使用"."在其他语言) - 这是一个完全合格的名字。 在文档搜索“PI”后，我们在第二个提示中得到这个全名。

到目前为止，我们的小 Rust 项目一直抛开`import`和`exclude`这些，会使讨论"Hello World"程序慢下来的东西。让这个程序可读性更强的`use`声明:

```rust
use std::f64::consts;

fn main() {
    let x = 2.0 * consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

为什么我们现在不需要这样做？
这是因为 Rust 的*prelude*在起作用，使许多基本功能无需显式 `use`语句。

## 数组和切片

所有静态类型的语言都有 _数组_，这在内存装有鼻子到尾巴的值。数组 _索引_ 从零开始:

```rust
// array1.rs
fn main() {
    let arr = [10, 20, 30, 40];
    let first = arr[0];
    println!("first {}", first);

    for i in 0..4 {
        println!("[{}] = {}", i,arr[i]);
    }
    println!("length {}", arr.len());
}
```

输出是:

```
first 10
[0] = 10
[1] = 20
[2] = 30
[3] = 40
length 4
```

在这种情况下，Rust 知道数组 _究竟_ 有多大，如果你尝试访问`arr[4]`，这将是一个 _编译错误_ 。

学习一门新语言往往涉及到 _忘却_ 来自旧语言的已知思维习惯; 如果你是一个 Pythonista，那么这些括号你想是`list`。快速产生思绪，这是 Rust 中的`list`等同物，但数组不是你正在想的那样; 他们是 _固定大小_。 他们也会是 _可变的_ (如果我们问得好)，但你不能添加新的元素。

在 Rust 中不常使用数组，因为数组的类型包含他们大小。 示例中的数组的类型是`[i32;4]`; `[10,20]`类型将会`[i32;2]`等等: 他们有 _不同类型_。 所以他们作为函数参数是件麻烦事。

常用的 _是_ _切片_。 你可以把它们看作是一个基本值数组的 _快照_ 。 它们的行为很像一个数组， 且 _知道他们的尺寸_ ，不像那些危险的 C 指针东东。

注意这里有两个重要的事情 - 如何写一个切片的类型，和你必须使用`&`将其传递给函数.

```rust
// array2.rs
// 读作 as: i32切片
fn sum(values: &[i32]) -> i32 {
    let mut res = 0;
    for i in 0..values.len() {
        res += values[i]
    }
    res
}

fn main() {
    let arr = [10,20,30,40];
    // 看着这里的 &
    let res = sum(&arr);
    println!("sum {}", res);
}
```

先忽略`sum`函数，看看`&[i32]`。 rust 数组和切片之间的关系类似于 C 数组和指针 之间的关系，除了两个重要的区别: rust 的切片会跟踪它们的大小 (如果你 尝试访问这个大小之外 会 _panic_)，并且想把数组作为一个切片传递，你必须明确地使用`&`操作符。

C 程序员读`&`作为"取地址符"，rust 程序员则是 _借用{borrow}_ 它。 这将是要学习的 rust 关键词。 借用是编程中常见模式的名称; 每当你通过引用传递 (几乎总是发生在动态语言中) 或 在 C 中传递指针时，原始所有者所拥有的任何东西被 _借用_ 了。

## 切和割

不能以通常的方式`{}`打印出一个数组，但你可以用`{:?}`做一个 *debug*性质的打印。

```rust
// array3.rs
fn main() {
    let ints = [1, 2, 3];
    let floats = [1.1, 2.1, 3.1];
    let strings = ["hello", "world"];
    let ints_ints = [[1, 2], [10, 20]];
    println!("ints {:?}", ints);
    println!("floats {:?}", floats);
    println!("strings {:?}", strings);
    println!("ints_ints {:?}", ints_ints);
}
```

这使:

```
ints [1, 2, 3]
floats [1.1, 2.1, 3.1]
strings ["hello", "world"]
ints_ints [[1, 2], [10, 20]]
```

所以，数组套数组是没问题,但重要的是，数组包括内容 _只能有一个类型_。 数组中的值
在内存中排列在一起，以便他们非常高效地访问。

如果你对这些变量实际的类型感到好奇，这有些能用的方法。就是用一个你知道会是错误的显式类型，来声明一个变量:

```rust
let var: () = [1.1, 1.2];
```

这是信息错误:

```
3 |     let var: () = [1.1, 1.2];
  |                   ^^^^^^^^^^ expected (), found array of 2 elements
  |
  = note: expected type `()`
  = note:    found type `[{float}; 2]`
```

(`{float}`意思是"一些不完全指定的浮点数类型)

切片会给你 _相同_ 数组的不同 _视角_ :

```rust
// slice1.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice1 = &ints[0..2];
    let slice2 = &ints[1..];  // 开放式范围!

    println!("ints {:?}", ints);
    println!("slice1 {:?}", slice1);
    println!("slice2 {:?}", slice2);
}
```

```
ints [1, 2, 3, 4, 5]
slice1 [1, 2]
slice2 [2, 3, 4, 5]
```

这是一个简洁的符号，类似于 Python 切片但是有很大区别: 从未有过任何数据的副本。 这些 _切片_ 都是`借用{borrow}` 他们自己的数组数据。 与数组存有一个非常亲密的关系，且 Rust 花很多精力来确保这种关系不会被破坏。

## 可选（Option）值

切片，就像数组一样，可以 _索引_。 Rust 在编译时知道数组的大小，但只有在运行时才知道分切片的大小。 所以`s[i]`在运行时会引起超出界限的错误和 _恐慌{panic}_。 这你不会想要，而一个安全启动中止 与 非常昂贵的切片 之间也有所不同。 _无一例外_。

冷静下，大招来了。 你不能在某些 try-block 中包装可怕的问题代码，用来"捕获错误" - 至少不是你每天都想使用的方式。 那么 Rust 如何保证安全?

有一种切片方法`get`，这并不恐慌{panic}。但是它返回了什么?

```rust
// slice2.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice = &ints;
    let first = slice.get(0);
    let last = slice.get(5);

    println!("first {:?}", first);
    println!("last {:?}", last);
}
// first Some(1)
// last None
```

`last`失败 (我们忘记了基于零的索引)，但返回了一个叫做`None`的东西。 `first`很好，但是作为一个 值包装在`Some`中。 欢迎`Options`类型!它可能是`Some` _或者_ `None`。

这`option w`类型有一些有用的方法:

```rust
    println!("first {} {}", first.is_some(), first.is_none());
    println!("last {} {}", last.is_some(), last.is_none());
    println!("first value {}", first.unwrap());

// first true false
// last false true
// first value 1
```

如果你 _打开{unwrap}_ `last`，你会得到一个恐慌{panic}。但至少你可以调用`is_some` - 如示例中，如果默认你有一个 没有值的变量:

```rust
    let maybe_last = slice.get(5);
    let last = if maybe_last.is_some() {
        *maybe_last.unwrap()
    } else {
        -1
    };
```

注意`*` - `Some`内部的精确类型是`&i32`，这是一个引用。 我们需要解引用回到一个`i32`的值.

这繁琐，一个快捷方式是`unwrap_or`, 如果返回的值是`None`的`Option`类型。 - 类型要匹配，因`get`返回一个引用。所以你必须写成`&i32`与`&-1`。最后再次使用`*`获得`i32`类型值。

```rust
    let last = *slice.get(5).unwrap_or(&-1);
```

很容易漏写`&`，但你有编译器的帮助。 如果它是`-1`，`rustc` says 'expected &{integer}， found integral variable'，然后告诉你'help: try`&-1`"。

你可以把`Option`想成一个可能包含一个值的 盒子，或者什么都没有 (`None`) (在 Haskell， 它被称为`Maybe`)。 可能包含 _任何_ 值，就是它的 _类型规范_ 。而在这种情况下，完整的类型是`Option<&i32>`，使用 C ++ 风格的表示 _泛型{generics}_。 打开这个 盒子可能会引起爆炸，但不像薛定谔的猫，我们可以事先知道它是否包含一个值。

在 Rust 函数/方法中， 返回这些可能的盒子(`Option`)，是非常常见的，所以学习如何舒适地[使用它们](https://doc.rust-lang.org/std/option/enum.Option.html)。

## 向量

我们将再次回到切片方法，但首先看看：向量{Vec}。 这些是 _灵活大小_ 的数组，其行为很像 Python 的`List`和 C++ 的`std::vector`。 事实上，rust 的`Vec`会有所不同，你可以将额外的值附加到一个向量上，当然注意，它必须声明为可变的。

```rust
// vec1.rs
fn main() {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v.push(30);

    let first = v[0];  // 同样，超出范围也会 panic
    let maybe_first = v.get(0);

    println!("v is {:?}", v);
    println!("first is {}", first);
    println!("maybe_first is {:?}", maybe_first);
}
// v is [10, 20, 30]
// first is 10
// maybe_first is Some(10)
```

一个常见的初学者错误是忘记`mut`，那你会得到一个有用的错误信息:

```
3 |     let v = Vec::new();
  |         - use `mut v` here to make mutable
4 |     v.push(10);
  |     ^ cannot borrow mutably
```

向量和切片之间有非常密切的关系:

```rust
// vec2.rs
fn dump(arr: &[i32]) {
    println!("arr is {:?}", arr);
}

fn main() {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v.push(30);

    dump(&v);

    let slice = &v[1..]; // <== 这个 &
    println!("slice is {:?}", slice);
}
```

那个小小的,很重要的借用符号`&`是为了 _迫使_ 向量进入切片。且它是完全说得通的，因为向量也在观察着一个有值的数组，不同的是该数组为 _动态地_ 分配。

如果你来自一种动态的语言，那么现在是时候开始讨论下了。 在系统语言中，程序存储器有两种: 堆栈和堆。 在堆栈上分配数据非常简单，但是堆栈是有限的; 通常是 MB 为单位。 堆可以是 GB，但是分配成本相对昂贵，并且这样的内存必须是之后 _释放_ 。在所谓的'管理'语言 (如 java，Go 和所谓的脚本语言) 这些细节都隐藏在'便利的市政工程'称 _垃圾收集器_ 中。 一旦系统确定数据不再引用的其他数据，它就会回到可用内存池。

一般来说，这是一个值得付出的代价。 玩堆栈非常不安全，
因为如果你犯了一个错误，在当前函数中覆盖返回地址，那么你跪了。

我写的第一个 C 程序是在 DOS PC 上，
抛开电脑本身。Unix 系统总是表现得更好，且只有 伴随一个 _segfault_ 的进程才会挂掉
。 为什么这比 Rust（或 Go）程序恐慌{panic}更糟？
因为 Rust 会当原始问题出现了，就会发生恐慌{panic}, 而不是像以前困惑程序怎么崩溃的，并吃掉你所有的功课。

恐慌{panic}就是 _内存安全_ ，它们在任何非法访问内存之前发生。 这是一个 C 中常见的安全问题，因为所有内存访问都是不安全的，并且一个狡猾的攻击者
可以利用这个弱点。

恐慌{panic}本身听起来是绝望的，无计划性的，但 Rust 的恐慌{panic}是结构化的 - 堆栈的 _释放_ 方式
与异常(抛出错误)情况发生时相同。 所有分配的对象都被删除，并且生成一个回溯。

垃圾收集的缺点？ 首先是它是浪费内存，
看看那些占有重要地位，越来越统治我们世界的小型嵌入式微芯片，
其次是它会在最糟糕的时候决定进行 _立即_ 清理
。 （有个妈妈的比喻是，她想在，你与新的情人快乐玩耍时，进行房间的打扫
）。 这些嵌入式系统需要当事物发生时，对其做出响应
（'实时'），并且不能容忍计划外的
清洗举动。 Roberto Ierusalimschy，Lua 的首席设计师（最优雅的动态语言设计师之一）
说，他不想飞机，是
依靠垃圾收集软件在飞。

回到 vectors ：当一个 vectors 被修改或创建时，它由堆分配内存，并变成
该内存的 _拥有者_ 。 切片从 vectors 的内存中借用。
当 vectors 死亡或 _drops_ 时，切片也会跟随 vectors 的动作。

## 迭代器

我们到目前为止，都没有提及的关键部分，也正是 rust 的难题 - 迭代器.

一个范围的 for 循环，是在使用迭代器(`0..n`，其实是类似于 Python 3 的`range`功能)。

迭代器很容易定义。 下面是一个"对象"，它使用`next`方法返回一个`Option`。只要这个值不是`None`，我们就一直`next`下去:

```rust
// iter1.rs
fn main() {
    let mut iter = 0..3;
    assert_eq!(iter.next(), Some(0));
    assert_eq!(iter.next(), Some(1));
    assert_eq!(iter.next(), Some(2));
    assert_eq!(iter.next(), None);
}
```

而这正是`for var in iter {}`所做的。

这似乎是定义 for 循环的一种低效方式，但是`rustc`在发布模式中会进行变态的优化，相信它会和`while`循环一样快。

这是对数组进行迭代的第一次尝试:

```rust
// iter2.rs
fn main() {
    let arr = [10, 20, 30];
    for i in arr {
        println!("{}", i);
    }
}
```

失败,但有帮助哟:

```
4 |     for i in arr {
  |     ^ the trait `std::iter::Iterator` is not implemented for `[{integer}; 3]`
  |
  = note: `[{integer}; 3]` is not an iterator; maybe try calling
   `.iter()` or a similar method
  = note: required by `std::iter::IntoIterator::into_iter`
```

按照`rustc`的建议，下面的程序按预期工作.

```rust
// iter3.rs
fn main() {
    let arr = [10, 20, 30];
    for i in arr.iter() {
        println!("{}", i);
    }

    // 切片将隐式转换为迭代器...
    let slice = &arr;
    for i in slice {
        println!("{}", i);
    }
}
```

实际上，迭代数组或切片，用这种方式比`for i in 0..slice.len() {}`效率更高，因为 Rust 不必痴迷于检查每个索引操作。

我们之前有一个，一系列整数总和的例子。 它涉及一个`mut`变量和循环。以下是 _惯用的_， 总和方式:

```rust
// sum1.rs
fn main() {
    let sum: i32  = (0..5).sum();
    println!("sum was {}", sum);

    let sum: i64 = [10, 20, 30].iter().sum();
    println!("sum was {}", sum);
}
```

请注意，这是其中一个需要明确说明的情况，就是该变量的 _类型_ ，因为不这样做， Rust 就没有足够的信息。 这里我们用两个不同的整数做总和，没有问题。 (如果用尽所有的名字，那创建一个新的同名变量也是没有问题的。 )

为了扩展需要，这有更多的[切片 方法](https://doc.rust-lang.org/std/primitive.slice.html)。 (另一个文档提示;在每个文档页的右边有一个'[-]，可单击该按钮以折叠方法列表。 然后你可以扩展任何看起来很有趣的细节。 那些看起来怪异的东西，现在就忽略它吧。

这个`windows`方法，提供了一个迭代器，层叠的值窗口。

```rust
// slice4.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice = &ints;

    for s in slice.windows(2) {
        println!("window {:?}", s);
    }
}
// window [1, 2]
// window [2, 3]
// window [3, 4]
// window [4, 5]
```

或`块{chunks}`:

```rust
    for s in slice.chunks(2) {
        println!("chunks {:?}", s);
    }
// chunks [1, 2]
// chunks [3, 4]
// chunks [5]
```

## 更多关于向量

有一个有用的小宏`vec!`用于初始化向量。 注意你可以使用`pop` _去除{remove}_ 向量结尾值，和 _扩展{extend}_ 一个兼容迭代器的向量。

```rust
// vec3.rs
fn main() {
    let mut v1 = vec![10, 20, 30, 40];
    v1.pop();

    let mut v2 = Vec::new();
    v2.push(10);
    v2.push(20);
    v2.push(30);

    assert_eq!(v1, v2);

    v2.extend(0..2);
    assert_eq!(v2, &[10, 20, 30, 0, 1]);
}
```

验证向量，它们之间每个对应值都相互比较，切片为值。

可以将值插入到向量中的任意位置。 `插入{insert}`或者使用`去除{remove}`移除。
这不像 push 和 pop 一样高效
，这些值将不得不被移动以腾出空间，所以请小心这些操作
向量。

vec 具有大小和 _capacity{容量}_。 如果你清除了一个 vec ，它的大小就变成了零，
但它仍保留其旧容量。 所以用`push`等来填充，只会
当尺寸大于该容量时，才会重新分配容量。

vec 可以排序，然后可以删除重复的 - 这些操作就在 vec 上。 (如果你想先复制，可使用`clone`).

```rust
// vec4.rs
fn main() {
    let mut v1 = vec![1, 10, 5, 1, 2, 11, 2, 40];
    v1.sort();
    v1.dedup();
    assert_eq!(v1, &[1, 2, 5, 10, 11, 40]);
}
```

## 字符串

Rust 中的字符串比其他语言中的字符串更复杂一些; `String`类型，
像`Vec`，动态分配并可调整大小。 （所以它就像 C ++ 的`std::string`
但不像 Java 和 Python 的不可变字符串。）但是一个程序可能包含很多
_string literals {字符串常量}_（如"hello"）和系统语言应该能够在执行时静态存储这些
。 若放在微型嵌入式来说，这可能意味着让
他们在 廉价的 ROM 而不是 昂贵的 RAM（对低功耗设备来说，RAM 是
在功耗方面也很昂贵。）所以 _系统_ 语言必须具有
两种字符串，分配的或静态的。

所以"hello"不是`String`类型。 它是`＆str`类型（发音为'string slice'）。
这就像 C ++ 中 `const char*`和 `std::string` 之间的区别，除了
`＆str`更智能。 实际上，`＆str`和`String`有一个很好的的相似关系
就是`&[T]`到`Vec<T>`。

```rust
// string1.rs
fn dump(s: &str) {
    println!("str '{}'", s);
}

fn main() {
    let text = "hello dolly";  // the string slice
    let s = text.to_string();  // it's now an allocated string

    dump(text);
    dump(&s);
}
```

再次, 借用符号 可以迫使`String`成为`&str`, 就像`Vec<T>`能会被迫使进`&[T]`。

在引擎盖下，`String`基本上是一个`Vec<u8>`，和`&str`是一个`&[u8]`, 但是那些字节 _必须_ 表示有效的 UTF-8 文本。

就像一个向量，你可以`push`一个字符，和`pop`出`String`结尾:

```rust
// string5.rs
fn main() {
    let mut s = String::new();
    // 初始化 空的!
    s.push('H');
    s.push_str("ello");
    s.push(' ');
    s += "World!"; //  `push_str`的简写
    // 移除最后的char
    s.pop();

    assert_eq!(s, "Hello World");
}
```

`to_string`可以将许多类型转换为字符串。 (如果可以用"{}"打印它们，那么它们就可以被转换) . `format!`宏是像`println!`使用相同的格式字符串，来构建更复杂的字符串的一种非常有用的方法。

```rust
// string6.rs
fn array_to_str(arr: &[i32]) -> String {
    let mut res = '['.to_string();
    for v in arr {
        res += &v.to_string();
        res.push(',');
    }
    res.pop();
    res.push(']');
    res
}

fn main() {
    let arr = array_to_str(&[10, 20, 30]);
    let res = format!("hello {}", arr);

    assert_eq!(res, "hello [10,20,30]");
}
```

注意`&`在前面的`v.to_string()`- `&`符号是在一个字符串切片上，而不是`String`，因此,它需要一点手法来匹配。

用于切片的`..`也与字符串一起工作:

```rust
// string2.rs
fn main() {
    let text = "static";
    let string = "dynamic".to_string();

    let text_s = &text[1..];
    let string_s = &string[2..4];

    println!("slices {:?} {:?}", text_s, string_s);
}
// slices "tatic" "na"
```

但是，你不能索引字符串! 这是因为它们使用的是 唯(一)真(正)编码 UTF-8，其中的"character"可能是一个字节数.

```rust
// string3.rs
fn main() {
    let multilingual = "Hi! ¡Hola! привет!";
    for ch in multilingual.chars() {
        print!("'{}' ", ch);
    }
    println!("");
    println!("len {}", multilingual.len());
    println!("count {}", multilingual.chars().count());

    let maybe = multilingual.find('п');
    if maybe.is_some() {
        let hi = &multilingual[maybe.unwrap()..];
        println!("Russian hi {}", hi);
    }
}
// 'H' 'i' '!' ' ' '¡' 'H' 'o' 'l' 'a' '!' ' ' 'п' 'р' 'и' 'в' 'е' 'т' '!'
// len 25
// count 18
// Russian hi привет!
```

⚠️ 现在,让我们思考下 - 有 25 个字节，但是只有 18 个字符! 但是,如果你使用类似`find`的方法，你会得到一个有效的索引(如果有的话)和任意切片也会没事。

( Rust 的`char`类型是一个 4 字节的 Unicode 代码点。所以字符串不是字符
的数组!)

字符串切片可能会像 Vec 索引一样爆炸，因为它使用字节偏移量。在这种情况下，
该字符串由两个字节组成，所以试图拉出第一个字节，可是一个 Unicode 错误。 所以，
注意只使用来自字符串方法的有效偏移来切分字符串。

```rust
    let s = "¡";
    println!("{}", &s[0..1]); // <-- 错, 这是多字节字符的第一个字节
```

拆解字符串是一种常见和有用的方式。字符串的`split_whitespace`
方法返回会 _迭代器_，然后，我们就选择去如何处理它。一个主要做法是需要
创建拆分子串的 vec 。

`collect`非常普遍，因此需要一些关于，处于 collect 的线索，也就是看其
显式的类型。

```rust
    let text = "the red fox and the lazy dog";
    let words: Vec<&str> = text.split_whitespace().collect();
    // ["the", "red", "fox", "and", "the", "lazy", "dog"]
```

你也可以这样说，传递迭代器到`扩展{extend}`方法:

```rust
    let mut words = Vec::new();
    words.extend(text.split_whitespace());
```

在大多数语言中，我们将不得不制作这些 _分离的，已分配 字符串_，
而在这里， Vec 中的每个片段，都是从原始字符串中借用的。
我们所分配的是持有切片的位置。

看看这个可爱的双线`| |`; 我们从`chars`得到了一个迭代器，
并只要那些不是 空格 的字符。 再次，`collect`需要
一个线索（我们可能想要一个 字符串向量=String):

```rust
    let stripped: String = text.chars()
        .filter(|ch| ! ch.is_whitespace()).collect();
    // theredfoxandthelazydog
```

这`filter`方法接受一个 _闭包函数_，这是 Rust 的 lambdas/匿名函数。这里的参数类型从上下文中是清楚的，所以显式规则是放松了的。

就是这样，你可以这样搞定 chars 的显式循环，将返回的字符切片推送到一个可变的向量中，但是这个更短，阅读性很好 ( _当_ 你习惯了)，同样也很快。使用一个循环的方式不是一种 _错_ ，然而，我会鼓励你，也写这个一串过的版本。

## 插曲: 获取命令行参数

到目前为止，我们的节目都生活在对外界的无知之中;现在是时候给他们提供数据。

`std::env::args`是你如何访问命令行参数法宝;它返回一个迭代器作为字符串的参数,包括程序名。

```rust
// args0.rs
fn main() {
    for arg in std::env::args() {
        println!("'{}'", arg);
    }
}
```

```
src$ rustc args0.rs
src$ ./args0 42 'hello dolly' frodo
'./args0'
'42'
'hello dolly'
'frodo'
```

返回一个`Vec`会更好吗? 这很容易，使用`collect`制作迭代器，使用该向量的`skip`方法跳过程序名。

```rust
    let args: Vec<String> = std::env::args().skip(1).collect();
    if args.len() > 0 { // we have args!
        ...
    }
```

这还不错;几乎所有的语言都会这样做.

阅读单个参数的 更 Rust-y 特色的方法(传递一个整数值):

```rust
// args1.rs
use std::env;

fn main() {
    let first = env::args().nth(1).expect("please supply an argument");
    let n: i32 = first.parse().expect("not an integer!");
    // do your magic
}
```

`nth(1)`为您提供迭代器的第二个值，以及`expect`方法就像一个`unwrap`但带有可读的信息。

将字符串转换为数字很简单，但您需要指定值的类型 - 还有什么是可以`parse`的，你知道吗?

这个程序可能会恐慌{panic}，不过对笨拙的测试程序来说还能用。但不要太习惯于这种方便的想法。

## 匹配

我们提取俄罗斯问候语的`string3.rs`代码，并不是通常的写法。 进入 _match_ 的世界吧：

```rust
    match multilingual.find('п') {
        Some(idx) => {
            let hi = &multilingual[idx..];
            println!("Russian hi {}", hi);
        },
        None => println!("couldn't find the greeting, Товарищ")
    };
```

`match`包括几个 _模式{patterns}_ ，用一个匹配值和后跟 胖箭头,用逗号分隔。 它方便地，将`Options`中的值与`idx`束缚起来。 你 _必须_ 指定所有的可能性，所以我们必须处理`None`。

一旦你习惯了 (我的意思是，打多几遍)，感觉比`is_some`检查更自然，因检查还需要一个额外的`Option`存储。

但是，如果你对这里的失败不感兴趣，那么`if let`会是你的朋友:

```rust
    if let Some(idx) = multilingual.find('п') {
        println!("Russian hi {}", &multilingual[idx..]);
    }
```

如果你想做一次匹配，且 _只_ 对一个可能的结果感兴趣，那这无疑是个方便的写法。

`匹配{match}`也会像一个 C 的`switch`声明，就像其他 Rust 构造一样可以返回一个值:

```rust
    let text = match n {
        0 => "zero",
        1 => "one",
        2 => "two",
        _ => "many",
    };
```

这个`_`就像 C 的`default`，是一个备用情况。如果你不提供一个默认, `rustc`会认为这是一个错误。(在 C++中，最好的期望是一个警告，会说很多关于各自的语言)。

Rust 的`匹配`语句也可以匹配范围。 请注意，这些范围是有
_three{三个}_ 点 ，并且是全包含性的范围，所以第一个条件将匹配 3。

```rust
    let text = match n {
        0...3 => "small",
        4...6 => "medium",
        _ => "large",
     };
```

## 读取文件

下一步是向世界展示的，是 _阅读文件_。

回想一下，`expect`就像`unwrap`，但可自定义一个错误消息。
在这里我们会扔掉一些错误:

```rust
// file1.rs
use std::env;
use std::fs::File;
use std::io::Read;

fn main() {
    let first = env::args().nth(1).expect("please supply a filename");

    let mut file = File::open(&first).expect("can't open the file");

    let mut text = String::new();
    file.read_to_string(&mut text).expect("can't read the file");

    println!("file had {} bytes", text.len());

}
```

```
src$ file1 file1.rs
file had 366 bytes
src$ ./file1 frodo.txt
thread 'main' panicked at 'can't open the file: Error { repr: Os { code: 2, message: "No such file or directory" } }', ../src/libcore/result.rs:837
note: Run with `RUST_BACKTRACE=1` for a backtrace.
src$ file1 file1
thread 'main' panicked at 'can't read the file: Error { repr: Custom(Custom { kind: InvalidData, error: StringError("stream did not contain valid UTF-8") }) }', ../src/libcore/result.rs:837
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

所以，`open`会失败，因为该文件不存在或者我们不允许读它，然后，`read_to_string`也会失败，因为该文件不包含有效的 UTF-8。 (虽然这么说，但你可以使用`read_to_end`并将 其内容 用 一个字节 vec 替代。) 对于不太大的文件，一口一口地阅读它们是有用的，且直接。

如果你知道其他语言的文件处理，你可能会想要知道，文件什么时候 _关闭{closed}_。如果我们正在写入该文件，那么不关闭它，可能导致数据丢失。 但是这里啊，当函数结束时，文件就会被关闭，应`file`变量被 _释放{dropped}_ 了。

要知道"抛出错误(throw-catch)"的做法习惯是很糟糕的。你不会想将这些代码放入函数中，因为它知道，它可以很容易地使整个程序崩溃。 所以现在我们必须谈论，`File::open`到底返回什么。如果`Option`是一个值，其可能包含或不包含任何内容，那么`Result`就是一个可能包含某些内容或一个错误的值。 他们都明白`unwrap` (和它的表弟`expect`) ，但它们完全不同。 `Result`是由 _二种_ 类型参数定义的，分别是`Ok`值和`Err`值。 `Result`'盒子' 有两个隔间，一个标签是`Ok`而另一个是`Err`.

```rust
fn good_or_bad(good: bool) -> Result<i32,String> {
    if good {
        Ok(42)
    } else {
        Err("bad".to_string())
    }
}

fn main() {
    println!("{:?}",good_or_bad(true));
    //Ok(42)
    println!("{:?}",good_or_bad(false));
    //Err("bad")

    match good_or_bad(true) {
        Ok(n) => println!("Cool, I got {}",n),
        Err(e) => println!("Huh, I just got {}",e)
    }
    // Cool, I got 42

}
```

(实际的"错误"类型是随意的，很多人使用字符串，直到人们对 Rust 错误类型产生兴趣)。 这是返回一个值 _或_ 另一个值的方便方法。

这些文件读取函数版本是不会崩溃。 因它返回一个`Result`，当然还要 _呼叫者{caller}_ 决定如何处理这个错误。

```rust
// file2.rs
use std::env;
use std::fs::File;
use std::io::Read;
use std::io;

fn read_to_string(filename: &str) -> Result<String,io::Error> {
    let mut file = match File::open(&filename) {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    let mut text = String::new();
    match file.read_to_string(&mut text) {
        Ok(_) => Ok(text),
        Err(e) => Err(e),
    }
}

fn main() {
    let file = env::args().nth(1).expect("please supply a filename");

    let text = read_to_string(&file).expect("bad file man!");

    println!("file had {} bytes", text.len());
}
```

第一次匹配 从`Ok`安全地提取值，这就成了该 match 的值。 如果它是`Err`，就返回错误，并重新包装为一个`Err`。

第二个匹配返回字符串，包装为`Ok`，否则返回
（再一次）错误。`Ok`中的实际值不重要，所以我们用`_`忽略
它。

当一个函数的大部分在处理错误时，会不太好看; 那么
'快乐'就会迷失了。往往这个问题，伴有很多
明确的提前返回，或者是 _ignoring errors{忽视了错误}_ 。（顺便说一下，
这可是在 Rust 世界中，最接近邪恶的东西。）

幸运的是,有一个捷径。

`std::io`模块定义了一个别名，名为`io::Result<T>`类型，这与`Result<T,io::Error>`相同，但更容易的类型。

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = File::open(&filename)?; // <== ?
    let mut text = String::new();
    file.read_to_string(&mut text)?;
    Ok(text)
}
```

这里的`?`, 也几乎完全匹配了`File::open`所做的；如果其结果是一个错误，那么它将立即返回错误。 否则,它将返回`Ok`结果。
最后，我们仍然需要把该字符串包成一个 `Result` 类型。

2017 年是一个 Rust 的好年，还有酷酷的`?`也变得稳定。你也能看到用于旧代码的`try!`宏:

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = try!(File::open(&filename));
    let mut text = String::new();
    try!(file.read_to_string(&mut text));
    Ok(text)
}
```

总而言之，你可以编写完全安全，且不丑的 Rust 代码，不需要什么异常捕获。
