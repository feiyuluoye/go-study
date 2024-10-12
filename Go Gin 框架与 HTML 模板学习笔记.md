## Go Gin 框架与 HTML 模板学习笔记

[TOC]



### 1. 简介

`Gin` 是 Go 语言中常用的高性能轻量级 HTTP Web 框架，适合快速开发 RESTful API 和 Web 应用。Gin 支持 HTML 模板渲染，基于 Go 标准库 `html/template`，提供了安全、高效的模板功能。本文主要介绍如何在 `Gin` 中使用 HTML 模板。

---

### 2. 安装与项目结构
首先，确保已安装 Go 语言环境，然后安装 `Gin` 框架：

```bash
go get -u github.com/gin-gonic/gin
```

#### 项目结构：
```
go-gin-template/
    ├── main.go
    └── templates/
        └── index.html
```

---

### 3. 基本使用

#### 3.1 创建基本的 HTTP 服务
```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default() // 创建Gin路由引擎

    // 加载模板文件夹中的所有HTML文件
    r.LoadHTMLGlob("templates/*")

    // 定义一个简单的处理器
    r.GET("/", func(c *gin.Context) {
        data := gin.H{
            "Title":   "Gin Template Example",
            "Message": "Welcome to Go Gin Templates!",
        }
        c.HTML(200, "index.html", data) // 渲染模板并输出到客户端
    })

    r.Run(":8080") // 启动HTTP服务
}
```

#### 3.2 创建模板文件
在 `templates` 目录下创建一个 `index.html` 文件：

**`templates/index.html`:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{.Title}}</title>
</head>
<body>
    <h1>{{.Message}}</h1>
</body>
</html>
```

#### 3.3 运行程序
```bash
go run main.go
```
浏览器访问 `http://localhost:8080`，你将看到页面渲染的内容。

---

### 4. Gin 模板渲染详解

- **`LoadHTMLGlob(pattern string)`**：使用文件匹配模式加载模板文件，支持批量加载，例如 `r.LoadHTMLGlob("templates/*")`。
  
- **`LoadHTMLFiles(filenames ...string)`**：手动指定模板文件路径加载多个模板文件。

- **`c.HTML(httpStatus int, name string, obj interface{})`**：渲染HTML模板，`name` 为模板文件名，`obj` 为传递的数据。

#### 4.1 传递数据
Gin 提供 `gin.H{}`（类似于 `map[string]interface{}`）来传递数据至模板：
```go
data := gin.H{
    "Title":   "Page Title",
    "Message": "Hello, World!",
}
```

模板中使用 `{{.Key}}` 来引用数据：
```html
<title>{{.Title}}</title>
<h1>{{.Message}}</h1>
```

---

### 5. 处理复杂数据结构

除了简单的数据类型，Gin模板支持传递数组、切片、结构体等复杂数据。

#### 5.1 传递切片数据
```go
r.GET("/list", func(c *gin.Context) {
    items := []string{"Item 1", "Item 2", "Item 3"}
    c.HTML(200, "list.html", gin.H{
        "Items": items,
    })
})
```

**模板文件 `list.html`:**
```html
<ul>
    {{range .Items}}
    <li>{{.}}</li>
    {{end}}
</ul>
```

---

### 6. 静态文件服务

Gin 提供了处理静态文件的简单方法。可以通过 `Static()` 方法提供静态文件目录，比如 CSS、JS 或图片。

```go
r.Static("/assets", "./assets")
```

这样，`/assets` 路径下的文件可以通过浏览器访问，例如 `http://localhost:8080/assets/style.css`。

---

### 7. 模板的高级特性

`html/template` 是 Go 的标准库，提供了灵活的模板语法。常用特性如下：

#### 7.1 条件判断
```html
{{if .Condition}}
    <p>Condition is true</p>
{{else}}
    <p>Condition is false</p>
{{end}}
```

#### 7.2 循环语句
```html
<ul>
    {{range .Items}}
    <li>{{.}}</li>
    {{end}}
</ul>
```

#### 7.3 自定义模板函数
Gin允许你为模板注册自定义函数，来格式化或处理数据。

```go
r.SetFuncMap(template.FuncMap{
    "upper": strings.ToUpper, // 注册自定义函数
})

r.LoadHTMLGlob("templates/*")
```

在模板中使用自定义函数：
```html
<p>{{upper .Message}}</p>
```

---

### 8. 总结

通过 Gin 框架，你可以高效地构建 Web 应用，同时借助 Go 语言的 `html/template` 来安全、灵活地渲染HTML页面。Gin 支持静态文件服务、复杂数据传递、自定义函数等高级功能，非常适合现代 Web 应用的开发。

### 主要学习要点：
- **Gin 路由与 HTML 模板集成**：掌握如何使用 Gin 渲染 HTML 模板。
- **模板语法**：学习 `html/template` 的基本语法，如变量插值、条件判断、循环等。
- **静态文件处理**：理解 Gin 如何提供静态资源服务。
- **自定义模板函数**：能够扩展模板功能，适应项目的需求。

通过以上内容，你可以开始在 `Gin` 中使用模板构建动态的 Web 页面，并逐步深入学习更多复杂功能。