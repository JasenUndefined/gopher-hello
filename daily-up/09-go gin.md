 Gin 的分析学习

## Gin 简介

Gin是使用Golang 语言实现的 Http Web 框架。

特性：

- 快速：路由不使用反射，基于Radix树，内存占用少
- 中间件：Http 请求之前会经常一些列的中间件处理
- 异常处理：服务始终可用，不会宕机
- JSON：Gin可以解析并验证请求的JSON
- 路由分组：对API分组，不同版本分组，不同功能分组，也支持分组嵌套
- 渲染内置：原生支持JSON，XML和HTML的渲染。

## 使用 Gin 启动服务

使用时，安装 Gin 包，然后就可以创建 gin.Default() 实例

```shell
# 下载 gin 包
go get -u -v github.com/gin-gonic/gin  
```

创建一个 Gin 程序。创建一个 main.go 文件，然后输入以下代码

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.String(200, "Hello, Geektutu")
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

- 使用了`gin.Default()`生成了一个实例，这个实例即 WSGI 应用程序
- `r.Get("/", ...)`声明了一个路由。在浏览器上就可以通过这个 Url 进行请求
- `r.Run()`函数来让应用运行在本地服务器上，默认监听端口是 _8080_，可以传入参数设置端口，例如`r.Run(":8001")`即运行在 _8001_端口。

## gin.Default

```go
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```

1、 `gin.Default()` 方法会创建默认的 Engine 实例，同时附加了 Logger 和 Recovery 中间件

- Logger：输出请求日志，并标准化日志的格式。
- Recovery：异常捕获，也就是针对每次请求处理进行 recovery 处理，防止因为出现 panic 导致服务崩溃，并同时标准化异常日志的格式。

2、在调用 `debugPrintWARNINGDefault` 方法，主要逻辑是检查 Go 版本是否达到 gin 的最低要求，再进行调试的日志 `[WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.` 的输出，用于提醒开发人员框架内部已经默认检查和集成了缺省值。

```go
func debugPrintWARNINGDefault() {
	if v, e := getMinVer(runtime.Version()); e == nil && v <= ginSupportMinGoVer {
		debugPrint(`[WARNING] Now Gin requires Go 1.13+.

`)
	}
	debugPrint(`[WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

`)
}
```

3、Engin 实例也是调用  New() 方法，所以 New() 就是进行 Engine 实例的初始化动作并返回。

## gin.New

观察一下源码，New() 返回一个新的空白 Engine 实例，没有附加任何中间件。

```go
func New() *Engine {
	debugPrintWARNINGNew()
	engine := &Engine{
		RouterGroup: RouterGroup{
			Handlers: nil,
			basePath: "/",
			root:     true,
		},
		FuncMap:                template.FuncMap{},
		RedirectTrailingSlash:  true,
		RedirectFixedPath:      false,
		HandleMethodNotAllowed: false,
		ForwardedByClientIP:    true,
		AppEngine:              defaultAppEngine,
		UseRawPath:             false,
		UnescapePathValues:     true,
		MaxMultipartMemory:     defaultMultipartMemory,
		trees:                  make(methodTrees, 0, 9),
		delims:                 render.Delims{Left: "{{", Right: "}}"},
		secureJsonPrefix:       "while(1);",
	}
	engine.RouterGroup.engine = engine
	engine.pool.New = func() interface{} {
		return engine.allocateContext()
	}
	return engine
}
```

- 同样先调用 `debugPrintWARNINGDefault` 方法，检查下版本
- RouterGroup 路由组，在内部用于配置路由器，所有的路由规则都有其管理。RouterGroup 与前缀 basepath 和处理程序（中间件）数组 Handlers 相关联

```go
type HandlersChain []HandlerFunc

type RouterGroup struct {
	Handlers HandlersChain
	basePath string
	engine   *Engine
	root     bool
}
```

- RedirectTrailingSlash：是否自动重定向，如果启用了，在无法匹配当前路由的情况下，则自动重定向到带有或不带斜杠的处理程序去
- RedirectFixedPath：是否尝试修复当前请求路径。开启时，gin 会去找一个相似的路由规则并在内部重定向过去，主要是对当前的请求路径进行格式清除（删除多余的斜杠）和不区分大小写的路由查找等。
- HandleMethodNotAllowed：判断当前路由是否允许调用其他方法。如果请求无对应路由则返回 405 响应结果
- ForwardedByClientIP：如果开启，则尽可能的返回真实的客户端 IP，先从 X-Forwarded-For 取值，如果没有再从 X-Real-Ip。
- UseRawPath：如果开启，则会使用 `url.RawPath` 来获取请求参数，不开启则还是按 url.Path 去获取。
- UnescapePathValues：是否对路径值进行转义处理。
- MaxMultipartMemory：相对应 `http.Request ParseMultipartForm` 方法，用于控制最大的文件上传大小。
- trees：多个压缩字典树（Radix Tree），每个树都对应着一种 HTTP Method。你可以理解为，每当你添加一个新路由规则时，就会往 HTTP Method 对应的那个树里新增一个 node 节点，以此形成关联关系。
- delims：用于 HTML 模板的左右定界符。

其中，如果没有设置的话，默认的配置如下所示：

```
// - RedirectTrailingSlash:  true
// - RedirectFixedPath:      false
// - HandleMethodNotAllowed: false
// - ForwardedByClientIP:    true
// - UseRawPath:             false
// - UnescapePathValues:     true
```

## r.GET

`r.GET` 方法是将定义的路由进行注册。

```go
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
	return group.handle(http.MethodGet, relativePath, handlers)
}

func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}
```

GET 方法绑定到 RouterGroup 实体上。然后调用 handle 方法进行路由路径组装，合并 Handler 后创建一个函数链 HandlersChain。最后将当前注册的路由规则（含 HTTP Method、Path、Handlers）追加到对应的树中。

### r.Run

```go
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine)
	return
}
```

Run方法是将路由器附加到 http.Server 并开始侦听和服务 HTTP 请求。它是 http.ListenAndServe(addr, router) 的快捷方式，http.ListenAndServe将 Engine 实例作为 `Handler` 注册进去，然后启动服务，开始对外提供 HTTP 服务。
注意：除非发生错误，否则此方法将无限期地阻塞调用 goroutine。

奇怪问题：ListenAndServe函数的参数是 string 和 Handler ，但是上面调用`err = http.ListenAndServe(address, engine)`时，第二个参数是 Engine 实例，却能够传入？

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

这就是 Go 语言的特性， 如果某个结构体实现了 interface 定义声明的那些方法，那么就可以认为这个结构体实现了 interface。结构体 Engine 是实现了`Handler`接口的 `ServeHTTP` 方法的，也就是符合 `http.Handler` 接口标准。



参考资料：https://golang2.eddycjy.com/posts/ch2/01-simple-server/