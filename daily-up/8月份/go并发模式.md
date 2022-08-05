# Go 的并发模式

> 携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第6天，[点击查看活动详情

类似于设计模式一样，为更好应对异步和并发的开发，对这类的场景进行抽象封装，以便提供一个统一的解决方案。

## for select 模式

```go
// for无限循环，或 for range 循环
for{
    select {
        // 使用一个 channel 控制
    }
}

// 例子1
for{
    select {
     case <- done:
      return
     case result <- s:
        // do something
    }
}
// 例子 2
for{
    select {
     case <- done:
      return
     default:
        // do something
    }
}
```

通过使用 for 循环+ select 的并发模式，只要有 case 分支满足条件就执行哪个，直到满足退出条件则退出循环。

如上面例子，done 分支主要用于退出循环，case result 分支主要用于接收循环的值，处理业务。也可以将此分支改为 default ，这样就会一直执行 default 分支的任务，直到 接收到 done 整个for 循环结束。

## select timeout 模式

当需要请求第三方服务数据时，由于响应时间的差别，为防止因第三方服务问题导致一直等待响应，需要设置一个超时时间。那么可以使用 select timeout 模式

```go
func doAction(){
    go func(){
     result <- getFromThirdApi()
    }()

    select {
        case v:= <- result:
            fmt.println("成功获取值")
        case <- time.After(5*time.second):
            fmt.println("网络超时")
    }
}
```

通过 time.After 函数设置一个超时时间，这样 select 分支就不会因异常造成select无限等待。

## Pipeline 模式

Pipeline 模式即为流水线模式，将一件事拆解成多个工序，然后由多个工序组装完成最后的工作。每道工序通过 channel 把数据传递给下一工序。

```go
func doFirst() <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for i := 0; i < 10; i++ {
            out <- spew.Sprint("配件", i)
        }
    }()
    return out
}
func doSecond(in <-chan string) <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for c := range in {
            out <- "组装（" + c + "）"
        }
    }()
    return out
}
func doThird(in <-chan string) <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for c := range in {
            out <- "打包（" + c + "）"
        }
    }()
    return out
}

func main() {
    da := doFirst()
    sa := doSecond(da)
    pa := doThird(sa)

    for p := range pa {
        fmt.Println(p)
    }
}
```

- 设置三道工序，doFirst、 doSecond、doThird，每道工序都将最后结果存储在 channel 内，然后作为下一个工序的参数

- 每道工序对应一个函数，函数内都有协程 goroutine 和 channel

- 最后一个组织者将工序串起来，完成整个流水线。也是整个数据的数据流

Pipeline 是一种并发处理的模式，而不是Go 中的管道 channel。主要是将每个步骤都拆解成各自独立工序，每个工序都有各自的 channel ，并将channel 输出作为下一次工序的入参。第一道工序可能只有输出，最后工序可能只有输入。

## Fade-In 扇入 和 Fade-out 扇出

- 多个协程 goroutine 同时从同一 Channel 读取数据，直到关闭，称为 Fade-Out。

- 多个协程 goroutine 从多个输入 channel 读取数据然后从同一个 channel 输出，直到所有的输入 channel 都关闭则将停止，称为 Fade-in

```go
func merge(ins ...<-chan string) <-chan string {
    var wg sync.WaitGroup
    out := make(chan string)

    p := func(in <-chan string) {
        defer wg.Done()
        for c := range in {
            out <- c
        }
    }
    wg.Add(len(ins))
    // 启动多个协程，将多个输入 channel都发送到 out
    for _, cs := range ins {
        go p(cs)
    }

    // 异步等待所有的输入数据都处理完后关闭输出out
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}



func main() {
    da := doFirst()

    sa1 := doSecond(da)
    sa2 := doSecond(da)
    sa3 := doSecond(da)
    sa := merge(sa1,sa2,sa3)

    pa := doThird(sa)

    for p := range pa {
        fmt.Println(p)
    }
}
```

增加 merge 工序，允许接收多个 channel，其中对每个 chan 都单独处理，并把处理结果都发送到变量 out。即多个协程并发，将多个chan 合并成一个。
