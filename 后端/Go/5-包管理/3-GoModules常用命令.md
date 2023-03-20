# Golang 依赖管理工具go module

### go module 简介

------

go modules 是golang 1.11新加的特性，用来管理模块中包的依赖关系

#### go mod命令

| 命令                           | 作用                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| go mod init <项目模块名称>     | 生成 go.mod 文件                                             |
| go mod download [path@version] | 下载 go.mod 文件中指明的所有依赖 **[path@version] 是非必写的** |
| go mod tidy                    | 依赖关系处理，根据mod文件                                    |
| go mod graph                   | 查看现有的依赖结构                                           |
| go mod edit                    | 编辑 go.mod 文件                                             |
| go mod vendor                  | 将依赖包复制到项目下的vendor目录                             |
| go mod verify                  | 校验一个模块是否被篡改过                                     |
| go mod why                     | 查看为什么需要依赖某模块                                     |

> 注：`go mod vendor`
>
> **如果包被屏蔽(墙)，可以使用这个命令，随后使用go build -mod =vendor编译**

### 引入第三方包

**GOPROXY代理设置**

1.打开gomod方式，和设置下载点，在终端输入go env，查看是否更改成功。

GOPROXY 的默认值是：`https://proxy.golang.org/github.com/ 国内无法访问`

- 阿里云
  [https://mirrors.aliyun.com/goproxy/](https://links.jianshu.com/go?to=https%3A%2F%2Fmirrors.aliyun.com%2Fgoproxy%2F)

- 七牛云
  [https://goproxy.cn](https://links.jianshu.com/go?to=https%3A%2F%2Fgoproxy.cn%2F),direct

```shell
go env -w GO111MODULE=on

go env -w GOPROXY=https://goproxy.cn,direct

go env -w GOSUMDB=off （关闭包的有效性验证）

go env -w GOSUMDB=“sum.golang.google.cn” （也可设置国内提供的sum 验证服务）

备注：-w 标记 要求一个或多个形式为 NAME=VALUE 的参数且覆盖默认的设置
```



