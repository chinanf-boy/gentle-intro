
# 线程,网络和共享

## 改变不变

如果你感觉猪头 (如我) ,你想知道是否_曾经_可能避开借用检查器的限制. 

考虑下面的小程序,它编译和运行没有问题. 

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

答案已经改变了 - 但还是_变量_ `回答`是不可变的!

这显然是非常安全的,因为单元格内的值只能通过访问`组`和`得到`. 这是通过的盛大名称_内部可变性_. 通常被称为_遗传可变性_: 如果我有一个结构值`v`,那么我只能写字段`v.a`如果`v`本身是可写的. `细胞`价值观放宽了这个规则,因为我们可以改变其中包含的价值`组`即使细胞本身不可变. 

然而,`细胞`只适用于`复制`类型 (例如派生的原始类型和用户类型) `复制`特征) . 

对于其他值,我们必须得到一个可以工作的参考,可变或不可变. 这是什么`RefCell`规定 - 您明确要求参考包含的值: 

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

再次,`欢迎`没有被宣布为可变!

明确的解除引用操作符`*`可能在Rust中有点混乱,因为通常你不需要它 - 例如`greeting.borrow () . LEN () `因为方法调用会隐含地解除引用,所以很好. 但是你_做_需要`*`拉出底层`&串`从`greeting.borrow () `或者`&mut字符串`从`greeting.borrow_mut () `. 

用一个`RefCell`并不总是安全的,因为从这些方法返回的任何引用必须遵循通常的规则. 

```rust
    let mut gr = greeting.borrow_mut(); // gr is a mutable borrow
    *gr = "hola".to_string();

    assert_eq!(*greeting.borrow(), "hola"); // <== we blow up here!
....
thread 'main' panicked at 'already mutably borrowed: BorrowError'
```

如果你已经可以借用,你不能永久借钱!除了 - 这很重要 - 违反规则的事情发生在_运行_. 解决方案 (一如既往) 是尽可能地限制可变借入的范围 - 在这种情况下,您可以在此处放置前两行的块,以便可变参考`gr`在我们再次借用之前下降. 

所以,这不是一个你没有理由使用的功能,因为你会的_不_得到一个编译时错误. 这些类型提供_动态借款_在通常的规则使某些事情不可能的情况下. 

## 共享引用

到目前为止,价值与其借用引用之间的关系在编译时已经清楚并且已知. 价值是所有者,并且引用不能超过它. 但许多案例根本不适合这种整洁的模式. 例如,假设我们有一个`播放机`结构和a`角色`结构. 一个`播放机`保持引用的向量`角色`对象. 这些价值观之间并没有一个整齐的一对一关系,并且说服了对方`rustc`合作变得讨厌. 

`Rc`作品像`框`- 堆内存被分配并且值被移动到它. 如果你克隆一个`框`它会分配一个值的完整克隆副本. 但克隆一个`Rc`是便宜的,因为每次你克隆它只是更新一个_引用计数_到了_相同的数据_. 这是一种古老且非常流行的内存管理策略,例如用于iOS / MacOS上的Objective C运行时.  (在现代C ++中,它是用`的std :: shared_ptr的`. ) 

当一个`Rc`被丢弃时,引用计数递减. 当该计数变为零时,拥有的值将被丢弃并释放内存. 

```rust
// rc1.rs
use std::rc::Rc;

fn main() {
    let s = "hello dolly".to_string();
    let rs1 = Rc::new(s); // s moves to heap; ref count 1
    let rs2 = rs1.clone(); // ref count 2

    println!("len {}, {}", rs1.len(), rs2.len());
} // both rs1 and rs2 drop, string dies.
```

您可以根据自己的喜好制作尽可能多的参考文献,以获得最初的价值 - 是的_动态借款_再次. 您不必仔细追踪价值之间的关系`Ť`及其参考资料`&T`. 有一些运行时间成本,所以它不是_第一_你选择的解决方案,但它使分享的模式成为可能,这将会使借用检查器变得不合适. 注意`Rc`为您提供不可变的共享引用,因为否则这会破坏借用的基本规则之一. 一只豹在不停止成为豹的情况下不能改变其斑点. 

在a的情况下`播放机`,它现在可以保持它的角色`VEC <RC <角色>>`并且事情很好 - 我们可以添加或删除角色,但不能_更改_他们在创作之后. 

但是,如果每个都是`播放机`需要保持对a的引用_球队_作为一个向量`播放机`参考?然后一切都变得不可变,因为所有的东西`播放机`值需要被存储为`Rc`!这是地方`RefCell`变得必要. 团队可能被定义为`VEC <RC <RefCell <玩家>>>`. 现在可以更改一个`播放机`价值使用`borrow_mut`,_提供_没有人'检查出'一个参考`播放机`与此同时. 例如,假设我们有一条规则,即如果某位球员发生了某种特殊情况,那么他们的所有球队都会变得更强大: 

```rust
    for p in &self.team {
        p.borrow_mut().make_stronger();
    }
```

所以应用程序代码不是太糟糕,但类型签名会有点吓人. 你总是可以用一个简化它们`类型`别名: 

```rust
type PlayerRef = Rc<RefCell<Player>>;
```

## 多线程

在过去的二十年中,从原始处理速度转向具有多核的CPU. 因此,充分利用现代计算机的唯一方法是保持所有这些内核的繁忙. 正如我们所看到的那样,在后台产生子进程当然是可能的`命令`但仍然存在一个同步问题: 我们不确切地知道这些孩子何时完成而不等待他们. 

还有其他原因需要分开_执行线程_, 当然. 例如,您无法锁定整个流程,只能等待阻止I / O. 

产卵线程在Rust中很简单 - 喂食`卵`一个在后台执行的闭包. 

```rust
// thread1.rs
use std::thread;
use std::time;

fn main() {
    thread::spawn(|| println!("hello"));
    thread::spawn(|| println!("dolly"));

    println!("so fine");
    // wait a little bit
    thread::sleep(time::Duration::from_millis(100));
}
// so fine
// hello
// dolly
```

显然,只是"稍等一下"并不是一个非常严格的解决方案!打电话最好`加入`在返回的对象上 - 然后主线程等待生成的线程完成. 

```rust
// thread2.rs
use std::thread;

fn main() {
    let t = thread::spawn(|| {
        println!("hello");
    });
    println!("wait {:?}", t.join());
}
// hello
// wait Ok(())
```

这是一个有趣的变化: 强制新线程恐慌. 

```rust
    let t = thread::spawn(|| {
        println!("hello");
        panic!("I give up!");
    });
    println!("wait {:?}", t.join());
```

我们如预期般惊慌失措,但只有恐慌线程死亡!我们仍然设法打印出来的错误信息`加入`. 所以是的,恐慌并不总是致命的,但线程相对昂贵,所以这不应被视为处理恐慌的常规方式. 

    hello
    thread '<unnamed>' panicked at 'I give up!', thread2.rs:7
    note: Run with `RUST_BACKTRACE=1` for a backtrace.
    wait Err(Any)

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

Rust坚持我们处理连接失败的情况 - 即该线程发生恐慌.  (当发生这种情况时,你通常不会退出主程序,只记下错误,重试等) 

没有特定的线程执行顺序 (该程序为不同的运行提供不同的顺序) ,这是关键 - 它们确实是_独立的执行线程_. 多线程很简单;什么是困难的_并发_- 管理和同步多个执行线程. 

## 线程不会借用

线程闭包有可能捕获值,但是可以通过_移动_,而不是_借款_!

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

这很公平!想象一下,从一个函数产生这个线程 - 它将在函数调用完成后存在`名称`被丢弃. 所以补充`移动`解决了我们的问题. 

但这是一个_移动_,所以`名称`可能只会出现在一个线程中!我想强调一下_是_可能分享参考,但他们需要有`静态的`一生: 

```rust
let name = "dolly";
let t1 = thread::spawn(move || {
    println!("hello {}", name);
});
let t2 = thread::spawn(move || {
    println!("goodbye {}", name);
});
```

`名称`在整个项目期间存在`静态的`) ,所以`rustc`对封闭永不过时感到满意`名称`. 但是,大多数有趣的参考文献没有`静态的`寿命!

线程无法共享相同的环境 -  by_设计_在铁锈. 特别是,他们不能共享常规引用,因为关闭会移动捕获的变量. 

_共享参考_但很好,因为他们的生命是'只要需要' - 但你不能使用`Rc`为了这. 这是因为`Rc`不是_线程安全_- 它针对非线程情况进行了优化. 幸运的是,这是一个编译错误使用`Rc`这里;编译器一直在看你的背部. 

对于线程,你需要`的std ::同步::弧`- '弧'代表'原子参考计数'. 也就是说,它保证了引用计数将在一个逻辑操作中被修改. 为了保证这一点,它必须确保操作被锁定,以便只有当前线程才能访问. `克隆`但实际上制作副本的成本仍然要低得多. 

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

我故意创建了一个包装类型`串`这里 ('新类型') 自从我们的`MyString的`不执行`克隆`. 但是_共享参考_可以克隆!

共享参考`名称`通过使用新的引用传递给每个新线程`克隆`并将其移入封口. 这有点冗长,但这是一种安全模式. 恰恰因为问题如此不可预测,安全在并发中很重要. 一个程序可能在你的机器上运行良好,但偶尔会在服务器上崩溃,通常在周末. 更糟糕的是,这些问题的症状不容易诊断. 

## 通道

有线程间发送数据的方法. 这是在Rust使用_渠道_. `的std ::同步:: MPSC ::信道 () `返回由...组成的元组_接收器_频道和_寄件人_渠道. 每个线程都通过发件人的副本`克隆`,和电话`发送`. 同时主线程调用`的recv`在接收器上. 

'MPSC'代表'Multiple Producer Single Consumer'. 我们创建了多个试图发送到频道的线程,并且主线程"消耗"了频道. 

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

由于线程在结束执行之前发送响应,因此无需在此加入,但显然这可能随时发生. `的recv`将阻塞,并且如果发送者通道断开连接,将返回错误. `recv_timeout`只会在给定的时间段内阻塞,并且可能还会返回超时错误. 

`发送`从不阻塞,这很有用,因为线程可以在不等待接收器处理的情况下推出数据. 另外,通道被缓冲,所以可以发生多个发送操作,这将按顺序接收. 

但是,不阻止意味着`好`并不意味着'已成功发送消息'!

一个`sync_channel` _不_阻止发送. 如果参数为零,则发送阻塞直到recv发生. 线程必须满足或_会合_ (根据声音原则,法语中大多数声音听起来更好. ) 

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

通过调用,我们可以很容易地导致错误`的recv`当没有相应的时候`发送`例如通过循环`因为我在0..4`. 线程结束,并且`tx`滴,然后`的recv`将失败. 这也会发生,如果线程恐慌,导致其堆栈解开,丢弃任何值. 

如果`sync_channel`是用一个非零参数创建的`ñ`,那么它的行为就像一个最大尺寸为的队列`ñ`-`发送`只会在尝试添加多个时才会阻止`ñ`值到队列. 

频道是强类型的 - 在这里频道有类型`i32`- 但类型推理使得这种隐含的. 如果您需要传递不同类型的数据,那么枚举是表达这一点的好方法. 

## 同步

我们来看看_同步_. `加入`是非常基本的,只是等到一个特定的线程完成. 一个`sync_channel`同步两个线程 - 在最后一个例子中,衍生线程和主线程完全锁定在一起. 

屏障同步是线程必须等到的检查点_所有_他们已经达到了这一点. 然后他们可以像以前一样继续前进. 屏障是由我们想要等待的线程数创建的. 和以前一样,我们使用use`弧`与所有主题共享障碍. 

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

线程做他们半随机的事情,全部满足,然后继续. 它就像一种可恢复的`加入`当您需要将工作片段分散到不同的线程并且想要在所有片断完成时采取一些行动时有用. 

## 共享状态

线程怎么样_修改_共享状态?

回想一下`RC <RefCell <T >>`战略_动态_在共享引用上做一个可变的借用. 线程相当于`RefCell`是`互斥`- 你可以通过调用来获得可变参考`锁`. 虽然存在此引用,但其他线程无法访问它. `互斥`代表'相互排斥' - 我们锁定了一段代码,以便只有一个线程可以访问它,然后解锁它. 你拿到了锁`锁`方法,并在参考被删除时解锁. 

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

这不像使用那样简单`RefCell`因为如果另一个线程在锁定时发生恐慌,那么请求互斥锁就可能失败.  (在这种情况下,文档实际上建议只是退出线程`摅`因为事情严重错了!) 

把这个可变的借用保持尽可能短是更重要的,因为只要互斥锁被锁定,其他的线程_受阻_. 这不是昂贵的计算的地方!所以通常这样的代码会像这样使用: 

```rust
// ... do something in the thread
// get a locked reference and use it briefly!
{
    let mut data = data_ref.lock().unwrap();
    // modify data
}
//... continue with the thread
```

## 更高级别的操作

最好找到更高级的线程化方法,而不是自己管理同步. 一个例子是当你需要平行做事并收集结果时. 一个非常酷的箱子是[流水线化](https://docs.rs/pipeliner/0.1.1/pipeliner/)它有一个非常直接的API. 这是'你好,世界!'`- 一个迭代器给我们输入,我们执行到`ñ

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

并行操作的值. 

这当然是一个愚蠢的例子,因为该操作的计算起来非常便宜,但显示了并行运行代码的容易程度. 这是更有用的东西. _并行执行网络操作非常有用,因为它们可能需要时间,并且您不希望等待它们_所有

在开始工作之前完成. 这个例子非常粗糙 (相信我,有更好的方法可以做到这一点) ,但这里我们要关注这个原则. `我们重用这个`贝壳`函数定义在第4节中调用`平

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

在一系列IP4地址上. 

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

我的家庭网络上的结果如下所示: 在前半秒内,活动地址会非常快速地通过,然后等待负面结果进入. 否则,我们会等待一分钟的好时间!您现在可以继续从输出中删除ping时间等事情,尽管这只能在Linux上运行. `平`是普遍的,但确切的输出格式对于每个平台都是不同的. 为了做得更好,我们需要使用跨平台的Rust网络API,所以让我们进入网络. 

## 解决地址问题的更好方法

如果你_只是_想要可用性和不详细的ping统计信息`的std ::网:: ToSocketAddrs`特质将为你做任何DNS解析: 

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

它是一个迭代器,因为通常有多个接口与一个域相关联 - 同时有Google的IPV4和IPV6接口. 

所以,我们天真地使用这种方法来重写流水线示例. 大多数网络协议都使用地址和端口: 

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

这比ping示例快得多,因为它只是检查IP地址是否有效 - 如果我们为它提供了一个实际域名列表,DNS查找可能需要一些时间,因此并行性的重要性. 

令人惊讶的是,它只是工作. 标准库中的所有内容实现的事实`调试`非常适合勘探和调试. 迭代器正在返回`结果` (因此`好`) 和在那`结果`是一个`IntoIter`变成一个`SocketAddr`这是一个带有ipv4或ipv6地址的枚举. 为什么`IntoIter`?由于套接字可能有多个地址 (例如,ipv4和ipv6) . 至少对于我们这个简单的例子来说,这也起到了令人惊讶的作用. 

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

首先摅`摆脱了`结果`,然后我们明确地将第一个值从迭代器中取出. `该结果`当我们给出一个无意义的地址时 (比如没有端口的地址名称) ,通常会变得很糟糕. `TCP客户端服务器

## Rust为最常用的网络协议TCP提供了一个直接的界面. 

它具有很强的抗错能力,是我们网络世界的基础 -包_的数据被发送和接收,并带有确认. _相比之下,UDP将数据包发送到野外而没有得到确认 - 有一个笑话是"我可以告诉你一个关于UDP的笑话,但你可能得不到它. " (关于网络的笑话只对"有趣"这个词的特殊含义有趣) 

但是,错误处理是_非常_对于网络来说很重要,因为任何事情都可能发生,并且最终会. 

TCP作为客户端/服务器模型工作;服务器监听一个地址和一个特定的地址_网络端口_,并且客户端连接到该服务器. 建立连接后,客户端和服务器可以与套接字进行通信. 

`TcpStream ::连接`需要任何可以转换成一个`SocketAddr`,特别是我们一直使用的纯色琴弦. 

Rust中的一个简单的TCP客户端很容易 -  a`TcpStream`结构是可读和可写的. 像往常一样,我们必须带来`读`,`写`和别的`的std :: IO`特质纳入范围: 

```rust
// client.rs
use std::net::TcpStream;
use std::io::prelude::*;

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:8000").expect("connection failed");

    write!(stream,"hello from the client!\n").expect("write failed");
 }
```

服务器并不复杂,我们建立了一个监听器并等待连接. 当客户端连接时,我们得到一个`TcpStream`在服务器端. 在这种情况下,我们读取客户端写入字符串的所有内容. 

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

这里我随机选择了一个更无端口的端口号,但是[大多数港口](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)被赋予一些含义. 

请注意,双方必须就协议达成一致意见 - 客户希望它能够向流写入文本,并且服务器期望从流中读取文本. 如果他们不玩同一个游戏,那么情况就会发生在一方被阻塞的情况下,等待从未到来的字节. 

检查错误非常重要 - 网络I / O可能因多种原因失败,并且在本地文件系统上可能出现一次蓝色月亮的错误可能会定期发生. 有人可以通过网线传输,另一方可能会崩溃,等等. 这个小服务器不是很健壮,因为它会在第一次读取错误时崩溃. 

这是一个更坚实的服务器,可以在不失败的情况下处理错误. 它还具体读取一个_线_从流,这是使用完成`IO :: BufReader`创造一个`IO ::的BufRead`我们可以打电话给他们`read_line`. 

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

    // accept connections and get a TcpStream
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

`read_line`可能会失败`handle_connection`,但由此产生的错误是安全处理的. 

像这样的单向通信当然是有用的 - 例如. 通过网络提供的一组服务,希望在一个中心位置将他们的状态报告集中在一起. 但期望有礼貌的回复是合理的,即使只是'好'!

一个简单的例子是一个基本的'回声'服务器. 客户端将一些以换行符结尾的文本写入服务器,并使用换行符接收相同的文本 - 流是可读写的. 

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

服务器有一个有趣的转折点. 只要`handle_connection`变化: 

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

这是一个简单的双向套接字通信的常见问题;我们想要读取一行,因此需要将可读流提供给`BufReader`- 但它_消耗_溪流!所以我们必须克隆这个流,创建一个引用相同底层套接字的新结构. 然后我们有幸福. 
