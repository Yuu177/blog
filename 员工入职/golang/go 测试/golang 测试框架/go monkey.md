[TOC]

## go monkey 介绍

用来给变量、函数和方法打桩。注意：**moneky 不是线程安全的，不能用在并发测试中**。

- 安装库

`github.com/agiledragon/gomonkey`

- Test Method

```bash
$ cd test 
$ go test -gcflags=all=-l
```

`-gcflags=all=-l` 这一行命令表示关闭编译器内联优化。不关闭可能导致 mock 失败。

## go monkey 使用

### mock 函数

假设我们要调用远程服务器的某个方法。在本地跑测试用例的时候我们不可能真正的建立远程连接，所以需要进行打桩。

```go
package main

import (
	"testing"

	"github.com/agiledragon/gomonkey"
)

// 远程服务器方法
func callRemoteAdd(a, b int) (int, error) {
	// do something in remote computer
	return 0, nil
}

// 本地方法
func compute(a, b int) (int, error) {
	sum, err := callRemoteAdd(a, b)
	return sum, err
}

func TestCompute(t *testing.T) {
    // 对远程服务器的方法进行打桩
	patches := gomonkey.ApplyFunc(callRemoteAdd, func(a, b int) (int, error) {
        return 2, nil
	})
	defer patches.Reset()

	sum, err := compute(1, 1)
	if sum != 2 || err != nil {
		t.Errorf("expected %v, got %v", 2, sum)
	}
}

```

### mock 方法

```go
type computer struct {
}

func (c *computer) CallRemoteAdd(a, b int) (int, error) {
	// do something in remote computer
	return -1, nil
}

func (c *computer) compute(a, b int) (int, error) {
	sum, err := c.CallRemoteAdd(a, b)
	return sum, err
}

func TestComputeMethod(t *testing.T) {
	var c *computer
	// 因为用到反射，所以被打桩的方法名必须为公有的（即首字母大写）
	patches :=gomonkey.ApplyMethod(reflect.TypeOf(c), "CallRemoteAdd", func(_ *computer, a, b int) (int, error) {
		return 2, nil
	})
	defer patches.Reset()

	sum, err := c.compute(1, 1)
	if sum != 2 || err != nil {
		t.Errorf("expected %v, got %v", 2, sum)
	}
}
```

### mock 全局变量

```go
var num = 10

func TestGlobalVar(t *testing.T) {
	patches := gomonkey.ApplyGlobalVar(&num, 12)
	defer patches.Reset()

	if num != 12 {
		t.Errorf("expected %v, got %v", 12, num)
	}
}
```

### mock 函数序列

方法序列也同理

```go
func getRandomString() string {
	return "hello world"
}

func TestGetRandomString(t *testing.T) {
	outputs := []gomonkey.OutputCell{
		{Values: gomonkey.Params{"123"}, Times: 2}, // 模拟函数的第 1,2 次输出。Times 表示输出的次数。
		{Values: gomonkey.Params{"456"}},           // 模拟函数的第 3 次输出
		{Values: gomonkey.Params{"789"}},           // 模拟函数的第 4 次输出
	}

	// 打桩
	patches := gomonkey.ApplyFuncSeq(getRandomString, outputs)
	defer patches.Reset()

	fmt.Printf("getRandomString(): %v\n", getRandomString())
	fmt.Printf("getRandomString(): %v\n", getRandomString())
	fmt.Printf("getRandomString(): %v\n", getRandomString())
	fmt.Printf("getRandomString(): %v\n", getRandomString())
}
```

OutputCell 的 Values 代表返回的值。Times 表示返回几次。

```go
type Params []interface{}
type OutputCell struct {
	Values Params
	Times  int
}
```

运行结果输出

```go
=== RUN   TestGetRandomString
getRandomString(): 123
getRandomString(): 123
getRandomString(): 456
getRandomString(): 789
--- PASS: TestGetRandomString (0.00s)
```

### 注意事项

Monkey 框架的实现中大量使用了反射机制，但是，go1.6 版本和更高版本（比如go1.7）的反射机制有些差异：

> 在 **go1.6 版本中反射机制会导出所有方法**（不论首字母是大写还是小写），而在**更高版本中反射机制仅会导出首字母大写的方法**。

反射机制的这种差异导致了 Monkey 框架的第二个缺陷：在 go1.6 版本中可以成功打桩的首字母小写的方法，当 go 版本升级后 Monkey 框架会显式触发 panic。

## 参考链接

https://www.cnblogs.com/lanyangsh/p/14587921.html