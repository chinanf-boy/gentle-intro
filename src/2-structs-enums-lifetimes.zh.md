
# 结构{structs},枚举{enums}和匹配{match}

## Rust喜欢 move 它, move 它

我想稍微回退一下,给你看一些令人惊讶的东西: 

```rust
// move1.rs
fn main() {
    let s1 = "hello dolly".to_string();
    let s2 = s1;
    println!("s1 {}", s1);
}
```

我们得到以下错误: 

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

Rust与其他语言有不同的行为. 在变量总是`引用{references}` (如Java或Python) 的语言中,`s2`成为对引用的字符串对象的又一个引用`s1`. 在C ++中,`s1`是一种`价值{value}`,它是 _复制_ 至`s2`. 但Rust会改变价值. 它没有看到`字符串{strings}`是可复制的 ("不实现复制trait") . 

我们不会看到像数字这样的"原始"类型,因为它们只是数值;他们被允许复制,因为他们便宜复制. 但`String`已经分配了包含"Hello dolly"的内存,并且复制将涉及分配更多内存并复制`字符{char}`. Rust不会默默地做到这一点. 

考虑一个`String`包含"Moby-Dick"的全文. 它不是一个很大的结构,只有文本的内存地址,它的大小以及分配块的大小. 复制这将是昂贵的,因为该内存分配在堆上,副本将需要自己分配的块. 

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

第二个值是一个字符串切片 (`&str`) ,它与字符串指向相同的内存,大小 - 只是人的名字. 便宜复制!

第三个值是一个`f64`- 只有8个字节. 它不涉及任何其他内存,所以它的复制一样便宜移动. 

`复制{Copy}`值只能通过它们在内存中的表示来定义,而当Rust拷贝时,它只是在其他地方复制这些字节. 类似地, 一个没有`复制{Copy}`的`价值{value}`也是 _只能移动{move}_. 与C ++不同的是,Rust在复制和移动方面没有聪明之处. 

> 对 具有引用的值 隐形`移动{move}`值, 在 rust 是错误的

用函数调用重写将显示完全相同的错误: 

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

在这里,你有一个选择. 您可以传递对该字符串的`引用{&}`,或者使用它`clone`方法明确拷贝. 一般来说,第一种是更好的方法. 

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

错误消失. 但你很少看到一个简朴`String`像这样的引用,因为传递一个字符串文字是非常丑陋的 _和_ 涉及创建一个临时字符串. 

```rust
    dump(&"hello world".to_string());
```

因此,声明该函数的最佳方式是: 

```rust
fn dump(s: &str) {
    println!("{}", s);
}
```

然后两个 `dump(&s1)` 和 `dump("hello world")` 好好工作.  (这里`Deref`强制踢进来,Rust会转换`&String`至`&str`为你. ) 总而言之,分配 `非复制值{non-Copy}`将值从一个位置`移动{move}`到另一个位置. 

否则,`锈{rust}`将被迫 _隐式_ 做一个`副本{copy}`,并打破rust本身`承诺{promise}`-明确分配. 

## 变量的范围

所以,经验法则是更愿意保留对原始数据的`引用{&}` - 以`"借用{borrow}"`它. 

但一个引用必须 _不_ 超过`老板{owner}`!

首先,Rust是一个 _块范围的{block-scoped}_ 语言. 变量仅在其块的持续时间内存在: 

```rust
{
    let a = 10;
    let b = "hello";
    {
        let c = "hello".to_string();
        // a,b and c are visible 有
    }
    // the string c is dropped 没有了
    // a,b are visible 有
    for i in 0..a {
        let b = &b[1..];
        // original b is no longer visible - it is shadowed.
        // 原来的b不再可见 - 它是阴影。
    }
    // the slice b is dropped
    // i is _not_ visible!
}
```

循环变量 (如`i`) 有点不同,它们只在循环块中可见. 创建一个使用相同名称的新变量并不是一个错误 (`'shadowing'`) ,但它可能会造成混淆. 

当一个变量'超出范围',那么它是 _扔掉了{dropped}_. 任何使用的内存都会被回收,而其他内存则被回收 _资源{resources}_ 由该变量所拥有的信息将返回给系统 - 例如,丢弃一个`文件{File}`关闭它. 这是一件好事. 未使用的资源在不需要时立即回收. 

 (另一个特定于Rust的问题是变量可能看起来在范围内,但其值已经`移动{move}`. ) 
 
 这里有一个引用, `rs1`是有价值的, 而`tmp`它只在其封锁期间存在:

```rust
01 // ref1.rs
02 fn main() {
03    let s1 = "hello dolly".to_string();
04    let mut rs1 = &s1;
05    {
06        let tmp = "hello world".to_string();
07        rs1 = &tmp;
08    }
09    println!("ref {}", rs1);
10 }
```

 我们`借用{borrow}`的价值 `s1`然后借用的价值`tmp`. 但`tmp`在该块之外不存在价值!

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

哪里`tmp`?走了,死了,回到了天空中的大堆: _扔掉了{dropeed}_. 铁锈在这里拯救你从 **C** 的可怕的'悬挂指针'问题 - 一个指向陈旧数据的引用. 

> 在 区块中, `rs1`-指向-> `&tmp`, 但在区块结束后, tmp 整个都被 _扔掉了{dropped}_ , 这个时候 `rs1` 也就指向一个空指针

## 元组

从函数返回多个值有时非常有用. 元组是一个方便的解决方案: 

```rust
// tuple1.rs

fn add_mul(x: f64, y: f64) -> (f64,f64) {
    (x + y, x * y)
}

fn main() {
    let t = add_mul(2.0,10.0);

    // can debug print
    println!("t {:?}", t);

    // can 'index' the values
    println!("add {} mul {}", t.0,t.1);

    // can _extract_ values
    let (add,mul) = t;
    println!("add {} mul {}", add,mul);
}
// t (12, 20)
// add 12 mul 20
// add 12 mul 20
```

元组可能包含 _不同_ 类型,这是与数组的主要区别. 

```rust
let tuple = ("hello", 5, 'c');

assert_eq!(tuple.0, "hello");
assert_eq!(tuple.1, 5);
assert_eq!(tuple.2, 'c');
```

他们出现在一些`迭代器{Iterator}`方法. `枚举{enumerate}`就像同名的 ython生成器 一样: 

```rust
    for t in ["zero","one","two"].iter().enumerate() {
        print!(" {} {};",t.0,t.1);
    }
    //  0 zero; 1 one; 2 two;
```

`zip`将两个迭代器,组合成一个 _包含来自两者的值的元组_ 的迭代器: 

```rust
    let names = ["ten","hundred","thousand"];
    let nums = [10,100,1000];
    for p in names.iter().zip(nums.iter()) {
        print!(" {} {};", p.0,p.1);
    }
    //  ten 10; hundred 100; thousand 1000;
```

## 结构{Structs}

元组很方便,但是在说`t.1`并且追踪每个部分的含义, 不够直截了当

rust _结构_ 包含命名 _领域{fields}_ : 

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

虽然不应该假定任何特定的内存布局,但是结构体的值将在内存中相邻放置,因为编译器会组织内存以提高效率,而不是大小,并且可能会存在填充. 

初始化这个结构有点笨拙,所以我们想要移动构造一个`Person`融入其自身的功能. 这个功能可以做成一个 _关联功能_ 的`Person`把它放进去`impl`块: 

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

这个名字没有什么魔力或保留`new`这里. 请注意,它使用C ++进行访问 - 如使用双冒号的符号`::`. 

这是一个`Person` _方法_,这需要一个_引用自我{reference self}_论据: 

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

该`self`明确使用并作为`引用`传递.  (你可以想到`&self`作为简称`slef: &Person`. ) 

关键字`Self`指的是结构类型 - 你可以在脑海中替代`Person`对于`Self`这里: 

```rust
    fn copy(&self) -> Self {
        Self::new(&self.first_name,&self.last_name)
    }
```

方法可以允许修改数据, 使用 _可变的自我{mutable self}_ 论据: 

```rust
    fn set_first_name(&mut self, name: &str) {
        self.first_name = name.to_string();
    }
```

数据会 _移动{move}_ 当使用简单的`self`参数时,

```rust
    fn to_tuple(self) -> (String,String) {
        (self.first_name, self.last_name)
    }
```

 (尝试使用`&self`- 结构不会在没有战斗的情况下放开数据!) 

请注意,之后`v.to_tuple()`被调用,然后`v`已经移动并且不再可用. 

总结: 

-   没有`self`参数: 您可以将函数与结构关联,如`new`"构造". 
-   `&self`参数: 可以使用结构体的值,但不能改变它们
-   `&mut self`参数: 可以修改这些值
-   `self`参数: 将消耗将移动的值. 

如果您尝试执行一个调试转储`Person`,你会得到一个信息错误: 

    error[E0277]: the trait bound `Person: std::fmt::Debug` is not satisfied
      --> struct2.rs:23:21
       |
    23 |     println!("{:?}", p);
       |                     ^ the trait `std::fmt::Debug` is not implemented for `Person`
       |
       = note: `Person` cannot be formatted using `:?`; if it is defined in your crate,
        add `#[derive(Debug)]` or manually implement it
       = note: required by `std::fmt::Debug::fmt`

编译器提供建议,所以我们放了`#[derive(Debug)]`在`Person`前面,现在有明智的输出: 

    Person { first_name: "John", last_name: "Smith" }

该 _指示{directive}_ 使编译器生成一个`Debug`实施{implementation},这是非常有益的. 对于你的结构来说这是一个很好的习惯,所以它们可以打印出来 (或者用字符串写成`format!`) .  (这样做 _默认_ 会非常`不生锈{un-Rustlike}`. ) 这是最后的小程序: 


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

## 寿命{Lifetimes}开始咬人

通常结构体包含值,但通常它们还需要包含`引用{&}`. 假设我们想在一个结构中放置一个`字符串切片{&str}`,而不是一个字符串值. 

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

    error[E0106]: missing lifetime specifier
     --> life1.rs:5:8
      |
    5 |     s: &str
      |        ^ expected lifetime parameter


为了理解`投诉{complaint}`,你必须从Rust的角度看问题. 

如果不知道它的生命周期,它将不允许存储引用. 所有`引用{&}`都是从某个价值中`借用{borrowed}`的,而且所有的`价值{value}`都是有`生命{lifetimes}`的. 引用的生命周期不能长于该值的生命周期. Rust不能允许这种引用可能突然失效的情况. 

现在,字符串切片, 借用字符串文字 _像"hello"或来自_ `String`值. 字符串文字在整个程序期间存在,称为"静态{static}"寿命. 

所以这是有效的 - 我们向Rust保证字符串切片总是指向这样的`静态{static}`字符串: 

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

这不是最 _漂亮_ 符号,但有时丑是精确的必要代价. 

这也可以用来指定从函数返回的字符串片段: 

```rust
fn how(i: u32) -> &'static str {
    match i {
    0 => "none",
    1 => "one",
    _ => "many"
    }
}
```

这适用于静态字符串的特殊情况,但这是非常严格的. 

但是我们可以指定`引用{&}`的生命周期 _至少与之一样长_ 就像结构本身一样. 

```rust
// life3.rs

#[derive(Debug)]
struct A <'a> {
    s: &'a str
}

fn main() {
    let s = "I'm a little string".to_string();
    let a = A { s: &s };

    println!("{:?}", a);
}
```

`生命周期{Lifetimes}`通常被称为'a','b'等,但您可以在此称之为'我{me}'. 

在这之后,我们的`a`结构和`s`字符串受到严格合同的约束: `a`借用`s`,并且不能超越它. 

有了这个结构体定义,我们想写一个函数返回一个`一个`值: 

```rust
fn makes_a() -> A {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

但`a` 需要 lifetime - "预期寿命参数{expected lifetime parameter}": 

      = help: this function's return type contains a borrowed value,
       but there is no value for it to be borrowed from
      = help: consider giving it a 'static lifetime

`rustc`提供建议,所以我们遵循它: 

```rust
fn makes_a() -> A<'static> {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

而现在的错误是

    8 |      A { s: &string }
      |              ^^^^^^ does not live long enough
    9 | }
      | - borrowed value only lives until here

这是无法安全工作的,因为`string`将在函数结束时被删除,并且不会引用`string`可以超过它. 

您可以将生命周期参数视为值类型的一部分. 

有时候,结构中包含一个值似乎是个好主意 _和_ 从该价值中借鉴的引用. 这基本上是不可能的,因为结构必须是 _活动_ ,任何移动都将使引用无效. 没有必要这样做 - 例如,如果你的结构有一个字符串字段-string,并且需要提供切片-str,那么它可以保留索引并且有一个方法来生成实际的切片. 

## 特点{Traits}

请注意Rust不会拼写`struct` _class_. 关键字`类`在其他语言中是如此超载,意味着它有效地关闭了原来的想法. 

让我们这样说吧: Rust结构不能 _继承_ 来自其他结构;他们都是独特的类型. 没有 _sub-typing{子}_ . 他们是愚蠢的数据. 

又怎样 _做_ 一个建立类型之间的关系? 这正是 _Traits_ 进来的时候. 

`rustc`经常谈到`实现{implementing} x 特点{trait}`所以现在是时候正确地讨论 _Traits_ 了. 

这里有一个定义 _Traits_ 的例子 _implementing_ 它适用于特定类型. 

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

它太酷了;我们有 _增加了一种新方法_ 二者皆是`i32`和`f64`!

熟悉Rust就会学习标准库的基本trait (他们倾向于打包) . 

`调试`是非常普遍的. 我们给`Person`一个方便的默认实现`#\[derive(Debug)]`,但说我们想要一个`Person`以全名显示:

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

 `write!`是一个非常有用的宏 - 在这里`f`是实现`Write`.  (这也适用于`File`- 甚至是一个`String`. ) 
 
 `显示{Display}`控制如何使用"{}"打印值,并执行就像`调试`. 作为一个有用的副作用,`ToString`自动实施-任何实现`Display`. `所以如果我们执行`Display`对于`Person`, 然后`p.to_string()`也适用. 

`Clone`定义了该方法`clone`,并且可以简单地用"#"[deriv(Clone)]"如果所有的领域都实施`Clone`. 

## 示例: 遍历浮点范围的迭代器

我们已经遇到范围之前 (`0..n`) 但它们不适用于浮点值.  (您可以 _强制性_ 这一点,但最终你会得到一个无趣的1.0步. ) 

回想一下迭代器的非正式定义;它是一个带有结构体`next`方法可能会返回`Some`事情或`None`. `在这个过程中,迭代器本身被修改,它保持迭代的状态 (如next索引等等) . 迭代的数据通常不会改变, (但请参阅`VEC::drain`对于修改其数据的有趣的迭代器) . 

这里是正式的定义: [迭代器trait](https://doc.rust-lang.org/std/iter/trait.Iterator.html). 

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    ...
}
```

我们在这里见面[关联类型{associated type}](https://doc.rust-lang.org/stable/book/associated-types.html)的`迭代器`trait. 这个trait必须适用于任何类型,所以你必须以某种方式指定返回类型. 方法`next`可以在不使用特定类型的情况下编写 - 而是指那个类型参数`Item`来自`Self`. 

迭代器trait`f64`写入`Iterator<Item=f64>`,它可以理解为"Iterator的关联类型Item设置为f64". 

该`...`指的是 _提供的方法_ 的`迭代器`. 你只需要定义`Item`和`next`,并为您定义提供的方法. 

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

这是因为0.1不能精确表示为float,所以需要一些格式化帮助. 更换`调用println!`有了这个

```rust
println!("{:.1} ", x);
```

我们得到更清洁的产出 (这个[格式](https://doc.rust-lang.org/std/fmt/index.html)意思是'小数点后一位小数'. ) 所有默认的迭代器方法都可用,所以我们可以将这些值收集到一个`向量{Vec}`中,映射它们等等. 

```rust
    let v: Vec<f64> = range(0.0, 1.0, 0.1).map(|x| x.sin()).collect();
```

## 通用函数

我们需要一个函数来抛出实现的任何值`调试`. `以下是对通用函数的第一次尝试,我们可以在其中传递引用 _任何_ 价值类型. `T`是一个类型参数,需要在函数名称后面声明: 

```rust
fn dump<T> (value: &T) {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
```

但是,Rust显然对这种通用类型一无所知`T`: 

    error[E0277]: the trait bound `T: std::fmt::Debug` is not satisfied
    ...
       = help: the trait `std::fmt::Debug` is not implemented for `T`
       = help: consider adding a `where T: std::fmt::Debug` bound

为了这个工作,Rust需要被告知`T`确实实施`Debug`!

```rust
fn dump<T> (value: &T)
where T: std::fmt::Debug {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
// value is 42
```

Rust通用功能需要 _Traits bounds_ 类型 - 我们在这里说"T是实现Debug的任何类型". `rustc`是非常有用的,并且确切地说明需要提供什么界限. 

现在Rust知道这个 _trait_ `T`,它可以给你敏感的编译器消息: 

```rust
struct Foo {
    name: String
}

let foo = Foo{name: "hello".to_string()};

dump(&foo)
```

而错误是"_Traits_ `std::fmt::Debug`没有实施`Foo`". 

函数在动态语言中已经是泛型的,因为值会带有它们的实际类型,并且类型检查在运行时发生 - 或者惨败. 对于较大的程序,我们确实想在编译时想知道问题!这些语言的程序员不应平静地坐在编译器的错误之中,而必须处理程序运行时才会出现的问题. 墨菲定律意味着这些问题往往会发生在最不方便/灾难性的时刻. 

平方数的操作是通用的: `x * x`将适用于整数,浮点数和一般用于任何关于乘法运算符的知识`*`. 但是类型边界是什么?

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

第一个问题是Rust不知道`T`可以乘以: 

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

遵循编译器的建议,让我们使用限制该类型参数[那个_Traits_](https://doc.rust-lang.org/std/ops/trait.Mul.html),用于实现乘法运算符`*`: 

```rust
fn sqr<T> (x: T) -> T
where T: std::ops::Mul {
    x * x
}
```

哪个仍然不起作用: 

    rror[E0308]: mismatched types
     --> gen2.rs:6:5
      |
    6 |     x * x
      |     ^^^ expected type parameter, found associated type
      |
      = note: expected type `T`
      = note:    found type `<T as std::ops::Mul>::Output`

什么`rustc`是说这种类型的`x * x`是关联的类型`T::Output`,不`T`. 实际上没有理由的类型`x * x`与类型相同`x`,例如,两个向量的点积是一个标量. 

```rust
fn sqr<T> (x: T) -> T::Output
where T: std::ops::Mul {
    x * x
}
```

现在的错误是: 

    error[E0382]: use of moved value: `x`
     --> gen2.rs:6:7
      |
    6 |     x * x
      |     - ^ value used here after move
      |     |
      |     value moved here
      |
      = note: move occurs because `x` has type `T`, which does not implement the `Copy` trait

所以,我们需要进一步限制类型!

```rust
fn sqr<T> (x: T) -> T::Output
where T: std::ops::Mul + Copy {
    x * x
}
```

那 (终于) 起作用了. 冷静地倾听编译器经常会让你更接近魔术点,当......事情干净地编译时. 

它 _是_ 在C ++中更简单一点: 

```cpp
template <typename T>
T sqr(x: T) {
    return x * x;
}
```

但 (说实话) C ++在这里采用牛仔策略. C ++`模板{template}`错误很不好,因为所有的编译器都知道 (最终) 是某些操作符或方法没有被定义. C ++委员会知道这是一个问题,所以他们正在努力[概念](https://en.wikipedia.org/wiki/Concepts_(C%2B%2B)),这几乎与Rust中的`trait约束类型`参数非常相似. 

Rust通用函数一开始可能看起来有点压倒性,但是显式意味着只要看看定义,就能确切地知道可以安全地提供哪种值. 

这些函数被调用 _单态{monomorphic}_ ,与...相反 _多态{polymorphic}_. 函数的主体是为每个唯一类型分别编译的. 通过多态函数,相同的机器代码可以动态地与每种匹配类型一起工作 _调度{dispatching}_ 正确的方法. 

`Monomorphic`生成更快的代码,专用于特定类型,并且通常可以 _内联{inlined}_. 所以何时`sqr(x)`被看到,它被有效地取代`x * x`. 缺点是大的泛型函数会产生大量的代码,对于每一种可能导致的类型 _代码膨胀_. 与往常一样,有折衷; 有经验的人学会为工作做出正确的选择. 

## 简单的枚举

`枚举{enums}`类型具有一些确定的值. 例如,一个方向只有四个可能的值. 

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

他们可以在结构上定义方法,就像结构一样. 该`match`表达是处理`enum`值的基本方式. 

```rust
impl Direction {
    fn as_str(&self) -> &'static str {
        match *self { // *self has type Direction
            Direction::Up => "Up",
            Direction::Down => "Down",
            Direction::Left => "Left",
            Direction::Right => "Right"
        }
    }
}
```

标点符号很重要. 注意`*`之前`self`. 很容易忘记,因为Rust经常会假设它 (我们说过`self.first_name`,而不是`(*self).first_name`) . 但是,`匹配{matching}`是一个更精确的业务. 将它排除在外可能会产生一大堆消息,这些消息可归结为这种类型的不匹配: 

       = note: expected type `&Direction`
       = note:    found type `Direction`

这是因为`self`有类型`&Direction`,所以我们必须投入`*`至 _deference_ 方式. 

像结构一样,枚举可以实现 traits ,也可以实现我们的朋友`#[derive(Debug)]`可以添加到`Direction`: 

```rust
        println!("start {:?}",start);
        // start Left
```

所以`as_str`方法并不是真的必要,因为我们总是可以从中得到名字`Debug`.  (但`as_str`不 _不分配{allocate}_ ,这可能很重要.) 

你不应该在这里假设任何特定的顺序 - 没有的默许整数"ordinal"值. 

这里有一个方法来定义每个`方向`值的'后继者'. 非常方便通配符使用 _暂时{wildcard}_ 将枚举名称放入方法上下文中: 

```rust
    fn next(&self) -> Direction {
        use Direction::*;
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

所以这将在这个特定的,任意的顺序中通过各个方向循环不休. 它 (事实上) 非常简单状态机. 

这些枚举值无法比较: 

    assert_eq!(start, Direction::Left);

    error[E0369]: binary operation `==` cannot be applied to type `Direction`
      --> enum1.rs:42:5
       |
    42 |     assert_eq!(start, Direction::Left);
       |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
       |
    note: an implementation of `std::cmp::PartialEq` might be missing for `Direction`
      --> enum1.rs:42:5

解决办法就是说`\#\[派生(Debug,PartialEq)]`在前面`enum Direction`. `这是一个重要的观点 -  Rust用户定义的类型开始新鲜和朴素. 

你通过实施共同的特点给他们合理的默认行为. 这也适用于结构 - 如果你要求Rust得出 _derive_ `PartialEq`对于一个结构体来说,它会做出明智的事情,假设所有的领域都实现它并建立一个比较. 如果不是这样,或者你想重新定义平等,那么你可以自由明确定义`PartialEq`. 

Rust也有'C风格的枚举': 

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

它们用一个整数值进行初始化,并可以通过类型转换将其转换为该整数. 

你只需要给名字一个值,然后每次增加一个值: 

```rust
enum Difficulty {
    Easy = 1,
    Medium,  // is 2
    Hard   // is 3
}
```

顺便说一下,'名字'太模糊了,就像一直在说'thingy'. 这里的合适名词是 _变种_ -`速度`有变种`慢`,`中`和`快速`. 

这些枚举 _做_ 有一个自然的顺序,但你必须很好地问. 放置后`#[derive(PartialEq,PartialOrd)]`在前面`枚举速度`,那确实如此`Speed::Fast>Speed::Slow`和`Speed::Medium!=Speed::Slow`. 

## 以其全部荣耀枚举

完全形式的rust 类似于类固醇上的 C 联盟,就像法拉利相比,菲亚特Uno. 考虑以类型安全的方式存储不同值的问题. 

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

同样,这个枚举只能包含 _一个_ 这些价值观;其大小将是最大变体的大小. 

到目前为止,并不是真正的超级跑车,虽然枚举知道如何打印出来是很酷的. 但他们也知道如何 _哪一种_ 它们包含的价值,和 _那_ 是超级`match`: 

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

 (那就是这样`Option`和`Result`是 - 枚举.) 
 
 我们喜欢这个`eat_and_dump`函数,但我们希望将该值作为引用传递,因为当前发生了`移动{move}`并且该值被'吃掉'了:

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

借用引用资料你无法做到. Rust不会让你 _提取_ 包含在原始值中的字符串. 它没有抱怨`Number`因为它很高兴复制`f64`,但是`String`不执行`Copy`. 

我之前提到过

`match`挑剔精确 _类型;_ 在这里,我们按照提示进行操作;

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

现在我们只是借用对包含字符串的引用. 在我们继续前进之前,充满了成功的Rust编辑的欣快感,让我们暂停一下. `rustc`在生成足够上下文以供人类使用的错误方面非常优秀 _修正_ 错误不一定 _理解_ 错误. 

这个问题是匹配的正确性,以及借用检查者决定阻止任何违反规则的企图的结合. 其中一条规则是你不能抽出属于某种拥有类型的价值. C ++的一些知识在这里是一个障碍,因为C ++将复制出问题的方式,无论是复制甚至是 _说得通_ . 如果你尝试从一个矢量中抽出一个字符串,你会得到完全相同的错误`*v.get(0).unwrap()`(`*`因为索引返回引用. )它不会让你这样做. (有时`clone`这不是一个坏的解决方案.)

(顺便一提,正是出于这个原因,`v[0]`,它不适用于像字符串这样的非可复制值. `你必须或者借用`&v[0]`或clone-`v[0].clone()`)

至于`match`,你可以看到`Str(s)=>`作为简称`Str(s: String)=>`. 局部变量(通常称为 _捆绑_ ) 被建造. 通常推断的类型很酷,当你吃掉一个值并提取其内容时. 但我们真正需要的是`s: &String`,和`ref`是一个暗示,确保这一点: 我们只是想借用该字符串. 

在这里,我们确实想提取该字符串,并且不关心之后的枚举值. `_`像往常一样会匹配任何东西

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

命名重要 - 这就是所谓的`to_str`,不是`as_str`. 你可以编写一个方法,只是将该字符串作为一个借用`Option<&String>` (引用将需要与枚举值相同的生命周期. ) 但是你不会调用它to_str`.

你可以写`to_str`像这样 - 它完全等价: 

```rust
    fn to_str(self) -> Option<String> {
        if let Value::Str(s) = self {
            Some(s)
        } else {
            None
        }
    }
```

## 更多关于匹配

回想一下,元组的值可以用'()'来提取: 


```rust
    let t = (10,"hello".to_string());
    ...
    let (n,s) = t;
    // t has been moved. It is No More
    // n is i32, s is String
```
这是一个特例

_解构{destructuring}_;_我们有一些数据,并希望将其分开 (像这里) 或只是借用它的值. 无论哪种方式,我们都可以得到结构的各个部分. 

语法与在中使用的相似`match`. 这里我们明确地借用了这些值. 

```rust
    let (ref n,ref s) = t;
    // n and s are borrowed from t. It still lives!
    // n is &i32, s is &String
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
    // p still lives, since x and y can and will be copied
    // both x and y are f32
```

时间重温`match`有一些新的模式. 前两种模式完全一样`let`解构 - 它只匹配第一个元素为零的元组,但是 _任何_ 串;第二个增加了一个`if`所以它只匹配` (1, "hello") `. 最后,只是一个变量匹配 _什么_. 这是有用的,如果`match`适用于表达式,并且不希望将变量绑定到该表达式. `_`像变量一样工作,但被忽略. 这是完成一个常用的方法`match`. 

```rust
fn match_tuple(t: (i32,String)) {
    let text = match t {
        (0, s) => format!("zero {}", s),
        (1, ref s) if s == "hello" => format!("hello one!"),
        tt => format!("no match {:?}", tt),
        // or say _ => format!("no match") if you're not interested in the value
     };
    println!("{}", text);
}
```

为什么不匹配` (1, "hello") `?匹配是一个确切的业务,编译器会抱怨: 

      = note: expected type `std::string::String`
      = note:    found type `&'static str`

我们为什么需要`ref s`?这是一个稍微隐晦的问题 (查找E00008错误) ,如果你有一个 _如果守卫_ 你需要借用,因为如果守卫发生在不同的环境中,否则将会发生移动. 这是实施漏洞的情况. 

如果类型 _是_ `&str`那么我们直接匹配它: 

```rust
    match (42,"answer") {
        (42,"answer") => println!("yes"),
        _ => println!("no")
    };
```

什么适用于`match`适用于`if let`. 这是一个很酷的例子,因为如果我们得到一个`Some`,我们可以在里面匹配,只从元组中提取字符串. 所以没有必要嵌套`if let`陈述在这里. 我们用`_`因为我们对元组的第一部分不感兴趣. 

```rust
    let ot = Some((2,"hello".to_string());

    if let Some((_,ref s)) = ot {
        assert_eq!(s, "hello");
    }
    // we just borrowed the string, no 'destructive destructuring'
```

使用时会出现一个有趣的问题`解析{parse}` (或任何需要从上下文中计算出其返回类型的函数) 

```rust
    if let Ok(n) = "42".parse() {
        ...
    }
```

那么,这是什么类型的`n`?不知何故,你必须提供一个提示 - 什么样的整数?它甚至是一个整数?

```rust
    if let Ok(n) = "42".parse::<i32>() {
        ...
    }
```

这种不太优雅的语法被称为"涡轮运算符{turbofish operator}". 

如果你正在返回一个函数`Result`,那么问号运算符提供了一个更加优雅的解决方案: 

```rust
    let n: i32 = "42".parse()?;
```

但是,解析错误需要转换为错误类型`Result`,这是我们稍后讨论时要讨论的话题[错误处理](6-error-handling.html). 

## 关闭{Closures}

Rust的力量来源于很多 _关闭_. 它们最简单的形式就像快捷功能一样: 

```rust
    let f = |x| x * x;

    let res = f(10);

    println!("res {}", res);
    // res 100
```

在这个例子中没有明确的类型 - 一切都是从整数字面量10开始推导出来的. 

如果我们运行,我们会收到错误`f`在不同类型 -  Rust已经决定`f`必须在整数类型上调用: 

        let res = f(10);

        let resf = f(1.2);
      |
    8 |     let resf = f(1.2);
      |                  ^^^ expected integral variable, found floating-point variable
      |
      = note: expected type `{integer}`
      = note:    found type `{float}`

所以,第一次调用修复了参数的类型`x`. 这相当于这个功能: 

```rust
    fn f (x: i32) -> i32 {
        x * x
    }
```

但功能和封闭之间存在很大差异,_距离_ 从明确的打字需要. 这里我们评估一个线性函数: 

```rust
    let m = 2.0;
    let c = 1.0;

    let lin = |x| m*x + c;

    println!("res {} {}", lin(1.0), lin(2.0));
    // res 3 5
```

你不能用明确的做`fn`形式 - 它不知道封闭范围内的变量. 关闭了 _借_ `m`和`c`从其上下文来看. 

现在,这是什么类型`lin`?只要`rustc`知道. 在引擎盖下,封闭是一个 _结构_ 这是可调用的 ('实现调用操作符') . 它的行为就好像它是这样写出来的: 

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

编译器当然是有帮助的,把简单的闭包语法变成所有的代码!你确实需要知道闭包是一个 _结构_ 和它 _借阅_ 来自其环境的价值. 因此它有一个 _lifetime_. 

所有关闭都是独特的类型,但它们有共同的特点. 所以即使我们不知道确切的类型,我们知道一般的约束: 

```rust
fn apply<F>(x: f64, f: F) -> f64
where F: Fn(f64)->f64  {
    f(x)
}
...
    let res1 = apply(3.0,lin);
    let res2 = apply(3.14, |x| x.sin());
```

用英语: `应用{apply}`效劳于 _任何_ 类型`T`这样`T`器物`Fn(f64) -> f64`- 也就是说,这是一个需要的功能`f64`并返回`f64`. 

打完电话后`apply(3.0,lin)`,试图访问`lin`给出一个有趣的错误: 

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

而已,`apply`吃了我们的关闭. 还有这个结构的实际类型`rustc`弥补实施它. 始终将封闭视为结构是有帮助的. 

调用闭包是 _方法调用_: 三种功能 trait 对应于三种方法: 

-   `Fn` 结构传递为`&self`
-   `FnMut` 结构传递为`&mut self`
-   `FnOnce` 结构传递为`self`

所以封闭可能会改变它的 _捕获_ 引用: 

```rust
    fn mutate<F>(mut f: F)
    where F: FnMut() {
        f()
    }
    let mut s = "world";
    mutate(|| s = "hello");
    assert_eq!(s, "hello");
```

注意`mut`-`f`需要可变这个工作. 

但是,你无法逃避借用规则. 考虑这个: 

```rust
let mut s = "world";

// closure does a mutable borrow of s
let mut changer = || s = "world";

changer();
// does an immutable borrow of s
assert_eq!(s, "world");
```

无法完成!错误是我们不能借用`s`在声明中,因为它之前已经被封闭借用过`changer`作为可变的. 只要封闭存在,其他代码就不能访问`s`,所以解决方案是通过将闭包放在一个有限的范围内来控制这个生命周期: 

```rust
let mut s = "world";
{
    let mut changer = || s = "world";
    changer();
}
assert_eq!(s, "world");
```

在这一点上,如果你习惯了JavaScript或Lua等语言,你可能会想到Rust闭包的复杂性,而不是它们在这些语言中的直截了当. 这是Rust承诺不作出任何分配的必要成本. 在JavaScript中,等效`mutate (function(){s ="hello";})`将始终导致动态分配闭包. 

有时你不希望封闭借用这些变量,而是 _移动_ 他们. 

```rust
    let name = "dolly".to_string();
    let age = 42;

    let c = move || {
        println!("name {} age {}", name,age);
    };

    c();

    println!("name {}",name);
```

最后的错误`println`是: "使用移动值: `name`"所以这里有一个解决方案 - 如果我们没有 _想保持_ 名称`活着 - 是将clone副本移入`关闭{move}`中:

```rust
    let cname = name.to_string();
    let c = move || {
        println!("name {} age {}",cname,age);
    };
```

 为什么需要移动`关闭{move}`? 因为我们可能需要在原始上下文不再存在的地方调用它们. 经典案例是创建一个 _thread{线程}_. 移动的封闭不会借用,所以没有 _lifetime_ . 
 
 迭代器方法中主要使用闭包. 回想一下`范围{range}`我们定义的迭代器遍历一系列浮点数. 使用闭包对此 (或任何其他迭代器) 进行操作很简单: 

```rust
    let sine: Vec<f64> = range(0.0,1.0,0.1).map(|x| x.sin()).collect();
```

`map`没有在向量上定义 (尽管很容易创建一个这样的trait) ,因为那样 _一切_ 地图将创建一个新的矢量. 这样,我们有一个选择. 总之,没有创建临时对象: 

```rust
 let sum: f64 = range(0.0,1.0,0.1).map(|x| x.sin()).sum();
```

它 (事实上) 会像写出明确的循环一样快! 如果Rust关闭与Javascript关闭一样"无摩擦{frictionless}",那么这种性能保证是不可能的. 

`过滤filter`是另一种有用的迭代器方法 - 它只允许通过匹配条件的值: 

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

三种类型 (再次) 对应于三种基本参数类型. 

假设我们有一个向量`String`值.以下是显式的迭代器类型,然后 _隐式_,以及迭代器返回的实际类型. 

```rust
for s in vec.iter() {...} // &String
for s in vec.iter_mut() {...} // &mut String
for s in vec.into_iter() {...} // String

// implicit!
for s in &vec {...} // &String
for s in &mut vec {...} // &mut String
for s in vec {...} // String
```

就我个人而言,我更喜欢明确,但了解这两种形式及其含义非常重要. 

`into_iter` _消耗_ 矢量并提取它的字符串,所以之后矢量不再可用 - 它已被移动. 这是 Pythonistas 过去常说的一个确定的问题`for s in vec`!

所以隐含的形式`for s in &vec`通常是你想要的,就像`&T`在向函数传递参数时是一个很好的默认值. 

理解这三种类型是如何工作是很重要的,因为Rust严重依赖于类型推导 - 在关闭参数中你不会经常看到明确的类型. 这是一件好事,因为如果所有这些类型都明确的话它会很嘈杂 _键入_. 然而,这个紧凑的代码的价格是你需要知道隐式类型究竟是什么!

`map`取得迭代器返回的任何值并将其转换为其他值,但是`filter`需要一个 _引用_ 到那个价值. 在这种情况下,我们正在使用`iter`所以迭代器项目类型是`&String`. 注意`filter`接收到这种类型的引用. 

```rust
for n in vec.iter().map(|x: &String| x.len()) {...} // n is usize
....
}

for s in vec.iter().filter(|x: &&String| x.len() > 2) { // s is &String
...
}
```

在调用方法时,Rust会自动删除,所以问题不明显. 但`|x:&& String|`x =="one"|将 _不会_ 工作,因为操作员对类型匹配更加严格. `rustc`会抱怨没有这样的运营商进行比较`&&String`和`&str`. 所以你需要明确的选择来做到这一点`&&String`变成一个`&String`哪一个 _想要_ match. 

```rust
for s in vec.iter().filter(|x: &&String| *x == "one") {...}
// same as implicit form:
for s in vec.iter().filter(|x| *x == "one") {...}
```

如果省略显式类型,则可以修改参数以使其类型`s`就是现在`&String`: 

```rust
for s in vec.iter().filter(|&x| x == "one")
```

这通常是你将如何看待它写的. 

## 用动态数据构建

一个最强大的技术是 _一个包含对自身引用的结构_. 

这是基本构建块 _二叉树_ ,用C表示 (每个人最喜欢的老亲戚都喜欢使用没有保护的电动工具. ) 

```rust
    struct Node {
        const char *payload;
        struct Node *left;
        struct Node *right;
    };
```

你不能这样做 _直接{directly}_ 包含`Node`领域,因为那么大小`Node`取决于大小`Node`...它只是不计算. 所以我们使用指针`Node`结构,因为指针的大小总是已知的. 

如果`left`不是`NULL`,那`Node`将有一个`左{left}`指向另一个节点,因此无限期无限. 

Rust不会`NULL` (至少不是_安然_) 所以这显然是一份工作`Option`. 但你不能只是把一个`Node`在那里面`Option`,因为我们不知道大小`Node` (等等) . 这是一份工作`Box`,因为它包含一个指向数据的分配指针,并且一直具有固定大小. 

所以这里是Rust的等价物,使用`type`创建一个别名: 

```rust
type NodeBox = Option<Box<Node>>;

#[derive(Debug)]
struct Node {
    payload: String,
    left: NodeBox,
    right: NodeBox
}
```

 (Rust以这种方式宽恕 - 不需要前瞻性声明. ) 
 
 并且第一个测试程序: 


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

由于"{:#?}" ('#'表示'extended') ,输出结果非常漂亮. 


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

现在,什么时候发生`root`被丢弃? 所有字段都被删除; 如果树的"分支"被丢弃,_它们_ 就会下降其领域等等. `Box::new`可能是最接近你会得到一个`new`关键字,但我们没有必要`delete`要么`free`. 

我们现在必须为这棵树制定一个用途. 请注意,可以订购字符串: 'bar'&lt;'foo','abba'>'aardvark';所谓的"字母顺序".  (严格来说,这是词汇顺序,因为人类语言非常多样化,并且有着奇怪的规则. ) 

这是一个按字符串的顺序插入节点的方法. 我们将新数据与当前节点进行比较 - 如果较少,则尝试插入左侧,否则尝试插入右侧. 左边可能没有节点,那么`set_left`等等. 

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

注意`match`- 如果是的话,我们会提供一个可变的引用`Option`是`Some`,并应用`insert`方法. 否则,我们需要创建一个新的`Node`对于左侧等等. `Box`是一个 _聪明_ 指针;请注意,不需要"拆箱{unboxing}"来呼叫`Node`方法就可以了!

这里是输出树: 

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

比其他字符串'小于'的字符串放在左侧,否则放在右侧. 

参观时间. 这是 _按顺序遍历_ - 我们访问左边,在节点上做点什么,然后访问右边. 

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

所以我们按顺序访问这些字符串!请注意重新出现`ref`-`if let`使用完全相同的规则`match`. 

## 通用结构

考虑前面的二叉树的例子. 这将是 _严重刺激_ 必须为所有可能的有效载荷重写它. 所以这是我们的通用`Node`与它的类型参数`T`. 

```rust
type NodeBox<T> = Option<Box<Node<T>>>;

#[derive(Debug)]
struct Node<T> {
    payload: T,
    left: NodeBox<T>,
    right: NodeBox<T>
}
```

该实现显示了语言之间的差异. 有效载荷的基本操作是比较,所以T必须与之相当`<`等 实施`PartialOrd`. 类型参数必须在其中声明`impl`阻止其约束: 

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

所以通用结构需要在尖括号中指定类型参数,如C ++. Rust通常足够聪明,可以从上下文中得出这个类型参数 - 它知道它有一个`Node<T>`,并知道它的`insert`方法已通过`T`. 第一次打电话`insert`钉下来`T`成为`String`. 如果有任何进一步的电话不一致,它会投诉. 

但是你确实需要适当地限制这种类型!
