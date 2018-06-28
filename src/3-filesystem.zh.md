
# 文件系统和进程

## 再看看读取 Files

在 [第1部分的末尾](http://chinanf-boy.github.io/gentle-intro/1-basics.zh.html#a%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6),我展示了如何读取整个文件到字符串. 自然,这并不总是一个好主意,所以这里是如何逐行读取文件. 

`fs::File` impl{实现} 了`io::Read`,这是一个可读性的  trait  . 这个 trait 定义了一个能填补`u8`切片的字节`read`方法 - 这是唯一的 _要求_  trait 的方法,你会得到一些 _提供_ 免费的方法,很像`Iterator`. 您可以使用`read_to_end`填 充可读的内容 到 字节 Vec, 还有`read_to_string`可以填充一个 string - 必须是 utf-8 编码. 

这是一个'原始{raw}'阅读,没有缓冲. 对于缓冲阅读, 我们有`io::BufRead` trait ,给出 `read_line` 和一个`lines`迭代器{Iterator}. `io::BufReader`将给 _任何_ 可读性 提供`io::BufRead`实现. 

`fs::File` _也_ impl `io::Write`. 

确保所有这些 traits 可用的最简单方法是`use std::io::prelude::*`. 

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

fn read_all_lines(filename: &str) -> io::Result<()> {
    let file = File::open(&filename)?;

    let reader = io::BufReader::new(file);

    for line in reader.lines() {
        let line = line?;
        println!("{}", line);
    }
    Ok(())
}
```

该`let line = line?`可能看起来有点奇怪. 该`line`返回的迭代器实际上是一个`io::Result<String>`我们用`?`解开它. 因为事情 _能够_ 在迭代过程中出现错误 如: I/O错误,吞噬不是 utf-8 的字节块,等等. 

`line`作为一个迭代器,可以直接使用`collect`从一个文件读入一个字符串向量,或者用`enumerate`迭代器{Iterator}打印带行号的 line. 

然而,这并不是读取 所有行 的最有效方式,因为每行都分配了一个新字符串. 使用`read_line`效率更高,虽然更尴尬. 请注意,返回的行包含换行符,可以使用`trim_right`进行移除. 

```rust
    let mut reader = io::BufReader::new(file);
    let mut buf = String::new();
    while reader.read_line(&mut buf)? > 0 {
        {
            let line = buf.trim_right();
            println!("{}", line);
        }
        buf.clear();
    }
```

这导致分配少得多,因为 _clearing{清除}_ 该字符串不释放其分配的内存;一旦字符串有足够的容量,不会再有分配. 

这是我们使用一个`块{}`来控制借用的情况之一. `line`是借用的`buf`,这个借用必须在我们修改之前完成`buf`. Rust 再一次试图阻止我们做一些愚蠢的事情,那就是访问`line` _后_ 我们已经清除了缓冲区.  (借阅检查者 有时可能会受到限制,rust 是由于"非词汇生命周期{non-lexical  lifetimes}",它会分析代码并看到`buf.clear()`之后`line`不使用. ) 

这不是很漂亮. 我不能给你一个适当的迭代器,它返回缓冲区的引用,但我可以给你一些 *看起来像* 一个迭代器的东西.


首先定义一个通用结构体; 类型参数`R`是'任意实现 Read 的类型'. `它包含读者{reader} 和我们要借用的 缓冲区{buffer}. 

```rust
// file5.rs
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Lines<R> {
    reader: io::BufReader<R>,
    buf: String
}

impl <R: Read> Lines<R> {
    fn new(r: R) -> Lines<R> {
        Lines{reader: io::BufReader::new(r), buf: String::new()}
    }
    ...
}
```

然后`next`方法. 它返回一个`Option`- 就像迭代器,当它返回`None`时迭代器{Iterator}结束. 返回的类型是一个`Result`是因为`read_line`可能会失败, 我们永远不要丢失错误 . 所以如果失败了,我们把它的错误包括进去`Some<Result>`. 否则,它可能读取了零字节,这是文件的自然结束 - 不是错误,只是一个`None`. 

此时,缓冲区包含附有换行符 (`\n`) 的行. 修剪掉它,然后打包字符串片. 

```rust
    fn next<'a>(&'a mut self) -> Option<io::Result<&'a str>>{
        self.buf.clear();
        match self.reader.read_line(&mut self.buf) {
            Ok(nbytes) => if nbytes == 0 {
                None // no more lines!
            } else {
                let line = self.buf.trim_right();
                Some(Ok(line))
            },
            Err(e) => Some(Err(e))
        }
    }
```

现在,请注意 生命周期 如何工作. 我们需要明确的 lifetime ,因为 Rust 永远不会让我们在不知道他们的 lifetime 的情况下发放借用的字符串片. 在这里,我们说这个借用的字符串的 lifetime 在`self` lifetime 中. 

而且 lifetime 的这个签名不兼容`迭代器{Iterator}`的接口. 但是如果兼容就很容易出现问题; 考虑`collect`试图制作这些字符串切片的 Vec . 这是不可能的,因为它们都是从同一个可变字符串中借用的! (如果您已 read _所有_ 文件转换为字符串,然后是字符串`line`迭代器{Iterator}可以返回字符串切片,因为它们都是借用原始字符串的 _不同_ 部分) . 

由此产生的循环更清晰,文件缓冲对用户是不可见的. 

```rust
fn read_all_lines(filename: &str) -> io::Result<()> {
    let file = File::open(&filename)?;

    let mut lines = Lines::new(file);
    while let Some(line) = lines.next() {
        let line = line?;
        println!("{}", line);
    }

    Ok(())
}
```

你甚至可以这样写循环,字符串切片拉出来 显式匹配 : 

```rust
    while let Some(Ok(line)) = lines.next() {
        println!("{}", line)?;
    }
```

这很诱人,但你在这里抛出一个可能的错误;每当发生错误时,此循环都会静静地停止. 特别是,它将停止在 Rust 无法将 line 转换为 utf-8 的第一个位置. 适合休闲代码,不适合生产代码!

## 写入到Files

我们遇到了`write!`宏当遇到`Debug`实现- 它也适用于任何实现`Write`的东西. 所以这是另一种`print!`: 

```rust
    let mut stdout = io::stdout();
    ...
    write!(stdout,"answer is {}\n", 42).expect("write failed");
```

如果有 _可能的_ 错误,你必须处理它. 它可能不是很 _容易_ 但它可能发生. 它通常很好,因为如果你正在做文件 I/O 你应该在一个上下文中加入`?`. 

但有一个区别: `print!`为每个写入锁定标准输出. 这通常是您想要输出的内容,因为没有锁定 多线程程序 可能会 以有趣的方式 混淆输出. 但是,如果你 抽出大量文字,那么`write!`将会变得更快. 

对于任意文件我们用到`write!`. 在 `write_out`的结尾, `out`会扔掉文件自然关闭

```rust
// file6.rs
use std::fs::File;
use std::io;
use std::io::prelude::*;

fn write_out(f: &str) -> io::Result<()> {
    let mut out = File::create(f)?;
    write!(out,"answer is {}\n", 42)?;
    Ok(())
}

fn main() {
  write_out("test.txt").expect("write failed");
}
```

如果你关心性能,你需要知道 Rust 文件默认是无缓冲的. 所以每个小的写入请求都会直接进入操作系统,而且这将会明显变慢. 我提到了这一点,因为这种默认设置与其他编程语言不同,并且可能导致令人震惊的发现: Rust 可能被脚本语言遗留下来的地方! 就像`Read`和`io::BufReader`, `io::BufWriter`是带缓冲的`Write`. 

## 文件,路径和目录

这是一个用于在机器上打印 Cargo 目录的小程序. 最简单的情况是它是'〜/.cargo'. 这是一个Unix shell扩展,所以我们使用`env::home_dir`因为它是跨平台的.  (它可能会失败,但没有主目录的计算机, 无论如何都不会托管 Rust 工具. ) 

这里我们创建一个[PathBuf](https://doc.rust-lang.org/std/ops/trait.Mul.html)并使用它的`push`方法构建完整的文件路径就像 _组件_.  (这比 用'/','\\'或其他任何东西来要容易得多,取决于系统. ) 

```rust
// file7.rs
use std::env;
use std::path::PathBuf;

fn main() {
    let home = env::home_dir().expect("no home!");
    let mut path = PathBuf::new();
    path.push(home);
    path.push(".cargo");

    if path.is_dir() {
        println!("{}", path.display());
    }
}
```

一个`PathBuf`就好像`String`- 它拥有一组可扩展的角色,但具有专门用于构建路径的方法. 但其大部分功能都来自借用版本`Path`,这就像`&str`. 所以,例如,`is_dir`是一个`Path`方法. 

这可能听起来像一种继承形式,但魔法[Deref](https://doc.rust-lang.org/book/deref-coercions.html) trait 的工作方式不同. 它就像`String/&str` - `PathBuf`引用可 _包裹{Coerce}_ 成为`Path`引用.  ('Coerce'是一个很强的词,但这确实是 Rust 为你做转换的少数几个地方之一. ) 

```rust
fn foo(p: &Path) {...}
...
let path = PathBuf::from(home);
foo(&path);
```

`PathBuf`与`OsString`有亲密的关系,它代表我们直接从系统获得的字符串.  (有一个相应的`OsString/&OsStr`关系. ) 

这样的字符串不是 _保证_ 可以表示为 utf-8 !现实生活是一个[复杂的事情](https://news.ycombinator.com/item?id=10519932),特别是看到'他们为什么这么辛苦'的答案. 总而言之,首先有几年的 ASCII 传统编码,以及其他语言的多种特殊编码. 其次,人类语言很复杂. 例如'noël'是 _五个_ Unicode代码点!

确实,现代操作系统文件名的大部分时间都是 Unicode (Unix方面的 utf-8 ,Windows方面的UTF-16) ,又或者不是. Rust必须严格处理这种可能性. 例如,`Path`有一个方法`as_os_str`它返回一个`&OsStr`, 但是`to_str`方法返回一个`Option<&str>`. 并不总是可能!

人们在这一点上遇到了麻烦,因为他们已经过分依赖'string'和'character'作为唯一必要的抽象. 正如爱因斯坦所说, 编程语言必须尽可能简单,但并不简单. 系统语言 _需要_ 一个`String/&str`区别 (拥有与借用: 这也非常方便) ,如果它希望在 Unicode 字符串上标准化,那么它需要另一种类型来处理无效 Unicode 的文本 - 因此有了`OsString/&OsStr`. 请注意,这些类型没有任何有趣的 类似字符串的方法, 因为我们本来就不知道无效 Unicode. 

但是,人们习惯于像处理字符串一样处理文件名,这就是 Rust 使用`PathBuf`方法操作文件路径更容易的原因. 

您可以`pop`连续去除路径组件. 这里我们从程序的当前目录开始: 

```rust
// file8.rs
use std::env;

fn main() {
    let mut path = env::current_dir().expect("can't access current dir");
    loop {
        println!("{}", path.display());
        if ! path.pop() {
            break;
        }
    }
}
// /home/steve/rust/gentle-intro/code
// /home/steve/rust/gentle-intro
// /home/steve/rust
// /home/steve
// /home
// /
```

这是一个有用的变化. 我有一个搜索 配置{config} 文件 的程序,其规则是它可能出现在当前目录的任何子目录中. 所以我创建`/home/steve/rust/config.txt`并 启动此程序`/home/steve/rust/gentle-intro/code`: 

```rust
// file9.rs
use std::env;

fn main() {
    let mut path = env::current_dir().expect("can't access current dir");
    loop {
        path.push("config.txt");
        if path.is_file() {
            println!("gotcha {}", path.display());
            break;
        } else {
            path.pop();
        }
        if ! path.pop() {
            break;
        }
    }
}
// gotcha /home/steve/rust/config.txt
```

如此像 **git** 的工作方式当它想知道当前的回购是什么.

有关文件的详细信息 (其大小,类型等) 被称为它的 _元数据_. 与往常一样,可能存在错误 - 不仅仅是"找不到",而且如果我们没有权限读取此文件. 

```rust
// file10.rs
use std::env;
use std::path::Path;

fn main() {
    let file = env::args().skip(1).next().unwrap_or("file10.rs".to_string());
    let path = Path::new(&file);
    match path.metadata() {
        Ok(data) => {
            println!("type {:?}", data.file_type());
            println!("len {}", data.len());
            println!("perm {:?}", data.permissions());
            println!("modified {:?}", data.modified());
        },
        Err(e) => println!("error {:?}", e)
    }
}
// type FileType(FileType { mode: 33204 })
// len 488
// perm Permissions(FilePermissions { mode: 436 })
// modified Ok(SystemTime { tv_sec: 1483866529, tv_nsec: 600495644 })
```

文件的长度 (以字节为单位) 和修改时间很容易解释.  (注意我们可能无法获得这个时间!) 文件类型有方法`is_dir`,`is_file`和`is_symlink`. 

`权限{perissions}`是一个有趣的. Rust 努力成为跨平台的,所以这是'最低公分母'的例子. 一般来说,你可以查询的只是文件是否只读 - '权限'概念在Unix中被扩展,并为 `用户/群组/其他` 提供 `读/写/可执行`的权限. 

但是,如果您对 Windows 不感兴趣,那么引入特定于平台的 traits 将至少为我们提供权限模式位.  (像往常一样,一个 trait 只有在它可见时才会触发. ) 然后,应用到程序它自己的能被赋予可执行: 

```rust
use std::os::unix::fs::PermissionsExt;
...
println!("perm {:o}",data.permissions().mode());
// perm 755
```

 (注意"{:o}"用于打印 _八进制_) 

 (Windows 上的文件是否可执行取决于其扩展名. 可执行文件的扩展名可以在PATHEXT`环境变量 - '.exe','. bat'等等) . 
 
 `std::fs`包含许多用于处理文件的有用功能,例如复制或移动文件,制作符号链接和创建目录. 
 
 要查找目录的内容,`std::fs::read_dir`提供了一个迭代器. 以下是扩展名为".rs"且大小大于1024字节 的所有文件: 

```rust
fn dump_dir(dir: &str) -> io::Result<()> {
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let data = entry.metadata()?;
        let path = entry.path();
        if data.is_file() {
            if let Some(ex) = path.extension() {
                if ex == "rs" && data.len() > 1024 {
                    println!("{} length {}", path.display(),data.len());
                }
            }
        }
    }
    Ok(())
}
// ./enum4.rs length 2401
// ./struct7.rs length 1151
// ./sexpr.rs length 7483
// ./struct6.rs length 1359
// ./new-sexpr.rs length 7719
```

明显,`read_dir`可能会失败 (通常是'找不到'或'没有权限') ,但是获取每个新条目可能会失败 (这就像是`line`迭代器{Iterator} 遍历 缓冲的阅读器的内容) . 另外,我们可能无法获取与条目对应的元数据. 一个文件可能没有扩展名,所以我们也必须检查. 

为什么不只是一个遍历路径的迭代器? 在 Unix 上这是这样的`opendir`系统调用起作用,但在 Windows 上,您无法在不获取元数据的情况下 迭代目录的内容. 所以这是一个相当优雅的妥协方案,它允许跨平台的代码尽可能高效. 

在这一点上你可以原谅感觉'错误疲劳{error fatigue}'. 但请注意 _错误总是存在_ - 这不是 Rust 正在发明新的. 它只是在努力让你无法忽视它们. 任何操作系统调用都可能失败. 

Java 和 Python 等语言引发异常;像 Go 和 Lua 这样的语言返回两个值,其中第一个是 Resul t,第二个是 错误: 像 Rust 也一样,它考虑到是库函数引发错误的不良方式. 所以有很多错误检查和函数的早期返回. 

Rust使用`Result`因为它 either-or{没有或者}: 你不能同时得到 Result 和 错误. 问号运算符 使处理错误更加清晰. 

## 进程

一个基本的需求是程序运行程序,或者 _启动过程_ . 你的程序可以 _启动{launch}_ 尽可能多的子进程,顾名思义他们与父母有特殊的关系. 

运行程序很简单,使用`Command`struct,它构建传递给程序的参数: 

```rust
use std::process::Command;

fn main() {
    let status = Command::new("rustc")
        .arg("-V")
        .status()
        .expect("no rustc?");

    println!("cool {} code {}", status.success(), status.code().unwrap());
}
// rustc 1.15.0-nightly (8f02c429a 2016-12-15)
// cool true code 0
```

所以`new`收到该程序的名称 (它将查找`PATH`,如果不是绝对文件名) ,`arg`增加了一个新的 _argument_ ,并且`status`导致它运行. 这返回一个`Result`,这是`Ok`如果程序实际运行,包含一个`退出状态{ExitStatus}`. 在这种情况下,程序成功,并返回 退出码0 (使用`unwrap`是因为如果程序被信号杀死了,我们不总是得到代码) . 

如果我们改变了`-V`至`-v` (一个容易的错误) `rustc`失败: `所以有三种可能性: 

    error: no input filename given

    cool false code 101

程序不存在,很糟糕,或者我们不允许运行它

-   程序运行,但没有成功 - 非零退出代码
-   程序运行,零退出代码. 
-   成功!默认情况下,程序的标准输出和标准错误流将发送到终端. 

我们经常对捕获这种输出非常感兴趣,所以就是`output`方法.

```rust
// process2.rs
use std::process::Command;

fn main() {
    let output = Command::new("rustc")
        .arg("-V")
        .output()
        .expect("no rustc?");

    if output.status.success() {
        println!("ok!");
    }
    println!("len stdout {} stderr {}", output.stdout.len(), output.stderr.len());
}
//Ok!
// len stdout 44 stderr 0
```

 与`status`一样我们的程序会阻塞,直到子进程结束,我们返回三件事情 - 状态 (如以前) ,标准输出的内容和标准错误的内容. 
 
 捕获的输出简单`Vec<u8>`- 只是字节. 回想一下,我们不能保证我们从操作系统收到的数据是正确编码的 utf-8 字符串. 事实上,我们不能保证它 _甚至_ 是一个字符串 - 程序可能会返回任意二进制数据. 

如果我们确信输出是 utf-8 ,那么`String::from_utf8`将转换这些 Vec 或字节 - 它返回一个`Result`因为这种转换可能不会成功. 一个更马虎的功能是`String::from_utf8_lossy`这将很好地进行转换, 并在失败时插入无效的 Unicode 标记. 

下面是一个使用 shell 运行程序的有用函数. 这使用通常的 shell机制 将标准错误连接到标准输出. 在 Windows上 shell 的名字是不同的,但是除此之外的东西可以按预期工作. 

```rust
fn shell(cmd: &str) -> (String,bool) {
    let cmd = format!("{} 2>&1",cmd);
    let shell = if cfg!(windows) {"cmd.exe"} else {"/bin/sh"};
    let flag = if cfg!(windows) {"/c"} else {"-c"};
    let output = Command::new(shell)
        .arg(flag)
        .arg(&cmd)
        .output()
        .expect("no shell?");
    (
        String::from_utf8_lossy(&output.stdout).trim_right().to_string(),
        output.status.success()
    )
}


fn shell_success(cmd: &str) -> Option<String> {
    let(output,success) = shell(cmd);
    if success {Some(output)} else {None}
}
```

我修整了右边的任何空格, 所以如果你说的话,`shell("which rustc")`,您将获得没有任何额外换行的路径. 

您可以控制由启动程序的执行通过`Process`, 指定它将运行的目录使用`current_dir`方法和它所使用的环境变量`env`. 

到目前为止,我们的程序只是等待子进程完成. 如果你使用`spawn`方法,我们立即返回,并且必须明确地等待它完成 - 或者在此期间去做其他事情! 这个例子还显示了如何同时抑制 标准错误和标准错误: 

```rust
// process5.rs
use std::process::{Command,Stdio};

fn main() {
    let mut child = Command::new("rustc")
        .stdout(Stdio::null())
        .stderr(Stdio::null())
        .spawn()
        .expect("no rustc?");

    let res = child.wait();
    println!("res {:?}", res);
}
```

默认情况下,子项"继承{inherits}"父项的标准输入和输出. 在这里,我们将 孩子的输出控制 重定向到"nowhere". 这相当于在 Unix shell 中说`>/dev/null 2>/dev/null`. 

现在, 在 Rust可以使用 shell  (`sh`要么`cmd`) 来完成这些事情 . 但通过这种方式,您可以完全程序化地控制进程创建. 

例如,如果我们有`.stdout(Stdio::piped())`那么孩子的标准输出被重定向到管道. 然后`child.stdout`是你可以用来直接读取输出的东西 (例如: implements `Read`) . 同样,你可以使用`.stdin(Stdio::piped()) `方法,以便您可以写入`child.stdin`. 

但如果我们使用了`wait_with_output`代替`wait`, 那么它返回一个`Result<Output>`并将孩子的输出记录到`Output`的`sudout`字段作为一个`Vec<u8>`就像以前一样. 

该`Child`结构也给你一个明确的`kill`方法. 
