
## 面向对象的Rust

每个人都来自某个地方,以前的编程语言以特定的方式实现了面向对象编程 (OOP) 的可能性很大: 

-   'classes'作为工厂生成 _对象_ (通常被称为 _实例_) 并定义唯一的类型. 
-   类可能 _继承_ 来自其他 class (他们的 _父母_) ,继承数据 (_fields_) 和行为 (_方法_) 
-   如果 A 是 B 的 父母,那么可以将B的一个实例传递给期望A (_分类{subtyping}_) 
-   一个对象应该隐藏它的数据 (_封装_) ,只能用方法操作. 

面向对象 _设计_ 然后是识别类 ('名词') 和方法 ('动词') ,然后建立它们之间的关系, _是一个_ 和 _有一个_. 

在旧版"星际迷航"系列中,医生会对船长说: "这是人生,吉姆,而不是我们所知道的生活". 这非常适用于Rust风格的面向对象: 它会带来震撼,因为Rust数据聚合 (结构,枚举和元组) 是愚蠢的. 你可以在其上定义方法,并使数据本身是私有的,所有通常的封装策略,但它们都是 _无关的类型_. 没有数据分类和数据继承 (除了专门的案例) `Deref`强制转换. ) 

Rust中各种数据类型之间的关系使用 _ trait {traits}_. 学习Rust的很大一部分是理解标准库 trait 如何操作,因为这是将所有数据类型粘合在一起的意义网络. 

特质{trait} 很有趣,因为它们与主流语言的概念之间没有一一对应的关系. 这取决于你是动态还是静态思考. 在动态的情况下,它们更像Java或Go接口 - interfaces. 

### 特质{trait} 对象

考虑一下第一个用来介绍特质{trait} 的例子: 

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

这是一个有很大影响的小程序: 

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

这是一个Rust需要某种类型指导的情况 - 我特别需要一个引用的向量是关于 impl 的`显示{Show}`. 现在请注意i32`和`f64`彼此没有任何关系,但他们都明白这一点`显示`方法,因为它们都实施相同的特质{trait} . 这种方法是 _虚拟{virtual}_,因为实际的方法对于不同的类型有不同的代码,但是正确的方法是基于的 _运行时{runtime}_ 信息. 这些引用被调用[特质{trait} 对象](https://doc.rust-lang.org/stable/book/trait-objects.html)

和 _那_ 是如何将不同类型的对象放在同一个 向量{Vec} 中. 如果您来自Java或Go背景,您可以考虑`显示{Show}`充当接口. 

这个例子的一点细化 - 我们 _框{box}_ 价值. 一个盒子包含对分配在堆上的数据的引用,并且非常像引用 - 它是一个 _智能指针_. 当库超出范围和`Drop`启动,然后释放内存. 

```rust
let answer = Box::new(42);
let maybe_pi = Box::new(3.14);

let show_list: Vec<Box<Show>> = vec![question,answer];
for d in &show_list {
    println!("show {}",d.show());
}
```

不同之处在于,您现在可以使用此 Vec ,将其作为引用传递给它,或者不必跟踪任何借用的引用就可以将其传递出去. 当 Vec 被丢弃时,这些框将被丢弃,并且所有的内存都被回收. 

## 动物

出于某种原因,任何关于面向对象和继承的讨论似乎最终都会讨论动物. 
它创造了一个不错的故事: "看,猫是食肉动物,食肉动物是动物". 

但我会从Ruby宇宙的经典口号开始: "如果它嘎嘎叫,那就是鸭子". 你所有的对象必须做的是定义 `嘎嘎{quacks}` 他们可以被认为是鸭子,虽然在一个非常狭窄的方式.

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

在这里,我们有两种完全不同的类型 (其中一个非常愚蠢,甚至没有数据) ,并且是的,它们都是` quacks()`. 
其中一个有点奇怪 (对于鸭子) ,但他们共享相同的方法名称,Rust可以从类型安全的方式保存这些对象的集合. 

类型安全是一件奇妙的事情. 没有静态类型,你可以插入一个 _猫_ 进入这个Quackers集合,导致运行时混乱. 

这是一个有趣的: 


```rust
// and why the hell not!
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

我能说什么? 它嘎嘎声叫,它一定是一只鸭子. 
有趣的是,你可以将你的 trait 应用于任何 rust 值,而不仅仅是"对象". 
(以来`嘎嘎`通过引用,有明确的解除引用`*`得到整数. ) 

然而,你只能用同一个库的特质{trait} 和类型来做这件事,所以标准库不能'猴子补丁',这是另一块红宝石人的做法 (也不是最受人羡慕的) . 

到目前为止,这个特质{trait} `嘎嘎`表现得非常像Java接口,并且像现代Java接口一样 _提供_ 这个方法.如果你已经实现了,它将提供一个默认实现 _需要_ 方法.  (该`迭代器`特质{trait} 就是一个很好的例子. ) 

但是,请注意特质{trait} 不属于 _定义类型_ 和实施任何类型的新 trait ,但要受同一库限制. 

可以将引用传递给任何人`嘎嘎`实施者: 

```rust
fn quack_ref (q: &Quack) {
    q.quack();
}

quack_ref(&d);
```

这是分类,Rust风格. 

由于我们在这里进行编程语言比较101,所以我会提到Go对这个嘎嘎的业务有一个有趣的看法 - 如果有一个Go接口`嘎嘎`,而一个类型有一个`嘎嘎`方法,那么类型就满足了`嘎嘎`而不需要明确的定义. `这也打破了定义好的Java模型,并且允许编译时鸭式输入,代价是一些清晰和类型安全. 但是鸭子打字有一个问题. 

坏OOP的标志之一是, 太多的方法有一些通用名称`run`. `"如果它已经有了 run(),它必须是可 run 的"`,听起来不像原来那么容易! 所以Go接口可能会是 _偶然_ 有效 在Rust,两者都是`Debug`和`Display`特质{trait} 定义`fmt`方法,但他们真的意味着不同的事情. 

所以Rust的特质{trait} 允许传​​统 _多态_ OOP. 但是遗传呢?人们通常是指 _实现继承{impl}_ 而Rust则是 _接口继承_. 就好像一位Java程序员从未使用过`延伸{extend}`而是使用`实现{implements}`. 实际上,这是[推荐的做法](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)通过Alan Holub. 他说: 

> 我曾经参加过一个Java用户组会议,James Gosling (Java的发明人) 是这个会议的特色
>
> 扬声器. 在令人难忘的问答环节中,有人问他: "如果你能再一次做Java,
>
> 你会改变什么?""我会放弃 class ,"他回答说,在笑声平息后,

> 他解释说,真正的问题不是类本身,而是实现继承{implementation inheritance}(扩展关系{extends relationship}) . 

> 接口继承 (实现关系{the implements relationshup}) 是可取的. 

> 尽可能避免实现继承{implementation inheritance}

所以即使在Java中,你也可能过度使用类. 

实现继承有一些严重的问题. 

但它的确如此 _方便_. 有这个胖基类叫动物和它有很多有用的功能 (它甚至可能暴露它的内部!) ,我们的派生类`猫`可以使用. 也就是说,它是一种代码重用的形式. 

但是代码重用是一个单独的问题. 理解Rust时,区分 实现 和 接口继承 很重要. 

请注意, trait 可能有 _提供_ 方法. 考虑`迭代器`- 你只 _需要_ 重写`next`,但却免费获得大量的方法. 这与现代Java接口的"default"方法类似. 这里我们只定义`名称{name}`和`upper_case`是为我们定义的. 我们 也是 _可以_ 覆盖`upper_case`,但事实并非如此 _需要_. 

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

这是一个 _类_ 的代码重用,为真,但请注意,它不适用于数据,只有 接口!

## 鸭子和泛型

Rust中一个通用友好的鸭子函数的例子就是这样一个简单的例子: 

```rust
fn quack<Q> (q: &Q)
where Q: Quack {
    q.quack();
}

let d = Duck();
quack(&d);
```

类型参数是 _任何_ 实现的类型`嘎嘎{Quack}`. 之间有一个重要的区别`嘎嘎`和`quack_ref`在上一节中定义. 这个函数的 main 是编译的 _每个_ 调用类型,并且不需要虚拟方法;这些功能可以完全内联. 它使用特质{trait} `嘎嘎`以不同的方式,作为一个 _约束{where}_ 在泛型类型上. 

这是相当于通用的C ++`嘎嘎` (注意`常量`) : 

```cpp
template <class Q>
void quack(const Q& q) {
    q.quack();
}
```

请注意,类型参数不受任何限制. 

这是非常多的编译时鸭式输入 - 如果我们将 非quackable 类型引用传递,那么编译器会抱怨 `不嘎嘎` 方法. 至少这个错误是在编译时发现的,但是当一个类型被意外Quackable时会更糟,就像Go一样. 更多涉及的模板函数和类导致可怕的错误消息,因为 _没有_ 对泛型的限制. 你可以定义一个可以处理Quacker指针迭代的函数: 


```cpp
template <class It>
void quack_everyone (It start, It finish) {
    for (It i = start; i != finish; i++) {
        (*i)->quack();
    }
}
```

这将会被执行 _每个_ 迭代器类型`It`. Rust的等价物有点更具挑战性:

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

Rust中的迭代器不是鸭式的,但是必须是 impl `迭代器`的类型
,在这种情况下,迭代器提供了一些盒子`嘎嘎`. 所涉及的类型没有歧义,值必须满足`嘎嘎`. 通常,函数签名是关于通用Rust函数的最具挑战性的事情,这就是为什么我建议阅读标准库的源代码 - impl 通常比 声明 简单得多!

这里唯一的类型参数是实际的迭代器类型,这意味着这将适用于任何可以传递序列的任何东西`Box<Duck>`,而不仅仅是一个向量迭代器. 

## 继承

面向对象设计的一个常见问题是试图将事情强加到一个 _是一个_ 关系,而忽视 _有一个_ 关系. [该四人帮 ](https://en.wikipedia.org/wiki/Design_Patterns)
二十二年前在他们的设计模式书中说过"首选组合继承".

这里有一个例子: 你想模拟一些公司的员工,并且`雇员{Employee}`似乎是一个 class 的好名字. 

然后,经理是一个员工 (这是真的) ,所以我们开始用一个构建我们的层次结构

`雇员`的子类`经理{Manager}`. `这并不像看起来那么聪明. 也许我们对识别重要名词感到厌倦,也许我们 (无意识地) 认为经理和员工是不同种类的动物? 这更好的是雇员`有一个`角色`集合,然后一个经理就是一个`雇员`有更多的责任和能力. 

或考虑车辆 - 从自行车到300吨矿车. 有多种方式可以考虑车辆,道路 (全地形,城市,铁路等) ,电源 (电力,柴油,柴油电力等) ,货物或人等等. 您根据一个方面创建的任何固定层次的类都会忽略所有其他方面. 也就是说,有多种可能的车辆分类!

构成在Rust中更为重要,原因很明显,您无法从基类以惰性方式继承功能. 

组合也很重要,因为借用检查器足够聪明,可以知道借用不同的结构域是单独的借入. 

你可以有一个字段的可变借入,同时拥有另一个字段的不可变借入,等等. Rust不能告诉一个方法只访问一个字段,所以为了方便实现,这些字段应该用自己的方法来构造.  (该 _外部_ 结构的接口可以是任何你喜欢使用合适的 trait 的. ) 

"拆分拆分{split borrowing}"的一个具体例子会使这个更清晰. 

我们拥有一个一些字符串的结构,并且有一个可变的借用第一个字符串的方法. 

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

 (这是Rust命名约定的一个例子 - 这些方法应该以`_mut`) `
 
 现在,一种借用两个字符串的方法,重用第一种方法: 


```rust
    fn borrow_both(&self) -> (&str,&str) {
        (self.borrow_one_mut(), &self.two)
    }
```

不行! 我们从可变的`self`和 _也_ 从中 不可变 借用 `self`. 如果Rust允许这样的情况发生,那么无法保证 不可改变的引用 不会改变. 

解决方案很简单: 


```rust
    fn borrow_both(&self) -> (&str,&str) {
        (&self.one, &self.two)
    }
```

这很好,因为借款检查员认为这些是独立的借款. 

所以想象这些字段是一些任意的类型,你可以看到在这些字段上调用的方法, 不会导致借用问题. 有一种限制但非常重要的"继承"[Deref](https://rust-lang.github.io/book/second-edition/ch15-02-deref.html)
这是'取消引用'操作符'*`的 trait 。
`String`实现`Deref<Target=str>`，所以`＆str`上定义的所有方法都是自动的
也可用于`String`! 以类似的方式，`Foo`的方法可以直接
调用`Box<Foo>`。 有些人觉得这有点神奇，但它非常方便。
现代Rust中有一种更简单的语言，但使用起来并不令人愉快。
它确实应该用于那些拥有可变类型和更简单借用的情况
类型。

一般在Rust中有 _特质{trait} 继承_: 

```rust
trait Show {
    fn show(&self) -> String;
}

trait Location {
    fn location(&self) -> String;
}

trait ShowTell: Show + Location {}
```

最后一个 trait 简单地将我们两个不同的 trait 合并为一个,尽管它可以指定其他方法. 

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

现在,如果我有价值`foo`类型`Foo`,那么对该值的引用将会满足`&Show`,`&Location`要么`&ShowTell` (这暗示着两者) . 

这是一个有用的小宏: 

```rust
macro_rules! dbg {
    ($x:expr) => {
        println!("{} = {:?}",stringify!($x),$x);
    }
}
```

它需要一个参数 (用`$x`表示) 必须是'表达'. 我们打印出它的价值,并且一个 _字符串化_ 值的版本. C程序员可以是在这一点上 _小_ 得意,但这意味着如果我通过了`1 + 2` (一个表达) `stringify!(1 + 2)`是文字字符串"1 + 2". 这会为我们在玩代码时节省一些打字的时间: 

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

show(&boo);

// foo.show() = "Pete"
// foo.location() = "bathroom"
// st.show() = "Pete"
// st.location() = "bathroom"
// r.show() = "Alice"
// r.location() = "cupboard"
// s.show() = "Alice"
```

这个 _是_ 面向对象,而不是你习惯的那种. 

请注意`Show`引用传递给`show`, 不可能是 _动态_ 升级为`ShowTell`! 具有更多动态类系统的语言允许您检查给定对象是否是类的实例, 然后对该类型执行动态转换. 一般来说这不是一个好主意,因为这个原因,特别是不能在Rust中工作, `Show`引用已经"忘记"它最初是一个`ShowTell`引用. 

你总是有选择: 多态,通过特质{trait} 对象,或单态,通过泛型约束的特质{trait} . 现代C ++和Rust标准库倾向于采用通用路由,但多态路由并未过时. 您必须了解不同的权衡 - 泛型生成最快的代码,可以内联. 这可能会导致代码膨胀. 但并非所有事情都必须如此 _尽可能快_- 在典型的程序 run 的生命周期中,它可能只发生"少"几次. 

所以,这里有一个总结: 

-   `类`扮演的角色在数据和 trait 之间共享
-   结构和枚举是愚蠢的,虽然你可以定义方法和做数据隐藏
-   一个 _有限_ 子类型的形式是可能的数据使用`Deref` trait 
-    trait 没有任何数据,但可以实现任何类型 (不仅仅是结构) 
-    trait 可以从其他 trait 继承
-    trait 可以提供方法,允许接口代码重用
-    trait 给你两个虚拟方法 (多态) 和通用约束 (单态) 

## 示例: Windows API

GUI工具包是传统OOP广泛使用的领域之一. 一个`EditControl`或者一个`ListWindow`是-一个`窗口{Window}`等等. 这使得编写Rust绑定到GUI工具包比编写它更困难. 

Win32编程可以在Rust中完成[直接](https://www.codeproject.com/Tips/1053658/Win-GUI-Programming-In-Rust-Language),它比原来的C稍微笨拙一点. 当我从C到C ++毕业时,我想要更干净的东西,并且做了我自己的OOP包装. 典型的Win32 API函数是[ShowWindow](https://docs.rs/user32-sys/0.0.9/i686-pc-windows-gnu/user32_sys/fn.ShowWindow.html)用于控制窗口的可见性. 现在,一个`EditControl`有一些专门的功能,但它都是用Win32完成的`HWND` ('handle to window') 不透明的值. 你想要`EditControl`也有一个`show`方法,传统上这将通过实现继承来完成. 您 _不_ 想要为每种类型输出所有这些继承的方法!但Rust trait 提供了一个解决方案. 会有一个`窗口{Window}` trait : 

```rust
trait Window {
    // you need to define this!
    fn get_hwnd(&self) -> HWND;

    // and all these will be provided
    fn show(&self, visible: bool) {
        unsafe {
         user32_sys::ShowWindow(self.get_hwnd(), if visible {1} else {0})
        }
    }

    // ..... oodles of methods operating on Windows

}
```

所以,实现struct为`EditControl`可以包含一个`HWND`, 并通过定义一种方法执行`窗口{Window}`;`EditControl`是一种继承自的`窗口{Window}`特质{trait} 并定义扩展接口. 就像是`ComboxBox`- 其行为像一个`EditControl` _和_ 一个`ListWindow`可以通过 trait 继承轻松实现. 

Win32 API ('32'不再意味着'32位') 实际上是面向对象的,但是老式的,受 Alan Kay 定义的影响: 对象包含隐藏的数据,并且由 _消息{messages}_ 控制. 因此,任何Windows应用程序的核心都有一个消息循环,各种窗口 (称为'窗口类') 都用它们自己的switch 语句实现这些方法. 有一个消息叫`WM_SETTEXT`但实现可能会有所不同: 标签的文本更改,但顶级窗口的标题更改. 

[这里](https://gabdube.github.io/native-windows-gui/book_20.html)是一个相当有前途的最小Windows GUI框架. 但根据我的口味,有太多了`unwrap`实例正在发生 - 其中一些甚至没有错误. 

这是因为 NWG 正在利用消息的松散动态性质. 通过适当的类型安全接口,编译时会捕获更多的错误. 

在[下一版](https://rust-lang.github.io/book/second-edition/ch17-00-oop.html)Rust编程语言手册 对Rust中 面向对象的含义 进行了很好的讨论. 
