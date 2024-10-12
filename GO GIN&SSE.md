## GO GIN & SSE DEMO

[TOC]

将GIN项目与SSE库结合，写一个简单的demo,具备正常的API接口和流处理接口，实现了两种类型的 Server-Sent Events (SSE) 流，以及一个简单的用户管理接口。

下面是对每个接口的详细描述：

### 接口描述：

####1. SSE 接口

##### 1.1 `/events/time` - 时间流

- **功能**：每秒向客户端发送一次当前的时间。
- **路径**：`/events/time`
- **请求方式**：`GET`
- **返回格式**：SSE（文本事件流 `text/event-stream`）
- **数据内容**：每条消息包含当前时间，格式为 RFC3339，例如：`data: 2024-09-14T15:00:00Z`
- **实现细节**：
  - 使用 Gin 处理 HTTP 请求。
  - 设置 HTTP 头以指示客户端进行流式传输。
  - 通过 `for` 循环持续发送当前时间，并使用 `flusher.Flush()` 刷新输出缓冲区，确保数据实时传送到客户端。
  - 每次发送时间后，等待 1 秒钟。

##### 1.2 `/events/numbers` - 数字流

- **功能**：每 2 秒向客户端发送一个递增的数字。
- **路径**：`/events/numbers`
- **请求方式**：`GET`
- **返回格式**：SSE（文本事件流 `text/event-stream`）
- **数据内容**：每条消息包含一个递增的数字，例如：`data: 0`，`data: 1`，`data: 2` 等。
- **实现细节**：
  - 与时间流相似，设置 HTTP 头并初始化 SSE 传输。
  - 使用 `for` 循环持续发送递增的数字，每发送一次数字后，等待 2 秒钟。

#### 2. 用户管理接口

##### 2.1 `/user/:id` - 获取用户信息

- **功能**：根据用户 ID 获取用户信息。
- **路径**：`/user/:id`
- **请求方式**：`GET`
- **参数**：
  - `id`：用户的唯一标识符，通过 URL 参数传递。
- **返回格式**：JSON
- **数据内容**：返回包含用户 ID 和名称的 JSON 对象，例如：`{"user_id": "1", "name": "John Doe"}`
- **实现细节**：
  - 从 URL 中获取用户 ID。
  - 假设从数据库或其他数据源获取用户信息，当前示例中返回了硬编码的用户信息。

##### 2.2 `/user` - 创建用户

- **功能**：创建一个新的用户。
- **路径**：`/user`
- **请求方式**：`POST`
- **请求体**：JSON 格式，包含用户名称，例如：`{"name": "Alice"}`
- **返回格式**：JSON
- **数据内容**：返回新创建的用户信息，包括用户 ID 和名称，例如：`{"user_id": "12345", "name": "Alice"}`
- **实现细节**：
  - 解析请求体中的 JSON 数据，获取用户名称。
  - 验证数据是否合法。
  - 假设将用户信息存储到数据库或其他存储系统，当前示例中返回了硬编码的用户 ID 和名称。

### 项目结构

```
project/
│
├── main.go
├── handlers/
│   ├── event_number.go
│   ├── event_time.go
│   ├── user.go
```

#### 1. `main.go`

`main.go` 文件会引入所有的路由注册函数，包括 `event_time` 和 `event_number`。

```go
package main

import (
	"project/handlers"

	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	// 注册 SSE 路由
	handlers.RegisterEventTimeRoutes(router)
	handlers.RegisterEventNumberRoutes(router)

	// 注册用户路由
	handlers.RegisterUserRoutes(router)

	router.Run(":8080")
}
```

#### 2. 创建 `handlers/event_time.go`

`event_time.go` 文件只处理时间流的 SSE 接口。

```go
package handlers

import (
	"fmt"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
)

func RegisterEventTimeRoutes(router *gin.Engine) {
	router.GET("/events/time", timeStream)
}

// 时间流
func timeStream(c *gin.Context) {
	c.Writer.Header().Set("Content-Type", "text/event-stream")
	c.Writer.Header().Set("Cache-Control", "no-cache")
	c.Writer.Header().Set("Connection", "keep-alive")

	flusher, ok := c.Writer.(http.Flusher)
	if !ok {
		c.String(http.StatusInternalServerError, "Streaming unsupported!")
		return
	}

	for {
		// 发送当前时间
		fmt.Fprintf(c.Writer, "data: %s\n\n", time.Now().Format(time.RFC3339))
		flusher.Flush()
		time.Sleep(1 * time.Second)
	}
}
```

#### 3. 创建 `handlers/event_number.go`

`event_number.go` 文件只处理数字流的 SSE 接口。

```go
package handlers

import (
	"fmt"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
)

func RegisterEventNumberRoutes(router *gin.Engine) {
	router.GET("/events/numbers", numbersStream)
}

// 数字流
func numbersStream(c *gin.Context) {
	c.Writer.Header().Set("Content-Type", "text/event-stream")
	c.Writer.Header().Set("Cache-Control", "no-cache")
	c.Writer.Header().Set("Connection", "keep-alive")

	flusher, ok := c.Writer.(http.Flusher)
	if !ok {
		c.String(http.StatusInternalServerError, "Streaming unsupported!")
		return
	}

	number := 0
	for {
		// 发送递增的数字
		fmt.Fprintf(c.Writer, "data: %d\n\n", number)
		flusher.Flush()
		number++
		time.Sleep(2 * time.Second)
	}
}
```

#### 4. `handlers/user.go`

`user.go` 文件保持不变，用于处理用户相关的接口。

```go
package handlers

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func RegisterUserRoutes(router *gin.Engine) {
	router.GET("/user/:id", getUser)
	router.POST("/user", createUser)
}

func getUser(c *gin.Context) {
	userId := c.Param("id")
	// 假设在这里处理获取用户信息的逻辑
	c.JSON(http.StatusOK, gin.H{
		"user_id": userId,
		"name":    "John Doe",
	})
}

func createUser(c *gin.Context) {
	var json struct {
		Name string `json:"name" binding:"required"`
	}
	if err := c.ShouldBindJSON(&json); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	// 假设在这里处理创建用户的逻辑
	c.JSON(http.StatusCreated, gin.H{
		"user_id": "12345",
		"name":    json.Name,
	})
}
```

#### 5. 运行服务器

1. 将 `main.go`、`event_time.go`、`event_number.go` 和 `user.go` 保存到项目中。
2. 运行项目：
   ```bash
   go run main.go
   ```
3. 服务器将在 `localhost:8080` 上运行。

### 小结

- **模块化**：将每个 SSE 接口拆分到单独的文件中，可以提高代码的可读性和可维护性。
- **路由注册**：在 `main.go` 中分别调用 `RegisterEventTimeRoutes` 和 `RegisterEventNumberRoutes` 注册各自的路由。
- **灵活扩展**：这种结构可以很容易地扩展新的 SSE 接口，只需创建新的文件并注册路由。

这样做可以让每个文件更专注于自己的任务，保持代码清晰整洁，有利于团队合作和代码维护。