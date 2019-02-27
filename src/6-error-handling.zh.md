# 错误处理

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [基本的错误处理](#%E5%9F%BA%E6%9C%AC%E7%9A%84%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)
- [simeple-error 的简单错误](#simeple-error-%E7%9A%84%E7%AE%80%E5%8D%95%E9%94%99%E8%AF%AF)
- [严重错误的 error-chain](#%E4%B8%A5%E9%87%8D%E9%94%99%E8%AF%AF%E7%9A%84-error-chain)
- [链接错误](#%E9%93%BE%E6%8E%A5%E9%94%99%E8%AF%AF)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 基本的错误处理

如果你不能使用问号操作符，那么在 Rust 中的错误处理会很笨拙。
为了这种实现的快乐，我们需要返回一个可以接受任何错误的`Result`。 所有错误都会实现`std::error::Error`trait，这样 _任何_ 错误都可以转换成一个`Box<Error>`。

说我们需要处理 I/O 错误和从 String 转换到数字的 _两种_ 错误:

```rust
// box-error.rs
use std::fs::File;
use std::io::prelude::*;
use std::error::Error;

fn run(file: &str) -> Result<i32,Box<Error>> {
    let mut file = File::open(file)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // Result<usize>
    Ok(contents.trim().parse()?)
}
```

所以，这给出了的两个问号，一个给 I/O 错误 (无法打开文件,或无法读取为 String) 以及转换错误一个。 最后,我们将结果包装在`Ok`内。Rust 可以根据返回类型签名，从`parse`得出应转换为`i32`。

很容易为`Result`类型创建一个简写:

```rust
type BoxResult<T> = Result<T,Box<Error>>;
```

但是，我们的程序将具有特定于应用程序的错误条件，还需要创建自己的错误类型。错误类型的基本要求也很简单:

- 可以 impl `Debug`
- 必须 impl `Display`
- 必须 impl `Error`

还有啊，你的错误可以做它喜欢做的事情。

```rust
// error1.rs
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct MyError {
    details: String
}

impl MyError {
    fn new(msg: &str) -> MyError {
        MyError{details: msg.to_string()}
    }
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f,"{}",self.details)
    }
}

impl Error for MyError {
    fn description(&self) -> &str {
        &self.details
    }
}

// 一个返回我们错误结果的测试函数
fn raises_my_error(yes: bool) -> Result<(),MyError> {
    if yes {
        Err(MyError::new("borked"))
    } else {
        Ok(())
    }
}
```

老输入`Result<T,MyError>`会乏味的，许多 Rust 模块会定义它们自己的`Result`- 例如`io::Result<T>`是`Result<T,io::Error>`的简写。

在下一个例子中，当一个 String 不能被解析为一个浮点数时，我们需要处理特定的错误。

现在`?`工作的方式，是从 _表达_ 的错误到必 _返回_ 的错误的一种转换。 并且这个转换由`From` trait 表示。`Box<Error>`一样是这样工作的，因为它为所有实现了`Error`的类型实现`From`。

此时您可以继续使用便捷的别名`BoxResult`，像以前一样 catch 所有事情; 会有一个我们的错误到`Box<Error>`的转换，这对小型应用程序来说是一个很好的选择。 但我想显示其他错误，明确与我们的错误类型的合作。

`ParseFloatError`实现了 `Error`, 所以`description()`方法可用。

```rust
use std::num::ParseFloatError;

impl From<ParseFloatError> for MyError {
    fn from(err: ParseFloatError) -> Self {
        MyError::new(err.description())
    }
}

// and test!
fn parse_f64(s: &str, yes: bool) -> Result<f64,MyError> {
    raises_my_error(yes)?;
    let x: f64 = s.parse()?;
    Ok(x)
}
```

第一个`?`还行 (一种类型总用`From`转换自己) 和第二个`?`将转换`ParseFloatError`到`MyError`。

结果如下:

```rust
fn main() {
    println!(" {:?}", parse_f64("42",false));
    println!(" {:?}", parse_f64("42",true));
    println!(" {:?}", parse_f64("?42",false));
}
//  Ok(42)
//  Err(MyError { details: "borked" })
//  Err(MyError { details: "invalid float literal" })
```

不会太复杂，就有点啰嗦。 该繁琐处是不得不为所有其他需要与`MyError`玩耍的错误类型，编写`From` - 或者简单点，依靠`Box<Error>`。 新手会因为多种方式在 Rust 中做同样的事情而感到困惑; 总是有另一种方法帮鳄梨削皮。代价有很多灵活选择。 200 行的错误处理程序可比大型应用程序简单得多。若您想将您的'宝贝'打包为一个 Cargo crate，那么错误处理就变得至关重要。

目前，问号运算符仅适用于`Result`，不是`Option`，这是一个功能，而不是一个限制。 `Option`有一个`ok_or_else`，该方法将自己转换成一个`Result`。例如说，我们有一个`HashMap`，若没有定义键的话，则必须失败:

```rust
    let val = map.get("my_key").ok_or_else(|| MyError::new("my_key not defined"))?;
```

现在这里返回的错误是很清楚的! (该形式 使用闭包，因此只有在查找失败时才会创建错误值。)

## 提供简单错误的 simeple-error

该[simple-error](https://docs.rs/simple-error/0.1.9/simple_error/)crate 为你提供基于一个字符串 的基本错误类型，正如我们在这里定义的那样，以及一些方便的宏。如同其他任何错误一样，`Box<Error>`也可以正常工作:

```rust
#[macro_use]
extern crate simple_error;

use std::error::Error;

type BoxResult<T> = Result<T,Box<Error>>;

fn run(s: &str) -> BoxResult<i32> {
    if s.len() == 0 {
        bail!("empty string");
    }
    Ok(s.trim().parse()?)
}

fn main() {
    println!("{:?}", run("23"));
    println!("{:?}", run("2x"));
    println!("{:?}", run(""));
}
// Ok(23)
// Err(ParseIntError { kind: InvalidDigit })
// Err(StringError("empty string"))
```

`bail!(s)`宏扩展为`return SimpleError::new(s).into();`- 提前返回转换 _成_ 接收的类型签名。

你需要使用`BoxResult`，混合`SimpleError`类型与其他错误，因为我们无法为它实现`From`, 因为它的 trait 和类型都来自其他箱子（安全问题)。

## 提供严重错误的 error-chain

非凡的应用程序，看过来[error_chain](http://brson.github.io/2016/11/30/starting-with-error-chain)crate。Rust 的一个小宏魔法的漫漫长路。

创建一个二进制包`cargo new --bin test-error-chain`，并进到这个目录。 编辑`Cargo.toml`，添加`error-chain="0.8.1"`到最后。

**error-chain** 为你做的是什么, 创建我们所需的所有定义的手动执行错误类型; 创建一个结构体，并实现必要的 trait : `Display`，`Debug`和`Error`，也默认实现 `From` ， 所以字符串 可以转换成错误。

我们的`src/main.rs`文件看起来像这样。所有的主要程序都是给`run`调用，打印出错误，并用非零退出代码结束程序。 `error_chain`宏，会在定义`error`的模块里面生成所有所需的 - 在一个更大的程序中，你会把`error`的模块放在它自己的文件中。 我们需要把放进`error`的所有东西，带回到全局作用域，因为我们的代码需要生成的 traits。 默认情况下，随带有一个`Error`结构和一个`Result`的定义。

我们也要求`From`的实现，这样使用`foreign_links`，`std::io::Error`才会转换为我的错误类型:

```rust
#[macro_use]
extern crate error_chain;

mod errors {
    error_chain!{
        foreign_links {
            Io(::std::io::Error);
        }
    }
}
use errors::*;

fn run() -> Result<()> {
    use std::fs::File;

    File::open("file")?;

    Ok(())
}


fn main() {
    if let Err(e) = run() {
        println!("error: {}", e);

        std::process::exit(1);
    }
}
// error: No such file or directory (os error 2)
```

'foreign_links'让我们的生活更轻松，因为问号符号现在知道如何转换`std::io::Error`进入我们的`error::Error`。 (在引擎盖下，宏正在创建一个`From<std::io::Error>`转换实现，正如前面所述。 )

所有的行动都发生在`run`;让我们打印出作为第一个程序参数给出的文件的前 10 行。 有可能或不会有这样的参数，这不一定是错误的。 这里我们要转换一个`Option<String>`到一个`Result<String>`。`Option`有两个做这种转换的方法，我选择了最简单的一种。 我们的`Error`类型为`&str`实现`From`，所以用一个简单的文本就可以很容易制作一个错误。

```rust
fn run() -> Result<()> {
    use std::env::args;
    use std::fs::File;
    use std::io::BufReader;
    use std::io::prelude::*;

    let file = args().skip(1).next()
        .ok_or(Error::from("provide a file"))?;

    let f = File::open(&file)?;
    let mut l = 0;
    for line in BufReader::new(f).lines() {
        let line = line?;
        println!("{}", line);
        l += 1;
        if l == 10 {
            break;
        }
    }

    Ok(())
}
```

(再次) 有一个有用的小宏`bail!`，用于'抛出'错误。`ok_or`方法的一个替代方案:

```rust
    let file = match args().skip(1).next() {
        Some(s) => s,
        None => bail!("provide a file")
    };
```

会像`?`一样，它 _提前返回_。

返回的错误包含一个`ErrorKind`枚举，这使我们能够区分各种各样的错误。 总有一个`Msg`变体 (当你用`Error::from(str)`) 和`foreign_links`申明包装 I/O 错误的`Io`:

```rust
fn main() {
    if let Err(e) = run() {
        match e.kind() {
            &ErrorKind::Msg(ref s) => println!("msg {}",s),
            &ErrorKind::Io(ref s) => println!("io {}",s),
        }
        std::process::exit(1);
    }
}
// $ cargo run
// msg provide a file
// $ cargo run foo
// io No such file or directory (os error 2)
```

添加新的错误很简单。 添加一个`Error`部分给`error_chain!`宏:

```rust
    error_chain!{
        foreign_links {
            Io(::std::io::Error);
        }

        errors {
            NoArgument(t: String) {
                display("no argument provided: '{}'", t)
            }
        }

    }
```

这定义了`Display`如何应用在这种新的错误。 现在我们可以更具体地处理'no argument'的错误，喂给`ErrorKind::NoArgument`一个`String`值:

```rust
    let file = args().skip(1).next()
        .ok_or(ErrorKind::NoArgument("filename needed".to_string()))?;
```

现在有一个额外的，您必须匹配的`ErrorKind`变体:

```rust
fn main() {
    if let Err(e) = run() {
        println!("error {}",e);
        match e.kind() {
            &ErrorKind::Msg(ref s) => println!("msg {}", s),
            &ErrorKind::Io(ref s) => println!("io {}", s),
            &ErrorKind::NoArgument(ref s) => println!("no argument {:?}", s),
        }
        std::process::exit(1);
    }
}
// cargo run
// error no argument provided: 'filename needed'
// no argument "filename needed"
```

一般来说，尽可能使错误尽可能具有特定的意义，_尤其_ 如果是一个库函数! 这种 match-on-kind 技术几乎相当于传统的异常处理，您可以在`catch`要么`except`块种匹配异常类型。

综上所述，**error-chain**为你创建一个类型`Error`，`std::result::Result<T,Error>`定义为`Result<T>`。 `Error`包含一个枚举`ErrorKind`，并且默认情况下有一个变体`Msg`用于从 String 创建的错误。 你用`foreign_links`来定义外部错误，这有两件事。首先，它创建一个新的`ErrorKind`变种。 其次,它在这些外部错误上实现了`From`，所以他们可以转换成我们的错误。新的错误变体很容易地添加。许多恼人的样板代码被淘汰。

## 错误的链化

但这个箱子提供的非常酷的东西是 _error-链化_.

作为一个 _用户_ ，当一个方法只是'抛出'一个通用的 I/O 错误时，这是烦人的。 好吧，它不能打开一个文件，很好，但这又是什么文件? 简单点来说，这个信息对我有什么用处?

`error_chain` 给出了 _error-链化_ 答案， 这有助于解决过度通用错误的问题。 当我们尝试打开文件时，我们可以懒洋洋地用`?`，看着它变成`io::Error`， 或者你可以选择 _链化_ 这错误。

```rust
// 普通错误
let f = File::open(&file)?;

// 一个特殊的错误链
let f = File::open(&file).chain_err(|| "unable to read the damn file")?;
```

这里是该程序的新版本， _没有_ 导入'foreign'错误，只是默认值:

```rust
#[macro_use]
extern crate error_chain;

mod errors {
    error_chain!{
    }

}
use errors::*;

fn run() -> Result<()> {
    use std::env::args;
    use std::fs::File;
    use std::io::BufReader;
    use std::io::prelude::*;

    let file = args().skip(1).next()
        .ok_or(Error::from("filename needed"))?;

    ///////// 显式链化! ///////////
    let f = File::open(&file).chain_err(|| "unable to read the damn file")?;

    let mut l = 0;
    for line in BufReader::new(f).lines() {
        let line = line.chain_err(|| "cannot read a line")?;
        println!("{}", line);
        l += 1;
        if l == 10 {
            break;
        }
    }

    Ok(())
}


fn main() {
    if let Err(e) = run() {
        println!("error {}", e);

        /////// 查看错误链... ///////
        for e in e.iter().skip(1) {
            println!("caused by: {}", e);
        }

        std::process::exit(1);
    }
}
// $ cargo run foo
// error unable to read the damn file
// caused by: No such file or directory (os error 2)
```

所以`chain_err`方法接受原始错误，并创建一个包含原始错误的新错误 - 这可以无限期地持续下去。 这个闭包函数期待那些能 _转换_ 为错误的值。

Rust 宏可以明显地为您节省大量的打字工作。 `error-chain`甚至提供了一个取代整个主程序的捷径:

```rust
quick_main!(run);
```

(`run`就是所有行动的地点，无需管其他。 )
