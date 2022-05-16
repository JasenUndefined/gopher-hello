# Protobuf

Protocol Buffer  简称 Protobuf 。 是Google出品的性能优异、跨语言、跨平台的序列化库。

## 序列化

序列化，就是将数据结构或对象的状态转换成可以存储或传输的格式。比如将数据存储为文件，或存储到内存中。传输的格式，比如网络传输。

反向操作就是反序列化。

## Protobuf 历史

- proto1 ：初代版本，仅在 Google 内部使用，且几乎所有开发者都会使用它。
- proto2 ：由于依赖关系，Google 将其开源。开发者重写了 Protobuf 的实现，保留了Proto1 的设计，开源的 Protobuf 不依赖任何库，代码更清晰。
- proto3 ：简化 proto2的开发，提高了开发的效能，但是存在和proto2不兼容的问题

## Protobuf 优势

经常在项目中使用最多的是在 HTTP 请求时会使用 JSON/XML 格式进行传输。当然还有很多其他格式，可能你接触的比较少。比如，BSON、Thrift、Avro等等。

在对比各种序列化库，包含对比序列化和反序列化的性能，以及序列化后的数据大小。总体来说，有很多很有利的优势：

- Protobuf 序列化和反序列化的性能是最高的。
- protobuf 是二进制格式，所以编码后的数据比 json/xml 还小。
- 目前 Protobuf 支持很多语言，也是支持跨平台，被广泛应用。



## 安装 Protobuf 编译器

#### 安装

- 方法一：从 github 上下载编译器：github发布地址： https://github.com/protocolbuffers/protobuf/releases

- 方法二：也可使用 `brew`

```
// 直接安装或者指定版本号 brew install protobuf@3.19.0
brew install protobuf 

// 安装完后查看版本号
 protoc --version
```

#### 编译成指定语言类库

protoc编译器支持将proto文件编译成多种语言版本的代码。生成类库的命令：

```
protoc --proto_path=IMPORT_PATH --<lang>_out=DST_DIR path/to/file.proto
```

- `--proto_path=IMPORT_PATH`：可以在 .proto 文件中 import 其他的 .proto 文件，proto_path 即用来指定其他 .proto 文件的查找目录。如果没有引入其他的 .proto 文件，该参数可以省略。
- `--<lang>_out=DST_DIR`：指定生成代码的目标文件夹，例如 –go_out=. 即生成 GO 代码在当前文件夹，另外支持 cpp/java/python/ruby/objc/csharp/php 等语言

下一篇详细说明各语言类库生成方法。

## Protonuf Demo

新建一个 demo.proto 文件

```protobuf
// 指定 proto 版本
synatx = "proto3"

// 定义数据结构
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- 第一行，指定 protobuf 的版本，如果没有指定版本将默认是 proto2 格式。
- 定义了一个 message 类型的数据结构：SearchRequest。



参考资料

- [Protobuf 终极教程](https://colobu.com/2019/10/03/protobuf-ultimate-tutorial-in-go/)

一些其他的资料：

1. [protobuf 代码仓库 - github.com](https://github.com/protocolbuffers/protobuf)
2. [golang protobuf 代码仓库 - github.com](https://github.com/golang/protobuf)
3. [Remote procedure call 远程过程调用 - wikipedia.org](https://en.wikipedia.org/wiki/Remote_procedure_call)
4. [Groupcache Go语言版 memcached - github.com](https://github.com/golang/groupcache)
5. [Language Guide (proto3) 官方指南 - google.com](https://developers.google.com/protocol-buffers/docs/proto3)
6. [Proto Style Guide 代码风格指南 - google.com](https://developers.google.com/protocol-buffers/docs/style)
7. [Protocol Buffer 插件列表 - github.com](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)

