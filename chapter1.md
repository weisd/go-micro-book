# go-micro 框架结构


框架组件

- Cmd  
    解析命令行参数、环境变量，定义启动命令
- Registry
    服务注册发现
- Selector
    服务负载均衡策略，目录有Random、RoundRobin，默认使用Random
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
    
    