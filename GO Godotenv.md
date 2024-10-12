## GO Godotenv

[TOC]

`godotenv` 是一个用于加载和解析 `.env` 文件的 Go 包，这些文件通常用于管理应用程序的环境变量。使用 `.env` 文件可以将配置参数与代码分离，便于管理不同环境（如开发、测试、生产）的配置。

## 安装 `godotenv`

要在 Go 项目中使用 `godotenv`，首先需要安装它：

```bash
go get github.com/joho/godotenv
```

安装完成后，你就可以在代码中引入 `godotenv` 并使用它了。

## 创建 `.env` 文件

在项目的根目录创建一个 `.env` 文件，用于存储环境变量，例如：

```env
# .env 文件
APP_NAME=MyApp
APP_ENV=development
DB_HOST=localhost
DB_PORT=5432
DB_USER=user
DB_PASSWORD=secret
```

## 加载 `.env` 文件

在 Go 代码中使用 `godotenv` 来加载 `.env` 文件的内容，并将其作为环境变量导入：

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
)

func main() {
	// 加载 .env 文件
	err := godotenv.Load()
	if err != nil {
		log.Fatalf("Error loading .env file: %v", err)
	}

	// 读取环境变量
	appName := os.Getenv("APP_NAME")
	appEnv := os.Getenv("APP_ENV")
	dbHost := os.Getenv("DB_HOST")
	dbPort := os.Getenv("DB_PORT")

	fmt.Println("App Name:", appName)
	fmt.Println("App Environment:", appEnv)
	fmt.Println("Database Host:", dbHost)
	fmt.Println("Database Port:", dbPort)
}
```

### 解释

1. **加载 `.env` 文件**：
   ```go
   err := godotenv.Load()
   ```
   `godotenv.Load()` 会加载当前目录中的 `.env` 文件，并将其中的内容作为环境变量导入。如果没有找到 `.env` 文件或发生其他错误，它会返回一个错误。

2. **读取环境变量**：
   ```go
   appName := os.Getenv("APP_NAME")
   ```
   使用 `os.Getenv("变量名")` 来获取环境变量的值，如果该变量不存在，返回空字符串。

3. **错误处理**：
   ```go
   if err != nil {
       log.Fatalf("Error loading .env file: %v", err)
   }
   ```
   如果 `.env` 文件加载失败，使用 `log.Fatalf` 打印错误并退出程序。

## `godotenv` 的其他用法

### 1. 指定 `.env` 文件路径

如果 `.env` 文件不在当前目录，或者你有多个 `.env` 文件，可以通过 `godotenv.Load()` 指定文件路径：

```go
err := godotenv.Load("/path/to/your/.env")
if err != nil {
	log.Fatalf("Error loading .env file: %v", err)
}
```

### 2. 加载多个 `.env` 文件

你可以同时加载多个 `.env` 文件，后面的文件会覆盖前面的相同变量：

```go
err := godotenv.Load(".env", ".env.production")
if err != nil {
	log.Fatalf("Error loading .env files: %v", err)
}
```

### 3. 加载 `.env` 文件到指定的环境

如果你不希望将 `.env` 文件的变量直接加载到环境变量中，可以使用 `godotenv.Read()` 来读取 `.env` 文件内容：

```go
envMap, err := godotenv.Read(".env")
if err != nil {
	log.Fatalf("Error reading .env file: %v", err)
}

fmt.Println(envMap["APP_NAME"]) // 打印 APP_NAME 的值
```

### 4. 从 Reader 加载环境变量

你还可以从 `io.Reader`（如字符串、文件）加载环境变量：

```go
import (
	"strings"
	"github.com/joho/godotenv"
)

func main() {
	envString := "APP_NAME=MyApp\nAPP_ENV=production"
	r := strings.NewReader(envString)
	godotenv.Load(r)
	fmt.Println(os.Getenv("APP_NAME")) // 输出：MyApp
}
```

### 5. 将环境变量写入 `.env` 文件

可以使用 `godotenv.Write` 方法将一个 map 写入到 `.env` 文件中：

```go
import (
	"github.com/joho/godotenv"
	"os"
)

func main() {
	envMap := map[string]string{
		"APP_NAME": "MyApp",
		"APP_ENV":  "production",
		"DB_HOST":  "localhost",
	}

	err := godotenv.Write(envMap, ".env.new")
	if err != nil {
		fmt.Println("Error writing .env file:", err)
	} else {
		fmt.Println("Successfully wrote .env.new file")
	}
}
```

### 6. 自动检测 `.env` 文件

`godotenv` 提供了 `godotenv.Overload` 方法，它会自动加载项目目录中所有找到的 `.env` 文件，并且每个文件的变量都会覆盖前一个文件中相同的变量：

```go
err := godotenv.Overload()
if err != nil {
	log.Fatalf("Error loading .env files: %v", err)
}
```

## 总结

- `godotenv` 是一个非常简单、轻量级的包，用于加载和解析 `.env` 文件，并将其中的内容导入环境变量。
- 使用 `.env` 文件可以轻松管理项目的配置信息，并支持不同的环境配置（如开发、测试、生产）。
- 了解如何加载 `.env` 文件、指定路径、加载多个文件以及将环境变量写入 `.env` 文件等操作，可以帮助你更好地管理和使用环境变量。