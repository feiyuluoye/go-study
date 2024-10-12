

## Go conc库学习与使用

[TOC]

`sourcegraph/conc` 是由 Sourcegraph 开发的并发控制库，用于 Go 语言中更方便地管理和协调 Goroutines。`conc` 旨在简化 Go 并发编程，提供更直观的 API 来处理并发操作，尤其是 Goroutines 的生命周期管理、错误处理以及资源安全访问。

### 主要功能和特点

1. **Group 协调 Goroutines**：
   - `conc` 提供了一种 `Group` API，可以帮助管理多个 Goroutines 的并行执行，类似于 `sync.WaitGroup`，但提供了更多的高级特性，例如错误处理和结果收集。

2. **并发任务控制**：
   - `conc` 的 `Group` 可以控制任务的并发度，确保不会过多创建 Goroutines，避免过度并发导致系统压力。

3. **自动错误处理和结果收集**：
   - `conc` 提供了内置的错误处理机制，每个 Goroutine 的错误都能被安全地收集到，避免了 Go 中手动编写错误通道的麻烦。同时，`conc` 还可以收集并发任务的返回结果。

4. **上下文支持**：
   - `conc` 可以与 `context.Context` 一起使用，支持超时、取消等控制。

5. **简洁易用**：
   - 与 Go 标准库相比，`conc` 提供了简洁且易用的 API，降低了并发编程的复杂性。

### `conc` 的安装

可以通过 `go get` 来安装 `conc` 库：

```bash
go get github.com/sourcegraph/conc
```

### 典型使用场景

1. **并行执行多个 Goroutines 并等待它们完成**：
   使用 `conc.Group` 来管理多个 Goroutine，可以确保主 Goroutine 等待所有并发任务完成。

2. **错误收集和处理**：
   可以自动收集并发 Goroutines 中的错误，并提供一个统一的处理机制。

3. **限制并发数**：
   控制 Goroutines 的并发数量，防止系统过载。

### 示例代码

#### 并行执行多个 Goroutines

这是一个简单的示例，演示如何使用 `conc.Group` 来并行执行多个 Goroutines：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    // 创建一个 conc.Group 来管理并发任务
    var group conc.Group

    // 添加多个并发任务
    for i := 0; i < 5; i++ {
        i := i // 避免闭包引用问题
        group.Go(func() {
            // 模拟一些工作
            time.Sleep(time.Duration(i) * time.Second)
            fmt.Printf("Task %d completed\n", i)
        })
    }

    // 等待所有任务完成
    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }
    fmt.Println("All tasks completed")
}
```

#### 错误处理

如果 Goroutines 可能返回错误，`conc.Group` 提供了错误收集功能：

```go
package main

import (
    "errors"
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    var group conc.Group

    // 向 group 添加多个并发任务，其中某个任务会返回错误
    for i := 0; i < 5; i++ {
        i := i
        group.Go(func() error {
            time.Sleep(time.Duration(i) * time.Second)
            if i == 3 {
                return errors.New("error in task 3")
            }
            fmt.Printf("Task %d completed\n", i)
            return nil
        })
    }

    // 等待所有任务完成，并捕获错误
    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }
    fmt.Println("All tasks completed successfully")
}
```

#### 限制并发 Goroutines 数量

通过 `conc.LimitedGroup`，可以限制同时运行的 Goroutines 数量：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    // 创建一个 LimitedGroup，限制并发 Goroutines 的数量为 3
    group := conc.NewLimitedGroup(3)

    // 添加多个并发任务
    for i := 0; i < 10; i++ {
        i := i
        group.Go(func() {
            // 模拟一些工作
            time.Sleep(time.Second)
            fmt.Printf("Task %d completed\n", i)
        })
    }

    // 等待所有任务完成
    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }
    fmt.Println("All tasks completed")
}
```

在这个例子中，即使我们添加了 10 个 Goroutines，实际上只有 3 个任务会同时执行，其余任务会等待前面的任务完成后再继续执行。

#### 使用 `context.Context` 进行任务控制

`conc.Group` 支持与 `context.Context` 一起使用，可以处理任务的超时和取消操作：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    // 创建一个带有超时的 context
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    // 创建一个 conc.Group
    var group conc.Group
    group.WithContext(ctx)

    // 添加并发任务
    for i := 0; i < 5; i++ {
        i := i
        group.Go(func() error {
            time.Sleep(2 * time.Second)
            fmt.Printf("Task %d completed\n", i)
            return nil
        })
    }

    // 等待任务完成或超时
    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }
    fmt.Println("All tasks completed or context timed out")
}
```

如果任务在超时时间内没有完成，则会自动取消未执行完的任务。

### 常见问题

在使用 `sourcegraph/conc` 时，虽然库本身已经提供了简化并发编程的 API，并且在设计上避免了一些常见的错误，但仍然有可能会遇到一些 `panic` 问题。以下是一些可能会遇到的 `panic` 情况，以及相应的原因和解决方案：

### 1. **任务中发生 `panic`**

由于 `conc.Group` 中的每个任务实际上是一个 Goroutine，因此如果某个任务内部发生了 `panic`，默认情况下整个进程都会终止。这是 Go 语言中的常见现象。如果一个任务中出现了未捕获的运行时错误（如数组越界、空指针引用等），会导致 `panic`。

#### 原因：
- 任务函数内有运行时错误，例如空指针访问、除零等。
  
#### 解决方法：
- 在任务中使用 `recover` 来捕获 `panic`，以防止程序崩溃。

**示例：**
```go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    var group conc.Group

    for i := 0; i < 3; i++ {
        i := i
        group.Go(func() {
            defer func() {
                if r := recover(); r != nil {
                    log.Printf("Recovered from panic in task %d: %v\n", i, r)
                }
            }()
            if i == 2 {
                // 模拟一个panic
                panic("something went wrong")
            }
            time.Sleep(time.Second)
            fmt.Printf("Task %d completed\n", i)
        })
    }

    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }
    fmt.Println("All tasks completed")
}
```
在这个示例中，`recover()` 会捕获任务中的 `panic`，防止整个程序崩溃。

### 2. **`conc.Group` 重复调用 `Wait()`**

`conc.Group` 设计用于管理一组并发 Goroutines，并在调用 `Wait()` 时阻塞直到所有 Goroutines 完成。如果你尝试多次调用 `Wait()`，会引发 `panic`，因为 `conc.Group` 只能等待 Goroutines 一次。

#### 原因：
- 调用 `group.Wait()` 后，试图再次调用 `Wait()`。

#### 解决方法：
- 确保每个 `conc.Group` 的 `Wait()` 只调用一次。如果需要重新启动并发任务，应该创建一个新的 `conc.Group` 实例。

**示例：**
```go
var group conc.Group
group.Go(func() {
    time.Sleep(time.Second)
})
group.Wait()

// 再次调用会触发 panic
// group.Wait() // 不能重复调用 Wait
```

### 3. **在 `Wait()` 之前修改任务**

如果你在调用 `Wait()` 后，试图向 `conc.Group` 中添加新的任务，可能会引发 `panic`。这是因为 `conc.Group` 一旦进入等待状态，就不能再接受新的任务。

#### 原因：
- 在调用 `group.Wait()` 后，继续调用 `group.Go()` 添加任务。

#### 解决方法：
- 确保所有任务在调用 `Wait()` 之前都已经添加完毕。如果需要重新添加任务，应该创建一个新的 `conc.Group`。

**示例：**
```go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    var group conc.Group

    group.Go(func() {
        time.Sleep(time.Second)
        fmt.Println("Task 1 completed")
    })

    if err := group.Wait(); err != nil {
        log.Fatalf("Error occurred: %v", err)
    }

    // 不能在 Wait() 后添加任务，否则会 panic
    // group.Go(func() { fmt.Println("New Task") })
}
```

### 4. **`context` 被取消或超时导致的 `panic`**

在与 `context.Context` 一起使用时，如果 `context` 被取消或超时，任务可能会提前终止。如果没有正确处理这种情况，可能会导致 `panic`，尤其是在任务依赖外部资源或状态时。

#### 原因：
- `context` 被取消后，某些任务未能正确处理取消信号，继续执行。

#### 解决方法：
- 在任务中检查 `context.Context` 是否被取消，并正确处理任务的中止。可以通过 `ctx.Done()` 来监听 `context` 的取消信号。

**示例：**
```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    var group conc.Group
    group.WithContext(ctx)

    for i := 0; i < 3; i++ {
        i := i
        group.Go(func() error {
            select {
            case <-time.After(3 * time.Second): // 任务超时长于 context 超时
                fmt.Printf("Task %d completed\n", i)
            case <-ctx.Done(): // 处理 context 取消
                fmt.Printf("Task %d canceled due to context timeout\n", i)
                return ctx.Err()
            }
            return nil
        })
    }

    if err := group.Wait(); err != nil {
        log.Printf("Error occurred: %v\n", err)
    }
}
```
在这个例子中，当 `context` 超时时，任务会被正确取消，并不会继续运行。

### 5. **并发访问共享资源引发的 `panic`**

尽管 `conc.Group` 本身并不会引发与共享资源相关的 `panic`，但如果任务中不安全地并发访问共享变量或资源，仍可能导致数据竞争或 `panic`。

#### 原因：
- 多个 Goroutines 同时访问或修改共享资源而未使用适当的同步机制。

#### 解决方法：
- 使用同步机制（如 `sync.Mutex`）来保护对共享资源的访问，避免数据竞争。

**示例：**
```go
package main

import (
    "fmt"
    "sync"
    "time"

    "github.com/sourcegraph/conc"
)

func main() {
    var group conc.Group
    var mu sync.Mutex // 保护共享资源
    count := 0

    for i := 0; i < 5; i++ {
        group.Go(func() {
            time.Sleep(1 * time.Second)
            mu.Lock() // 锁定共享资源
            count++
            mu.Unlock() // 解锁
        })
    }

    group.Wait()
    fmt.Printf("Final count: %d\n", count)
}
```

在这个示例中，通过 `sync.Mutex` 锁定共享资源 `count`，避免了可能的数据竞争或 `panic`。

### 学习与使用建议

1. **多线程任务协调**：在需要同时执行多个任务时，可以通过 `conc.Group` 轻松管理。
2. **错误处理**：如果需要捕捉每个 Goroutine 的执行错误，可以利用 `conc.Group` 的内置错误处理机制。
3. **并发限制**：当系统对并发量有上限时，`conc.LimitedGroup` 是非常有用的工具，可以帮助控制 Goroutines 数量。
4. **与上下文配合**：在需要任务超时、取消等场景时，建议结合 `context.Context` 使用。

### 总结

`sourcegraph/conc` 是一个简化 Go 并发编程的高效工具库。它提供了比标准库更简洁、更功能丰富的 API，能够更好地处理 Goroutines 的错误、结果以及并发数量控制。同时，它还集成了 `context`，适合处理超时和取消任务的场景。

`sourcegraph/conc` 设计简单且安全，但在使用过程中，开发者仍需注意 Goroutines 常见的并发问题。常见的 `panic` 情况包括任务中的运行时错误、重复调用 `Wait()`、`context` 超时、以及不安全的并发访问。通过正确的任务管理、错误处理和同步机制，可以有效避免这些问题。