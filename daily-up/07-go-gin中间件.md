go gin 中间件



中间件，在正式处理 http 请求之前，进行一层逻辑控制，类似于拦截器。请求前拦截，比如可以进行登录时效验证，或日志记录，或捕获错误等等。但不像 Java 语言，通过注解的方式写在控制器或者方法的之上。在 Go 中是将其用在路由之前的。

在gin 中，直接使用 `gin.Default()` 初始化 gin 对象，其中它包含了一个自带默认中间件的 `*Engine`。观察以下代码，你会发现 Default 方法中默认绑定了2个中间件，Logger 和 Recovery，主要用于打印日志输出和 panic 处理

```go
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```

在 Go gin中的中间件使用方式如下，通过使用 Use 方法设置的，它接收一个可变的参数：

```go
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes
```

所以，中间件必须是能够返回 HandlerFunc 类型的函数。

```go
// HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)
```

了解中间件的定义，那可以自定义中间件。只要我们自己实现一个`HandlerFunc`，就可以自定义一个自己的中间件。

```go
func CustomMiddleware() gin.HandlerFunc {
	return func(ctx *gin.Context) {
		ctx.String(200, "success")
	}
}

// 使用
r.use(CustomMiddleware)
```

根据中间件注册位置划分，可以分为

- 全局中间件：所有的请求都经过中间件
- 局部中间件：只针对部分注册的路由生效。如果将中间件在某路由下注册，则只针对该路径下有效。比如再耨个群组路由下，或者说在某单个路由下。

Next() 方法

Next() 只允许在中间件函数中使用，用于挂起请求业务逻辑处理。即**当前中间件中调用`c.Next()`时会中断当前中间件中后续的逻辑，转而执行后续的中间件和handlers，等他们全部执行完以后再回来执行当前中间件的后续代码。**

````go
func CustomMiddleware() gin.HandlerFunc {
	return func(ctx *gin.Context) {
		println("hello before")
		ctx.Next()
		println("hello after")
		ctx.String(200, "success")
	}
}
````

