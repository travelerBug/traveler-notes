



tcp 简单示例：

服务器：

```go
package main

import (
	"fmt"
	"net"
)

func main() {
     //1. 创建一个监听
     listener, err := net.Listen("tcp", ":5000")
    	if listener == nil {
		fmt.Println("listener is nil:", err)
		return
	}
	if err != nil {
		fmt.Println("Error listen:", err)
		return
	}

	defer listener.Close()
         //2. 阻塞---等待连接
	conn, err := listener.Accept()
	if err != nil {
			fmt.Println("Error accepting connection:", err)
			return
	}
         //3. 处理连接--接收/发送数据
         go handleConnect()
}

// 处理连接
func handleConnect(conn net.Conn) {
       //处理完请求后关闭连接
	defer conn.Close()
	//3.等到了一个客户端,打印当前客户端地址
	fmt.Println("client address is ", conn.RemoteAddr())
	//创建缓存
	buf := make([]byte, 1024)
		n, err := conn.Read(buf)
		if err != nil {
			fmt.Println("Error reading", err)
			return
		}
	fmt.Println("客户端发来了：", string(buf[:n]))
	conn.Write(buf[:n])
	
}
```

客户端：

```go
package main

import (
	"fmt"
	"net"
)

func main() {
    //1. 创建一个tcp连接
	conn, err := net.Dial("tcp", "127.0.0.1:5000")
	if err != nil {
		fmt.Println("Error connecting:", err)
		return
	}
	fmt.Println("connect is success")
	//退出时关闭连接
	defer conn.Close()
    
    		byteLength, err2 := conn.Write([]byte(“hello world”))
		if err2 != nil {
			fmt.Println("Error sending data:", err2)
			return
		}
		fmt.Println("send data count:", byteLength)
		if byteLength == 0 {
			continue
		}
		buf := make([]byte, 1024)
		n, err := conn.Read(buf)
		if err != nil {
			fmt.Println("Error reading", err)
			return
		}
		fmt.Println("rec data:", string(buf[:n]))
}
```

