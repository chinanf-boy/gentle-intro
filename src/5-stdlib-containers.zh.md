# 标准库容器

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [阅读文档](#%E9%98%85%E8%AF%BB%E6%96%87%E6%A1%A3)
- [Maps](#maps)
- [示例: 计算单词](#%E7%A4%BA%E4%BE%8B-%E8%AE%A1%E7%AE%97%E5%8D%95%E8%AF%8D)
- [Sets](#sets)
- [示例: 交互式命令处理](#%E7%A4%BA%E4%BE%8B-%E4%BA%A4%E4%BA%92%E5%BC%8F%E5%91%BD%E4%BB%A4%E5%A4%84%E7%90%86)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 阅读文档

在本节中,我将简要介绍 Rust 标准库的一些常见部分. 文档非常好,但有一点讲解和一些例子总是有用的.

最初,阅读 Rust 文档可能很具挑战性,所以我会经历`Vec`举个例子. 一个有用的提示是勾选'[-]'框来折叠文档.(如果使用下载标准库源代码`rustup component add rust-src` 一个 '[src]'链接将出现在此旁边.)这可以让您全面了解所有可用的方法.

首先要注意的是 _并非所有可能的方法_ 被定义`Vec`本身. 它们是(大部分) 可改变 vec 的方法,例如`push`. 有些方法仅适用于类型匹配某些约束的 Vec.例如,你只能打电话`dedup`(删除重复项) ,如果这个类型确实是可以相互比较的东西. 有多个`impl`定义的块`Vec`针对不同类型的约束.

然后是非常特殊的关系`Vec<T>`和`&[T]`. 任何在 切片 上工作的方法也可以直接在 vec 上工作,而不必明确地使用`as_slice`方法. 这种关系表达为`Deref<Target=[T]>`. 当你通过引用预计会发生 切片 的东西来传递一个 Vec 时,这也会起作用 - 这是类型之间自动转换的少数几个地方之一. 所以像 切片 方法`first`,它可能 - 返回对第一个元素的引用,或者`last`,也为 Vec 工作. 许多方法与相应的字符串方法类似,所以就是这样`split_at`为了在索引处获得一对 切片 ,`starts_with`检查 vec 是否以值序列开始,并且`contains`检查 vec 是否包含特定值.

没有`search`方法来查找特定值的索引,但这里有一条经验法则: 如果在容器上找不到方法,请在 迭代器 上查找一个方法:

```Rust
    let v = vec![10,20,30,40,50];
    assert_eq!(v.iter().position(|&i| i == 30).unwrap(), 2);
```

(该`&`是因为这是一个 迭代器 _引用 _- 或者你可以说`*i== 30`为了比较).

同样,没有`map`对 vec 的方法,因为`iter().map(...).collect()`也会做这项工作. Rust 不喜欢不必要地分配 - 通常你不需要这样的结果`map`作为实际分配的 Vec.

所以我建议你熟悉所有的 迭代器 方法,因为它们对编写好的 Rust 代码至关重要,而不必一直写出循环. 与往常一样,编写小程序来探索 迭代器 方法,而不是在更复杂的程序中与它们搏斗.

该`Vec<T>`和`&[T]`方法之后是共同的 trait : Vec 知道如何进行自己的调试显示(但只有在元素实现时才是如此) `Debug`).同样,如果它们的元素是可克隆的,它们是可克隆的. 他们执行`Drop`当 vec 最终死亡时发生这种情况;内存被释放,并且所有元素也被丢弃.

该`Extend` trait 意味着来自 迭代器 的值可以被添加到一个没有循环的 Vec 中:

```Rust
v.extend([60,70,80].iter());
let mut strings = vec!["hello".to_string(), "dolly".to_string()];
strings.extend(["you","are","fine"].iter().map(|s| s.to_string()));
```

还有`FromIterator`,它可以让 vec _初始化{constructed}_ 来自 迭代器 .(迭代器 `collect`方法倾向于此).

任何容器都需要可迭代. 回想一下有[三种 迭代器 ](2-structs-enums-lifetimes.html#the-three-kinds-of-iterators)

```Rust
for x in v {...} // returns T, consumes v
for x in &v {...} // returns &T
for x in &mut v {...} // returns &mut T
```

该`for`声明依赖于`IntoIterator`特质,实际上有三种实现.

然后是索引,由`Index`控制(从 vec 中读取) 和`IndexMut`(修改一个 Vec). 有很多可能性,因为还有分片索引,就像`v[0..2]`,返回这些返回片,以及平原`v[0]`它返回对第一个元素的引用.

这里有一些实现`From` trait.例如`Vec::from("hello".to_string())`会给你一个字符串底层字节的 Vec `Vec<u8>`. 现在,已经有一种方法`into_bytes`在`String`上,为什么冗余?有多种方式来做同样的事情似乎令人困惑. 但是这是必要的,因为显式 trait 使泛型方法成为可能.

有时候 Rust 类型系统的局限性会让事情变得笨拙. 这里的一个例子是如何`PartialEq`是 _分别_ 定义为最大尺寸为 32 的阵列!(这会变得更好.)这可以方便地将 Vec 与数组进行比较,但要注意大小限制.

还有[隐藏的宝石](http://xion.io/post/code/ Rust -iter-patterns.html)深埋在文档中. 正如 Karol Kuczmarski 所说: "因为说实话: 没有人滚动那么远". 如何处理 迭代器 中的错误? 假设你映射了一些可能失败并返回的操作`Result`,然后想要收集结果:

```Rust
fn main() {
    let nums =["5","52","65"];
    let iter = nums.iter().map(|s| s.parse::<i32>());
    let converted: Vec<_> = iter.collect();
    println!("{:?}",converted);
}
//[Ok(5), Ok(52), Ok(65)]
```

很公平,但现在你必须小心地解开这些错误! 但是,如果你要求 vec 的话, Rust 已经知道如何做正确的事情 _包括_ 在一个`Result`- 也就是说,无论是 vec 还是错误:

```Rust
    let converted: Result<Vec<_>,_> = iter.collect();
//Ok([5, 52, 65])
```

如果转换错误?然后你会得到`Err`遇到第一个错误. 这是非常灵活的一个很好的例子 `collect`是.(这里的符号可能会吓人 -`Vec<_>`意味着"这是一个 Vec ,为我制定出实际的类型`和`Result&lt;vec&lt;_>_>还要求 Rust 计算错误类型.)

所以有一个 _批量{lot}_ 文档中的详细信息. 但它肯定比 C ++文档所说的更清晰`的std::vec`

> 对元素施加的要求取决于对容器执行的实际操作. 一般来说,要求元素类型是完整类型并且符合 Erasable 的要求,但是许多成员函数会施加更严格的要求.

用 C ++,你是独立的. rust 的清晰度一开始令人望而生畏,但当你学习阅读约束条件时,你将确切地知道任何特定的方法`Vec`需要.

我建议你使用源代码`rustup component add rust-src`,因为标准库源代码非常易读,并且方法实现通常不如方法声明那么可怕.

## Maps

_Maps_(有时叫 _关联数组_ 要么 _dicts{字典}_ ); 让你查看与一个相关的值 _key{key}_. 这不是一个真正的花式概念,可以用一系列元组来完成:

```Rust
    let entries =[("one","eins"),("two","zwei"),("three","drei")];

    if let Some(val) = entries.iter().find(|t| t.0 == "two") {
        assert_eq!(val.1,"zwei");
    }
```

这对于小 map 来说很好,只需要为 key 定义相等,但搜索需要线性时间 - 与 map 大小成比例.

一个`HashMap`有一个更好 _批量{lot}_ 要搜索的 key/值 对的列表:

```Rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("one","eins");
map.insert("two","zwei");
map.insert("three","drei");

assert_eq!(map.contains_key("two"), true);
assert_eq!(map.get("two"), Some(&"zwei"));
```

`&"zwei"`?这是因为`get`返回一个 _引用_ 到价值,而不是价值本身. 这里的值类型是`&str`,所以我们得到一个`&&str`. 一般来说它 _具有_ 作为引用,因为我们不能 _移动{move}_ 一种超出其拥有类型的价值.

`get_mut`就好像`get`但返回一个可能的可变引用. 这里我们有一个从字符串到整数的映射,并希望更新 key'two'的值:

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

请注意,获取可写入的引用发生在它自己的块中 - 否则,我们将有一个可变的借位持续到结束,然后 Rust 不会允许您从`map`再次借与`map.get("two")`;它不能允许任何可读的引用,同时已经有一个可写引用的范围.(如果是这样,它不能保证那些可读的引用将保持有效.)所以解决方案是确保可变借用不会持续很长时间.

这不是最优雅的 API,但我们不能抛弃任何可能的错误. Python 会抛出一个异常,而 C ++只会创建一个默认值.(这很方便,但偷偷摸摸;容易忘记的价格 a*map[ "two"]`总是返回一个整数是我们不能区分 zero 和 '未找到' 之间的区别, *加\_ 一个额外的条目被创建!)

没有人只是运行 `unwrap`, 除了例子中除外. 但是,您看到的大多数 Rust 代码都由一些独立的示例组成!比赛发生的可能性更大: 我们可以遍历 key/值对,但不是以任何特定的顺序.

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

也有

```Rust
for(k,v) in map.iter() {
    println!("key {} value {}", k,v);
}
// key one value eins
// key three value drei
// key two value zwei
```

按`key`和`值`方法分别通过 key 和值返回 迭代器 ,这使得创建值的 Vec 变得容易.

## 示例: 计算单词

与文本有关的一个有趣的事情是计数字频率.

将文本分解为单词很简单`split_whitespace`,但是我们真的要尊重标点符号. 所以这些词应该被定义为只包含字母字符. 这些词也需要作为小写字母进行比较. 在 map 上做一个可变的查找很简单,但是处理查找失败的情况有点尴尬.

幸运的是,有一种更新 map 值的方式:

```Rust
let mut map = HashMap::new();

for s in text.split(|c: char| ! c.is_alphabetic()) {
    let word = s.to_lowercase();
    let mut count = map.entry(word).or_insert(0);
    *count += 1;
}
```

如果没有对应于某个单词的现有计数,那么让我们为该单词和单词创建一个包含零和 _插{insert}_ 它进入 map.它正是 C ++映射所做的,除非它明确地完成并且没有偷偷摸摸.

这段代码中只有一个显式类型,这就是`char`因为弦的怪癖而需要`模式{Pattern}`使用的特质`split`. 但我们可以推断出 key 类型是`String`和 value 类型是`i32`.

运用[福尔摩斯历险记](http://www.gutenberg.org/cache/epub/1661/pg1661.txt)来自古腾堡计划,我们可以更彻底地进行测试. 唯一字词的总数(`map.len()`) 是 8071.

如何找到最常见的二十个单词?首先,将映射转换为(key, value) 元组的 Vec .(因为我们使用了,所以这消耗了 map `into_iter`.)

```Rust
let mut entries: Vec<_> = map.into_iter().collect();
```

接下来我们可以按降序排列. `sort_by`期待的结果`cmp`方法来自于`Ord` trait ,它是由整型值类型实现的:

```Rust
    entries.sort_by(|a,b| b.1.cmp(&a.1));
```

最后打印出前 20 个条目:

```Rust
    for e in entries.iter().take(20) {
        println!("{} {}", e.0, e.1);
    }
```

(好吧,你 _可以_ 只是循环`0..20`并在这里给这个 Vec 编制索引 - 这没有错,只是有点不经意 - 而且对于大型迭代来说可能更昂贵.)

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

有点惊喜 - 那个空的文本是什么?这是因为`split`适用于单字符分隔符,因此任何标点符号或额外空格都会导致新的分割.

## Sets

Sets 是 map ,您只关心关 key 字,而不关联任何关联的值. 所以`inserts`只需要一个值,然后使用`contains`用于测试一个值是否在一个集合中.

像所有容器一样,您可以创建一个`HashSet`来自 迭代器.这正是什么 `collect`确实,一旦你给了它必要的类型提示.

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

注意(如预期的那样) 重复插入同一个 key 不起作用,并且集合中值的顺序并不重要.

没有通常的操作,它们不会被设置:

```Rust
let fruit = make_set("apple orange pear");
let colours = make_set("brown purple orange yellow");

for c in fruit.intersection(&colours) {
    println!("{:?}",c);
}
// "orange"
```

他们都创建 迭代器 ,并且可以使用 `collect`使这些成为集合.

这是一个快捷方式,就像我们为 vec 定义的那样:

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

就像所有的 Rust 泛型一样,你需要限制类型 - 这只能用于理解平等的类型(`Eq`) 并且为其存在"散列函数"(`Hash`).请记住,没有 _类型_ 叫`迭代器`,所以`I`代表任何类型 _impl_ `迭代器`.

这种在标准库类型上实现我们自己的方法的技术似乎有点过于强大,但是同样存在规则. 我们只能为自己的特质做到这一点. 如果结构和 trait 来自同一个箱子(特别是 stdlib ) ,那么这种实现将不被允许. 通过这种方式,您可以避免造成混乱.

在祝贺自己如此聪明,方便的捷径之前,您应该意识到后果. 如果`make_set`是这样写的,所以这些是拥有字符串的集合,然后是实际类型`相交{intersect}`可能会惊喜:

```Rust
fn make_set(words: &str) -> HashSet<String> {
    words.split_whitespace().map(|s| s.to_string()).collect()
}
...
// intersect is HashSet<&String>!
let intersect = fruit.intersection(&colours).to_set();
```

除此之外, Rust 不会突然开始复制拥有的字符串. `相交`contains 单个`&String`借来`fruit`. 我可以向你保证,当你开始修补生命时,这会给你带来麻烦!更好的解决方案是使用 迭代器 `clone`方法来创建交集的所有字符串副本.

```Rust
// intersect is HashSet<String> - much better
let intersect = fruit.intersection(&colours).cloned().to_set();
```

一个更强大的定义`to_set`可能`self.cloned().collect()`,我邀请您试用.

## 示例: 交互式命令处理

与程序进行交互式会话通常很有用. 每行都被读入并分成单词;该命令在第一个单词上查找,其余单词作为参数传递给该命令.

自然实现是从命令名到闭包的映射. 但是,我们如何存储关闭,因为它们都会有不同的大小? 拳击他们将他们复制到堆上:

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

我们在第二次推动时遇到了非常明显的错误:

      = note: expected type `[closure@closure4.rs:4:21: 4:28]`
      = note:    found type `[closure@closure4.rs:5:21: 5:28]`
    note: no two closures, even if identical, have the same type

`rustc`导出了一个过于具体的类型,所以有必要强制该 Vec 具有该类型 _盒装 trait 类型_ 在事情刚刚开始之前:

```Rust
    let mut v: Vec<Box<Fn(f64)->f64>> = Vec::new();
```

我们现在可以使用相同的技巧,并将这些盒装封口保存在一个`HashMap`. 我们仍然需要警惕终身,因为关闭可以从他们的环境中借用.

作为第一个制作它们很诱人`FnMut`- 也就是说,他们可以修改任何捕获的变量. 但是我们会有不止一个命令,每个命令都有自己的闭包,所以你不能随意借用相同的变量.

所以关闭通过了*可变引用*作为一个参数,加上一段字符串 切片(`&[&str]`) 代表命令参数. 他们会返回一些`Result`- 我们会用`String`首先是错误.

`D`是数据类型,可以是任何大小的数据.

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
        self.callbacks.insert(name.to_string(),Box::new(callback));
    }
```

`cmd`传递一个名称和任何与我们的签名相匹配的封闭,这个签名被装箱并输入到 map 中. `Fn`意味着我们的关闭借用他们的环境,但不能修改它. 它是声明比实际实现更可怕的通用方法之一!忘记明确的生命是一个常见的错误 - Rust 不会让我们忘记这些封闭只限于他们的环境!

现在阅读和运行命令:

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

这是非常简单明了的 - 将行分成单词作为 vec ,查找 map 中的第一个单词,并用存储的可变数据和其余单词调用闭包. 空行会被忽略,不会被视为错误.

接下来,让我们定义一些帮助函数,使我们的闭包更容易返回正确和不正确的结果. 有一个 有点 _小_ 聪明在继续;它们是通用函数,适用于任何可以转换为的类型`String`.

```Rust
fn ok<T: ToString>(s: T) -> CliResult {
    Ok(s.to_string())
}

fn err<T: ToString>(s: T) -> CliResult {
    Err(s.to_string())
}
```

最后,主程序. 看看如何`OK(答案)`作品 - 因为整数知道如何将自己转换为字符串!

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

这里的错误处理有点笨拙,我们稍后会看到如何在这种情况下使用问号运算符. 基本上,特定的错误`std::num::ParseIntError`实现这个特点 `std::error::Error`,我们必须把它带入使用范围`描述`方法 - 铁锈不让 trait 运作,除非它们是可见的.

在行动中:

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

以下是一些明显的改进,供您尝试. 首先,如果我们给`cmd`第三个参数是帮助行,然后我们可以存储这些帮助行并自动执行"帮助"命令. 其次,有一些命令编辑和历史是 _非常_ 方便,所以使用[ Rust yline](https://crates.io/crates/ Rust yline)从 Cargo 的库中.
