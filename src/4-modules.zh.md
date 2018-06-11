
# 模块和货物

## 模块

随着程序变得越来越大,有必要将它们分散到多个文件中,并将函数和类型放在不同的位置_命名空间_. 这两种解决方案都是Rust解决方案_模块_. 

C做的是第一个,而不是第二个,所以你最终会遇到类似的可怕名字`primitive_display_set_width`等等. 实际的文件名可以任意命名. 

Rust在全名看起来像`原始::显示:: set_width`,之后说`使用primitive :: display`你可以把它称为`显示:: set_width`. 你甚至可以说`使用primitive :: display :: set_width`然后只是说`set_width`,但这并不是一个好主意. `rustc`不会混淆,但是_您_稍后可能会感到困惑. 但为了这个工作,文件名必须遵循一些简单的规则. 

一个新的关键字`mod`用于将模块定义为可以写入Rust类型或函数的块: 

```rust
mod foo {
    #[derive(Debug)]
    struct Foo {
        s: &'static str
    }
}

fn main() {
    let f = foo::Foo{s: "hello"};
    println!("{:?}", f);
}
```

但它仍然不太正确 - 我们得到'struct Foo是私人的'. 为了解决这个问题,我们需要`酒馆`要导出的关键字`富`. 然后错误更改为'结构foo :: Foo的字段是私人的',所以放`酒馆`前场`小号`出口`富::小号`. 然后事情就会奏效. 

```rust
    pub struct Foo {
        pub s: &'static str
    }
```

需要一个明确的`酒馆`意味着你必须_选择_哪些项目要通过模块公开. 从模块导出的一组函数和类型称为它的_接口_. 

隐藏结构的内部通常会更好,并且只允许通过方法访问: 

```rust
mod foo {
    #[derive(Debug)]
    pub struct Foo {
        s: &'static str
    }

    impl Foo {
        pub fn new(s: &'static str) -> Foo {
            Foo{s: s}
        }
    }
}

fn main() {
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

为什么隐藏实施是一件好事?因为这意味着您可以在不中断界面的情况下稍后进行更改,否则模块的消费者不会太依赖其细节. 大规模编程的大敌是代码纠结的倾向,因此理解一段代码是不可能的. 

在一个完美的世界里,一个模块做一件事,做得好,并保持自己的秘密. 

何时不要隐藏?正如Stroustrup所说,当接口_是_执行,就像`struct Point {x: f32,y: f32}`. 

_中_一个模块,所有项目对所有其他项目都可见. 这是一个舒适的地方,每个人都可以成为朋友,知道彼此的私密细节. 

每个人都可以根据自己的喜好,将程序分成不同的文件. 我开始对500条线路感到不舒服,但我们都同意超过2000条线路正在推动它. 

那么如何将这个程序分解成单独的文件呢?

我们把这个`foo`代码到`foo.rs`: 

```rust
// foo.rs
#[derive(Debug)]
pub struct Foo {
    s: &'static str
}

impl Foo {
    pub fn new(s: &'static str) -> Foo {
        Foo{s: s}
    }
}
```

并使用一个`mod foo`声明_无_主程序中的一个块: 

```rust
// mod3.rs
mod foo;

fn main() {
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

现在`rustc mod3.rs`会引发`foo.rs`也要编译. 没有必要用makefiles搞笑!

编译器也会看`的modname / mod.rs`,所以这将工作,如果我创建一个目录`嘘`包含一个文件`mod.rs`: 

```rust
// boo/mod.rs
pub fn answer()->u32 {
    42
}
```

现在主程序可以将两个模块作为单独的文件使用: 

    // mod3.rs
    mod foo;
    mod boo;

    fn main() {
        let f = foo::Foo::new("hello");
        let res = boo::answer();
        println!("{:?} {}", f,res);
    }

到目前为止,还有`mod3.rs`,含有`主要`,一个模块`foo.rs`和一个目录`嘘`含`mod.rs`. 通常的惯例是包含的文件`主要`只是叫`main.rs`. 

为什么有两种方法可以做同样的事情?因为`BOO / mod.rs`可以参考其中定义的其他模块`嘘`,更新`BOO / mod.rs`并添加一个新模块 - 注意这是明确导出的.  (没有`酒馆`,`酒吧`只能在里面看到`嘘`模块) . 

```rust
// boo/mod.rs
pub fn answer()->u32 {
	42
}

pub mod bar {
    pub fn question() -> &'static str {
        "the meaning of everything"
    }
}
```

然后我们有与答案相对应的问题 (`酒吧`模块在里面`嘘`) : 

```rust
let q = boo::bar::question();
```

该模块块可以被拉出`BOO / bar.rs`: 

```rust
// boo/bar.rs
pub fn question() -> &'static str {
    "the meaning of everything"
}
```

和`BOO / mod.rs`变为: 

```rust
// boo/mod.rs
pub fn answer()->u32 {
	42
}

pub mod bar;
```

总之,模块是关于组织和可见性的,这可能涉及或不涉及单独的文件. 

请注意`使用`与导入无关,只是指定模块名称的可见性. 例如: 

```rust
{
    use boo::bar;
    let q = bar::question();
    ...
}
{
    use boo::bar::question();
    let q = question();
    ...
}
```

重要的一点是没有_单独编译_这里. 主程序及其模块文件将每次重新编译. 尽管如此,较大的程序需要花费相当长的时间`rustc`在渐进式编译中越来越好. 

## 包装箱

Rust的"编译单位"是_箱_,它是一个可执行文件或一个库. 

要分别编译上一节中的文件,请先构建`foo.rs`作为锈_静态库_箱: 

    src$ rustc foo.rs --crate-type=lib
    src$ ls -l libfoo.rlib
    -rw-rw-r-- 1 steve steve 7888 Jan  5 13:35 libfoo.rlib

我们现在可以_链接_这到我们的主要程序中: 

    src$ rustc mod4.rs --extern foo=libfoo.rlib

但主要程序现在必须像这样,在那里`extern`名称与链接时使用的名称相同. 有一个隐含的顶级模块`foo`与图书馆箱子相关联: 

    // mod4.rs
    extern crate foo;

    fn main() {
        let f = foo::Foo::new("hello");
        println!("{:?}", f);
    }

人们开始吟唱'货物!货物!'让我来证明这个构建Rust的低级视图. 我是'Know Thy Toolchain'的忠实信徒,当我们看着使用Cargo管理项目时,这会减少你需要学习的新魔法数量. 

模块是基本的语言功能,可用于货运项目之外. 

    src$ ls -lh mod4
    -rwxrwxr-x 1 steve steve 3,4M Jan  5 13:39 mod4

现在该理解为什么Rust的二进制文件如此之大: 这很胖!_有一个_批量在该可执行文件中的调试信息. 

这不是一件坏事,如果你想使用一个调试器,并且当你的程序发生混乱时实际上需要有意义的回溯. 

    src$ strip mod4
    src$ ls -lh mod4
    -rwxrwxr-x 1 steve steve 300K Jan  5 13:49 mod4

那么让我们去除这些调试信息并查看: _对于如此简单的事情,仍感觉有点大,但是这个程序链接_静态针对Rust标准库. 这是一件好事,因为您可以将此可执行文件交给任何具有正确操作系统的人 - 他们不需要"Rust运行时". ` (和`rustup甚至可以让你为其他操作系统和平台进行交叉编译. ) 

我们可以动态链接到Rust运行时并获得真正的小exes: 

    src$ rustc -C prefer-dynamic mod4.rs --extern foo=libfoo.rlib
    src$ ls -lh mod4
    -rwxrwxr-x 1 steve steve 14K Jan  5 13:53 mod4
    src$ ldd mod4
    	linux-vdso.so.1 =>  (0x00007fffa8746000)
    	libstd-b4054fae3db32020.so => not found
    	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3cd47aa000)
    	/lib64/ld-linux-x86-64.so.2 (0x00007f3cd4d72000)

这'找不到'是因为`rustup`不会全局安装动态库. 至少在Unix上我们可以破解我们的快乐方式 (是的,我知道最好的解决方案是符号链接) . Rust没有

    src$ export LD_LIBRARY_PATH=~/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib
    src$ ./mod4
    Foo { s: "hello" }

哲学上_动态链接的问题,与Go一样. _只是当每6周有一个稳定版本时,不得不重新编译所有内容. 如果你有一个适合你的稳定版本,那么很酷. 随着Rust的稳定版本越来越多地被OS包管理器交付,动态链接将变得更加流行. 货物

## 与Java或Python相比,Rust标准库不是很大,

虽然功能比C或C ++更强大,但主要依赖于操作系统提供的库. 

但访问社区提供的库很简单[crates.io](https://crates.io)运用**货物**. Cargo将查找正确的版本并为您下载源代码,并确保下载其他所需的箱子. 

我们来创建一个需要阅读JSON的简单程序. 这种数据格式的使用非常广泛,但是对于包含在标准库中的数据格式太专业了. 所以我们初始化一个Cargo项目,使用'--bin',因为默认是创建一个库项目. 

    test$ cargo init --bin test-json
         Created binary (application) project
    test$ cd test-json
    test$ cat Cargo.toml
    [package]
    name = "test-json"
    version = "0.1.0"
    authors = ["Your Name <you@example.org>"]

    [dependencies]

使项目依赖于[JSON箱子](http://json.rs/doc/json/),编辑'Cargo.toml'文件,如下所示: 

    [dependencies]
    json="0.11.4"

然后用Cargo进行第一次构建: 

    test-json$ cargo build
        Updating registry `https://github.com/rust-lang/crates.io-index`
     Downloading json v0.11.4
       Compiling json v0.11.4
       Compiling test-json v0.1.0 (file:///home/steve/c/rust/test/test-json)
        Finished debug [unoptimized + debuginfo] target(s) in 1.75 secs

这个项目的主文件已经被创建 - 它是'src'目录中的'main.rs'. 它开始时只是一个'你好世界'的应用程序,所以让我们编辑它是一个适当的测试程序. 

请注意非常方便的'原始'字符串字面值 - 否则我们需要转义那些双引号并以丑为结尾: 

```rust
// test-json/src/main.rs
extern crate json;

fn main() {
    let doc = json::parse(r#"
    {
        "code": 200,
        "success": true,
        "payload": {
            "features": [
                "awesome",
                "easyAPI",
                "lowLearningCurve"
            ]
        }
    }
    "#).expect("parse failed");

    println!("debug {:?}", doc);
    println!("display {}", doc);
}
```

您现在只能编译和运行此项目`main.rs`已经改变. 

    test-json$ cargo run
       Compiling test-json v0.1.0 (file:///home/steve/c/rust/test/test-json)
        Finished debug [unoptimized + debuginfo] target(s) in 0.21 secs
         Running `target/debug/test-json`
    debug Object(Object { store: [("code", Number(Number { category: 1, exponent: 0, mantissa: 200 }),
     0, 1), ("success", Boolean(true), 0, 2), ("payload", Object(Object { store: [("features",
     Array([Short("awesome"), Short("easyAPI"), Short("lowLearningCurve")]), 0, 0)] }), 0, 0)] })
    display {"code":200,"success":true,"payload":{"features":["awesome","easyAPI","lowLearningCurve"]}}

调试输出显示了JSON文档的一些内部细节,但使用了一个普通的"{}"`显示`特质,从解析的文档重新生成JSON. 

我们来探索一下JSON API. 如果我们无法提取数值,这将毫无用处. 该`as_TYPE`方法返回`选项<类型>`因为我们无法确定该字段是否存在或是否属于正确类型.  (见[文档JsonValue](http://json.rs/doc/json/enum.JsonValue.html)) 

```rust
    let code = doc["code"].as_u32().unwrap_or(0);
    let success = doc["success"].as_bool().unwrap_or(false);

    assert_eq!(code, 200);
    assert_eq!(success, true);

    let features = &doc["payload"]["features"];
    for v in features.members() {
        println!("{}", v.as_str().unwrap()); // MIGHT explode
    }
    // awesome
    // easyAPI
    // lowLearningCurve
```

`特征`这里是一个参考`JsonValue`- 它必须是一个参考,因为否则我们会试图移动一个_值_脱离JSON文档. 这里我们知道它是一个数组,所以`成员 () `将返回一个非空的迭代器`&JsonValue`. 

如果"有效载荷"对象没有"功能"键,该怎么办?然后`特征`将被设置为`空值`. 不会有爆炸. 这种便利表达了JSON的自由形式,任何东西的本质. 如果结构不匹配,您应该检查收到的任何文档的结构并创建自己的错误. 

您可以修改这些结构. 如果我们有`让mut doc`那么这会做你所期望的: 

```rust
    let features = &mut doc["payload"]["features"];
    features.push("cargo!").expect("couldn't push");
```

该`推`将失败,如果`特征`不是一个数组,因此它返回`结果< () >`. 

这是一个非常漂亮的使用宏来生成_JSON文字_: 

```rust
    let data = object!{
        "name"    => "John Doe",
        "age"     => 30,
        "numbers" => array![10,53,553]
    };
    assert_eq!(
        data.dump(),
        r#"{"name":"John Doe","age":30,"numbers":[10,53,553]}"#
    );
```

为了这个工作,你需要显式地从JSON箱导入宏: 

```rust
#[macro_use]
extern crate json;
```

由于JSON的无定形,动态类型性质和Rust的结构化,静态性质之间的不匹配,使用这个箱子有一个缺点.  (自述明确提到'摩擦') 所以如果你_没有_想要将JSON映射到Rust数据结构,您最终会做很多检查,因为您不能认为接收到的结构与您的结构相匹配!为此,更好的解决方案是[serde_json](https://github.com/serde-rs/json)你在哪里_连载_Rust数据结构转换为JSON和_反序列化_JSON进入Rust. 

为此,请创建另一个货物二进制项目`货物新--bin测试 -  serde-json`,进入`测试SERDE,JSON`目录和编辑`Cargo.toml`. 像这样编辑它: 

    [dependencies]
    serde="0.9"
    serde_derive="0.9"
    serde_json="0.9"

并编辑`SRC / main.rs`是这样的: 

```rust
#[macro_use]
extern crate serde_derive;
extern crate serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u8,
    address: Address,
    phones: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Address {
    street: String,
    city: String,
}

fn main() {
    let data = r#" {
     "name": "John Doe", "age": 43,
     "address": {"street": "main", "city":"Downtown"},
     "phones":["27726550023"]
    } "#;
    let p: Person = serde_json::from_str(data).expect("deserialize error");
    println!("Please call {} at the number {}", p.name, p.phones[0]);

    println!("{:#?}",p);
}
```

你已经看到了`派生`属性之前,但是`serde_derive`箱子定义_自定义派生_为特别`连载`和`反序列化`特征. 结果显示生成的Rust结构体: 

    Please call John Doe at the number 27726550023
    Person {
        name: "John Doe",
        age: 43,
        address: Address {
            street: "main",
            city: "Downtown"
        },
        phones: [
            "27726550023"
        ]
    }

现在,如果你使用了`json`那么你需要几百行自定义转换代码,主要是错误处理. 单调乏味,容易搞砸,而不是你想要付出努力的地方. 

如果你从外部来源处理结构良好的JSON (如果需要,可以重新映射字段名称) ,这显然是最好的解决方案,并为Rust程序通过网络与其他程序共享数据提供了一个强大的方法 (因为一切都能理解JSON这些天) . 关于很酷的事情`serde` (用于SERialization DEserialization) 是也支持其他文件格式,例如`toml`,这是货运中常用的配置友好格式. 因此,您的程序可以将.toml文件读入结构中,并将这些结构编写为.json. 

序列化是一项重要的技术,Java和Go存在类似的解决方案 - 但有很大的不同. 在这些语言中,数据的结构可以在这里找到_运行_运用_反射_,但在这种情况下,序列化代码是在_编译时间_- 更高效!

Cargo被认为是Rust生态系统的一大优势,因为它为我们做了很多工作. 否则,我们不得不从Github下载这些库,构建为静态库箱,并将它们与程序链接. 这对于C ++项目来说是很痛苦的,如果货运不存在的话,对于Rust项目来说,这几乎是痛苦的. C ++在它的痛苦中有点独特,所以我们应该将它与其他语言的包管理器进行比较. npm (用于JavaScript) 和pip (用于Python) 为您管理依赖关系和下载,但分发故事更难,因为程序的用户需要安装NodeJS或Python. 但Rust程序与它们的依赖关系是静态链接的,所以它们可以在没有外部依赖的情况下再次发给你的好友. 

## 更多宝石

处理除简单文本以外的任何内容时,正则表达式使您的生活变得更加轻松. 这些通常适用于大多数语言,我将在这里假定对正则表示法有基本的了解. 使用[正则表达式](https://github.com/rust-lang/regex)把"regex ="0.2.1"'放在"[依赖]"在您的Cargo.toml. 

我们将再次使用"原始字符串",以便反斜杠不必转义. 在英语中,这个正则表达式"完全匹配两个数字,字符': ',然后是任意数字. 捕获两组数字": 成功的产出实际上有三个

```rust
extern crate regex;
use regex::Regex;

let re = Regex::new(r"(\d{2}):(\d+)").unwrap();
println!("{:?}", re.captures("  10:230"));
println!("{:?}", re.captures("[22:2]"));
println!("{:?}", re.captures("10:x23"));
// Some(Captures({0: Some("10:230"), 1: Some("10"), 2: Some("230")}))
// Some(Captures({0: Some("22:2"), 1: Some("22"), 2: Some("2")}))
// None
```

捕获_- 整场比赛,和两组数字. _这些正则表达式不是锚定_默认情况下如此_正则表达式**将追捕第一场比赛,跳过任何不匹配的东西. ** (如果你遗漏了' () ',它只会给我们整场比赛. ) 

这是可能的_名称_那些捕捉,并且将正则表达式分散在多行,甚至包括注释!编译正则表达式可能会失败 (第一个_期望_) 或者匹配可能失败 (第二个_期望_) . 在这里,我们可以使用结果作为关联数组并查找按名称捕获. 

```rust
let re = Regex::new(r"(?x)
(?P<year>\d{4})  # the year
-
(?P<month>\d{2}) # the month
-
(?P<day>\d{2})   # the day
").expect("bad regex");
let caps = re.captures("2010-03-14").expect("match failed");

assert_eq!("2010", &caps["year"]);
assert_eq!("03", &caps["month"]);
assert_eq!("14", &caps["day"]);
```

正则表达式可以分解符合模式的字符串,但不会检查它们是否有意义. 也就是说,你可以指定和匹配_句法_的ISO风格的日期,但_语义_他们可能是无稽之谈,比如"2014-24-52". 

为此,您需要专门的日期时间处理,由其提供[计时](https://github.com/lifthrasiir/rust-chrono). 你需要在做日期时决定一个时区: 

```rust
extern crate chrono;
use chrono::*;

fn main() {
    let date = Local.ymd(2010,3,14);
    println!("date was {}", date);
}
// date was 2010-03-14+02:00
```

但是,这不推荐,因为喂它不好的日期会导致恐慌! (尝试一个假日期来看看这个. ) 你需要的方法是ymd_opt`返回`LocalResult &lt;日期>`您还可以直接解析日期时间,无论是以标准UTC格式还是使用自定义`

```rust
    let date = Local.ymd_opt(2010,3,14);
    println!("date was {:?}", date);
    // date was Single(2010-03-14+02:00)

    let date = Local.ymd_opt(2014,24,52);
    println!("date was {:?}", date);
    // date was None
```

格式[. ](https://lifthrasiir.github.io/rust-chrono/chrono/format/strftime/index.html#specifiers)这些自我相同的格式允许您按照您想要的格式打印日期. 我特别强调了这两个有用的箱子,因为它们将成为大多数其他语言的标准库的一部分. 

事实上,这些箱子的胚胎形态曾经是Rust stdlib的一部分,但被切开了. _这是一个蓄意的决定: Rust团队非常重视stdlib的稳定性,所以一旦他们在不稳定的夜间版本中进行孵化,只有beta和stable才能保持稳定. 对于需要实验和改进的图书馆来说,他们保持独立并且能够跟踪货物会更好. 出于所有实际目的,这两个箱子是_标准 - 它们不会消失,并且可能会在某个时候折回到stdlib中. 
