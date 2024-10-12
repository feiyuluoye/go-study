## GO Serial 学习与说明



[TOC]

`github.com/goburrow/serial` 是一个 Go 包，提供了一种简单的方式来与串口进行交互。以下是该包的主要特性和用法的简要概述：

### 主要特性
- **跨平台支持**：支持 Windows、macOS 和 Linux。
- **简单的 API**：提供直接的函数来打开、读取、写入和配置串口。
- **配置选项**：允许设置波特率、数据位、奇偶校验、停止位和超时。

### 安装
要使用该包，通过 Go 模块进行安装：

```bash
go get github.com/goburrow/serial
```

### 基本用法
以下是如何使用该包的简单示例：

```go
package main

import (
    "fmt"
    "log"
    "github.com/goburrow/serial"
)

func main() {
    // 配置串口设置
    options := serial.OpenOptions{
        PortName:              "COM3", // 更改为您的端口
        BaudRate:              9600,
        DataBits:              8,
        StopBits:              1,
        Parity:                serial.NoParity,
        RTS:                   true,
        DTR:                   true,
    }

    // 打开串口
    port, err := serial.Open(&options)
    if err != nil {
        log.Fatalf("打开串口时出错: %v", err)
    }
    defer port.Close()

    // 向串口写入数据
    _, err = port.Write([]byte("Hello Serial"))
    if err != nil {
        log.Fatalf("向串口写入时出错: %v", err)
    }

    // 从串口读取数据
    buf := make([]byte, 100)
    n, err := port.Read(buf)
    if err != nil {
        log.Fatalf("从串口读取时出错: %v", err)
    }

    fmt.Printf("接收到: %s\n", buf[:n])
}
```

### 配置选项
- **PortName**: 串口的名称（例如，“COM3”、“/dev/ttyUSB0”）。
- **BaudRate**: 通信的速度（例如，9600、115200）。
- **DataBits**: 数据位数（通常为 8）。
- **StopBits**: 停止位数（1 或 2）。
- **Parity**: 奇偶校验设置（无奇偶校验、奇数校验、偶数校验）。
- **RTS/DTR**: 请求发送和数据终端就绪的控制信号。

### 错误处理
在打开、读取和写入串口时，请确保适当地处理错误，如示例所示。

### 其他功能
您还可以配置超时，并在需要时使用该包进行异步通信。

这应该为您在 Go 应用程序中使用 `goburrow/serial` 包提供了一个坚实的基础！