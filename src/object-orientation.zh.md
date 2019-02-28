## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION， INSTEAD RE-RUN doctoc TO UPDATE -->

- [面向对象的 Rust](#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84-rust)
  - [trait 对象](#%E7%89%B9%E8%B4%A8trait-%E5%AF%B9%E8%B1%A1)
- [动物](#%E5%8A%A8%E7%89%A9)
- [鸭子和泛型](#%E9%B8%AD%E5%AD%90%E5%92%8C%E6%B3%9B%E5%9E%8B)
- [继承](#%E7%BB%A7%E6%89%BF)
- [示例: Windows API](#%E7%A4%BA%E4%BE%8B-windows-api)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Rust 中的面向对象

世界各地的人们，以前的编程语言，是以某种方式实现面向对象编程 (OOP) 的可能性很大:

- '类'作为生成 _对象_ (通常被称为 _实例_) 并定义唯一的类型的工厂。
- 类可能 _继承_ 其他类 (_父母_) 的数据 (_字段_) 和行为 (_方法_)。
- 如果 B 继承 A，那么将 B 的一个实例传递给希望接收 A (_子类_)的函数，是可能的。
- 一个对象应该隐藏它的数据 (_封装_) ，只用方法操作。

面向对象的 _设计理念_ 在于识别类 ('名词') 和方法 ('动词') ，然后建立它们之间的关系，关心它 _是一个_ 什么 和它 _有一个_ 什么。

在旧版"星际迷航"系列中，医生会对船长说: "这是人生，吉姆，但不是我们所知道的人生"。 这非常适用于 Rust 的面向对象风格: 它会带来震撼，因为 Rust 数据容器类型 (结构，枚举和元组) 都是很笨的，虽然你可以在其上定义方法，使数据本身是私有的，搭上所有常用的封装策略，但是它们之间都是 _不相干的类型_。 没有父类，没有继承 (除了`Deref`强制转换的个例。)

Rust 中各种数据类型之间的关系，由所拥有的 _trait_ 来确立。 要学好 Rust 的很大一部分，是理解标准库 trait 是如何操作的，因为这是把所有数据类型粘在一起的意识网络。

trait 很有趣，因为它们与主流语言的概念之间没有一一对应的关系。 这取决于你是站在动态还是静态的角度思考。 在动态的情况下，它们更像 Java 或 Go 的接口。

### trait 对象

考虑一下，第一个介绍 trait 的例子:

```rust
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
```

受到很大影响的小程序，如下:

```rust
fn main() {
    let answer = 42;
    let maybe_pi = 3.14;
    let v: Vec<&Show> = vec![&answer,&maybe_pi];
    for d in v.iter() {
        println!("show {}",d.show());
    }
}
// show four-byte signed 42
// show eight-byte float 3.14
```

这是 Rust 需要一些类型指导的一种情况 - 我指定了一个 Vec，其存有实现了 `Show` 的引用 。请注意 i32`和`f64`彼此之间是没有任何关系的，但他们都知道`show`方法，因为它们都实现了相同的 trait 。 这些名称方法是 _虚构的_，因为不同类型的实际方法有不同的代码，而正确的那个方法要根据 _运行时_ 信息决定。或[trait 对象](https://doc.rust-lang.org/stable/book/trait-objects.html)信息。

_这_ 就是你可以将不同类型的对象放在同一个 Vec 的原因(`Vec<&Show>`)。 如果您有 Java 或 Go 背景，您可以把`Show`想成是接口。

细化下 - 我们把值放进`Box`。一个盒子包含对分配在堆上数据的引用，并在行为上非常像引用 - 它是一个 _智能指针_。 像引用，当盒子超出作用域和`Drop`会启动，然后释放内存。

```rust
let answer = Box::new(42);
let maybe_pi = Box::new(3.14);

let show_list: Vec<Box<Show>> = vec![question,answer];
for d in &show_list {
    println!("show {}",d.show());
}
```

不同之处在于，您现在可以使用该 Vec，当成一个引用去传递，或者不必跟踪任何借用的引用就可以将其传递出去。 当 Vec 被释放时，这些盒子值跟被释放，并且所有的内存都被回收。

## 动物

出于某种原因，任何关于面向对象和继承的讨论，似乎最终都会讨论到动物。
它创造了一个不错的故事: "看，猫是食肉动物，而食肉动物是动物"。

但我会从 Ruby 宇宙的经典口号: "如果它嘎嘎叫，那就是鸭子" 开始。 你所有的对象必须做的，就是定义 `嘎嘎{quacks}`方法和，狭义来看，它就是鸭子。

```rust
trait Quack {
    fn quack(&self);
}

struct Duck ();

impl Quack for Duck {
    fn quack(&self) {
        println!("quack!");
    }
}

struct RandomBird {
    is_a_parrot: bool
}

impl Quack for RandomBird {
    fn quack(&self) {
        if ! self.is_a_parrot {
            println!("quack!");
        } else {
            println!("squawk!");
        }
    }
}

let duck1 = Duck();
let duck2 = RandomBird{is_a_parrot: false};
let parrot = RandomBird{is_a_parrot: true};

let ducks: Vec<&Quack> = vec![&duck1,&duck2,&parrot];

for d in &ducks {
    d.quack();
}
// quack!
// quack!
// squawk!
```

在这里，我们有两种完全不同的类型 (其中一个非常笨，甚至没有数据) ，并且是的，它们都能`quacks()`。
其中一个的行为有点奇怪 (比如:一只鸭子(duck))，但他们共享相同的方法名称，Rust 可以从类型安全的方式，保存这类对象的集合。

类型安全是一件奇妙的事情。若没有静态类型，你甚至可以插入一个 _猫_ 进入这个 Quackers 集合，最终导致运行时的混乱。

这是一个有趣的:

```rust
// 为什么不是!
impl Quack for i32 {
    fn quack(&self) {
        for i in 0..*self {
            print!("quack {} ",i);
        }
        println!("");
    }
}

let int = 4;

let ducks: Vec<&Quack> = vec![&duck1,&duck2,&parrot,&int];
...
// quack!
// quack!
// squawk!
// quack 0 quack 1 quack 2 quack 3
```

我能说什么? 它嘎嘎声叫，它一定是一只鸭子。
有趣的是，你可以把你的 trait 应用到任何 Rust 值，而不仅仅是"对象"。

(因`quack`以引用类型的方式传递，需要有明确的解引用符号`*`，来得到整数。)

然而，你只能用同一个库的 trait 和类型来做这件事，标准库是不允许打'补丁'的，这是另一个 Ruby 人的做法 (也还不是最受欢迎的做法) 。

到目前为止，这个 `Quack` trait 表现得非常像 Java 接口，并且同现代 Java 接口一样，实现 _必要_ 方法， _提供_ 这个方法的默认实现。 (该`Iterator`trait 就是一个很好的例子。 )

但是，请注意 trait 不属于 _定义类型_ ，和为任何类型实现新的 trait ，但要受到同一个库的限制。

还能要求，只接收`Quack`实现者的引用:

```rust
fn quack_ref (q: &Quack) {
    q.quack();
}

quack_ref(&d);
```

而这，就是 Rust 风格的'类'。

由于我们在这里进行 101 编程语言比较，所以我提个， Go 对这个嘎嘎工程的一个有趣看法 - 如果有一个 Go 接口`Quack`，和具有`quack`方法的一个类型，那么类型就满足了`Quack`接口，不需要明确的定义。这也打破了定义好的 Java 模型，并且允许编译时填鸭式类型，代价是一些清晰和类型安全。

但是填鸭式类型有一个问题。OOP 的坏标志之一是，太多的方法有一些通用方法名称，如`run`。"如果它已经有了 run()，它必是能运行的"，听起来不像鸭鸭那么友善! 所以这让 Go 接口变成了 _偶然_ 有效。在 Rust，虽然`Debug`和`Display`trait 两者都定义了`fmt`方法，但他们真的就是不同的事情。

所以 Rust 的 trait 允许传统 _多态_ OOP。但继承怎么办呢? 人们常指 _实现继承_ ，Rust 则是 _接口继承_。就好像一位 Java 程序员不去`extend`(扩展)，改为`implements`(实现)。实际上，这是[推荐的做法](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)来自 Alan Holub。他说:

> 我曾经参加过一个 Java 用户组会议，James Gosling (Java 的发明人) 是这个会议的功能
>
> 演讲者. 在令人难忘的问答环节中,有人问他: "如果你能再一次做 Java,
>
> 你会改变什么?""我会放弃 class ,"他回答说,在笑声平息后,

> 他解释说,真正的问题不是类本身,而是实现继承{implementation inheritance}，老是在扩展关系{extends relationship}。

> 接口继承 (实现关系{the implements relationshup}) 是可取的。

> 尽可能避免实现继承{implementation inheritance}

所以即使在 Java 中，你也可能过度使用类。

实现继承有一些严重的问题。但它的确很 _方便_。 如有个臃肿的基类，叫动物和它有很多有用的功能 (它甚至可能暴露它的内部!)，到了我们的派生类，`猫`就可以使用。也就是说，它是一种代码重用的形式。但是代码重用是一个单独的问题。

理解 Rust 时，区分 实现/接口继承 很重要。

请注意， trait 可能有 _已提供_ 的方法。想想`Iterator`- 你只 _需要_ 重写`next`方法，却免费获得大量的方法。 这与现代 Java 接口的"default"方法类似。下面，我们只定义`name`，和`upper_case`是定义好(默认)的。 我们 _可以_ 覆盖`upper_case`，但没有 _必要_。

```rust
trait Named {
    fn name(&self) -> String;

    fn upper_case(&self) -> String {
        self.name().to_uppercase()
    }
}

struct Boo();

impl Named for Boo {
    fn name(&self) -> String {
        "boo".to_string()
    }
}

let f = Boo();

assert_eq!(f.name(),"boo".to_string());
assert_eq!(f.upper_case(),"BOO".to_string());
```

这是个 _如同_ 代码重用的示例，是真的，但注意，它不适用于数据，只适用于接口!

## 鸭子和泛型

Rust 中一个泛型友好的鸭子函数，就是这样一个简单的例子:

```rust
fn quack<Q> (q: &Q)
where Q: Quack {
    q.quack();
}

let d = Duck();
quack(&d);
```

类型参数是 _任何_ 实现了`Quack`的类型。`quack`与上节提到的`quack_ref`之间有一个重要的区别。 这函数的主体会为 _每个_ 调用类型进行编译，并且不需要虚构方法; 这些函数可以完全内联编译。不同的方式使用 `Quack` trait ，作为在泛型类型上的一个 _约束_。

这是相当于 C ++的泛型`quack` (注意这个`const`) :

```cpp
template <class Q>
void quack(const Q& q) {
    q.quack();
}
```

请注意，类型参数不受任何限制。

这是非常多的编译时鸭式输入 - 如果我们传递一个不存在`quack`方法的类型的引用，那么编译器会抱怨没有 `quack` 方法。 至少这个错误是在编译时发现的，但是当一个类型被意外有`quack`时会更糟，Go 就可能发生。 相关的更多模板函数和类会导致可怕的错误消息，因为 _没有_ 对泛型的限制。

你可以定义一个函数，它可以处理在 Quacker 指针上的迭代:

```cpp
template <class It>
void quack_everyone (It start, It finish) {
    for (It i = start; i != finish; i++) {
        (*i)->quack();
    }
}
```

_每个_ 迭代器类型`It`都实现。 Rust 的等价物多少更具挑战性:

```rust
fn quack_everyone <I> (iter: I)
where I: Iterator<Item=Box<Quack>> {
    for d in iter {
        d.quack();
    }
}

let ducks: Vec<Box<Quack>> = vec![Box::new(duck1),Box::new(duck2),Box::new(parrot),Box::new(int)];

quack_everyone(ducks.into_iter());
```

Rust 中的迭代器不是鸭类型的，必须是实现了`Iterator`的类型
，在这种情况下，迭代器提供了一些盒化`Quack`。 所涉及的类型没有歧义，值必须满足`Quack`。 通常，函数签名是一个 Rust 泛型函数的最具挑战性的事情，这就是为什么我建议阅读标准库的源代码 - 实现 通常比 声明简单得多!

这里唯一的类型参数是实际的迭代器类型，意味着，任何可以具有`Box<Duck>`序列的迭代器都能用，而不仅仅是一个 Vec 迭代器。

## 继承

面向对象设计的一个常见问题是，试图将事情强加到一个 _是一个_ 什么的关系中，而忽视 _有一个_ 什么的关系。 [四人帮](https://en.wikipedia.org/wiki/Design_Patterns)
二十二年前在他们的设计模式书中，说过"首选继承布局"。

这里有一个例子: 你想模拟一些公司的员工，并且`雇员{Employee}`似乎是一个类的好名字。然后，经理 **是一个** 员工 (这是真的) ，所以我们开始用一个构建我们的层次结构：

`Employee`的子类`经理{Manager}`。这并不像看起来那么流畅。 也许是我们对识别重要名词感到厌倦，也许我们 (无意识地) 认为经理和员工是不同种类的动物? 更好的方式，是雇员 **有一个** 一个 `Roles（角色）`的集合，然后一个经理，仅仅是一个有更多的责任和能力的`Employee`。

或考虑车辆 - 从自行车到 300 吨矿车。 有多种方式可以考虑车辆，道路需求 (全地形，城市，铁路等) ，电源来源 (电力，柴油，柴油电力等) ，运货物还是人等等。当您根据一个方面，去创建任何固定层次的类，都会忽略所有其他方面。也就是说，可能有多种的车辆分类!

布局在 Rust 中更为重要，原因很明显，您无法从基类以惰性方式继承功能。

布局还是很重要，因为借用检查器足够聪明，可以知道借来的不同结构字段都是独立的借用。你可以有一个字段的可变借用，同时拥有另一个字段的不可变借用，等等。

Rust 不能说，一个方法只访问一个字段，所以为了实现方便，这些字段应该用自己的方法来构造。 (结构的 _外部_ 接口，可以是任何你喜欢使用的合适 trait。 )

"拆分借来的{split borrowing}"的一个具体例子，会使这个更清晰。

有个拥有一些字符串的结构，和一个方法，是说能可变借用第一字符串。

```rust
struct Foo {
    one: String,
    two: String
}

impl Foo {
    fn borrow_one_mut(&mut self) -> &mut String {
        &mut self.one
    }
    ....
}
```

(这是 Rust 命名约定的一个例子 - 这类方法应该以`_mut`结尾)

现在，一种借用两个字符串的方法，重用第一种方法:

```rust
    fn borrow_both(&self) -> (&str,&str) {
        (self.borrow_one_mut(), &self.two)
    }
```

这会失败! 因我们既有个`self`的可变借用，*又*有个 _也_ `self`的不可变借用。 如果 Rust 允许这样的情况发生，那么无法保证 不可变引用(借用) 不会改变。

解决方案很简单:

```rust
    fn borrow_both(&self) -> (&str,&str) {
        (&self.one, &self.two)
    }
```

好了，因为借用检查员，认为这些是独立的借用。所以想象这些字段是一些任意类型，你可以看到在这些字段上调用的方法，不会导致借用问题。

使用[Deref](https://rust-lang.github.io/book/second-edition/ch15-02-deref.html)是一种限制但非常重要的"继承"，这是'解引用'符号`*`(语法糖)的 实际 trait 。`String`实现了`Deref<Target=str>`，所以`＆str`上定义的所有方法，自动都可用于`String`! 类似的，`Foo`的方法可以直接调用`Box<Foo>`，有些人觉得这有点... 神奇，但它是非常方便能力。
现代 Rust 中有一种更简单的语言，但使用起来并不令人愉快。
它确实应该用于，一个具有所有权-可变的类型和一个简单的借用类型的情况。

一般来说， 这就是 Rust 中的 _trait 继承_:

```rust
trait Show {
    fn show(&self) -> String;
}

trait Location {
    fn location(&self) -> String;
}

trait ShowTell: Show + Location {}
```

最后一个 trait 简单地将我们两个不同的 trait 合并为一个，尽管它可以指定其他方法。

现在的情况和以前一样:

```rust
#[derive(Debug)]
struct Foo {
    name: String,
    location: String
}

impl Foo {
    fn new(name: &str, location: &str) -> Foo {
        Foo{
            name: name.to_string(),
            location: location.to_string()
        }
    }
}

impl Show for Foo {
    fn show(&self) -> String {
        self.name.clone()
    }
}

impl Location for Foo {
    fn location(&self) -> String {
        self.location.clone()
    }
}

impl ShowTell for Foo {}
```

现在，如果我有`Foo`类型的`foo`值，那么对该值的引用将会满足`&Show`，`&Location`或是`&ShowTell` (这暗示着两者) 三个。

这是一个有用的小宏:

```rust
macro_rules! dbg {
    ($x:expr) => {
        println!("{} = {:?}",stringify!($x),$x);
    }
}
```

它需要一个参数 (用`$x`表示) 必须是一个'表达式(expression)'。 我们打印出它的值，和一个 _字符串化_ 的版本。 C 程序员会在这一点上有些 _小_ 得意，这意味着如果我传递了`1 + 2` (一个表达式) `stringify!(1 + 2)`是字面字符串"1 + 2"。 这会为我们在玩代码时节省一些打字的时间:

```rust
let foo = Foo::new("Pete","bathroom");
dbg!(foo.show());
dbg!(foo.location());

let st: &ShowTell = &foo;

dbg!(st.show());
dbg!(st.location());

fn show_it_all(r: &ShowTell) {
    dbg!(r.show());
    dbg!(r.location());
}

let boo = Foo::new("Alice","cupboard");
show_it_all(&boo);

fn show(s: &Show) {
    dbg!(s.show());
}

show(&boo); // `Show`引用传递给`show`

// foo.show() = "Pete"
// foo.location() = "bathroom"
// st.show() = "Pete"
// st.location() = "bathroom"
// r.show() = "Alice"
// r.location() = "cupboard"
// s.show() = "Alice"
```

这些就 _是_ 面向对象，但不是你习惯的那种。

请注意，`Show`引用传递给`show`，它不可能是 _动态_ 升级为`ShowTell`! 占据更多动态类系统范畴的语言，允许您检查给定对象是否是类的实例，然后对该类型执行动态转换。 一般来说这不是一个好主意，特别是不能在 Rust 中工作，因为`Show`引用已经"忘记"它最初是一个`ShowTell`引用。

你总有选择: 多态，通过 trait 对象，或是单态，通过泛型约束的 trait 。 现代 C ++和 Rust 标准库倾向于采用泛型路由，但多态路由并未过时。 您必须了解'路'的不同 - 泛型生成最快的代码，且可以内联。 这可能会导致代码膨胀。 但并非所有事情都要 _尽可能快_- 有时某个程序运行的生命周期中，可能只发生"那么"几次。

最后，这里有一个总结:

- `class`所扮演的角色在数据和 traits 之间共享。
- 结构和枚举是笨的，虽然你可以定义方法和做数据隐藏。
- 使用`Deref` trait ，可以对数据进行一个子类型化的 _限制_ 形式。
- trait 没有任何数据，但可以实现任何类型 (不仅仅是结构)。
- trait 可以从其他 trait 继承。
- trait 可以提供方法，允许接口代码重用。
- trait 给你两个虚构方法 (多态) 和泛型约束 (单态)。

## 示例: Windows API

GUI 工具包是广泛使用传统 OOP 的领域之一。 一个`EditControl`或者一个`ListWindow`**是一个**`Window`等等。这使得编写 Rust 绑定到 GUI 工具包，比使用它更困难。

Win32 编程在 Rust 中可以[直接](https://www.codeproject.com/Tips/1053658/Win-GUI-Programming-In-Rust-Language)完成，它比原来的 C 稍微笨拙一点。 当我从 C 到 C ++ 毕业时，我想要更干净的东西，并且做了我自己的 OOP 包装。

一个典型的 Win32 API 函数是[ShowWindow](https://docs.rs/user32-sys/0.0.9/i686-pc-windows-gnu/user32_sys/fn.ShowWindow.html)，用于控制窗口的可见性。 现在，一个`EditControl`有一些专门的功能，但它都是用 Win32 `HWND` ('handle to window') 不透明的值完成的。若你想要`EditControl`也有一个`show`方法，传统上这将通过实现继承来完成，但您 _不_ 想要为每种类型输出所有这些继承的方法! 而 Rust trait 提供了一个解决方案，这会有一个`Window` trait :

```rust
trait Window {
    // 你需要定义这个！
    fn get_hwnd(&self) -> HWND;

    // 所有这些都将提供
    fn show(&self, visible: bool) {
        unsafe {
         user32_sys::ShowWindow(self.get_hwnd(), if visible {1} else {0})
        }
    }

    // .....在 Windows 上运行的大量方法

}
```

所以，`EditControl`的实现结构只能包含一个`HWND`，并通过定义一种方法实现`Window`; `EditControl`是一种继承自`Window`的 trait ，并定义了扩展接口。比如像`ComboxBox`这样的 - 其行为像一个`EditControl` _和_ 可以通过 trait 继承轻松实现 一个`ListWindow`。

Win32 API ('32'不再意味着'32 位') 实际上是面向对象的，但是老一辈，受 Alan Kay 定义的影响: 对象包含隐藏的数据，并且由 _消息{messages}_ 控制。因此，任何 Windows 应用程序的核心都有一个消息循环，各种窗口 (称为'窗口类') 都用它们自己的 switch 语句实现这些方法。 其中有一个消息，可能有不同的实现，叫`WM_SETTEXT`: 标签的文本更改，顶级窗口的标题会变化。

[这里](https://gabdube.github.io/native-windows-gui/book_20.html)是一个相当有前途的最小 Windows GUI 框架。 但根据我的口味，有太多了`unwrap`实例 - 其中一些甚至没有错误。

这是因为 NWG 正在利用消息的松散动态性质。通过适当的类型安全接口，编译时会捕获更多的错误。

在[下一版](https://rust-lang.github.io/book/second-edition/ch17-00-oop.html)的 Rust 编程语言手册中， Rust 对面向对象的含义进行了很好的讨论。
