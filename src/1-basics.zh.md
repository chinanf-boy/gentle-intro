
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

Rust是一种带分号的花括号语言,C ++风格的评论和a`主要`功能 - 迄今为止,非常熟悉. 感叹号表明这是一个_宏_呼叫. 对于C ++程序员来说,这可能是一个关闭,因为它们使用了非常愚蠢的C宏 - 但我可以确保这些宏能够更强大和更理智. 

对于其他任何人来说,它可能是"很棒,现在我必须记得何时该说砰!". 但是,编译器非常有用;如果你忽略了那个惊叹号,你会得到: 

    error[E0425]: unresolved name `println`
     --> hello2.rs:2:5
      |
    2 |     println("Hello, World!");
      |     ^^^^^^^ did you mean the macro `println!`?

学习一门语言意味着要熟悉它的错误. 试着看看编译器是一个严格但友好的帮手而不是电脑_叫喊_在你身上,因为你在开始时会看到很多红墨水. 对于编程人员来说,你的程序比在实际的人类面前炸毁要好得多. 

下一步是介绍一个_变量_: 

```rust
// let1.rs
fn main() {
    let answer = 42;
    println!("Hello {}", answer);
}
```

拼写错误是_编_错误,而不是类似Python或JavaScript等动态语言的运行时错误. 这将为您节省很多压力!如果我写了'answr'而不是'answer',编译器实际上是_不错_关于它: 

    4 |     println!("Hello {}", answr);
      |                         ^^^^^ did you mean `answer`?

该`调用println!`宏需要一个[格式字符串](https://doc.rust-lang.org/std/fmt/index.html)和一些价值观;它与Python 3使用的格式非常相似. 

另一个非常有用的宏是`ASSERT_EQ!`. 这是在Rust中进行测试的主力;您_断言_两件事必须相等,如果不是,_恐慌_. 

```rust
// let2.rs
fn main() {
    let answer = 42;
    assert_eq!(answer,42);
}
```

这不会产生任何输出. 

    thread 'main' panicked at
    'assertion failed: `(left == right)` (left: `42`, right: `40`)',
    let2.rs:4
    note: Run with `RUST_BACKTRACE=1` for a backtrace.

但改变42 - 40:_运行时错误_在生锈. 

## 循环和得

有什么有趣的可以做不止一次:

```rust
// for1.rs
fn main() {
    for i in 0..5 {
        println!("Hello {}", i);
    }
}
```

的_范围_不包容,所以呢`我`从0到4. _这是方便alanguage中_诸如数组从0. 

和有趣的事情要做_有条件地_:

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

`我% 2`如果2可以分为是零`我`清晰地;_锈使用c风格的运营商. _有_必须_绕着街区使用花括号. 

这个做了同样的事情,写在一个更有趣的方式:

```rust
// for3.rs
fn main() {
    for i in 0..5 {
        let even_odd = if i % 2 == 0 {"even"} else {"odd"};
        println!("{} {}", even_odd, i);
    }
}
```

传统上,编程语言_声明_ (喜欢`如果`) 和_表达式_ (喜欢`1 + I`) . 在鲁斯特里,几乎所有的东西都有价值并且可以表达. 严重丑陋的C'三元操作符'`i%2 == 0?"偶": "奇"`不需要. 

请注意,这些块中没有任何分号. 

## 添加事物

计算机非常擅长算术. 这是第一次尝试添加从0到4的所有数字: 

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

"不可变"?一个变量不能_变化_?`让`默认变量只能在声明时赋值. 添加魔术字`mut` (_请_makethis变量可变) 的诀窍: 

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

当来自其他语言时,这可能令人费解,其中默认情况下变量可以被重写. 什么使得某个"可变"的东西是它在运行时被分配了一个计算值 - 这不是一个_不变_. 这也是这个词在数学中的用法,就像我们说'小n是S中最大的数字'. 

声明变量是有原因的_只读_默认. 在更大的程序中,很难跟踪写入的地方. 所以Rust能够明确地表现出类似于可变性 ('写入能力') 的东西. 语言中有很多聪明,但它试图不隐藏任何东西. 

Rug既是静态类型又是强类型的,它们通常是混淆的,但是考虑C (静态但弱类型) 和Python (动态但强类型) . 在静态类型中,类型在编译时是已知的,而动态类型仅在运行时知道. 

然而,此刻,它感觉像是生锈了. _躲藏_那些类型来自你. 究竟是什么类型?`我`?编译器可以从0开始,_类型推断_并提出`i32` (四字节有符号整数) . 

让我们做一个改变`零`进入之内`零`. 然后我们得到错误: 

    error[E0277]: the trait bound `{float}: std::ops::AddAssign<{integer}>` is not satisfied
     --> add3.rs:5:9
      |
    5 |         sum += i;
      |         ^^^^^^^^ the trait `std::ops::AddAssign<{integer}>` is not implemented for `{float}`
      |

好了,蜜月结束了: 这意味着什么?每个操作符 (类) `+=`与A对应_特质_这是一个抽象的接口,必须为每种具体的类型来实现. 稍后我们将详细地处理特性,但是这里您需要知道的是`附加赋值`实现该特性的名称`+=`运算符,错误是说浮点数不能实现整数的这个运算符.  (运算符特性的完整列表) [在这里](https://doc.rust-lang.org/std/ops/index.html)) 

同样,Rust喜欢显式ℴℴ它不会默默地把那个整数转换成一个浮标. 

我们必须_铸造_这个值显式地一个浮点值. 

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

_功能_编译器是一个地方不会为你工作类型. 

这实际上是一个深思熟虑的决定,因为语言,像Haskell havesuch强大的类型推断,几乎没有显式的类型名称. 

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

'sactually好Haskell风格放在函数的显式类型签名. 

这个总是生锈需要. `2.0`与`2`然后就会明显错误:

    8 |     let res = sqr(2);
      |                   ^ expected f64, found integral variable
      |

你很少会看到函数使用`返回`声明. 

```rust
fn sqr(x: f64) -> f64 {
    x * x
}
```

Moreoften,它会像这样:`{ }`itslast表达式的值,就像if-as-an-expression. 

因为分号插入半自动生成由人类手指,可以添加ithere并得到以下错误:

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

的`()`类型是空的类型,没有什么结果,`无效`零,什么都没有. _一切Rusthas值,但有时它只是没有. _编译器知道这是常见的错误,实际上_该死的不寻常的_这是). 

这种不能回转表达风格的几个例子:

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

它不是错误的使用`返回`,但没有它,代码就会更干净. 你仍然会使用`返回`对于_提前回来_从一个函数. 

一些操作可以被优雅地表达_递归_: 

```rust
fn factorial(n: u64) -> u64 {
    if n == 0 {
        1
    } else {
        n * factorial(n-1)
    }
}
```

起初这可能有些奇怪,然后最好用铅笔和纸质材料制作一些例子. 通常不是最多的_高效_然而,这样做的方式. 

值也可以通过_参考_. 一个参考是由创建`&`和_取消引用_通过`*`. 

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

如果你想要一个函数来修改它的一个参数呢?输入_可变引用_: 

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

这比C ++更像C ++. 你必须明确地传递参数 (与`&`) 和明确_提领_同`*`. 然后投掷`穆特`因为它不是默认的.  (我一直觉得C++引用太容易错过C相比. ) 

基本上,锈是引入一些_摩擦_这里,并不是那么巧妙地推动你直接从函数返回值. 幸运的是,生锈的方式表达了"操作成功,结果就是这样". `MUT`不需要那么频繁. 当我们有一个大对象并且不想复制它时,传递引用是很重要的. 

变量样式后的类型适用于`让`同样,当你真的想改变变量的类型: 

```rust
let bigint: i64 = 0;
```

## 学习在哪里找到绳索

现在是开始使用文档的时候了. 这将安装在您的机器上,您可以使用`斯塔普博士`在浏览器中打开它. 

注意_搜索_在顶部,因为这将是你的朋友;它完全离线运行. 

假设我们想知道数学函数在哪里,所以搜索"COS". 前两个命中显示它为单精度和双精度浮点数字定义. 它定义在_价值本身_作为一种方法,像这样: 

```rust
let pi: f64 = 3.1416;
let x = pi/2.0;
let cosine = x.cos();
```

结果将是零;我们显然需要一个更权威的PIN!

 (为什么我们需要一个明确的`f64`类型?因为没有它,常数也可以. `f32`或`f64`,这些都是非常不同的. )

让我引用的例子`因为`,但写为一个完整的程序(`维护!`的表弟`assert_eq !`;

```rust
fn main() {
    let x = 2.0 * std::f64::consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

`表达式必须是真实的):`是一口!`::`意味着同样的在c++中,(通常使用". _在其他语言)ℴℴ这是一个_. `我们从第二个打击getthis全名搜寻`. 

到目前为止,我们的小锈项目一直免费`进口`和`包括`东西会慢下来的讨论"Hello World"程序. `让我们让这个程序可读性更强`声明:

```rust
use std::f64::consts;

fn main() {
    let x = 2.0 * consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

为什么不现在我们需要这样做吗?`这是因为生锈使很多基本功能可见withoutexplicit`语句通过生锈_前奏_. 

## 数组和片

所有静态类型的语言都有_阵列_,这些价值包装着鼻子到尾巴的记忆. 数组是_索引_从零开始: 

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

在这种情况下,Rust知道_究竟_阵列有多大,如果你尝试访问`ARR [4]`这将是一个_编译错误_. 

学习一门新语言往往涉及到_忘却_来自语言的心理习惯已经知道;如果你是一个Pythonista,那么这些括号表示`名单`. 我们会得到Rust的等同物`名单`很快,但数组不是你正在寻找的机器人;他们是_大小固定_. 他们可以_易变的_ (如果我们问得好) ,但你不能添加新的元素. 

在Rust中不常使用数组,因为数组的类型包含itssize. 示例中的数组的类型是`[I32;4]`;类型`[10,20]`将会`[I32;2 ]`等等: 他们有_不同类型_.  所以他们是私生子,作为函数论证. 

什么_是_常用的是_片_. 你可以把它们看作是_意见_一个基本的值数组. 它们的行为很像一个数组,_知道他们的尺寸_不像那些危险的动物C指针. 

注意两个重要的事情-如何写一个切片的类型,并且你必须使用`&`将其传递给函数. 

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

忽略代码`总和`一阵子,看看`[I32 ]`. Read数组和切片之间的关系类似于C数组和指针之间的关系,除了两个重要的区别ℴℴRebug切片跟踪它们的大小 (如果你尝试访问这个大小之外的WRALIST) ,并且你必须明确地说你想把数组作为一个切片传递. 使用`&`操作员. 

C程序员发音`&`作为"地址",生锈程序员会借用它. 这将是学习生锈的关键词. 借用是编程中常见的模式的名称;每当你通过引用传递 (几乎总是发生在动态语言中) 或在C中传递指针时,任何东西都被原始所有者所拥有. 

## 切片切割

不能以通常的方式打印出一个数组. `{}`但你可以做一个_调试_打印与`{:? }`. 

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

数组的数组是没问题,但重要的是,containsvalues数组_只有一个类型_. 数组中的值下每个otherin内存,以便安排_高效的访问. _如果你好奇这些变量的实际类型,这是一个有用的技巧. 

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

(`{浮动}`意思是"一些浮点类型不完全指定的)

片给你不同_的观点_的_相同_数组:

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

这是一个简洁的符号,类似于Python片但是有很大区别:从未有过任何数据的一个副本. 这些片_他们的数据来自theirarrays. _他们有一个非常亲密的关系,数组,和铁锈花很多精力来确保不分解的关系. 

## 可选值

片,就像数组一样,可以_索引_. Rust在编译时知道数组的大小,但只有在运行时才知道分片的大小. 所以`S [I]`在运行时会引起超出界限的错误_恐慌_. 这实际上并不是你想要发生的事情 - 它可能是安全发射中止与遍布佛罗里达州的非常昂贵卫星的散射片之间的区别. 还有_没有例外_. 

让它沉入水中,因为它是一种冲击. 你不能在某些try-block中包装可怕的可能paniccode,并且"捕获错误" - 至少不是你每天都想使用的方式. 那么Rust如何安全?

有一种切片方法`得到`这并不恐慌. 但是它返回了什么?

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

`持续`失败 (我们忘记了基于零的索引) ,但返回了一个叫做的东西`没有`. `第一`很好,但是作为一个价值包装出现`一些`. 欢迎来到`选项`类型!它可能是_或_ `一些`要么`没有`. 

的`选项`类型有一些有用的方法:

```rust
    println!("first {} {}", first.is_some(), first.is_none());
    println!("last {} {}", last.is_some(), last.is_none());
    println!("first value {}", first.unwrap());

// first true false
// last false true
// first value 1
```

如果你_打开_ `去年`,你会得到一个恐慌. `但至少你可以叫`首先确保ℴℴ例如,如果你有一个不同的默认没有价值:

```rust
    let maybe_last = slice.get(5);
    let last = if maybe_last.is_some() {
        *maybe_last.unwrap()
    } else {
        -1
    };
```

注意`*`ℴℴ内部的精确类型`一些`是`手机等`,这是一个参考. `我们需要这回到一个废弃`价值. 

冗长的,这是一个快捷方式-`unwrap_or`将返回的值是如果`选项`是`没有一个`. `ℴℴ类型必须匹配`returnsa参考. `所以你必须组成`与`&-1`. 最后再次使用`*`获得价值`i32`. 

```rust
    let last = *slice.get(5).unwrap_or(&-1);
```

这很容易错过`&`,但编译器有你在这里. 如果它是`-1`,`rustc`说'预期&{整数},找到整数变量',然后'帮助: 尝试`&-1`". 

你可以想到`选项`作为一个可能包含一个值的框,或者什么也不是 (`没有`)  (它被称为`也许`在Haskell) . 它可能包含_任何_有价值的,就是这样_类型参数_. 在这种情况下,完整的类型是`选项<&I32>`,使用C ++风格的表示法_仿制药_.  打开这个盒子可能会引起爆炸,但不像施罗德林格的猫,我们可以事先知道它是否包含一个值. 

这是非常常见的生锈功能/方法返回这些可能的盒子,所以学习如何[使用它们](https://doc.rust-lang.org/std/option/enum.Option.html)舒适地. 

## 向量

我们将再次返回切片方法,但首先返回向量. 这些是_可重复使用的_数组和行为很像Python`表`和C++`STD: : 载体`. 锈病型`Vec`事实上,不同的是,你可以将额外的值附加到一个向量上,注意它必须声明为可变的. 

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

一个常见的初学者错误是忘记`穆特`你会得到一个有用的错误信息: 

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

那么小,那么重要的借用算子`&`是_强迫_向量进入ASLICE. 它是完全有意义的,因为向量也在监视数组的值,数组的分配也不同. _动态地_.

如果你来自一种动态的语言,那么现在是时候开始讨论了. 在系统语言中,程序存储器有两种: 堆栈和堆. 在堆栈上分配数据非常简单,但是堆栈是有限的;通常是按兆字节的顺序. 堆可以是千兆字节,但是分配相对昂贵,并且这样的内存必须是_释放_后来. 在所谓的'管理'的语言 (如java,而且所谓的脚本语言) 这些细节都隐藏在你的thatconvenient市政公用称_垃圾收集器_. 一旦系统确定数据不再引用的其他数据,它回到poolof可用内存. 

第一个C程序我写在DOS(PC)拿出整个电脑. 

Unix系统总是表现得更好,只有过程diedwith_. _为什么这个比生锈(或去)计划恐慌吗?_因为最初的问题发生的时候会发生恐慌,不是当programhas变得无可救药的困惑和吃你所有的作业. _恐慌是

解除_正如例外. _所有分配的对象是下降,生成一个回溯. 

现在_. _(妈妈的类比是她想打扫你的房间,当你在adelicate阶段新情人). _这些嵌入式系统需要应对的事情_回到向量:当修改或创建一个向量,它从堆中分配,成为了

老板_的内存. _切片_内存从向量中. _当向量或死亡_,它让记忆. _迭代器

## 我们到目前为止没有提及的关键部分生锈难题ℴℴ迭代器. 

for循环在使用迭代器范围是(`其实是类似于thePython 3`范围`功能). `迭代器很容易定义非正式. 

这是一个"对象"`下一个`方法返回一个`选项`. 只要这个价值不是`没有`,我们一直在打电话`下一个`: 

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

而这正是什么`对于var iter {}`确实. 

这似乎是定义for循环的低效方式,但是`rustc`在发布模式中会进行疯狂的assoptimizations,并且它会和a一样快`而`循环. 

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

实际上,迭代数组或切片这种方式使用起来效率更高`我在0..slice.len () {}`因为Rust不必痴迷于检查每个索引操作. 

我们之前有一个总结一系列整数的例子. 它涉及一个`mut`变量和循环. 这是_惯用的_,亲做平分的方式: 

```rust
// sum1.rs
fn main() {
    let sum: i32  = (0..5).sum();
    println!("sum was {}", sum);

    let sum: i64 = [10, 20, 30].iter().sum();
    println!("sum was {}", sum);
}
```

请注意,这是其中一个需要明确说明的情况. _类型_对于变量,因为否则生锈没有足够的信息. 这里我们用两个不同的整数做和,没有问题.  (如果用尽所有的名字来创建一个新的同名变量也是没有问题的. ) 

在这样的背景下,更多的[切片法](https://doc.rust-lang.org/std/primitive.slice.html)将更有意义.  (另一个文档提示;在每个文档页的右边有一个'[-]单击该按钮以折叠方法列表. 然后你可以扩展任何看起来很有趣的细节. 任何看起来怪异的东西,现在就忽略它. 

这个`窗户`方法提供了一个迭代器的重叠值窗口. 

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

或`块`: 

```rust
    for s in slice.chunks(2) {
        println!("chunks {:?}", s);
    }
// chunks [1, 2]
// chunks [3, 4]
// chunks [5]
```

## 更多关于向量ⅆ

有一个有用的小宏`VEC!`用于初始化向量. 注意你可以_去除_向量结束时的值`流行音乐`和_延伸_一个使用任何兼容迭代器的向量. 

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

可以将值插入到任意位置的向量中. `插入`并移除`去除`. 

这不是有效的推动和出现由于价值观必须搬到腾出空间,所以对bigvectors小心这些操作. _能力_. `如果你`一个向量,它的大小为零,但它仍然保留着古老的能力. `所以给它注入`等只有requiresreallocation规模变大时比能力. 

向量可以排序,然后可以删除重复的ℴℴ这些操作in-placeon向量. `(如果你想先复制,使用`). 

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

字符串在生锈比其他语言更涉及;`的`类型,如`Vec`动态分配,resizeable. `(这就像c++`但不像Java和Python的不可改变的字符串). _但程序可能包含很多_(比如"你好")和一个系统的语言应该能够storethese静态执行本身. _在嵌入式微指令,这可能意味着puttingthem廉价罗,而不是昂贵的RAM(对于低功耗设备,RAM的功耗也贵了. )_一个

所以"hello"不是的类型`字符串`. `它的类型是`(发音"串片"). `就像之间的区别`和`的std :: string`在C ++中,除外`&STR`更聪明得多. 事实上,`&STR`和`串`像对待一样,彼此之间有着非常类似的关系`&[T]`至`VEC <T>`. 

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

再次,借用运营商可以强制`串`成`&STR`, 就像`VEC <T>`可能会被强迫进入`&[T]`. 

在引擎盖下,`串`基本上是`VEC <U8>`和`&STR`是`[U8]`但是那些字节_必须_表示有效的UTF-8文本. 

就像一个向量,你可以`推`一个人物和`流行音乐`一结束`弦`: 

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

可以将许多类型转换为字符串. `托弦` (如果可以用"{}"显示它们,那么它们可以被转换) . `格式!`宏是使用相同的格式字符串来构建更复杂的字符串的一种非常有用的方法. `打印!`.

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

注意`&`在前面`V.ToStruts () `-在字符串条上定义运算符,而不是`弦`因此,它需要一点说服力来匹配. 

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

但是,你不能索引字符串!这是因为它们使用一个真正的编码UTF-8,其中"字符"可能是字节数. 

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

现在,让它沉进去ℴℴ有25个字节,但是只有18个字符!但是,如果你使用类似的方法`找到`,你会得到一个有效的索引(如果发现)和thenany切片会没事的. 

(生锈`字符`类型是一个4字节的Unicode代码点. _字符串是_arraysof识字课!)

字符串分割可能爆炸像向量索引,因为它使用字节偏移量. 

```rust
    let s = "¡";
    println!("{}", &s[0..1]); <-- bad, first byte of a multibyte character
```

在本例中,字符串包含两个字节,所以试图拿出第一个字节是一个Unicode错误. `因此要小心只使用有效的抵消来自字符串部分字符串的方法. `方法返回一个_迭代器_,然后选择如何处理它. 

`一个常见needis创建一个向量分裂的子字符串. `很一般,所以需要一些线索_什么_这是收集ℴℴhencethe显式类型. 

```rust
    let text = "the red fox and the lazy dog";
    let words: Vec<&str> = text.split_whitespace().collect();
    // ["the", "red", "fox", "and", "the", "lazy", "dog"]
```

你也可以这样说,通过迭代器进入`扩展`方法:

```rust
    let mut words = Vec::new();
    words.extend(text.split_whitespace());
```

在大多数语言中,我们必须让这些_单独分配字符串_每个片,而这里的向量是借款从原始字符串. 

我们分配的空间保持片. `收集`needsa线索(我们可能想要一个向量的识字课,说):

```rust
    let stripped: String = text.chars()
        .filter(|ch| ! ch.is_whitespace()).collect();
    // theredfoxandthelazydog
```

的`过滤器`方法接受一个_关闭_,这是锈 - 说forlambdas或匿名功能. 这里的参数类型从上下文中是清楚的,所以显式规则是放松的. 

是的,你可以这样做,作为一个字符的显式循环,将返回的片段推送到一个可变的向量中,但是这个更短,读得很好 (_什么时候_你当然习惯了) ,同样快. 它不是_罪_然而,使用循环,我鼓励你也写这个版本. 

## 插曲: 获取命令行参数

到目前为止,我们的节目都生活在对外界的无知之中;现在是时候给他们提供数据. 

`的std :: ENV :: ARGS`是你如何访问命令行参数;它返回一个迭代器作为字符串的参数,包括程序名. 

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

返回a会更好吗?`Vec`?这很容易使用`搜集`使用迭代器来创建该向量`跳跃`方法移过节目名称. 

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

`第n (1) `为您提供迭代器的第二个值,以及`期望`就像一个`摅`带有可读的信息. 

将字符串转换为数字很简单,但您需要指定值的类型 - 还有什么可以`解析`知道?

这个程序可能会恐慌,这对于笨拙的测试程序来说很好. 但是,不要因为这种便利的习惯而难以忍受. 

## 匹配

代码在`斯特林3.RS`我们提取俄语问候语的方式并不是通常所写的. 进入_比赛_: 

```rust
    match multilingual.find('п') {
        Some(idx) => {
            let hi = &multilingual[idx..];
            println!("Russian hi {}", hi);
        },
        None => println!("couldn't find the greeting, Товарищ")
    };
```

`比赛`包括几个_模式_用一个匹配值跟随胖箭头,用逗号分隔. 它方便地从`选择权`把它束缚起来`idx`.  你_必须_指定所有的可能性,所以我们必须处理`没有`.

一旦你习惯了 (我的意思是,把它打满几遍) ,感觉比自然的更自然. `伊斯梅尔`检查需要一个额外的存储`选择权`.

但是如果你对这里的失败不感兴趣,那么`如果让`是你的朋友: 

```rust
    if let Some(idx) = multilingual.find('п') {
        println!("Russian hi {}", &multilingual[idx..]);
    }
```

这是方便的,如果你想做一场比赛,是_只有_对一个可能的结果感兴趣. 

`匹配`也可以像一个C`开关`声明中,像其他铁锈constructscan返回值:

```rust
    let text = match n {
        0 => "zero",
        1 => "one",
        2 => "two",
        _ => "many",
    };
```

的`_`就像C`默认的`ℴℴ这是一个备用情况. `如果你不提供一个`会认为这是一个错误. 

(在c++中最好的期望是一个警告,获利很多关于各自的语言). `匹配`语句也可以匹配范围. _请注意,这些范围_点和包容的范围,所以,第一个条件将匹配3. 

```rust
    let text = match n {
        0...3 => "small",
        4...6 => "medium",
        _ => "large",
     };
```

## 读取文件

下一步是向世界展示我们的程序_阅读文件_. 

回想一下,`预计`就像`打开`但给了一个自定义的错误消息. 

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

在这里我们aregoing扔掉一些错误:`开放`可以失败,因为该文件不存在或者我们不允许读它,然后呢`read_to_string`可能会失败,因为该文件不包含有效的UTF-8.  (这是足够的,你可以使用`read_to_end`并将其内容放入bytesinstead的矢量中. ) 对于不太大的文件,以一g reading的方式读取它们是有用和直接的. 

如果你知道其他语言的文件处理的任何事情,你可能会怀疑文件是什么_关闭_. 如果我们正在写这个文件,那么不关闭它可能导致数据丢失. 但是当函数结束时,这里的文件被关闭,而`文件`变量是_下降_. 

这种"抛出错误"的做法是习惯太过分了. 你不想将这些代码放入函数中,因为它知道它可以很容易地使整个程序崩溃. 所以现在我们必须谈论到底是什么`文件::打开`returns.If`选项`是一个值,可能包含或不包含任何内容`结果`是一个可能包含某些内容或错误的值. 他们都明白`摅` (和它的表弟`期望`) 但它们完全不同. `结果`由...定义_二_键入参数,为`好`价值和`呃`价值. `结果`盒子有两个隔间,一个标签`好啊`而另一个`厄尔`.

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

 (实际的"错误"类型是任意的,很多人使用字符串,直到它们对生锈错误类型感到满意) . 这是一种方便的方法. _任何一个_返回一个值_或_另一个. 

该版本的文件读取功能不会崩溃. 它返回一个`结果`这就是_呼叫者_谁必须决定如何处理这个错误. 

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

第一次比赛安全地提取价值`好啊`这就成了比赛的价值所在. 如果是`厄尔`它返回错误,重新包装为`厄尔`.

第二个匹配返回字符串,封装为`好啊`,否则 (再次) 错误. 中的实际价值`好啊`是不重要的,所以我们点燃`γ`. 

这不是那么漂亮;_当大多数的错误处理函数,然后是愉快路径丢失. _往往会有这样的问题,有很多ofexplicit早期的回报,或者只是

幸运的是,有一个捷径. 

的`std::io`模块定义了一个类型别名`io::结果< T >`这是画出来一样吗`结果误差< T,io::>`和容易类型. 

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = File::open(&filename)?;
    let mut text = String::new();
    file.read_to_string(&mut text)?;
    Ok(text)
}
```

那`吗?`运营商也几乎完全匹配`文件:打开`,如果结果是一个错误,那么它将立即返回错误. `否则,它将返回`结果. 

最后,我们仍然需要结束字符串作为结果. `吗?`是一个很酷的东西就成了稳定. `你仍能看到宏`用于旧代码:

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = try!(File::open(&filename));
    let mut text = String::new();
    try!(file.read_to_string(&mut text));
    Ok(text)
}
```

总之,可以写绝对安全生锈,不丑,withoutneeding例外. 
