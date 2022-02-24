[TOC]

## 学习内容

![学习思维导图](.resource/golang 基础学习.jpeg)

## 名词解释

### GOPATH 和 GOROOT
不同于其他语言，go 中没有项目的说法，只有包, 其中有两个重要的路径，GOROOT 和 GOPATH

Go 开发相关的环境变量如下：

- GOROOT 就是 Go 的安装目录。

- GOPATH 是我们的工作空间,保存 go 项目代码和第三方依赖包。

### GO111MODULE 介绍

- GO111MODULE = off，go 命令行将不会支持 module 功能，寻找依赖包的方式将会沿用旧版本那种通过 vendor 目录或者 GOPATH 模式来查找。

- GO111MODULE = on，go 命令行会使用 modules，而一点也不会去 GOPATH 目录下查找。

- GO111MODULE = auto，默认值，go 命令行将会根据当前目录来决定是否启用 module 功能。这种情况下可以分为两种情形：
  - 1、当前目录在 GOPATH/src 之外且该目录包含 go.mod 文件。
  - 2、当前文件在包含 go.mod 文件的目录下面。

ps：修改修改 module 模式。`go env -w GO111MODULE=on`

### go 命令

- go mod tidy // 添加需要用到但 go.mod 中查不到的模块；删除未使用的模块

## Go 基础

### 反射

反射就是程序能够在运行时检查变量和值，求出它们的类型。

`reflect.Type` 表示 `interface{}` 的具体类型，而 `reflect.Value` 表示它的具体值。`reflect.TypeOf()` 和 `reflect.ValueOf()` 两个函数可以分别返回 `reflect.Type` 和 `reflect.Value`。

例子：

```go
package main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    t := reflect.TypeOf(q)
    v := reflect.ValueOf(q)
    fmt.Println("Type ", t) // Type  main.order
    fmt.Println("Value ", v) // Value  {456 56}
}

func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}

```

- 其他补充
  1. `reflect.ValueOf(a).Interface()` // 取出 a reflect.Value 的值作为 interface{}
  2. `reflect.ValueOf(a).String()` // 取出 reflect.Value 的值作为 string
  3. `Value.Elem()` // 来获取由原始 reflect.Value 包装的值所指向的值（reflect.Value）。

- 注意

在 **go1.6 版本中反射机制会导出所有方法**（不论首字母是大写还是小写），而在**更高版本中反射机制仅会导出首字母大写的方法**。

### 网络编程

- 一些乱七八糟的代码

```go
package main

import (
    "bufio"
    "encoding/gob"
    "io"
    "net"
)

func main() {
    // listen
    // func Listen(network, address string) (Listener, error)
    l, _ := net.Listen("tcp", ":8888") 
    
    // accept
    // Accept() (Conn, error)。Conn 是一个 interface，它实现了 ReadWriterCloser 这个接口
    c, _ := l.Accept() 

    var buff bufio.Writer
    enc := gob.NewEncoder(&buff)
    type Data struct {
        a int
    }
    var data Data
    enc.Encode(data) // 把 data 序列化后的数据存到 buff 中

    var conn io.ReadWriteCloser
    dec := gob.NewDecoder(conn)
    var body interface{}
    dec.Decode(body) // 把 conn 中的数据反序列化到 body 中
}

```

#### http

- mux.HandleFunc

```go
mux := http.NewServeMux()
mux.HandleFunc("/api", apiFunc)
mux.HandleFunc("/", indexFunc)
server := http.Server {
    Addr: "127.0.0.1:3001",
    Handler: mux,
}
server.ListenAndServe()
```

- http.HandleFunc

```go
http.HandleFunc("/api", apiFunc)
http.HandleFunc("/", indexFunc)
http.ListenAndServe("127.0.0.1:3001", nil)
```

它们的区别是啥？通过查看 http 包的 HandleFunc 源码可以看出来。我们是用了 DefaultServerMux。第一种相当于自己管理 ServeMux 实例。第二种则是用 go 提供的默认实例。

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

参考链接：https://ask.csdn.net/questions/1011378

### Mutex

Mutex 用于提供一种加锁机制（Locking Mechanism），可确保在某时刻只有一个协程在临界区运行，以防止出现竞态条件。

Mutex 可以在 sync 包内找到。Mutex 定义了两个方法：Lock 和 Unlock。所有在 Lock 和 Unlock 之间的代码，都只能由一个 Go 协程执行，于是就可以避免竞态条件。

```go
mutex.Lock()
x = x + 1
mutex.Unlock()
```

在上面的代码中，x = x + 1 只能由一个 Go 协程执行，因此避免了竞态条件。

如果有一个 Go 协程已经持有了锁（Lock），当其他协程试图获得该锁时，这些协程会被阻塞，直到 Mutex 解除锁定为止。

### WaitGroup

WaitGroup 用于等待一批 Go 协程执行结束。程序控制会一直阻塞，直到这些协程全部执行完毕。

WaitGroup 使用计数器来工作。当我们调用 WaitGroup 的 Add 并传递一个 int 时，WaitGroup 的计数器会加上 Add 的传参。要减少计数器，可以调用 WaitGroup 的 Done() 方法。Wait() 方法会阻塞调用它的 Go 协程，直到计数器变为 0 后才会停止阻塞。

### 信道

```go
a := make(chan int) // 定义了一个 int 类型的信道 a。
data := <- a // 读取信道 a
a <- data // 写入信道 a
```

无缓冲的信道发送与接收默认是阻塞的。当把数据发送到信道时，程序控制会在发送数据的语句处发生阻塞，直到有其它 Go 协程从信道读取到数据，才会解除阻塞。与此类似，当读取信道的数据时，如果没有其它的协程把数据写入到这个信道，那么读取过程就会一直阻塞着。

```go
package main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {  
    done := make(chan bool)
    go hello(done)
    <-done // 信道接收数据这里发生了阻塞，直到 hello 中往 done 写入数据，程序才往下执行
    fmt.Println("main function")
}
```


### 接口

- 如何确保某个类型实现了某个接口的所有方法呢？

一般可以使用下面的方法进行检测，如果实现不完整，编译期将会报错。

```go
var _ Person = (*Student)(nil)
var _ Person = (*Worker)(nil)
```

将空值 nil 转换为 *Student 类型，再转换为 Person 接口，如果转换失败，说明 Student 并没有实现 Person 接口的所有方法。

### import

go 语言导包时 `.` 和 `_` 的区别是什么？

- `import _ "fmt"` 当导入一个包时，该包下的文件里所有 `init()` 函数都会被执行，然而，有些时候我们并不需要把整个包都导入进来，仅仅是是希望它执行 `init()` 函数而已。这个时候就可以使用 `import _` 引用该包。
- `import . "fmt"` 省略前缀的包名，也就是调用的 `fmt.Println("hello world")` 可以省略的写成 `Println("hello world")`。

### 格式化打印

`%v` 按默认格式输出

`%+v` 在 %v 的基础上额外输出字段名

`%#v` 在 `%+v` 的基础上额外输出类型名

`%t` 打印布尔值：true / false

`fmt.Printf("%%\n")` // 不转义输出一个 %

### generate

go generate 命令是 go 1.4 版本里面新添加的一个命令，当运行 go generate 时，它将扫描与当前包相关的源代码文件，找出所有包含 "//go:generate" 的特殊注释，提取并执行该特殊注释后面的命令，命令为可执行程序，形同 shell 下面执行。

### time

- 转换关系图

![time](./.resource/time.png)

- 实例代码

```go
// 日期格式转 Time
func timeStr2Time(fmtStr, valueStr, locStr string) time.Time {
	loc := time.Local
	if locStr != "" {
		loc, _ = time.LoadLocation(locStr) // 设置时区
	}
	if fmtStr == "" {
		fmtStr = "06-01-02T15:04:05"
	}
	t, _ := time.ParseInLocation(fmtStr, valueStr, loc)
	return t
}

// Time 转日期格式
func time2TimeStr(t time.Time) string {
	fmtStr := "2006-01-02 15:04:05" // format 格式必须是这个
	return t.Format(fmtStr)
}

// 时间戳转 Time
func timeStamp2Time(ts int64) time.Time {
	return time.Unix(ts, 0)
}

// Time 转时间戳
func time2TimeStamp(t time.Time) int64 {
	return t.Unix() // 返回时间戳的单位是秒
}

// 构造 Time
func constructTime() time.Time {
	// func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time { ... }
	return time.Date(20, 1, 1, 1, 1, 1, 1, time.Now().Location())
}
```

- 参考链接

[Golang 时间操作大全](https://blog.csdn.net/asd1126163471/article/details/112504777?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.2&utm_relevant_index=4)

### json

- 序列化（编码）

```go
// NewEncoder 创建一个将数据写入 w 的 Encoder。
func NewEncoder(w io.Writer) Encoder
// Encode 将 v 的 json 编码写入输出流，并会写入一个换行符
func (enc *Encoder) Encode(v interface{}) error
// Marshal 函数返回 v 的 json 编码。
func Marshal(v interface{}) ([]byte, error)
```

- 反序列化（解码）

```go
// NewDecoder 创建一个从 r 读取并解码 json 对象的 Decoder，解码器有自己的缓冲，并可能超前读取部分 json 数据
func NewDecoder(r io.Reader) Decoder
// Decode 从输入流读取下一个 json 编码值并保存在 v 指向的值里，参见 Unmarshal 函数的文档获取细节信息。
func (dec *Decoder) Decode(v interface{}) error
// Unmarshal 函数解析 json 编码的数据并将结果存入 v 指向的值。
func Unmarshal(data []byte, v interface{}) error
```

- 实例代码

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
)

type Student struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	st := Student{Name: "panyu", Age: 18}

	var buf bytes.Buffer
	encoder := json.NewEncoder(&buf)
	encoder.Encode(st) // 序列化
	fmt.Println(buf.String())

	st1 := Student{}
	json.Unmarshal(buf.Bytes(), &st1) // 反序列化
	fmt.Printf("st1: %+v\n", st1)

	bts, _ := json.Marshal(st) // 序列化

	r := bytes.NewBuffer(bts)
	decoder := json.NewDecoder(r)
	st2 := Student{}
	decoder.Decode(&st2) // 反序列化
	fmt.Printf("st2: %+v\n", st2)
}

```

- **JSON 的 Encode/Decode，Marshal/Unmarshal 之间的关系和区别？**

In the `encoding/json` package, the `Marshal` function and the inverse `Unmarshal` function return and operate on single fixed bytes slices. They transform single objects to bytes, and vice versa.

There are also the `Encoder` and `Decoder` types. These contain the `Encode` and `Decode` methods, and they operate on streams of bytes, taking an `io.Reader` and `io.Writer` respectively. They also allow multiple objects to be serialized or deserialized with a newline delimiter using those streams.

The underlying mechanisms of `Marshal/Unmarsha`l functions and the `Encoder/Decoder` types are identical, they both use the same internal `encodeState.marshal` and `decodeState.unmarshal` codepaths. The only real difference is they provide alternative access for various usage patterns.

- 总结一下就是：`Encode/Decode`，`Marshal/Unmarshal` 方法底层调用的都是同一个方法进行序列化或者反序列化。

- 未知格式的 json

上面的代码是我们解析已知的固定格式的 json。但是有时候我们拿到的 json 是未知格式的。那么我们可以用第三方包  simplejson 去解析。参考链接：https://studygolang.com/articles/31227

### 读写文件

- read

```go
func readFile(fileName string) {
	fd, err := os.Open(fileName)
	if err != nil {
		log.Fatal(err)
	}
	defer fd.Close()
	b := bufio.NewReader(fd)
	for {
		line, err := b.ReadString('\n') // 遇到换行结束
		if err != nil {
			break
		}
		line = strings.Trim(line, "\n") // 注意这里会把换行读取进来，需要去掉
        fmt.Println(line)
	}
	return
}
```

- write

```go
func save2File(data []byte, filePath string) {
	f, err := os.OpenFile(filePath, os.O_TRUNC|os.O_WRONLY|os.O_CREATE, 0666)
	if err != nil {
		log.Fatal(err.Error())
	}
	_, err = f.Write(data)
	if err != nil {
		log.Fatal(err.Error())
	}
	f.Close()
}
```

### 闭包

- 闭包就是函数返回一个匿名函数

匿名函数是一个“内联“语句或表达式。匿名函数的优越性在于可以直接使用函数内的变量。

```go
func myfunc() func() int { // 返注意返回值，这里返回一个函数
    i := 0
    return func() int { // 匿名函数
        i += 1 // 可以使用匿名函数外的变量
        return i
    }
}
```

### 指针接收器和值接收器

方法接收者为对象的指针与值有什么区别呢？

如果方法接收者为对象的**指针**，则会修改原对象；如果方法接收者为对象的值，那么在方法中被操作的是原对象的副本，不会影响原对象。

```go
package main  
  
import "fmt"  

type Integer int 
 
// double 乘 2 
func (p *Integer) double() int {   
    *p = *p * 2   
    fmt.Printf("double p = %d\n", *p)  
    return 0   
}  

// square 平方  
func (p Integer) square() int {   
    p = p * p   
    fmt.Printf("square p = %d\n", p)  
    return 0   
}  
  
func main() {
    var i Integer = 2   
    i.double()				// receiver 为对象指针，原对象被修改
    fmt.Println("i = ", i)  
    
    i.square()				// receiver 为对象值，原对象不会被修改
    fmt.Println("i = ", i)  
}

```



## go 工具

### protoc-gen-go

- 安装命令

`go get -u github.com/golang/protobuf/protoc-gen-go` 

- 问题补充

`go get -u github.com/golang/protobuf/protoc-gen-go` 这个时候下载是在 GOPATH 下。需要把 protoc-gen-go mv 到 /usr/local/bin/。或者系统 PATH 设置有 GOPATH 路径那就不用移动。

- 参考链接

https://mp.weixin.qq.com/s/ntYd-b0f7YU7wOaWOHGGzQ

### msgp

msgp 是 MessagePack 的缩写，是一种高效的二进制序列化格式工具。基本 JSON 能做的事，msgp 都能做，而且比 JSON 更快，更小。

- 安装命令

`go get github.com/tinylib/msgp `

- 代码结构

```bash
.
├── go.mod
├── go.sum
├── main.go
└── protocol
    ├── protocol.go
    ├── protocol_gen.go
    └── protocol_gen_test.go
```

实例代码

`protocol.go`

```go
//go:generate msgp
package protocol

type Student struct {
	Name string
	Age  int
}
```

`main.go`

```go
package main

import (
	"bytes"
	"fmt"
	"test/protocol"

	"github.com/tinylib/msgp/msgp"
)

func main() {
	var bufEncode bytes.Buffer

	st := protocol.Student{Name: "xiaobai", Age: 18}
	msgp.Encode(&bufEncode, &st) // 把 st 序列化后往 bufEncode 写入数据

	st1 := protocol.Student{}
	msgp.Decode(&bufEncode, &st1) // 反序列化
	fmt.Printf("st1: %v\n", st1)
}
```

## go 杂项问题

### vscode go mod 报红

因为项目中有些包是引用到了 c++ 的代码，所以需要配置一下 c++ 相关的东西。

1. 在 ~/.zshrc 中加入（这里我用的是 zsh，bash 是另一个文件名字）

```bash
export CGO_LDFLAGS_ALLOW="pkg-config"
export CGO_CFLAGS_ALLOW="-Xpreprocessor"
```

2. 安装相关的库 brew install vips

3. vscode 添加 includePath

编辑 .vscode/c_cpp_properties.json

```bash
includePath": [
	"${workspaceFolder}/**",
 	"/usr/local/include/**" // 添加多这一行
],
```

4. 重启 vscode

## go 数据结构

go 标准库中只有数组和 map 这两个数据结构，要用到其他的数据结构就必须引用第三方的库

- set

github.com/deckarep/golang-set
