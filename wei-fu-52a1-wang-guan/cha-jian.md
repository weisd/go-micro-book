# 网关插件

网关的插件，其实就是http server的middleware

插件的执行过程，在web服务启动之前

官方支持的插件有：
    - auth
    - cors
    - gzip
    - ip_whitelist
    - metrics
    - trace
    
更多插件，看https://github.com/micro/go-plugins/tree/master/micro

## 使用

使用插件，只要在init方法中，注册需要的插件就可以了

示例

```go
package main

import (
	"log"
	"github.com/micro/cli"
	"github.com/micro/micro/plugin"
)

func init() {
	plugin.Register(plugin.NewPlugin(
		plugin.WithName("example"),
		plugin.WithFlag(cli.StringFlag{
			Name:   "example_flag",
			Usage:  "This is an example plugin flag",
			EnvVar: "EXAMPLE_FLAG",
			Value: "avalue",
		}),
		plugin.WithInit(func(ctx *cli.Context) error {
			log.Println("Got value for example_flag", ctx.String("example_flag"))
			return nil
		}),
	))
}
```

## 实现一个插件很简单

先看一下插件定义

```go
// Plugin is the interface for plugins to micro. It differs from go-micro in that it's for
// the micro API, Web, Sidecar, CLI. It's a method of building middleware for the HTTP side.
type Plugin interface {
    // Global Flags
    Flags() []cli.Flag
    // Sub-commands
    Commands() []cli.Command
    // Handle is the middleware handler for HTTP requests. We pass in
    // the existing handler so it can be wrapped to create a call chain.
    Handler() Handler
    // Init called when command line args are parsed.
    // The initialised cli.Context is passed in.
    Init(*cli.Context) error
    // Name of the plugin
    String() string
}

// Manager is the plugin manager which stores plugins and allows them to be retrieved.
// This is used by all the components of micro.
type Manager interface {
        Plugins() map[string]Plugin
        Register(name string, plugin Plugin) error
}

// Handler is the plugin middleware handler which wraps an existing http.Handler passed in.
// Its the responsibility of the Handler to call the next http.Handler in the chain.
type Handler func(http.Handler) http.Handler
```