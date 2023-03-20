

### 1.1创建proto文件

创建一个product.proto文件,把以下必要头加上

```protobuf
//指定当地前proto语法版本,有2和3
syntax = "proto3";
//option go_package="path;name" path 表示生产的go文件存放的地址,会自动生成目录
option go_package="../service";
//包名
package service;

message User{
  string username=1;
  int32 age =2;
  repeated string password=3;
  repeated string addresses = 4;
}
//定义服务主体
service ProdService{
  //定义方法
  rpc GetProductStock(ProductRequest) returns (ProductResponse);
}
```

### 1.2 测试proto生成go代码

#### 1.2.1官方插件 protoc-gen-go不支持service的生成 需要安装protoc-gen-go-grpc

```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

生成命令

生成到go文件的在proto文件下

`--go_opt=paths=source_relative` 和`--go-grpc_opt=paths=source_relative`

```
protoc --go_out=. --go_opt=paths=source_relative  --go-grpc_out=. --go-grpc_opt=paths=source_relative pbfiles/product.proto
```

生成到指定目录

```shell
protoc --go_out=. --go-grpc_out=. pbfiles/product.proto
```

注：

1. `--go_out `和`--go-grpc_out`后面跟`.`就行
2. proto 文件中go_package="path;name" path 表示生产的go文件存放的地址,会自动生成目录

####  1.2.2 github版本的插件protoc-gen-go是以下的成功命令

```shell
protoc   --go_out=./  ./test.proto
```

> **--go_out 路径和proto文件中指定的go_package的路径关系？**   
>
> 1. --go_out指定的是外层路径必须存在的 
>
> 2. go_package的路径是在--go_out的子路径
>

 proto含有service生成go代码

```protobuf
protoc --go_out=plugins=grpc:./ ./product.proto 
```

生成添加grpc环境`plugins=grpc:`

