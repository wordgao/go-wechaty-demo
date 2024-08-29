# go-wechaty-demo
[![Wechaty Contributor Program](https://img.shields.io/badge/Wechaty-Contributor%20Program-green.svg)](https://wechaty.js.org/docs/contributing/)
[![Powered by Wechaty](https://img.shields.io/badge/Powered%20By-Wechaty-brightgreen.svg)](https://wechaty.js.org)
# go 环境与依赖常用命令
```
go mod download 下载模块到本地缓存，缓存路径是$GOPATH/pkg/mod/cache
go mod edit 是提供了命令版编辑go.mod的功能，例如go mod edit -fmt go.mod会格式化go.mod
go mod graph 把模块之间的依赖图显示出来
go mod init 初始化模块（例如把原本dep管理的依赖关系转换过来）
go mod tidy 增加缺失的包，移除没用的包
go mod vendor 把依赖拷贝到vendor/目录下
go mod verify 确认依赖关系
go mod why 解释为什么需要包和模块
go get -u -v github.com/wechaty/go-wechaty 手动显示下载挂载指定的包
```
### TOKEN 申请地址

|token type|register url|
| ------------- | -------- |
|puppet_donut|[token申请地址](https://qiwei.juzibot.com/corpPremium/wechaty)|
|puppet_wxwork|[token申请地址](https://qiwei.juzibot.com/corpPremium/wechaty)|
|puppet_paimon|[token申请地址](http://175.27.139.176)|
|puppet_PadLocal|[token申请地址](http://pad-local.com)|

# puppet service token 类型说明
## [token puppet 协议类型说明](https://wechaty.js.org/docs/explainations/glossary)
| Name                                                         | Platform | Protocol | Life Cycle |
| ------------------------------------------------------------ | -------- | -------- | ---------- |
| [WXWork](https://wechaty.js.org/docs/puppet-services/wxwork) | WeCom    | Windows  | Beta       |
| [Paimon](https://wechaty.js.org/docs/puppet-services/paimon) | WeChat   | Pad      | Beta       |
| [PadLocal](https://wechaty.js.org/docs/puppet-services/padlocal) | WeChat   | Pad      | Beta       |
| [Donut](https://wechaty.js.org/docs/puppet-services/donut)   | WeChat   | Windows  | Deprecated |
## 程序是通过token类型，自动调用对应的Puppet Protocol Service 协议。
### token类型
```
puppet_wxwork_7ce4cf8a1ab789166c39c6: WXWork TOKEN
puppet_padlocal_7ce48a1ab789166c39c6: PadLocal TOKEN
puppet_paimon_7ce4cf8a1ab789166c39c6: Paimon TOKEN
puppet_donut_e7ce4cf8a1ab789166c39c6: Donut TOKEN
56945fa4-7ce8-4cf8-a1a9-b789166c39c6: UUIDv4 TOKEN
```
# go-wechaty  base core setting init
## Base setting init

```go

// 根据token类型自动区分puppet协议
token := "puppet_paimon_3903662148cf1ac4243667487a8778xx" //Ipad

// 初始化wechaty
// Endpoint  & Token 2选一
// Endpoint：指定服务器
// Token：根据token返回结果获取对应的puppet对应的协议
bot := wechaty.NewWechaty(wechaty.WithPuppetOption(wp.Option{
Token: token,
//Endpoint: "127.0.0.1:30001",
//Token: "3d415ebb-7a6f-4cba-b602-1f4ae400f011",

// 超时控制
Timeout: time.Duration(2 * time.Minute),
}), wechaty.WithName("demo"))
```

# 事件监听

```go
// 根据监听事件做进一步处理。

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

// 系统信道监听 转发 到quitSing<-
//os.Interrupt 表示中断
//os.Kill 杀死退出进程
signal.Notify(quitSing, os.Interrupt, os.Kill)

//循环检测是否收到程序中断/退出的信号，如果有则自动退出程序。
select {
//<-quitSing 读
//quitSing<- 写
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
# 正常提示信息
```console
2021/04/26 12:42:59 PuppetService Start()
2021/04/26 12:43:01 PuppetService onGrpcStreamEvent({type:EVENT_TYPE_SCAN payload:{"qrcode":"http://weixin.qq.com/x/4-M1Tg2B68epptbxxxx","status":2}})
扫码登陆: ScanStatusWaiting
https://wechaty.github.io/qrcode/http://weixin.qq.com/x/4-M1Tg2B68epptbxxxx
```

# token错误提示信息

```console
2021/04/26 13:11:00 PuppetService Start()
2021/04/26 13:11:01 discoverServiceEndPoint() err: http.Status:404 Not Found
2021/04/26 13:11:01 puppet start err:  PuppetService Start() rejection: no endpoint
panic: PuppetService Start() rejection: no endpoint

goroutine 1 [running]:
main.main()
	D:/GoProjects/src/go-wechaty-demo/main.go:61 +0x2b7
Process finished with the exit code 2
```
# 常见问题
### 由于0.412，以及wechaty1.2版本TLS不兼容，0.412实际上还是处于关闭状态，因此必须设置为关闭状态
```

- WECHATY_PUPPET_SERVICE_NO_TLS_INSECURE_SERVER=true
- WECHATY_PUPPET_SERVICE_NO_TLS_INSECURE_CLIENT=true

```
### go-wechaty 0.411开始启用token前缀insecure_token
#### 出现如下错误，解决办法保证go-wechaty中的token和傀儡服务器中的自定义token保持一致即可，前提是都包含前缀insecure_
#### token错误提示如下。
```
#### 第 1 种类似错误提示
ERRO[2024-08-01 16:07:22.671] w.initPuppet err: wechaty Puppet Service requires a SNI as prefix of the token.
You can add the "insecure_" prefix to your token
like: "insecure_ce8f6c05-9c6b-41c3-8a0b-1684d8ce51ff
and try again module=wechaty


#### 第 2 种类似错误提示
ERRO[2024-08-01 16:09:53.681] puppet start err: PuppetService Start() rejection: startGrpcStream err:rpc error: code = Unavailable desc = name resolver error: token insecure_ece8f6c05-9c6b-41c3-8a0b-1684d8ce51ff does not exist module=wechaty


```
# docker-compose一次性过，需要修改的主要参数后面标记有星号*
```
services:
  wechatPuppet.:
    image: wechaty/wechaty:1.20
    container_name: wechatPuppet
    restart: always
    network_mode: host
    tty: true
    ports:
      - "1234:1234" # 对外暴露端口，自定义设置*
    volumes:
      - "$PWD:/home/nisa/wechaty"
      - /etc/localtime:/etc/localtime:ro #时间同步
      - /etc/timezone:/etc/timezone:ro  #时间同步
    environment:
      - SET_CONTAINER_TIMEZONE=true #时区
      - CONTAINER_TIMEZONE=Asia/Shanghai #时区
      - /etc/timezone:/etc/timezone #时区
      - /etc/localtime:/etc/localtime #时区
      - TAKE_FILE_OWNERSHIP=true #权限
      - I18N_LOCALE=zh-CN #字符集
      - LC_ALL=zh_CN.UTF-8 #字符集
      - WECHATY_PUPPET_PADLOCAL_TOKEN=puppet_padlocal_ef42cca12a8f41beb78f32ebe1ea94da # 授权上游TOKEN *
      - WECHATY_PUPPET_SERVER_PORT=1234 # 自定义 傀儡服务端口 *
      - WECHATY_PUPPET=wechaty-puppet-padlocal # 协议 *
      - WECHATY_LOG=verbose
      - WECHATY_PUPPET_SERVICE_NO_TLS_INSECURE_SERVER=true #关闭服务端tls *
      - WECHATY_PUPPET_SERVICE_NO_TLS_INSECURE_CLIENT=true #关闭客户端tls *
      - WECHATY_TOKEN=insecure_63facbf5-3d6f-489e-9f7c-fe8dff7e5e55 # 自定义token，包括前缀 *
#       5.设置微信日志
#        启用verbose日志消息输出以便于调试。#
#        更多选项是：#
#        silly: 将输出所有调试消息
#        verbose: 推荐的调试级别
#        info: 禁用调试信息
#        warning: 只有警告信息
#        silence: 没有日志消息
```

