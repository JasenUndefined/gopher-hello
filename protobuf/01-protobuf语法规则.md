# Protobuf 语法

## 定义消息

```protobuf
syntax = "proto3" // 指定版本号
package HelloPackage; // 定义包

option go_package=".";// 定义结构体可选项
import "other.proto";  // 引入其他结构体

message 消息名{
	// 消息体
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

## 数据类型

| proto类型 | go类型  | 备注                          | proto类型 | go类型  | 备注                       |
| :-------- | :------ | :---------------------------- | :-------- | :------ | :------------------------- |
| double    | float64 |                               | float     | float32 |                            |
| int32     | int32   |                               | int64     | int64   |                            |
| uint32    | uint32  |                               | uint64    | uint64  |                            |
| sint32    | int32   | 适合负数                      | sint64    | int64   | 适合负数                   |
| fixed32   | uint32  | 固长编码，适合大于2^28的值    | fixed64   | uint64  | 固长编码，适合大于2^56的值 |
| sfixed32  | int32   | 固长编码                      | sfixed64  | int64   | 固长编码                   |
| bool      | bool    |                               | string    | string  | UTF8 编码，长度不超过 2^32 |
| bytes     | []byte  | 任意字节序列，长度不超过 2^32 |           |         |                            |

标量类型如果没有被赋值，则不会被序列化，解析时，会赋予默认值

- strings：空字符串
- bytes：空序列
- bools：false
- 数值类型：0

## 枚举

当 message 结构中的字段需要一组预定义的值时，就可以使用枚举

```protobuf
syntax = "proto3";

enum GenderType //枚举消息类型，使用enum关键词定义,一个电话类型的枚举类型
{
    femail = 0; //proto3版本中，首成员必须为0，成员不应有相同的值
    mail = 1;
}

message Student {
	string name;
	GenderType gender;
}
```

- 枚举类型的第一个选项的标识符必须时0

当枚举中出现相同值的标识符时，可以通过开启允许别名 allow_alias

```protobuf
enum status //枚举消息类型，使用enum关键词定义,一个电话类型的枚举类型
{
    option allow_alias = true; // 允许为不同的枚举值赋予相同的标识符
    unknow = 0; //proto3版本中，首成员必须为0，成员不应有相同的值
    start = 1;
    running = 
}
```

## 数组类型

数组类型，通过在字段前加 repeated 关键字，标记当前字段是一个数组。

```
message Msg {
  // 只要使用repeated标记类型定义，就表示数组类型。
  repeated int32 arrays = 1;
}
```

## Map 类型

语法：map<key_type, value_type> map_field = N;

```protobuf
syntax = "proto3";
message Product
{
    string name = 1; // 商品名
    // 定义一个k/v类型，key是string类型，value也是string类型
    map<string, string> attrs = 2; // 商品属性，键值对
}
```



Proto3 代码风格，可参考：https://developers.google.com/protocol-buffers/docs/style

参考资料：

- https://www.tizi365.com/archives/371.html
- https://developers.google.com/protocol-buffers/docs/proto3

