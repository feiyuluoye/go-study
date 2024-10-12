## GO Message Bus

[TOC]

`Message Bus`（消息总线）是一种设计模式，用于在不同系统组件之间进行异步通信。它使得系统的模块可以松耦合地进行交互，而不需要彼此直接依赖。消息总线通常用于事件驱动的架构中，特别是在分布式系统、微服务以及面向事件的系统设计中。

在 Go 中，`Message Bus` 通常用于处理异步事件或在组件之间传递消息，允许多个发布者和订阅者之间进行消息传递。以下是对 `Message Bus` 的学习内容及使用方法：

## 核心概念

- **Publisher（发布者）**：发送消息或事件的组件。它将消息发布到消息总线上。
- **Subscriber（订阅者）**：接收消息或事件的组件。它订阅了特定类型的消息，并在消息发布时收到通知。
- **Topic/Channel（主题或通道）**：消息可以按照主题分类，订阅者订阅特定主题的消息。

### 为什么使用 `Message Bus`？

1. **解耦**：发布者和订阅者不需要彼此知道对方的存在，促进了松耦合架构。
2. **扩展性**：通过引入消息总线，系统可以轻松添加或移除新的订阅者，而不会影响到其他模块。
3. **异步通信**：消息总线允许异步的消息传递，适用于事件驱动或需要异步处理的场景。
4. **分布式架构**：在微服务架构中，消息总线用于实现服务之间的通信。

## Go 中的 `Message Bus` 实现

### 1. 手动实现 `Message Bus`

在 Go 中，可以手动使用 `channels` 和 `goroutines` 来实现一个简单的 `Message Bus`，用于在不同的模块间传递消息。

#### 示例代码：简易 Message Bus 实现

```go
package main

import (
	"fmt"
	"sync"
)

// 消息总线结构体
type MessageBus struct {
	subscribers map[string][]chan interface{}
	lock        sync.RWMutex
}

// 创建一个新的消息总线
func NewMessageBus() *MessageBus {
	return &MessageBus{
		subscribers: make(map[string][]chan interface{}),
	}
}

// 订阅消息
func (mb *MessageBus) Subscribe(topic string) <-chan interface{} {
	mb.lock.Lock()
	defer mb.lock.Unlock()

	ch := make(chan interface{}, 1)
	mb.subscribers[topic] = append(mb.subscribers[topic], ch)
	return ch
}

// 发布消息
func (mb *MessageBus) Publish(topic string, msg interface{}) {
	mb.lock.RLock()
	defer mb.lock.RUnlock()

	if subscribers, found := mb.subscribers[topic]; found {
		for _, ch := range subscribers {
			// 异步发送消息到通道
			go func(c chan interface{}) {
				c <- msg
			}(ch)
		}
	}
}

// 取消订阅
func (mb *MessageBus) Unsubscribe(topic string, ch <-chan interface{}) {
	mb.lock.Lock()
	defer mb.lock.Unlock()

	if subscribers, found := mb.subscribers[topic]; found {
		for i, subscriber := range subscribers {
			if subscriber == ch {
				mb.subscribers[topic] = append(subscribers[:i], subscribers[i+1:]...)
				close(subscriber) // 关闭通道
				break
			}
		}
	}
}

func main() {
	// 创建消息总线
	bus := NewMessageBus()

	// 订阅者1订阅 "topic1"
	subscriber1 := bus.Subscribe("topic1")
	// 订阅者2订阅 "topic1"
	subscriber2 := bus.Subscribe("topic1")

	// 发布者发布消息到 "topic1"
	bus.Publish("topic1", "Hello, World!")

	// 订阅者接收消息
	fmt.Println("Subscriber 1 received:", <-subscriber1)
	fmt.Println("Subscriber 2 received:", <-subscriber2)

	// 发布者发布另一个消息
	bus.Publish("topic1", "Another message")

	// 订阅者接收新消息
	fmt.Println("Subscriber 1 received:", <-subscriber1)
	fmt.Println("Subscriber 2 received:", <-subscriber2)

	// 取消订阅
	bus.Unsubscribe("topic1", subscriber1)

	// 再次发布消息，只有订阅者2会收到
	bus.Publish("topic1", "Message after unsubscribe")

	// 订阅者2接收消息
	fmt.Println("Subscriber 2 received:", <-subscriber2)
}
```

### 解释：

1. **MessageBus 结构体**：
   - `subscribers`: 存储订阅者的 map，`key` 是订阅的主题，`value` 是该主题的订阅者（一个 `chan` 列表）。
   - `lock`: 用于并发安全操作的读写锁。

2. **Subscribe 方法**： 
   - 允许用户订阅某个主题，返回一个接收消息的通道。

3. **Publish 方法**：
   - 发布消息到指定的主题，消息通过异步的 goroutine 发送到所有订阅者。

4. **Unsubscribe 方法**：
   - 从某个主题中移除订阅者，关闭其通道。

### 运行结果：

```bash
Subscriber 1 received: Hello, World!
Subscriber 2 received: Hello, World!
Subscriber 1 received: Another message
Subscriber 2 received: Another message
Subscriber 2 received: Message after unsubscribe
```

### 2. 使用库 `go-micro` 中的 `Event Bus`

`go-micro` 是一个强大的微服务框架，它的 `Event Bus` 功能实现了分布式消息传递，可以用作消息总线。使用 `go-micro` 的消息总线非常适合微服务架构中的服务通信。

#### 安装 `go-micro`

```bash
go get github.com/asim/go-micro/v3
```

#### 使用 `go-micro` 实现消息总线

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/asim/go-micro/v3"
	"github.com/asim/go-micro/v3/broker"
	"github.com/asim/go-micro/v3/events"
)

func main() {
	// 创建 go-micro 服务
	service := micro.NewService(micro.Name("message-bus"))
	service.Init()

	// 事件发布者
	eventBus := events.NewEventStore(service.Client())
	topic := "example.topic"

	// 发布消息
	go func() {
		for i := 0; i < 5; i++ {
			err := eventBus.Publish(topic, fmt.Sprintf("Message %d", i))
			if err != nil {
				log.Fatalf("Error publishing message: %v", err)
			}
			fmt.Println("Published message", i)
		}
	}()

	// 订阅消息
	_, err := eventBus.Subscribe(topic, func(ev *broker.Event) error {
		fmt.Printf("Received message: %s\n", string(ev.Message.Body))
		return nil
	})
	if err != nil {
		log.Fatalf("Error subscribing to topic: %v", err)
	}

	// 运行服务
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```

#### 解释：

1. **Event Bus**：`go-micro` 提供的 `Event Bus` 用于发布和订阅事件。
2. **Publish**：使用 `Publish` 方法发布消息到某个主题。
3. **Subscribe**：通过 `Subscribe` 方法订阅某个主题，当消息发布时，订阅者会接收到事件。

### 3. 使用库 `gobot` 实现消息总线

如果你在构建机器人系统或者处理传感器/硬件交互，可以使用 `gobot` 库，它同样支持消息总线模式。

```bash
go get github.com/hybridgroup/gobot
```

然后可以使用类似的方式来创建事件总线和订阅发布。

## 总结

- **消息总线**是一种解耦的设计模式，适用于分布式系统和事件驱动架构。
- Go 中可以通过使用 `channels` 和 `goroutines` 实现自定义的消息总线，也可以使用 `go-micro` 等库来实现更复杂的分布式消息总线功能。
- 消息总线的典型应用包括：微服务通信、事件驱动架构、异步任务执行等。