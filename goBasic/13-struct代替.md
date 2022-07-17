# Go 实现 OOP：用 Struct 代替 Class

## Go 是面向对象吗？

Go 中没有 Class 的概念，它其实不是一个纯粹的面向对象的编程语言。

Go 有类型和方法，也支持面向对象的编程风格。但是没有类型层次的结构。Go 的接口是定义一组方法的集合的类型，这些接口使用简单且通用，它们也支持嵌入到其他类型中，方便提供与子类类似但不相同的东西。

## struct 代替 class

Go 没有 class 但是有 struct ，可以给 struct 添加方法。还可以将数据和对数据的操作的方法进行绑定。这像是 Class。

```go
package modules

type Users struct {
    Name string
    Age int
    Status bool
}

func (u Users) SetUserStatus(){
    u.Status = false;
    fmt.Printf("设置 %s 的状态为 %t \n ",u.Name,u.Status)
}

// main.go
package main
func main(){
    user := modules.Users{
        Name:"Tom",
        Age:12
        Status:false
    }
    user.SetUserStatus()
}
```

以上代码定义了结构体 Users ，再绑定一个 setUserStatus 的方法，方法里就可以操作结构体的属性。在 mian 文件中我们就可以初始化一个 User 结构体的变量 user，然后操作该变量的方法。这和 Class 很相似。

## New() 函数代替构造函数

在 C#、Java 中，创建一个类时都会自带一个不带参数的构造函数。那么在 Go 中可以使用 New() 方法发挥构造函数的作用。

刚才在上面的 Demo 中我们定义并初始化了一个 Users 结构体的变量 user。当我们使用零值定义 Users 结构体时会怎么样？

```go
package main
func main(){
    var u modules.Users
    u.setUserStatus()
}


// 执行 go run main.go 结果如下：
设置  的状态为 false 
```

如你所见，使用零值创建的变量，没有有效的 name ，输出的结果中没有 name 值，只是一个空字符串。在 Java 语言中，我们经常会使用构造函数来解决这个问题，使用参数化的构造函数来创建有效的对象。比如 Java 程序中：

```java
// 在 java 语言中
public class Users{
    public string Name
    Users(name){
        this.name = name;
    }
}

// 使用
Users user = new User("Tom")
```

Go 不支持构造函数。为避免发生其他包访问类型为 Users 的结构体时定义零值的变量，产生无效的作用。通过提供一个名为 New 的函数，该函数用所需的值初始化类型 Users。 

在Go 中，将创建 T 类型值的函数命名为 NewT(params) 是一种约定。这将充当构造函数。当包里只有一种类型，则该约定就是将函数命名为 New(params), 而不是 NewT(params)。

所以我们正确使用方式如下：

```go
package modules

type users struct {
    name string
    age int
    status bool
}

func New(name string, age int, status bool) Users {
    user := users{name, age, status}
    return user
}

func (u Users) setUserStatus(){
    u.status = false;
    fmt.Printf("设置 %s 的状态为 %t \n ",u.name,u.status)
}
```

从上面的代码上可以看到：

- 我们将结构体 Users 改为私有的，它的字段也都设置为私有的，防止外部其他的包访问。因为我们不需要除了 modules 包外的其他地方访问 users 的字段，除非有其他特定场景需要。

- 另外，增加了一个公有的 New 函数，该函数的参数就是 user 结构体的字段，并且返回一个新创建的 user 实例。

使用只需调用 New 函数，这样就能防止创建不可用的 user struct 类型值。这也是创建 user 的唯一方法。

```go
// 使用 
package main
func main(){
 user := modules.New("Jack", 30, true)
 user.setUserStatus()
}
```

因此，Go 虽然不支持 class ，但我们可以有效的使用 struct 来代替 class ，并使用 New 函数来代替构造函数。这样就可以实现 OOP。

参考资料：

- https://juejin.cn/post/7107265773306904613
- https://golangbot.com/structs-instead-of-classes/
- https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/

> 本文正在参加[技术专题18期-聊聊Go语言框架](https://juejin.cn/post/7117898969866305566 "https://juejin.cn/post/7117898969866305566")
