# 模块和 Cargo

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [模块](#%E6%A8%A1%E5%9D%97)
- [Crates](#crates)
- [Cargo](#cargo)
- [更多宝石](#%E6%9B%B4%E5%A4%9A%E5%AE%9D%E7%9F%B3)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 模块

随着程序变得越来越大，有必要将它们分散到多个文件中，和将函数和类型放在不同的 _命名空间_。 这些问题的 Rust 解决方案就是 _模块_。

C 语言 吃了第一个螃蟹，而不是第二个，所以你最终会遇到类似`primitive_display_set_width`的可怕名字等等。实际上，文件名可以任意命名。

Rust 使用的全名看起来像`primitive::display::set_width`，之后可使用`use primitive::display`，这样就能用`display::set_width`代替。 你甚至可以说`use primitive::display::set_width`，然后只能用`set_width`，但这并不是一个好方式。 `rustc`虽然不会混淆，但是 _您_ 稍后可能会感到困惑。为了这个工作，文件名必须遵循一些简单的规则。

一个新的关键字`mod`，用于将模块定义为，可以写入 Rust 类型或函数的块:

```rust
mod foo {
    #[derive(Debug)]
    struct Foo {
        s: &'static str
    }
}

fn main(){
    let f = foo::Foo{s: "hello"};
    println!("{:?}", f);
}
```

但它仍不正确 - 我们得到'struct Foo is 私人{private}'。 为了解决这个问题，我们需要允许`Foo`导出的`pub`关键字。然后错误又变为'结构的 foo::Foo 字段是私人的'，再放了`pub`后, 能导出`Foo::s`。事情办好了。

```rust
    pub struct Foo {
        pub s: &'static str
    }
```

一个明确的`pub`，意味着你必须 _选择_ 哪些内容要通过模块公开。从模块导出的一组函数和类型称为它的 _接口{interface}_。

隐藏结构内部，通常会更好，并且只允许通过方法访问:

```rust
mod foo {
    #[derive(Debug)]
    pub struct Foo {
        s: &'static str
    }

    impl Foo {
        pub fn new(s: &'static str)-> Foo {
            Foo{s: s}
        }
    }
}

fn main(){
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

为什么隐藏 实现(impl) 是一件好事? 因为这意味着您可以在不中断接口，没有模块使用者太注意其细节的情况下稍后进行更改。 大规模编程的大敌是细节代码纠结的倾向，因此去理解一段一段代码，实际做了什么是不可能的。

在一个完美的世界里，一个模块做一件事，做好，并保持自己的秘密。

何时不要隐藏? 正如 Stroustrup 所说，当接口 _为_ 实现，就像`struct Point {x: f32,y: f32}`结构要导出。

一个模块 _中_ ，所有的项对所有的其他项都可见。 这是一个舒适的地方，每个人都可以成为朋友，知道彼此的私密细节。

每个人都可以根据自己的喜好，将程序分成不同的文件。我开始对 500 感到不舒服，那就 超过 2000 好了，随你喜欢(或有规定)。

那么如何将这个程序分解成单独的文件呢?

我们把这个`foo`代码到`foo.rs`:

```rust
// foo.rs
#[derive(Debug)]
pub struct Foo {
    s: &'static str
}

impl Foo {
    pub fn new(s: &'static str)-> Foo {
        Foo{s: s}
    }
}
```

并在主`main`程序中，_不_ 在一个`区块{}`内，使用一个`mod foo`声明,:

```rust
// mod3.rs
mod foo;

fn main(){
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

现在`rustc mod3.rs`也会引发`foo.rs`编译。 没有必要用 makefiles 来搞笑!

编译器也会看`MODNAME/mod.rs`，所以，如果我创建一个目录`boo`，其包含一个文件`mod.rs`，这也会工作:

```rust
// boo/mod.rs
pub fn answer()->u32 {
    42
}
```

现在主程序可以将两个模块作为单独的文件使用:

```rust
// mod3.rs
mod foo;
mod boo;

fn main() {
    let f = foo::Foo::new("hello");
    let res = boo::answer();
    println!("{:?} {}", f,res);
}
```

到目前为止，`mod3.rs`含有`main`，一个模块`foo.rs`和一个含`mod.rs`的目录`boo`。 通常的惯例是包含`main`的文件，就叫`main.rs`。

为什么有两种可做同样事情的方法? 因为`boo/mod.rs`，可让`boo`引用定义的其他模块，更新`boo/mod.rs`，并添加一个新模块 - 注意导出明确性。(若没有`pub`，`bar`只能看看在`boo`模块里面).

```rust
// boo/mod.rs
pub fn answer()->u32 {
	42
}

pub mod bar {
    pub fn question()-> &'static str {
        "the meaning of everything"
    }
}
```

然后，我们有了问题相对应的答案(`bar`模块在`boo`里面):

```rust
let q = boo::bar::question();
```

该模块部分可以被拉到`boo/bar.rs`:

```rust
// boo/bar.rs
pub fn question()-> &'static str {
    "the meaning of everything"
}
```

和`boo/mod.rs`变为:

```rust
// boo/mod.rs
pub fn answer()->u32 {
	42
}

pub mod bar;
```

总之，模块是关于组织和可见性的，这可能涉及或不涉及单独的文件。

请注意`use`与导入无关，只是指定模块名称的可见性。 例如:

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

重要的一点是，这里没有 _单独编译_ 说法。 主程序及其模块文件每次都要重新编译。也就是这样，较大的程序需要花费相当长(非常)的时间, 当然`rustc`的渐进式编译会越来越好。

## Crates

Rust 的"编译单位"是 _箱子{crate}_ ，它是一个可执行文件或一个库。

要分别编译上一节中的文件，请先构建`foo.rs`作为 rust _静态库_ 箱:

```bash
src$ rustc foo.rs --crate-type=lib
src$ ls -l libfoo.rlib
-rw-rw-r-- 1 steve steve 7888 Jan  5 13:35 libfoo.rlib
```

我们现在可以 _链接_ 这到我们的主要程序中:

```
src$ rustc mod4.rs --extern foo=libfoo.rlib
```

但，主要程序现在必须像这样，这个`extern`名称与链接时使用的名称相同。有一个隐式的顶级模块`foo`与 库 crate 相关联:

```rust
// mod4.rs
extern crate foo;

fn main(){
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

在人们开始欢呼'Cargo!Cargo!'之前，让我过一遍这个 Rust 构建的底层环境。我是'Know Thy Toolchain'的忠实信徒, 若我们从一开始就使用 Cargo 管理项目，会减少你需要学习的新魔法数量。模块是基本的语言功能，可用于 Cargo 项目之外。

现在该理解下，为什么 Rust 的二进制文件如此之大:

```
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 3,4M Jan  5 13:39 mod4
```

这很胖! 因 在该可执行文件中有 _许多_ 调试信息.

这不是一件坏事，如果你想调试，并当你的程序发生混乱时，实际上需要有意义的回溯。那么让我们去除这些调试信息，并查看:

```
src$ strip mod4
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 300K Jan  5 13:49 mod4
```

对如此简单的事情，尺寸仍感觉有点大，但是这个程序 _静态_ 链接 Rust 标准库。这是一件好事，因为您可以将此可执行文件交给任何具有正确操作系统的人 - 他们不需要"Rust 运行时"，就可以启用该文件。(还有，`rustup`甚至可以让你根据其他操作系统和平台 进行跨平台编译。 )

我们可以 动态 链接到 Rust 运行时，并获得真正的小:

```bash
src$ rustc -C prefer-dynamic mod4.rs --extern foo=libfoo.rlib
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 14K Jan  5 13:53 mod4
src$ ldd mod4
    linux-vdso.so.1 => (0x00007fffa8746000)
    libstd-b4054fae3db32020.so => not found
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6(0x00007f3cd47aa000)
    /lib64/ld-linux-x86-64.so.2(0x00007f3cd4d72000)
```

这'找不到 no found'是因为`rustup`不会全局安装动态库。 至少在 Unix 上 我们可以用我们的快乐方式破解(是的，我知道最好的解决方案是符号链接)。

```bash
src$ export LD_LIBRARY_PATH=~/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib
src$ ./mod4
Foo { s: "hello" }
```

Rust 没有动态链接的 _玄学_ 问题，与 Go 一样。 只是当每 6 周发布一个稳定版本时，不得不重新编译所有内容。 如果你有一个适合你的稳定版本，那么很酷。 随着 Rust 的稳定版本更新换代，越来越多地移交给 OS 包管理器控制, 动态链接将变得更加流行。

## Cargo

与 Java 或 Python 相比，Rust 标准库不是很大。虽然功能 比 C 或 C ++ 更强大，但主要依赖于操作系统提供的库。

但用 **Cargo** 访问[crates.io](https://crates.io)社区提供的库很简单。 Cargo 查找正确的版本，并为您下载源代码，并确保下载其他所需的 crate。

我们来创建一个需要 读取 JSON 的简单程序。 这种数据格式的使用非常广泛，但是对于包含在标准库中的数据格式太偏科了。下面我们展示下，我们初始化一个 Cargo 项目，可以不使用'--bin'，因为默认就是创建一个二进制项目。

```bash
test$ cargo init --bin test-json
        Created binary(application)project
test$ cd test-json
test$ cat Cargo.toml
[package]
name = "test-json"
version = "0.1.0"
authors = ["Your Name <you@example.org>"]

[dependencies]
```

让项目依赖[JSON crate](http://json.rs/doc/json/)，编辑'Cargo.toml'文件,如下所示:

```toml
[dependencies]
json="0.11.4"
```

然后用 Cargo 进行第一次构建:

```bash
test-json$ cargo build
Updating registry `https://github.com/rust-lang/crates.io-index`
Downloading json v0.11.4
Compiling json v0.11.4
Compiling test-json v0.1.0(file:///home/steve/c/rust/test/test-json)
Finished debug [unoptimized + debuginfo] target(s)in 1.75 secs
```

在用 Cargo 初始化这个项目的时候，主文件已经被 _创建_ , 它是'src'目录中的'main.rs'。 开始时，只是一个'你好世界'的应用程序，现在让它变成一个适当的测试程序。

请注意，非常方便的'原始{raw}'字符串字面量的使用 - 否则我们需要转义那些双引号，一段丑陋的格式：

```rust
// test-json/src/main.rs
extern crate json;

fn main(){
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

`main.rs`改好后，您现在编译和运行此项目.

```bash
test-json$ cargo run
    Compiling test-json v0.1.0(file:///home/steve/c/rust/test/test-json)
    Finished debug [unoptimized + debuginfo] target(s)in 0.21 secs
        Running `target/debug/test-json`
debug Object(Object { store: [("code", Number(Number { category: 1, exponent: 0, mantissa: 200 }),
    0, 1),("success", Boolean(true), 0, 2),("payload", Object(Object { store: [("features",
    Array([Short("awesome"), Short("easyAPI"), Short("lowLearningCurve")]), 0, 0)] }), 0, 0)] })
display {"code":200,"success":true,"payload":{"features":["awesome","easyAPI","lowLearningCurve"]}}
```

调试(debug)输出了 JSON 文档的一些内部细节，而用，一个普通的"{}"，使用了`Display` trait，从解析的文档重生成 JSON。

我们来探索一下 JSON API。 如果我们无法提取数值，这将毫无用处。 该`as_TYPE`方法会返回`Option<TYPE>`, 因为我们无法确定该字段是否存在或是否属于正确类型。 (见[ JsonValue 的文档](http://json.rs/doc/json/enum.JsonValue.html))

```rust
    let code = doc["code"].as_u32().unwrap_or(0);
    let success = doc["success"].as_bool().unwrap_or(false);

    assert_eq!(code, 200);
    assert_eq!(success, true);

    let features = &doc["payload"]["features"];
    for v in features.members(){
        println!("{}", v.as_str().unwrap()); // MIGHT explode
    }
    // awesome
    // easyAPI
    // lowLearningCurve
```

`features`这里是一个`JsonValue`引用 - 它必须是一个引用，否则我们会试图移动一个 _值_ ，这会脱离 JSON。这里我们知道它是一个数组，所以`members()`将返回一个非空的`&JsonValue`迭代器。

如果"payload"对象没有"features"键，该怎么办? 那么`features`将被设置为`Null`。 不会有爆炸。 这种便利表达了 JSON 的自由表达任何东西的本质。 如果结构不匹配，您应该检查收到的任何文档结构，并创建自己的错误。

如果我们有`let mut doc`，您可以修改这些结构。记得加上 expect:

```rust
    let features = &mut doc["payload"]["features"];
    features.push("cargo!").expect("couldn't push");
```

如果`feature`不是一个数组，该`push`将失败，因此它 panic。

使用一个宏，来生成 _JSON 字面量_，漂亮:

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

为了这个宏工作，你需要显式地从 JSON 箱导入宏 :

```rust
#[macro_use]
extern crate json;
```

由于 JSON 的无定形，动态性质 和 Rust 的结构化，静态性质之间的不匹配，使用这个 crate 有一个缺点。 (readme 明确提到'有摩擦{friction}')，所以如果你 _确_ 要将 JSON 映射到 Rust 数据结构，您最终会做很多检查，因为您不能认为接收到的结构与您的结构相匹配! 为此，更好的解决方案是[serde_json](https://github.com/serde-rs/json)， 它可以将 Rust 数据结构 _序列化_ 为 JSON ，和 JSON _反序列化_ 到 Rust。

为此，请创建另一个 Cargo 二进制项目`Cargo new --bin test-serde-json`，进入`test-serde-json`目录和编辑`Cargo.toml`。 像这样编辑它:

```toml
[dependencies]
serde="0.9"
serde_derive="0.9"
serde_json="0.9"
```

并编辑`src/main.rs`:

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

fn main(){
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

你之前已经看到了`derive`属性，但是`serde_derive` crate 为特有的`Serialize`和`Deserialize` trait ，定义了 _自定义派生{custom derives}_。生成的 Rust 结构体结果:

```json
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
```

现在，如果你使用了`json`，那么你需要几百行的自定义转换代码，主要是错误处理。 单调乏味，容易搞砸，这些都不是你想要付出努力的地方。

如果，你想从外部来源处理结构良好的 JSON (如果需要，可以重新映射字段名称)，`serde`显然是最好的解决方案，并为 Rust 程序通过网络与其他程序共享数据提供了一个强大的方法(因为如今一切都能理解 JSON)。 关于`serde`很酷的事情(名字来源于，SERialization:序列化 DEserialization: 反序列化 的 大写字母)是支持其他文件格式，例如`toml`，这是 cargo 中常用的配置友好格式。 因此，您的程序可以将 `.toml` 文件读入结构中，并将这些结构编写为`.json`。

序列化是一项重要的技术，Java 和 Go 存在类似的解决方案 ，但有很大的不同。 在这些语言中，数据的结构可以在 _运行时_ 运用 _反射_ 找到，但现这情况，序列化代码是在 _编译时_- 更高效!

Cargo 被认为是 Rust 生态系统的一大优势，因为它为我们做了很多工作。 否则，我们不得不从 Github 下载这些库，构建为 静态库-crate ，并将它们与程序链接。 这对于 C ++ 项目来说是很痛苦的，如果 Cargo 不存在的话，Rust 项目相当于痛苦 C++ 本身。 C ++ 的痛苦中带点独特，所以我们应该将它与其他语言的包管理器进行比较。 npm(用于 JavaScript) 和 pip(用于 Python) 为您管理依赖关系和下载， 但分发流程更难，因为程序的用户需要安装 NodeJS 或 Python。 但 Rust 程序与它们的 依赖关系 是静态链接的，所以它们可以在没有外部依赖的情况下，再次发给你的好友。

## 更多的宝藏

处理除简单文本以外的任何内容时，正则表达式使您的生活变得更加轻松。 这通常适用于大多数语言，在这里假定你对正则表示法有基本的了解。 使用[正则表达式](https://github.com/rust-lang/regex), 把"regex ="0.2.1"'放在"[dependencies]"在您的 Cargo.toml。

我们将再次使用"raw 字符串"，以便反斜杠不必转义。 在中文，这个正则表达式意思是 "完全匹配两个数字，后接字符':'，再是任意数字。共捕获两组数字":

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

成功的产出实际上有三个 _捕获_ 项 - 全匹配，和两组数字。 默认情况下这些正则表达式不是 _确定的_ ， 所以 _正则表达式_ 将捕第一个出现的匹配，跳过任何不匹配的东西。 (如果你遗漏了'()'，它只会给我们全匹配。 )

可以 _命名_ 那些捕捉项，并且将正则表达式分散在多行，甚至包括注释! 编译正则表达式可能会失败(第一个 _expect_)或者匹配可能失败(第二个 _expect_)。 在这里，我们可以使用结果作为关联数组，并按名称查找。

```rust
let re = Regex::new(r"(?x)
(?P<year>\d{4}) # the year
-
(?P<month>\d{2})# the month
-
(?P<day>\d{2})  # the day
").expect("bad regex");
let caps = re.captures("2010-03-14").expect("match failed");

assert_eq!("2010", &caps["year"]);
assert_eq!("03", &caps["month"]);
assert_eq!("14", &caps["day"]);
```

正则表达式可以分解符合模式的字符串，但不会检查它们是否有意义。 也就是说，你可以指定和匹配的 ISO _语法_ 风格的日期，但 _语义_ 可能是无稽之谈，比如"2014-24-52"。

为此，您需要专门的日期时间处理，由[计时 chrono](https://github.com/lifthrasiir/rust-chrono)提供。 你或需要做日期时，决定一个时区:

```rust
extern crate chrono;
use chrono::*;

fn main(){
    let date = Local.ymd(2010,3,14);
    println!("date was {}", date);
}
// date was 2010-03-14+02:00
```

但是，这不推荐，因为喂它不好的日期会导致恐慌!(尝试一个假日期)你需要的方法是 `ymd_opt`，其返回`LocalResult<Date>`。

```rust
    let date = Local.ymd_opt(2010,3,14);
    println!("date was {:?}", date);
    // date was Single(2010-03-14+02:00)

    let date = Local.ymd_opt(2014,24,52);
    println!("date was {:?}", date);
    // date was None
```

您还可以直接解析日期时间，无论是以 标准 UTC 格式 还是 使用自定义[格式{formats}](https://lifthrasiir.github.io/rust-chrono/chrono/format/strftime/index.html#specifiers) 这些完全相同的的格式允许您， 按照想要的格式打印日期。 我特别强调了这两个有用的 crate ，因为它们将成为大多数其他语言的标准库的一部分。

事实上，这些 crate 的胚胎形态曾经是 Rust stdlib 的一部分，但被切开了。这是个有意的决定: Rust 团队非常重视 stdlib 的稳定性，所以只有在不稳定的夜间版本诞生，而后活过 beta 和 stable 的功能才能保持稳定。 对于需要实验和改进的 库 来说，他们保持独立，并且 Cargo 能够跟踪会更好。 出于所有实际原因，这两个 crate 会是 _标准_ ，它们不会消失，并且可能会在某个时候折回到 stdlib 中。
