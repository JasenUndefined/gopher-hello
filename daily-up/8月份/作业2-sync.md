# GO中使用 sync 包自由的控制并发

## 资源竞争

channel 常用于并发通信，要保证并发安全，主要使用互斥锁。在并发的过程中，当一个内存被多个 goroutine 同时访问时，就会产生资源竞争的情况。这块内存也可以称为共享资源。

并发时对于共享资源必然会出现抢占资源的情况，如果是对某资源的统计，很可能就会导致结果错误。为保证只有一个协程拿到资源并操作它，可以引入互斥锁 sync.Mutex。

## sync.Mutex

互斥锁，指的是并发时，在同一时刻只有一个协程执行某段代码，其他协程只有等待该协程执行完成后才能继续执行。

```go
var (sum int 
 mutex sync.Mutex)


func add(i int){
    mutex.Lock()
    sum+=i
    mute.Unlock()
}
```

使用 mutex 加锁保护 sum+ =i 的代码片段，这样这个片段区就无法同时被多个协程访问，当有协程进入该片段区，那其他的协程就只有等待，以此保证临界区的并发安全。

sync.Mutex 只有 Lock()和 Unlock() 方法，它们是成对存在的，且Lock后一定要执行Unlock进行释放锁。所以可以使用 defer 语句释放锁，以保证锁一定会被释放。

```go
func add(i int){
    mutex.Lock()
    defer mutex.Unlock()
    sum += i
}
```

## sync.RWMutex

上面例子是对 sum 写操作时使用sync.Mutex 保证并发安全，当多个协程进行读取操作时，避免因并发产生读取数据不正确，也是可以使用互斥锁 sync.Mutex。

```go
func getSum(){
    mutex.Lock()
    defer mutex.Unlock()
    b:=sum
    return b
}
```

多个协程 goroutine 同时读写的资源竞争问题解决，还需要考虑性能问题，每次读写共享资源都加锁，也会导致性能低。

多个协程并发进行读写时会遇到以下情况：

- 写时不能同时读，易读到脏数据

- 读时不能同时写，因为会导致结果不一致

- 读时同时写，因数据不变，无论多少个 goroutine 读都是并发安全

使用读写锁 sync.RWMutex 优化代码：

```go
var mutex sync.RWMutex
func readSum() int {
    mutex.RLock()
    defer mutex.RUnlock()
    b := sum
    return b
}
```

读写锁的性能比互斥锁性能好。

## sync.WaitGroup

为了能够监听所有的协程的执行，一旦所有的goroutine 都执行完成，程序应当及时退出节省时间提高性能。通过使用 sync.WaitGroup 来解决。使用步骤如下：

- 声明一个 sync.WaitGroup ,通过 add 方法增加计数器值，有几个协程就计算几个

- 每个协程结束后就调用 Done 方法，主要是让计数器减1

- 最后调用 Wait 方法一直等待，直到计数器为 0 时，所有跟踪的协程都执行完毕

```go
func run() {
	var wg sync.WaitGroup
	wg.Add(100)
	for i := 0; i < 100; i++ {
		go func() {
			defer wg.Done()
			add(10)
		}()
	}
    wg.Wait()
}
```

通过 sync.WaitGroup 可以很好地跟踪协程.

## sync.Once

sync.Once 作用是让代码只执行一次。详细使用是调用方法 once.Do 方法，具体实现：

```go
func main(){
    var once sync.once
    oneFunc := func(){
        println("once func")
    }
    once.Do(oneFunc)
}
```

sync.Once 适用于创建某个对象的单例、只加载一次的资源等只执行一次的场景。

## sync.Cond

使用 sync.WaitGroup 主要是控制等待所有的协程都执行完毕，才最终完成。但是当遇到场景是，只有等待所有条件都准备好才开始。sync.Cond 相当于发号施令，只有通知执行所有的协程才可以执行，重点是所有协程需等待唤醒才可以开始。

所以 sync.Cond 具有阻塞协程和唤醒协程的功能。详细的用法：

- 通过 sync.NewCond 函数生成一个 *sync.Cond，用于阻塞和唤醒协程

- 调用 cond.Wait() 方法阻塞当前协程等待，需要注意调用 cond.Wait() 方法要加锁

- 调用 cond.Broadcast() 后所有协程才开始执行

```go
func run() {
	cond := sync.NewCond(&sync.Mutex{})
	var wg sync.WaitGroup
	wg.Add(101)
	for i := 0; i < 100; i++ {
		go func(num int) {
			defer wg.Done()
			fmt.Println(num, "号正在 awaiting......")
			cond.L.Lock()
			cond.Wait() //等待所有协程准备完成
			fmt.Println(num, "号开始跑……")
			cond.L.Unlock()
		}(i)
	}
	// 等待所有的协程都进入 wait 状态
	time.Sleep(2*time.Second)
	go func() {
		defer wg.Done()
		// 所有都准备完成，开始
		cond.Broadcast()
	}()
	
	// 防止函数提前返回退出
	wg.Wait()
}
```


