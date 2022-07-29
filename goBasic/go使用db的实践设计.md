golang 使用 MongoDB

本文将主要实践：使用 Golang 语言连接到 MongoDB 服务器,执行基本的CRUD操作。

基本步骤：

- 安装 MongoDB 和重要 Golang 包
- 连接 MongoDB 服务，ping 服务，列出存在的数据库
- 通过 Demo 使用各种函数执行 CRUD 操作

## 安装 MongoDB 和 Golang 包

给你的平台安装 MongoDB 其实很简单。你可以从[这里](https://www.mongodb.com/try/download/shell)获取到各种平台各种版本。选择适合的环境版本进行安装。

## 安装 MongoDB

### 安装 MongoDB 的 Golang 包

为了使用 Golang 语言操作 MongoDB，你需要在你的终端执行以下命令：

```
go mod init
go get go.mongodb.org/mongo-driver/mongo
go get go.mongodb.org/mongo-driver/bson
```

### BSON 序列化数据

在以上的命令行中，你可以看到我们安装 BSON 包。BSON是一种序列化的格式，它类似于JSON。

来自 BSON 的文献说明：

> *BSON [bee · sahn], short for Bin­ary JSON, is a bin­ary-en­coded seri­al­iz­a­tion of JSON-like doc­u­ments. Like JSON, BSON sup­ports the em­bed­ding of doc­u­ments and ar­rays with­in oth­er doc­u­ments and ar­rays. BSON also con­tains ex­ten­sions that al­low rep­res­ent­a­tion of data types that are not part of the JSON spec.*

BSON 就是 Binary JSON ，是类 JSON 文档的二进制编码序列化。 与 JSON 一样，BSON 支持将文档和数组嵌入到其他文档和数组中。 BSON 还包含允许表示不属于 JSON 规范的数据类型的扩展。

我们将在 MongoDB 中使用它，用于保存文档数据以及在查询中处理文档时创建过滤器等。更多想要了解 BSON可以访问[这里](https://bsonspec.org./)。

### 连接MongoDB

接下来使用 Golang 创建连接 MongoDB。首先了解下，标准连接字符串格式：

```
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```

- 可以创建不同形式的 Context 。先创建一个最简单的 Context ，即 `context.TODO()`。

```golang
func InitMongoDB(mongoURL) {
    ctx := context.TODO()
    opts := options.Client().ApplyURI(mongoURL)

    client, err := mongo.Connect(ctx, opts)
    if err != nil {
        println("mongodb 连接失败")
        panic(err)
    }

    defer client.Disconnect(ctx)
    println("mongodb 连接成功")
}
```

有时可能出现端口不正确

参考资料：

- https://medium.com/better-programming/how-to-use-golang-with-mongodb-26f043d31d23
- https://www.mongodb.com/docs/manual/reference/connection-string/