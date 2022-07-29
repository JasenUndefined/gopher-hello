# go-zero 初步使用

go-zero 是一个集成啦各种工程实践的 web 和 rpc 框架。对于现在微服务盛行的风气下，不得不学习下 go-zero。其设计理念是对于微服务框架设计，在保障微服务稳定性的同时也特别注重研发效率。所以可以看到其有很强大的功能，且能够支持很多功能。

更多详细的项目内容可以访问项目库: https://gitee.com/kevwan/go-zero

## 环境搭建

首先需要安装工具 goctl 。它是go-zero 的生成工具 goctl，可以根据定义的 api 文件一键生成 Go, iOS, Android, Kotlin, Dart, TypeScript, JavaScript 代码，并可直接运行。

```bash
# Go 1.15 及之前版本
GO111MODULE=on GOPROXY=https://goproxy.cn/,direct go get -u github.com/zeromicro/go-zero/tools/goctl@latest

# Go 1.16 及以后版本
GOPROXY=https://goproxy.cn/,direct go install github.com/zeromicro/go-zero/tools/goctl@latest
```

安装 goctl 工具

```bash
brew install goctl
```

通过 goctl 快速生成 api 服务

```
goctl api new greet
cd greet
go mod init 
go mod tidy
go run greet.go -f etc/greet-api.yaml
```

就这样，一个简单的 api 服务初始化完成，然后就可以启动服务，打开配置文件可以看到配置文件中默认端口号是888

```bash
go run greet.go -f etc/greet-api.yaml
```

## 源码分析

### server 服务

通过 server := rest.MustNewServer(c.RestConf) 获取一个 server 实例。实例创建的最底层server 是由 server.ngin 和 server.router 组成。而 ngin 是一个 engine 实例。

```go
type engine struct {
    conf                RestConf                        // 配置信息
    routes               []featuredRoutes                // 初始路由组信息
    unauthorizedCallback handler.UnauthorizedCallback    // 认证
    unsignedCallback     handler.UnsignedCallback        // 签名
    middlewares          []Middleware                    // 中间件
    shedder              load.Shedder
    priorityShedder      load.Shedder
    tlsConfig            *tls.Config
}
```

### Router 路由

路由是通过实现 httpx.Router 接口的实例。实现了 ServeHttp 方法。再通过 Handle 方法进行注册请求路由。进入到 Handle 方法具体实现，可以看到是将 server.ngin.routes 映射到路由树上

```go
func NewRouter() httpx.Router {
    return &patRouter{
        trees: make(map[string]*search.Tree),
    }
}

type Router interface {
    http.Handler
    Handle(method, path string, handler http.Handler) error
    SetNotFoundHandler(handler http.Handler)
    SetNotAllowedHandler(handler http.Handler)
}
```

## 启动服务

```go
    server.Start()

func (s *Server) Start() {
    handleError(s.ngin.start(s.router))
}

func (ng *engine) start(router httpx.Router) error {
    if err := ng.bindRoutes(router); err != nil {
        return err
    }

    if len(ng.conf.CertFile) == 0 && len(ng.conf.KeyFile) == 0 {
        return internal.StartHttp(ng.conf.Host, ng.conf.Port, router, ng.withTimeout())
    }

    return internal.StartHttps(ng.conf.Host, ng.conf.Port, ng.conf.CertFile,
        ng.conf.KeyFile, router, func(svr *http.Server) {
            if ng.tlsConfig != nil {
                svr.TLSConfig = ng.tlsConfig
            }
        }, ng.withTimeout())
}
```

server 的启动主要是用调用结构体engine 的 start 方法，参数是一个router， 然后使用绑定，将路由进行绑定到 ngin 上 。这个接口也是 http.handler 类型处理器。最后 internal.StartHttps 调用了 http.Server 启动服务。

参考资料：

- https://learnku.com/articles/66596

- https://juejin.cn/post/7087973893763235871

> 本文正在参加[技术专题18期-聊聊Go语言框架](https://juejin.cn/post/7117898969866305566 "https://juejin.cn/post/7117898969866305566")
