golang 连接 mongodb 

使用长连接，短连接 还是连接池？

## 长连接和短连接

首先了解下长连接和短连接原理和机制：

- 长连接：建立一个长期的连接，每次有请求时只需取出来直接使用
- 短连接：每次来一个请求时创建一个连接，使用完后请求结束则断开连接

具体的使用还是需要根据场景来分析，如果连接使用很低频的场景就不适合长连接，短连接就很合适且简单，不需要做连接的管理。对于频繁使用连接场景，短连接会频繁创建和销毁连接也会影响程序的性能，同时长连接的缺点也很明显，我们需要管理连接。

## 连接池

个人认为，无论是长连接还是短连接，使用关系型数据库，都可以通过使用连接池来提供分配，管理和释放数据库连接。它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个。

连接池可以在程序启动时创建足够的数据库连接，并将这些连接组成一个连接池。程序需要使用时可以动态的对池中的连接进行申请、使用和释放。如果程序并发请求时需要多于连接池的连接数，则需要在请求队列中排队等待。程序管理连接池时应根据连接使用率，动态增加或减少连接池中的连接数。

## 构建 mongodb 连接

### 获取包

```go
go get go.mongodb.org/mongo-driver/mongo
```

### 创建连接

```go
func initMongoDB() *mongo.Client {
	ctx := context.TODO()
	opts := options.Client().ApplyURI("your_mongodb_url")

	MongoClient, err := mongo.Connect(ctx, opts)
	if err != nil {
		println("mongodb 连接失败")
		panic(err)
	}

	err = MongoClient.Ping(ctx, nil)
	if err == nil {
		println("mongodb 连接成功")
	}
	return MongoClient
}
```

创建了 mongodbClient 后就可以进行相关的 CRUD 操作。如对执行的数据集进行操作

```go
db := MongoCli.Database("your_db_name")

collection := db.Collection("your_collection_name")
data := models.Users{"Jhonny", 18}
res, err := collection.InsertOne(context.TODO(), data)
```

处理完任务后通过命令断开连接

```go
MongoCli.Disconnect(context.Background())
```

## 构建 mongodb 连接池

连接池的作用就是为了提供性能，在池中存有已经创建好的连接，当有请求过来时直接使用已创建好的连接，这样可以省略创建连接和销毁连接的过程，从而提供性能。

如果自己设计一个连接池，那么需要考虑问题：

- 创建连接池对象，程序启动时创建指定数量的连接
- 当有请求过来时，直接从连接池中得到连接池。如果连接池对象没有空闲的连接且连接数没有达到最大，则创建一个连接，如果达到最大，则设定一定的超时时间，来获取连接
- 执行连接访问服务
- 访问服务完成后，释放连接。释放并非真正释放而是将其放在空闲队列中，当空闲连接数大于初识空闲连接数则释放连接

以下是 mongodb 连接池模式

```go
func MongoDBPool() (*mongo.Database, error) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	opts := options.Client().ApplyURI(mongoUri)
	// 设置最大连接数 -默认100， 不设置最大是64
	opts.SetMaxPoolSize(50)
	client, err := mongo.Connect(ctx, opts)
	if err != nil {
		return nil, err
	}

	return client.Database("using"), nil
}
```



