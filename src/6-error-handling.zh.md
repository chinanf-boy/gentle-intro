
# 错误处理

## 基本的错误处理

如果你不能使用问号操作符来实现快乐,那么在生锈中的错误处理可能很笨拙,我们需要返回一个`Result`它可以接受任何错误. 所有错误都会实现这个 trait `std::error::error`, 所以 _任何_ 错误可以转换成一个`Box<Error>`. 

说我们 _都_ 需要处理 I/O 错误和从转换String到数字的错误: 

```rust
// box-error.rs
use std::fs::File;
use std::io::prelude::*;
use std::error::Error;

fn run(file: &str) -> Result<i32,Box<Error>> {
    let mut file = File::open(file)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents.trim().parse()?)
}
```

所以这是I/O错误的两个问号 (无法打开文件,或无法读取为String) 以及转换错误的一个问号. 最后,我们将结果包装在内`Ok`.rust可以从返回类型中得出结论`parse`应该转换为`i32`. 

很容易为此创建一个快捷方式`Result`类型: 

```rust
type BoxResult<T> = Result<T,Box<Error>>;
```

但是,我们的程序将具有特定于应用程序的错误条件,并且需要创建我们自己的错误类型. 基本要求很简单: 

-   可以 impl `Debug`
-   必须 impl `Display`
-   必须 impl `Error`

否则,你的错误可以做到它喜欢的东西. 

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

// a test function that returns our error result
fn raises_my_error(yes: bool) -> Result<(),MyError> {
    if yes {
        Err(MyError::new("borked"))
    } else {
        Ok(())
    }
}
```

打字`Result<T,MyError>`变得单调乏味,许多防锈模块定义它们自己的模块`Result`- 例如`IO::Result<T>`是简短的`Result<T,io::Error>`. 

在下一个例子中,当一个String不能被解析为一个浮点数时,我们需要处理特定的错误. 

现在这样`?`工作寻找从错误的转换 _表达_ 到必须的错误 _返回_. 并且这个转换由表达`From` trait. `Box<Error>`像它一样工作,因为它实现`From`为所有类型实施`Error`. 

此时您可以继续使用便捷的别名`BoxResult`并且赶上所有事情;会有我们的错误转化为`Box<Error>`这对小型应用程序来说是一个很好的选择. 但我想显示其他错误可以明确地与我们的错误类型合作. 

`ParseFloatError` impl `Error`, 所以`description()`被定义为. 

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

首先`?`是好的 (一种类型总是转换为自己的`From`) 和第二个`?`将转换`ParseFloatError`至`MyError`. 

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

不要太复杂,虽然有点啰嗦. 该繁琐的一点是不得不写`From`所有其他错误类型都需要进行转换`MyError`- 或者你只是依靠`Box<Error>`. 新手会因为多种方式在 rust 中做同样的事情而感到困惑;总是有另一种方法去剥离鳄梨 (或者如果你感觉嗜血,那么就去皮肤上) . 灵活性的代价有很多选择. 200行程序的错误处理可以比大型应用程序简单得多. 如果您想将您的宝贵物品作为 Cargo 打包,那么错误处理就变得至关重要. 

目前,问号运算符仅适用于`Result`,不`Option`,这是一个功能,而不是一个限制. `Option`有一个`ok_or_else`它将自己转换成一个`Result`例如,说我们有一个`HashMap`并且如果没有定义键则必须失败: 

```rust
    let val = map.get("my_key").ok_or_else(|| MyError::new("my_key not defined"))?;
```

现在这里返回的错误是完全清楚的! (该表单使用闭包,因此只有在查找失败时才会创建错误值. ) 

## simeple-error的简单错误

该[simple-error](https://docs.rs/simple-error/0.1.9/simple_error/)crate为你提供基于String的基本错误类型,正如我们在这里定义的那样,以及一些方便的宏. 就像任何错误一样,`Box<Error>`也可以正常工作: 

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

`bail!(s)`扩展到`return SimpleError::new(s).into();`- 尽早返回转换 _成_ 接收类型. 

你需要使用`BoxResult`用于混合`SimpleError`键入其他错误,因为我们无法执行`From`, 因为它的特点和类型都来自其他箱子. 

## 严重错误的error-chain

对于非平凡的应用程序有一个看点[error_chain](http://brson.github.io/2016/11/30/starting-with-error-chain)crate.

rust 的一个小宏魔法漫长路

创建一个二进制包`cargo new --bin test-error-chain`并到这个目录. 编辑`Cargo.toml`并添加`error-chain="0.8.1"`到最后. 

**error-chain** 为你做的是什么, 创建我们所需的所有定义的手动执行错误类型;创建一个结构体,并实现必要的 trait : `Display`,`Debug`和`Error`它也是默认的 impl `From` , 所以String可以转换成错误. 

我们的`src/main.rs`文件看起来像这样. 所有的主要程序都是运行`run`,打印出错误,并用非零退出代码结束程序. 宏`error_chain`生成所需的所有定义`Error`模块 - 在一个更大的程序中,你会把它放在它自己的文件中. 我们需要把所有东西都放进去`Error`回到全局范围,因为我们的代码将需要生成生成的 trait. 默认情况下,会有一个`Error`结构和一个`Result`用这个错误定义. 

这里我们也要求`From`要这样做`std::io::Error`将使用转换为我的错误类型`foreign_links`: 

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

'foreign_links'让我们的生活更轻松,因为问号操作员现在知道如何转换`std::io::Error`进入我们的`error::Error`.  (在引擎盖下,宏正在创建一个`From<std::io::Error>`转换,正如前面所述. ) 

所有的行动都发生在`run`;让我们打印出作为第一个程序参数给出的文件的前10行. 有可能也可能不会有这样的论点,这不一定是错误的. 这里我们要转换一个`Option<String>`变成一个`Result<String>`. 那里有两个`Option`做这种转换的方法,我选择了最简单的一种. 我们的`Error`类型的工具`From`对于`&str`,所以用一个简单的文本信息就很容易发生错误. 

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

 (再次) 有一个有用的小宏`bail!`为'抛出'错误. 替代方案`ok_or`方法可能是: 

```rust
    let file = match args().skip(1).next() {
        Some(s) => s,
        None => bail!("provide a file")
    };
```

喜欢`?`它做了一个 _提前返回_. 

返回的错误包含一个枚举`ErrorKind`,这使我们能够区分各种各样的错误. 总是有一个变体`Msg` (当你说`Error::from(str)`) 和`foreign_links`已宣布`Io`它包装 I/O 错误: 

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

添加新的错误很简单. 添加一个`Error`部分`error_chain!`宏: 

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

这定义了如何`Display`适用于这种新的错误. 现在我们可以更具体地处理'no argument'的错误,喂养`ErrorKind::NoArgument`一个`String`值: 

```rust
    let file = args().skip(1).next()
        .ok_or(ErrorKind::NoArgument("filename needed".to_string()))?;
```

现在有一个额外的`ErrorKind`您必须匹配的变体: 

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

一般来说,尽可能使错误尽可能具有特定的意义,_尤其_ 如果这是一个库函数!这种 match-on-kind 技术几乎相当于传统的异常处理,您可以匹配异常类型在`catch`要么`except`块. 

综上所述,**error-chain**创建一个类型`Error`, 和为你定义`Result<T>`成为`std::result::Result<T,Error>`. `Error`包含一个枚举`ErrorKind`并且默认情况下有一个变体`Msg`用于从String创建的错误. 你用来定义外部错误`foreign_links`这有两件事. 首先,它创建一个新的`ErrorKind`变种. 其次,它定义了`From`在这些外部错误,所以他们可以转换为无辜的. 新的错误变体可以很容易地添加. 许多恼人的样板代码被淘汰. 

## 链接错误

但这个 箱子{crate} 提供的非常酷的东西是 _error-chainning_. 

作为一个 _库用户_ ,当一个方法只是'抛出'一个通用的I/O错误时,这是​​令人烦恼的. 好吧,它不能打开一个文件,很好,但是什么文件?基本上,这个信息对我有什么用处?

`error_chain`作为 _error-chainning_ 这有助于解决过度通用错误的问题. 当我们尝试打开文件时,我们可以懒洋洋地依靠转`io::Error`而用`?`, 要么 _链{chain}_ 错误. 

```rust
// non-specific error
let f = File::open(&file)?;

// a specific chained error
let f = File::open(&file).chain_err(|| "unable to read the damn file")?;
```

这里是该程序的新版本 _没有_ 导入'foreign'错误,只是默认值: 

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

    ///////// chain explicitly! ///////////
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

        /////// look at the chain of errors... ///////
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

所以`chain_err`方法接受原始错误,并创建一个包含原始错误的新错误 - 这可以无限期地继续. 预计关闭将返回任何可能的值 _转换_ 进入错误. 

Rust 宏可以明显地为您节省大量的打字工作. `error-chain`甚至提供了一个取代整个主程序的捷径: 

```rust
quick_main!(run);
```

 (`run`无论如何,所有的行动都是在那里进行的. ) 
