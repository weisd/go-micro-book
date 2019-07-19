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
 
 - 