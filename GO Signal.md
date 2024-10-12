## GO Signal



/

[TOC]

在Go中，`os/signal` 包提供了对操作系统信号的访问和处理。信号是进程之间通信的一种方式，通常由操作系统生成，表示某些事件发生，比如进程终止、挂起、或者其他异步事件。Go中的 `os/signal` 包可以捕获这些信号并做出相应的处理。

### 1. **常见信号类型**
在Unix系统中常见的信号包括：
- `SIGINT` (Ctrl+C)：进程中断信号。
- `SIGTERM`：终止信号，通常用于有序停止进程。
- `SIGHUP`：挂起信号，通常用于重新加载配置。
- `SIGKILL`：强制杀死进程。
- `SIGUSR1` / `SIGUSR2`：用户定义的信号，可以自定义处理。

### 2. **捕获信号的步骤**
Go中的信号处理主要通过 `os/signal` 包的几个函数实现：
- `signal.Notify()`: 注册信号接收器，捕获特定的信号。
- `signal.Ignore()`: 忽略某些信号。
- `signal.Stop()`: 停止接收信号。

信号的捕获通常结合 `chan os.Signal` 信道使用，用于异步接收信号。

### 3. **简单的Signal处理示例**

以下是一个简单的Go程序，展示如何捕获 `SIGINT` 和 `SIGTERM` 信号，并优雅地关闭应用：

```go
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	// 创建一个通道用于接收信号
	sigs := make(chan os.Signal, 1)
	done := make(chan bool, 1)

	// 注册要捕获的信号
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

	// 启动一个goroutine监听信号
	go func() {
		sig := <-sigs
		fmt.Println("\nReceived signal:", sig)
		done <- true
	}()

	fmt.Println("Waiting for signal...")
	<-done
	fmt.Println("Exiting gracefully")
}
```

### 4. **解释代码**
1. **信号通道**: 
   - `sigs := make(chan os.Signal, 1)` 创建了一个信号通道，容量为1，负责接收信号。
   - `signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)` 注册了两个信号，`SIGINT` (Ctrl+C) 和 `SIGTERM` (终止信号)，这些信号会通过 `sigs` 通道传递。
   
2. **信号监听**:
   - 通过 `go func()` 启动一个新的 goroutine 来等待信号，`sig := <-sigs` 表示阻塞等待信号的到来。当信号到来时，将其打印并通过 `done` 通道通知主线程继续执行。

3. **优雅退出**:
   - 程序在 `<-done` 处阻塞，直到捕获到信号后，通过 `done <- true` 解除阻塞，优雅地关闭程序。

### 5. **忽略信号**
有时我们可能不希望处理某些信号，而是忽略它们。这可以使用 `signal.Ignore()` 来实现。

```go
package main

import (
	"os"
	"os/signal"
	"syscall"
)

func main() {
	// 忽略 SIGHUP 和 SIGINT
	signal.Ignore(syscall.SIGHUP, syscall.SIGINT)

	// 程序会继续执行而不会响应 SIGHUP 和 SIGINT 信号
	select {}
}
```

### 6. **实际应用：优雅关闭HTTP服务器**

结合HTTP服务器和信号捕获，可以实现优雅地关闭服务器，确保在接收到 `SIGINT` 或 `SIGTERM` 信号时，完成当前处理的请求后再关闭服务器。

```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	// 创建 HTTP 服务器
	server := &http.Server{Addr: ":8080"}

	// 定义一个简单的处理器
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, World!")
	})

	// 启动服务器
	go func() {
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			fmt.Printf("listen: %s\n", err)
		}
	}()
	fmt.Println("Server started on :8080")

	// 创建信号通道
	sigs := make(chan os.Signal, 1)

	// 监听 SIGINT 和 SIGTERM 信号
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

	// 等待信号到来
	<-sigs
	fmt.Println("\nShutting down server...")

	// 创建一个5秒的上下文用于优雅关闭服务器
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	if err := server.Shutdown(ctx); err != nil {
		fmt.Println("Server forced to shutdown:", err)
	}

	fmt.Println("Server exiting")
}
```

### 7. **总结**

- **`signal.Notify`**: 注册信号通知，可以捕获多个信号并通过信道传递。
- **信号处理**: 通过goroutine监听信号，配合主程序的正常执行，可以实现优雅退出或执行其他自定义操作。
- **实际应用**: 可以用于优雅关闭服务器、重载配置文件或处理进程间通信。