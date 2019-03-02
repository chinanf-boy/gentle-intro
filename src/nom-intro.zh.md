## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [用 nom 解析文本](#%E7%94%A8-nom-%E8%A7%A3%E6%9E%90%E6%96%87%E6%9C%AC)
- [一个 Nom 解析器返回什么](#%E4%B8%80%E4%B8%AA-nom-%E8%A7%A3%E6%9E%90%E5%99%A8%E8%BF%94%E5%9B%9E%E4%BB%80%E4%B9%88)
- [合并解析器](#%E5%90%88%E5%B9%B6%E8%A7%A3%E6%9E%90%E5%99%A8)
- [解析数字](#%E8%A7%A3%E6%9E%90%E6%95%B0%E5%AD%97)
- [多个匹配进行操作](#%E5%A4%9A%E4%B8%AA%E5%8C%B9%E9%85%8D%E8%BF%9B%E8%A1%8C%E6%93%8D%E4%BD%9C)
- [解析算术表达式](#%E8%A7%A3%E6%9E%90%E7%AE%97%E6%9C%AF%E8%A1%A8%E8%BE%BE%E5%BC%8F)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 用 nom 解析文本

[nom](https://github.com/Geal/nom)，[ (文档在这里) ](https://docs.rs/nom)是 Rust 的解析器库，它非常值得新手投资。

如果你必须解析一个已知的数据格式，比如 CSV 或者 JSON，那么最好使用一个专门库，像[Rust CSV](https://github.com/BurntSushi/rust-csv)或者[第 4 节](4-modules.zh.html#cargo)讨论的 JSON 库。

同样，对配置文件使用专用的解析器，比如[ini](https://docs.rs/rust-ini/0.10.0/ini/)要么[toml](http://alexcrichton.com/toml-rs/toml/index.html)。 (后一个特别酷，因为它与 Serde 框架结合在一起，就像我们看到的[serde_json](https://docs.rs/serde_json)。

但是如果文本不规则，或者某种格式，那么你需要扫描那些文本，但不是通过写很多乏味的字符串处理代码。 常建议去看看[正则表达式](https://github.com/rust-lang/regex)，但认识久后，会感到沮丧，因正则表达式可能并不透明。 nom 提供了一种解析文本的方法。足够强大，大体上讲就是，组合更简单的解析器。 正则表达式有其局限性，例如，[使用正则表达式来解析 HTML](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags)，不怎么行吧，但是你 _可以_ 使用 nom 解析 HTML。 如果你有兴趣编写自己的编程语言，nom 是一个很好的借鉴，作为领略这条艰难的道路的开端。

有一些用于学习 nom 的优秀教程，但我想从 hello-world 级开始建立一些初步的熟悉感。 您需要了解的基本知识 - 首先，nom 一直是宏，其次，nom 倾向于使用切片 ，而不是字符串。 第一要点，是你必须特别小心才能获得 nom 表达式，因为错误信息不会很友善。 第二要点是 nom 可以用于 _任何_ 数据格式，而不仅仅是文本。 人们使用 nom 解码二进制协议和文件头。它也可以在 UTF-8 以外的编码中，与"文本"合作。

nom 的最新版本与字符串切片工作良好，不过，您需要使用以`_s`结尾的宏.

```rust
#[macro_use]
extern crate nom;

named!(get_greeting<&str,&str>,
    tag_s!("hi")
);

fn main() {
    let res = get_greeting("hi there");
    println!("{:?}",res);
}
// Done(" there", "hi")
```

该`named!`宏会创建函数，函数需要一些输入类型(`&[u8]`默认) 并将第二个类型返回到尖括号中。 `tag_s!`匹配字符流中的一个字面字符串，其值是表示该文字的字符串切片。(如果你想与`&[u8]`合作，那用`tag!`宏。)

我们用一个`&str`，调用定义的`get_greeting`解析器，并返回一个[IResult](http://rust.unhandledexpression.com/Nom/enum.IResult.html)。实际上，我们得到了匹配的值。

看看" there" - 这是匹配后剩下的字符串切片。

我们想忽略空格。`tag_s!`包进`ws!`，我们就可以在空格,制表符或换行符的任何位置匹配"hi":

```rust
named!(get_greeting<&str,&str>,
    ws!(tag_s!("hi"))
);

fn main() {
    let res = get_greeting("hi there");
    println!("{:?}",res);
}
// Done("there", "hi")
```

结果就像之前的"hi"，只不过剩下的字符串是"there"!，空格已被跳过。

很好地匹配了"hi"，尽管这还不是很有用。

让我们匹配"hi"_或_"bye"。 `alt!`宏 ("备选项") 采用`|`符号分割解析器表达式，这样就可以匹配其中的 _任何_ 。 请注意，您可以在这里使用空格来使解析器函数更易于阅读:

```rust
named!(get_greeting<&str>,
    ws!(alt!(tag_s!("hi") | tag_s!("bye")))
);
println!("{:?}", get_greeting(" hi "));
println!("{:?}", get_greeting(" bye "));
println!("{:?}", get_greeting("  hola "));
// Done("", "hi")
// Done("", "bye")
// Error(Alt)
```

最后一场匹配失败了，因为没有其他备选项能匹配"hola".

显然，我们需要了解`IResult`类型多一点，但首先让我们比较这与正则表达式的解决方案:

```rust
    let greetings = Regex::new(r"\s*(hi|bye)\s*").expect("bad regex"); // (hi|bye) 两种可能性
    let caps = greetings.captures(" hi ").expect("match failed");
    println!("{:?}",caps);
// Captures({0: Some(" hi "), 1: Some("hi")})
```

正则表达式肯定更 _紧凑{compact}_! 我们需要将`()`放在，由`|`分隔的两种可能性中, 所以我们让 greeting _捕获(captures)_ 些什么或者没有。第一个结果是整个字符串，第二个是匹配的捕获。 ('|'是正则表达式中所谓的'备选'操作符，这是`alt!`宏语法的动机名。)

但，这是一个非常简单的正则表达式，它们很快就会变得复杂。因作为一种文本微语言，你必须转义重要的字符如`*`和`(`。 如果我想匹配"(hi)"或"(bye)"，则正则表达式变为 `\s*\((hi|bye)\)\s*`，但是 nom 解析器只是变成了`alt!(tag_s!("(hi)") | tag_s!("(bye)"))`。

这也是一个重量级的依赖。在这款相当微弱的 i5 笔记本电脑上，nom 的例子大约需要 0.55 秒的时间编译完成，这并不比"Hello world"慢多少。 但正则表达式的例子大约需要 0.90s。 该 Nom 示例的剥离版本生成的可执行文件约为 0.3Mb (与静态链接的 Rust 程序大约一样)，而正则表达式为 0.8Mb。

## 一个 Nom 解析器返回什么

<!-- HERE -->

[IResult](http://rust.unhandledexpression.com/nom/enum.IResult.html)与标准`Result`类型有一个有趣的区别 - 有三种可能性:

- `Done`- 成功 - 您将得到结果和剩余的字节
- `Error`- 未能解析 - 你得到一个错误
- `不完全{Imcomplete}`- 需要更多数据

我们可以写一个`dump`泛型函数，处理可以调试打印的任何返回值。 也说明下`to_result`方法会返回一个常规`Result`- 这可能是大多数情况下，用到的方法，因它不是返回值，就是返回一个错误。

```rust
#[macro_use]
extern crate nom;
use nom::IResult;
use std::str::from_utf8;
use std::fmt::Debug;

fn dump<T: Debug>(res: IResult<&str,T>) {
    match res {
      IResult::Done(rest, value) => {println!("Done {:?} {:?}",rest,value)},
      IResult::Error(err) => {println!("Err {:?}",err)},
      IResult::Incomplete(needed) => {println!("Needed {:?}",needed)}
    }
}


fn main() {
    named!(get_greeting<&str,&str>,
        ws!(
            alt!( tag_s!("hi") | tag_s!("bye"))
        )
    );

    dump(get_greeting(" hi "));
    dump(get_greeting(" bye hi"));
    dump(get_greeting("  hola "));

    println!("result {:?}", get_greeting(" bye  ").to_result());
}
// Done Ok("") "hi"
// Done Ok("hi") "bye"
// Err Alt
// result Ok("bye")
```

解析器返回任何未解析的文本，并且能够表明没有足够的输入字符，对 流{stream} 解析非常有用。常见情况下，`to_result`会是你的朋友。

## 合并解析器

让我们继续 greeting(问候) 示例，并设想问候语包含"hi"或"bye"，再加上一个名字。`Nom::alpha`匹配一系列字母字符。`pair!`宏将收集两个匹配解析器的结果，作成一个元组:

```rust
    named!(full_greeting<&str,(&str,&str)>,
        pair!(
            get_greeting,
            nom::alpha
        )
    );

    println!("result {:?}", full_greeting(" hi Bob  ").to_result());
// result Ok(("hi", "Bob"))
```

现在，进一步想象，这个 greeter 有点害羞或不知道其他人的名字: 让我们把名字变成可选。 自然而然，元组的第二个值变成了`Option`。

```rust
    named!(full_greeting<&str, (&str,Option<&str>)>,
        pair!(
            get_greeting,
            opt!(nom::alpha)
        )
    );

    println!("result {:?}", full_greeting(" hi Bob  ").to_result());
    println!("result {:?}", full_greeting(" bye ?").to_result());
// result Ok(("hi", Some("Bob")))
// result Ok(("bye", None))
```

留意下，将现有的问候语解析器与，一个名称挑选的解析器合并起来很简单，结果就是名字是可选的。 见识到 nom 的强大力量了吧，这也是它被称为"解析器组合库"的原因。 您可以从更简单的解析器构建复杂的解析器，您可以单独测试它们。 (在这一点上，等价的正则表达式开始看起来像一个 Perl 程序: 正则表达式不能很好地合并。 )

但是，我们还不能葛优瘫! `full_greeting(" bye")`可能会有`Imcomplete`错误。 Nom 知道"bye"后面可能会有一个名字，并希望我们给它更多的数据。 _流_ 解析器工作的时候到了，你可以把文件按块喂它吃，但是在这里我们需要告诉 nom 输入已完成。

```rust
    named!(full_greeting<&str,(&str,Option<&str>)>,
        pair!(
            get_greeting,
            opt!(complete!(nom::alpha)) // 已完成
        )
    );

    println!("result {:?}", full_greeting(" bye ").to_result());
// result Ok(("bye", None))
```

## 解析数字

nom 提供了一个`digit`函数，它与一系列数字相匹配。 所以我们使用`map!`，将字符串转换为整数，并返回完整的`Result`类型。

```rust
use nom::digit;
use std::str::FromStr;
use std::num::ParseIntError;

named!(int8 <&str, Result<i8,ParseIntError>>,
    map!(digit, FromStr::from_str)
);

named!(int32 <&str, Result<i32,ParseIntError>>,
    map!(digit, FromStr::from_str)
);

println!("{:?}", int8("120"));
println!("{:?}", int8("1200"));
println!("{:?}", int8("x120"));
println!("{:?}", int32("1202"));

// Done("", Ok(120))
// Done("", Err(ParseIntError { kind: Overflow }))
// Error(Digit)
// Done("", Ok(1202))
```

所以我们得到的是，一个解析器的`IResult`，包含转换而来的`Result`- 当然，在这里失败的原因不止一种。 请注意，我们的转换函数的主体具有完全相同的代码; 而实际转换取决于函数的返回类型。

整数可能有标志。 我们可以将整数作为一对来捕获，其中的第一个值可能是一个符号，第二个值可能是后面的任何数字。

考虑:

```rust
named!(signed_digits<&str, (Option<&str>,&str)>,
    pair!(
        opt!(alt!(tag_s!("+") | tag_s!("-"))),  // 会是一个标志吗?
        digit
    )
);

println!("signed {:?}", signed_digits("4"));
println!("signed {:?}", signed_digits("+12"));
// signed Done("", (None, "4"))
// signed Done("", (Some("+"), "12"))
```

当我们对中间结果不感兴趣时，只需要所有的匹配输入，那`recognize!`是你需要的。

```rust
named!(maybe_signed_digits<&str,&str>,
    recognize!(signed_digits)
);

println!("signed {:?}", maybe_signed_digits("+12"));
// signed Done("", "+12")
```

使用这种技术，我们可以识别浮点数。 同样，我们在所有这些匹配项上，将字节切片映射到字符串切片。 `tuple!`是泛型化的`pair!`，尽管我们对这里生成的元组不感兴趣。 `complete!`是需要解决不完整的问候时的相同问题 - "12"是没有可选浮点数部分的有效数字。

```rust
named!(floating_point<&str,&str>,
    recognize!(
        tuple!(
            maybe_signed_digits,
            opt!(complete!(pair!(
                tag_s!("."),
                digit
            ))),
            opt!(complete!(pair!(
                alt!(tag_s!("e") | tag_s!("E")),
                maybe_signed_digits
            )))
        )
    )
);
```

通过定义一个宏小助手，搞一些测试。 如果`floating_point`匹配它给出的所有字符串，那测试通过。

```rust
macro_rules! nom_eq {
    ($p:expr,$e:expr) => (
        assert_eq!($p($e).to_result().unwrap(), $e)
    )
}

nom_eq!(floating_point, "+2343");
nom_eq!(floating_point, "-2343");
nom_eq!(floating_point, "2343");
nom_eq!(floating_point, "2343.23");
nom_eq!(floating_point, "2e20");
nom_eq!(floating_point, "2.0e-6");
```

(虽然有时候，感觉宏有点 _小_ 肮脏，但让你的测试漂亮是件好事。)

然后我们可以解析和转换浮点数。在这里，是不顾一切错误的乌托邦式例子:

```rust
    named!(float64<f64>,
        map_res!(floating_point, FromStr::from_str)
    );
```

注意的是，要逐步构建复杂的解析器，那么首先，单独测试每个部分。这是解析器组合器相较于正则表达式的强大优势。这是分而治之的经典编程策略。

## 多个匹配进行操作

我们与`pairs!`和`tuple!`见过面了，它将固定数量的匹配捕获项，作成 Rust 元组。

这有`many0`和`many1`- 他们都捕获无限数量的匹配项，作成 Vec。 不同的是，前一个可能会捕获"零或多个"，后一个则是"一个或多个" (如正则表达式`*`与`+`之间的差异) 所以`many1!(ws!(float64))`会解析"1 2 3"到`vec![1.0,2.0,3.0]`，但会在空字符串上会失败。

`fold_many0`是一个 _递算_ 操作。使用二元运算符将匹配值组合为单个值。例如，这就是 Rust 开发者 以前如何对迭代器进行求和`sum`加入; 这个`fold`从一个初始值 (这里是零) 开始，启动 _累加器(acc)_ ，并`+`迭代器的迭代值`v`，并返回给`acc`,继续。

> 流程就像`acc = 0 + v0`(第一次)，`acc = v0 + v1`(第二次)...

```rust
    let res = [1,2,3].iter().fold(0,|acc,v| acc + v);
    println!("{}",res);
    // 6
```

以下是 nom 等价物:

```rust
    named!(fold_sum<&str,f64>,
        fold_many1!(
            ws!(float64),
            0.0,
            |acc, v| acc + v
        )
    );

    println!("fold {}", fold_sum("1 2 3").to_result().unwrap());
    //fold 6
```

到目前为止，我们必须捕获每个表达式，或者只是用`recognize!`抓住所有匹配的字节:

```rust
    named!(pointf<(f64,&[u8],f64)>,
        tuple!(
            float64,
            tag_s!(","),
            float64
        )
    );

    println!("got {:?}", nom_res!(pointf,"20,52.2").unwrap());
 //got (20, ",", 52.2)
```

对于更复杂的表达式，捕获所有解析器的结果，会导致相当不整洁的类型!我们可以做得更好.

`do_parse!`让你只提取你感兴趣的值- 感兴趣的匹配项用`>>`分割: 格式是 `name:parser}`。最后，括号里有一个代码块。

```rust
    #[derive(Debug)]
    struct Point {
        x: f64,
        y: f64
    }

    named!(pointf<Point>,
        do_parse!(
            first: float64 >>
            tag_s!(",") >>
            second: float64
            >>
            (Point{x: first, y: second}) // first  second 是 临时值
        )
    );

    println!("got {:?}", nom_res!(pointf,"20,52.2").unwrap());
// got Point { x: 20, y: 52.2 }
```

对标签值 (就是那个逗号)不感兴趣，但我们将两个浮点值分配给用于构建结构的临时值。最后的代码可以是任何 Rust 表达式，随你。

## 解析算术表达式

随着必要的知识建立，我们可以做简单的算术表达式。

这是用正则表达式，无法真正完成的一个很好的例子。

这个想法是从下往上建立表达式。表达式由 _terms_ 组成，如加或减。 terms 由 _factors_ 组成，它们相乘或除。 和（现在）factor 只是浮点数：

```rust
    named!(factor<f64>,
        ws!(float64)
    );

    named!(term<&str,f64>, do_parse!(
        init: factor >>
        res: fold_many0!(
            tuple!(
                alt!(tag_s!("*") | tag_s!("/")),
                factor
            ),
            init,
            |acc, v:(_,f64)| {
                if v.0 == "*" {acc * v.1} else {acc / v.1}
            }
        )
        >> (res)
    ));

    named!(expr<&str,f64>, do_parse!(
        init: term >>
        res: fold_many0!(
            tuple!(
                alt!(tag_s!("+") | tag_s!("-")),
                term
            ),
            init,
            |acc, v:(_,f64)| {
                if v.0 == "+" {acc + v.1} else {acc - v.1}
            }
        )
        >> (res)
    ));
```

这更准确地表达了我们的定义 - 一个表达式至少包含一个 term，和零或多个加减项。

我们不`collect`它们，而用适当的 _fold_ 操作。 (这是 Rust 不能很好地处理表达式类型的情况之一，所以我们需要一个类型提示。) 这样做会建立正确的 _运算符优先级_ -`*`总是比`+`优先等等。我们在这里需要浮点数断言，而正好[有一个箱](http://brendanzab.github.io/approx/approx/index.html).

将`approx ="0.1.1"`，添加到您的 Cargo.toml 中，就可以开始了:

```rust
#[macro_use]
extern crate approx;
...
    assert_relative_eq!(fold_sum("1 2 3").to_result().unwrap(), 6.0);
```

我们来定义一个方便的小宏，来测试。`stringify!`将表达式转换为，我们可以输入`expr`的字符串字面量, 然后将测试得到的值，与 Rust 的执行结果进行比较。

```rust
    macro_rules! expr_eq {
        ($e:expr) => (assert_relative_eq!(
            expr(stringify!($e).to_result().unwrap(),
            $e)
        )
    }

    expr_eq!(2.3);
    expr_eq!(2.0 + 3.0 - 4.0);
    expr_eq!(2.0*3.0 - 4.0);
```

这非常酷 - 只需几行即可获得 表达式评估器! 但它能变得更好。 我们在`因素{factor}`解析器，增加了一个数字的替代方案 - 能包含在括号内的表达式:

```rust
    named!(factor<&str,f64>,
        alt!(
            ws!(float64) |
            ws!(delimited!( tag_s!("("), expr, tag_s!(")") ))
        )
    );

    expr_eq!(2.2*(1.1 + 4.5)/3.4);
    expr_eq!((1.0 + 2.0)*(3.0 + 4.0*(5.0 + 6.0)));
```

最厉害的是现在，能 _递归_ 定义这堆表达式!

`delimited!`的特别魔力，在于括号可以嵌套 - nom 确保括号匹配。

我们现在已经拥有，超越正则表达式的能力了，0.5Mb 的剥离版可执行文件仍然是"hello world"正则表达式程序的一半大小。
