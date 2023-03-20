# http服务器搭建流程

---

- http请求处理--net/http
- 设置路由
- 设置静态资源目录



## http请求处理

```go
func main {
	//根目录 设置回调函数
	http.HandleFunc("/", myHandle)

	// addr：监听的地址
	// handler：回调函数
	http.ListenAndServe("localhost:8080", nil)
}
	// handler 函数
func myHandle(res http.ResponseWriter, req *http.Request) {
	//fmt.Fprintf(writer, "Welcome to go http")

	fmt.Println(req.RemoteAddr, "连接成功")
	// 请求方式：GET POST DELETE PUT UPDATE
	fmt.Println("method:", req.Method)
	// /go
	fmt.Println("url:", req.URL.Path)
	fmt.Println("header:", req.Header)
	fmt.Println("body:", req.Body)
	// 回复
	res.Write([]byte("www.5lmh.com"))
}
```

这样就启动成功了，这时客户端已经开始请求了。

## 路由

 **Go 的`net/http`包为 HTTP 协议提供了很多功能。它做得不好的一件事是复杂的请求路由**

**例如：**

- **支持基于方法的路由**是指可以很容易地根据请求方法(`GET`、`POST`等)将不同的 HTTP 请求分发给不同的[handler](https://so.csdn.net/so/search?q=handler&spm=1001.2101.3001.7020)。
- **支持 URL 路径中的变量**，意思是router可以很容易地声明像`/movies/{ id }`这样的路由，其中`{ id }`是 URL 路径中的动态变量值。
- **支持正则regexp 路由模式**，意思是router可以很容易地声明像`/movies/{[ a-z-] + }`这样的路由，其中`[ a-z-] +` 是 URL 路径中必需的 regexp 匹配。
- **支持基于主机的路由**，意思是，router可以很容易地根据 URL 主机(如 `www.example. com`)而不仅仅是 URL 路径向不同的handler发送 HTTP 请求。
- **支持自定义路由规则**，意思是router可以很容易地为路由请求添加自定义规则(例如根据 IP 地址或header中`Authorization`的值路由到不同的handler)。
- **冲突路由**指的是当注册两个(或更多)路由模式时，它们可能匹配相同的请求 URL 路径。例如，如果您注册了路由`/blog/{ slug }`和`/blog/new`，那么带有路径`/blog/new` 的 HTTP 请求将会匹配这两个路由。

**注意:** 从软件工程的角度来看，路由冲突并不是件好事。它可能就是 bug 和混乱的来源，我们应该在应用程序中尽量避免它们。

常见的路由

### http.ServeMux  ---Go 标准库中的一部分

### go-chi
