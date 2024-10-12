## Go HTTP库

Go的 `net/http` 包是一个强大且易于使用的库，用于构建HTTP服务器和客户端。通过它，你可以轻松实现HTTP请求的处理、路由、静态文件服务等功能。下面重点以及一个简单的Demo示例。



[TOC]



#### 1. **基础HTTP服务器**
- 使用 `http.ListenAndServe` 启动服务器。
- 使用 `http.HandleFunc` 或者 `http.Handle` 来处理不同的URL路径。

#### 2. **处理请求与响应**
- `http.Request` 结构体：包含客户端请求的所有信息。
- `http.ResponseWriter` 接口：用于向客户端写入HTTP响应。

#### 3. **路由与处理器**
- `http.HandleFunc` 可以将处理逻辑和特定的URL路径绑定在一起。
- 自定义处理器：实现 `http.Handler` 接口，创建复杂的路由。

#### 4. **HTTP方法处理**
- `http.MethodGet`, `http.MethodPost` 等用于区分GET、POST等请求方法。

#### 5. **读取请求参数**
- 查询参数通过 `r.URL.Query()` 访问。
- POST表单数据通过 `r.FormValue()` 访问。

#### 6. **中间件和请求处理链**
- 创建中间件来处理请求的认证、日志等常见任务。

#### Go `net/http` Demo: 简单的HTTP服务器

```go
package main

import (
	"fmt"
	"net/http"
)

// 首页处理器
func homeHandler(w http.ResponseWriter, r *http.Request) {
	// 返回欢迎信息
	fmt.Fprintf(w, "Welcome to the Go HTTP Server!")
}

// 关于页面处理器
func aboutHandler(w http.ResponseWriter, r *http.Request) {
	// 返回关于信息
	fmt.Fprintf(w, "This is the About Page!")
}

// 带参数的处理器
func helloHandler(w http.ResponseWriter, r *http.Request) {
	// 从URL查询参数获取 "name" 值
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}
	// 返回Hello信息
	fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
	// 路由设置
	http.HandleFunc("/", homeHandler)           // 处理首页请求
	http.HandleFunc("/about", aboutHandler)     // 处理关于页面请求
	http.HandleFunc("/hello", helloHandler)     // 处理带参数的URL

	// 启动服务器，监听8080端口
	fmt.Println("Starting server on :8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

### 运行步骤
1. **保存代码**: 将上述代码保存为 `main.go`。
2. **运行服务器**: 通过命令 `go run main.go` 启动服务器。
3. **访问页面**:
   - 打开浏览器访问 `http://localhost:8080`，将看到 "Welcome to the Go HTTP Server!"。
   - 访问 `http://localhost:8080/about`，将看到 "This is the About Page!"。
   - 访问 `http://localhost:8080/hello?name=Go`，将看到 "Hello, Go!"。

### 进一步学习
- **处理JSON数据**: 通过 `json.NewEncoder()` 和 `json.NewDecoder()` 实现JSON请求和响应的处理。
- **使用第三方路由库**: 可以使用类似 `gorilla/mux` 的第三方库，提供更复杂的路由功能。

在Go中，`http.Client` 用于发起HTTP请求，而 `http.Server` 用于处理来自客户端的请求并响应。`net/http` 包内置了这些功能，能够轻松实现HTTP客户端和服务端。

#### 1. **HTTP客户端 (http.Client) 使用**

`http.Client` 是Go中用于发送HTTP请求的核心组件。你可以使用它来发送GET、POST等请求，并处理服务器返回的响应。

##### 基本使用流程：

- 创建 `http.Client` 实例。
- 使用 `client.Get()`、`client.Post()` 等方法发起请求。
- 使用 `resp.Body` 读取响应数据，并记得关闭 `resp.Body`。

##### HTTP客户端示例：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"strings"
)

func main() {
	client := &http.Client{}

	// 1. 发送 GET 请求
	resp, err := client.Get("https://jsonplaceholder.typicode.com/posts/1")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	// 读取响应
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("GET Response:")
	fmt.Println(string(body))

	// 2. 发送 POST 请求
	data := strings.NewReader(`{"title":"foo","body":"bar","userId":1}`)
	resp, err = client.Post("https://jsonplaceholder.typicode.com/posts", "application/json", data)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	body, err = ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("\nPOST Response:")
	fmt.Println(string(body))
}
```

##### 客户端说明：

1. **GET请求**: `client.Get(url)` 发送GET请求，并读取返回的响应。
2. **POST请求**: `client.Post(url, contentType, body)` 发送POST请求，传入JSON数据。

#### 2. **HTTP服务端 (http.Server) 使用**

`http.Server` 用于创建HTTP服务器，它能够处理客户端的请求并做出响应。

##### 基本使用流程：

- 使用 `http.HandleFunc()` 注册路由和对应的处理函数。
- 使用 `http.ListenAndServe()` 启动HTTP服务器并监听特定端口。

##### HTTP服务端示例：

```go
package main

import (
	"fmt"
	"net/http"
)

// 处理根路由请求
func homeHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Welcome to the home page!")
}

// 处理 POST 请求
func postHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method == http.MethodPost {
		fmt.Fprintf(w, "POST request received")
	} else {
		http.Error(w, "Invalid request method", http.StatusMethodNotAllowed)
	}
}

func main() {
	// 路由配置
	http.HandleFunc("/", homeHandler)     // 处理首页路由
	http.HandleFunc("/post", postHandler) // 处理POST请求

	// 启动HTTP服务器，监听8080端口
	fmt.Println("Server starting at port 8080...")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

##### 服务端说明：

1. **注册路由**: 使用 `http.HandleFunc()` 将特定路径与处理函数关联。
2. **处理GET/POST请求**: 可以使用 `r.Method` 检查请求类型。
3. **启动服务器**: 使用 `http.ListenAndServe(":8080", nil)` 启动服务。

#### 3. **客户端与服务端结合使用示例**

下面我们创建一个简单的HTTP服务器和客户端，客户端向服务器发送请求，服务器返回响应。

##### HTTP服务端：

```go
package main

import (
	"fmt"
	"net/http"
)

// 返回Hello信息
func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, Go HTTP Server!")
}

func main() {
	http.HandleFunc("/hello", helloHandler)

	fmt.Println("Server starting at port 8080...")
	http.ListenAndServe(":8080", nil)
}
```

##### HTTP客户端：

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("http://localhost:8080/hello")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	defer resp.Body.Close()

	// 读取响应
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Response from server:")
	fmt.Println(string(body))
}
```

##### 说明：

1. **服务器**: 监听 `/hello` 路由，返回简单的 "Hello, Go HTTP Server!" 消息。
2. **客户端**: 向服务器发送GET请求，打印服务器的响应。