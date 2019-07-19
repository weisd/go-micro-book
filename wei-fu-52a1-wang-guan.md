# 网关

micro 的网关 由micro api 命令创建，支持以下处理方式

* api http请求访问对应api service 
* rpc http请求访问对应 srv service
* proxy http代码，就是
* event http转换成有pulish event
* web  也是http代码，支持websocket
* /rpc http直接转换成rpc请求

micro的网关本质就是个web-server,使用github.com/gorilla/mux路由，各方式的区别在于转换的目标request,reponse不同，下面简单列出各个方式的区别

### API Handler

* Content-type: any
* Body: any
* 转换格式：api.Request/api.Response
* Path: /\[service\]/\[method\]
* 解析规则: path
* 配置: --handler=api 或 环境变量  MICRO\_API\_HANDLER=api

### RPC Handler

* Content-Type: application/json or application/protobuf
* Body: json/protobuf
* 转换格式: body解析json/protobuf出来的内容作为request
* Path: /\[service\]/\[method\]
* 解析规则: path
* 配置: --handler=rpc 或 环境变量 MICRO\_API\_HANDLER=rpc

### Proxy Handler

* Content-Type: any
* Body: any
* 转换格式: http反向代理
* Path: /\[service\]
* 解析规则: path
* 配置: --handler=proxy 或 环境变量 MICRO\_API\_HANDLER=proxy

### Event Handler

* Content-Type: any
* Body: any
* 转换格式: proto.Event
* Path: /\[topic\]/\[event\]
* 解析规则: path
* 配置: --handler=event 或 环境变量 MICRO\_API\_HANDLER=event

### Web Handler

* Content-Type: any
* Body: any
* 转换格式: http反射代码
* Path: /\[service\]
* 解析规则: path
* 配置: --handler=web 或 环境变量 MICRO\_API\_HANDLER=web

### RPC endpont

* Content-Type: application/json or application/x-www-form-urlencoded
* Body: json/form
* Path: /rpc
* 转换格式: request请求所需参数，service、method、request
* 解析规则: body

  实例：

  ```
  curl -d 'service=go.micro.srv.greeter' \
     -d 'method=Say.Hello' \
     -d 'request={"name": "Bob"}' \
     http://localhost:8080/rpc
  ```

## 解析规则

解析规则使用网关的**namespace**和http的path来确定对应service、method

规则如下

RPC  
 一个service名为 go.micro.api.greeter 方法名为 Greeter.Hello 解析规则如下

| Path | Service | Method |
| :--- | :--- | :--- |
| /foo/bar | go.micro.api.foo | Foo.Bar |
| /foo/bar/baz | go.micro.api.foo | Bar.Baz |
| /foo/v1/bar | go.micro.api.foo | V1.Bar |
| /foo/v1/bar-baz | go.micro.api.foo | V1.BarBaz |
| /foo/bar/baz/cat | go.micro.api.foo.bar | Baz.Cat |

带有version的api URL

| Path | Service | Method |
| :--- | :--- | :--- |
| /foo/bar | go.micro.api.foo | Foo.Bar |
| /v1/foo/bar | go.micro.api.v1.foo | Foo.Bar |
| /v1/foo/bar/baz | go.micro.api.v1.foo | Bar.Baz |
| /v2/foo/bar | go.micro.api.v2.foo | Foo.Bar |
| /v2/foo/bar/baz | go.micro.api.v2.foo | Bar.Baz |

Proxy 代码解析规则

反向代码，只解析出service名

| Path | Service | Service Path |
| :--- | :--- | :--- |
| /foo | go.micro.api.foo | /foo |
| /foo/bar | go.micro.api.foo | /foo/bar |
| /greeter | go.micro.api.greeter | /greeter |
| /greeter/:name | go.micro.api.greeter | /greeter/:name |



