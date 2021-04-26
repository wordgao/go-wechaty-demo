# go-wechaty  base core setting init



```
go mod download下载模块到本地缓存，缓存路径是$GOPATH/pkg/mod/cache
go mod edit是提供了命令版编辑go.mod的功能，例如go mod edit -fmt go.mod会格式化go.mod
go mod graph把模块之间的依赖图显示出来
go mod init初始化模块（例如把原本dep管理的依赖关系转换过来）
go mod tidy增加缺失的包，移除没用的包
go mod vendor把依赖拷贝到vendor/目录下
go mod verify确认依赖关系
go mod why解释为什么需要包和模块
go get -u -v github.com/xxx/xxx 手动显示下载指定的包
```



# Base setting init

```go

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
```
# 事件监听

```go
// 根据监听的内容进一步出处理对应的信息

//扫码监听
bot.OnScan(onQrCode)

//登陆监听
bot.OnLogin(onLogin)

// 信息监听
bot.OnMessage(onMessage)

// 退出监听
bot.OnLogout(onLogout)

err := bot.Start()
if err != nil {
panic(err)
}

// 创建os通道监听
var quitSing = make(chan os.Signal)

// 系统信道监听 转发 到quitSing->
signal.Notify(quitSing, os.Interrupt, os.Kill)

select {
//<-quitSing 读
//quitSing-< 写
case <-quitSing:
log.Fatal("退出成功！")
}
```

# 登陆

```go
func onLogin(ctx *wechaty.Context, user *user.ContactSelf) {
fmt.Printf("账户 %s 登陆成功！\n", user.Name())
}
```

# 扫码

```go
func onQrCode(ctx *wechaty.Context, qrCode string, status schemas.ScanStatus, data string) {
fmt.Printf("扫码登陆: %v\nhttps://wechaty.github.io/qrcode/%s\n", status, qrCode)
}
```

# 退出

```go
func onLogout(ctx *wechaty.Context, user *user.ContactSelf, reason string) {
fmt.Printf("账号 %s 退出状态: %s\n", user, reason)
}
```

# 信息处理

```go
func onMessage(ctx *wechaty.Context, message *user.Message) {

if message.Self() {
return
}

if message.Age() > 2*60*time.Second {
log.Println("消息已丢弃，因为它太旧（超过2分钟）")
}

if message.Type() != schemas.MessageTypeText || message.Text() != "#ding" {
log.Println("消息已丢弃，因为它与#ding不匹配")
return
}

// 1. 回复信息
_, err := message.Say("dong")
if err != nil {
log.Println(err)
return
}

}
```

