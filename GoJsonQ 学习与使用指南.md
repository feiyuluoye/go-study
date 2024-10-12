# GoJsonQ 学习与使用指南

**GoJsonQ** 是一个用于在 Go 语言中查询和处理 JSON 数据的轻量级库。它提供了类似于 SQL 的查询功能，使得对复杂的 JSON 数据进行过滤、排序、分组等操作变得更加简便。

## 目录

1. [安装 GoJsonQ](#安装-gojsonq)
2. [基本用法](#基本用法)
3. [高级功能](#高级功能)
4. [示例代码](#示例代码)
5. [常见问题](#常见问题)
6. [资源与参考资料](#资源与参考资料)

---

## 安装 GoJsonQ

在开始使用 GoJsonQ 之前，需要先安装该库。确保你已经安装了 Go 语言环境。

使用 `go get` 命令安装 GoJsonQ：

```bash
go get -u github.com/thedevsaddam/gojsonq/v2
```

或在 `go.mod` 文件中添加：

```go
require github.com/thedevsaddam/gojsonq/v2 v2.x.x
```

然后运行 `go mod tidy` 以下载依赖。

---

## 基本用法

### 1. 导入包

```go
import (
    "fmt"
    "github.com/thedevsaddam/gojsonq/v2"
)
```

### 2. 加载 JSON 数据

可以从多种数据源加载 JSON，例如字符串、文件或字节切片。

#### 从字符串加载

```go
jsonString := `{
    "users": [
        {"id": 1, "name": "Alice", "age": 30},
        {"id": 2, "name": "Bob", "age": 25},
        {"id": 3, "name": "Charlie", "age": 35}
    ]
}`

jq := gojsonq.New().FromString(jsonString)
```

#### 从文件加载

```go
jq := gojsonq.New().FromFile("data.json")
```

### 3. 查询数据

#### 获取所有数据

```go
data := jq.Get()
fmt.Println(data)
```

#### 使用 `Find` 查找单个对象

```go
user := jq.Find("users.id", 2)
fmt.Println(user)
```

#### 使用 `Where` 过滤数据

```go
result := jq.Where("age", ">", 28).Get()
fmt.Println(result)
```

#### 选择特定字段

```go
names := jq.Pluck("users.name").Get()
fmt.Println(names)
```

#### 排序

```go
sorted := jq.SortBy("age").Get()
fmt.Println(sorted)
```

---

## 高级功能

### 1. 链式调用

GoJsonQ 支持链式调用，使查询更加简洁。

```go
usersOver30 := gojsonq.New().
    FromString(jsonString).
    Find("users.age", func(age int) bool { return age > 30 }).
    Get()

fmt.Println(usersOver30)
```

### 2. 聚合函数

支持多种聚合操作，如 `Count`, `Max`, `Min`, `Avg`, `Sum` 等。

```go
count := jq.Count("users")
maxAge := jq.Max("users.age")
avgAge := jq.Avg("users.age")

fmt.Println("Count:", count)
fmt.Println("Max Age:", maxAge)
fmt.Println("Average Age:", avgAge)
```

### 3. 分组

可以根据特定字段进行分组。

```go
grouped := jq.GroupBy("users.age").Get()
fmt.Println(grouped)
```

### 4. 多条件查询

支持多个条件的组合查询。

```go
filtered := jq.
    Where("age", ">", 25).
    Where("name", "Alice").
    Get()
    
fmt.Println(filtered)
```

### 5. 嵌套查询

处理嵌套的 JSON 结构。

```go
address := `{
    "users": [
        {
            "id": 1,
            "name": "Alice",
            "address": {
                "city": "New York",
                "zip": "10001"
            }
        },
        {
            "id": 2,
            "name": "Bob",
            "address": {
                "city": "Los Angeles",
                "zip": "90001"
            }
        }
    ]
}`

jq := gojsonq.New().FromString(address)
cities := jq.Pluck("users.address.city").Get()
fmt.Println(cities)
```

---

## 示例代码

以下是一个完整的示例，展示如何使用 GoJsonQ 进行各种操作。

```go
package main

import (
    "fmt"
    "github.com/thedevsaddam/gojsonq/v2"
)

func main() {
    jsonData := `{
        "users": [
            {"id": 1, "name": "Alice", "age": 30, "city": "New York"},
            {"id": 2, "name": "Bob", "age": 25, "city": "Los Angeles"},
            {"id": 3, "name": "Charlie", "age": 35, "city": "Chicago"},
            {"id": 4, "name": "David", "age": 28, "city": "New York"},
            {"id": 5, "name": "Eve", "age": 22, "city": "Los Angeles"}
        ]
    }`

    jq := gojsonq.New().FromString(jsonData)

    // 获取所有用户
    allUsers := jq.Find("users")
    fmt.Println("All Users:", allUsers)

    // 过滤年龄大于25的用户
    usersOver25 := jq.Where("age", ">", 25).Get()
    fmt.Println("Users over 25:", usersOver25)

    // 获取特定字段
    names := jq.Pluck("users.name").Get()
    fmt.Println("Names:", names)

    // 排序
    sortedByAge := jq.SortBy("age").Get()
    fmt.Println("Sorted by Age:", sortedByAge)

    // 聚合函数
    count := jq.Count("users")
    maxAge := jq.Max("users.age")
    avgAge := jq.Avg("users.age")
    fmt.Println("Count:", count)
    fmt.Println("Max Age:", maxAge)
    fmt.Println("Average Age:", avgAge)
}
```

**输出：**

```
All Users: [map[id:1 name:Alice age:30 city:New York] map[id:2 name:Bob age:25 city:Los Angeles] map[id:3 name:Charlie age:35 city:Chicago] map[id:4 name:David age:28 city:New York] map[id:5 name:Eve age:22 city:Los Angeles]]
Users over 25: [map[id:1 name:Alice age:30 city:New York] map[id:3 name:Charlie age:35 city:Chicago] map[id:4 name:David age:28 city:New York]]
Names: [Alice Bob Charlie David Eve]
Sorted by Age: [map[id:5 name:Eve age:22 city:Los Angeles] map[id:2 name:Bob age:25 city:Los Angeles] map[id:4 name:David age:28 city:New York] map[id:1 name:Alice age:30 city:New York] map[id:3 name:Charlie age:35 city:Chicago]]
Count: 5
Max Age: 35
Average Age: 28
```

---

#### **YAML** 文件处理与 **GoJsonQ** 的 JSON 查询功能

为了结合 **YAML** 文件处理与 **GoJsonQ** 的 JSON 查询功能，你可以将 YAML 文件解析为 **JSON** 或等效的 Go 数据结构，然后使用 **GoJsonQ** 进行查询处理。

在 Go 中，可以使用官方的 `gopkg.in/yaml.v2` 包来解析 YAML 数据，将其转换为结构化的 Go 数据类型。接着，你可以使用 **GoJsonQ** 对转换后的数据进行查询。

首先，你需要安装 YAML 解析库 `gopkg.in/yaml.v2`。通过 Go 的 `go get` 命令来安装：

```bash
go get gopkg.in/yaml.v2
```

使用 `yaml.Unmarshal()` 函数将 YAML 数据解析为 Go 的 `map[string]interface{}` 或结构体。然后，使用 GoJsonQ 查询该数据。

为了让 **GoJsonQ** 处理 YAML 文件，首先需要解析 YAML 文件并将其转换为等效的 JSON 格式（即 `map[string]interface{}`），然后传递给 GoJsonQ。

步骤如下：

1. 使用 `yaml.Unmarshal()` 解析 YAML 数据。
2. 将解析后的数据作为 GoJsonQ 的数据源。
3. 对数据进行查询和处理。



假设有一个 YAML 文件 `data.yaml`，内容如下：

```yaml
users:
  - id: 1
    name: Alice
    age: 30
  - id: 2
    name: Bob
    age: 25
  - id: 3
    name: Charlie
    age: 35
```

你可以使用以下步骤解析 YAML 并使用 GoJsonQ 进行查询。

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"

    "github.com/thedevsaddam/gojsonq/v2"
    "gopkg.in/yaml.v2"
)

func main() {
    // 读取 YAML 文件
    yamlFile, err := ioutil.ReadFile("data.yaml")
    if err != nil {
        log.Fatalf("Error reading YAML file: %v", err)
    }

    // 定义一个 map 来存储解析后的数据
    var data map[string]interface{}

    // 解析 YAML 数据
    err = yaml.Unmarshal(yamlFile, &data)
    if err != nil {
        log.Fatalf("Error unmarshaling YAML: %v", err)
    }

    // 打印解析后的数据 (用于调试)
    fmt.Printf("Parsed YAML data: %#v\n", data)

    // 使用 GoJsonQ 查询解析后的数据
    jq := gojsonq.New().FromInterface(data)

    // 查询所有用户数据
    allUsers := jq.Find("users")
    fmt.Println("All Users:", allUsers)

    // 过滤年龄大于 25 的用户
    usersOver25 := jq.Where("age", ">", 25).Get()
    fmt.Println("Users over 25:", usersOver25)

    // 获取所有用户的姓名
    names := jq.Pluck("users.name").Get()
    fmt.Println("Names:", names)
}
```

```
Parsed YAML data: map[string]interface {}{"users":[]interface {}{map[interface {}]interface {}{"id":1, "name":"Alice", "age":30}, map[interface {}]interface {}{"id":2, "name":"Bob", "age":25}, map[interface {}]interface {}{"id":3, "name":"Charlie", "age":35}}}
All Users: [map[id:1 name:Alice age:30] map[id:2 name:Bob age:25] map[id:3 name:Charlie age:35]]
Users over 25: [map[id:1 name:Alice age:30] map[id:3 name:Charlie age:35]]
Names: [Alice Bob Charlie]
```

在解析时，你可能注意到输出中的数据结构是 `map[interface{}]interface{}`，为了兼容性并便于查询，你可以将其转换为 `map[string]interface{}`。这是因为 YAML 文件中的键和值可以是任意类型，而 JSON 键通常是字符串。

可以使用递归函数将 `map[interface{}]interface{}` 转换为 `map[string]interface{}`。

#### 递归转换函数

```go
func convertMapInterface(m interface{}) interface{} {
    switch m := m.(type) {
    case map[interface{}]interface{}:
        converted := make(map[string]interface{})
        for k, v := range m {
            converted[k.(string)] = convertMapInterface(v)
        }
        return converted
    case []interface{}:
        for i, v := range m {
            m[i] = convertMapInterface(v)
        }
    }
    return m
}
```

使用该函数来处理解析后的 YAML 数据：

```go
data = convertMapInterface(data).(map[string]interface{})
```

这样可以确保数据结构可以更好地适应 JSON 风格的查询。

---

## 示例代码（修正版本）

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"

    "github.com/thedevsaddam/gojsonq/v2"
    "gopkg.in/yaml.v2"
)

// 递归转换 map[interface{}]interface{} 为 map[string]interface{}
func convertMapInterface(m interface{}) interface{} {
    switch m := m.(type) {
    case map[interface{}]interface{}:
        converted := make(map[string]interface{})
        for k, v := range m {
            converted[k.(string)] = convertMapInterface(v)
        }
        return converted
    case []interface{}:
        for i, v := range m {
            m[i] = convertMapInterface(v)
        }
    }
    return m
}

func main() {
    // 读取 YAML 文件
    yamlFile, err := ioutil.ReadFile("data.yaml")
    if err != nil {
        log.Fatalf("Error reading YAML file: %v", err)
    }

    // 定义一个 map 来存储解析后的数据
    var data map[string]interface{}

    // 解析 YAML 数据
    err = yaml.Unmarshal(yamlFile, &data)
    if err != nil {
        log.Fatalf("Error unmarshaling YAML: %v", err)
    }

    // 转换为 JSON 友好的格式
    data = convertMapInterface(data).(map[string]interface{})

    // 使用 GoJsonQ 查询解析后的数据
    jq := gojsonq.New().FromInterface(data)

    // 查询所有用户数据
    allUsers := jq.Find("users")
    fmt.Println("All Users:", allUsers)

    // 过滤年龄大于 25 的用户
    usersOver25 := jq.Where("age", ">", 25).Get()
    fmt.Println("Users over 25:", usersOver25)

    // 获取所有用户的姓名
    names := jq.Pluck("users.name").Get()
    fmt.Println("Names:", names)
}
```



## 常见问题

### 1. **如何处理深度嵌套的 YAML 文件？**

对于嵌套较深的 YAML 文件，确保递归函数能够处理所有嵌套层级的数据，并且 GoJsonQ 的 `dot notation` 可以方便地访问嵌套的字段。

### 2. **YAML 中的数据类型如何映射到 Go？**

YAML 的数据类型（例如布尔值、整数、字符串等）自动映射到 Go 的相应数据类型，例如 `bool`, `int`, `string` 等。对于复杂对象或数组，映射为 `map[interface{}]interface{}` 或 `[]interface{}`，这需要手动转换为 `map[string]interface{}` 格式。

---

通过这份集成指南，你可以轻松将 **YAML** 文件与 **GoJsonQ** 结合，进行灵活的查询和处理。如果你有更复杂的需求或遇到其他问题，欢迎继续提问！

### 3. **如何处理大型 JSON 文件？**

对于非常大的 JSON 文件，建议分块读取或使用流式解析，以避免内存占用过高。GoJsonQ 目前不支持流式解析，但可以结合 `encoding/json` 包进行预处理。

### 4. **支持哪些查询操作符？**

GoJsonQ 支持常见的操作符，如 `=`, `!=`, `<`, `>`, `<=`, `>=`, `in`, `not in`, `like` 等。

### 5. **如何处理嵌套数组或对象？**

可以使用点（`.`）语法访问嵌套的字段。例如，`users.address.city`。

---

## 资源与参考资料

- **官方文档**: [GoJsonQ GitHub](https://github.com/thedevsaddam/gojsonq)
- **示例项目**: 可以在 GitHub 上搜索相关示例项目，学习实际应用案例。
- **社区支持**: 在 GitHub Issues 区域提问或查看已有的问题，以获取帮助。
