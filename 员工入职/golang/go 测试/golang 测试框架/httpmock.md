[TOC]

## httpmock 介绍

Easy mocking of http responses from external resources.

## 一个简单的 demo

```go
package network

import (
	"fmt"
	"net/http"
	"testing"

	"github.com/jarcoal/httpmock"
)

func handle() {
	c := http.DefaultClient
	req, _ := http.NewRequest("GET", "/hello", nil)
	resp, err := c.Do(req)
	if err != nil {
		return
	}
	defer resp.Body.Close()
	fmt.Printf("resp: %v\n", resp)
}

func TestDoHttp(t *testing.T) {
	httpmock.Activate()
	defer httpmock.DeactivateAndReset()

	httpmock.RegisterResponder("GET", `/hello`, httpmock.NewStringResponder(200, string("hello world")))

	handle() // 测试函数
}

```

1、调用 Activate 方法启动 httpmock 环境

2、在 defer 里面调用 `DeactivateAndReset` 结束 mock

3、通过 `httpmock.RegisterResponder` 方法进行 mock 规则注册。

4、这时候再通过 http client 发起的请求就都会被 httpmock 拦截，如果匹配到刚刚注册的规则就会按照注册的内容返回对应 response。如果匹配不到，显示 no responder found。

- 规则注册还可以通过正则表达式。

```go
// Regexp match (could use httpmock.RegisterRegexpResponder instead)
  httpmock.RegisterResponder("GET", `=~^https://api\.mybiz\.com/articles/id/\d+\z`,
    httpmock.NewStringResponder(200, `{"id": 1, "name": "My Great Article"}`))
```

## 参考链接

https://www.jianshu.com/p/545963b593de