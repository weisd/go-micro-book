# 客户端处理流程

### 实例化client

一般go-micro项目都会启动一个service, service中都包含一个client, 所以，如果是go-micro框架的项目，直接调从service 中就可以得到client, 然后new对应要调用的client就行了

例子：

```
s := grpc.NewService()
s.Init()

// user import proto生成的代码
userClient := user.NewService("", s.Client())

// 调方法，传ctx,request，返回reponse
response, err := userClient.Hello(context.Background(), &hello.Request{
	Name: "world",
})

```

### client调用流程

 go-micro会生成对应方法的代码，下面解析一下调用过程
 
 - 调用前已知service名，方法名
 - 实例化client.Request, response
 - 调用client.Call方法
 - 从 Registry中取service名对应的服务器节点列表, 请求后会有个cache把结果缓存、并watch变化自动更新
 - 调用Selector.Select方法选择需要连接的服务器节点，默认使用Random策略
 - 如果未找到对应serive的节点，返回not found error
 - 生成一个带timeout的context
 - 包装wrapper准备调用
 - 包装retry设定重试次数
 - 调用call【见下一流程】，之前先调用Backoff，允许客户端延时请求
 - 调用结束记录select节点成功，Selector.Mark，这一步一般也不做处理
 
### call流程
 - 生成msg.Header
 - 通过content-type取到对应codec
 - 从连接池中取一个transport, defer release
 - 用transport实例化codec
 - 实例化stream, reponse
 - 统计请求计数+1
 - stream.Send() 发送reqeust, 其实就是调用codec.Write
 - stram.Recv() 接收reponse, 其实就是调用codec.ReadHeader, codec.ReadBody
 - Recv结果会映射到对应message
 - done