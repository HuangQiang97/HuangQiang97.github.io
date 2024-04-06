[toc]

### Go程

1，Go 程（goroutine）是由 Go 运行时管理的轻量级线程， 

```GO
go f(x, y, z) // 会启动一个新的 Go 程并执行  f(x, y, z)，
```

2，在单一Go程中，事件发生的顺序即为程序所表达的顺序。

3，若以下条件均成立，则对变量 `v` 的读取操作 r 就允许对 `v` 的写入操作 w 进行监测（r读取w写入内容）：`r `不发生在` w `之前；在 `w` 之后 `r` 之前，不存在其它对 `v` 进行的写入操作` w`。

4，当多个Go程访问共享变量 `v` 时，它们必须通过同步事件而不是程序顺序来建立发生顺序的条件，以此确保读取操作能监测到预期的写入。

```go
var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f() // go 语句会在当前Go程开始执行前启动创建的新的的Go程。
}
// 输出空字符串
fmt.Println(a)
```

5，Go程无法确保在程序中的任何事件发生之前退出。若一个Go程的作用必须被另一个Go程监测到，需使用锁或信道通信之类的同步机制来建立顺序关系。读取操作 r 可能监测到与其并发的写入操作 w 写入的值。并不意味着发生在 r 之后的读取操作会监测到发生在 w 之前的写入操作。要使用显式的同步解决这些问题。

```go
var a, b int
func f() {
	a = 1 //a,b多步操作，非原子性
	b = 2
}
func g() {
	print(b)
	print(a)
}
 // 2 0,在其他线程中能检测到b发生变化，未必能检测到a也发生了变化。
func main() {
	go f()
	g()
}
```

```go
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "hello, world"
	g = t
}
// 即便 main 能够监测到 g != nil 并退出循环， 它也无法保证能监测到 g.msg 的初始化值。 线程缓存未及时写入主存。
func main() { 
	go setup()
	for g == nil {
	}
	print(g.msg)
}
```

### 信道

1，Go 程在相同的地址空间中运行，因此在访问共享的内存时必须进行同步。信道是带有类型的管道，可以通过信道操作符 `<-、->` 来发送或者接收值。  	默认情况下，发送和接收操作在另一端准备好之前都会阻塞。


```go
ch := make(chan int) //创建信道
ch <- v    // 将 v 发送至信道 ch。
v := <-ch  // 从 ch 接收值并赋予 v。
```

---

2，通道的的基本特性：

*  对于同一个通道，发送操作之间是互斥的，接收操作之间也是互斥的。在同一时刻，Go 语言的运行时系统只会执行对同一个通道的任意个发送操作中的某一个。直到这个元素值被完全复制进该通道之后，其他针对该通道的发送操作才可能被执行。在同一时刻，运行时系统也只会执行对同一个通道的任意个接收操作中的某一个。直到这个元素值完全被移出该通道之后，其他针对该通道的接收操作才可能被执行。

* 发送操作和接收操作中对元素值的处理都是不可分割的。对于通道中的同一个元素值来说，发送操作和接收操作之间也是互斥的。元素值从外界进入通道时会被复制。进入通道的并不是在接收操作符右边的那个元素值，而是它的副本（浅拷贝,针对缓冲通道）。元素值从通道进入外界时会被复制（浅拷贝,针对缓冲通道）,第一步是生成正在通道中的这个元素值的副本，并准备给到接收方，第二步是删除在通道中的这个元素值。缓冲通道会作为收发双方的中间件。元素值会先从发送方复制到缓冲通道，之后再由缓冲通道复制给接收方（浅拷贝,针对缓冲通道）。当发送操作在执行的时候发现空的通道中，正好有等待的接收方，那么它会直接把元素值复制给接收方。复制方式为浅拷贝：只是拷贝值以及值中直接包含的东西， 对于复合结构只拷贝他的直接成员。处理元素值时都是一气呵成的，绝不会被打断（原子性）。发送操作要么还没复制元素值，要么已经复制完毕，绝不会出现只复制了一部分的情况。接收操作在准备好元素值的副本之后，一定会删除掉通道中的原值，绝不会出现通道中仍有残留的情况。无缓冲信道不存在缓冲拷贝，发送发直接阻塞，当接收方到来时直接复制给接收方。

* 发送操作在完全完成之前会被阻塞。接收操作也是如此。发送操作包括了“复制元素值”和“放置副本到通道内部”这两个步骤。在这两个步骤完全完成之前，发起这个发送操作的那句代码会一直阻塞在那里。接收操作通常包含了“复制通道内的元素值”“放置副本到接收方”“删掉原值”三个步骤。在所有这些步骤完全完成之前，发起该操作的代码也会一直阻塞。阻塞代码其实就是为了实现操作的互斥和元素值的完整。当多个发送操作被阻塞，现信道可用时：通道会优先通知最早因此而等待的、那个发送操作所在的 goroutine，后者会再次执行发送操作。如果通道已空，那么对它的所有接收操作都会被阻塞，直到通道中有新的元素值出现。通道会通知最早等待的那个接收操作所在的 goroutine，并使它再次执行接收操作。对于值为`nil`的通道，不论它的具体类型是什么，对它的发送操作和接收操作都会永久地处于阻塞状态。


3，信道是双向的，但只能接受到对方法发来的消息（半双工）使用信道Go 程可以在没有显式的锁或竞态变量的情况下进行同步。  

```go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 将和送入 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}
	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // 从 c 中接收，在两个go程都运行完前当前go程在阻塞
	fmt.Println(x, y, x+y)
}

```

4，单向通道：只能发送数据：make(chan  <- int, 1)；只能接受数据:   make(<- chan int, 1)，通常声明一个只有一端（发送端或者接收端）能用的通道没有任何意义，单向通道最主要的用途就是约束其他代码的行为。调用时：只需要把一个元素类型匹配的双向通道传给它就行了， Go 语言在这种情况下会自动地把双向通道转换为函数所需的单向通道。在方法内部只能使用函数声明的单向信道的功能。

```GO
type Notifier interface {
	SendInt(ch chan<- int) //在该接口的所有实现类型中的SendInt方法都会受到限制。
}
```

5，它将共享的值通过信道传递，实际上，多个独立执行的线程从不会主动共享。在任意给定的时间点，只有一个Go程能够访问该值。不要通过共享内存来通信，而应通过通信来共享内存。

6，无缓冲信道（容量为0）在通信时会同步交换数据，它能确保（两个Go程的）计算处于确定状态。无缓冲信道不能存储数据，往里面写数据后写go程会被阻塞，直到里面的数据被取走。无论是发送操作还是接收操作，一开始执行就会被阻塞，直到配对的操作也开始执行，才会继续传递。非缓冲通道是在用同步的方式传递数据。只有收发双方对接上了，数据才会被传递。数据是直接从发送方复制到接收方的，中间并不会用非缓冲通道做中转。相比之下，缓冲通道则在用异步的方式传递数据。信道可以是 带缓冲的。将缓冲长度作为第二个参数提供给 `make` 来初始化一个带缓冲的信道：  

```go
ch := make(chan int, 100)
```

​		 仅当信道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接受方会阻塞。  若信道是不带缓冲的，那么在接收者收到值前， 发送者会一直阻塞；若信道是带缓冲的，则发送者仅在值被复制到缓冲区前阻塞；若缓冲区已满，发送者会一直等待直到某个接收者取出一个值空出一个缓冲区为止。

7，通道的长度(len())代表通道当前包含的元素个数，容量(cap())就是初始化时你设置的那个数。

8，由于数据同步发生在信道的接收端（发送发生在接受之前），信号必须在信道的接收端获取，而非发送端。

9，带缓冲的信道可被用作信号量，例如限制吞吐量。

方案一：尽管只有 `MaxOutstanding` 个Go程能同时运行，但 `Serve` 还是为每个进入的请求都创建了新的Go程,只是只有`MaxOutstabding`个操作同时进行，其它操作还是被创建出来，只是Go程被阻塞。若请求来得很快， 该程序就会无限地消耗资源。

```go

var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
	sem <- 1 // 往信道中写数据，标志占用一个资源
	process(r)  // 可能需要很长时间。
	<-sem    // 往信道中取数据，标志释放一个资源
}

func Serve(queue chan *Request) {
	for {
		req := <-queue
		go handle(req)  // 无需等待 handle 结束。
	}
}
```

方案二 ：循环变量`req`在每次迭代时会被重用，因此 `req` 变量会被在所有的Go程间共享

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) 
            <-sem
        }()
}
```
 我们需要确保 `req` 对于每个Go程来说都是唯一的。可以将 `req` 的值作为实参传入到该Go程的闭包中来实现：

```go
func Serve(queue chan *Request) {
	for req := range queue {
		sem <- 1
		go func(req *Request) { //将当前req与函数绑定，立即执行
			process(req)
			<-sem
		}(req)
	}
}
```

方案三：以相同的名字创建新的变量，`req := req`在Go中这样做是合法且惯用的。用相同的名字获得了该变量的一个新的副本， 以此来局部地刻意屏蔽循环变量，使它对每个Go程保持唯一。

```go
func Serve(queue chan *Request) {
	for req := range queue {
		req := req // 为该Go程创建 req 的新实例。
		sem <- 1
		go func() {
			process(req)
			<-sem
		}()
	}
}
```

方案四：启动固定数量的 `handle` Go程，一起从请求信道中读取数据。Go程的数量限制了同时调用 `process` 的数量。`Serve` 同样会接收一个通知退出的信道， 在启动所有Go程后，它将阻塞并暂停从信道中接收消息。

```go
func handle(queue chan *Request) {
	for r := range queue { // 从quene中取出还没被处理的请求，quene长度减一
		process(r)
	}
}

func Serve(clientRequests chan *Request, quit chan bool) {
	// 启动处理程序
	for i := 0; i < MaxOutstanding; i++ {
		go handle(clientRequests) //MaxOutstanding个handle同时从clientRequests获取任务处理请求
	}
	<-quit  // 等待通知退出。
}
```
10，信道是数值，它可以被分配并像其它值到处传递。 这种特性通常被用来实现安全、并行的多路分解。若该类型包含一个可用于回复的信道， 那么每一个客户端都能为其回应。以下为 `Request` 类型的大概定义。

```go
var sem = make(chan int, MaxOutstanding)
var clientRequests=make(chan *Request,10)
// 客户端提供了一个函数及其实参，此外在请求对象中还有个接收应答的信道。
type Request struct {
	args        []int
	f           func([]int) int
	resultChan  chan int
}
func sum(a []int) (s int) {
	for _, v := range a {
		s += v
	}
	return
}

func handle(req *Request) {
		req.resultChan <- req.f(req.args)
}

func Serve(queue chan *Request) {
	for req := range queue {
		sem <- 1
		go func(req *Request) { //闭包将当前req与函数绑定
			handle(req)
			<-sem
		}(req)
	}
}



func main(){
    request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
    // 发送请求
    clientRequests <- request
    Serve(clientRequests)
    // 等待回应
    fmt.Printf("answer: %d\n", <-request.resultChan)
}
```

11，发送者可通过 `close` 关闭一个信道来表示没有需要发送的值了。接收者可以通过为接收表达式分配第二个参数来测试信道是否被关闭：`v, ok := <-ch`如果通道关闭时，里面还有元素值未被取出，v仍会是通道中的某一个元素值，而ok一定会是`true`。通过ok，来判断通道是否关闭是可能有延时的。若没有值可以接收且信道已被关闭:`ok =false`，不要让接收方关闭通道，而应当让发送方做这件事。向一个已经关闭的信道发送数据会引发程序恐慌（panic）。  试图关闭一个已经关闭了的通道，也会引发 panic，若在信道关闭后从中接收数据，并且信道内为空，接收者就会收到该信道返回的零值。信道与文件不同，通常情况下无需关闭它们。只有在必须告诉接收者不再有需要发送的值时才有必要关闭

```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c) //终止一个 range 循环，不然range不会终止。  
}
func main() {
	c := make(chan int, 5)
	go fibonacci(10, c)
	for i := range c { //依据len调用，在取出值和加入值后len都会变化
		fmt.Println(i)
	}
}
```

12，循环 `for i := range c` 会不断从信道接收值,等价于`i<-c`，当无数据可以读取时就会堵塞，直到它被关闭。 

13， `select` 语句使一个 Go 程可以等待多个通信操作。 ：每个`case`表达式中都必须包含通道的读或者写；`select`语句会查看哪些case的读写操作能成功执行，只是查看能否执行，不是真的执行，然后开始选择能成功执行的候选分支，进行读写操作，执行对应case内容，然后结束当前select ；当多个分支都准备好时会随机选择一个执行case，而随机的引入就是为了避免饥饿问题的发生，然后结束当前select 。如果所有的候选分支都不满足选择条件，那么默认分支就会被执行，如果这时没有默认分支，那么`select`语句就会立即进入阻塞状态，直到至少有一个候选分支满足选择条件为止。一旦有一个候选分支满足选择条件，`select`语句就会被唤醒，这个候选分支就会被执行。

https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-select/

为了在尝试发送或者接收时不发生阻塞，可使用 `default` 分支。如果在`select`语句中发现某个通道已关闭，可以把该信道设为 nil 屏蔽掉它所在的分支，简单地在`select`语句的分支中使用`break`语句，只能结束当前的select语句的执行，在`select`语句与`for`语句联用时可以设置标志位与goto实现跳出循环。

```go
//break 方式
oop:
	for {
		select { 
		case _, ok := <-ch1: //ch1非空或许信道被关闭且没有值时执行此语句
			if !ok {
				ch1 = nil //ch1已经关闭且没有值，将他设置为nil，以屏蔽ch1
			}
			fmt.Println("ch1")
		case _, ok := <-ch2: //ch2非空或许信道被关闭且没有值时执行此语句
			if !ok {
				break loop //跳出for循环
			}
			fmt.Println("ch2")
		default: // 所有分支都阻塞时执行此分支
			time.Sleep(50 * time.Millisecond)
		}
	}
	fmt.Println("END")
// goto方式
for {
	select { 
		case _, ok := <-ch1: //ch1非空或许信道被关闭且没有值时执行此语句
			if !ok {
				ch1 = nil //ch1已经关闭且没有值，将他设置为nil，以屏蔽ch1
			}
			fmt.Println("ch1")
		case _, ok := <-ch2: //ch2非空或许信道被关闭且没有值时执行此语句
			if !ok {
				goto loop //跳出for循环
			}
			fmt.Println("ch2")
		default: // 所有分支都阻塞时执行此分支
			time.Sleep(50 * time.Millisecond)
		}
	}
loop:
	fmt.Println("END")
```

### 互斥锁

1，保证每次只有一个 Go 程能够访问一个共享的变量，Go 标准库中提供了 `sync.Mutex` 互斥锁类型及其两个方法：  `Lock`，`Unlock`，可以通过在代码前调用 `Lock` 方法，在代码后调用 `Unlock` 方法来保证一段代码的互斥执行。可以用 `defer` 语句来保证互斥锁一定会被解锁。

```go
// SafeCounter 的并发使用是安全的。
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc 增加给定 key 的计数器的值。
func (c *SafeCounter) Inc1(key string) {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	c.v[key]++
	c.mux.Unlock()
}

// Value 返回给定 key 的计数器的当前值。
func (c *SafeCounter) inc2(key string)  {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	defer c.mux.Unlock()
	c.v[key]++
}

c := SafeCounter{v: make(map[string]int)}
for i := 0; i < 1000; i++ {
    go c.Inc1("somekey1")
    go c.Inc1("somekey2")
}
```

2，`sync` 包实现了两种锁的数据类型：`sync.Mutex` 和 `sync.RWMutex`。对于任何 `sync.Mutex` 或 `sync.RWMutex` 类型的变量 `l` 以及 *n* < *m* ，对 `l.Unlock()` 的第 *n* 次调用在对 `l.Lock()` 的第 *m* 次调用返回前发生。

3，`sync` 包通过 `Once` 类型为存在多个Go程的初始化提供了安全的机制。多个go程可为特定的 `f` 执行 `once.Do(f)`，启动时没有任何go程被执行完毕，标志位为0，多个go程竞争一个同步锁，竞争成功的go程获得同步锁，其它线程阻塞，在他执行完毕后将执行标志位写为1并释放锁，其他go程再次开始竞争锁，拿到锁后检查标志位，发现标志位为1，直接返回并释放锁，其它go程再次竞争，如此往复。最终只有一个go程会成功运行 `f()`。

```go
type Once struct {
	// done indicates whether the action has been performed.
	// It is first in the struct because it is used in the hot path.
	// The hot path is inlined at every call site.
	// Placing done first allows more compact instructions on some architectures (amd64/386),
	// and fewer instructions (to calculate offset) on other architectures.
    // 热路径是非常频繁执行的一系列指令。假定字段值在内存中的布局与结构定义中的相同，访问结构的第一个字段时，我们可以直接引用对结构的指针以访问第一个字段。
    // 要访问其他字段，除了结构指针之外，我们还需要提供与第一个值的偏移量。不同于数组，数组中元素类型相同，占用内存大小相同，可以通过起始地址+元素下标*元素大小直接获得目标地址，在结构体中各个字段类型不同，占用大小不同，需要挨个累加才能获得目标地址。
    // 在机器代码中，此偏移量是随指令传递的附加值，这会使机器指令更长。对性能的影响是CPU必须对struct指针执行偏移量的加法运算以获得要访问的值的地址。因此，用于访问结构的第一个字段的机器代码更加紧凑和快速。

	done uint32
	m    Mutex
}

func (o *Once) Do(f func()) {
	// Note: Here is an incorrect implementation of Do:
	//	CAS策略，乐观锁
	//	if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
	//		f()
	//	}
	//
	// Do guarantees that when it returns, f has finished.
	// This implementation would not implement that guarantee:
	// given two simultaneous calls, the winner of the cas would
	// call f, and the second would return immediately, without
	// waiting for the first's call to f to complete.
	// This is why the slow path falls back to a mutex, and why
	// the atomic.StoreUint32 must be delayed until after f returns.

	if atomic.LoadUint32(&o.done) == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}

```



```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}
	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}
	for i := 0; i < 10; i++ {
		<-done
	}
}

# Output:
Only once
```

### 并行与并发

1，并发是用可独立执行的组件构造程序的方法， 并行则是为了效率在多CPU上平行地进行计算。如果计算过程能够被分为几块 可独立执行的过程，它就可以在每块计算结束时向信道发送信号，从而实现并行处理。

```go
type Vector []float64

// 将此操作应用至 v[i], v[i+1] ... 直到 v[n-1]
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
	for ; i < n; i++ {
		v[i] += u.Op(v[i])
	}
	c <- 1    // 发信号表示这一块计算完成。
}
```

在循环中启动了独立的处理块，每个CPU将执行一个处理。它们有可能以乱序的形式完成并结束，只需在所有Go程开始后接收，并统计信道中的完成信号即可。

```go
const NCPU = 4  // CPU核心数

func (v Vector) DoAll(u Vector) {
	c := make(chan int, NCPU)  // 缓冲区是可选的，但明显用上更好
	for i := 0; i < NCPU; i++ {
		go v.DoSome(i*len(v)/NCPU, (i+1)*len(v)/NCPU, u, c)
	}
	// 排空信道。
	for i := 0; i < NCPU; i++ {
		<-c    // 等待任务完成
	}
	// 一切完成。
}
```

2，Go是种并发而非并行的语言，且Go的模型并不适合所有的并行问题。目前Go运行时的实现默认并不会并行执行代码，它只为用户层代码提供单一的处理核心。 任意数量的Go程都可能在系统调用中被阻塞，而在任意时刻默认只有一个会执行用户层代码。若希望CPU并行执行， 就必须告诉运行时你希望同时有多少Go程能执行代码。导入 `runtime` 包,通过`runtime.NumCPU()` 得到当前机器的逻辑CPU核心数。调用`runtime.GOMAXPROCS(NCPU)`。 

### 初始化

1，每个源文件都可以通过定义自己的无参数 `init` 函数来设置一些必要的状态。 运行顺序：导入的包中全局变量的初始化 -> 导入的包中的`init()` -> 当前文件中全局变量的初始化 -> 当前文件中的`init()` -> 当前文件中`main()`

### 异常

1，内建的 `panic` 函数，它会产生一个运行时错误并终止程序。该函数接受一个任意类型的实参（一般为字符串），并在程序终止时打印。`func panic(v interface{})`

2，实际的库函数应避免 `panic`。若问题可以被屏蔽或解决， 最好就是让程序继续运行而不是终止整个程序。一个可能的反例就是初始化： 若某个库真的不能让自己工作，且有足够理由产生Panic。当 `panic` 被调用后，程序将立刻终止当前函数的执行，并开始回溯Go程的栈，依据入栈顺序`FILO`运行被推迟(defer)的函数。若回溯到达Go程栈的顶端，程序就会终止。调用 `recover` 将停止回溯过程，并返回传入 `panic` 的实参。由于在回溯时只有被推迟函数中的代码在运行，因此 `recover` 只能在被推迟的函数中才有效。`recover` 的一个应用就是在服务中终止失败的Go程而无需杀死其它正在执行的Go程。

```go
func server(workChan <-chan *Work) {
	for work := range workChan {
		go safelyDo(work)
	}
}

func safelyDo(work *Work) {
    //若 `do(work)` 触发了Panic，其结果就会被记录， 而该Go程会被干净利落地结束，不会干扰到其它Go程。
	defer func() {
		if err := recover(); err != nil {
			log.Println("work failed:", err)
		}
	}()
	do(work)
}
```

### 测试

1，Go拥有一个轻量级的测试框架，它由 `go test` 命令和 `testing` 包构成。通过创建一个名字以 `XXX_test.go` 结尾的，包含名签名为 `func TestXXX (t *testing.T)` 函数的文件来编写测试。 测试框架会运行每一个这样的函数；若该函数调用了像 `t.Error` 或 `t.Fail` 这样表示失败的函数，此测试即表示失败。

```go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

接着使用 `go test` 运行该测试：

```shell
$ go test stringutil
```

若在包目录下运行 `go` 工具，也可以忽略包路径

```shell
$ go test
```

### 动态库与静态库

1，静态连接库就是把(lib)文件中用到的函数代码直接链接进目标程序，程序运行的时候不再需要其它的库文件；动态链接就是把调用的函数所在文件模块（DLL）和调用函数在文件中的位置等信息链接进目标程序，程序运行的时候根据函数映射表再从DLL中寻找相应函数代码，然后调入堆栈执行，因此需要相应DLL文件的支持。

2，如果采用静态链接库，lib 中的指令都全部被直接包含在最终生成的 EXE  文件中了。若使用动态链接库 DLL，该 DLL 不必被包含在最终 EXE 文件中，EXE 文件执行时可以“动态”地引用和卸载这个与 EXE 独立的  DLL  文件。

3，静态链接库中不能再包含其他的动态链接库或者静态库。在动态链接库中还可以再包含其他的动态或静态链接库。

4，动态库中：如果在当前工程中有多处对dll文件中同一个函数的调用，那么执行时，这个函数只会留下一份拷贝。静态库中：如果有多处对lib文件中同一个函数的调用，那么执行时，该函数将在当前程序的执行空间里留下多份拷贝，而且是一处调用就产生一份拷贝。

5，静态链接库的特点：代码装载速度快，执行速度略比动态链接库快； 只需保证在开发者的计算机中有正确的.LIB文件，在以二进制形式发布程序时不需考虑在用户的计算机上.LIB文件是否存在及版本问题，可避免DLL地狱等问题。 使用静态链接生成的可执行文件体积较大，包含相同的公共代码，造成浪费；

5，动态链接库的特点：更加节省内存并减少页面交换；DLL文件与EXE文件独立，只要输出接口不变（即名称、参数、返回值类型和调用约定不变），更换DLL文件不会对EXE文件造成任何影响，因而极大地提高了可维护性和可扩展性；不同编程语言编写的程序只要按照函数调用约定就可以调用同一个DLL函数；适用于大规模的软件开发，使开发过程独立、耦合度小，便于不同开发者和开发组织之间进行开发和测试。使用动态链接库的应用程序不是自完备的，它依赖的DLL模块也要存在，如果使用载入时动态链接，程序启动时发现DLL不存在，系统将终止程序并给出错误信息。而使用运行时动态链接，系统不会终止，但由于DLL中的导出函数不可用，程序会加载失败；速度比静态链接慢。当某个模块更新后，如果新模块与旧的模块不兼容，那么那些需要该模块才能运行的软件将无法运行。

### 命令行参数

1，自定义参数说明

```go
//flag.ExitOnError参数错误时退出，PanicOnError：参数错误时触发恐慌
flag.CommandLine = flag.NewFlagSet("", flag.ExitOnError) 
// 自定义帮助输出
flag.CommandLine.Usage = func() {
	// 自定义片段
    fmt.Fprintf(os.Stderr, "Usage of %s:\n", "question")
    // 默认片段
    flag.PrintDefaults()
}
```

2，参数绑定：支持的参数:`int,floatstring,bool,duration(时间),var(自定义，要实现flag包里的Value接口，然后使用flag.Var())`，两个flag分享一个变量时可以用来做命令参数缩写，要确保两者使用同一默认值，并且他们必须在init()函数中设置。

```go
// 参数地址，参数名称，默认值，说明
flag.StringVar(&name, "name", "everyone", "The greeting object.")
```

3，参数解析

```go
flag.Parse()
```

```go
package main
import (
	"flag"
	"fmt"
	"os"
)
// 变量声明
var name string
func init() {
    // 命令行初始化
	flag.CommandLine = flag.NewFlagSet("", flag.ExitOnError)
	flag.CommandLine.Usage = func() {
		fmt.Fprintf(os.Stderr, "Usage of %s:\n", "question")
		flag.PrintDefaults()
	}
    // 参数绑定
	flag.StringVar(&name, "name", "everyone", "The greeting object.")
}
func main() {
    // 参数解析
	flag.Parse()
	fmt.Printf("Hello, %s!\n", name)
}
// 执行顺序： 变量声明 -> 初始化 -> 参数绑定 -> 参数解析
```

```bash
go run demo.go -name=name
go run demo.go --help
```

