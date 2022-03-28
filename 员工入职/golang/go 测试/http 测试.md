[TOC]

# http 测试

## 测试 http handler 接口

### 创建真实网络连接

假设需要**测试某个 http API 接口**的 handler 能够正常工作，例如 `helloHandler`

```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello world"))
}
```

那我们可以创建真实的网络连接进行测试：

```go
// test code
import (
	"io/ioutil"
	"net"
	"net/http"
	"testing"
)

func handleError(t *testing.T, err error) {
	t.Helper()
	if err != nil {
		t.Fatal("failed", err)
	}
}

func TestConn(t *testing.T) {
	ln, err := net.Listen("tcp", "127.0.0.1:0")
	handleError(t, err)
	defer ln.Close()

	http.HandleFunc("/hello", helloHandler)
	go http.Serve(ln, nil)

	resp, err := http.Get("http://" + ln.Addr().String() + "/hello")
	handleError(t, err)

	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	handleError(t, err)

	if string(body) != "hello world" {
		t.Fatal("expected hello world, but got", string(body))
	}
}
```

- `net.Listen("tcp", "127.0.0.1:0")`：监听一个未被占用的端口，并返回 Listener。
- 调用 `http.Serve(ln, nil)` 启动 http 服务。
- 使用 `http.Get` 发起一个 Get 请求，检查返回值是否正确。
- 尽量不对 http 和 net 库使用 mock，这样可以覆盖较为真实的场景。

### 使用 httptest

针对 http 开发的场景，使用标准库 net/http/httptest 进行测试更为高效。

上述的测试用例改写如下：

```go
// test code
import (
	"io/ioutil"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestConn(t *testing.T) {
	req := httptest.NewRequest("GET", "http://example.com/foo", nil)
	w := httptest.NewRecorder()
	helloHandler(w, req)
	bytes, _ := ioutil.ReadAll(w.Result().Body)

	if string(bytes) != "hello world" {
		t.Fatal("expected hello world, but got", string(bytes))
	}
}
```

使用 httptest 模拟请求对象(req)和响应对象(w)，达到了相同的目的。

**小结**

- 对 http api 接口测试，建议使用 net/http/httptest。

## httpmock

Easy mocking of http responses from external resources.

### 一个简单的 demo

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

### 参考链接

https://www.jianshu.com/p/545963b593de