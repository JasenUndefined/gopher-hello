实践｜Go 集成Kafka 完成自动消息转发到飞书

## 前言

因在学习 Kafka 时，为了更有效的对 Kafka 熟悉，通过使用以下小功能完成对 Kafka 实践使用和认识。在之前的文章中 [[实践篇]使用 Docker 安装 Kafka - 掘金](https://juejin.cn/post/7109650169271943181) 分享了在本地电脑上安装 Kafka 。接下来就是进一步实践集成到项目中。

## 需求设计

- 创建 Golang 项目

- 集成 Kfaka

- 生产者接口进行生产消息

- 消费者读取消息并使用飞书的 webhook 功能，将消息转发到飞书中

## 详细设计

### 生产者

### 消费者

### 飞书webhook

创建发送飞书 webhook 工具文件 feishu_util ，主要的工作就是将获取到的消息转发到飞书中。目前实现比较简单，后续可以拓展支持多个文本。

```go
func SendFeishu(text string) interface{} {
    url := ""
    data := fmt.Sprintf(`{"msg_type":"text","content":{"text": "%s"}}`, text)
    return httpPost(url, data)
}

func httpPost(url string, data string) interface{} {
    method := "POST"
    payload := strings.NewReader(data)

    client := &http.Client{}
    req, err := http.NewRequest(method, url, payload)

    if err != nil {
        fmt.Println(err)
        return nil
    }
    req.Header.Add("Content-Type", "application/json")

    res, err := client.Do(req)
    if err != nil {
        fmt.Println(err)
        return nil
    }
    defer res.Body.Close()

    body, err := ioutil.ReadAll(res.Body)
    if err != nil {
        fmt.Println("请求失败")
        return nil
    }
    fmt.Println(string(body))
    return body
}
```

其中发送 webhook 请求，主要使用 Go 的 net/http 库发送 POST 请求。实现飞书最基本的消息发送，body 参数中 msg_type = text ,其还支持其他的类型：text、post、image、file、audio、media、sticker、interactive、share_chat、share_user等，消息内容 content，json结构序列化后的字符串。根据不同的消息类型对应不同的内容。

参考资料：

- [[实践篇]使用 Docker 安装 Kafka - 掘金](https://juejin.cn/post/7109650169271943181)
