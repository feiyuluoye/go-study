

## GO闭包

[TOC]

高级闭包的使用在 Go 中非常灵活和强大，特别是在需要保存状态或对外部变量进行复杂操作的场景中。闭包可以捕获并保持外部变量的引用，从而允许函数在多次调用时共享状态。这使得它在需要持久化上下文或动态生成行为的地方非常有用。

#### 1. **累加器（状态保持器）**
闭包可以用来创建一个累加器，它可以在每次调用时保存并更新状态。这个例子展示了如何通过闭包创建一个简单的计数器。

```go
package main

import "fmt"

// 创建一个计数器闭包
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

func main() {
    c1 := counter()
    fmt.Println(c1())  // 输出 1
    fmt.Println(c1())  // 输出 2
    fmt.Println(c1())  // 输出 3

    // 新的计数器实例，状态独立
    c2 := counter()
    fmt.Println(c2())  // 输出 1
}
```

这里 `counter` 函数返回了一个匿名函数，这个匿名函数引用并保存了 `count` 变量，即使 `counter` 函数已经返回，`count` 仍然存在并在后续调用中被修改。

#### 2. **缓存（记忆化）**
闭包可以用来创建缓存函数，通过保存已经计算过的结果来避免重复计算。这种模式在性能优化中非常有用，尤其是当某个操作成本较高时。

```go
package main

import "fmt"

// 创建一个带缓存的 Fibonacci 函数
func fibonacci() func(int) int {
    cache := map[int]int{0: 0, 1: 1}
    
    return func(n int) int {
        if result, found := cache[n]; found {
            return result
        }
        cache[n] = fibonacci()(n-1) + fibonacci()(n-2)
        return cache[n]
    }
}

func main() {
    fib := fibonacci()
    fmt.Println(fib(10))  // 输出 55
    fmt.Println(fib(50))  // 输出 12586269025
}
```

在这个例子中，`fibonacci` 函数返回一个闭包，该闭包使用一个缓存来存储 Fibonacci 序列的计算结果。每次计算一个新的 Fibonacci 数时，它会首先检查缓存，避免重复计算。

#### 3. **工厂函数**
闭包可以用于创建工厂函数，根据输入生成具有不同行为的函数。例如，你可以生成不同的数学操作函数。

```go
package main

import "fmt"

// 创建一个工厂函数返回不同的数学操作闭包
func createOperation(op string) func(int, int) int {
    switch op {
    case "+":
        return func(a, b int) int {
            return a + b
        }
    case "-":
        return func(a, b int) int {
            return a - b
        }
    case "*":
        return func(a, b int) int {
            return a * b
        }
    case "/":
        return func(a, b int) int {
            if b != 0 {
                return a / b
            }
            return 0
        }
    default:
        return func(a, b int) int {
            return 0
        }
    }
}

func main() {
    add := createOperation("+")
    fmt.Println(add(5, 3))  // 输出 8

    multiply := createOperation("*")
    fmt.Println(multiply(4, 2))  // 输出 8
}
```

在这个例子中，`createOperation` 是一个工厂函数，根据操作符参数返回对应的闭包函数，可以用于进行加、减、乘、除等不同的数学操作。

#### 4. **函数式编程风格**
通过闭包，你可以实现类似于函数式编程的操作，比如 `map`、`filter` 和 `reduce`，这些操作可以用来对数据进行处理。

```go
package main

import "fmt"

// 使用闭包实现 map 操作
func mapFunc(arr []int, f func(int) int) []int {
    result := make([]int, len(arr))
    for i, v := range arr {
        result[i] = f(v)
    }
    return result
}

func main() {
    numbers := []int{1, 2, 3, 4, 5}
    
    // 使用闭包对数组进行平方操作
    squares := mapFunc(numbers, func(n int) int {
        return n * n
    })
    
    fmt.Println(squares)  // 输出 [1 4 9 16 25]
}
```

这里通过闭包实现了一个简单的 `mapFunc` 操作，传入一个匿名函数用于对每个元素进行操作。

#### 5. **创建动态行为的函数**
闭包也可以根据外部输入生成具有不同行为的函数，比如生成一个带有自定义前缀的日志记录器。

```go
package main

import "fmt"

// 创建一个日志记录器
func createLogger(prefix string) func(string) {
    return func(message string) {
        fmt.Println(prefix + ": " + message)
    }
}

func main() {
    infoLogger := createLogger("INFO")
    errorLogger := createLogger("ERROR")

    infoLogger("This is an info message.")  // 输出 INFO: This is an info message.
    errorLogger("This is an error message.")  // 输出 ERROR: This is an error message.
}
```

通过使用闭包，`createLogger` 函数创建了带有特定前缀的日志记录器，每个日志记录器都具有自己独特的行为。

#### 6. **控制访问权限**
闭包可以用于实现类似于面向对象编程中的私有变量和方法。通过返回的闭包函数，你可以限制外部访问某些变量，只允许通过函数来操作这些变量。

```go
package main

import "fmt"

// 创建一个闭包，用于限制对内部变量的直接访问
func bankAccount() (func(int) int, func() int) {
    balance := 0
    deposit := func(amount int) int {
        balance += amount
        return balance
    }
    getBalance := func() int {
        return balance
    }
    return deposit, getBalance
}

func main() {
    deposit, getBalance := bankAccount()

    deposit(100)
    fmt.Println(getBalance())  // 输出 100

    deposit(50)
    fmt.Println(getBalance())  // 输出 150
}
```

在这个例子中，`balance` 变量只能通过 `deposit` 和 `getBalance` 函数访问，从而实现了类似于私有变量的效果。

### 总结
高级闭包的常见使用场景包括：
- 状态保持（如计数器）
- 缓存和性能优化（记忆化）
- 动态生成行为（工厂模式）
- 函数式编程风格的操作（如 `map`、`filter`）
- 控制访问权限（模拟私有变量）

闭包使得 Go 的函数行为更加灵活，能够轻松实现复杂的逻辑和状态管理，从而增强代码的表达力和简洁性。