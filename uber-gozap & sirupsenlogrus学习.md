## uber-go/zap & sirupsen/logrus学习

[TOC]

### `uber-go/zap` 和 `sirupsen/logrus`日志库对比

`uber-go/zap` 和 `sirupsen/logrus` 是 Go 中两个常用的日志库，各有其优点和适用场景。以下是对这两个库的对比以及它们的使用场景：

#### 1. **性能**

- **Zap**：
  - 以高性能和低内存分配著称。
  - 以效率为核心设计，使用预分配的 JSON 编码器，避免了反射的使用。
  - 适用于日志记录频繁、对性能要求极高的高吞吐量应用（例如微服务、实时系统）。

- **Logrus**：
  - 更易用，但性能较 Zap 慢。
  - 使用反射，提供更简单的接口，但会带来一定的性能开销。
  - 适合那些易用性和灵活性比纯性能更重要的应用（例如命令行工具、内部工具）。

#### 2. **易用性和灵活性**

- **Zap**：
  - 提供两种模式：`SugaredLogger`（更易用，性能略低）和 `Logger`（高性能，灵活性较低）。
  - 创建结构化日志时相对更冗长，灵活性不如 Logrus。
  - 由于注重性能，设置较复杂，学习曲线更陡峭。

- **Logrus**：
  - 用户友好的 API，可轻松创建结构化日志。
  - 使用反射提供更直观的结构化日志接口。
  - 更灵活，上手更容易，适合初学者或简单项目。

#### 3. **结构化日志**

- **Zap**：
  - 默认专注于结构化日志。
  - 使用强类型字段，提高类型安全性，减少运行时错误的可能性。
  - 示例：
    ```go
    logger.Info("User login",
        zap.String("username", "john_doe"),
        zap.Int("user_id", 42),
    )
    ```

- **Logrus**：
  - 支持使用字段的结构化日志，在添加字段方面更灵活。
  - 示例：
    ```go
    logrus.WithFields(logrus.Fields{
        "username": "john_doe",
        "user_id":  42,
    }).Info("User login")
    ```

#### 4. **生态系统和可扩展性**

- **Zap**：
  - 与 Logrus 相比，内置钩子较少，更关注核心性能。
  - 支持自定义编码器和高级配置，用于不同的输出格式。

- **Logrus**：
  - 丰富的生态系统，包含许多内置钩子和第三方扩展。
  - 易于扩展并集成到不同的日志后端，例如将日志发送到外部系统（例如 ElasticSearch、Logstash）。

#### 5. **输出格式**

- **Zap**：
  - 主要输出 JSON 格式，适合结构化日志和机器解析。
  - 支持控制台编码，但在输出定制方面灵活性不如 Logrus。

- **Logrus**：
  - 默认提供更易读的人类可读格式，如文本格式，方便在开发或调试期间阅读。
  - 也支持 JSON 输出，允许轻松定制输出格式。

#### 6. **使用场景**

- **Zap**：
  - 高性能服务、微服务、实时系统以及日志记录可能成为瓶颈的应用。
  - 需要结构化日志以进行日志解析和分析的系统。

- **Logrus**：
  - 当易用性和灵活性比性能更重要的应用。
  - 命令行工具、开发环境或内部工具，适合需要人类可读日志的场景。

#### 总结

- **使用 `Zap`** 如果你需要最高的性能，且应用的高吞吐量对结构化日志和最小化开销有严格要求。
- **使用 `Logrus`** 如果你更注重易用性和灵活性，应用的性能不是首要考虑因素，或者需要与各种日志后端轻松集成。

在许多情况下，选择取决于应用的具体需求，比如所需的性能水平以及对易用性和效率的重视程度。

### Zap库在Gin中的实践

要在 `gin` 项目中使用 `uber-go/zap` 并将其封装成一个可复用的函数，可以按照以下步骤进行：

#### 1. **安装 `zap`**

首先，确保你的项目中安装了 `zap`：

```bash
go get -u go.uber.org/zap
```

#### 2. **封装 `zap` 日志**

创建一个封装 `zap` 日志初始化和使用的函数。以下是一个示例代码，包括了如何将 `zap` 日志集成到 `gin` 中：

#### `logger.go` - 封装 `zap` 日志

```go
package logger

import (
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"time"
)

// InitZapLogger initializes a zap logger
func InitZapLogger() (*zap.Logger, error) {
	// Customizing the zap logger
	config := zap.Config{
		Encoding:         "json", // or "console"
		Level:            zap.NewAtomicLevelAt(zap.InfoLevel),
		OutputPaths:      []string{"stdout"},
		ErrorOutputPaths: []string{"stderr"},
		EncoderConfig: zapcore.EncoderConfig{
			TimeKey:        "time",
			LevelKey:       "level",
			NameKey:        "logger",
			CallerKey:      "caller",
			MessageKey:     "msg",
			StacktraceKey:  "stacktrace",
			LineEnding:     zapcore.DefaultLineEnding,
			EncodeLevel:    zapcore.LowercaseLevelEncoder, // Lowercase level names
			EncodeTime:     zapcore.ISO8601TimeEncoder,    // ISO8601 time format
			EncodeDuration: zapcore.SecondsDurationEncoder,
			EncodeCaller:   zapcore.ShortCallerEncoder,
		},
	}
	
	return config.Build()
}
```

#### 3. **将 `zap` 集成到 `gin` 中**

在 `gin` 项目中，使用中间件将 `zap` 集成到 HTTP 请求处理流程中。

#### `main.go` - 集成 `zap` 日志到 `gin`

```go
package main

import (
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	"your_project/logger" // Replace with the correct path to logger.go
	"time"
)

func main() {
	// Initialize zap logger
	zapLogger, err := logger.InitZapLogger()
	if err != nil {
		panic(err)
	}
	defer zapLogger.Sync() // Flush any buffered log entries

	// Create a new Gin router
	r := gin.New()

	// Custom middleware to log requests with zap
	r.Use(func(c *gin.Context) {
		startTime := time.Now()
		
		// Process request
		c.Next()
		
		// Log request details
		zapLogger.Info("Incoming request",
			zap.String("method", c.Request.Method),
			zap.String("path", c.Request.URL.Path),
			zap.Int("status", c.Writer.Status()),
			zap.Duration("latency", time.Since(startTime)),
			zap.String("client_ip", c.ClientIP()),
		)
	})

	// Define routes
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	// Start the server
	r.Run(":8080")
}
```

#### 4. **说明**

- **`InitZapLogger`**: 封装了 `zap` 的初始化逻辑，创建了一个 `json` 格式的日志配置。可以根据需要调整日志级别、输出路径和编码格式。
- **Gin 中间件**: 在 `gin` 中注册一个中间件，记录 HTTP 请求的基本信息（如方法、路径、状态码、耗时等）到 `zap` 日志。
- **日志输出**: 日志被格式化为 JSON 格式，并输出到标准输出（`stdout`），但你也可以将其配置为输出到文件或其他位置。

#### 5. **运行项目**

运行项目后，访问 `http://localhost:8080/ping`，你将在控制台看到 `zap` 格式化的日志输出，如：

```json
{"level":"info","time":"2024-09-19T12:34:56.789+0800","caller":"main.go:23","msg":"Incoming request","method":"GET","path":"/ping","status":200,"latency":0.001234,"client_ip":"127.0.0.1"}
```

通过这种方式，你可以将 `zap` 日志记录集成到 `gin` 项目中，并通过中间件捕获和记录 HTTP 请求的相关信息。

要在 `gin` 项目中使用 `sirupsen/logrus` 并封装成一个可复用的函数，可以按照以下步骤进行：



### logrus在GIN中的实践

#### 1. **安装 `logrus`**

首先，确保在你的项目中安装了 `logrus`：

```bash
go get -u github.com/sirupsen/logrus
```

#### 2. **封装 `logrus` 日志**

创建一个封装 `logrus` 日志初始化和使用的函数。以下是一个示例代码，包括了如何将 `logrus` 日志集成到 `gin` 中：

#### `logger.go` - 封装 `logrus` 日志

```go
package logger

import (
	"os"

	"github.com/sirupsen/logrus"
)

// InitLogrusLogger initializes a logrus logger
func InitLogrusLogger() *logrus.Logger {
	// Create a new instance of the logger
	logger := logrus.New()

	// Set the output to stdout
	logger.Out = os.Stdout

	// Set the log level (e.g., Info, Warn, Error)
	logger.SetLevel(logrus.InfoLevel)

	// Set the log format to JSON (or Text)
	logger.SetFormatter(&logrus.JSONFormatter{
		TimestampFormat: "2006-01-02 15:04:05",
	})
	
	// Alternatively, use TextFormatter for more readable output
	// logger.SetFormatter(&logrus.TextFormatter{
	//     FullTimestamp: true,
	// })

	return logger
}
```

#### 3. **将 `logrus` 集成到 `gin` 中**

在 `gin` 项目中，使用中间件将 `logrus` 集成到 HTTP 请求处理流程中。

#### `main.go` - 集成 `logrus` 日志到 `gin`

```go
package main

import (
	"time"

	"github.com/gin-gonic/gin"
	"github.com/sirupsen/logrus"
	"your_project/logger" // Replace with the correct path to logger.go
)

func main() {
	// Initialize logrus logger
	log := logger.InitLogrusLogger()

	// Create a new Gin router
	r := gin.New()

	// Custom middleware to log requests with logrus
	r.Use(func(c *gin.Context) {
		startTime := time.Now()

		// Process request
		c.Next()

		// Log request details
		log.WithFields(logrus.Fields{
			"method":     c.Request.Method,
			"path":       c.Request.URL.Path,
			"status":     c.Writer.Status(),
			"latency":    time.Since(startTime),
			"client_ip":  c.ClientIP(),
		}).Info("Incoming request")
	})

	// Define routes
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})

	// Start the server
	r.Run(":8080")
}
```

#### 4. **说明**

- **`InitLogrusLogger`**: 这个函数初始化了一个 `logrus` 日志实例。你可以根据需要设置日志级别和输出格式（如 JSON 或文本）。
- **Gin 中间件**: 在 `gin` 中注册一个中间件，记录 HTTP 请求的基本信息（如方法、路径、状态码、耗时等）到 `logrus` 日志。
- **日志输出**: 日志被格式化为 JSON 格式，并输出到标准输出（`stdout`）。你也可以选择 `TextFormatter` 来生成更易读的输出。

#### 5. **运行项目**

运行项目后，访问 `http://localhost:8080/ping`，你将在控制台看到 `logrus` 格式化的日志输出，例如：

```json
{"client_ip":"127.0.0.1","latency":0.001234,"method":"GET","path":"/ping","status":200,"time":"2024-09-19T12:34:56+08:00","msg":"Incoming request","level":"info"}
```

或者使用 `TextFormatter` 时：

```
time="2024-09-19 12:34:56" level=info msg="Incoming request" client_ip=127.0.0.1 latency=0.001234 method=GET path=/ping status=200
```

通过这种方式，你可以将 `logrus` 日志记录集成到 `gin` 项目中，并通过中间件捕获和记录 HTTP 请求的相关信息。