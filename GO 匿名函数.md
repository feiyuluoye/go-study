## GO 匿名函数

/

[TOC]

匿名函数在 Go 语言中的使用场景广泛，尤其在需要简洁表达逻辑、临时操作或封装行为的场合。以下是常见的匿名函数使用场景：

### 1. **回调函数**
匿名函数非常适合用作回调函数，因为可以在需要时直接定义逻辑，而不需要单独定义命名函数。例如，在事件处理或某个操作完成后的回调中使用匿名函数。

```go
package main

import "fmt"

func process(action func(string)) {
    action("Task completed!")
}

func main() {
    process(func(msg string) {
        fmt.Println(msg)
    })
}
```

### 2. **goroutine 中的操作**
匿名函数特别适合用于 goroutine 中，因为它可以让你在内联代码中执行逻辑，而不必定义额外的函数。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go func() {
        fmt.Println("Goroutine executed")
    }()
    
    time.Sleep(time.Second)  // 让主程序等待一秒，避免 goroutine 未完成主程序就结束
}
```

### 3. **延迟操作（defer）**
`defer` 关键字用于在函数返回之前执行某些操作，匿名函数可以配合 `defer` 使用来简化复杂的清理操作。

```go
package main

import "fmt"

func main() {
    defer func() {
        fmt.Println("Clean up resources")
    }()
    
    fmt.Println("Main function execution")
}
```

### 4. **内联处理逻辑**
匿名函数可以作为一种简洁的方式，将某些仅在特定场景中使用的逻辑内联处理，比如在循环或条件判断中使用匿名函数。

```go
package main

import "fmt"

func main() {
    result := func(a, b int) int {
        return a * b
    }(3, 5)
    
    fmt.Println(result)  // 输出 15
}
```

### 5. **闭包**
匿名函数作为闭包可以捕获其外部环境中的变量，这在需要保存状态或上下文的场景中非常有用。

```go
package main

import "fmt"

func main() {
    counter := 0
    increment := func() int {
        counter++
        return counter
    }

    fmt.Println(increment())  // 输出 1
    fmt.Println(increment())  // 输出 2
}
```

### 6. **过滤、映射等函数式编程风格**
在处理集合数据时，匿名函数可以用于类似函数式编程中的 `map`、`filter` 和 `reduce` 操作，尽管 Go 没有内置这类操作，但可以通过匿名函数灵活实现。

```go
package main

import "fmt"

func filter(nums []int, condition func(int) bool) []int {
    result := []int{}
    for _, num := range nums {
        if condition(num) {
            result = append(result, num)
        }
    }
    return result
}

func main() {
    numbers := []int{1, 2, 3, 4, 5, 6}
    
    // 使用匿名函数过滤出偶数
    evenNumbers := filter(numbers, func(n int) bool {
        return n%2 == 0
    })
    
    fmt.Println(evenNumbers)  // 输出 [2 4 6]
}
```

### 7. **测试中的临时逻辑**
在单元测试或集成测试中，匿名函数可以用于快速构造一些临时的逻辑以测试特定的功能。

```go
package main

import "fmt"

func main() {
    test := func() string {
        return "Test successful"
    }
    
    fmt.Println(test())
}
```

### 8. **短期存在的逻辑操作**
有时某些操作只在某一处需要实现，定义一个全局或局部函数显得冗余，匿名函数可以直接在使用处进行定义并执行，避免污染全局命名空间。

```go
package main

import "fmt"

func main() {
    data := []int{1, 2, 3, 4, 5}
    
    // 匿名函数来求和
    sum := func(numbers []int) int {
        total := 0
        for _, number := range numbers {
            total += number
        }
        return total
    }(data)
    
    fmt.Println(sum)  // 输出 15
}
```

### 总结
匿名函数的主要使用场景包括：
- 回调函数
- goroutine 内的操作
- defer 中的延迟操作
- 内联处理逻辑
- 闭包捕获外部变量
- 数据过滤、映射等操作
- 测试中的临时逻辑
- 简化短期存在的逻辑

这些场景使得匿名函数在代码中可以有效提升灵活性、减少重复定义，适合简短且不需要重复使用的操作。