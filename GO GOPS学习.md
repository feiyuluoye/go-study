## GO GOPS学习

[TOC]



`gops` 是一个用于列出和诊断系统中正在运行的 Go 程序的命令行工具。它能够显示 Go 进程的详细信息，包括堆栈跟踪、内存统计信息、Go 版本等，并提供了一些实用的诊断命令。

### 安装 gops
你可以通过以下命令安装 `gops`：
```bash
go get -u github.com/google/gops
```
### 使用 gops
安装完成后，你可以通过在命令行中输入 `gops` 来查看当前系统中运行的所有 Go 程序。如果需要更详细的诊断信息，你可以在 Go 程序中嵌入 `gops` 的代理（agent），以便 `gops` 命令能够与之通信并获取更多信息。

### 在 Go 程序中嵌入 gops 代理
要在 Go 程序中嵌入 `gops` 代理，你需要在 `main` 函数中添加以下代码：
```go
import "github.com/google/gops/agent"

func main() {
    if err := agent.Listen(agent.Options{}); err != nil {
        log.Fatal(err)
    }
    // ... 你的程序逻辑 ...
}
```
这样，`gops` 就能够通过代理来访问你的程序并提供诊断信息。

### gops 的常用命令
- `gops`：列出所有 Go 进程。
- `gops <pid>`：显示指定 PID 的 Go 进程的详细信息。
- `gops tree`：以树状图形式显示所有 Go 进程。
- `gops stack <pid>`：显示指定 PID 的 Go 进程的堆栈跟踪。
- `gops memstats <pid>`：显示指定 PID 的 Go 进程的内存统计信息。
- `gops pprof-heap <pid>`：获取并打开指定 PID 的 Go 进程的堆分析。
- `gops pprof-cpu <pid>`：获取并打开指定 PID 的 Go 进程的 CPU 分析。

当然，以下是 `gops` 常用命令的一些示例用法：

1. **列出所有 Go 进程**：
   ```bash
   gops
   ```
   这个命令会显示系统中所有正在运行的 Go 进程的简要信息。

2. **显示指定 PID 的 Go 进程的详细信息**：
   ```bash
   gops <pid>
   ```
   将 `<pid>` 替换为你想要查看的进程的进程ID。这将显示该进程的详细信息，包括内存使用情况、CPU 使用率、启动命令等。

3. **以树状图形式显示所有 Go 进程**：
   ```bash
   gops tree
   ```
   这个命令会以树状结构展示所有 Go 进程及其相互关系，帮助你理解进程之间的父子关系。

4. **显示指定 PID 的 Go 进程的堆栈跟踪**：
   ```bash
   gops stack <pid>
   ```
   使用 `<pid>` 替换为目标进程的ID。这个命令会显示该进程的所有 goroutine 的堆栈跟踪，这对于调试死锁或理解程序的执行流程非常有用。

5. **显示指定 PID 的 Go 进程的内存统计信息**：
   ```bash
   gops memstats <pid>
   ```
   这个命令会显示指定进程的内存分配和垃圾回收统计信息。

6. **获取并打开指定 PID 的 Go 进程的堆分析**：
   ```bash
   gops pprof-heap <pid>
   ```
   这个命令会获取指定进程的堆分析并启动 `go tool pprof` 来交互式地查看堆使用情况。

7. **获取并打开指定 PID 的 Go 进程的 CPU 分析**：
   ```bash
   gops pprof-cpu <pid>
   ```
   这个命令会获取指定进程的 CPU 分析并启动 `go tool pprof` 来查看 CPU 使用情况。

### 注意事项

- 如果你的程序没有嵌入 `gops` 代理，你将无法使用除了基本列表以外的诊断功能。
- 如果你的程序使用了 UPX 压缩，`gops` 可能无法识别它是一个 Go 程序。