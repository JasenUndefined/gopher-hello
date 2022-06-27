Go 中使用 panic
---
theme: cyanosis
---
> 持续创作，加速成长！这是我参与「掘金日新计划 · 6 月更文挑战」的第28天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468 "https://juejin.cn/post/7099702781094674468")

## Go 没有try/catch

Go 没有异常类型，也没有像其他语言中通过 try/catch 的方式对错误的代码块进行包装。Go 倡导“Errorsare values!”的处理思想，它将 error 作为一个返回值，开发者对 error 进行处理或者忽略。常常处理的方式就是进行 if/else 判断。

```
func f() error {
    result , err := DoFunc();
    if err != nil {
        return error;
    }
    // ... 	其他逻辑
}
```

对于那些无法意料的错误，没有 try/catch 方法进行捕获，是否有其他的方式可以解决？

## Panic

Panic is a built-in function that stops the ordinary flow of control and begins panicking.

Panic 是一个内置函数，它可以停止普通的控制流程并开始发生程序异常。传递给 panic 函数的任意参数将在程序终止时打印出来。

```
func panic(interface{})  
```

## Panic 使用场景

在程序开发中，开发者应尽可能的去使用错误，只有在程序无法继续执行的情况下才去使用 panic 和 recover。panic 和 recover 可以当作是类似于 Java 等其他语言中的 try/catch 语法。所以可以在以下两种情况下使用：

-   开发者导致的错误，如绑定错误端口
-   发生不可恢复的错误导致程序无法继续执行可使用 panic 退出
-   程序启动时，强依赖服务出现故障时可使用 panic 退出
-   程序启动时，未符合要求的配置可使用 panic 退出

所以只要是出错时，程序都不应该支持启动成功，此时使用 panic 就可以阻止程序启动成功。所有其他错误，都应该使用 error 的方法返回错误信息。

## panic recover

```
defer func() {
	if err := recover(); err != nil {
		fmt.Println("Panic recover, Err =", err, "Stack =", string(debug.Stack()))
	}
}()

panic("something error")
```

函数在执行过程中，出现问题调用 panic 函数后抛出一个错误，同时函数的的正常执行流程将终止，但在 panic 之前定义了 defer 语句，那么 执行完 defer 后 goroutine 立即停止执行。

defer 函数中调用了 recover 函数，主要用于捕获 panic 的错误信息。所以遇到 panic 时程序将进入 defer ，执行 recover 恢复程序，并打印了错误信息和堆栈信息。

以下情况下，将发生不同的结果：

-   函数中有 panic 有 recover，程序不会终止
-   函数中有 panic 无 recover ，程序直接终止
-   子函数有 panic 有 recover ，不影响主函数的执行
-   子函数有 panic 主函数有 recover，程序不会终止
-   子协程有 panic ，主函数有 recover ，程序将会终止
-   同一子协程有 panic 有 recover ，主函数程序不受影响

所以，如果在同一个协程内，recover 可以捕获任何位置的 panic ，如果不是在同一个协程内则无法捕获，导致程序终止执行。
