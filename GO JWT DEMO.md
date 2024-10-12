## GO JWT DEMO

[TOC]

**JSON Web Token (JWT)** 是一种基于令牌的认证方法，在客户端生成一个加密的令牌来存储用户信息，并在每次请求时将该令牌发送给服务器进行验证。JWT 的广泛应用场景包括用户认证、授权、信息交换等。以下是 JWT 的使用方法和常见场景：

### JWT 使用流程

1. **用户登录**
   - 用户通过登录页面提交用户名和密码。
   - 服务器验证用户凭证（通常是查询数据库中的用户信息）。

2. **生成 JWT**
   - 服务器验证用户身份成功后，生成 JWT 并将其返回给客户端。
   - JWT 包含用户信息以及签名，签名由服务器使用秘密密钥生成，确保令牌的完整性和真实性。

3. **发送 JWT**
   - 客户端将 JWT 存储在浏览器的本地存储或会话存储中，或者作为 Cookie 存储。
   - 客户端在每次请求时将 JWT 放在请求头部的 `Authorization` 字段中发送给服务器。

4. **验证 JWT**
   - 服务器从请求中获取 JWT，并使用相同的密钥验证其签名，确保令牌未被篡改。
   - 解析 JWT 获取其中的用户信息，并根据需要执行后续操作（例如，授权访问资源）。

5. **访问受限资源**
   - 服务器验证通过后，允许用户访问受限资源。

### JWT 结构

JWT 由三部分组成，每部分使用点 (`.`) 分隔：
- **Header**: 包含令牌的元数据，如类型（JWT）和加密算法（如 HMAC SHA256）。
- **Payload**: 包含声明（claims），通常是用户信息，如用户 ID、用户名、以及令牌的过期时间。
- **Signature**: 由 Header 和 Payload 结合服务器密钥生成，用于验证令牌的真实性和完整性。

JWT 格式示例：
```plaintext
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsIm5hbWUiOiJKb2huIERvZSIsImV4cCI6MTYyNTIzMzYwMH0.abc123signature
```

### 使用 JWT 的场景

1. **用户认证和授权**
   - **单点登录 (SSO)**: JWT 非常适合单点登录，用户登录一次即可访问多个应用程序或服务。
   - **API 认证**: 在微服务架构中，JWT 可以用来在服务之间传递认证信息，确保请求者有权访问特定的服务。

2. **移动和单页应用（SPA）**
   - 在移动应用和单页应用中，使用 JWT 作为身份验证令牌，可以在客户端存储令牌（如在 `localStorage` 中）并在后续请求中使用。
   - 减少了服务器存储会话数据的需求，降低了服务器的负载。

3. **信息交换**
   - JWT 可以在不同的服务或应用之间安全地传递信息，因为它是自包含的，并且可以被签名和加密。
   - 可以用于传递非敏感的用户信息（如用户 ID、权限等）。

4. **访问控制**
   - 在资源访问控制中，可以使用 JWT 的声明（claims）来包含用户的角色和权限，服务器根据这些信息决定是否允许访问特定资源。

### JWT 实现示例（Go 语言）

以下是如何在 Go 中使用 `github.com/golang-jwt/jwt` 库实现 JWT 认证的示例：

#### 1. 生成 JWT

```go
import (
    "github.com/golang-jwt/jwt"
    "time"
)

var jwtKey = []byte("your_secret_key")

func GenerateJWT(username string) (string, error) {
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "username": username,
        "exp":      time.Now().Add(time.Hour * 72).Unix(), // 令牌过期时间
    })

    tokenString, err := token.SignedString(jwtKey)
    if err != nil {
        return "", err
    }
    return tokenString, nil
}
```

#### 2. 验证 JWT

```go
import (
    "github.com/golang-jwt/jwt"
    "net/http"
)

func AuthenticateJWT(tokenString string) (*jwt.Token, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        return jwtKey, nil
    })
    if err != nil || !token.Valid {
        return nil, err
    }
    return token, nil
}

func JWTMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        tokenString := r.Header.Get("Authorization")
        if tokenString == "" {
            http.Error(w, "Missing Authorization header", http.StatusUnauthorized)
            return
        }

        _, err := AuthenticateJWT(tokenString)
        if err != nil {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```

#### 3. 在 Gin 中使用

```go
import (
    "github.com/gin-gonic/gin"
    "net/http"
)

func JWTAuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        tokenString := c.GetHeader("Authorization")
        if tokenString == "" {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "Missing Authorization header"})
            c.Abort()
            return
        }

        _, err := AuthenticateJWT(tokenString)
        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
            c.Abort()
            return
        }

        c.Next()
    }
}

func main() {
    router := gin.Default()
    router.Use(JWTAuthMiddleware())

    router.GET("/protected", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "This is a protected route"})
    })

    router.Run(":8080")
}
```

### 注意事项
- **安全性**: 确保在生产环境中使用足够复杂的密钥 (`jwtKey`)。避免在令牌中存储敏感信息，因为它可以被解码。
- **令牌过期**: 确保设置令牌的过期时间，并在服务器上验证令牌是否已过期。
- **存储**: 在客户端存储 JWT 时，请注意安全风险。避免将 JWT 存储在不安全的地方（如浏览器的 `localStorage`）中，可以考虑使用安全的 HTTP-only cookies。

JWT 提供了一种简单而强大的方式来处理用户认证和授权，但需要小心处理安全细节，确保系统的安全性。