# go-micro 框架结构


### 框架组件
整个rpc框架由以下部分组成

- Cmd  
    解析命令行参数、环境变量，定义启动命令
- Registry
    服务注册发现，支持mdns、consul、etcd、memory、gossip、kubernetes、nats、zookeeper，默认使用mdns
- Selector
    服务负载均衡策略，支持Random、RoundRobin，默认使用Random
- Transport
    服务通信传输方式,支持http、tcp、udp、grpc、nats、rabbitmq，默认使用http
- Codec
    数据的传输格式，支持text、json、jsonrpc、protorpc、proto、grpc、bytes，会权限header的content-type自动选择对应的方式，默认使用proto
- Client
    rpc客户端
- Server
    rpc服务器端
- Broker
    消费订阅组件，支持http、nats、grpc、kafka、nsq、redis、rabbitmq等，默认使用http
    
    
### 大致流程
    
#### rpc服务端(service)启动流程
    
    - cmd解析命令行、环境变量参数，确定使用的client、server、registry、transport、broker,以及一些自定义参数
    - 把要提供的rpc方法（handlers）和 消息处理方法（subscribers） 注册到service，生成方法路由表（service.Router）
    - 申请端口号，确定节点ip
    - 注册到Registry，把注册在service中的handlers、subscribers 转成 注册信息（registry.Endpoint）随service所有的节点信息（registry.Node）以及 service的名称、版本组成registry.Service存入Registry中
    - 连接Broker,用于接口推送消息
    - 开放端口，监听客户端连接
    - 异步定时任务更新registry.Service到Registry，防止注册信息过期
    
#### rpc服务端(service)关闭流程
    
    - 接收到退出信号，删除注册到Registry的信息
    - 是判断是否配置等待连接请求结束，如果有则等待
    - 断开Broker连接
    - 发信号，退出异步定时任务
    
#### rpc服务端(service)处理客户端请求
    - transport.Socket.Recv 接收 transport.Message
    - 如果配置waitGroup（wg）, wg计数+1
    - 记录header 到metadata
    - 记录local/remote ip 到metadata
    - 生成context,附带metadata
    - 如果配置了 timeout 参数 ，生成timeout的context
    - 根据content-type取到codec, 如果content-type未设置，默认application/protobuf
    - 如果取不到对应content-type的codec, 返回错误
    - 创建codec,创建request,创建response实例
    - 调用路由表方法Router.ServeRequest(ctx, request, response)，执行对应方法
    - 如果配置wg, wg.Done()
    - 调用结束
    
#### rpc服务端(service)处理请求内容ServeRequest过程
    - 使用对应codec读取msg.Header
    - 从header中取到请求的服务名和方法名，并从方法路由表中取出对应的服务方法
    - 使用对应codec读取msg.Body
    - 使用反射生成对应的方法参数，返回值
    - 执行服务方法Call，用的反射function.Call, call的时候如果配置有wrapper会一层层执行对应wrapper
    - 执行结果生成Message,通过codec都结果序列化，再由 transport.Socket.Send传回客户端
    
    
    
    