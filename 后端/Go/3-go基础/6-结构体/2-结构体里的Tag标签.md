# 结构体里的 Tag 标签

---

一般情况下，我们定义结构体每个字段都是由字段名字以及字段的类型构成，例如：

```GO
type Book struct {
    Name   string
    Target string
    Spend  int
}
```

## Tag 的使用

在字段上增加一个属性，这个属性是用反引号括起来的一个字符串，我们称之为 **Tag(标签)** 。例如：

```go
type Person struct {
    Name   string `json:"name"`
    Target string `json:"target"`
    Spend  int    `json:"spend,omitempty"`
}
```

结构体的 `Tag` 可以是任意的字符串面值，但是通常是一系列用空格分隔的 `key:"value"` 键值对序列；因为值中含有双引号字符，因此成员 `Tag` 一般用原生字符串面值的形式书写。一般我们常用在 `JSON` 的数据处理方面。

`json` 开头键名对应的值用于控制 **encoding/json** 包的编码和解码的行为，并且 **encoding/…** 下面其它的包也遵循这个约定。`Tag` 中 `json` 对应值的第一部分用于指定 `JSON` 对象的名字，比如将 Go 语言中的 `TotalCount` 成员对应到 `JSON` 中的 `total_count` 对象。

上面的例子中 `gender` 字段的 `Tag` 还带了一个额外的 `omitempty` 选项，表示当 Go 语言结构体成员为空或零值时不生成该 `JSON` 对象（这里 `false` 为零值）。例如：

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Book struct {
    Name   string `json:"name"`
    Target string `json:"target"`
    Spend  int    `json:"spend,omitempty"`
}

func main() {
    // Book 1 without Spend
    book1 := Book{
        Name: "Go语言微服务",
        Target:  "全面掌握Go语言",
    }
    // 结构体转为 JSON
    data1, err := json.Marshal(book1)
    if err != nil {
        panic(err)
    }
    // book1 won't print Spend attribute
    fmt.Printf("%s\n", data1)

    // Book 2 have Gender attribute
    book2 := Book{
        Name:   "Go语言微服务",
        Target:  "全面掌握Go语言",
        Spend:  9,
	}
    // 结构体转为 JSON
    data2, err := json.Marshal(book2)
    if err != nil {
        panic(err)
    }
    // person2 will print Gender attribute
    fmt.Printf("%s\n", data2)
}

```

可以看到，因为 `Spend` 字段里有 `omitempty` 属性，因此 **encoding/json** 在将此结构体对象转化为 `JSON` 字符串时，发现对象里面的 `Spend` 为 false ， 0 ，空指针，空接口，空数组，空切片，空映射，空字符串中的一种，就会被忽略。

## Tag 的获取

Tag 的格式上面已经说了，它是由反引号括起来的一系列用空格分隔的 `key:"value"` 键值对序列：

```GO
`key1:"value1" key2:"value2" key3:"value3"`
```

那么我们如何获取到结构体中的 Tag 呢？这里我们用反射的方法。

使用反射的方法获取 Tag 步骤如下：

1. 获取字段
2. 获取 Tag
3. 获取键值对

其中获取字段有三种方式，而获取键值对有两种方式。

```GO
// 三种获取 field
field := reflect.TypeOf(obj).FieldByName("Name")
field := reflect.ValueOf(obj).Type().Field(i)  // i 表示第几个字段
field := reflect.ValueOf(&obj).Elem().Type().Field(i)  // i 表示第几个字段

// 获取 Tag
tag := field.Tag

// 获取键值对
labelValue := tag.Get("label")
labelValue,ok := tag.Lookup("label")
// Get 当没有获取到对应 Tag 的内容，会返回空字符串
```

下面是一个获取 Tag 以及键值对的例子：

```GO

package main

import (
    "fmt"
    "reflect"
)

type Book struct {
    Name   string `json:"name"`
    Target string `json:"target"`
    Spend string `json:"spend,omitempty"`
}

func main() {
    p := reflect.TypeOf(Book{})
    name, _ := p.FieldByName("Name")
    tag := name.Tag
    fmt.Println("Name Tag :", tag)
    keyValue, _ := tag.Lookup("json")
    fmt.Println("key: json, value:", keyValue)
}
```