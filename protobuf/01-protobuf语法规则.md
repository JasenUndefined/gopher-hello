# Protobuf 语法

```protobuf
// 指定 proto 版本
synatx = "proto3";
package HelloPackage;

option go_package=".";
import "other.proto";

// 定义数据结构
message HelloRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- synatx 定义 proto 语法版本。没有指定时将默认为 proto2 版本
- package 关键字，是将此消息结构体封装在包里，同时也可以避免出现 message 类型一样的名字时产生冲突，有了包名就可以进行区分。
- import 引入其他的 proto 文件

```shell
import  "other.proto";
import public "other2.proto";
import weak "other.proto";
```

引入时支持 public 和 week 关键字。一般尽量少使用，因其在规范中介绍比较少。

​    1. public 关键字：文件中使用 public 引入其他文件时，另外文件引入你的文件时也会引入第三方文件。

​	2. week 关键字： 允许引入的文件不存在，只为了 Google 内部使用。

- option 关键字 是用于定义 proto 文件时进行标注一些列的 options 。 Options 不会改变整个文件声明的含义，但能够影响特定环境下处理方式。它也可用在 message、enum、service 的定义中。

 参考资料：

- https://www.tizi365.com/archives/371.html
- https://developers.google.com/protocol-buffers/docs/proto3

