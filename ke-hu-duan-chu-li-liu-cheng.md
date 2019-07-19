# 客户端处理流程

### 实例化client

一般go-micro项目都会启动一个service, service中都包含一个client, 所以，如果是go-micro框架的项目，直接调从service 中就可以得到client


```
s := grpc.NewService()

s.Client()

```
