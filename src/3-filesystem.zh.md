# 文件系统和进程

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [再看看读取文件](#%E5%86%8D%E7%9C%8B%E7%9C%8B%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6)
- [写进文件](#%E5%86%99%E8%BF%9B%E6%96%87%E4%BB%B6)
- [文件,路径和目录](#%E6%96%87%E4%BB%B6%E8%B7%AF%E5%BE%84%E5%92%8C%E7%9B%AE%E5%BD%95)
- [进程](#%E8%BF%9B%E7%A8%8B)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 再看看读取文件

在 [第 1 部分的末尾](http://chinanf-boy.github.io/gentle-intro/1-basics.zh.html#a%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6)，我展示了如何读取整个文件到一个字符串。 自然，这并不总是一个好法子，所以，现在介绍下如何逐行读取文件。

`fs::File`实现了`io::Read`，这是一个具备可读性的 trait 。 这个 trait 定义了一个能填充`u8`切片字节的`read`方法 - 唯一 _要求_ 的方法，还免费*提供*一些方法，很像
`Iterator`。 您可以使用`read_to_end`填充可读的内容 到 字节 Vec， 还有`read_to_string`可以填充到 一个 string - 必须是 utf-8 编码。

这是一个'原始'读取，没有缓冲区。 对于缓冲性读取, 我们有`io::BufRead` trait，给了我们 `read_line` 和 一个`lines`迭代器。`io::BufReader`将给 _任何_ 具备可读性的类型 提供`io::BufRead`实现。

`fs::File` _也_ 实现了`io::Write`。

确保所有这些 traits 可用的最简单方法是，`use std::io::prelude::*`。

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

fn read_all_lines(filename: &str) -> io::Result<()> {
    let file = File::open(&filename)?;

    let reader = io::BufReader::new(file);// 实现`io::BufRead`

    for line in reader.lines() {
        let line = line?;
        println!("{}", line);
    }
    Ok(())
}
```

这里的`let line = line?`看起来可能有点奇怪。迭代器返回的`line`实际上是一个`io::Result<String>`，我们用`?`解开它。因为在迭代过程中可能 _会_ 出现错误，如: I/O 错误，不是 utf-8 的字节块，等等。

`lines`作为一个迭代器，可以直接使用`collect`从一个文件读取为一个字符串向量，或者用`enumerate`迭代器打印带行号的 line。

然而，这并不是读取 所有行 的最有效方式，因为每行都要分配一个新字符串，有成本。使用`read_line`效率更高，虽然更难看些。请注意，返回的行是包含换行符，可以使用`trim_right`进行移除。

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

分配内存的举动少得很多，因为字符串不会释放其分配的内存， _clearing{清除}_ 也只是缓存区; 一旦字符串有足够的容量，不会再有分配。

这是我们使用一个`块{}`来控制单一借用的情况。`line`是`buf`的借用，而这个借用必须在我们修改`buf`之前完结。Rust 再一次试图阻止我们做一些愚蠢的事情，那就是 _在_ 我们已经清除了缓冲区 _后_ ，访问`line`。(借用检查者有时会有所限制，Rust 由于"非词汇生命周期{non-lexical lifetimes}"，它会分析代码并看到，在`buf.clear()`之后`line`是不使用的。)

完成的不是很漂亮。 虽然我不能给你一个，能返回缓冲区引用的完全迭代器，但我可以给你一些 _看起来像_ 一个迭代器的东西。

首先定义一个泛型结构; 类型参数`R`是'任意实现 Read 的类型'。结构包含读者{reader} 和我们要借用的缓冲区{buf}。

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

然后是`next`方法。 它返回一个`Option`- 就像一个迭代器，当它返回`None`时，迭代器结束。这返回的类型嵌套个`Result`是因为`read_line`可能会失败，我们永远不要错失错误。 所以如果失败了，我们把它的错误包进`Some<Result>`。 否则，文件的自然结束时，它可能读取到零字节 - 而零字节不是错误，只是一个`None`。

此时，缓冲区包含附有换行符 (`\n`) 的行，修剪掉它，然后打包成字符串切片。

```rust
    fn next<'a>(&'a mut self) -> Option<io::Result<&'a str>>{
        self.buf.clear();
        match self.reader.read_line(&mut self.buf) {
            Ok(nbytes) => if nbytes == 0 {
                None // 没有更多行啦!
            } else {
                let line = self.buf.trim_right(); // trim_right的函数签名：`pub fn trim_right(&self) -> &str`
                Some(Ok(line))
            },
            Err(e) => Some(Err(e))
        }
    }
```

现在，请注意 生命周期 如何工作。 我们需要明确的 生命周期 ，因为 Rust 永远不会让我们，在不知道他们的 生命周期 的情况下，搞到借用的字符串切片。在这里，我们说这个借用的字符串的 生命周期 在`self` 的生命周期里面。

而且，生命周期的这个签名不兼容`Iterator`的接口(/trait)， 但是如果兼容就很容易出现问题; 考虑到`collect`试图制作这些字符串切片的 Vec，可这是不能工作的，因为它们都是从 _同一个_ 可变字符串`self.buf`中借用的! (如果您已 读取了文件的 _所有_ ，并转换为字符串，而这个字符串的`lines`迭代器是可以返回字符串切片，因为它们都是借用原始字符串的 _不同_ 部分)。

最终，得到的循环结果更清晰，文件缓冲区对用户是不可见的。

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

你甚至可以这样写循环，显式匹配可以从字符串切片拉出来 :

```rust
    while let Some(Ok(line)) = lines.next() {
        println!("{}", line)?;
    }
```

这很诱人，但你在这里抛出一个可能的错误;每当发生错误时，此循环都会静静地停止。 特别是，它将停止在，无法将 line 转换为 utf-8 的第一处位置。适合休闲代码，不适合生产代码!

## 写进文件

在`Debug`实现处，我们遇到了`write!`宏，- 它也适用于任何实现了`Write`的东西。那另一种，是`print!`:

```rust
    let mut stdout = io::stdout();
    ...
    write!(stdout,"answer is {}\n", 42).expect("write failed");
```

如果 _可能_ 有错误，你必须处理它，不 _容易_ 但可能发生。通常还好，因为如果是文件 I/O，(一般情况下)应该能加个`?`。

但有一个区别: `print!`为每个写锁定 stdout。 通常是您想要输出的内容，因为若没有锁定，多线程程序会混淆输出，搞笑的方式。 但是，如果是要甩出大量文字，那么`write!`会更快。

对任意文件，我们用到`write!`。 在 `write_out`的结尾, `out`变量会释放，文件自然关闭。

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

如果你关心性能，你需要知道 Rust 文件默认是无缓冲的。 所以每个小的写入请求都会直接进入操作系统，而这会明显变慢。我提到了这一点，因为这种默认设置与其他编程语言不同，并且可能导致令人震惊的发现: Rust 可能有脚本语言遗留的残渣!

`io::BufWriter`像`Read`和`io::BufReader`，但带有缓冲的`Write`。

## 文件,路径和目录

这是一个用于在机器上打印 Cargo 目录的小程序。最简单的情况是'~/.cargo'。在一个 Unix shell 环境，所以我们使用`env::home_dir`，因为它是跨平台的。 (它可能会失败，但没有主目录的计算机, 无论如何都不会打算托管 Rust 工具的。 )

这里，我们创建一个[PathBuf](https://doc.rust-lang.org/std/ops/trait.Mul.html)，并使用它的`push`方法，构建完整的文件路径就像 _组件_。 (这比 用`/`,`\`或其他任何东西来要容易得多，老要顾虑系统。)

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

一个`PathBuf`就好像`String`- 它拥有一组可扩展的方法，但具有专门用于构建路径的方法。 但其大部分功能都来自借用版本的`Path`，这就像`&str`。 所以，举个例子，`is_dir`就是一个`Path`方法。

这可能听起来像一种继承形式，但魔法[Deref](https://doc.rust-lang.org/book/deref-coercions.html) trait 的工作方式不同。就像`String/&str`能一起工作 - `PathBuf`引用可 _包裹{Coerce}_ 成`Path`引用。 ('Coerce'是一个很强的词，但这确实是 Rust ，为你提供转换的几个地方之一。 )

```rust
fn foo(p: &Path) {...}
...
let path = PathBuf::from(home);
foo(&path);
```

`PathBuf`与`OsString`有亲密的关系，它代表我们直接从系统获得的字符串。(相应的，`OsString/&OsStr`关系. )

这样的字符串不 _保证_ 可以表示为 utf-8! 现实生活是一个[复杂的事情](https://news.ycombinator.com/item?id=10519932)，特别是看到'他们为什么这么辛苦'的答案。总而言之,首先有几年的 ASCII 传统编码，以及其他语言的多种特殊编码。 其次，人类语言很复杂。 例如'noël'是 _五个_ Unicode 代码点!

确实，现代操作系统文件名的大部分都是 Unicode 格式 (Unix 方面的 utf-8 ，Windows 方面的 UTF-16) ，又或者不是。 Rust 必须严格处理这种可能性。 例如，`Path`有一个`as_os_str`方法。它返回一个`&OsStr`，但是`to_str`方法却是返回一个`Option<&str>`。随缘!

人们在这一点上遇到了麻烦，因为他们已经过分依赖'string'和'character'作为唯一的必要抽象。 正如爱因斯坦所说， 编程语言必须尽可能简单，但并不简单。 系统语言 _需要_ 区分一个`String/&str`(拥有与借用: 这也非常方便) ，如果它希望站在标准化的 Unicode 字符串上，那么它需要另一种类型来处理无效 Unicode 的文本 - 因此有了`OsString/&OsStr`。 请注意，这些类型没有任何有趣的，类似字符串的方法，因为我们本来就不知道无效 Unicode 编码。

但是，人们习惯像处理字符串一样处理文件名，这就是 Rust 使用`PathBuf`方法操作文件路径，会更容易的原因。

您可以`pop`连续去除路径组件。 这里我们从程序的当前目录开始:

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

这是一个有用的变化。 我有一个搜索 配置{config} 文件 的程序，其规则是，它可能出现在当前目录的任何子目录中。 所以我创建`/home/steve/rust/config.txt`，并在`/home/steve/rust/gentle-intro/code`启动此程序:

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

如此像 **git** 的工作方式，当它想知道当前的存储库是是什么的时候。

有关文件的详细信息 (其大小,类型等) 被称为它的 _元数据_。 与往常一样，可能存在错误 - 不仅仅是"找不到"，或是我们没有权限读取此文件。

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

文件的长度 (以字节为单位) 和修改时间很容易解释。 (注意我们可能无法获得这个时间!) 文件类型有方法`is_dir`,`is_file`和`is_symlink`。

`权限{perissions}`是一个有趣的点。 Rust 努力成为跨平台的，所以这是'木桶的最短木条'的例子。 一般来说，你可以查询的仅是，文件是否只读 - '权限'概念在 Unix 中被扩展，并为 `用户/群组/其他` 提供 `读/写/可执行`的权限。

但是，如果您对 Windows 不感兴趣，那么引入特定于平台的 traits 将至少为我们提供，权限模式位数。 (像往常一样，一个 trait 只有在它可见时才会触发。 ) 然后，应用到程序:

```rust
use std::os::unix::fs::PermissionsExt;
...
println!("perm {:o}",data.permissions().mode());
// perm 755
```

(注意"{:o}"用于打印 _八进制_)

(Windows 上的文件是否可执行取决于其扩展名。可执行文件的扩展名可以在`PATHEXT`环境变量找到 - '.exe','. bat'等等) .

`std::fs`包含许多用于处理文件的有用功能，例如复制或移动文件，制作符号链接和创建目录。

要查找目录的内容，`std::fs::read_dir`提供了一个迭代器。 以下是扩展名为".rs"且大小大于 1024 字节 的所有文件:

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

显然，`read_dir`可能会失败 (通常是'找不到'或'没有权限')，但是获取每个新条目时，也可能会失败 (这就像是`line`迭代器 遍历 缓冲的 reader 的内容)。 另外，我们可能无法获取与条目对应的元数据。 一个文件可能没有扩展名，所以我们也必须检查。

为什么不仅搞出，一个遍历路径的迭代器? 在 Unix 上，是`opendir`系统调用在起作用，但在 Windows 上，您无法在不获取元数据的情况下，迭代目录的内容。所以，这已是一个相当优雅的妥协方案，它允许跨平台的代码尽可能的高效。

关于感觉到'错误疲劳{error fatigue}'，你可以被原谅。但请注意 _错误总是存在_ - 这不是 Rust 的新发明。 它只是在努力让你无法忽视它们。 任何操作系统调用都可能失败。

Java 和 Python 等语言会引发异常; 像 Go 和 Lua 这样的语言返回两个值,其中第一个是 结果，第二个是 错误: 像 Rust 也一样，它要考虑到是库函数引发错误的不良。所以才有，这么多错误检查和函数的提前返回。

Rust 使用`Result`，因为它有两面性(either-or): 你不能同时得到 一个结果 和 错误。`?`问号运算符使处理错误更加清晰。

## 进程

一个基本的需求是程序去运行程序，或者 _启动进程_ 。 你的程序可以 _启动{launch}_ 尽可能多的子进程，顾名思义此类进程间有特殊的关系。

使用`Command`结构，运行子程序很简单，拿到传递给子程序的构建参数:

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

看啊，`new`收到该程序的名称 (它将查找`PATH`，如果不是绝对文件名的话)，`arg`增加了一个新的 _尾随参数_，并且`status`导致它运行。 这返回一个`Result`，若为`Ok`，说明程序运行了，且包含一个`退出状态{ExitStatus}`。在这种情况下，程序成功，并返回 退出码 0 (使用`unwrap`是因为，如果程序被信号杀死了，我们不总是得到退出代码)。

如果我们改变了`-V`至`-v` (一个易犯的错误)，导致`rustc`失败:

```
error: no input filename given

cool false code 101
```

所以有三种可能性:

- 程序不存在,很糟糕,或者我们不允许运行它
- 程序运行,但没有成功 - 非零退出代码
- 程序运行,零退出代码。成功!

默认情况下，程序的 stdout 和标准错误流将发送到终端。

我们经常对这种输出非常感兴趣，也就是`output`方法.

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

与`status`一样，我们的程序会阻塞，直到子进程结束，我们返回三个值 - status (如上)，stdout 的内容和标准错误的内容。

捕获到的内容输出是简单的`Vec<u8>`- 只是字节。 回想一下，我们不能保证,我们从操作系统收到的数据是正确的 utf-8 编码 字符串。 事实上，我们 _甚至_ 不能保证它是一个字符串 - 程序可能会返回任意二进制数据。

如果我们确信输出是 utf-8，那么`String::from_utf8`将转换这些 Vec 或字节 - 它返回的是一个`Result`，因为这种转换可能不会成功。 一个更迷糊的函数是`String::from_utf8_lossy`，能很好地转换，并在转换失败时插入无效的 Unicode`�`标记。

下面是一个使用 shell 来运行程序的有用函数。这使用通常的 shell 机制，将标准错误连接到 stdout。 在 Windows 上 shell 的名字是不同的，但是除此之外的东西可以按预期工作。

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

我修整了右边的任何空格, 所以，如果你说`shell("which rustc")`的话，您将获得没有任何额外换行的路径。

您可以通过`Process`控制启动程序的执行, 使用`current_dir`方法指定它将运行的目录，和它所使用的环境变量`env`。

到目前为止，我们的程序只是等待子进程完成. 如果你使用`spawn`方法，我们立即返回，可以明确地等待它完成 - 或者在此期间去做其他事情! 这个例子还显示了如何同时抑制 标准错误和标准错误:

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

默认情况下，子进程"继承"父进程的标准输入和输出。 在这里，我们将 孩子的输出控制重定向到"没有"。 这相当于在 Unix shell 中说`>/dev/null 2>/dev/null`。

现在， 你在 Rust 可以使用 shell (`sh`要么`cmd`) 来完成这些事情 。 但通过这种方式，您可以完全程序化地控制进程的创建。

例如，如果我们编写的是`.stdout(Stdio::piped())`，那么孩子的 stdout 就被重定向到管道。那`child.stdout`就是你可以用来直接读取输出的东西 (例如: 要实现了`Read`)。 同样，你可以使用`.stdin(Stdio::piped())`方法，以便您可以写进`child.stdin`。

但，如果我们使用`wait_with_output`代替`wait`, 那么它会返回一个`Result<Output>`，并将孩子的输出，会以一个`Vec<u8>`，记录到`Output`的`sudout`字段，就像之前的一样。

`Child`结构，也给你一个明确的`kill`方法。
