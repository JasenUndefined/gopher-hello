# Go 的 RPC 简单入门

现大型项目都会采用微服务的形式，其中还会包含服务注册、治理、监控等一套完整体系的功能。在 Go SDK 中，内置 net/rpc 包可以快速实现 RPC 功能。net/rpc 包提供了通过网络访问服务端对象方法的能力。

## 基于 TCP 的 RPC

### 服务端

```go
package rpcServer

// 定义一个远程服务对象
type MathService struct {
}

// 定义参数结构体
type Args struct {
	A, B int
}

// 定义远程服务对象的一个 Add 方法，结果通过 reply 返回
func (m *MathService) Add(args Args, reply *int) error {
	*reply = args.A + args.B
	return nil
}

// 服务端 注册rpc 服务对象
package main
func main() {
    // 注册 RPC 服务对象
	rpc.RegisterName("MathService", new(rpcServer.MathService))
	l, e := net.Listen("tcp", "2222")
	if e != nil {
		log.Fatal("listen error:", e)
	}
	rpc.Accept(l)
}

```

- 定义远程服务对象 MathService ，并提供相应的方法 Add， 且方法必须返回 error 类型

- 定义一个参数结构体，作为 Add 方法的参数

- Add 方法还需提供一个返回值，通过指针变量方式返回

- 定义完服务对象后，就需要注册到服务列表中，使用 Go 提供的 net/rpc 包，使用 RegisterName 方法将远程服务对象注册进来

- RegisterName 方法接收两个参数：服务名称和具体的服务对象。

- 通过 net.Listen 函数建立一个TCP 链接，监听端口。

- 最后通过 rpc.Accept 函数在该 TCP 链接上提供 MathService 这个 RPC 服务。

要定义远程服务对象，对象的方法必须满足以下条件：

- 方法可导出，方法的类型也可导出

- 方法必须有2个参数,一个时调用者提供参数，一个是返回给调用者的，且必须是指针类型的。参数类型必须是可导出的

- 方法必须返回一个 error 类型

```go
func (t *T)MethodName(argType T1, replyType * T2) error
```

### 客户端

构建完 RPC 服务了，那么就可以在客户端中使用该服务。客户端调用远程 RPC 服务使用步骤：

- 通过 rpc.Dial 函数建立 TCP 链接

- 使用 call 方法调用远程服务的方法，Call 方法有三个参数：
  
  - 参数1：调用远程方法的名字，通常是"注册服务名称.该服务的方法“
  
  - 参数2：远程服务方法的参数1
  
  - 参数3：远程服务方法的参数2 。必须是个指针。因该参数其实就是远程服务方法的返回值，这样客户端就可以获取到客户端的返回值

```go

// 客户端调用 rpc 服务对象
package main
func main(){
    client, err := rpc.Dial("tcp", "localhost:1222")
	if err != nil {
		log.Fatal("dialing:", err)
	}
	args := rpcServer.Args{A: 10, B: 12}
	var reply int
	err = client.Call("MathService.Add", args, &reply)
	if err != nil {
		log.Fatal("MathService.Add error:", err)
	}
	fmt.Printf("MathService.Add: %d+%d=%d", args.A, args.B, reply)
}
```

## 基于 HTTP 的RPC

RPC 可以支持 TCP 协议调用，还可以通过 HTTP 协议调用。

### 服务端

基于TCP 协议的可以改成支持 HTTP 协议。定义远程服务对象还是一样，只需要将注册服务时改为支持 HTTP 协议的调用，实现如下:

```go
func main() {
	rpc.RegisterName("MathService", new(rpcServer.MathService))
	// 让 rpc 支持 http 协议
    rpc.HandleHTTP()
	l, e := net.Listen("tcp", ":1222")
	if e != nil {
		log.Fatal("listen error:", e)
	}
    // 使用 http的服务
	http.Serve(l, nil)
}
```

只需要调整2处，这样就可以支持 Http 协议。在客户端调用时，只需要创建客户端时改成以支持 HTTP 协议方式创建链接:`client, err := rpc.DialHTTP("tcp", "localhost:1222")`

所以，HTTP协议的RPC 其内核其实还是 TCP 协议。


