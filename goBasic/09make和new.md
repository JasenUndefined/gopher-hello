# 内存分配：new 和 make

> 持续创作，加速成长！这是我参与「掘金日新计划 · 6 月更文挑战」的第12天，[点击查看活动详情](https://juejin.cn/post/7099702781094674468)

程序的运行都需要内存，在变量的创建，函数调用，数据计算等都需要内存。在 Go 中由语言自己管理，开发时只需声明变量，Go 语言就会根据变量的类型自动分配相应的内存。变量的声明和初始化时涉及内存的分配，除了使用 var 关键字声明，还会使用内置函数 new 和 make 。

## make 关键字

make 作用：初始化内置的数据结构，如 slice 、 哈希表 、 channel 

```go
slice := make([]int, 0, 100)
hash := make(map[int]string, 10)
ch := make(chan int, 5)
```

## new 关键字

new 作用：根据传入的类型分配一定内存空间并返回指向该内存空间的指针

```go
i = new(int)
*i = 1
sp =new(string) 
*sp= "tom"
```

通过内置的 new 函数 生成一个 *string ，并赋值给变量 sp。

内置函数 new 的作用就是根据传入的类型申请一块内存，然后返回指向这块内存的指针，指针指向的数据就是该类型的零值。通过 new 函数分配内存并返回指向该内存的指针后，就可以通过该指针对这块内存进行赋值、取值等操作。

```go
sp1 = new(string)
fmt.Println(*sp1)//打印空字符串,也就是string的零值。
```

复合类型也支持使用 new 关键字进行声明和初始化。

```go
type person struct {
   name string
   age int
}

func NewPerson（name string, age int） *person {
  p := new(person)
  p.name = name;
  p.age = age
  
  return p
}

// 使用
pp := NewPerson()
println(pp.name)
```

通过使用 Newperson 函数进行一层包装，根据不同的参数，可以得到不同 *person 变量。NewPerson 函数其实就是个工厂函数。

所以，new 函数只用于分配内存，并把内存清零，也就是返回一个指向该类型的零值指针。new 函数一般用于需要显式返回指针的情况，不常使用；make 函数只用于 slice 、map、chan 这三种内置类型的创建和初始化。