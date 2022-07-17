# 实践从简单到使用设计模式封装 Redis

## 背景

在学习 Golang 开发时，因部分场景需引入 Redis 的使用。以下主要讲述对于循序渐进使用 Redis 过程。从简单创建缓存客户端，然后进一步优化封装自定义的缓存客户端。

## 引入 Redis

先获取 redis 第三方包 go-redis，它支持连接哨兵以及集群模式的 Redis

```shell
go get -u github.com/go-redis/redis
```

定义一个全局 Redis 客户端 RedisDB，根据 go-redis 提供例子，初始化客户端

```go
package plugins
var RedisDB *redis.Client
func InitRedis() {
     RedisDB = redis.NewClient(&redis.Options{
     Addr: "localhost:6379",
     Password: ""
     DB: 1,
 })
    pong, err := RedisDB.Ping().Result()
    if err != nil {
        fmt.Println(pong, err)
    }
    fmt.Println("redis 连接成功")
}

// main.go 中使用
package main
func main(){
 plugins.InitRedis()
 plugins.RedisDB.set("key001","Hello World", time.Second * 60)
}
```

如果这样用在一个自己的开发的小项目里没有什么问题，但如果是某个产品的服务，其还是有很多的优化空间的。

## 封装 Redis 操作

现有的工具包提供了很多的缓存方法，但我可能不需要太多，且部分的方法可能还会有其他的功能需要加入处理。所以可以基于 go-redis 封装一个仅有自己需要方法的 redis 。

```go
package cache
// 定义 MyRedis 结构体
type MyRedis struct {
    Client *redis.Client
}

var redisClient *MyRedis

// 封装 redis 实例，提供获取
func GetInstance() *MyRedis {
    return redisClient
}

// 使用单例模式进行封装
var once sync.Once

// 创建 redis 客户端，并给封装的Redis的客户端进行初始化
func NewMyRedis() *redis.Client {
    myRedis := redis.NewClient(&redis.Options{
        Addr:     "http://localhost",
        Password: "",
        DB:       1,
    })

    once.Do(func() {
        redisClient.Client = myRedis
    })

    return myRedis

}

// 自定义 Exist 方法
func (mr *MyRedis) Exist(key string) bool {
    if mr.Client.Get(key) != nil {
        return true
    }
    return false
}

// set 方法
func (mr *MyRedis) Set(key string, value interface{}, duration time.Duration) bool {
    result := mr.Client.Set(key, value, duration)
    if result.Err() != nil {
        return false
    }
    return true
}
```

1、定义一个 MyRedis 的结构体，其含有 redis 客户端 。MyRedis 就是基于 go-redis 实现自己的redis 客户端，并给该结构体绑定自有的 redis 操作方法

2、创建 NewMyRedis 函数，用于创建 redis 实例。创建的实例是用于初始化 MyRedis 的，其中使用 sync.Once 进行封装，避免出现并发情况。

3、创建一个 GetInstance 函数，用于获取自定义 MyRedis 实例的

以上步骤完成了封装自己的 Redis 操作，使用时需要先进行建立 redis 连接完成初始化 redis 的工作，然后在应用时就可以获取 MyRedis的实例，直接进行相关操作。详细的实现如下：

```go
// main.go  使用封装后的 redis
package main
func main(){
    // 项目启动时初始化 redis 
    cache.NewMyRedis()
    pong, err := rclient.Ping().Result()
    if err != nil {
        fmt.Println(pong, err)
    }
    fmt.Println("redis 连接成功")
}

// 其他 package 中使用
package user
func getUser() {
    result := cache.GetInstance().Exist("user_001")
    if !result {
        fmt.Println("不存在该数据")
    }
}
```

## 代理模式封装缓存客户端

当需求变更，希望可以支持多种缓存，比如 Redis 、memcached等等，希望可以支持多种缓存产品。可以进一步优化使用代理模式进一步增强代码的可扩展性。

![](/Users/jasenyang/Documents/pictures/pro-pic/2022-07-14-08-03-58-image.png)

代理模式：通过定义一个代理类来代理具体类的控制和访问

代理模式构成：

- 抽象主题类：通过接口或者抽象类定义具体主题对象要实现的方法

- 具体主题类：实现了抽象主题类的具体方法，是最终代理类要引用的具体类

- 代理类：提供对外的代理类，内部引用具体的类，它可以访问、控制或扩展真实主题的功能

```go
// 定义缓存接口，提供自定义的 get、set 方法
type ICache interface {
    Set(key string, value interface{}, ttl time.Duration)
    Get(key string) interface{}
}

type CacheRedis struct {
    Client *redis.Client
}

var credis *CacheRedis

func NewCacheRedis() *redis.Client {
    myRedis := redis.NewClient(&redis.Options{
        Addr:     "http://localhost",
        Password: "",
        DB:       1,
    })
    once.Do(func() {
        credis.Client = myRedis
    })

    return myRedis
}

func (credis *CacheRedis) Set(key string, value interface{}, ttl time.Duration) {
    credis.Client.Set(key, value, ttl)
}
func (credis CacheRedis) Get(key string) interface{} {
    return credis.Client.Get(key)
}

type CacheLocal struct {
    Client *redis.Client
}

func (credis *CacheLocal) Set(key string, value interface{}, ttl time.Duration) {
    credis.Client.Set(key, value, ttl)
}
func (credis CacheLocal) Get(key string) interface{} {
    return credis.Client.Get(key)
}
```

1. 定义了一个 ICache 接口 ，其中定义了缓存客户端需要进行哪些操作

2. 分别定义了2种具体的缓存类，实现了 ICache 接口。同时也需要提供初始化这2种缓存Client 的函数

3. 创建一个对外的代理类 CacheClient。其中增加了一个 InitInstance 函数 和 getDriver方法。
   
   - InitInstance 函数主要用于根据类型进行初始化缓存实例。这里进行了升级，和普通的代理模式不同，这里通过类型进行判断进行不同的缓存实例，从而实现根据不同的类型使用不同的缓存实例
   
   - getDriver 方法，通过 getDriver 实现动态代理，代理的过程由该方法来判断使用何种缓存，继而调用其真正的缓存方法。

```go
type CLIENT_TYPE string

const (
    CACHE_REDIS CLIENT_TYPE = "CACHE_REDIS"
    CACHE_LOCAL             = "CACHE_LOCAL"
)

type CacheClient struct {
    ClientType CLIENT_TYPE
    CRedis     *CacheRedis
    CLocal     *CacheLocal
}

var CacheCli *CacheClient
var ccache *CacheRedis
var lcache *CacheLocal

// 初始化缓存 cli 的函数，用于项目程序启动时使用
func InitInstance(clientType CLIENT_TYPE) {
    CacheCli.ClientType = clientType
    switch clientType {
    case CACHE_LOCAL:
        CacheCli.CRedis.Client = NewCacheRedis()
    case CACHE_REDIS:
        CacheCli.CLocal.Client = NewCacheRedis()
    default:
        break
    }
}

func (client *CacheClient) getDriver() ICache {
    if client.CLocal.Client != nil {
        return client.CLocal
    }
    return client.CRedis
}

func (credis *CacheClient) Set(key string, value interface{}, ttl time.Duration) {
    credis.getDriver().Set(key, value, ttl)
}
func (credis *CacheClient) Get(key string) interface{} {
    return credis.getDriver().Get(key)
}
```

以上是使用过程中定义的源代码，具体的在main 函数中使用如下：

在项目启动时初始化缓存 Cli，然后就可以使用结构体为 CacheClient 的代理缓存 Cli：CacheCli 进行redis 操作。

```go
func main(){
   // 根据缓存类型初始化缓存 Cli
  cache.InitInstance(cache.CACHE_REDIS)

  // 使用结构体为 CacheClient 的代理缓存 Cli：CacheCli 进行redis 操作
  data := cache.CacheCli.Get("hhh001_11") 
  fmt.Println(data)
}
```

使用代理的优势：

- 主要起到一个中介作用，对外屏蔽具体目标对象的实现。

- 在一定程度上降低了系统的耦合度，具体目标对象内部方法业务变更不影响调用者

- 增加程序的可扩展性，当需要扩展多个目标对象时可以增加目标类型，增加目标类型所对应的对象的实现，然后代理类只需要在初始化时通过不同类型初始化不同的目标对象。

## 扩展思考

思考点1：代理模式是否还可以继续优化？

应该还有需要优化的点。暂未想到，欢迎留言指点。

思考点2：当作一种设计模版，使用于其他的中间件，比如数据库，消息队列，存储服务？

代理模式这种方式适用于多个目标对象的，然后根据不同需求动态切换不同的目标对象。比如多种数据库的时候，可以通过 config 配置数据库类型，动态的初始化对应的数据库。也可以代码上控制不同的类型动态切换不同的数据源。

思考点3：其他方式实现？

暂时没有想到其他的方式封装 Redis 。如果你有很好的建议可以留言，让我学习学习。

参考资料：

- [[瞎写个 golang 脚本]03-连接 DB 和 redis - 掘金](https://juejin.cn/post/7084997454772305957)
- [设计模式之抽象工厂模式：「替换多种缓存，代理抽象场景」](https://www.sparksys.top/archives/42)
- [代理模式（代理设计模式）详解](http://c.biancheng.net/view/1359.html)

> 我正在参与掘金技术社区创作者签约计划招募活动，[点击链接报名投稿](https://juejin.cn/post/7112770927082864653 "https://juejin.cn/post/7112770927082864653")。
