# 第一章 语言基础

## 1.3 数组、字符串和切片

### 1.3.1 数组

固定长度的特定元素的序列（0个或多个）。

- 数组的`长度`是数组的一部分，所以不同长度的数组一定不是同一类型的
- GO语言中数组是`值语义`（而非指针或引用）
- 可以将数组看成特殊的结构体，只不过数组可以通过`range操作`或者`下标索引`来访问内部数据
- `range`操作性能可能会更好，而且会保证不会出现`越界操作`
- 除了值数组之外，还可以定义字符串数组，结构体数组，函数数组，接口数组，通道数组等等
- 长度为0的数组为空数组，如：`[0]int`，在内存中不占用空间，可以用于通道的同步操作：`ch <- [0]int`，不过一般会用`struct{}`
- 数组类型是切片和字符串的基础

### 1.3.2 字符串

- 字符串的元素不可修改，是一个只读的字节数组。
- 字符串的底层结构是:

```go
type StringHeader struct {
    Data uintptr
    Len int
}
```

- 字符创其实是一个结构体，字符串的赋值过程其实就是reflect.StringHeader的复制过程，不涉及底层字节数组的复制
- 字符传虽然不是切片，但支持切片操作

### 1.3.2 切片

切片的底层数据结构有三部分，如下：

```go
type SliceHeader struct {
    Data uintptr
    Len int
    Cap int
}
```

包括指向底层数组的指针，长度和容量。

nil切片和空切片

- nil切片：底层指针指向nil，这个切片等于nil；
- 空切片：底层的指针指向一个数组，但是长度容量都是0，没有内容。

append函数的用法

- 尾部加数据：超过容量限制后，重新分配内存，并对数据进行拷贝；
- 头部加数据：大概率会重新分配内存，一般来说，往头部添加元素性能会差很多；
- append函数返回的也是一个切片，所以也支持链式操作。

以下是一个利用空切片长度为0性质的一个例子：

```go
func TrimSpace(s []byte) []byte {
    b := s[:0]
    for x := range s {
        if x != ' ' {
            b = append(b, x)
        }
    }

    return b
}
```

在使用切片时要注意可能出现的内存泄露，如：

```go
    var a []*int{ ... }
    a = a[:(len(a)-1)]
```

被删除的最后一个元素仍被引用，可能导致垃圾回收器操作被阻碍

## 1.4 函数方法和接口

函数对应操作序列，时程序的基本组成元素。GO中函数分为具名函数和匿名函数：具名函数一般对用包级函数，是匿名函数的一种特例。匿名函数引用了外部作用域中的变量时，就成了`闭包函数`，闭包函数时函数式编程语言的核心。

方法是绑定到一个具体类型的函数，GO中的方法是依托于类型的，必须在编译时静态绑定。

接口定义了方法的集合，接口对应的方法是动态绑定的。GO通过接口实现了鸭子类型。

### 1.4.1 函数

在GO语言中，函数是第一类对象（first class），可以将函数保存到变量中去。

GO的`可变参`函数的本质，实际上是切片作为函数参数的函数。

GO语言中，函数传值都是值传递，指针同样也会拷贝，只不过拷贝后的指针仍指向和原来的指针一样的地址。

在GO语言中，不需要太过于操心堆和栈的问题；同样不要假设变量在内存中的位置是固定不变的，指针随时可能发生变化。（使用CGO时不能再C语言中长期持有GO语言对象中对象的地址）。

### 1.4.2 方法

在C++中，方法对应一个类对象的成员函数，是关联到对象的具体虚表上的；
GO的方法却是关联到类型的，这样才能编译时完成静态绑定；
JAVA号称是纯粹的面向对象语言，因为JAVA中函数是不能独立存在的，每个函数都必然属于某个类。

一个面向对象的程序会用方法开表达其属性对应的操作，这样的话，使用这个对象的用户就不需要直接去操作对象，而是借助方法来完成这些事情。

面向对象更多的只是一种思想，在C语言中也存在类似的思想，我们写一段C风格的代码：

```go
//文件对象
type File struct {
    fd int
}

//打开文件
func OpenFile(name string) (f *File, err error) {
    //...
}

//关闭文件
func CloseFile(f *File) error {
    //...
}

//读取文件
func ReadFile(f *File, offset int64, data []byte) error {
    //...
}

```

其中的OpenFile类似构造函数，用于打开文件对象，CloseFile类似析构函数，用于关闭文件对象，ReadFile则是普通的成员函数。

**注意**：类型的定义和对应方法的定义都应该在同一个包中，因此，类似`int`这种内置类型因为作用域是全局的，所以是无法添加方法的

**方法的继承**：组合，匿名字段的嵌套

### 1.4.3 接口

**鸭子类型**：走起来像鸭子，叫起来像鸭子，可以把它当做鸭子；在GO中指，如果一个对象只要看起来像是某种接口的实现，那么它就可以作为该接口类型使用。

GO的接口类型是延迟绑定，可以实现类似虚函数的多态

我们可以定制自己的输出对象，将每个字符转换为大写然后输出

```go
type UpperWriter struct {
 io.Writer
}

func (p *UpperWriter) Write(data []byte) (n int, err error) {
 return p.Writer.Write(bytes.ToUpper(data))
}

func main() {
 fmt.Fprintln(&UpperWriter{os.Stdout}, "helloworld")
}

```

GO是一种强类型语言，基础类型是不支持隐式转换的，但是接口的转换却非常灵活。
但有时候对象和接口之间太灵活了，需要人为的限制这种无意间的适配。
常用做法是设置一个特殊方法来做区分，不过这也是一种“君子协定”，很容易故意伪造，更严格一点的方法是给接口定义一个私有方法，私有方法名字包含绝对路径，因此只有在包内部实现这个私有方法才可能满足这个接口，不过这种方法同样不是万无一失的，且这个接口只能在包内部使用。通过嵌入testing.TB接口来伪造私有方法，如下：

```go
type TB struct {
 testing.TB
}

func (p *TB) Fatal(args ...interface{}) {
 fmt.Println("TB.Fatal disabled")
}

func main() {
 var tb testing.TB = new(TB)
 tb.Fatal("hello")
}
```

我们自己的TB结构中重新实现了Fatal方法，通过对象隐式转换为testing.TB接口类型（因为内嵌了testing.TB匿名字段，所以是满足这个接口的），然后再调用自己的方法。
这种通过嵌入匿名接口或者匿名指针对象来实现继承的做法其实是一种纯虚继承，继承的只有接口指定的规范，真正的实现在运行的时候才被注入。

## 1.5 面向并发的内存模型

从理论上看，多线程和基于消息的并发编程是等价的。GO语言是基于消息并发模型的集大成者，将基于CSP模型内置到了语言中，通过一个go关键字就可以轻易的启动一个Goroutine，GO语言中的goroutine是共享内存的。

### 1.5.1 Goroutine和系统线程

Groutine和系统线程并不是等价的，虽然实际上只有量的区别。系统线程有一个固定大小的栈，一般是2MB，而goroutine是动态栈（2KB ~ 1GB）。

### 1.5.2 原子操作

**最小的且不可并行的操作。** 通常，若多个并发体对同一个共享资源进行的操作是原子的，那么同一时刻最多只能有一个并发体对这个资源操作。从**线程**角度看，当前线程修改共享资源期间，其他线程是不能访问该资源的。

一般情况下，原子操作都是通过**互斥**访问来保证的，通常由特殊的CPU指令提供保护，只想模拟粗粒度的原子操作，可以借助sync.Mutex来实现。但是使用互斥锁来保护一个**数值型**的共享资源麻烦且效率低下，GO同样提供了sync/atomic包对原子操作提供支持。

原子操作和互斥锁配合可以实现相当高效的单件（单例）模式，如下：

```go
type singleton struct {}

var (
    instance    *singleton
    initialized uint32
    mu          sync.Mutex
)

func GetInstance() *singleton {
    if atomic.LoadUint32(&initialized) == 1 {
        return instance
    }

    mu.Lock()
    defer mu.UnLock()

    if instance == nil {
        defer atomic.StoreUint32(&initialized, 1)
        instance = &instance{}
    }

    return instance
}
```

将通用的代码提出来，就得到了sync.Once的实现：

```go
type Once struct {
    m       Mutex
    done    uint32
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 1 {
        return
    }
    o.mu.Lock()
    defer o.mu.UnLock()

    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}

```

基于Once的单例模式：

```go
var (
    instance *singleton
    once sync.Once
)

func GetInstance() *singleton {
    once.Do(func() {
        instance = *singleton{}
    })

    return instance
}

```

此外，atomic.Value原子对象提供了Load()和Store()两个原子方法，分别用于加载和保存数据，返回值和参数都是interface{}类型。

### 1.5.3 顺序一致性内存模型

通俗化：看到的顺序就是执行的顺序，就满足顺序一致性

如果只是简单的想在线程之间进行数据同步的话，原子操作已经为编程人员提供了一些编程保障。这种保障有一个前提：**顺序一致性的内存模型**。看一个例子：

```go
var a string
var done bool

func setup() {
    a = "hello"
    b = true
}

func main() {
    go setup()
    for !done {}
    pirnt(a)
}

```

GO语言并不保证main函数中观测到对done的写入是在a的写入操作之后，因此很有可能打印一个空字符串；因为两个线程之间没有同步事件，setup的写入操作甚至可能不会被main看到，main可能会陷入死循环中。

即在一个goroutine中执行 `a=1`,`b=2`这两个语句，对于这个goroutine来说对a的赋值要先于对b的赋值，但是在另一个goroutine中就不能保证先后顺序了，甚至一些goroutine都不能看到他们的变化。

不同的gouroutine之间，并不满足顺序一致性的内存模型，需要通过定义明确的`同步事件`来作为参考，如果两个事件之间不可排序，我们就说这两个事件是并发的。

之前提到的原子操作解决不了这个问题，我们无法确定两个原子操作之间的顺序。可以通过锁或者通道来解决这个问题

### 1.5.4 初始化顺序

初始化和执行总是从main.main()开始执行的，，但是如果main包中导入其他包，则会递归的初始化这些包（所以这些包之间不可以相互引入）。

大体流程是：从main包开始，初始化引入的包，初始化常量，初始化变量，执行init()函数，进入main.mian()函数。对于其他包，同样的是，初始化引入的包，初始化常量，初始化变量，执行init()函数，这是一个递归的过程。

### 1.5.5 goroutine的创建

go语句会创建当前goroutine对应函数返回前创建新的goroutine。如：

```go
var a string

func f() {
    print(a)
}

func hello() {
    a = "hello"
    go f()
}

```

创建goroutine的**go f()**语句和hello()函数的执行是在同一个goroutine中的，根据语句的书写顺序，可以确定goroutine的创建是发生在对a的赋值之后和hello函数返回之前。但是新创建的goroutine中f()和hello()函数返回这两个事件是没有办法确定顺序的，也就是并发的。(往往是函数的返回比较靠前，导致新创建的goroutine还没来得及执行就结束了)

### 1.5.6 基于通道的通信

通道分为无缓冲（同步）和有缓冲（异步）两种，无缓冲通道的每一次发送操作都有与其对应的接收操作相匹配，发送和接收操作通常发生在不同的goroutine中（在同一个goroutine上执行两个操作很容易发生死锁）。

对于有缓冲的通道，第K此接收操作应该发生在第K+C个发生操作之前，C是通道容量（可以把无缓冲通道看成缓冲为0的有缓冲通道）

### 1.5.7 不靠谱的同步

```go
func main() {
    go println("hello")
    time.Sleep(time.Second)
}
```

不能保证业务代码一定能在1s中跑完

## 1.6 常见的并发模式

理论来源于是通信顺序进程（CSP）。

### 1.6.1 并发版本的hello world

```go
func main() {
    var mu sync.Mutex

    go func() {
        fmt.Println("hello")
        mu.Lock()
    }()

    mu.UnLock
}
```

对没上锁的锁进行解锁操作会导致运行时异常。在上面的代码中上锁和解锁的操作在不同的goroutine中，不满足顺序一致性，解锁行为可能发生在上锁之前，导致解锁时发生异常。

修复的方式是在main函数中执行两次加锁，而不是一次解锁行为：

```go
func main() {
    var mu sync.Mutex

    mu.Lock()
    go func() {
        fmt.Println("hello world")
        mu.UnLock()
    }
    mu.Lock()
}
```

第二次加锁时会因为锁被占用而阻塞，驱动mian函数中创建的Goroutine继续执行，解锁后main函数的第二次Lock同样会取消阻塞。

同样，可以通过通道来实现同步。

### 1.6.2 生产者/消费者模式

生产者/消费者模型是最常见的，主要通过平衡生产线程和消费线程的工作能力来提高程序的整体处理速度。简单的讲，就是生产者生产一些数据，放到成果队列中，同时消费者从成果队列中取这些数据。

GO语言中借用通道，很容易实现生产者/消费者模式，如下：

```go
func Producer(factor int, out chan<-int) {
    for i := 0; ; i ++ {
        out <- i * factor
    }
}

func Consumer(in <- chan int) {
    for v := range in {
        fmt.Println(v)
    }
}

func main() {
    ch := make(chan int, 64)
    go Producer(3, ch)
    go Producer(5, ch)
    go Consumer(ch)
    //不靠谱的同步
    //time.Sleep(5 * time.Second)
    //用ctrl+c退出
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    fmt.Printf("quit (%v)\n", <-sig)
}

```

### 1.6.2 发布/订阅模式

简写为pub/sub模式，消息的生产者是pub，消费者是sub，生产者和消费者是M:N的关系，在生产者/消费者模型中，消息会被发到一个`成果队列`中，pub/sub模式则是将消息发布给一个`主题`。

pubsub.go

```go
//简易的多主题的发布者/订阅者库
package pubsub

import (
 "sync"
 "time"
)

type (
 subscriber chan interface{}         //订阅者是一个通道
 topicFunc  func(v interface{}) bool //主题为一个过滤器
)

//发布者对象
type Publisher struct {
 m           sync.RWMutex             //读写锁
 buffer      int                      //订阅队列的缓存大小
 timeout     time.Duration            //发布超时时间
 subscribers map[subscriber]topicFunc //订阅者信息
}

//构建一个发布者对象，可以设置超时时间和缓存队列长度
func NewPublisher(buffer int, timeout time.Duration) *Publisher {
 return &Publisher{
  buffer:      buffer,
  timeout:     timeout,
  subscribers: make(map[subscriber]topicFunc),
 }
}

//添加一个订阅者，订阅过滤器筛选后的主题
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
 ch := make(chan interface{}, p.buffer)
 p.m.Lock()
 p.subscribers[ch] = topic
 p.m.Unlock()
 return ch
}

//添加一个订阅者，订阅所有主题
func (p *Publisher) Subscribe() chan interface{} {
 return p.SubscribeTopic(nil)
}

//退出订阅
func (p *Publisher) Evivt(sub chan interface{}) {
 p.m.Lock()
 defer p.m.Unlock()

 delete(p.subscribers, sub)
 close(sub)
}

//关闭发布者对象，且关闭所有的订阅者通道
func (p *Publisher) Close() {
 p.m.Lock()
 defer p.m.Unlock()

 for sub := range p.subscribers {
  delete(p.subscribers, sub)
  close(sub)
 }
}

//发布一个主题
func (p *Publisher) Publish(v interface{}) {
 p.m.RLock()
 defer p.m.RUnlock()

 var wg sync.WaitGroup
 for sub, topic := range p.subscribers {
  wg.Add(1)
  go p.sendTopic(sub, topic, v, &wg)
 }
 wg.Wait()
}

//发送主题，可以容忍一定的超时
func (p *Publisher) sendTopic(sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup) {
 defer wg.Done()
 //topic的本质是过滤器函数，为nil表示不过滤，直接将所有消息发送给订阅者
 //不为nil的时候，表示一个过滤器，不满足特定条件直接过滤，满足的话在一定超时范围内可以将消息发给订阅者
 if topic != nil && !topic(v) {
  return
 }

 select {
 case sub <- v:
 case <-time.After(p.timeout):
 }
}

```

下面的例子中，有两个订阅者分别订阅了含有全部主题和含有“golang”主题：

```go
package main

import (
 "fmt"
 "strings"
 "test/pubsub"
 "time"
)

func main() {
 p := pubsub.NewPublisher(10, 100*time.Millisecond)
 defer p.Close()

 all := p.Subscribe()
 golang := p.SubscribeTopic(func(v interface{}) bool {
  if s, ok := v.(string); ok {
   return strings.Contains(s, "golang")
  }
  return false
 })

 p.Publish("hello, world")
 p.Publish("hello, golang")

 go func() {
  for msg := range all {
   fmt.Println("all:", msg)
  }
 }()

 go func() {
  for msg := range golang {
   fmt.Println("golang:", msg)
  }
 }()

 time.Sleep(3 * time.Second)
}

```

在这个模式下，每条消息都会发给多个订阅者，发布者通常不知道，也不关心哪一个订阅者在接收主题消息。订阅者和发布者都可以在运行时动态添加，他们之间是一种松散的耦合关系，这使得系统的复杂性随时间推移而增长。现实生活中，天气预报就可以应用这种并发模式。

### 1.6.4 控制并发数

有时候我们需要适当的控制并发的程度。实现原理就是，通过带缓存的通道的发送和接收规则来实现最大并发阻塞。不仅可以控制最大的**并发数目**，还可以通过缓存通道的使用量和最大容量的比值来判断程序运行的**并发率**。通道为空时可以认为是空闲状态，满则是繁忙状态。

### 1.6.5 胜者为王

采取并发编程的动机有很多：

- 简化问题：一个类对应一个处理线程
- 提升性能：多核上开两个线程一般比一个线程要快
- 快速响应：某个搜索引擎返回结果，就关闭其他页面

## 1.7 错误和异常

程序中总有一部分函数总是要求必须能执行成功，比如从切片中读写元素，map中取存在的元素等，除非程序有bug，或者遇到灾难性的不可预料的情况，遇到这些情况直接终止程序就好了。

排除异常的情况，如果程序运行失败仅仅被认为是几个预期结果之一。对于那些将失败看做是预期的结果的函数，会在返回值的最后一个来传递信息。若失败的原因只有一个，一般用一个bool值，有多个则用error接口。

在go中，**错误被认为是一种可以预期的结果，异常则是非预期的**。
