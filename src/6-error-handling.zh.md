
# 错误处理

## 基本的错误处理

如果你不能使用问号操作符来实现快乐,那么在生锈中的错误处理可能很笨拙,我们需要返回一个`结果`它可以接受任何错误. 所有错误都会实现这个特性`的std ::错误::错误`, 所以_任何_错误可以转换成一个`框<错误>`. 

说我们需要处理_都_从转换字符串到数字的I / O错误和错误: 

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

所以这是I / O错误的两个问号 (无法打开文件,或无法读取为字符串) 以及转换错误的一个问号. 最后,我们将结果包装在内`好`.rust可以从返回类型中得出结论`解析`应该转换为`i32`. 

很容易为此创建一个快捷方式`结果`类型: 

```rust
type BoxResult<T> = Result<T,Box<Error>>;
```

但是,我们的程序将具有特定于应用程序的错误条件,并且Sowe需要创建我们自己的错误类型. 基本要求很简单: 

-   可以实施`调试`
-   必须执行`显示`
-   必须执行`错误`

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

打字`结果<t时,myerror>`变得单调乏味,许多防锈模块定义它们自己的模块`结果`- 例如`IO ::结果<T>`是简短的`结果<t时,IO ::误差>`. 

在下一个例子中,当一个字符串不能被解析为一个浮点数时,我们需要处理特定的错误. 

现在这样`?`工作寻找从错误的转换_表达_到必须的错误_回_. 并且这个转换由表达`从`特征. `框<错误>`像它一样工作,因为它实现`从`为所有类型实施`错误`. 

此时您可以继续使用便捷的别名`BoxResult`并且赶上所有事情;会有我们的错误转化为`框<错误>`这对小型应用程序来说是一个很好的选择. 但我想显示其他错误可以明确地与我们的错误类型合作. 

`ParseFloatError`器物`错误`所以`描述 () `被定义为. 

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

首先`?`是好的 (一种类型总是转换为自己的`从`) 和第二个`?`将转换`ParseFloatError`至`MyError`. 

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

不要太复杂,虽然有点啰嗦. 该繁琐的一点是有towrite`从`所有其他错误类型都需要进行转换`MyError`- 或者你只是依靠`框<错误>`. 新手会因为多种方式在铁锈中做同样的事情而感到困惑;总是有另一种方法去剥离鳄梨 (或者如果你感觉嗜血,那么就去皮肤上) . 灵活性的代价有很多选择. 200行程序的错误处理可以比大型应用程序简单得多. 如果您想将您的宝贵物品作为货物箱打包,那么错误处理就变得至关重要. 

目前,问号运算符仅适用于`结果`,不`选项`,这是一个功能,而不是一个限制. `选项`有一个`ok_or_else`它将自己转换成一个`结果`例如,说我们有一个`HashMap中`并且如果没有定义键则必须失败: 

```rust
    let val = map.get("my_key").ok_or_else(|| MyError::new("my_key not defined"))?;
```

现在这里返回的错误是完全清楚的! (该表单使用闭包,因此只有在查找失败时才会创建错误值. ) 

## 简单错误的简单错误

该[简单的错误](https://docs.rs/simple-error/0.1.9/simple_error/)crate为你提供基于字符串的基本错误类型,正如我们在这里定义的那样,以及一些方便的宏. 就像任何错误一样,它可以正常工作`框<错误>`: 

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

`保释! (S) `扩展到`返回simpleerror :: new (s) .into () ;`- 尽早返回转换_成_接收类型. 

你需要使用`BoxResult`用于混合`SimpleError`键入其他错误,因为我们无法执行`从`因为它的特点和类型都来自其他箱子. 

## 严重错误的错误链

对于非平凡的应用程序有一个看点[error_chain](http://brson.github.io/2016/11/30/starting-with-error-chain)crate.a小宏魔法可以去锈很长的路...

创建一个二进制包`货物新--bin测试错误链`并改变到这个目录. 编辑`Cargo.toml`并添加`错误链="0.8.1"`到最后. 

什么**错误链**为你做的是创建我们手动执行错误类型所需的所有定义;创建一个结构体,并实现必要的特性: `显示`,`调试`和`错误`它也是默认的实现`从`所以字符串可以转换成错误. 

我们的第一`SRC / main.rs`文件看起来像这样. 所有的主要程序都是通话`跑`,打印出错误,并用非零退出代码结束程序. 宏`error_chain`生成所需的所有定义`错误`模块 - 在一个更大的程序中,你会把它放在它自己的文件中. 我们需要把所有东西都放进去`错误`回到全局范围,因为我们的代码将需要生成生成的特征. 默认情况下,会有一个`错误`结构和a`结果`用thaterror定义. 

这里我们也要求`从`要这样做`的std :: IO ::错误`将使用转换为我的错误类型`foreign_links`: 

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

'foreign_links'让我们的生活更轻松,因为问号操作员现在知道如何转换`的std :: IO ::错误`进入我们的`错误::错误`.  (在引擎盖下,宏正在创建一个`从<性病:: IO ::误差>`转换,正如前面所述. ) 

所有的行动都发生在`跑`;让我们打印出作为第一个程序参数给出的文件的前10行. 有可能也可能不会有这样的论点,这不一定是错误的. 这里我们要转换一个`选项<字符串>`变成一个`结果<字符串>`. 那里有两个`选项`做这种转换的方法,我选择了最简单的一种. 我们的`错误`类型的工具`从`对于`&STR`,所以用一个简单的文本信息就很容易发生错误. 

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

 (再次) 有一个有用的小宏`保释!`为'抛出'错误. 替代方案`ok_or`方法可能是: 

```rust
    let file = match args().skip(1).next() {
        Some(s) => s,
        None => bail!("provide a file")
    };
```

喜欢`?`它做了一个_提前回报_. 

返回的错误包含一个枚举`ErrorKind`,这使我们能够区分各种各样的错误. 总是有一个变体`味精` (当你说`错误::从 (STR) `) 和`foreign_links`已宣布`Io`它包装I / O错误: 

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

添加新的错误很简单. 添加一个`错误`部分`error_chain!`宏: 

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

这定义了如何`显示`适用于这种新的错误. 现在我们可以更具体地处理'无争论'的错误,喂养`errorkind :: noargument`一个`串`值: 

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

一般来说,尽可能使错误尽可能具有特定的意义,_尤其_如果这是一个库函数!这种match-on-kind技术几乎相当于传统的异常处理,您可以在a中匹配异常类型`抓住`要么`除`块. 

综上所述,**错误链**创建一个类型`错误`为你定义`导致<T>`成为`的std ::结果::结果<t时,误差>`. `错误`包含一个枚举`ErrorKind`并且默认情况下有一个变体`味精`用于从字符串创建的错误. 你用来定义外部错误`foreign_links`这有两件事. 首先,它创建一个新的`ErrorKind`变种. 其次,它定义了`从`在这些外部错误,所以他们可以转换为无辜的. 新的错误变体可以很容易地添加. 许多恼人的样板代码被淘汰. 

## 链接错误

但这个箱子提供的非常酷的东西是_错误链接_. 

作为一个_库用户_,当一个方法只是'抛出'一个通用的I / O错误时,这是​​令人烦恼的. 好吧,它不能打开一个文件,很好,但是什么文件?基本上,这个信息对我有什么用处?

`error_chain`不_错误链接_这有助于解决过度通用错误的问题. 当wetry打开文件时,我们可以懒洋洋地依靠转换`IO ::错误`运用`?`, 要么_链_错误. 

```rust
// non-specific error
let f = File::open(&file)?;

// a specific chained error
let f = File::open(&file).chain_err(|| "unable to read the damn file")?;
```

这里是该程序的新版本_没有_导入'外国'错误,只是默认值: 

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

所以`chain_err`方法接受原始错误,并创建一个包含原始错误的新错误 - 这可以无限期地继续. 预计关闭将返回任何可能的值_转换_进入错误. 

生锈的宏可以明显地为您节省大量的打字工作. `错误链`甚至提供了一个取代整个主程序的捷径: 

```rust
quick_main!(run);
```

 (`跑`无论如何,所有的行动都是在那里进行的. ) 
