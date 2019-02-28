# 标准库范畴

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [阅读文档](#%E9%98%85%E8%AF%BB%E6%96%87%E6%A1%A3)
- [Maps](#maps)
- [示例: 计算词数](#%E7%A4%BA%E4%BE%8B-%E8%AE%A1%E7%AE%97%E8%AF%8D%E6%95%B0)
- [Sets](#sets)
- [示例: 交互式命令处理](#%E7%A4%BA%E4%BE%8B-%E4%BA%A4%E4%BA%92%E5%BC%8F%E5%91%BD%E4%BB%A4%E5%A4%84%E7%90%86)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 阅读文档

在本节中，我将简要介绍 Rust 标准库的一些常见部分。文档非常好，但有一点讲解和一些例子总是有好处的。

最初，阅读 Rust 文档可能很具挑战性，所以我会举个`Vec`例子。 一个有用的提示是勾选'[-]'框来折叠文档。(如果使用下载标准库源代码`rustup component add rust-src` ，一个 '[src]'链接将出现在此旁边。)这些可以让您全面了解所有可用的方法。

首先要注意的是， _并非所有可能的方法_ 都定义`Vec`本身。 它们是(大部分) 可改变 vec 的方法，例如`push`。有些方法仅适用于类型匹配某些约束的 Vec。例如，你只能调用`dedup`(删除重复项) ，如果这个类型确实是可以相互比较的东西。 `Vec`有多个`impl`定义块，针对不同类型的约束。

然后是`Vec<T>`和`&[T]`间非常特殊的关系。 任何在切片上工作的方法也可以直接在 vec 上工作，而不必明确地使用`as_slice`方法。 这种关系表达为`Deref<Target=[T]>`。当你为需要 切片 的函数传递一个 Vec 时，这也会起作用 - 这是类型之间自动转换的几个之一。 所以像切片方法`first`，它可能 - 返回对第一个元素的引用，或者`last`，同时也为 Vec 工作。 许多切片方法与相应的字符串方法类似，所以就有了，为了在索引处获得一对切片的`split_at`方法，和`starts_with`能检查 vec 是否以某值序列开始，还有`contains`能检查 vec 是否包含特定值。

要知道，是没有查找特定值的索引的`search`方法的，但这里有一条经验法则: 如果在方法集上找不到想要的方法，请在 迭代器 上查找方法:

```Rust
    let v = vec![10,20,30,40,50];
    assert_eq!(v.iter().position(|&i| i == 30).unwrap(), 2);
```

(该`&`是因为这是一个建于 _引用_ 上 的迭代器 - 或者你可以用`*i == 30`)。

同样， vec 上没有`map`方法，因为`iter().map(...).collect()`会做这项工作。 Rust 不喜欢不必要地分配(内存) - 通常你不需要`map`这样的过程结果，因这会是实际分配的 Vec，浪费。

所以我建议你熟悉所有的 迭代器 方法，因为它们对编写好的 Rust 代码至关重要，不必一直写出循环。 与往常一样，编写小程序来探索 迭代器 方法，而不是在更复杂的程序中与它们搏斗。

`Vec<T>`和`&[T]`方法拥抱共同的 trait : Vec 知道如何进行自己的调试显示(但，只有其元素也实现`Debug`，才如此。) 同样，如果它们的元素是可克隆的，那就是可克隆的。 他们实现了`Drop`，那当 vec 最终死亡时就会发生对应情况; 内存被释放，并且所有元素也被释放。

该`Extend` trait 是说，不需要一个循环，就可以让 迭代器 的值添加到一个 Vec 中:

```Rust
v.extend([60,70,80].iter());
let mut strings = vec!["hello".to_string(), "dolly".to_string()];
strings.extend(["you","are","fine"].iter().map(|s| s.to_string()));
```

还有`FromIterator`，它可以让 vec 由迭代器 _构成{constructed}_ 。(迭代器`collect`方法依赖这个)。

任何容器(vec...)都需要可迭代。 回想一下有[三种迭代器](./2-structs-enums-lifetimes.zh.html#%E4%B8%89%E7%A7%8D%E8%BF%AD%E4%BB%A3%E5%99%A8)

```Rust
for x in v {...} // 返回 T, 消耗 v
for x in &v {...} // 返回 &T
for x in &mut v {...} // 返回 &mut T
```

该`for`声明依赖于`IntoIterator`trait，实际上有三种实现。

然后是索引，由`Index`控制(从 vec 中读取) 和`IndexMut`(修改一个 Vec)。存在很多可能性，因为还有切片索引，像`v[0..2]`会返回切片，以及`v[0]`会返回对第一个元素的引用。

这里有一些`From` trait 的实现。例如`Vec::from("hello".to_string())`会给你一个字符串底层字节`Vec<u8>`的 Vec 。 现在，已经有一种`into_bytes`方法在`String`上，为什么要重复? 有多种方式来做同样的事情似乎很困惑，但是这是必要的，因为显式 trait 使泛型方法成为可能。

有时候， Rust 类型系统的局限性会让事情变得笨拙。 这里的一个例子是`PartialEq` 要为尺寸 32 的数组 _单独_ 定义!(以后会变得更好。) 虽然可以将 Vec 与数组进行方便地比较，但要注意大小限制。

还有[隐藏的珠宝](http://xion.io/post/code/rust-iter-patterns.html)深埋在文档中。 正如 Karol Kuczmarski 所说: "因为说实话: 没有人会滚动那么远"。 如何处理迭代器中的错误? 假设你映射了一些可能失败的操作，就返回`Result`好了，然后收集结果:

```Rust
fn main() {
    let nums =["5","52","65"];
    let iter = nums.iter().map(|s| s.parse::<i32>());
    let converted: Vec<_> = iter.collect();
    println!("{:?}",converted);
}
//[Ok(5), Ok(52), Ok(65)]
```

还行，但现在你必须小心地解开这些错误! 但是，如果你要求 vec _包裹_ 在一个`Result`的话， 那 Rust 会知道如何做正确的事情 - 也就是说，无论是一个 vec 还是一个错误，都能处理了:

```Rust
    let converted: Result<Vec<_>,_> = iter.collect();
//Ok([5, 52, 65])
```

如果这有个错误? 然后你会在遇到第一个错误时得到`Err`。 这是一个灵活`collect`的好例子。(这里的符号可能会吓人 -`Vec<_>`意味着"这是一个 Vec ，忽略实际的类型，和`Result<Vec<_>,_>`还要求 Rust 忽略错误类型。)

文档中有 _许多_ 的详细信息。 但它肯定比 C ++文档所说的`std::vec`更清晰。

> 对元素施加的要求取决于对容器执行的实际操作。 一般来说，要求元素类型是完整类型并且符合 Erasable 的要求，但是许多成员函数会施加更严格的要求。

用 C ++，你是独立思考的。 Rust 的清晰度一开始就让人尊敬，但当你学习阅读约束条件时，你将确切地知道`Vec`要的任何特定方法.

我建议你使用`rustup component add rust-src`获得源代码，因为标准库源代码非常易读，并且方法实现通常不如方法声明那么可怕。

## Maps

_Maps_(有时叫 _关联数组_ 要么 _dicts{字典}_ ); 可以让你存放 键值对的数据结构。这不是一个光想的概念，可以用元组+数组来完成:

```Rust
    let entries =[("one","eins"),("two","zwei"),("three","drei")];

    if let Some(val) = entries.iter().find(|t| t.0 == "two") {
        assert_eq!(val.1,"zwei");
    }
```

对小 map 来说，还能用，且只需要与 定义的 key 相等就好啦，但搜索需要线性时间 - 与 map 大小成比例。

要想搜索 _许多_ 键/值 对，有个更好的`HashMap`:

```Rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("one","eins");
map.insert("two","zwei");
map.insert("three","drei");

assert_eq!(map.contains_key("two"), true);
assert_eq!(map.get("two"), Some(&"zwei"));
```

为什么是`&"zwei"`? 这是因为`get`返回一个 _引用_ ，而不是 **值本身** 。而 **这个值** 的类型是`&str`，所以我们得到一个`&&str`。 一般来说，它 _必须_ 作为一个引用，因为我们不能只 _移动_ 值，而不管其拥有的类型。

`get_mut`就好像`get`，但返回一个可能的可变引用。 这里我们有一个字符串/整数的映射，并希望更新'two'键的值:

```Rust
let mut map = HashMap::new();
map.insert("one",1);
map.insert("two",2);
map.insert("three",3);

println!("before {}", map.get("two").unwrap());

{
    let mut mref = map.get_mut("two").unwrap();
    *mref = 20;
}

println!("after {}", map.get("two").unwrap());
// before 2
// after 20
```

请注意，获取的可写引用发生在它自己的块中 - 否则，我们将有一个可变的借用持续到结束，那样的话， Rust 不会允许`map.get("two")`从`map`再次借用; 同一作用域内已经有一个可变引用，不允许再出现(同一个 map 的)任何引用。(因为，它不能保证那些只读的引用保持有效。)所以解决方案是确保可变借用，不会持续很长时间。

这不是最优雅的 API，但我们不能抛弃任何可能的错误。 Python 会抛出一个异常，而 C ++只会创建一个默认值。(这很方便，但偷偷摸摸;容易忘记`a_map["two"]`的成本，也总是返回一个整数，让我们不能区分 `0` 和 '未找到' 之间的区别， _还加上_ 一个额外项被创建!)

其实，没有人只调用`unwrap`, 除了例子外。 但是，您看到的大多数 Rust 代码都由一些独立的示例组成! 匹配发生的可能性更大:

```Rust
if let Some(v) = map.get("two") {
    let res = v + 1;
    assert_eq!(res, 3);
}
...
match map.get_mut("two") {
    Some(mref) => *mref = 20,
    None => panic!("_now_ we can panic!")
}
```

我们可以遍历 key/值对，但(实际)不以任何特定的顺序。

```Rust
for(k,v) in map.iter() {
    println!("key {} value {}", k,v);
}
// key one value eins
// key three value drei
// key two value zwei
```

也有分别按键和值，返回迭代器的`key`/`values`方法，这使得创建值的 Vec 变得容易。

## 示例: 计算词数

与文本有关的一个有趣的事情是，计数字的频率。

用`split_whitespace`将文本分解为单词很简单，但是我们要真的遵循标点符号。总之，这些词应该被定义为只包含字母字符，也需要换成小写字母进行比较。

直接在 一个 map 上做一个可变的查找，虽然处理查找失败的情况有点尴尬。但幸运的是，有一种更新 map 值的方式:

```Rust
let mut map = HashMap::new();

for s in text.split(|c: char| ! c.is_alphabetic()) {
    let word = s.to_lowercase();
    let mut count = map.entry(word).or_insert(0);
    *count += 1;
}
```

如果没有对应于某个单词的现有计数，那么让我们为该单词创建一个包含零的项，并 _插{insert}_ 进 map。它正是 C ++映射所做的，除了它是明确的，不是偷偷摸摸的。

这段代码中只有一个显式类型`char`，因为`split`的使用关系到字符串`Pattern`trait 的怪癖。但我们可以推断出 key 类型是`String`和 value 类型是`i32`。

根据 Gutenberg 项目 的[福尔摩斯历险记](http://www.gutenberg.org/cache/epub/1661/pg1661.txt)，我们可以更彻底地进行测试。 唯一字词的总数(`map.len()`) 是 8071。

如何找到最常见的二十个单词? 首先，将 map 转换为 (key, value) 元组的 Vec。(若我们使用了`into_iter`，会消耗了 map。)

```Rust
let mut entries: Vec<_> = map.into_iter().collect();
```

接下来，我们可以按降序排列。 `sort_by`接收来自于`Ord` trait 的`cmp`方法的结果，它是由整型值类型实现的:

```Rust
    entries.sort_by(|a,b| b.1.cmp(&a.1));
```

最后打印出，前 20 个项:

```Rust
    for e in entries.iter().take(20) {
        println!("{} {}", e.0, e.1);
    }
```

(好吧，你也 _可以_ 只是用`0..20`循环，并索引 Vec 就好了 - 这虽然没有错，但有点随便 - 而且对于大型迭代来说，可能更昂贵。)

```
  38765
the 5810
and 3088
i 3038
to 2823
of 2778
a 2701
in 1823
that 1767
it 1749
you 1572
he 1486
was 1411
his 1159
is 1150
my 1007
have 929
with 877
as 863
had 830
```

有点惊喜 - 首个空文本是什么? 这是因为`split`，适用于单字符分隔符，因此任何标点符号或额外空格都会导致新的分割。

## Sets

Sets(集合) 是 只关心 key 的 map，而不关联任何值。 所以`inserts`只需要一个值，可使用`contains`用于测试一个值是否在一个集合中。

像所有容器一样，您可以用迭代器创建一个`HashSet`。这正是 `collect`所做的，一旦你给了它必要的类型提示。

```Rust
// set1.rs
use std::collections::HashSet;

fn make_set(words: &str) -> HashSet<&str> {
    words.split_whitespace().collect()
}

fn main() {
    let fruit = make_set("apple orange pear orange");

    println!("{:?}", fruit);
}
// {"orange", "pear", "apple"}
```

注意(如预期的那样) 重复插入同一个 key 是不起作用的，且，集合中值的顺序并不重要。

没有常用操作，它们不会成为集合:

```Rust
let fruit = make_set("apple orange pear");
let colours = make_set("brown purple orange yellow");

for c in fruit.intersection(&colours) {
    println!("{:?}",c);
}
// "orange"
```

他们都创建 迭代器，并且可以使用 `collect`使这些成为集合。

这是一个快捷方式，就像我们为 vec 定义的那样:

```Rust
use std::hash::Hash;

trait ToSet<T> {
    fn to_set(self) -> HashSet<T>;
}

impl<T,I> ToSet<T> for I
where T: Eq + Hash, I: Iterator<Item=T> {

    fn to_set(self) -> HashSet<T> {
       self.collect()
    }
}

...

let intersect = fruit.intersection(&colours).to_set();
```

就像所有的 Rust 泛型一样，你需要限制类型 - 这只能为理解平等(`Eq`) 的类型和存在一个"散列函数"(`Hash`)的`T`实现。请记住，没有叫`Iterator`的 _类型_ ，所以`I`代表任何 _实现_ `Iterator`的类型。

这种在标准库类型上，实现我们自己的方法的技术似乎有点过于强大，但是同样存在规则。 我们只能为自己的 trait 做到这一点。 如果结构和 trait 来自同一个箱子(特别是 stdlib ) ，那么这种实现将不被允许。这种方式，您可以避免造成混乱。

在祝贺有如此聪明，方便的捷径之前，您应该意识到后果。 如果`make_set`是这样写的，若这些是所有权字符串的集合，那`intersect`的实际类型可能会是个'惊喜':

```Rust
fn make_set(words: &str) -> HashSet<String> {
    words.split_whitespace().map(|s| s.to_string()).collect()
}
...
// intersect 是 HashSet<&String>!
let intersect = fruit.intersection(&colours).to_set();
```

一般情况下， Rust 不会突然开始复制所有权字符串。 `intersect`包含从`fruit`借来的单个`&String`。我可以向你保证，当你开始修补生命周期时，这会给你带来麻烦! 更好的解决方案是使用迭代器的`cloned`方法来创建 `intersection` 的所有权字符串副本。

```Rust
// intersect 是 HashSet<String> - 更好了
let intersect = fruit.intersection(&colours).cloned().to_set();
```

比`to_set`更强大，可能会是`self.cloned().collect()`，我邀请您试试。

## 示例: 交互式命令处理

与程序进行交互式会话通常很有用。 每行都被读入并分割成单词; 该命令在第一个单词上查找，其余单词作为参数传递给该命令。

一个自然的实现是 命令名称/闭包 的 map。 但是，我们如何存储闭包，因为它们都会有不同的大小? 将他们放入盒子(Box)，那么他们会复制到堆上:

这是第一次尝试:

```Rust
    let mut v = Vec::new();
    v.push(Box::new(|x| x * x));
    v.push(Box::new(|x| x / 2.0));

    for f in v.iter() {
        let res = f(1.0);
        println!("res {}", res);
    }
```

我们在第二次 push 时，遇到了非常明显的错误:

```
  = note: expected type `[closure@closure4.rs:4:21: 4:28]`
  = note:    found type `[closure@closure4.rs:5:21: 5:28]`
note: no two closures, even if identical, have the same type
```

`rustc`导出了一个过于省略的类型，所以在事情刚刚开始之前，有必要强制该 Vec 具有 _Box trait 类型_ :

```Rust
    let mut v: Vec<Box<Fn(f64)->f64>> = Vec::new();
```

我们现在可以使用相同的技巧，并将这些盒化的闭包保存在一个`HashMap`。 我们仍然需要警惕生命周期，因为闭包可以从他们的环境中借用。

直接选择`FnMut`作为闭包签名具有诱导性- 换句话说，他们可以修改任何捕获的变量。 但，我们会有不止一个命令，每个命令都有自己的闭包，所以你不能随意可变借用相同的变量。

最后，闭包被传递一个*可变引用*作为一个参数，加上一段字符串切片(`&[&str]`) 代表命令参数，会返回一些`Result`- 若为错误我们会用`String`。

`D`是数据类型，可以是任何带有一个固有大小的数据。

```Rust
type CliResult = Result<String,String>;

struct Cli<'a,D> {
    data: D,
    callbacks: HashMap<String, Box<Fn(&mut D,&[&str])->CliResult + 'a>>
}

impl<'a,D: Sized> Cli<'a,D> {
    fn new(data: D) -> Cli<'a,D> {
        Cli{data: data, callbacks: HashMap::new()}
    }

    fn cmd<F>(&mut self, name: &str, callback: F)
    where F: Fn(&mut D, &[&str])->CliResult + 'a {
        self.callbacks.insert(name.to_string(),Box::new(callback)); // 装箱
    }
```

`cmd`被传递一个名称和任何与我们的签名相匹配的闭包，这个闭包被装箱并输入到 map 。 `Fn`意味着我们的闭包借用他们的环境，但不能修改。 它是一个声明比实际实现更可怕的泛型方法! 忘记明确生命周期是一个常见的错误 - Rust 不会让我们忘记这些闭包限于他们环境的生命周期!

现在读取和运行命令:

```Rust
    fn process(&mut self,line: &str) -> CliResult {
        let parts: Vec<_> = line.split_whitespace().collect();
        if parts.len() == 0 {
            return Ok("".to_string());
        }
        match self.callbacks.get(parts[0]) {
            Some(callback) => callback(&mut self.data,&parts[1..]),
            None => Err("no such command".to_string())
        }
    }

    fn go(&mut self) {
        let mut buff = String::new();
        while io::stdin().read_line(&mut buff).expect("error") > 0 {
            {
                let line = buff.trim_left();
                let res = self.process(line);
                println!("{:?}", res);

            }
            buff.clear();
        }
    }
```

非常简单明了 - 将行分成单词，做成 vec ，查找 map 中的第一个单词，并用存储的可变数据和其余单词调用闭包。 空行会被忽略，不会被视为错误。

接下来，让我们定义一些帮助函数，使我们的闭包更容易返回正确和不正确的结果。 这有点 _小_ 聪明;它们是泛型函数，适用于任何可以转换为`String`的类型。

```Rust
fn ok<T: ToString>(s: T) -> CliResult {
    Ok(s.to_string())
}

fn err<T: ToString>(s: T) -> CliResult {
    Err(s.to_string())
}
```

最后，主程序。看看`ok(answer)`如何工作 - 试因为整数知道如何将自己转换为字符串!

```Rust
use std::error::Error;

fn main() {
    println!("Welcome to the Interactive Prompt! ");

    struct Data {
        answer: i32
    }

    let mut cli = Cli::new(Data{answer: 42});

    cli.cmd("go",|data,args| {
        if args.len() == 0 { return err("need 1 argument"); }
        data.answer = match args[0].parse::<i32>() {
            Ok(n) => n,
            Err(e) => return err(e.description())
        };
        println!("got {:?}", args);
        ok(data.answer)
    });

    cli.cmd("show",|data,_| {
        ok(data.answer)
    });

    cli.go();
}
```

这里的错误处理有点笨拙，我们稍后会看到如何在这种情况下，使用问号运算符。 基本上来说，特定的`std::num::ParseIntError`错误实现`std::error::Error` trait，为了使用`description`方法，要导入该 trait - Rust 在 trait 是可见的情况下，才让 它们 运作。

一次行动:

```
Welcome to the Interactive Prompt!
go 32
got["32"]
Ok("32")
show
Ok("32")
goop one two three
Err("no such command")
go 42 one two three
got["42", "one", "two", "three"]
Ok("42")
go boo!
Err("invalid digit found in string")
```

以下是一些供您尝试的明显改进。 首先，如果我们给到`cmd`第二参数是帮助行，那么我们可以存储这些帮助行，并自动执行一个"help"命令。 其次，有一些命令编辑和历史是 _非常_ 方便的，所以从 Cargo 的库中使用[rustyline](https://crates.io/crates/rustyline)crate。
