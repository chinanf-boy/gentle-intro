# 线程,网络和共享

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [改变不可变的](#%E6%94%B9%E5%8F%98%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84)
- [共享引用](#%E5%85%B1%E4%BA%AB%E5%BC%95%E7%94%A8)
- [多线程](#%E5%A4%9A%E7%BA%BF%E7%A8%8B)
- [线程不借](#%E7%BA%BF%E7%A8%8B%E4%B8%8D%E5%80%9F)
- [通道](#%E9%80%9A%E9%81%93)
- [同步](#%E5%90%8C%E6%AD%A5)
- [共享的状态](#%E5%85%B1%E4%BA%AB%E7%9A%84%E7%8A%B6%E6%80%81)
- [更高级别的操作](#%E6%9B%B4%E9%AB%98%E7%BA%A7%E5%88%AB%E7%9A%84%E6%93%8D%E4%BD%9C)
- [解决地址问题的更好方法](#%E8%A7%A3%E5%86%B3%E5%9C%B0%E5%9D%80%E9%97%AE%E9%A2%98%E7%9A%84%E6%9B%B4%E5%A5%BD%E6%96%B9%E6%B3%95)
- [TCP 客户端服务器](#tcp-%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%9C%8D%E5%8A%A1%E5%99%A8)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 改变不可变的

如果你感觉很猪头 (如我)，你想知道是否 _有过_ 可能避开借用检查器的限制。

考虑下面的小程序，它编译和运行没有问题。

```rust
// cell.rs
use std::cell::Cell;

fn main() {
    let answer = Cell::new(42);

    assert_eq!(answer.get(), 42);

    answer.set(77);

    assert_eq!(answer.get(), 77);
}
```

answer 已经改变了 - 但是`answer` _变量_ 是不可变的!

这显然是非常安全的，因为单元格内的值只能通过`set`和`get`访问。 这正是盛大名称 _内部可变性_。 通常被称为 _遗传可变性_: 如果我有一个结构值`v`，如果`v`本身是可写的，那么我可以加个字段`v.a`。 `Cell`值放宽了这个规则，因为我们可以用`set`改变其中包含的值，即使 cell 本身不可变。

然而，`Cell`只适用于`Copy`类型 (例如，派生了`Copy`trait 的原始类型和用户类型)。

对于其他值，我们必须得到一个可以工作的引用，可变或不可变。这`RefCell`提供的是 - 您明确要求，一个包含值的引用:

```rust
// refcell.rs
use std::cell::RefCell;

fn main() {
    let greeting = RefCell::new("hello".to_string());

    assert_eq!(*greeting.borrow(), "hello");
    assert_eq!(greeting.borrow().len(), 5);

    *greeting.borrow_mut() = "hola".to_string();

    assert_eq!(*greeting.borrow(), "hola");
}
```

再次，`greeting`不是可变的声明!

明确的解引用操作符`*`，可能在 Rust 中有点混乱，因为通常你不需要它 - 例如`greeting.borrow().len()`因为方法调用会隐含地解引用，所以很好。 但是你 _确实_ 需要`*`，从`greeting.borrow()`把底层`&String`拉出，或者从`greeting.borrow_mut()`把`&mut String`拉出。

用`RefCell`并不总是安全的，因为从这些方法返回的任何引用必须遵循通常的规则。

```rust
    let mut gr = greeting.borrow_mut(); // gr 是一个可变借用
    *gr = "hola".to_string();

    assert_eq!(*greeting.borrow(), "hola"); //<== 这里我们失败了!
....
thread 'main' panicked at 'already mutably borrowed: BorrowError'
```

你能在有可变借用的情况下，再搞个不可变借用 - 这很重要 - 违反规则的事情发生在 _运行时_。 解决方案 (一如既往) 是尽可能地限制可变借用的作用域 - 在这种情况下，您可以在此处放置两行`区域块{}`，以便可变引用`gr`在我们再次借用之前释放。

所以，若没有理由，这不应该是一个你使用的功能，除非你 _不_ 会得到一个编译时错误。 这些类型在通常规则下做不到的情况下，提供 _动态借用_ 的策略。

## 共享的引用

目前来说，值与其借来的引用之间的关系在编译时已经清楚明了。 值是所有者，且引用不能长命过它。 但许多案例根本不适合这种整洁的模式。 例如，假设我们有一个`Player`结构和一个`Role`结构。 一个`Player`存有多个`Role`对象引用的 一个 Vec。 这些值之间并没有一个整齐的一对一关系，并且`rustc`难以合作。

`Rc`工作上像`Box`- 分配堆内存和值会移到该内存。如果你克隆一个`Box`，它会分配一个值的深拷贝。 但克隆一个`Rc`是便宜的，因为每次你克隆它只是更新一个到 _相同的数据_ 的 _引用计数_ 。 这是一种古老且非常流行的内存管理策略，例如用于 iOS/MacOS 上的 Objective C 运行时。 (在现代 C ++中，它是用`std::shared_ptr`。 )

>译者: 扩展阅读，[非官方中文:`Rc<T>` 引用计数智能指针](https://kaisery.github.io/trpl-zh-cn/ch15-04-rc.html)

当一个`Rc`被释放时，引用计数递减。 当该计数变为零时，拥有的值将被丢弃并释放内存。

```rust
// rc1.rs
use std::rc::Rc;

fn main() {
    let s = "hello dolly".to_string();
    let rs1 = Rc::new(s); // s 移动 堆; ref 计为 1
    let rs2 = rs1.clone(); // ref 计为 2

    println!("len {}, {}", rs1.len(), rs2.len());
} //  rs1 和 rs2 释放, 字符串 挂了.
```

您可以根据自己的喜好，制作尽可能多的最初值的引用 - 这会再次 _动态借用_ 。 您不必盯着`T`及其引用`&T`值之间的关系。因有一些运行时间成本，所以它不是你选择的 _第一_ 解决方案，但它使共享模式成为可能，这是借用检查器所允许的。 注意`Rc`为您提供不可变的共享引用，因为若是可变引用，会破坏借用的基本规则之一。(超过一个可变引用，导致竟态问题)

在一个`Player`的情况下，它现在将它的 角色定位(roles) 存为`Vec<Rc<Role>>`并且工作很好 - 我们可以添加或删除 Role，但不能在创建后 _更改_ 他们。

但是，如果每个`Player`都有一个对 _team_ 的引用，_team_ 也就是个对一些`Player`引用的 Vec? 那么一切都变得不可变，因为`Player`所有的值需要被存储为`Rc`! 而这时候`RefCell`变得很有必要。 team 可能被定义为`Vec<Rc<RefCell<Player>>>`。也许现在想用`borrow_mut`更改一个`Player`值，与此同时不会 _提出_ 对 一个`Player`的引用 '查房'。 例如，假设我们有一条规则，即如果某位球员变强，那么队伍的所有人都会变得更强:

```rust
    for p in &self.team {
        p.borrow_mut().make_stronger();
    }
```

所以应用程序代码不是太糟糕，但类型签名会有点吓人。 你总是可以用一个`type`别名，来简化它们:

```rust
type PlayerRef = Rc<RefCell<Player>>;
```

## 多线程

在过去的二十年中，从原始处理速度转向具有多核的 CPU。 因此，充分利用现代计算机的唯一方法是保持所有这些核心繁忙。 正如我们所看到的，通过`Command`可以在后台 spawn 子进程，但这仍存在一个同步问题: 我们不能不等，因为我们不确切地知道这些孩子何时完成。

还有其他原因需要分开 _执行线程_， 当然。 例如，您锁定整个进程，只为了等待 I/O 的堵塞。

Spawn 线程在 Rust 中很简单 - 喂给`spawn`一个在后台执行的闭包，就可以了。

```rust
// thread1.rs
use std::thread;
use std::time;

fn main() {
    thread::spawn(|| println!("hello"));
    thread::spawn(|| println!("dolly"));

    println!("so fine");
    // 稍等一下
    thread::sleep(time::Duration::from_millis(100));
}
// so fine
// hello
// dolly
```

显然，只是"稍等一下"并不是一个非常严格的解决方案! 更好的是，在返回的对象上调用`join` - 然后主线程会等待生成的线程结束。

```rust
// thread2.rs
use std::thread;

fn main() {
    let t = thread::spawn(|| {
        println!("hello");
    });
    println!("wait {:?}"， t.join());
}
// hello
// wait Ok(())
```

这是一个有趣的变化: 强制新线程恐慌。

```rust
    let t = thread::spawn(|| {
        println!("hello");
        panic!("I give up!");
    });
    println!("wait {:?}", t.join());
```

我们如预期般 panic，但只有恐慌的线程死亡! 我们仍会打印出来的`join`错误信息。 所以是的，恐慌并不总是致命的，但线程相对昂贵，所以这不应被视为处理恐慌的常规方式。

```
hello
thread '<unnamed>' panicked at 'I give up!', thread2.rs:7
note: Run with `RUST_BACKTRACE=1` for a backtrace.
wait Err(Any)
```

返回的对象可以用来跟踪多个线程:

```rust
// thread4.rs
use std::thread;

fn main() {
    let mut threads = Vec::new();

    for i in 0..5 {
        let t = thread::spawn(move || {
            println!("hello {}", i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().expect("thread failed");
    }
}
// hello 0
// hello 2
// hello 4
// hello 3
// hello 1
```

Rust 坚持我们处理连接失败的情况 - 即该线程发生恐慌。 (当发生这种情况时，你通常不会退出主程序，只记下错误，重试等)

没有特定的线程执行顺序 (不同运行提供不同顺序) ，这就是关键 - 它们确实是 _独立的执行线程_。 多线程不难; _并发_ 才难 - 管理和 同步多个执行的线程.

## 线程不借

线程中的闭包函数有可能捕获值，但通过 _移动_，而不是 _借用_!

```rust
// thread3.rs
use std::thread;

fn main() {
    let name = "dolly".to_string();
    let t = thread::spawn(|| {
        println!("hello {}", name);
    });
    println!("wait {:?}", t.join());
}
```

以下是有用的错误消息:

```
error[E0373]: closure may outlive the current function, but it borrows `name`, which is owned by the current function
 --> thread3.rs:6:27
  |
6 |     let t = thread::spawn(|| {
  |                           ^^ may outlive borrowed value `name`
7 |         println!("hello {}", name);
  |                             ---- `name` is borrowed here
  |
help: to force the closure to take ownership of `name` (and any other referenced variables), use the `move` keyword, as shown:
  |     let t = thread::spawn(move || {
```

很好解释! 想象一下使用`move`的情况，从一个函数产生这个线程 - 在函数调用结束和原`name`被释放之后，该线程还可能活着。所以添加`move`解决了我们的问题。

但这是一个 _移动_ ，所以`name`可能只会出现在一个线程中! 我想强调的 _是_ ，这可能为共享引用，但他们需要有`静态`生命周期:

```rust
let name = "dolly";
let t1 = thread::spawn(move || {
    println!("hello {}", name);
});
let t2 = thread::spawn(move || {
    println!("goodbye {}", name);
});
```

`name`在整个项目期间存在（`静态`)，所以`rustc`对闭包不会长命过`name`感到满意。 但是，大多数有趣的引用没有`静态`生命周期!

线程无法共享相同的环境 - 这是 Rust 的 _设计_。 特别是,他们不能共享常规引用，因为闭包会移动捕获的变量。

_共享引用_ 还好，因为他们的生命周期是'与需要的一样长' - 但你不能为此而使用`Rc`. 这是因为`Rc`不是 _线程安全_ 的- 它针对非线程情况进行了优化。 幸运的是，使用`Rc`在这里是个编译错误;编译器一直在你的背后。

对于线程，你需要`std::sync::Arc`- 'Arc'代表'原子引用计数'。 也就是说，它保证了引用计数将在一个逻辑操作中被修改。 为了保证这一点，它必须确保操作被锁定，以便只有当前线程才能访问。`clone`实际上的成本比 copy 仍要低得多。

```rust
// thread5.rs
use std::thread;
use std::sync::Arc;

struct MyString(String);

impl MyString {
    fn new(s: &str) -> MyString {
        MyString(s.to_string())
    }
}

fn main() {
    let mut threads = Vec::new();
    let name = Arc::new(MyString::new("dolly"));

    for i in 0..5 {
        let tname = name.clone();
        let t = thread::spawn(move || {
            println!("hello {} count {}", tname.0, i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().expect("thread failed");
    }
}
```

这里，虽然我们`MyString`不实现`clone`，但我故意创建了一个`String`的包装类型 (一个'新类型')。 _共享引用_ 可以 clone!

共享引用`name`通过使用`clone`一个新引用，传递给每个新线程并将其移入闭包。 这有点冗长，但这是一种安全模式。 恰恰因为问题如此不可预测，安全在并发中很重要。 一个程序可能在你的机器上运行良好，但偶尔会在服务器上崩溃，通常在周末。 更糟糕的是，这些问题的症状不容易诊断。

## 通道

有线程间发送数据的方法. 这是， Rust 在使用 _通道_. `std::sync::mpsc::channel()`返回，一个由 _接收器{receiver}_ 通道 和 _寄件人{sender}_ 通道组成的元组。每个线程都收到了使用`clone`制作的发件人副本，和调用`send`。 同时主线程在接收器上调用`recv`。

'MPSC'代表'Multiple Producer Single Consumer'。 我们创建了多个试图发送到通道的线程，且主线程"消耗"这通道。

```rust
// thread9.rs
use std::thread;
use std::sync::mpsc;

fn main() {
    let nthreads = 5;
    let (tx, rx) = mpsc::channel();

    for i in 0..nthreads {
        let tx = tx.clone();
        thread::spawn(move || {
            let response = format!("hello {}", i);
            tx.send(response).unwrap();
        });
    }

    for _ in 0..nthreads {
        println!("got {:?}", rx.recv());
    }
}
// got Ok("hello 0")
// got Ok("hello 1")
// got Ok("hello 3")
// got Ok("hello 4")
// got Ok("hello 2")
```

由于线程在结束执行之前，会发送响应，显然这可能随时发生，因此这示例无需 `join`。 `recv`将阻塞，并且如果发送者通道断开连接，将返回一个错误。 `recv_timeout`只会在给定的时间段内阻塞，并且可能还会返回一个超时错误。

`send`从不阻塞，这很有用，因为线程可以在不等待接收器处理的情况下，推出数据。另外，通道有缓冲，所以可以发生多个发送操作，按顺序接收。

但是，不堵塞的情况下，`Ok`并不意味着'已成功发送消息'!

一个`sync_channel` _会_ 堵塞发送。如果参数为零，则发送会阻塞，直到 接收 发生。 线程必须满足或 _会合{rendezvous}_ (根据声音原则，法语中大多数声音听起来更好。 )

```rust
    let (tx, rx) = mpsc::sync_channel(0);

    let t1 = thread::spawn(move || {
        for i in 0..5 {
            tx.send(i).unwrap();
        }
    });

    for _ in 0..5 {
        let res = rx.recv().unwrap();
        println!("{}",res);
    }
    t1.join().unwrap();
```

在调用`recv`期间，若没有相应`send`，我们会很容易错误。例如，少个循环`for i in 0..4`，那么线程结束后，`tx`被扔掉，然后`recv`将失败。如果线程恐慌，导致其栈解散，释放任何值，也会错误。

如果`sync_channel`是用一个非零参数`n`创建的，那么它的行为就像一个具有最大尺寸`n`的队列-`send`只会在尝试添加，超过`n`个值到队列时，才会堵塞.

通道是强类型的 - 在这里通道有类型`i32`- 但类型推理使其隐藏。 如果您需要传递不同类型的数据，那么枚举是表达这一点的好方法。

## 同步

我们来看看 _同步_。 `join`是非常基本的操作，只是去等一个特定的线程完成。 一个`sync_channel`去同步两个线程 - 在最后一个例子中，衍生线程和主线程完全锁定在一起。

同步的 Barrier(屏障) 是一个检查点，在 _所有_ 的点都到位前，线程必须等着。都到位后，才可以像以前一样继续前进。 屏障是随着我们想要等待的线程们一起创建的。和以前一样，我们使用`use Arc`与所有线程共享 Barrier。

```rust
// thread7.rs
use std::thread;
use std::sync::Arc;
use std::sync::Barrier;

fn main() {
    let nthreads = 5;
    let mut threads = Vec::new();
    let barrier = Arc::new(Barrier::new(nthreads));

    for i in 0..nthreads {
        let barrier = barrier.clone();
        let t = thread::spawn(move || {
            println!("before wait {}", i);
            barrier.wait();
            println!("after wait {}", i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().unwrap();
    }
}
// before wait 2
// before wait 0
// before wait 1
// before wait 3
// before wait 4
// after wait 4
// after wait 2
// after wait 3
// after wait 0
// after wait 1
```

半随机的线程启动，全部满足后，继续。它就像一种可恢复`join`，当您需要将工作片段分散到不同的线程，并且想在所有线程完成时，采取一些行动，这时会有用。

## 共享的状态

线程怎么样 _修改_ 共享状态?

回想一下`RC<RefCell<T>>`策略，它是在共享引用上 _动态_ 做一个可变的借用。 而在线程上相当于`RefCell`的，就是`Mutex`- 你可以通过调用`lock`来获得可变引用。 当存在此引用，其他线程将无法访问它。 `互斥{mutex}`代表'相互排斥' - 我们锁定了一段代码，以便只有一个线程可以访问它，然后解锁它。 你用`lock`方法锁上，并在该引用被释放时解锁。

```rust
// thread9.rs
use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

fn main() {
    let answer = Arc::new(Mutex::new(42));

    let answer_ref = answer.clone();
    let t = thread::spawn(move || {
        let mut answer = answer_ref.lock().unwrap();
        *answer = 55;
    });

    t.join().unwrap();

    let ar = answer.lock().unwrap();
    assert_eq!(*ar, 55);

}
```

这不像使用`RefCell`那样简单，因为如果另一个线程在锁定时发生恐慌，那么请求互斥锁就可能失败。 (在这种情况下，文档的实际建议是用`unwrap`退出线程，因为事情严重错了!)

要这个可变的借用的存在，尽可能短更为重要，因为只要互斥锁被锁定，其他的线程都 _堵塞_。 这不应该你花大价钱的地方! 所以通常这样的代码会像这样使用:

```rust
// ... 在线程 do something
// 获得一个锁上的引用 并 短暂使用它
{
    let mut data = data_ref.lock().unwrap();
    // 修改数据
}
//... 线程继续
```

## 更高级别的操作

最好找到更高级的线程化方法，而不是自己管理同步。 一个例子是当你需要并发做事并收集结果时。 一个非常酷的箱子是[pipeliner](https://docs.rs/pipeliner/0.1.1/pipeliner/)它有一个非常直接的 API。 这里是该箱的'你好,世界!'`例子- 一个迭代器给我们输入，我们在值上执行`n`次并行操作。

```rust
extern crate pipeliner;
use pipeliner::Pipeline;

fn main() {
    for result in (0..10).with_threads(4).map(|x| x + 1) {
        println!("result: {}", result);
    }
}
// result: 1
// result: 2
// result: 5
// result: 3
// result: 6
// result: 7
// result: 8
// result: 9
// result: 10
// result: 4
```

这当然是一个笨例子，因为该操作计算起来非常便宜，但显示了并行运行代码的容易程度。

这有些更有用的东西. 并行执行网络操作非常有用，因为它们可能需要时间，并且在开始工作之前，您不希望等待它们 _所有_ 完成。

这个例子非常粗糙 (相信我，有更好的方法可以做到这一点)，但这里我们要关注这个实践。 我们重用定义在第 4 节中的这个`shell`函数，在一系列 IP4 地址上调用`ping`。

```rust
extern crate pipeliner;
use pipeliner::Pipeline;

use std::process::Command;

fn shell(cmd: &str) -> (String,bool) {
    let cmd = format!("{} 2>&1",cmd);
    let output = Command::new("/bin/sh")
        .arg("-c")
        .arg(&cmd)
        .output()
        .expect("no shell?");
    (
        String::from_utf8_lossy(&output.stdout).trim_right().to_string(),
        output.status.success()
    )
}

fn main() {
    let addresses: Vec<_> = (1..40).map(|n| format!("ping -c1 192.168.0.{}",n)).collect();
    let n = addresses.len();

    for result in addresses.with_threads(n).map(|s| shell(&s)) {
        if result.1 {
            println!("got: {}", result.0);
        }
    }

}
```

我的家庭网络上的结果如下所示:

```
got: PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=43.2 ms

--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 43.284/43.284/43.284/0.000 ms
got: PING 192.168.0.18 (192.168.0.18) 56(84) bytes of data.
64 bytes from 192.168.0.18: icmp_seq=1 ttl=64 time=0.029 ms

--- 192.168.0.18 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.029/0.029/0.029/0.000 ms
got: PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=110 ms

--- 192.168.0.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 110.008/110.008/110.008/0.000 ms
got: PING 192.168.0.5 (192.168.0.5) 56(84) bytes of data.
64 bytes from 192.168.0.5: icmp_seq=1 ttl=64 time=207 ms
...
```

在前半秒内，活动的地址会非常快速地输出，然后等待不好的结果。不然的话，我们会等待一分钟，才能看到活动地址! 您现在可以继续从输出中删除 ping 时间等事情，尽管这只能在 Linux 上运行。 `ping`是普遍的，但确切的输出格式在每个平台都是不同的。 为了做得更好，我们需要使用跨平台的 Rust 网络 API，所以让我们进入网络。

## 解决地址问题的更好方法

如果你 _只_ 想要可用，但不详细的 ping 统计信息，`std::net::ToSocketAddrs` trait 会为你做任何 DNS 解析:

```rust
use std::net::*;

fn main() {
    for res in "google.com:80".to_socket_addrs().expect("bad") {
        println!("got {:?}", res);
    }
}
// got V4(216.58.223.14:80)
// got V6([2c0f:fb50:4002:803::200e]:80)
```

它是一个迭代器，因为通常一个域名有多个相关联的接口 - Google 同时有 IPV4 和 IPV6 的接口。

所以，我们自然地使用这种方法来重写 pipeliner 示例。 大多数网络协议都使用地址和端口:

```rust
extern crate pipeliner;
use pipeliner::Pipeline;

use std::net::*;

fn main() {
    let addresses: Vec<_> = (1..40).map(|n| format!("192.168.0.{}:0",n)).collect();
    let n = addresses.len();

    for result in addresses.with_threads(n).map(|s| s.to_socket_addrs()) {
        println!("got: {:?}", result);
    }
}
// got: Ok(IntoIter([V4(192.168.0.1:0)]))
// got: Ok(IntoIter([V4(192.168.0.39:0)]))
// got: Ok(IntoIter([V4(192.168.0.2:0)]))
// got: Ok(IntoIter([V4(192.168.0.3:0)]))
// got: Ok(IntoIter([V4(192.168.0.5:0)]))
// ....
```

这比 ping 示例快得多，因为它只是检查 IP 地址是否有效 - 如果我们为它提供了一个实际域名列表，DNS 查找可能需要一些时间，这时并行性显得格外重要。

令人惊讶的是，它能工作。 标准库中的所有内容实现`Debug`的事实，非常适合勘探和调试。 迭代器正在返回`Result` (有`Ok`) 和这个`Result`是一个`IntoIter`裹`SocketAddr`，`SocketAddr`是一个带有 ipv4 或 ipv6 地址的枚举。 为什么是`IntoIter`? 由于套接字可能有多个地址 (例如，ipv4 和 ipv6) 。

```rust
    for result in addresses.with_threads(n)
        .map(|s| s.to_socket_addrs().unwrap().next().unwrap())
    {
        println!("got: {:?}", result);
    }
// got: V4(192.168.0.1:0)
// got: V4(192.168.0.39:0)
// got: V4(192.168.0.3:0)
```

这也能工作，惊讶吧。 首先 `unwarp`摆脱了`Result`，然后我们明确地将第一个值(Result 类型)从迭代器中取出(next)。当我们给出一个无意义的地址时 (比如没有端口的地址名称)，`Result`通常会变得很糟糕。

## TCP 客户端服务器

Rust 为最常用的网络协议 TCP ，提供了一个直接的接口。它具有很强的抗错能力，是我们网络世界的基础 - _包_ 的数据被发送和接收，并带有确认性。 相比之下，UDP 将数据包发送到外面，就不管确认性 - 有一个笑话是"我可以告诉你一个关于 UDP 的笑话，但你可能得不到这笑话。" (Jokes about networking are only funny for a specialized meaning of the word 'funny')

但是，错误处理是对于网络来说很 _非常_ 重要，因为任何事情都可能发生，并且最终会发生。

TCP 作为客户端/服务器工作的模型; 服务器监听一个地址和一个特定的 _网络端口_，并且客户端连接到该服务器。建立连接后，客户端和服务器可以用套接字进行通信。

`TcpStream::connect`需要可以转换成一个`SocketAddr`的任何结构，在这里是，一直使用的纯 string。

Rust 实现一个简单的 TCP 客户端很简单 - 一个`TcpStream`结构是可读和可写的。 像往常一样，我们必须将`Read`,`Write`和其他`std::io` tarit 纳入作用域:

```rust
// client.rs
use std::net::TcpStream;
use std::io::prelude::*;

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:8000").expect("connection failed");

    write!(stream,"hello from the client!\n").expect("write failed");
 }
```

服务器并不复杂，我们建立了一个监听器并等待连接。 当客户端连接时，我们在服务器端得到一个`TcpStream`。 在这种情况下，我们读取客户端写入字符串的所有内容。

```rust
// server.rs
use std::net::TcpListener;
use std::io::prelude::*;

fn main() {

    let listener = TcpListener::bind("127.0.0.1:8000").expect("could not start server");

    // accept connections and get a TcpStream
    for connection in listener.incoming() {
        match connection {
            Ok(mut stream) => {
                let mut text = String::new();
                stream.read_to_string(&mut text).expect("read failed");
                println!("got '{}'", text);
            }
            Err(e) => { println!("connection failed {}", e); }
        }
    }
}
```

这里我随机选择了一个无用端口的端口号，但是[大多数的端口](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)被赋予一些含义.

请注意，双方必须就协议达成一致 - 客户希望能够向 流{stream} 写入文本，并且服务器期望从 流{stream} 中读取文本。 如果他们不玩同一个游戏，那么情况就会发生在一方被阻塞的情况下，等待从未到来的字节。

检查错误非常重要 - 网络 I/O 可能因多种原因失败，并且在本地文件系统上，可能会定期发生一次 '蓝月亮'(blue moon) 的错误。 有人可以网线传输，另一方可能会崩溃，等等可能的情况。这个小服务器不是很健壮，因为它会在第一次读取错误时崩溃。

这是一个更坚实的服务器，可以在不崩溃的情况下处理错误。它还从数据流明确读取一个 _行{line}_ ，这是使用了`IO::BufReader`创造一个`IO::BufRead`，和我们调用`read_line`。

```rust
// server2.rs
use std::net::{TcpListener, TcpStream};
use std::io::prelude::*;
use std::io;

fn handle_connection(stream: TcpStream) -> io::Result<()>{
    let mut rdr = io::BufReader::new(stream);
    let mut text = String::new();
    rdr.read_line(&mut text)?;
    println!("got '{}'", text.trim_right());
    Ok(())
}

fn main() {

    let listener = TcpListener::bind("127.0.0.1:8000").expect("could not start server");

    // 接受 连接 和获得一个 TcpStream
    for connection in listener.incoming() {
        match connection {
            Ok(stream) => {
                if let Err(e) = handle_connection(stream) {
                    println!("error {:?}", e);
                }
            }
            Err(e) => { print!("connection failed {}\n", e); }
        }
    }
}
```

在`handle_connection`的`read_line`可能会失败，但由此产生的错误是安全处理的。

像这样的单向通信当然是有用的 - 例如，通过网络提供的一组服务，希望在一个中心位置将他们的状态报告集中在一起。 但期望有礼貌的回复是合理的，即使只有个'好'字!

一个基本的'echo'服务器的简单例子。 客户端将一些以换行符结尾的文本写入服务器，并使用换行符接收相同的文本 - stream 是可读写的.

```rust
// client_echo.rs
use std::io::prelude::*;
use std::net::TcpStream;

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:8000").expect("connection failed");
    let msg = "hello from the client!";

    write!(stream,"{}\n", msg).expect("write failed");

    let mut resp = String::new();
    stream.read_to_string(&mut resp).expect("read failed");
    let text = resp.trim_right();
    assert_eq!(msg,text);
}
```

服务器有一个有趣点。 只要改改`handle_connection`:

```rust
fn handle_connection(stream: TcpStream) -> io::Result<()>{
    let mut ostream = stream.try_clone()?;
    let mut rdr = io::BufReader::new(stream);
    let mut text = String::new();
    rdr.read_line(&mut text)?;
    ostream.write_all(text.as_bytes())?;
    Ok(())
}
```

这是一个简单的双向套接字通信的常见问题; 我们想要读取一行，因此需要将可读 stream 提供给`BufReader`- 但它 _消耗_ stream ! 所以我们必须克隆这个 stream，创建一个引用相同底层套接字的新结构。从此我们，幸福生活。
