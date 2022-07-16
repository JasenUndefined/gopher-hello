

以下是摘抄自一个对 Go 是不是面向对象编程语言的回答：

> Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are more general than in C++ or Java: they can be defined for any sort of data, even built-in types such as plain, “unboxed” integers. They are not restricted to structs (classes). 

Go 有类型和方法，也支持面向对象的编程风格。但是没有类型层次的结构。Go 的接口是定义一组方法的集合，这些方法集合仅仅只是被定义，它们没有在接口中实现。这些接口使用简单且通用，它们也支持嵌入到其他类型中，方便提供与子类类似但不相同的东西。







参考资料：

- https://golangbot.com/structs-instead-of-classes/


