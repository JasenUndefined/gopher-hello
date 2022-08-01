Go 中的 select 语句

多个通道 Channel 中信息的发送和接受处理的专用的语句—select 语句。select  语句会阻塞，直到其中的一个发送/接收操作准备好。select 语句和 switch 语句有点相似，但 select 语句在被执行时会选择执行其中的一个分支，且选择分支的方法完全是不相同的。

```go
ch1 = make(chan string)
ch2 = make(chan string)

ch1 <- "server1"
ch2 <- "server1"

select {
    case i := <- ch1:
      fmt.Printf("从ch1读取了数据%d", i)
    case j := <- ch2:
      fmt.Printf("从ch1读取了数据%d", i)
    default:
      fmt.Printf("no action...", i)
}
```

以上代码中，每个 case 后都只针对某个通道的接收语句，这个和 switch 不同，也没有 break。switch 语句右边是一个switch 表达式，但 select 右边是接大括号。

开始执行 select 语句时，所有跟在 case 关键字右边的表达式都会被求值，求值的顺序是自上而下，从左到右的。
