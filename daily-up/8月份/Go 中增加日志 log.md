# Go 中增加日志 log

## 日志

日志是一个程序必不可少的功能。当功能错误时可以协助我们快速定位问题的根源，追踪程序执行过程发生的数据变化。还有一种日志就是记录用户行为日志，如对某功能的增删该的操作日志，这类日志一般是提供给用户查看，可读性要求比较高。

## Go 中的日志

go 内置了 log 包，可以通过调用 log 函数，实现简单的日志打印。

```go
func main() {
    defer fmt.Println("panic退出前处理")
    log.Println("println日志")
    log.Panic("panic日志")
    log.Fatal("程序退出日志")
}
```

如果只是简单使用 log 进行打印日志的话，可读性比较差。为了通过日志进行定位，追踪问题。每条日志应当配置日期、时间、日志类型，日志信息等相关信息。

## 配置日志

```go
type LogLevel string

const (
    DEBUG LogLevel = "DEBUG"
    WARN  LogLevel = "WARN"
    INFO  LogLevel = "INFO"
    ERROR LogLevel = "ERROR"
)

type logUtil struct {
    level      LogLevel
    loggerDate time.Time
    message    string
}

func LoggerNew() *logUtil {
    return &logUtil{}
}

func (log *logUtil) INFO(message string) {
    log.level = INFO
    fmt.Printf("[%s] %s %s\n", log.level, time.Now().String(), message)
}
func (log *logUtil) WARN(message string) {
    log.level = WARN
    fmt.Printf("[%s] %s %s\n", log.level, time.Now().String(), message)
}
func (log *logUtil) Debug(message string) {
    log.level = DEBUG
    fmt.Printf("[%s] %s %s\n", log.level, time.Now().String(), message)
}
func (log *logUtil) ERROR(message string) {
    log.level = ERROR
    fmt.Printf("[%s] %s %s\n", log.level, time.Now().String(), message)
}
```

- 创建 logUtil 结构体，里面含有日志类型，日志时间和日志信息字段。

- 定义一个枚举日志类型 LogLevel，主要有 info、warn、debug、error 四种类型。

- 给 logUtil 绑定4个对应日志类型的方法，输出日志信息即可

- 定义一个 LoggerNew 函数返回日志实体 实现了自定义的logger

在使用时，直接调用对应的方法即可直接输出最终结果：

```go
func main(){
    LoggerNew().Debug("测试logger 打印 错误信息")
}

// 输出一条日志信息
[DEBUG] 2022-08-03 12:40:40.604241 +0800 CST m=+0.000610459 测试logger 打印 错误信息z在印日志输出在控制台上
```

在一个项目日志输出在控制台上，不推荐方式。应当将其输出到文件中。给 logUtil 结构体增加文件filepath 字段。再在 LoggerNew 函数中设置文件路径。以 Error 方法为例子，增加将日志写入文本中。

```go
type logUtil struct {
    level      LogLevel
    loggerDate time.Time
    message    string
    filePath   string
}

func LoggerNew() *logUtil {
    return &logUtil{
        filePath: fmt.Sprintf("./logs/%s.log", time.Now().Format("yyyy-dd-mm")),
    }
}

func (lg *logUtil) ERROR(message string) {
    lg.level = ERROR
    fmt.Printf("[%s] %s %s\n", lg.level, time.Now().String(), message)

    logFile, err := os.OpenFile(lg.filePath, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        log.Panic("打开日志文件异常")
    }
    log.SetOutput(logFile)
}
```
