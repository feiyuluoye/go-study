## Go websocket



Go 中的 `gorilla/websocket` 是一个常用且高效的 WebSocket 实现库，可以帮助你轻松地在 Web 应用中实现实时通信。学习 `gorilla/websocket` 的基本用法包括建立 WebSocket 连接、发送和接收消息、处理错误、以及在实际场景中的使用。以下是关于 `gorilla/websocket` 的学习步骤和一些使用示例。

### 1. 安装 `gorilla/websocket`
首先，你需要安装 `gorilla/websocket` 包，可以使用以下命令进行安装：
```bash
go get github.com/gorilla/websocket
```

### 2. 基本使用步骤
#### 2.1 建立 WebSocket 服务器
在服务器端，你需要升级 HTTP 连接到 WebSocket，并处理来自客户端的消息。以下是一个简单的 WebSocket 服务器示例：

```go
package main

import (
	"net/http"
	"github.com/gorilla/websocket"
	"log"
)

var upgrader = websocket.Upgrader{
	// CheckOrigin 函数用于控制 WebSocket 的跨域请求
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
}

func handleWebSocket(w http.ResponseWriter, r *http.Request) {
	// 将 HTTP 连接升级到 WebSocket
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("Upgrade error:", err)
		return
	}
	defer conn.Close()

	for {
		// 读取消息
		messageType, message, err := conn.ReadMessage()
		if err != nil {
			log.Println("Read error:", err)
			break
		}

		log.Printf("Received: %s", message)

		// 将消息返回给客户端
		err = conn.WriteMessage(messageType, message)
		if err != nil {
			log.Println("Write error:", err)
			break
		}
	}
}

func main() {
	http.HandleFunc("/ws", handleWebSocket)
	log.Println("Server started on :8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal("ListenAndServe error:", err)
	}
}
```

### 2.2 创建 WebSocket 客户端
客户端可以使用 JavaScript 来连接和与 WebSocket 服务器通信。以下是一个简单的 HTML 客户端示例：

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Test</title>
</head>
<body>
    <script>
        const ws = new WebSocket('ws://localhost:8080/ws');

        ws.onopen = function() {
            console.log('Connected to server');
            ws.send('Hello Server');
        };

        ws.onmessage = function(event) {
            console.log('Received from server:', event.data);
        };

        ws.onclose = function() {
            console.log('Connection closed');
        };
    </script>
</body>
</html>
```

### 3. 高级用法
#### 3.1 处理 JSON 消息
如果你需要处理 JSON 格式的消息，可以使用 `encoding/json` 包对消息进行编码和解码：

```go
import (
	"encoding/json"
)

// 示例结构体
type Message struct {
	Type string `json:"type"`
	Data string `json:"data"`
}

// 读取 JSON 消息
var msg Message
err := conn.ReadJSON(&msg)
if err != nil {
	log.Println("ReadJSON error:", err)
}

// 发送 JSON 消息
err = conn.WriteJSON(Message{Type: "response", Data: "Hello, Client"})
if err != nil {
	log.Println("WriteJSON error:", err)
}
```

#### 3.2 广播消息
在 WebSocket 服务器中，可能需要将消息广播给所有连接的客户端，可以使用一个中央的管理器来跟踪所有的连接：

```go
type Hub struct {
	connections map[*websocket.Conn]bool
	broadcast   chan []byte
}

var hub = Hub{
	connections: make(map[*websocket.Conn]bool),
	broadcast:   make(chan []byte),
}

func (h *Hub) run() {
	for {
		message := <-h.broadcast
		for conn := range h.connections {
			if err := conn.WriteMessage(websocket.TextMessage, message); err != nil {
				conn.Close()
				delete(h.connections, conn)
			}
		}
	}
}

func handleWebSocket(w http.ResponseWriter, r *http.Request) {
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("Upgrade error:", err)
		return
	}
	defer conn.Close()

	hub.connections[conn] = true

	for {
		_, message, err := conn.ReadMessage()
		if err != nil {
			log.Println("Read error:", err)
			break
		}
		hub.broadcast <- message
	}
}

func main() {
	go hub.run()
	http.HandleFunc("/ws", handleWebSocket)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 4. 常见使用场景
- **实时聊天应用**: 使用 WebSocket 可以实现实时聊天功能，支持双向通信。
- **实时通知系统**: 用于推送实时的事件通知，如股票价格更新、社交媒体提醒等。
- **多人在线游戏**: 实现实时互动的多人在线游戏，确保玩家之间的同步。
- **协同编辑工具**: 用于构建实时协作的编辑器，如多人同时编辑文档或白板。

通过学习和使用 `gorilla/websocket`，你可以在 Go 应用中轻松实现实时通信，满足各种实时数据传输的需求。