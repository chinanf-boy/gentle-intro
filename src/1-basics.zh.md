
# 基本

## 你好,世界!

自从编写第一个C版本以来,"hello world"的最初目的是测试编译器并运行一个实际的程序. 

```rust
// hello.rs
fn main() {
    println!("Hello, World!");
}
```

    $ rustc hello.rs
    $ ./hello
    Hello, World!

Rust是一种带分号的花括号语言, C ++ 风格和一个`main`函数 - 迄今为止,非常熟悉. `感叹号{!}`表明这是一个 _宏_ 呼叫. 对于 C ++ 程序员来说,这可能是一个退步,因为它们使用了非常愚蠢的C宏 - 但我可以确保这些宏能够更强大和更理智. 

对于其他任何人来说,它可能是"很棒,现在我必须记得何时该说砰!". 但是,编译器非常有用;如果你忽略了那个惊叹号,你会得到: 

    error[E0425]: unresolved name `println`
     --> hello2.rs:2:5
      |
    2 |     println("Hello, World!");
      |     ^^^^^^^ did you mean the macro `println!`?

学习一门语言意味着要熟悉它的错误. 试着看看编译器是一个严格但友好的帮手而不是电脑 _叫喊{shouting}_ 在你身上,因为你在开始时会看到很多红墨水. 对于编程人员来说,你的程序先炸毁比在实际的人类面前炸毁要好得多. 

下一步是介绍一个 _变量{variable}_: 

```rust
// let1.rs
fn main() {
    let answer = 42;
    println!("Hello {}", answer);
}
```

拼写错误是 _编译{compile}_ 错误,而不是类似 Python 或 JavaScript 等动态语言的运行时错误. 这将为您节省很多压力!如果我写了'answr'而不是'answer',编译器实际上是关于它的 _不错提示_ : 

    4 |     println!("Hello {}", answr);
      |                         ^^^^^ did you mean `answer`?

`println!`宏需要一个[格式字符串{format string}](https://doc.rust-lang.org/std/fmt/index.html)和一些 值 ;它与 Python 3使用的格式非常相似. 

另一个非常有用的宏是`assert_eq!`. 这是在Rust中进行测试的主力;您 _断言{assert}_ 两件事必须相等,如果不是,_panic{恐慌}_. 

```rust
// let2.rs
fn main() {
    let answer = 42;
    assert_eq!(answer,42);
}
```

这不会产生任何输出. 但一旦改变 42 到 40:

    thread 'main' panicked at
    'assertion failed: `(left == right)` (left: `42`, right: `40`)',
    let2.rs:4
    note: Run with `RUST_BACKTRACE=1` for a backtrace.

在 rust 中的  _运行时错误_ . 

## 循环和条件

有什么有趣的可以不止一次做:

```rust
// for1.rs
fn main() {
    for i in 0..5 {
        println!("Hello {}", i);
    }
}
```

这 _范围{range}_ 并不是全包括,而是`i`从0到4. 这是一种语言方便 _索引{indexes}_ 就像从0开始的数组

和有趣的事情要做 _有条件地{conditionally}_:

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

    even 0
    odd 1
    even 2
    odd 3
    even 4

`i % 2` 被 2 整除; 与 C 很像, 注意条件区域括号的配对

这个做了同样的事情,更有趣的方式写法:

```rust
// for3.rs
fn main() {
    for i in 0..5 {
        let even_odd = if i % 2 == 0 {"even"} else {"odd"};
        println!("{} {}", even_odd, i);
    }
}
```

传统上,编程语言 _声明{statements}_ (喜欢`if`) 和 _表达式{expressions}_ (喜欢`1 + i`) . 在 rust 里,几乎所有的东西都有价值并且可以表达. 不需要严重丑陋的 C '三元操作符'`i % 2 == 0?"even": "odd"`. 

⚠️请注意,这些区域中没有任何分号. `{"even"} else {"odd"}`

## 添加事物

计算机非常擅长算术. 这是第一次尝试添加从 0到4 的所有数字: 

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

但它没有编译: 

    error[E0384]: re-assignment of immutable variable `sum`
     --> add1.rs:5:9
    3 |     let sum = 0;
      |         --- first assignment to `sum`
    4 |     for i in 0..5 {
    5 |         sum += i;
      |         ^^^^^^^^ re-assignment of immutable variable

"不可变"? 一个变量不能 _变化vary{}_? `let`默认变量只能在声明时赋值. 添加魔术字`mut` (_请_ 让变量可变) 的诀窍: 

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

当来自其他语言时,这可能令人费解,其中默认情况下变量可以被重写. 它在运行时被分配了一个计算值到某个"可变"的东西 - 这不是一个 _不变{constant}_. 这也是这个词在数学中的用法,就像我们说'让 n 是 S 中最大的数字「'let n be the largest number in set S'」'. 

声明变量是默认 _只读_ 有原因的. 在更大的程序中,很难跟踪写入的地方. 所以 Rust 能够明确地表现出类似于可变性 ('写入能力') 的东西. 语言中有很多聪明,但它试图不隐藏任何东西. 

Rug既是静态类型又是强类型的,它们通常是混淆的,但是考虑 C (静态但弱类型) 和 Python  (动态但强类型) . 在静态类型中,类型在编译时是已知的,而动态类型仅在运行时知道. 

然而,此刻,它感觉像是 rust 了. _躲藏{hiding}_ 那些你的类型. 究竟是什么类型? `i`? 编译器可以从0开始, _类型推断_ 并提出`i32` (四字节有符号整数) . 

让我们做一个改变`0`到`0.0`. 然后我们得到错误: 

    error[E0277]: the trait bound `{float}: std::ops::AddAssign<{integer}>` is not satisfied
     --> add3.rs:5:9
      |
    5 |         sum += i;
      |         ^^^^^^^^ the trait `std::ops::AddAssign<{integer}>` is not implemented for `{float}`
      |

好了,蜜月结束了: 这意味着什么? 每个操作符 (像 `+=` ) 对应 _特质{trait}_ 而这是一个抽象的接口,必须为每种具体的类型来实现. 稍后我们将详细地处理 trait ,但是这里您需要知道的是`附加赋值{AddAssign}`实现了该名称`+=`运算符的 trait ,错误是说浮点数没有实现整数的`+=`运算符.  (运算符 trait 的完整列表) [在这里](https://doc.rust-lang.org/std/ops/index.html)) 

同样,Rust喜欢显式, 它不会默默地把那个整数转换成一个浮点数. 

我们必须显式地将该值 _转换{cast}_ 为浮点数.

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

_函数{Functions}_ 编译器是一个不会离开类型为你工作的地方. 

这实际上是一个深思熟虑的决定,因为语言,像 Haskell 拥有强大的类型推断,几乎没有显式的类型名称. 

这确实是好 Haskell 风格放在函数的显式类型签名. 

这个也是 rust 需要的.

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

Rust返回到了一个
本来的参数声明，其中类型跟在名称后面。 这就像
如何完成的在 Pascal等Algol 派生语言中。

再次，没有整数到浮点数的转换 - 如果你用'2'代替`2.0`，那么我们
得到一个明确的错误：

    8 |     let res = sqr(2);
      |                   ^ expected f64, found integral variable
      |

你很少会看到函数使用`return`声明. 更多时候,它会像这样:

```rust
fn sqr(x: f64) -> f64 {
    x * x
}
```

这是因为函数的主体（`{}`内部）具有 _最后值_ ,就像 if-as-an-expression. 

由于分号是由人的手指半自动插入的，因此您可以添加它
在 _最后值_ ，并得到以下错误：

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

这`()`类型是空的类型,没有什么结果,`无效{void}`,0,什么都没有. 一切Rus t都有个值,但有时它就是没有. 编译器知道这是常见的错误,实际上这就是 _该死的不寻常的_ ). 

> 也就是说, 如果你要返回, 就不能加 `分号{;}`

这种不能返回表达风格的几个例子:

```rust
// absolute value of a floating-point number
fn abs(x: f64) -> f64 {
    if x > 0.0 {
        x
    } else {
        -x
    }
}

// ensure the number always falls in the given range
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

它不是错误的使用`return`,但没有它,代码就会更干净. 但是对于从一个函数 _提前回来_ , 你仍然会使用`return`. 

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

起初这可能有些奇怪,然后最好用铅笔和纸质材料制作一些例子. 通常不是最多的 _高效_ 的方式. 然而,这样做

值也可以通过 _引用_. 一个引用是由创建`&`和 _取消引用_ 通过`*`. 

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

如果你想要一个函数来修改它的一个参数呢?输入 _可变引用_: 

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

这比 C ++ 更像 C ++ . 你必须明确地传递参数 (与`&`) 和明确 _非引用_ 同`*`. 然后键入`mut`, 因为它不是默认的.  (我一直觉得与 C 相比, C++ 引用太容易错过. ) 

基本上, Rust 是引入一些 _摩擦{friction}_ 这里,并不是那么巧妙地推动函数直接返回值. 幸运的是,  rust 的方式表达了"操作成功,结果就是这样". `mut`不需要那么频繁. 当我们有一个大对象并且不想复制它时,传递引用是很重要的. 

变量样式后的类型同样适用于`let`,当你真的想改变变量的类型: 

```rust
let bigint: i64 = 0;
```

## 学习在哪里找到绳索

现在是开始使用文档的时候了. 这将安装在您的机器上,您可以使用`rustup doc --std`在浏览器中打开它. 

注意 _搜索_ 在顶部,因为这将是你的朋友;它完全离线运行. 

假设我们想知道数学函数在哪里,所以搜索"cos". 前两个命中显示它为单精度和双精度浮点数字定义. 它定义在 _价值本身{value itself}_ 作为一种方法,像这样: 

```rust
let pi: f64 = 3.1416;
let x = pi/2.0;
let cosine = x.cos();
```

结果将是零;我们显然需要一个更权威的 pi-ness!

 (为什么我们需要一个明确的`f64`类型?因为没有它,常数也可以. `f32`或`f64`,这些都是非常不同的. )

让我引用的例子`cos`,但写为一个完整的程序(`assert!`的表弟`assert_eq!`;

```rust
fn main() {
    let x = 2.0 * std::f64::consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

`std::f64::consts::PI`是一口子! `::`意味着同样的在c++中,(通常使用"."在其他语言) - 这是一个完全合格的名字。 我们得到
这个全名来自第二次搜索“PI”的命中。

到目前为止,我们的小 Rust 项目一直免费`import`和`exclude`东西会慢下来的讨论"Hello World"程序. 我们让这个程序可读性更强的`use`声明:

```rust
use std::f64::consts;

fn main() {
    let x = 2.0 * consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

为什么我们现在不需要这样做？
这是因为 Rust 有助于使许多基本功能无需显示
通过 Rust _prelude_ 显式 `use` 的使用陈述。

## 数组和切片

所有静态类型的语言都有 _数组_ ,这些 值 包装着鼻子到尾巴的记忆. 数组是 _索引_ 从零开始: 

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

    first 10
    [0] = 10
    [1] = 20
    [2] = 30
    [3] = 40
    length 4

在这种情况下,Rust 知道 _究竟_ 阵列有多大,如果你尝试访问`arr[4]`这将是一个 _编译错误_ . 

学习一门新语言往往涉及到 _忘却_ 来自旧语言的已知心理习惯;如果你是一个 Pythonista,那么这些括号表示`list`. 我们会得到 Rust 的等同物`list`很快,但数组不是你正在寻找的机器人; 他们是 _大小固定_. 他们可以 _易变的_ (如果我们问得好) ,但你不能添加新的元素. 

在 Rust 中不常使用数组,因为数组的类型包含他们大小. 示例中的数组的类型是`[i32;4]`;类型`[10,20]`将会`[i32;2]`等等: 他们有 _不同类型_.  所以他们是私生子,作为函数论证. 

什么 _是_ 常用的是 _切片_. 你可以把它们看作是 _快照_ 一个基本的值数组. 它们的行为很像一个数组, _知道他们的尺寸_ 不像那些危险的动物C指针. 

注意两个重要的事情 - 如何写一个切片的类型,和你必须使用`&`将其传递给函数. 

```rust
// array2.rs
// read as: slice of i32
fn sum(values: &[i32]) -> i32 {
    let mut res = 0;
    for i in 0..values.len() {
        res += values[i]
    }
    res
}

fn main() {
    let arr = [10,20,30,40];
    // look at that &
    let res = sum(&arr);
    println!("sum {}", res);
}
```

忽略代码`sum`一阵子,看看`[i32]`. rust 数组和切片之间的关系类似于 C数组和指针 之间的关系,除了两个重要的区别: rust的切片跟踪它们的大小 (如果你 尝试访问这个大小之外 会 _panic_) ,并且你必须明确地说你想把数组作为一个切片传递. 使用`&`操作员. 

C程序员发音`&`作为"地址",rust程序员会 _借用{borrow}_ 它. 这将是学习 rust 的关键词. 借用是编程中常见的模式的名称;每当你通过引用传递 (几乎总是发生在动态语言中) 或 在C中传递指针时,原始所有者所拥有的任何东西被 _借用_. 

## 切片切割

不能以通常的方式打印出一个数组. `{}`但你可以做一个 _debug_ 打印与`{:?}`. 

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

    ints [1, 2, 3]
    floats [1.1, 2.1, 3.1]
    strings ["hello", "world"]
    ints_ints [[1, 2], [10, 20]]

数组的数组是没问题,但重要的是,数组包括内容 _只有一个类型_. 数组中的值排列在一起
在内存中，以便他们非常高效地访问。

刚和一个显式声明一个变量类型,你知道将是错误的:

```rust
let var: () = [1.1, 1.2];
```

这是信息错误:

    3 |     let var: () = [1.1, 1.2];
      |                   ^^^^^^^^^^ expected (), found array of 2 elements
      |
      = note: expected type `()`
      = note:    found type `[{float}; 2]`

(`{float}`意思是"一些浮点类型不完全指定的)

切片给你不同 _快照_ 的 _相同_ 数组:

```rust
// slice1.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice1 = &ints[0..2];
    let slice2 = &ints[1..];  // open range!

    println!("ints {:?}", ints);
    println!("slice1 {:?}", slice1);
    println!("slice2 {:?}", slice2);
}
```

    ints [1, 2, 3, 4, 5]
    slice1 [1, 2]
    slice2 [2, 3, 4, 5]

这是一个简洁的符号,类似于 Python 片但是有很大区别: 从未有过任何数据的一个副本. 这些 _切片_ 他们的数据来自 `借用{borrow}` 他们的数组. 他们有一个非常亲密的关系数组,和 rust 花很多精力来确保不分解的关系. 

## 可选值

切片,就像数组一样,可以 _索引_. Rust在编译时知道数组的大小,但只有在运行时才知道分切片的大小. 所以`s[i]`在运行时会引起超出界限的错误 _恐慌{panic}_. 这实际上并不是你想要发生的事情 - 它可能是 安全发射中止 与 遍布佛罗里达州的非常昂贵卫星的散射切片 之间的区别. 还有 _没有例外_. 

让它沉入水中,因为它是一种冲击. 你不能在某些 try-block 中包装可怕的可能问题代码,并且"捕获错误" - 至少不是你每天都想使用的方式. 那么 Rust 如何安全?

有一种切片方法`get`这并不恐慌{panic}. 但是它返回了什么?

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

`last`失败 (我们忘记了基于零的索引) ,但返回了一个叫做的东西`None`. `first`很好,但是作为一个 值包装 出现`Some`. 欢迎来到`Options`类型!它可能是`Some` _或者_ `None`. 

这`Option`类型有一些有用的方法:

```rust
    println!("first {} {}", first.is_some(), first.is_none());
    println!("last {} {}", last.is_some(), last.is_none());
    println!("first value {}", first.unwrap());

// first true false
// last false true
// first value 1
```

如果你 _打开{unwrap}_ `last`,你会得到一个恐慌{panic}. 但至少你可以叫`is_some`确保 - 例如,如果你有一个不同的默认 没有价值的变量:

```rust
    let maybe_last = slice.get(5);
    let last = if maybe_last.is_some() {
        *maybe_last.unwrap()
    } else {
        -1
    };
```

注意`*` - 内部的精确类型`Some`是`&i32`,这是一个引用. 我们需要这回到一个`i32`的值. 

冗长的,这是一个快捷方式-`unwrap_or`, 如果返回的值是`Option`和是`None`. - 类型必须匹配`get`一个引用. 所以你必须组成`&i32`与`&-1`. 最后再次使用`*`获得价值`i32`. 

```rust
    let last = *slice.get(5).unwrap_or(&-1);
```

这很容易错过`&`,但你在这里有编译器. 如果它是`-1`,`rustc` says 'expected &{integer}, found integral variable' ,然后'help: try`&-1`". 

你可以想到`Option`作为一个可能包含一个值的 Box,或者什么也不是 (`None`)  (在 Haskell, 它被称为`Maybe`) . 它可能包含 _任何_ 有价值的,就是这样 _类型参数_ . 在这种情况下,完整的类型是`Option<&i32>`,使用 C ++ 风格的表示法 _仿制药{generics}_.  打开这个Box可能会引起爆炸,但不像施罗德林格的猫,我们可以事先知道它是否包含一个值. 

这是非常常见的 rust 函数/方法返回这些可能的Box,所以学习如何舒适地[使用它们](https://doc.rust-lang.org/std/option/enum.Option.html). 

## 向量

我们将再次返回切片方法,但首先返回 向量{Vec} . 这些是 _可重复使用的_ 数组和行为很像 Python `List`和 C++ `std::vector`. rust 的`Vec`事实上,不同的是,你可以将额外的值附加到一个向量上,注意它必须声明为可变的. 

```rust
// vec1.rs
fn main() {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v.push(30);

    let first = v[0];  // will panic if out-of-range
    let maybe_first = v.get(0);

    println!("v is {:?}", v);
    println!("first is {}", first);
    println!("maybe_first is {:?}", maybe_first);
}
// v is [10, 20, 30]
// first is 10
// maybe_first is Some(10)
```

一个常见的初学者错误是忘记`mut`你会得到一个有用的错误信息: 

    3 |     let v = Vec::new();
      |         - use `mut v` here to make mutable
    4 |     v.push(10);
      |     ^ cannot borrow mutably

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

    let slice = &v[1..];
    println!("slice is {:?}", slice);
}
```

那么小,那么重要的借用`&`是为了 _强迫_ 向量进入切片. 它是完全有意义的,因为向量也在监视数组的值,数组的 _动态地_ 分配也不同.

如果你来自一种动态的语言,那么现在是时候开始讨论了. 在系统语言中,程序存储器有两种: 堆栈和堆. 在堆栈上分配数据非常简单,但是堆栈是有限的;通常是按兆字节的顺序. 堆可以是千兆字节,但是分配相对昂贵,并且这样的内存必须是 _释放_ 后来. 在所谓的'管理'的语言 (如java,Go和所谓的脚本语言) 这些细节都隐藏在你的便利的市政工程称 _垃圾收集器_. 一旦系统确定数据不再引用的其他数据,它回到可用内存池. 

一般来说，这是一个值得付出的代价。 玩堆栈非常不安全，
因为如果你犯了一个错误，覆盖当前的返回地址在一次函数中, 那么你倒下了

我写的第一个C程序（在DOS PC上）
拿出整台电脑。 Unix系统总是表现得更好，只有 _segfault_ 进程死掉了
。 为什么这比Rust（或Go）程序恐慌{panic}更糟？
因为当原始问题发生了会发生恐慌{panic}, 而不是之前
已经变得毫无希望地困惑和吃掉你所有的功课。 

恐慌{panic} _memory safe_ 因为它们在任何非法访问内存之前发生。 这是一个常见原因 C 中的安全问题，因为所有内存访问都是不安全的并且是狡猾的攻击者
可以利用这个弱点。

恐慌{panic}听起来绝望而无计划，但 Rust 的恐慌{panic}是结构化的 - 堆栈是 _unwound_
就像例外情况一样。 所有分配的对象都被删除，并且生成一个回溯。

垃圾收集的缺点？ 首先是它是浪费内存，这是
那些越来越统治我们世界的小型嵌入式微芯片中占有重要地位
其次是它会在最糟糕的时候决定进行 _现在_ 清理
。 （这个妈妈的比喻是，她想在你与新的情人舞动时打扫房间
）。 这些嵌入式系统需要对事物做出响应
当它们发生时（'实时'），并且不能容忍计划外的爆发
清洗。 Roberto Ierusalimschy，Lua的首席设计师（最优雅的动态语言设计师之一）
说，他不想乘坐飞机时
依靠垃圾收集软件。

回到 vectors ：当一个 vectors 被修改或创建时，它从堆中分配并变成
该内存的 _owner_ . 切片从 vectors 的内存中借用。
当 vectors 死亡或 _drops_ 时，切片也会跟着 vectors。

## 迭代器

我们到目前为止没有提及的关键部分  rust 难题 - 迭代器. 

for循环在使用迭代器范围是(`0..n`其实是类似于the Python 3`range`功能). 

迭代器很容易定义. 这是一个"对象"`next`方法返回一个`Option`. 只要这个价值不是`None`,我们一直`next`: 

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

而这正是`for var in iter {}`. 

这似乎是定义for循环的低效方式,但是`rustc`在发布模式中会进行疯狂的优化,并且它会和`while`循环一样快. 

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

失败,但有帮助: 

    4 |     for i in arr {
      |     ^ the trait `std::iter::Iterator` is not implemented for `[{integer}; 3]`
      |
      = note: `[{integer}; 3]` is not an iterator; maybe try calling
       `.iter()` or a similar method
      = note: required by `std::iter::IntoIterator::into_iter`

以下`rustc`的建议,下面的程序按预期工作. 

```rust
// iter3.rs
fn main() {
    let arr = [10, 20, 30];
    for i in arr.iter() {
        println!("{}", i);
    }

    // slices will be converted implicitly to iterators...
    let slice = &arr;
    for i in slice {
        println!("{}", i);
    }
}
```

实际上,迭代数组或切片这种方式使用起来效率更高`for i in 0..slice.len() {}`因为Rust不必痴迷于检查每个索引操作. 

我们之前有一个总结一系列整数的例子. 它涉及一个`mut`变量和循环. 这是 _惯用的_ ,总结评分 sum 的方式: 

```rust
// sum1.rs
fn main() {
    let sum: i32  = (0..5).sum();
    println!("sum was {}", sum);

    let sum: i64 = [10, 20, 30].iter().sum();
    println!("sum was {}", sum);
}
```

请注意,这是其中一个需要明确说明的情况. _类型_ 对于变量,因为否则  rust 没有足够的信息. 这里我们用两个不同的整数做和,没有问题.  (如果用尽所有的名字来创建一个新的同名变量也是没有问题的. ) 

在这样的背景下,更多的[切片法](https://doc.rust-lang.org/std/primitive.slice.html)将更有意义.  (另一个文档提示;在每个文档页的右边有一个'[-]单击该按钮以折叠方法列表. 然后你可以扩展任何看起来很有趣的细节. 任何看起来怪异的东西,现在就忽略它. 

这个`windows`方法提供了一个迭代器的重叠值窗口. 

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

有一个有用的小宏`vec!`用于初始化向量. 注意你可以使用`pop` _去除{remove}_ 向量结束时的值和 _延伸_ 一个使用任何兼容迭代器的向量. 

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

向量相互比较,以值为单位. 

可以将值插入到任意位置的向量中. `插入{insert}`或者使用`去除{remove}`移除. 
这不像从那以后的推动和弹出一样有效
这些值将不得不被移动以腾出空间，所以请小心这些操作
向量.

 vec 具有大小和 _capacity{容量}_。 如果你清除了一个 vec ，它的大小就变成了零，
但它仍保留其旧容量。 所以用`push`等来填充只需要
当尺寸大于该容量时重新分配。

vec 可以排序,然后可以删除重复的 - 这些操作就在 vec 上 . (如果你想先复制,使用`clone`). 

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

Rust中的字符串比其他语言中的字符串更复杂一些; `String`类型，
像`Vec`，动态分配并可调整大小。 （所以它就像 C ++ 的`std::string`
但不像Java和 Python 的不可变字符串。）但是一个程序可能包含很多
_string literals {切片}_（如“hello”）和系统语言应该能够存储
这些静态可执行文件本身。 在嵌入式微软中，这可能意味着推出
他们在 廉价的ROM 而不是 昂贵的RAM（对于低功耗设备，RAM是
在功耗方面也很昂贵。）_系统_ 语言必须具有
两种字符串，分配或静态。

所以"hello"不是`String`类型。 它是`＆str`类型（发音为'string slice'）。
这就像 C ++ 中 `const char*`和 `std::string` 之间的区别，除了
`＆str`更智能。 实际上，`＆str`和`String`有一个很好的的相似关系
如`&[T]`到`Vec<T>`。

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

再次,借用操作 可以强制`String`成`&str`, 就像`Vec<T>`可能会被强迫进入`&[T]`. 

在引擎盖下,`String`基本上是`Vec<u8>`和`&str`是`[u8]`, 但是那些字节 _必须_ 表示有效的UTF-8文本. 

就像一个向量,你可以`push`和`pop`一个`String`: 

```rust
// string5.rs
fn main() {
    let mut s = String::new();
    // initially empty!
    s.push('H');
    s.push_str("ello");
    s.push(' ');
    s += "World!"; // short for `push_str`
    // remove the last char
    s.pop();

    assert_eq!(s, "Hello World");
}
```

`to_string`可以将许多类型转换为字符串.  (如果可以用"{}"显示它们,那么它们可以被转换) . `format!`宏是使用相同的格式字符串来构建更复杂的字符串的一种非常有用的方法就像`println!`.

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

注意`&`在前面的`v.to_string()`- 在字符串条上定义运算符,而不是`String`因此,它需要一点说服力来匹配. 

用于切片的符号也与字符串一起工作: 

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

但是,你不能索引字符串! 这是因为它们使用一个真正的编码UTF-8,其中"character"可能是字节数. 

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

⚠️ 现在,让我们思考 - 有25个字节,但是只有18个字符!但是,如果你使用类似的方法`find`,你会得到一个有效的索引(如果发现)和切片会没事的. 

(  rust `char{字符}`类型是一个4字节的Unicode代码点. 字符串不是字符
的数组!)

字符串切片可能会像 Vec 索引一样爆炸，因为它使用字节偏移量。 在这种情况下，
该字符串由两个字节组成，所以试图拉出第一个字节是一个Unicode错误。 所以，
注意只使用来自字符串方法的有效偏移来切分字符串。

```rust
    let s = "¡";
    println!("{}", &s[0..1]); <-- bad, first byte of a multibyte character
```

打破字符串是一种流行和有用的消遣方式。 字符串`split_whitespace`
方法返回 _iterator_，然后我们选择如何处理它。 共同需要
是创建拆分子串的 vec 。

`collect`非常普遍，因此需要收集一些关于 collect _它_ 的线索
显式类型。

```rust
    let text = "the red fox and the lazy dog";
    let words: Vec<&str> = text.split_whitespace().collect();
    // ["the", "red", "fox", "and", "the", "lazy", "dog"]
```

你也可以这样说,通过迭代器进入`扩展{extend}`方法:

```rust
    let mut words = Vec::new();
    words.extend(text.split_whitespace());
```

在大多数语言中，我们将不得不使这些 _分离的strings_，
而在这里 Vec 中的每个片段都是从原始字符串中借用的。
我们所分配的是保留切片的空间。

看看这个可爱的双线 `| |`; 我们得到了一个字符的迭代器，
并只采取那些不是 空格 的字符。 再次，`collect` 的需要
一个线索（我们可能想要一个字符向量，比如说):

```rust
    let stripped: String = text.chars()
        .filter(|ch| ! ch.is_whitespace()).collect();
    // theredfoxandthelazydog
```

这`filter`方法接受一个 _闭包函数_,这是 Rust  的 匿名功能. 这里的参数类型从上下文中是清楚的,所以显式规则是放松的. 

是的,你可以这样做,作为一个字符的显式循环,将返回的片段推送到一个可变的向量中,但是这个更短,读得很好 ( _当_ 你习惯了) ,同样快. 它不是 _罪{sin}_ 然而,使用循环,我鼓励你也写这个版本. 

## 插曲: 获取命令行参数

到目前为止,我们的节目都生活在对外界的无知之中;现在是时候给他们提供数据. 

`std::env::args`是你如何访问命令行参数;它返回一个迭代器作为字符串的参数,包括程序名. 

```rust
// args0.rs
fn main() {
    for arg in std::env::args() {
        println!("'{}'", arg);
    }
}
```

    src$ rustc args0.rs
    src$ ./args0 42 'hello dolly' frodo
    './args0'
    '42'
    'hello dolly'
    'frodo'

返回一个`Vec`会更好吗?这很容易使用`collect`使用迭代器来创建该向量`skip`方法跳过. 

```rust
    let args: Vec<String> = std::env::args().skip(1).collect();
    if args.len() > 0 { // we have args!
        ...
    }
```

这很好;几乎所有的语言都会这样做. 

阅读单个参数 (连同解析整数值一起) 的更多Rust-y方法: 

```rust
// args1.rs
use std::env;

fn main() {
    let first = env::args().nth(1).expect("please supply an argument");
    let n: i32 = first.parse().expect("not an integer!");
    // do your magic
}
```

`nth(1)`为您提供迭代器的第二个值,以及`expect`就像一个`unwrap`带有可读的信息. 

将字符串转换为数字很简单,但您需要指定值的类型 - 还有什么可以`parse`知道吗?

这个程序可能会恐慌{panic},这对于笨拙的测试程序来说很好. 但是,不要因为这种便利的习惯而难以忍受. 

## 匹配

我们提取俄罗斯问候语的`string3.rs`中的代码并不是如此
通常写。 输入 _match_：

```rust
    match multilingual.find('п') {
        Some(idx) => {
            let hi = &multilingual[idx..];
            println!("Russian hi {}", hi);
        },
        None => println!("couldn't find the greeting, Товарищ")
    };
```

`match`包括几个 _模式{patterns}_ 用一个匹配值跟随 胖箭头,用逗号分隔. 它方便地从`Options`把它与`idx{index}`束缚起来.  你 _必须_ 指定所有的可能性,所以我们必须处理`None`.

一旦你习惯了 (我的意思是,把它打满几遍) ,感觉比自然的更自然. `is_some`检查需要一个额外的存储`Option`.

但是如果你对这里的失败不感兴趣,那么`if let`是你的朋友: 

```rust
    if let Some(idx) = multilingual.find('п') {
        println!("Russian hi {}", &multilingual[idx..]);
    }
```

这是方便的,如果你想做一场 match,是 _只有_ 对一个可能的结果感兴趣. 

`匹配{match}`也可以像一个C`switch`声明中,就像其他Rust构造一样
可以返回一个值:

```rust
    let text = match n {
        0 => "zero",
        1 => "one",
        2 => "two",
        _ => "many",
    };
```

` _ `就像C`默认的 - 这是一个备用情况.如果你不提供一个默认, 会认为这是一个错误. 

(在c++中最好的期望是一个警告,获利很多关于各自的语言). 

rust的`匹配`语句也可以匹配范围. 请注意，这些范围有
_three{三个}_ 点 并且是包含范围，所以第一个条件将匹配3。

```rust
    let text = match n {
        0...3 => "small",
        4...6 => "medium",
        _ => "large",
     };
```

## 读取文件

下一步是向世界展示我们的程序 _阅读文件_. 

回想一下,`expect`就像`unwrap`但给了一个自定义的错误消息. 
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

    src$ file1 file1.rs
    file had 366 bytes
    src$ ./file1 frodo.txt
    thread 'main' panicked at 'can't open the file: Error { repr: Os { code: 2, message: "No such file or directory" } }', ../src/libcore/result.rs:837
    note: Run with `RUST_BACKTRACE=1` for a backtrace.
    src$ file1 file1
    thread 'main' panicked at 'can't read the file: Error { repr: Custom(Custom { kind: InvalidData, error: StringError("stream did not contain valid UTF-8") }) }', ../src/libcore/result.rs:837
    note: Run with `RUST_BACKTRACE=1` for a backtrace.

所以`open`可以失败,因为该文件不存在或者我们不允许读它,然后呢`read_to_string`可能会失败,因为该文件不包含有效的UTF-8.  (这是足够的,你可以使用`read_to_end`并将其内容放入字节替代的 vec 中. ) 对于不太大的文件，一口一口地阅读它们是有用的
直截了当。 

如果你知道其他语言的文件处理的任何事情,你可能会怀疑文件是什么 _关闭{closed}_. 如果我们正在写这个文件,那么不关闭它可能导致数据丢失. 但是当函数结束时,这里的文件被关闭,而`File`变量是 _下降{dropped}_. 

这种"抛出错误"的做法是习惯太过分了. 你不想将这些代码放入函数中,因为它知道它可以很容易地使整个程序崩溃. 所以现在我们必须谈论到底`File::open` 返回什么. 如果`Option`是一个值,可能包含或不包含任何内容`Result`是一个可能包含某些内容或错误的值. 他们都明白`unwrap` (和它的表弟`expect`) 但它们完全不同. `Result`由 _二_ 类型参数定义,为`Ok`价值和`Err`价值. `Result`Box有两个隔间,一个标签`Ok`而另一个`Err`.

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

 (实际的"错误"类型是任意的,很多人使用字符串,直到它们对  rust 错误类型感到满意) . 这是一种方便的方法. _任何一个_ 返回一个值 _或_ 另一个. 

该版本的文件读取功能不会崩溃. 它返回一个`Result`这就是 _呼叫者{caller}_ 谁必须决定如何处理这个错误. 

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

第一次 match 安全地提取价值`Ok`这就成了match的价值所在. 如果是`Err`它返回错误,重新包装为`Err`.

第二个匹配返回字符串，包装为`Ok`，否则返回
（再次）错误。 `Ok`中的实际值不重要，所以我们忽略
它用`_`。

这不太好看; 当一个函数的大部分是错误处理时，那么
'快乐之路'迷路了。 去往往有这个问题，有很多
明确的早期返回，或者只是 _ignoring errors{忽视错误}_ 。 （也就是说，顺便说一下，
在Rust世界中最接近邪恶的东西.）

幸运的是,有一个捷径. 

`std::io`模块定义了一个类型别名`io::Result<T>`这正与`Result<T,io::Error>`相同且更容易的类型. 

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = File::open(&filename)?;
    let mut text = String::new();
    file.read_to_string(&mut text)?;
    Ok(text)
}
```

那`?`, 也几乎完全匹配`File::open`,如果结果是一个错误,那么它将立即返回错误. 否则,它将返回`Result`. 

最后,我们仍然需要结束字符串作为结果. `?`是一个很酷的东西慢慢就成了稳定. 你仍能看到宏`try!`用于旧代码:

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = try!(File::open(&filename));
    let mut text = String::new();
    try!(file.read_to_string(&mut text));
    Ok(text)
}
```

总而言之，可以编写完全安全的Rust，而不是丑陋的
需要例外。
