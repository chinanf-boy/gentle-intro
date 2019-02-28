## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [用 nom 解析文本](#%E7%94%A8-nom-%E8%A7%A3%E6%9E%90%E6%96%87%E6%9C%AC)
- [一个 nom 解析器返回](#%E4%B8%80%E4%B8%AA-nom-%E8%A7%A3%E6%9E%90%E5%99%A8%E8%BF%94%E5%9B%9E)
- [合并解析器](#%E5%90%88%E5%B9%B6%E8%A7%A3%E6%9E%90%E5%99%A8)
- [分析数字](#%E5%88%86%E6%9E%90%E6%95%B0%E5%AD%97)
- [多个匹配进行操作](#%E5%A4%9A%E4%B8%AA%E5%8C%B9%E9%85%8D%E8%BF%9B%E8%A1%8C%E6%93%8D%E4%BD%9C)
- [解析算术表达式](#%E8%A7%A3%E6%9E%90%E7%AE%97%E6%9C%AF%E8%A1%A8%E8%BE%BE%E5%BC%8F)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 用 nom 解析文本

[nom](https://github.com/Geal/nom),[ (在这里记录) ](https://docs.rs/nom)是 Rust 的解析器库,它非常值得初次投资.

如果你必须解析一个已知的数据格式,比如 CSV 或者 JSON,那么最好使用一个专门的类库[Rust CSV](https://github.com/BurntSushi/rust-csv)或者讨论的 JSON 库[第 4 节](4-modules.html#cargo).

同样,对于配置文件使用专用的解析器[ini](https://docs.rs/rust-ini/0.10.0/ini/)要么[toml](http://alexcrichton.com/toml-rs/toml/index.html). (最后一个特别酷,因为它与 Serde 框架结合在一起,就像我们看到的一样[serde_json](https://docs.rs/serde_json).

但是如果文本不规则,或者某种格式化,那么你需要扫描那些文本而不写很多乏味的字符串处理代码. 建议的去往往是[正则表达式](https://github.com/rust-lang/regex),但充分参与时,正则表达式可能令人沮丧地不透明. nom 提供了一种解析文本的方法,它可以通过组合更简单的解析器来构建. 例如,正则表达式有其局限性[使用正则表达式来解析 HTML](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags)但是你 _可以_ 使用 nom 解析 HTML. 如果你有兴趣编写自己的编程语言,nom 是一个很好的地方,你可以从这条艰难的道路开始.

有一些用于学习 nom 的优秀教程,但我想从 hello-world 级开始建立一些初步的熟悉性. 您需要了解的基本知识 - 首先,nom 一直是宏,其次,nom 倾向于使用 slices ,而不是字符串. 首先意味着你必须特别小心才能获得 nom 表达式,因为错误信息不会很友善. 第二种意思是 nom 可以用于 _任何_ 数据格式,而不仅仅是文本. 人们使用 nom 解码二进制协议和文件头. 它也可以在 UTF-8 以外的编码中使用"文本".

nom 的最新版本与字符串切片一起工作良好,但您需要使用以其结尾的宏`_s`.

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

该`named!`宏创建一些输入类型的函数 (`&[u8]`默认情况下) 并将第二个类型返回到尖括号中. `tag_s!`匹配字符流中的文字字符串,其值是表示该文字的字符串片段. (如果你想与之合作`&[u8]`然后使用`tag!`宏) .

我们称之为定义的解析器`get_greeting`与`&str`并找回一个[IResult](http://rust.unhandledexpression.com/Nom/enum.IResult.html)事实上,我们恢复了匹配值. 看看" there" - 这是匹配后剩下的字符串切片.

我们想忽略空白.

通过只是包装的`tag!`在`ws!`我们可以在空格,制表符或换行符的任何位置匹配"hi":

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

结果就像之前的"hi",剩下的字符串是"there"! 空格已被跳过.

"hi"很好地匹配,尽管这还不是很有用.

让我们匹配或"hi"或"bye". 该`alt!`宏 ("备用") 采用分隔符表达式分隔`|`并匹配 _任何_ 其中. 请注意,您可以在这里使用空格来使解析器函数更易于阅读:

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

最后一场比赛失败了,因为没有其他选择匹配"hola".

显然我们需要了解`IResult`类型多点,但首先让我们比较这与正则表达式的解决方案:

```rust
    let greetings = Regex::new(r"\s*(hi|bye)\s*").expect("bad regex");
    let caps = greetings.captures(" hi ").expect("match failed");
    println!("{:?}",caps);
// Captures({0: Some(" hi "), 1: Some("hi")})
```

正则表达式肯定更多 _紧凑{compact}_!我们需要将'()'放在由'|'分隔的两种可能性中, 所以我们会 _捕获_ greeting 或者没有. 第一个结果是整个字符串,第二个是匹配的捕获. ('|'是正则表达式中的所谓'交替{alternation}'操作符,这是该动机的动机`alt!`宏语法. )

但这是一个非常简单的正则表达式,它们很快就会变得复杂. 作为一种文字迷你语言,你必须逃避重要的人物如`*`和`(`. 如果我想匹配"(hi)"或"(bye)",则正则表达式变为 `\s*\((hi|bye)\)\s*` 但是 nom 解析器只是变成了`alt!(tag_s!("(hi)") | tag_s!("(bye)"))`.

这也是一个重量级的依赖. 在这款相当微弱的 i5 笔记本电脑上,nom 的例子大约需要 0.55 秒的时间进行编译,这并不比"Hello world"多得多. 但正则表达式的例子大约需要 0.90s. nom 示例的剥离版本生成可执行文件约为 0.3Mb (与静态链接的 Rust 程序一样小) ,而正则表达式为 0.8Mb.

## 一个 nom 解析器返回

[IResult](http://rust.unhandledexpression.com/nom/enum.IResult.html)与标准有一个有趣的区别`Result`类型 - 有三种可能性:

- `Done`- 成功 - 您将得到结果和剩余的字节
- `Error`- 未能解析 - 你得到一个错误
- `不完全{Imcomplete}`- 需要更多数据

我们可以写一个通用的`dump`函数处理可以调试打印的任何返回值. 这也说明了`to_result`方法返回一个常规`Result`- 这可能是您将用于大多数情况的方法,因为它返回 返回值或错误.

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

解析器返回任何未解析的文本,并且能够指示它们没有足够的输入字符来决定,对于 流{stream} 解析非常有用. 但通常`to_result`是你的朋友.

## 合并解析器

让我们继续问候示例,并设想问候语包含"hi"或"bye",再加上一个名字. `Nom::alpha`匹配一系列字母字符. 该`pair!`宏将收集匹配两个解析器作为元组的结果:

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

现在,进一步想象 greeter 会有点害羞或不知道任何人的名字: 让我们使名字可选. 自然而然,元组的第二个值变成了`Option`.

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

请注意,将现有的问候语解析器与分析器合并起来很简单,然后很容易使该名称成为可选项. 这是 nom 的强大力量,这也是它被称为"解析器组合库"的原因. 您可以从更简单的解析器构建复杂的解析器,您可以单独测试它们. (在这一点上,等价的正则表达式开始看起来像一个 Perl 程序: 正则表达式不能很好地结合. )

但是,我们还没有回家并且干燥!`full_greeting(" bye")`将会失败`不完全`错误. nom 知 道"bye"后面可能会有一个名字,并希望我们给它更多的数据. 这是怎么一个 _流_ 解析器需要工作,所以你可以按块填充文件块,但是在这里我们需要告诉 nom 输入已完成.

```rust
    named!(full_greeting<&str,(&str,Option<&str>)>,
        pair!(
            get_greeting,
            opt!(complete!(nom::alpha))
        )
    );

    println!("result {:?}", full_greeting(" bye ").to_result());
// result Ok(("bye", None))
```

## 分析数字

nom 提供了一个功能`数字{digit}`它与一系列数字相匹配. 所以我们使用`map!`,将字符串转换为整数,并返回完整的`Result`类型.

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

所以我们得到的是一个解析器`IResult`包含转换`Result`- 当然,在这里失败的方式不止一种. 请注意,我们的转换函数的主体具有完全相同的代码;实际转换取决于函数的返回类型.

整数可能有一个标志. 我们可以将整数作为一对来捕获,其中第一个值可能是一个符号,第二个值可能是后面的任何数字.

考虑:

```rust
named!(signed_digits<&str, (Option<&str>,&str)>,
    pair!(
        opt!(alt!(tag_s!("+") | tag_s!("-"))),  // maybe sign?
        digit
    )
);

println!("signed {:?}", signed_digits("4"));
println!("signed {:?}", signed_digits("+12"));
// signed Done("", (None, "4"))
// signed Done("", (Some("+"), "12"))
```

当我们对中间结果不感兴趣时 ​​,只需要所有的匹配输入`recognize!`是你需要的.

```rust
named!(maybe_signed_digits<&str,&str>,
    recognize!(signed_digits)
);

println!("signed {:?}", maybe_signed_digits("+12"));
// signed Done("", "+12")
```

使用这种技术,我们可以识别浮点数. 我们再次映射到所有这些匹配的字节片的字符串片. `tuple!`是泛化的`pair!`,尽管我们对这里生成的元组不感兴趣. `complete!`是需要解决我们与不完整的问候时使用的相同问题 - "12"是没有可选浮点部分的有效数字.

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

通过定义一个小助手宏,我们得到了一些通过测试. 如果测试通过`floating_point`匹配它给出的所有字符串.

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

(虽然有时候宏感觉到一点 _小_ 肮脏,让你的测试很漂亮是件好事. ) 然后我们可以解析和转换浮点数.

在这里,我会告诫大家,并抛弃错误: 请注意,如何逐步构建复杂的解析器,并首先单独测试每个部分.

```rust
    named!(float64<f64>,
        map_res!(floating_point, FromStr::from_str)
    );
```

这是解析器组合器对正则表达式的强大优势. 这是分而治之的经典编程策略. 通过

## 多个匹配进行操作

我们见面了`pairs!`和`tuple!`它将固定数量的匹配捕获为 Rust 元组.

还有`many0`和`many1`- 他们都捕获无限数量的匹配作为向量. 不同的是,第一个可能会捕获"零或多个",第二个"一个或多个" (如正则表达式之间的差异`*`与`+`) 所以`many1!(ws!(float64))`会解析"1 2 3"`vec![1.0,2.0,3.0]`,但会在空字符串上失败.

`fold_many0`是一个 _减少{reducing}_ 操作. 使用二元运算符将匹配值组合为单个值. 例如,这就是 Rust 人 以前如何对迭代器进行求和`sum`加入;这个折叠从一个初始值 (这里是零) 开始 _累加器_ 并保持使用`+`该累加器的值.

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

到目前为止,我们必须捕获每个表达式,或者只是抓住所有匹配的字节`recognize!`:

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

对于更复杂的表达式,捕获所有解析器的结果会导致相当不整洁的类型!我们可以做得更好.

`do_parse!`让你只提取你感兴趣的值`>>`- 感兴趣的匹配是形式`名称:解析器{name:parser}`. `最后,在圆括号中有一个代码块.

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
            (Point{x: first, y: second})
        )
    );

    println!("got {:?}", nom_res!(pointf,"20,52.2").unwrap());
// got Point { x: 20, y: 52.2 }
```

我们对该标签的值不感兴趣 (它只能是逗号) ,但我们将两个浮点值分配给用于构建结构的临时值.

最后的代码可以是任何 Rust 表达式.

## 解析算术表达式

随着必要的背景建立,我们可以做简单的算术表达式.

这是真正无法用正则表达式完成的一个很好的例子.

这个想法是从下往上建立表达式.

表达式由 _条款{terms}_ 组成,这是增加或减少. 术语由 _因素{factors}_ 组成，它们相乘或分开。 和（现在）因素只是浮点数字：

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

这更准确地表达了我们的定义 - 一个表达式至少包含一个术语,然后包含零个或多个正-负项.

我们不收集它们,但是 _折叠{fold}_ 他们使用适当的操作员. (这是 Rust 不能很好地处理表达式类型的情况之一,所以我们需要一个类型提示. ) 这样做会建立正确的 _运算符优先级_ -`*`总是赢`+`等等.

我们在这里需要浮点断言,并且[有一个库](http://brendanzab.github.io/approx/approx/index.html).

将`approx ="0.1.1"`行添加到您的 Cargo.toml 中,然后离开:

```rust
#[macro_use]
extern crate approx;
...
    assert_relative_eq!(fold_sum("1 2 3").to_result().unwrap(), 6.0);
```

我们来定义一个方便的小测试宏. `stringify!`将表达式转换为我们可以输入的字符串文字`expr`, 然后将结果与 Rust 如何评估结果进行比较.

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

这非常酷 - 只需几行即可获得 表达式评估器!但它变得更好. 我们增加了一个数字的替代方案`因子{factor}`解析器 - 包含在括号内的表达式:

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

凉爽的是现在定义了表达式 _递归_ 在表达方面!

特别的魔力`delimited!`是括号可以嵌套 - nom 确保括号匹配.

我们现在已经超越了正则表达式的能力,0.5Mb 的被剥离可执行文件仍然是"hello world"正则表达式程序的一半.
