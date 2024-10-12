## Paho-mqtt库的使用

[TOC]

### 1.  **安装 MQTT 库 **
首先，你需要安装 `paho.mqtt.golang`：

```bash
go get github.com/eclipse/paho.mqtt.golang
```

### 2. **MQTT 基本概念**
- **Broker（代理）**：MQTT 服务器，负责接收客户端发布的消息并分发给订阅这些主题的客户端。
- **Client（客户端）**：连接到代理的设备或应用程序，负责发布消息或订阅特定主题。
- **Topic（主题）**：MQTT 使用主题来对消息进行分类和路由，发布者将消息发送到特定的主题，订阅者从主题中接收消息。
- **QoS（服务质量）**：MQTT 提供 3 种服务质量级别（QoS 0、QoS 1、QoS 2），用来控制消息的传输可靠性。

### 3. **基本用法**
#### 3.1 创建 MQTT 客户端
首先，创建一个 MQTT 客户端并与代理连接：

```go
package main

import (
	"fmt"
	"log"
	"time"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	// 定义 MQTT 客户端选项
	opts := mqtt.NewClientOptions().
		AddBroker("tcp://localhost:1883"). // 代理地址
		SetClientID("go_mqtt_client").     // 客户端 ID
		SetUsername("user").               // 用户名（可选）
		SetPassword("password")            // 密码（可选）

	// 创建客户端
	client := mqtt.NewClient(opts)

	// 连接到 MQTT 代理
	if token := client.Connect(); token.Wait() && token.Error() != nil {
		log.Fatalf("MQTT Connection Error: %v", token.Error())
	}

	fmt.Println("Connected to MQTT broker")

	// 延迟关闭客户端
	defer client.Disconnect(250)
}
```

#### 3.2 发布消息
使用 `Publish` 方法可以向特定的主题发布消息：

```go
package main

import (
	"fmt"
	"log"
	"time"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	// 创建客户端选项和连接
	opts := mqtt.NewClientOptions().AddBroker("tcp://localhost:1883").SetClientID("publisher")
	client := mqtt.NewClient(opts)

	if token := client.Connect(); token.Wait() && token.Error() != nil {
		log.Fatalf("MQTT Connection Error: %v", token.Error())
	}
	defer client.Disconnect(250)

	// 发布消息
	topic := "example/topic"
	msg := "Hello MQTT"
	token := client.Publish(topic, 0, false, msg)
	token.Wait()

	fmt.Printf("Message published to topic %s: %s\n", topic, msg)
}
```

- `Publish` 方法的参数解释：
  - **topic**：消息的主题。
  - **QoS**：服务质量等级，`0` 表示尽力传输，`1` 表示至少传输一次，`2` 表示仅传输一次。
  - **retained**：是否保留消息，`false` 表示不保留。
  - **payload**：消息的内容。

#### 3.3 订阅主题
使用 `Subscribe` 方法可以订阅一个或多个主题，并接收这些主题的消息：

```go
package main

import (
	"fmt"
	"log"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	// 创建客户端选项和连接
	opts := mqtt.NewClientOptions().AddBroker("tcp://localhost:1883").SetClientID("subscriber")
	client := mqtt.NewClient(opts)

	if token := client.Connect(); token.Wait() && token.Error() != nil {
		log.Fatalf("MQTT Connection Error: %v", token.Error())
	}
	defer client.Disconnect(250)

	// 定义消息处理回调函数
	messageHandler := func(client mqtt.Client, msg mqtt.Message) {
		fmt.Printf("Received message: %s from topic: %s\n", msg.Payload(), msg.Topic())
	}

	// 订阅主题
	topic := "example/topic"
	token := client.Subscribe(topic, 0, messageHandler)
	token.Wait()

	fmt.Printf("Subscribed to topic %s\n", topic)

	// 保持运行，以便接收消息
	select {}
}
```

- **`Subscribe`** 方法的参数解释：
  - **topic**：要订阅的主题。
  - **QoS**：消息传输的服务质量等级。
  - **callback**：当有消息发布到该主题时，调用的回调函数。

#### 3.4 取消订阅
可以使用 `Unsubscribe` 方法取消对某个主题的订阅：

```go
client.Unsubscribe("example/topic").Wait()
fmt.Println("Unsubscribed from topic example/topic")
```

### 4. **高级用法**
#### 4.1 持久化会话
MQTT 支持持久化会话，即当客户端断开连接后，代理可以保留它的订阅和 QoS 1 或 2 消息。在重新连接时，客户端可以接收在离线期间发布的消息。

```go
opts := mqtt.NewClientOptions().
	AddBroker("tcp://localhost:1883").
	SetClientID("persistent_client").
	SetCleanSession(false)  // 启用持久化会话
```

#### 4.2 重连机制
可以通过 `SetAutoReconnect` 来设置自动重连，确保在网络故障后客户端自动尝试重新连接。

```go
opts := mqtt.NewClientOptions().
	AddBroker("tcp://localhost:1883").
	SetClientID("reconnect_client").
	SetAutoReconnect(true)  // 启用自动重连
```

#### 4.3 SSL/TLS 安全连接
如果 MQTT 代理启用了 SSL/TLS，你可以配置客户端使用加密连接：

```go
import (
	"crypto/tls"
	"crypto/x509"
	"io/ioutil"
	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	// 加载 CA 证书
	certpool := x509.NewCertPool()
	ca, err := ioutil.ReadFile("ca.crt")
	if err != nil {
		log.Fatalf("Failed to load CA certificate: %v", err)
	}
	certpool.AppendCertsFromPEM(ca)

	// 配置 TLS 连接
	tlsConfig := &tls.Config{
		RootCAs: certpool,
	}

	opts := mqtt.NewClientOptions().
		AddBroker("ssl://localhost:8883").
		SetClientID("ssl_client").
		SetTLSConfig(tlsConfig)  // 启用 TLS 加密

	client := mqtt.NewClient(opts)
	// 连接和发布消息...
}
```

### 5. **QoS（服务质量等级）详解**
MQTT 提供三种服务质量等级，用来决定消息的可靠性：
- **QoS 0**：消息传输 "尽力而为"，可能会丢失消息。
- **QoS 1**：保证消息至少传递一次，可能会有重复。
- **QoS 2**：保证消息仅传递一次，确保消息无丢失且无重复。

在 `Publish` 和 `Subscribe` 方法中，你可以指定不同的 QoS 级别。

### 6. **错误处理与回调**
可以通过为 `ClientOptions` 设置回调来处理 MQTT 客户端的不同事件：

```go
opts := mqtt.NewClientOptions().
	AddBroker("tcp://localhost:1883").
	SetClientID("callback_client").
	SetConnectionLostHandler(func(client mqtt.Client, err error) {
		fmt.Printf("Connection lost: %v\n", err)
	}).
	SetOnConnectHandler(func(client mqtt.Client) {
		fmt.Println("Connected to MQTT broker")
	})
```

- **`SetConnectionLostHandler`**：当连接丢失时触发的回调。
- **`SetOnConnectHandler`**：当连接成功时触发的回调。

### 7. **错误处理**
当执行 `Connect`、`Publish`、`Subscribe` 等操作时，可以通过 `token.Wait()` 来等待操作完成，并检查是否有错误发生：

```go
token := client.Publish("example/topic", 0, false, "Hello MQTT")
if token.Wait() && token.Error() != nil {
	log.Fatalf("Error publishing message: %v", token.Error())
}
```

### 8. **完整示例：发布和订阅**
一个完整的示例，演示如何发布和订阅消息：

```go
package main

import (
	"fmt"
	"log"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func main() {
	// 创建并连接 MQTT 客户端
	opts := mqtt.NewClientOptions().
		AddBroker("tcp://localhost:1883").
		SetClientID("mqtt_example")
	client := mqtt.NewClient(opts)
	if token := client.Connect(); token.Wait() && token.Error() != nil