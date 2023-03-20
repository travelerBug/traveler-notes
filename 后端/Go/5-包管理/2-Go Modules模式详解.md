# Go Modules模式

我们接下来用Go Modules的方式创建一个项目, 建议为了与GOPATH分开,不要将项目创建在GOPATH/src下.

#### go mod环境变量

可以通过 go env 命令来进行查看

```bash
GO111MODULE="auto"
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/caowg/Library/Caches/go-build"
GOENV="/Users/caowg/Library/Application Support/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
...
```

##### GO111MODULE

Go语言提供了 GO111MODULE这个环境变量来作为 Go modules 的开关，其允许设置以下参数：

- auto：只要项目包含了 go.mod 文件的话启用 Go modules，目前在 Go1.11 至 Go1.14 中仍然是默认值。
- on：启用 Go modules，推荐设置，将会是未来版本中的默认值。
- off：禁用 Go modules，不推荐设置。

可以通过来设置

```csharp
$ go env -w GO111MODULE=on
```

##### GOPROXY

这个环境变量主要是用于设置 Go 模块代理（Go module proxy）,其作用是用于使 Go 在后续拉取模块版本时直接通过镜像站点来快速拉取。

GOPROXY 的默认值是：`https://proxy.golang.org,direct`

`proxy.golang.org`国内访问不了,需要设置国内的代理.

- 阿里云
   [https://mirrors.aliyun.com/goproxy/](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.aliyun.com%2Fgoproxy%2F)
- 七牛云
   [https://goproxy.cn](https://links.jianshu.com/go?to=https%3A%2F%2Fgoproxy.cn%2F),direct

如:

```go
$ go env -w GOPROXY=https://goproxy.cn,direct
```

GOPROXY 的值是一个以英文逗号 “,” 分割的 Go 模块代理列表，允许设置多个模块代理，假设你不想使用，也可以将其设置为 “off” ，这将会禁止 Go 在后续操作中使用任何 Go 模块代理。

如:

```go
$ go env -w GOPROXY=https://goproxy.cn,https://mirrors.aliyun.com/goproxy/,direct
```

而在刚刚设置的值中，我们可以发现值列表中有 “direct” 标识，它又有什么作用呢？

实际上 “direct” 是一个特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取（比如 GitHub 等），场景如下：当值列表中上一个 Go 模块代理返回 404 或 410 错误时，Go 自动尝试列表中的下一个，遇见 “direct” 时回源，也就是回到源地址去抓取，而遇见 EOF 时终止并抛出类似 “invalid version: unknown revision...” 的错误。

##### GOSUMDB

它的值是一个 Go checksum database，用于在拉取模块版本时（无论是从源站拉取还是通过 Go module proxy 拉取）保证拉取到的模块版本数据未经过篡改，若发现不一致，也就是可能存在篡改，将会立即中止。

GOSUMDB 的默认值为：`sum.golang.org`，在国内也是无法访问的，但是 GOSUMDB 可以被 Go 模块代理所代理（详见：Proxying a Checksum Database）。

因此我们可以通过设置 GOPROXY 来解决，而先前我们所设置的模块代理 goproxy.cn 就能支持代理 sum.golang.org，所以这一个问题在设置 GOPROXY 后，你可以不需要过度关心。

另外若对 GOSUMDB 的值有自定义需求，其支持如下格式：

格式 1：<SUMDB_NAME>+<PUBLIC_KEY>。
 格式 2：<SUMDB_NAME>+<PUBLIC_KEY> <SUMDB_URL>。
 也可以将其设置为“off”，也就是禁止 Go 在后续操作中校验模块版本。

##### GONOPROXY/GONOSUMDB/GOPRIVATE

这三个环境变量都是用在当前项目依赖了私有模块，例如像是你公司的私有 git 仓库，又或是 github 中的私有库，都是属于私有模块，都是要进行设置的，否则会拉取失败。

更细致来讲，就是依赖了由 GOPROXY 指定的 Go 模块代理或由 GOSUMDB 指定 Go checksum database 都无法访问到的模块时的场景。

而一般**建议直接设置 GOPRIVATE，它的值将作为 GONOPROXY 和 GONOSUMDB 的默认值，所以建议的最佳姿势是直接使用 GOPRIVATE**。

并且它们的值都是一个以英文逗号 “,” 分割的模块路径前缀，也就是可以设置多个，例如：



```go
$ go env -w GOPRIVATE="git.example.com,github.com/eddycjy/mquote"
```

设置后，前缀为 [git.xxx.com](https://links.jianshu.com/go?to=http%3A%2F%2Fgit.xxx.com%2F) 和 [github.com/eddycjy/mquote](https://links.jianshu.com/go?to=http%3A%2F%2Fgithub.com%2Feddycjy%2Fmquote) 的模块都会被认为是私有模块。

如果不想每次都重新设置，我们也可以利用通配符，例如：



```go
$ go env -w GOPRIVATE="*.example.com"
```

这样子设置的话，所有模块路径为 [example.com](https://links.jianshu.com/go?to=http%3A%2F%2Fexample.com%2F) 的子域名（例如：[git.example.com](https://links.jianshu.com/go?to=http%3A%2F%2Fgit.example.com%2F)）都将不经过 Go module proxy 和 Go checksum database，**需要注意的是不包括 [example.com](https://links.jianshu.com/go?to=http%3A%2F%2Fexample.com%2F) 本身**。

# 使用Go Modules初始化项目

1. 开启Go Modules

```csharp
 $ go env -w GO111MODULE=on
```

又或是可以通过直接设置系统环境变量（写入对应的~/.bash_profile 文件亦可）来实现这个目的：

```dart
$ export GO111MODULE=on
```

1. 初始化项目

创建项目目录

```ruby
$ mkdir -p $HOME/aceld/modules_test
$ cd $HOME/aceld/modules_test
```

执行Go modules 初始化

```go
$ go mod init github.com/aceld/modules_test
go: creating new go.mod: module github.com/aceld/modules_test
```

在执行 go mod init 命令时，我们指定了模块导入路径为 github.com/aceld/modules_test。接下来我们在该项目根目录下创建 main.go 文件，如下：

```go
package main

import (
    "fmt"
    "github.com/aceld/zinx/znet"
    "github.com/aceld/zinx/ziface"
)

//ping test 自定义路由
type PingRouter struct {
    znet.BaseRouter
}

//Ping Handle
func (this *PingRouter) Handle(request ziface.IRequest) {
    //先读取客户端的数据
    fmt.Println("recv from client : msgId=", request.GetMsgID(), 
              ", data=", string(request.GetData()))

    //再回写ping...ping...ping
    err := request.GetConnection().SendBuffMsg(0, []byte("ping...ping...ping"))
    if err != nil {
      fmt.Println(err)
    }
}

func main() {
    //1 创建一个server句柄
    s := znet.NewServer()

    //2 配置路由
    s.AddRouter(0, &PingRouter{})

    //3 开启服务
    s.Serve()
}
```

我们看当前的main.go也就是我们的`aceld/modules_test`项目,是依赖一个叫`github.com/aceld/zinx`库的. `znet`和`ziface`只是`zinx`的两个模块.

接下来我们在$HOME/aceld/modules_test,本项目的根目录执行

```go
$ go get github.com/aceld/zinx/znet

go: downloading github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100
go: found github.com/aceld/zinx/znet in github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100
```

我们会看到 我们的go.mod被修改,同时多了一个go.sum文件.

1. 查看go.mod文件

```jsx
module github.com/aceld/modules_test

go 1.14

require github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100 // indirect
```

我们来简单看一下这里面的关键字

- module: 用于定义当前项目的模块路径
- go:标识当前Go版本.即初始化版本
- require: 当前项目依赖的一个特定的必须版本
- // indirect: 示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，有可能是你先手动 go get 拉取下来的，也有可能是你所依赖的模块所依赖的.我们的代码很明显是依赖的"github.com/aceld/zinx/znet"和"github.com/aceld/zinx/ziface",所以就间接的依赖了github.com/aceld/zinx

1. 查看go.sum文件
    在第一次拉取模块依赖后，会发现多出了一个 go.sum 文件，其详细罗列了当前项目直接或间接依赖的所有模块版本，并写明了那些模块版本的 SHA-256 哈希值以备 Go 在今后的操作中保证项目所依赖的那些模块版本不会被篡改。



```go
github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100 h1:Ez5iM6cKGMtqvIJ8nvR9h74Ln8FvFDgfb7bJIbrKv54=
github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100/go.mod h1:bMiERrPdR8FzpBOo86nhWWmeHJ1cCaqVvWKCGcDVJ5M=
github.com/golang/protobuf v1.3.3/go.mod h1:vzj43D7+SQXF/4pzW/hwtAqwc6iTitCiVSaWz5lYuqw=
```

我们可以看到一个模块路径可能有如下两种：

h1:hash情况

```undefined
github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100 h1:Ez5iM6cKGMtqvIJ8nvR9h74Ln8FvFDgfb7bJIbrKv54=
```

go.mod hash情况

```go
github.com/aceld/zinx v0.0.0-20200221135252-8a8954e75100/go.mod h1:bMiERrPdR8FzpBOo86nhWWmeHJ1cCaqVvWKCGcDVJ5M=
github.com/golang/protobuf v1.3.3/go.mod h1:vzj43D7+SQXF/4pzW/hwtAqwc6iTitCiVSaWz5lYuqw=
```

h1 hash 是 Go modules 将目标模块版本的 zip 文件开包后，针对所有包内文件依次进行 hash，然后再把它们的 hash 结果按照固定格式和算法组成总的 hash 值。

而 h1 hash 和 go.mod hash 两者，要不就是同时存在，要不就是只存在 go.mod hash。那什么情况下会不存在 h1 hash 呢，就是当 Go 认为肯定用不到某个模块版本的时候就会省略它的 h1 hash，就会出现不存在 h1 hash，只存在 go.mod hash 的情况。

# 修改模块的版本依赖关系

假定我们现在都zinx版本作了升级, 由zinx v0.0.0-20200221135252-8a8954e75100 升级到 zinx v0.0.0-20200306023939-bc416543ae24 (注意zinx是一个没有打版本tag打第三方库,如果有的版本号是有tag的,那么可以直接对应v后面的版本号即可)

那么,我们是怎么知道zinx做了升级呢, 我们又是如何知道的最新的zinx版本号是多少呢?

先回到$HOME/aceld/modules_test,本项目的根目录执行

```go
$ go get github.com/aceld/zinx/znet
go: downloading github.com/aceld/zinx v0.0.0-20200306023939-bc416543ae24
go: found github.com/aceld/zinx/znet in github.com/aceld/zinx v0.0.0-20200306023939-bc416543ae24
go: github.com/aceld/zinx upgrade => v0.0.0-20200306023939-bc416543ae24
```

这样我们,下载了最新的zinx, 版本是v0.0.0-20200306023939-bc416543ae24

然后,我么看一下go.mod

```jsx
module github.com/aceld/modules_test

go 1.14

require github.com/aceld/zinx v0.0.0-20200306023939-bc416543ae24 // indirect
```

我们会看到,当我们执行go get 的时候, 会自动的将本地将当前项目的require更新了.变成了最新的依赖.

好了, 现在我们就要做另外一件事,就是,我们想用一个旧版本的zinx. 来修改当前zinx模块的依赖版本号.

目前我们在$GOPATH/pkg/mod/github.com/aceld下,已经有了两个版本的zinx库

```ruby
/go/pkg/mod/github.com/aceld$ ls
zinx@v0.0.0-20200221135252-8a8954e75100
zinx@v0.0.0-20200306023939-bc416543ae24
```

目前,我们/aceld/modules_test依赖的是zinx@v0.0.0-20200306023939-bc416543ae24 这个是最新版, 我们要改成之前的版本zinx@v0.0.0-20200306023939-bc416543ae24.

回到/aceld/modules_test项目目录下,执行

```ruby
$ go mod edit -replace=zinx@v0.0.0-20200306023939-bc416543ae24=zinx@v0.0.0-20200221135252-8a8954e75100
```

然后我们打开go.mod查看一下

```jsx
module github.com/aceld/modules_test

go 1.14

require github.com/aceld/zinx v0.0.0-20200306023939-bc416543ae24 // indirect

replace zinx v0.0.0-20200306023939-bc416543ae24 => zinx v0.0.0-20200221135252-8a8954e75100
```

这里出现了replace关键字.用于将一个模块版本替换为另外一个模块版本。