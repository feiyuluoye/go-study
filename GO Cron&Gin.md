## GO Cron&Gin

[TOC]





### Robfig Cron介绍

`robfig/cron` 是 Go 中一个常用的定时任务库，提供了类似于 Unix cron 表达式的调度功能。可以使用它来执行定时任务，例如每天备份数据库、定期发送报告等。以下是对 `robfig/cron` 的学习和常见使用场景。

#### 1. **安装 `robfig/cron`**

首先，确保在项目中安装了 `robfig/cron`：

```bash
go get github.com/robfig/cron/v3
```

#### 2. **基本用法**

使用 `robfig/cron` 来创建和管理定时任务非常简单。下面是一些基本的使用步骤和示例代码：

#### 示例：创建一个简单的定时任务

```go
package main

import (
	"fmt"
	"time"

	"github.com/robfig/cron/v3"
)

func main() {
	// 创建一个新的 Cron 调度器
	c := cron.New()

	// 添加一个定时任务：每秒执行一次
	c.AddFunc("* * * * * *", func() {
		fmt.Println("Task running at", time.Now())
	})

	// 启动调度器
	c.Start()

	// 让主程序运行，等待定时任务执行
	select {} // 阻塞主 goroutine
}
```

在这个示例中，`AddFunc` 方法将一个匿名函数添加到调度器中，该函数将在每秒钟运行一次。`cron.New()` 创建一个新的调度器实例，`c.Start()` 启动它。

#### 3. **Cron 表达式**

`robfig/cron` 使用一种类 Unix 的 cron 表达式来定义任务的调度时间。标准格式为：

```
秒 分 时 日 月 星期
```

- `秒 (0-59)`
- `分 (0-59)`
- `时 (0-23)`
- `日 (1-31)`
- `月 (1-12)`
- `星期 (0-6)` (星期天=0)

#### 常用的 Cron 表达式示例：

- `"0 * * * * *"`：每分钟的第 0 秒执行
- `"0 30 * * * *"`：每小时的第 30 分钟执行
- `"0 0 12 * * *"`：每天中午 12:00 执行
- `"0 0 0 * * *"`：每天午夜执行
- `"0 0 0 1 * *"`：每月 1 号午夜执行

#### 4. **添加和管理任务**

你可以使用 `AddFunc` 添加任务，也可以使用 `AddJob` 来添加实现了 `cron.Job` 接口的任务：

```go
package main

import (
	"fmt"
	"time"

	"github.com/robfig/cron/v3"
)

// 自定义任务，必须实现 cron.Job 接口
type MyJob struct{}

func (m MyJob) Run() {
	fmt.Println("Custom job running at", time.Now())
}

func main() {
	c := cron.New()

	// 添加一个自定义任务
	c.AddJob("@every 1s", MyJob{})

	// 启动调度器
	c.Start()

	select {} // 阻塞主 goroutine
}
```

#### 5. **上下文支持**

`robfig/cron/v3` 支持任务执行的上下文，例如取消任务、超时等。以下示例展示了如何在任务中使用上下文：

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/robfig/cron/v3"
)

func main() {
	c := cron.New()

	c.AddFunc("*/5 * * * * *", func() {
		ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
		defer cancel()

		select {
		case <-time.After(1 * time.Second):
			fmt.Println("Task completed at", time.Now())
		case <-ctx.Done():
			fmt.Println("Task timed out")
		}
	})

	c.Start()
	select {}
}
```

#### 6. **使用场景**

- **定时任务执行**：可以用于执行各种周期性任务，比如每小时执行数据同步、每天发送报告等。
- **定期清理**：自动清理临时文件、日志或缓存。
- **定时备份**：每天或每周定时备份数据库或文件。
- **调度系统**：构建任务调度系统，定期运行一组任务。
- **批处理任务**：定期处理一批任务，如批量发送电子邮件。

#### 7. **高级用法**

- **任务移除**：可以使用返回的 `EntryID` 移除任务。
- **任务列表**：可以使用 `Entries()` 方法获取当前的任务列表。
- **停止调度器**：使用 `c.Stop()` 停止调度器。

#### 总结

`robfig/cron` 是一个功能强大的定时任务库，具有简单易用的 API 和灵活的调度表达式。它适用于各种需要定期执行任务的场景，例如数据同步、定时备份和批处理任务等。

### Cron 在Gin中实践使用

要将 `robfig/cron` 的定时任务封装到一个单独的文件中，并在 `Gin` 项目中使用，可以按照以下步骤进行：

#### 1. **安装 `robfig/cron`**

首先，确保安装了 `robfig/cron`：

```bash
go get github.com/robfig/cron/v3
```

#### 2. **封装定时任务**

创建一个单独的文件，例如 `cronjob.go`，用于封装定时任务的逻辑。

#### `cronjob.go` - 封装定时任务

```go
package cronjob

import (
	"fmt"
	"time"

	"github.com/robfig/cron/v3"
)

// CronJobManager 用于管理 cron 实例
type CronJobManager struct {
	cron *cron.Cron
}

// NewCronJobManager 创建并返回一个新的 CronJobManager
func NewCronJobManager() *CronJobManager {
	return &CronJobManager{
		cron: cron.New(),
	}
}

// Start 开始执行定时任务
func (m *CronJobManager) Start() {
	m.cron.Start()
}

// Stop 停止执行定时任务
func (m *CronJobManager) Stop() {
	m.cron.Stop()
}

// AddTask 添加一个新的定时任务
func (m *CronJobManager) AddTask(schedule string, task func()) (cron.EntryID, error) {
	return m.cron.AddFunc(schedule, task)
}

// ExampleTask 示例任务
func ExampleTask() {
	fmt.Println("Task running at", time.Now())
}
```

在这个封装中，我们定义了 `CronJobManager`，用于管理 `cron` 实例，包括添加任务、启动和停止任务。`AddTask` 方法用于添加定时任务。

#### 3. **在 `Gin` 项目中使用定时任务**

在 `Gin` 项目的入口文件（通常是 `main.go`）中引入并使用刚才封装的定时任务。

#### `main.go` - 在 `Gin` 中使用封装的定时任务

```go
package main

import (
	"github.com/gin-gonic/gin"
	"your_project/cronjob" // 替换为你项目中 cronjob.go 的路径
)

func main() {
	// 创建一个新的 Gin 路由器
	r := gin.Default()

	// 创建 CronJobManager 实例
	cronManager := cronjob.NewCronJobManager()

	// 添加定时任务，例如每 5 秒执行一次
	_, err := cronManager.AddTask("@every 5s", cronjob.ExampleTask)
	if err != nil {
		panic(err)
	}

	// 启动定时任务
	cronManager.Start()
	defer cronManager.Stop() // 确保程序结束时停止定时任务

	// 定义路由
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	// 启动 Gin 服务器
	r.Run(":8080")
}
```

#### 4. **代码说明**

- **`cronjob.go`**:
  - 封装了 `CronJobManager`，用于管理定时任务。
  - 提供了 `NewCronJobManager`、`Start`、`Stop` 和 `AddTask` 方法，方便在其他地方使用。
  - `ExampleTask` 是一个示例任务函数，用于打印当前时间。

- **`main.go`**:
  - 创建了 `CronJobManager` 的实例，并添加了一个每 5 秒执行一次的定时任务。
  - 使用 `cronManager.Start()` 启动定时任务。
  - 在 `Gin` 路由器上定义了一个简单的 `/ping` 路由用于测试。
  - 使用 `defer` 确保程序退出时停止所有定时任务。

#### 5. **运行项目**

运行项目，控制台会每 5 秒输出一次 `ExampleTask` 中的日志，例如：

```
Task running at 2024-09-19 12:34:56
Task running at 2024-09-19 12:35:01
```

访问 `http://localhost:8080/ping`，会看到：

```json
{
  "message": "pong"
}
```

通过这种方式，你可以在 `Gin` 项目中封装并使用 `robfig/cron` 的定时任务，使其更加模块化和易于维护。