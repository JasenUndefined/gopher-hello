Go 中正确使用依赖注入

## 依赖注入

依赖注入可以降低代码的耦合度。某功能依赖另一个实现，但是与该实现无任何责任。它只接收该实现并使用该实现，可以通过依赖注入方式进行解耦。比如以下例子，UserService 依赖很多模块。

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-08-08-15-13-53-image.png)

Service 依赖很多模块，我们可以将这部分内容都设置到 UserService 中，但这样耦合太高了。

```go
type UserService struct{}
type logger struct{}
type RoleService struct{}
type UserProvider struct{}

func (uServ *UserService)GetUser()error{
    loger := &logger{}
    provider := &UserProvider{}
    ....
    return nil
}
```

在 SOLID 原则中的 D，依赖倒置原则。说的是，**程序要依赖于抽象接口，不要依赖于具体实现**。注入依赖的一个重点时要避免注入具体的实现，即应该避免注入像 logger 、UserProvider 等结构。

## 依赖注入方式

### 构造函数注入

构造函数方式注入，在创建实例时就注入依赖项，而后就不能再改变。所以在使用时所有的依赖项都必须创建好，否则会产生错误。在 Go 中没有构造函数的定义，所以可以使用 New 函数充当构造函数。

```go
func NewUserService(log Logger, userProvider UserProvider) UserService {
    return &UserService{
        logger: log,
        userProvider: userProvider
    }
}
```

### 属性和方法注入

属性注入和方法注入非常相似。在 Java 中比较常见的是方法注入，而在C#中比较常见是属性注入。在 Go 中也支持这两种方式的注入，且大部分时候可以结合起来。

```go
type UserService struct{
    logger MyLogger
    provider UserProvider
}
func (s *UserService)GetUser(log MyLogger){
    s.logger = log
}
```

定义 UserService 结构体中也依赖了 MyLogger 和 UserProvider 的结构体，方法GetUser 参数是 MyLogger，这就是方法注入。按照这样设计时，这种注入是对于具体实现允许可变的，MyLogger 可能会有不同的实现，调用 GetUser 时只需传入对应 MyLogger 类型即可，且不关心是何种具体的实现。

## 使用第三方包

使用手动的方式构造函数注入方式的话，如果有很多的依赖项，那么就会看到传入的参数就会很多,就要手动去创建所有的依赖，无疑是种增加维护成本的操作。

```go
type UserService struct{}

func NewUserService(logger MyLogger,repositroy MyRepositroy,provider UserProvider,...){
    // ...
}

// 使用
func main(){
    // 所有的依赖项都声明，创建
    logger ：= &MyLogger{}
    repositroy ：= &MyRepositroy{}
    provider ：= &UserProvider{}
    ...
    // 所有的依赖项都注入
  userService := NewUserService(logger,repositroy,provider...)

}
```

像 Java 中通过一个能够发现依赖并创建依赖关系的容器。只要通过它就可以自动识别关系并创建处其所依赖的对象。

所以我们可以构造一个这样的容器，再通过反射，读取函数的接收和返回每个参数的类型，就可以构建其所以依赖其他实例，并创建出依赖关系图。这里可以推荐使用第三方 uber-go/dig 包（https://github.com/uber-go/dig）。

参考资料：

- https://medium.com/avenue-tech/dependency-injection-in-go-35293ef7b6
