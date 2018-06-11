
# 文件系统和进程

## 再看看Reading Files

在第1部分的末尾,我展示了如何将整个文件读入字符串. 自然,这并不总是一个好主意,所以这里是如何逐行读取文件. 

`FS ::文件`器物`IO ::阅读`,这是任何可读性的特征. 这个特质定义了一个`读`方法将填补一部分`u8`与字节 - 这是唯一的_需要_特质的方法,你会得到一些_提供_免费的方法,很像`迭代器`. 您可以使用`read_to_end`用可读的内容填充字节向量,并且`read_to_string`填充一个字符串 - 必须是UTF-8编码. 

这是一个'原始'阅读,没有缓冲. 对于缓冲阅读有`IO ::的BufRead`给我们的特质`read_line`和a`线`迭代器. `IO :: BufReader`将提供执行`IO ::的BufRead`对于_任何_可读. 

`FS ::文件` _也_器物`IO ::写`. 

确保所有这些特征可见的最简单方法是`使用std :: io :: prelude :: *`. 

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

该`让line = line?`可能看起来有点奇怪. 该`线`迭代器返回的实际上是一个`IO ::结果<字符串>`我们解开它`?`. 因为事情_能够_在迭代过程中出现错误 -  I / O错误,吞噬不是UTF-8的字节块,等等. 

`线`作为一个迭代器,可以直接使用一个文件读入一个字符串向量`搜集`,或者用行号打印出行`枚举`迭代器. 

然而,这并不是读取所有行的最有效方式,因为每行都分配了一个新字符串. 使用效率更高`read_line`,虽然更尴尬. 请注意,返回的行包含换行符,可以使用该换行符进行移除`trim_right`. 

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

这导致分配少得多,因为_空地_该字符串不释放其分配的内存;一旦字符串有足够的容量,不会再有分配. 

这是我们使用一个块来控制借用的情况之一. `线`是借来的`buf`,这个借用必须在我们修改之前完成`buf`. Rust再一次试图阻止我们做一些愚蠢的事情,那就是访问`线` _后_我们已经清除了缓冲区.  (借阅检查者有时可能会受到限制,铁锈是由于"非词汇生活时间",它会分析代码并看到线`之后不使用`buf.clear () `. ) `这不是很漂亮. 

我不能给你一个适当的迭代器,它返回缓冲区的引用,但我可以给你一些东西容貌_像一个迭代器. _首先定义一个通用结构体;

类型参数\[R`是'任何实现Read的类型'. `它包含读者和我们要借用的缓冲区. 然后

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

下一个`方法. `它返回一个`选项`- 就像迭代器,当它返回时`没有`迭代器结束. 返回的类型是a`结果`因为`read_line`可能会失败,我们_永远不要丢失错误_. 所以如果失败了,我们把它的错误包括进去`一些<结果>`. 否则,它可能读取了零字节,这是文件的自然结束 - 不是错误,只是一个`没有`. 

此时,缓冲区包含附有换行符 (\`\\ n') 的行. 修剪掉它,然后打包字符串片. 

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

现在,请注意生命时间如何工作. 我们需要明确的一生,因为Rust永远不会让我们在不知道他们的一生的情况下发放借来的字符串片. 在这里,我们说这个借来的字符串的生命周期在一生中`自`. 

而且这个签名与终身不兼容`迭代器`. 但是如果兼容性很容易出现问题;考虑`搜集`试图制作这些字符串切片的矢量. 这是不可能的,因为它们都是从同一个可变字符串中借用的! (如果您已阅读_所有_将文件转换为字符串,然后是字符串`线`迭代器可以返回字符串切片,因为它们都是从中借用的_不同_原始字符串的一部分) . 

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

你甚至可以这样写循环,因为显式匹配可以将字符串切片拉出来: 

```rust
    while let Some(Ok(line)) = lines.next() {
        println!("{}", line)?;
    }
```

这很诱人,但你在这里抛出一个可能的错误;每当发生错误时,此循环都会静静地停止. 特别是,它将停止在Rust无法将线路转换为UTF-8的第一个位置. 适合休闲代码,不适合生产代码!

## 写入文件

我们遇到了`写!`宏执行时`调试`- 它也适用于任何实现的东西`写`. 所以这是另一种说法`打印!`: 

```rust
    let mut stdout = io::stdout();
    ...
    write!(stdout,"answer is {}\n", 42).expect("write failed");
```

如果有错误_可能_,你必须处理它. 它可能不是很好_容易_但它可能发生. 它通常很好,因为如果你正在做文件I / O你应该在一个上下文中`?`作品. 

但有一个区别: `打印!`为每个写入锁定标准输出. 这通常是您想要输出的内容,因为没有锁定多线程程序可能会以有趣的方式混淆输出. 但是,如果你抽出大量文字,那么`写!`将会变得更快. 

对于我们需要的任意文件`写!`. 该文件在关闭时关闭`出`在结束时被丢弃`写出`,这既方便又重要. 

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

如果你关心性能,你需要知道Rust文件默认是无缓冲的. 所以每个小的写入请求都会直接进入操作系统,而且这将会明显变慢. 我提到了这一点,因为这种默认设置与其他编程语言不同,并且可能导致令人震惊的发现: Rust可能被脚本语言遗留下来!就像`读`和`IO :: BufReader`, 有`IO :: BufWriter`缓冲任何`写`. 

## 文件,路径和目录

这是一个用于在机器上打印货物目录的小程序. 最简单的情况是它是'〜/ .cargo'. 这是一个Unix shell扩展,所以我们使用`ENV :: HOME_DIR`因为它是跨平台的.  (它可能会失败,但没有主目录的计算机无论如何不会托管Rust工具. ) 

然后我们创建一个[PathBuf](https://doc.rust-lang.org/std/ops/trait.Mul.html)并使用它的`推`方法从其构建完整的文件路径_组件_.  (这比用'/','\\'或其他任何东西来欺骗要容易得多,取决于系统. ) 

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

一个`PathBuf`就好像`串`- 它拥有一组可扩展的角色,但具有专门用于构建路径的方法. 但其大部分功能都来自借用版本`路径`,这就像`&STR`. 所以,例如,`is_dir`是一个`路径`方法. 

这可能听起来像一种继承形式,但魔法[Deref](https://doc.rust-lang.org/book/deref-coercions.html)特质的工作方式不同. 它就像它一样工作`字符串/&STR`- 参考`PathBuf`可_裹挟_成为参考`路径`.  ('Coerce'是一个很强的词,但这确实是Rust为你做转换的少数几个地方之一. ) 

```rust
fn foo(p: &Path) {...}
...
let path = PathBuf::from(home);
foo(&path);
```

`PathBuf`与...有亲密的关系`OsString`,它代表我们直接从系统获得的字符串.  (有一个相应的`OsString /&OsStr`关系. ) 

这样的字符串不是_保证_可以表示为UTF-8!现实生活是一个[复杂的事情](https://news.ycombinator.com/item?id=10519932),特别是看到'他们为什么这么辛苦'的答案. 总而言之,首先有几年的ASCII传统编码,以及其他语言的多种特殊编码. 其次,人类语言很复杂. 例如'noël'是_五_Unicode代码点!

确实,现代操作系统文件名的大部分时间都是Unicode (Unix方面的UTF-8,Windows方面的UTF-16) ,除非它们不是. Rust必须严格处理这种可能性. 例如,`路径`有一个方法`as_os_str`它返回一个`&OsStr`,但是`to_str`方法返回一个`选项<&STR>`. 并不总是可能!

人们在这一点上遇到了麻烦,因为他们已经过分依赖'串'和'性格'作为唯一必要的抽象. 正如爱因斯坦所说,编程语言必须尽可能简单,但并不简单. 系统语言_需求_一个`字符串/&STR`区别 (拥有与借用: 这也非常方便) ,如果它希望在Unicode字符串上标准化,那么它需要另一种类型来处理无效Unicode的文本 - 因此`OsString /&OsStr`. 请注意,这些类型没有任何有趣的类似字符串的方法,这正是因为我们不知道编码. 

但是,人们习惯于像处理字符串一样处理文件名,这就是Rust使用文件路径更容易操作的原因`PathBuf`方法. 

您可以`流行的`连续去除路径组件. 这里我们从程序的当前目录开始: 

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

这是一个有用的变化. 我有一个搜索配置文件的程序,其规则是它可能出现在当前目录的任何子目录中. 所以我创建`/home/steve/rust/config.txt`并启动此程序`/家庭/史蒂夫/防锈/温柔的开场/码`: 

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

这几乎是如此**混帐**当它想知道当前的回购是什么时会起作用. 

有关文件的详细信息 (其大小,类型等) 被称为它的_元数据_. 与往常一样,可能存在错误 - 不仅仅是"找不到",而且如果我们没有权限读取此文件. 

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

`权限`是一个有趣的. Rust努力成为跨平台的,所以这是'最低公分母'的例子. 一般来说,你可以查询的只是文件是否只读 - '权限'概念在Unix中被扩展,并为用户/群组/其他编码读/写/可执行文件. 

但是,如果您对Windows不感兴趣,那么引入特定于平台的特征将至少为我们提供权限模式位.  (像往常一样,一个特征只有在它可见时才会触发. ) 然后,将该程序应用到它自己的可执行文件中: 

```rust
use std::os::unix::fs::PermissionsExt;
...
println!("perm {:o}",data.permissions().mode());
// perm 755
```

 (注意"{: o}"用于打印_八进制_) 

 (Windows上的文件是否可执行取决于其扩展名. 可执行文件的扩展名可以在PATHEXT`环境变量 - '.exe','. bat'等等) . `的std :: FS

`包含许多用于处理文件的有用功能,例如复制或移动文件,制作符号链接和创建目录. `要查找目录的内容,

的std :: FS :: read_dir`提供了一个迭代器. `以下是扩展名为".rs"且大小大于1024字节的所有文件: 明显

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

read_dir`可能会失败 (通常是'找不到'或'没有权限') ,但是获取每个新条目可能会失败 (这就像是`线`迭代器遍历缓冲的阅读器的内容) . `另外,我们可能无法获取与条目对应的元数据. 一个文件可能没有扩展名,所以我们也必须检查. 为什么不只是一个遍历路径的迭代器?

在Unix上这是这样的`执行opendir`系统调用起作用,但在Windows上,您无法在不获取元数据的情况下迭代目录的内容. 所以这是一个相当优雅的妥协方案,它允许跨平台的代码尽可能高效. 

在这一点上你可以原谅感觉'错误疲劳'. 但请注意_错误总是存在_- 这不是Rust正在发明新的. 它只是在努力让你无法忽视它们. 任何操作系统调用都可能失败. 

Java和Python等语言引发异常;像Go和Lua这样的语言返回两个值,其中第一个是结果,第二个是错误: 像Rust一样,它被认为是库函数引发错误的不良方式. 所以有很多错误检查和函数的早期返回. 

Rust使用`结果`因为它不是 - 或者: 你不能同时得到结果和错误. 问号运算符使处理错误更加清晰. 

## 流程

一个基本的需求是程序运行程序,或者_启动过程_. 你的程序可以_卵_尽可能多的儿童进程,顾名思义他们与父母有特殊的关系. 

运行程序很简单,使用`命令`struct,它构建传递给程序的参数: 

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

所以`新`收到该程序的名称 (它将被查找`路径`如果不是绝对文件名) ,`arg`增加了一个新的论点,并且`状态`导致它运行. 这返回一个`结果`,这是`好`如果程序实际运行,包含一个`退出状态`. 在这种情况下,程序成功,并返回退出码0摅`是因为如果程序被信号杀死了,我们不能总是得到代码) . `如果我们改变了

\-V`至`-v` (一个容易的错误) `rustc`失败: `所以有三种可能性: 

    error: no input filename given

    cool false code 101

程序不存在,很糟糕,或者我们不允许运行它

-   程序运行,但没有成功 - 非零退出代码
-   程序运行,零退出代码. 
-   成功!默认情况下,程序的标准输出和标准错误流将发送到终端. 

我们经常对捕获这种输出非常感兴趣,所以就是这样

产量`方法. `与...一样

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
// ok!
// len stdout 44 stderr 0
```

状态`我们的程序会阻塞,直到子进程结束,我们返回三件事情 - 状态 (如以前) ,标准输出的内容和标准错误的内容. `捕获的输出很简单

VEC &lt;U8>`- 只是字节. `回想一下,我们不能保证我们从操作系统收到的数据是正确编码的UTF-8字符串. _事实上,我们不能保证它甚至_是一个字符串 - 程序可能会返回任意二进制数据. 

如果我们确信输出是UTF-8,那么`字符串:: from_utf8`将转换这些向量或字节 - 它返回一个`结果`因为这种转换可能不会成功. 一个更马虎的功能是`字符串:: from_utf8_lossy`这将很好地进行转换并在无效的Unicode标记insert处插入失败. 

这是一个使用shell运行程序的有用函数. 这使用通常的shell机制将标准错误连接到标准输出. 在Windows上shell的名字是不同的,但是除此之外的东西可以按预期工作. 

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
    let (output,success) = shell(cmd);
    if success {Some(output)} else {None}
}
```

如果你说的话,我正在修整右边的任何空格`外壳 ("哪个rustc") `您将获得没有任何额外换行的路径. 

您可以控制由启动的程序的执行`处理`通过指定它将运行的目录使用`current_dir`方法和它所使用的环境变量`env`. 

到目前为止,我们的程序只是等待子进程完成. 如果你使用`卵`方法,我们立即返回,并且必须明确地等待它完成 - 或者在此期间去做其他事情!这个例子还显示了如何同时抑制标准错误和标准错误: 

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

默认情况下,子项"继承"父项的标准输入和输出. 在这种情况下,我们将孩子的输出手柄重定向到"无处". 这相当于说`> / dev / null 2> / dev / null`在Unix shell中. 

现在,可以使用shell来完成这些事情 (`sh`要么`cmd`) 在Rust. 但通过这种方式,您可以完全程序化地控制流程创建. 

例如,如果我们有`.stdout (STDIO ::管道 () ) `那么孩子的标准输出被重定向到管道. 然后`child.stdout`是你可以用来直接读取输出的东西 (即implements`读`) . 同样,你可以使用`.stdout (STDIO ::管道 () ) `方法,以便您可以写入`child.stdin`. 

但如果我们使用了`wait_with_output`代替`等待`那么它返回一个`结果<输出>`并将孩子的输出记录下来`标准输出`那个领域`产量`作为一个`VEC <U8>`就像以前一样. 

该`儿童`结构也给你一个明确的`杀`方法. 
