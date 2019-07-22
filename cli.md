# cli
cli 是一个命令行工具
它可以做：
 - 解析参数
 - 添加命令

```

// App is the main structure of a cli application. It is recommended that
// an app be created with the cli.NewApp() function
type App struct {
	// 程序的名称，默认取path.Base(os.Args[0]) 执行路径的文件名
	Name string
	// 帮助中显示的全称，默认同Name
	HelpName string
	// 程序的说明
	Usage string
	// 覆盖Usage的帮助文字
	UsageText string
	// 参数格式说明.
	ArgsUsage string
	// 程序的版本号
	Version string
	// 程序的说明
	Description string
	// 需要执行的命令列表
	Commands []Command
	// 需要解析的参数列表
	Flags []Flag
	// 是否允许 bash completion commands
	EnableBashCompletion bool
	// 是否隐藏默认帮助说明
	HideHelp bool
	// 是否隐藏version参数
	HideVersion bool
	// 分类，只能通过Categories()访问
	categories CommandCategories
	// bash-completion 参数执行的函数
	BashComplete BashCompleteFunc
	// 任何子命令执行前的执行操作
	// 如果执行返回error,程序中断
	Before BeforeFunc
	// 程序执行结束后执行的操作，程序panic也会执行after
	After AfterFunc

	// 没有子命令时执行的操作
	Action interface{}

	// 命令不存在时执行
	CommandNotFound CommandNotFoundFunc
	// 使用出错时执行
	OnUsageError OnUsageErrorFunc
	// 编译日期
	Compiled time.Time
	// 作者名单
	Authors []Author
	// 版权信息
	Copyright string
	// 作者名称（废弃，请使用Authors）
	Author string
	// 作者邮箱（废弃，请使用Authors）
	Email string
	// 程序执行输出
	Writer io.Writer
	// 错误输出
	ErrWriter io.Writer
	// 自定义信息
	Metadata map[string]interface{}
	// 返回程序特定信息.
	ExtraInfo func() map[string]string
	// 自定义帮助模板
	CustomAppHelpTemplate string

}
```


执行过程

- 解析os.Args,环境变量, 得到执行程序名称，子命令，参数，环境变量优先级大于flag
- 解析Flags定义的参数，如果多出未定义的参数会提示错误，得到一个flagSet
- 如果Before定义，执行Before
- 如果子命令 递归执行子命令
- 如果有action执行action,没有执行显示帮助内容
- 如果After定义，退出前执行After
 