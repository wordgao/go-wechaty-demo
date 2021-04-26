# go-wechaty-setting-demo
go-wechaty go wechaty

//go mod download下载模块到本地缓存，缓存路径是$GOPATH/pkg/mod/cache

//go mod edit是提供了命令版编辑go.mod的功能，例如go mod edit -fmt go.mod会格式化go.mod

//go mod graph把模块之间的依赖图显示出来

//go mod init初始化模块（例如把原本dep管理的依赖关系转换过来）

//go mod tidy增加缺失的包，移除没用的包

//go mod vendor把依赖拷贝到vendor/目录下

//go mod verify确认依赖关系

//go mod why解释为什么需要包和模块

// go get -u -v github.com/xxx/xxx 手动显示下载指定的包
# Base setting init

	// 根据token类型自动区分pupet协议
	token := "puppet_paimon_3903662148cf1ac4243667487a8778xx" //Ipad
	//token:="puppet_wxwork_afc82579e88f81xx" // 企业
	//token:="puppet_donut_ddaf151fbe8286xx" // web
  
// 初始化wechaty

bot := wechaty.NewWechaty(wechaty.WithPuppetOption(wp.Option{

	Token: token,
	
	// endpoint  & token 2选一

	// Endpoint 指定服务器
	
	// Token 根据token返回结果获取对应的pupet对应的服务

	//Endpoint: "127.0.0.1:30001",
	
	//Token: "3d415ebb-7a6f-4cba-b602-1f4ae400f011",

	// 超时控制
	
	Timeout: time.Duration(2 * time.Minute),
	
}), wechaty.WithName("demo"))
  
  
