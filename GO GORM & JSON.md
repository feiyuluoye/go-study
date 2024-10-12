## GO GORM & JSON

[TOC]

GORM 是一个在 Go 语言中广泛使用的 ORM（对象关系映射）库，提供了对数据库的操作和管理，使得开发者可以更方便地进行数据库操作。GORM 支持主流的数据库，如 MySQL、PostgreSQL、SQLite 和 SQL Server。

### GORM 安装

首先，你需要安装 GORM 和数据库驱动，例如 MySQL 驱动：

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

### 基本使用

#### 1. **初始化数据库连接**

在使用 GORM 前，你需要初始化一个数据库连接：

```go
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
}
```

- `dsn`：数据源名称，包括用户名、密码、主机、数据库名等信息。
- `gorm.Open`：使用 GORM 打开一个数据库连接。

#### 2. **定义模型**

在 GORM 中，结构体映射到数据库表，结构体字段映射到表的列：

```go
type User struct {
    ID       uint   `gorm:"primaryKey"`
    Name     string `gorm:"size:100"`
    Email    string `gorm:"unique"`
    Age      int
    Birthday time.Time
}
```

- `gorm:"primaryKey"`：指定字段为主键。
- `gorm:"size:100"`：指定字段长度。
- `gorm:"unique"`：指定字段唯一。

#### 3. **自动迁移**

GORM 可以根据模型自动创建或更新数据库表：

```go
err := db.AutoMigrate(&User{})
if err != nil {
    log.Fatal(err)
}
```

- `AutoMigrate`：自动迁移数据库结构，会创建表、添加字段、索引等，不会删除或改变现有数据。

#### 4. **基本 CRUD 操作**

##### 创建记录

```go
user := User{Name: "John Doe", Email: "john@example.com", Age: 30}
result := db.Create(&user)
if result.Error != nil {
    log.Fatal(result.Error)
}
```

- `db.Create`：插入一条记录到数据库。
- `result.RowsAffected`：插入的行数。
- `result.Error`：插入操作的错误。

##### 查询记录

```go
var user User
db.First(&user, 1) // 根据主键查询
db.First(&user, "email = ?", "john@example.com") // 条件查询
```

- `db.First`：查询第一条匹配的记录。

##### 更新记录

```go
db.Model(&user).Update("Age", 31)
db.Model(&user).Updates(User{Name: "Jane Doe", Age: 32}) // 使用结构体更新
db.Model(&user).Updates(map[string]interface{}{"Name": "Jane Doe", "Age": 32}) // 使用 map 更新
```

- `db.Model`：指定需要更新的模型。

##### 删除记录

```go
db.Delete(&user)
db.Delete(&User{}, 1) // 根据主键删除
```

- `db.Delete`：删除记录。

### 高级功能

#### 1. **查询条件**

GORM 支持丰富的查询条件：

- **Where 查询**：
    ```go
    var users []User
    db.Where("name = ?", "John").Find(&users)
    db.Where("age > ?", 25).Find(&users)
    ```

- **链式查询**：
    ```go
    db.Where("name = ?", "John").Or("name = ?", "Jane").Find(&users)
    ```

- **选择特定字段**：
    ```go
    db.Select("name", "age").Find(&users)
    ```

#### 2. **关系**

GORM 支持定义和操作一对一、一对多、多对多的关系：

- **一对多**：
    ```go
    type User struct {
        gorm.Model
        Name    string
        Articles []Article
    }
    
    type Article struct {
        gorm.Model
        Title  string
        UserID uint
    }
    
    // 查询用户及其相关文章
    var user User
    db.Preload("Articles").Find(&user)
    ```

- **多对多**：
    ```go
    type User struct {
        gorm.Model
        Name     string
        Languages []Language `gorm:"many2many:user_languages;"`
    }
    
    type Language struct {
        gorm.Model
        Name string
    }
    
    // 查询用户及其会的语言
    var user User
    db.Preload("Languages").Find(&user)
    ```

#### 3. **事务**

GORM 支持事务管理：

```go
func TransactionExample(db *gorm.DB) error {
    return db.Transaction(func(tx *gorm.DB) error {
        if err := tx.Create(&User{Name: "Alice"}).Error; err != nil {
            return err
        }

        if err := tx.Create(&User{Name: "Bob"}).Error; err != nil {
            return err
        }

        return nil
    })
}
```

- `db.Transaction`：运行一个事务，确保所有操作在同一个事务中。

#### 4. **钩子**

GORM 提供模型钩子，可以在模型生命周期的不同阶段执行代码，例如 `BeforeCreate`、`AfterCreate`、`BeforeUpdate`、`AfterUpdate` 等。

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
    u.Name = strings.ToUpper(u.Name)
    return
}
```

将 GORM 与 JSON 结合使用，可以方便地在 Web 应用中处理数据。例如，使用 GORM 从数据库中查询数据并将其转换为 JSON 格式返回给客户端，或者从客户端接收 JSON 数据并保存到数据库中。以下是如何在 Go 中使用 GORM 和 JSON，以及 PostgreSQL 数据库驱动的示例：

#### 安装 PostgreSQL 驱动

首先，安装 GORM 和 PostgreSQL 驱动：

```bash
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

#### 定义模型

在模型结构体中添加 JSON 标签，使其在序列化和反序列化时正确映射字段名称：

```go
package main

import (
    "time"
    "gorm.io/gorm"
)

type User struct {
    ID        uint      `gorm:"primaryKey" json:"id"`
    Name      string    `gorm:"size:100" json:"name"`
    Email     string    `gorm:"unique" json:"email"`
    Age       int       `json:"age"`
    Birthday  time.Time `json:"birthday"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}
```

- `gorm:"primaryKey"`：指定字段为主键。
- `json:"字段名"`：指定 JSON 中对应的字段名称。

#### 初始化数据库连接

连接到 PostgreSQL 数据库：

```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "log"
)

func main() {
    dsn := "host=localhost user=your_user password=your_password dbname=your_db port=5432 sslmode=disable TimeZone=Asia/Shanghai"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }

    // 自动迁移
    db.AutoMigrate(&User{})
}
```

- `dsn`：数据源名称，包括主机、用户名、密码、数据库名等信息。

#### 处理 JSON 数据

##### 创建记录

从 JSON 数据创建新记录：

```go
import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func CreateUser(c *gin.Context) {
    var user User
    // 绑定 JSON 到结构体
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    result := db.Create(&user)
    if result.Error != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": result.Error.Error()})
        return
    }

    c.JSON(http.StatusOK, user)
}
```

- `c.ShouldBindJSON(&user)`：将 JSON 数据绑定到 `User` 结构体。
- `db.Create(&user)`：插入记录到数据库。

##### 查询记录并返回 JSON

查询数据库并将结果返回为 JSON：

```go
func GetUser(c *gin.Context) {
    var user User
    id := c.Param("id")
    result := db.First(&user, id)
    if result.Error != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    c.JSON(http.StatusOK, user)
}
```

- `c.Param("id")`：获取 URL 参数。
- `db.First(&user, id)`：根据主键查询记录。
- `c.JSON(http.StatusOK, user)`：返回 JSON 响应。

##### 更新记录

从 JSON 数据更新记录：

```go
func UpdateUser(c *gin.Context) {
    var user User
    id := c.Param("id")

    // 查询用户
    if err := db.First(&user, id).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    // 绑定 JSON 到结构体
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // 保存更新
    if err := db.Save(&user).Error; err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusOK, user)
}
```

##### 删除记录

删除数据库中的记录：

```go
func DeleteUser(c *gin.Context) {
    var user User
    id := c.Param("id")

    if err := db.First(&user, id).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
        return
    }

    db.Delete(&user)

    c.JSON(http.StatusOK, gin.H{"message": "User deleted"})
}
```

#### 在 Gin 中使用 GORM 和 JSON

将以上函数集成到 Gin 路由中：

```go
func main() {
    r := gin.Default()

    // 初始化数据库连接
    dsn := "host=localhost user=your_user password=your_password dbname=your_db port=5432 sslmode=disable TimeZone=Asia/Shanghai"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal(err)
    }
    db.AutoMigrate(&User{})

    r.POST("/users", CreateUser)
    r.GET("/users/:id", GetUser)
    r.PUT("/users/:id", UpdateUser)
    r.DELETE("/users/:id", DeleteUser)

    r.Run(":8080")
}
```

### 注意事项

- **性能优化**：虽然 GORM 提供了方便的 ORM 功能，但它的性能可能不如原生 SQL。在需要高性能的场景下，尽量优化查询，避免不必要的复杂查询。
- **版本**：GORM 目前的最新版本是 v2，与 v1 有一些不兼容的变化。在使用前，请确保查阅相应版本的文档。
- **错误处理**：每个数据库操作都会返回 `*gorm.DB`，它包含 `Error` 属性，务必检查并处理这些错误。

- **字段映射**：GORM 默认使用结构体字段名作为数据库列名，可以通过 `gorm:"column:column_name"` 指定自定义列名。
- **JSON 序列化**：确保 JSON 标签与前端传递的字段名称一致，以正确地进行绑定和序列化。
- **安全性**：处理用户输入时，要验证和处理错误，防止 SQL 注入和其他安全问题。
- **数据库连接池**：在生产环境中，可以配置数据库连接池，以提高性能并避免资源泄漏。

### 练习与学习资源

- **官方文档**：GORM 的官方文档非常详细，提供了丰富的示例和指南：[GORM 官方文档](https://gorm.io/docs/)
- **社区教程**：网上有许多关于 GORM 的教程和示例，可以帮助你快速入门。
- **实战项目**：将 GORM 应用到实际项目中，比如开发一个简单的 RESTful API，可以帮助加深理解和掌握。