## GO Practise

[TOC]



#### 练习：切片

实现 `Pic`。它应当返回一个长度为 `dy` 的切片，其中每个元素是一个长度为 `dx`，元素类型为 `uint8` 的切片。当你运行此程序时，它会将每个整数解释为灰度值 （好吧，其实是蓝度值）并显示它所对应的图像。

图像的解析式由你来定。几个有趣的函数包括 `(x+y)/2`、`x*y`、`x^y`、`x*log(y)` 和 `x%(y+1)`。

（提示：需要使用循环来分配 `[][]uint8` 中的每个 `[]uint8`。）

（请使用 `uint8(intValue)` 在类型之间转换；你可能会用到 `math` 包中的函数。）

```go
package main

import (
    "math"
    "golang.org/x/tour/pic"
)

func Pic(dx, dy int) [][]uint8 {
    // 创建一个二维切片
    result := make([][]uint8, dy)
    for y := 0; y < dy; y++ {
        // 为每个 y 创建一个长度为 dx 的切片
        row := make([]uint8, dx)
        for x := 0; x < dx; x++ {
            // 使用一个公式来生成像素值，这里使用 x * log(y+1) 来生成图像
            row[x] = uint8(x * int(math.Log(float64(y+1))))
        }
        // 将该行添加到二维切片中
        result[y] = row
    }
    return result
}

func main() {
    pic.Show(Pic)
}

```

![截屏2024-10-12 16.00.20](/Users/apple/Library/Application Support/typora-user-images/截屏2024-10-12 16.00.20.png)

#### 练习：映射 

实现 WordCount。它应当返回一个映射，其中包含字符串 s 中每个“单词”的个数。 函数 wc.Test 会为此函数执行一系列测试用例，并输出成功还是失败。

```go
package main

import (
    "strings"
    "golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
    // 创建一个空的 map 来存储单词和它们的计数
    wordMap := make(map[string]int)

    // 使用 strings.Fields 将字符串切分成单词
    words := strings.Fields(s)

    // 遍历单词并更新 map 中对应单词的计数
    for _, word := range words {
        wordMap[word]++
    }

    return wordMap
}

func main() {
    // 测试函数
    wc.Test(WordCount)
}

```

![截屏2024-10-12 16.04.07](/Users/apple/Library/Application Support/typora-user-images/截屏2024-10-12 16.04.07.png)

#### 练习：斐波纳契闭包

让我们用函数做些好玩的。

实现一个 `fibonacci` 函数，它返回一个函数（闭包），该闭包返回一个[斐波纳契数列](https://zh.wikipedia.org/wiki/斐波那契数列) (0, 1, 1, 2, 3, 5, ...)。

```go
package main

import "fmt"

// 返回一个函数，生成斐波纳契数列
func fibonacci() func() int {
    // 初始化前两个斐波纳契数
    a, b := 0, 1
    return func() int {
        // 返回当前的 a 值（即斐波纳契数列的下一个数）
        fib := a
        // 更新斐波纳契数列，a = b，b = a + b
        a, b = b, a+b
        return fib
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        // 每次调用 f()，返回下一个斐波纳契数
        fmt.Println(f())
    }
}

```

![截屏2024-10-12 16.05.42](/Users/apple/Library/Application Support/typora-user-images/截屏2024-10-12 16.05.42.png)

#### 练习：Stringer 

通过让 IPAddr 类型实现 fmt.Stringer 来打印点号分隔的地址。 例如，IPAddr{1, 2, 3, 4} 应当打印为 "1.2.3.4"。

```go
package main

import (
    "fmt"
)

// 定义 IPAddr 类型，它是一个包含 4 个字节的数组
type IPAddr [4]byte

// 实现 fmt.Stringer 接口
func (ip IPAddr) String() string {
    // 返回通过 fmt.Sprintf 格式化后的点号分隔的 IP 地址
    return fmt.Sprintf("%d.%d.%d.%d", ip[0], ip[1], ip[2], ip[3])
}

func main() {
    // 定义一个 IPAddr 类型的变量并打印
    hosts := map[string]IPAddr{
        "loopback": {127, 0, 0, 1},
        "googleDNS": {8, 8, 8, 8},
    }
    
    for name, ip := range hosts {
        fmt.Printf("%v: %v\n", name, ip)
    }
}

```

```shell
googleDNS: 8.8.8.8
loopback: 127.0.0.1
```







#### 练习：错误

`Sqrt` 函数，修改它使其返回 `error` 值。

`Sqrt` 接受到一个负数时，应当返回一个非 nil 的错误值。复数同样也不被支持。

创建一个新的类型

```
type ErrNegativeSqrt float64
```

并为其实现

```
func (e ErrNegativeSqrt) Error() string
```

方法使其拥有 `error` 值，通过 `ErrNegativeSqrt(-2).Error()` 调用该方法应返回 `"cannot Sqrt negative number: -2"`。

**注意:** 在 `Error` 方法内调用 `fmt.Sprint(e)` 会让程序陷入死循环。可以通过先转换 `e` 来避免这个问题：`fmt.Sprint(float64(e))`。这是为什么呢？

修改 `Sqrt` 函数，使其接受一个负数时，返回 `ErrNegativeSqrt` 值。



```go
package main

import (
    "fmt"
    "math"
)

// 创建自定义错误类型
type ErrNegativeSqrt float64

// 实现 error 接口的 Error 方法
func (e ErrNegativeSqrt) Error() string {
    return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

// 实现 Sqrt 函数
func Sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, ErrNegativeSqrt(x)
    }
    // 计算平方根
    return math.Sqrt(x), nil
}

func main() {
    // 测试 Sqrt 函数
    fmt.Println(Sqrt(2))
    fmt.Println(Sqrt(-2))
}

```

```shell
- 代码解析：
ErrNegativeSqrt 类型：我们定义了一个新的类型 ErrNegativeSqrt，它是 float64 类型的别名。
Error 方法：为 ErrNegativeSqrt 类型实现了 Error() 方法。在该方法中，使用 fmt.Sprintf 来格式化错误消息。要注意的是，直接调用 fmt.Sprint(e) 会导致死循环，因为 fmt.Sprint(e) 会试图再次调用 Error() 方法，导致无限递归。因此，需要先将 e 转换为 float64 类型，再使用 fmt.Sprint(float64(e)) 格式化输出。
Sqrt 函数：在 Sqrt 函数中，我们首先检查 x 是否为负数，如果是负数，则返回自定义的 ErrNegativeSqrt 错误。否则，使用 math.Sqrt 计算并返回平方根。
- 输出示例：
1.4142135623730951 <nil>
0 cannot Sqrt negative number: -2
- 为什么直接 fmt.Sprint(e) 会导致死循环？
因为 fmt.Sprint(e) 会隐式调用 e.Error() 方法。如果你在 Error() 方法内再次调用 fmt.Sprint(e)，它又会再次调用 Error()，从而陷入无限递归。因此，我们在 Error() 方法中使用 fmt.Sprint(float64(e))，确保先将 e 转换为 float64，避免再次调用 Error() 方法。
```

