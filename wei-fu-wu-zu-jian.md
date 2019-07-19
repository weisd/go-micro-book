# 微服务组件

* 网关
  - api
  - http
  - rpc
  - proxy
* RPC
  - 服务发现
  - 负载均衡
  - 消息序列化
  - 通信传输协议
  - 消息发布订阅
- 配置中心
  - config
  
* 服务治理
  * metrics 统计
  * logger  日志
  * tracing 跟踪
  * breaker 熔断
  * ratelimiter 限流

### service种类

service被分为收下种类
 - api 提供json api服务，返回json
 - web 提供正常http response服务，像一些页面渲染，文件下载之类
 - event  提供broker pulish
 - srv 提供rpc调用



关系图如下：

![](/assets/service_types.png)



第一层 micro api/web 网关层，对外开放80、443端口

第二层 web/api service 业务逻辑层

第三层 srv 数据访问层
