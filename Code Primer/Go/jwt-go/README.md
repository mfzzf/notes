`jwt-go` 是一个用于生成、解析和验证 JSON Web Token (JWT) 的 Go 语言库。JWT 是一种用于安全传递声明的开放标准（RFC 7519），通常用于身份验证和信息交换。`jwt-go` 提供了一些简便的工具来生成和解析 JWT。

### 安装

你可以通过以下命令安装 `jwt-go`：

```bash
go get github.com/dgrijalva/jwt-go
```

### 使用示例

#### 1. 生成 JWT

首先，导入 `jwt-go` 包，并创建一个 JWT。

```go
package main

import (
	"fmt"
	"time"
	"github.com/dgrijalva/jwt-go"
)

func main() {
	// 创建一个新的 JWT Token
	// 定义签名密钥
	var secretKey = []byte("mysecretkey")

	// 创建一个自定义的声明
	claims := jwt.MapClaims{
		"sub": "1234567890", // 用户ID
		"name": "John Doe",  // 用户名
		"iat": time.Now().Unix(), // 签发时间
	}

	// 使用 HMAC 签名方法生成 token
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	// 签名并获得完整的 token 字符串
	tokenString, err := token.SignedString(secretKey)
	if err != nil {
		fmt.Println("Error generating token:", err)
		return
	}

	fmt.Println("Generated Token:", tokenString)
}
```

#### 2. 解析和验证 JWT

解析和验证 JWT 主要是通过 `jwt.Parse` 函数，使用签名密钥来验证令牌是否合法。

```go
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go"
	"strings"
)

func main() {
	// 假设你已经得到了一个有效的 token 字符串
	tokenString := "your-jwt-token-string-here"

	// 定义密钥
	var secretKey = []byte("mysecretkey")

	// 解析 token
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		// 确保签名方法正确
		if _, ok := token.Method.(*jwt.SigningMethodHS256); !ok {
			return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
		}
		return secretKey, nil
	})

	if err != nil {
		fmt.Println("Error parsing token:", err)
		return
	}

	// 校验 token 是否有效
	if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
		// 如果有效，可以在这里使用 claims
		fmt.Println("Token is valid!")
		fmt.Println("Claims:", claims)
	} else {
		fmt.Println("Invalid token")
	}
}
```

#### 3. 使用 JWT 存储额外信息

除了存储标准的声明（如 `sub`, `iat`, `exp` 等），你还可以在 JWT 中添加额外的自定义信息：

```go
claims := jwt.MapClaims{
	"sub":   "1234567890",
	"name":  "John Doe",
	"role":  "admin", // 自定义字段
	"email": "johndoe@example.com",
	"iat":   time.Now().Unix(),
}

token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
tokenString, err := token.SignedString(secretKey)
```

在解析时，`claims` 中将会包含 `role` 和 `email` 等自定义字段。

### 总结

- `jwt-go` 提供了简单的方法来生成、解析和验证 JWT。
- JWT 使用 `Header`, `Payload`, 和 `Signature` 组成，其中 `Payload` 可以包含自定义信息。
- 生成 JWT 时，通常使用签名密钥（`HS256`、`RS256` 等）来保证数据的完整性。
- 解析 JWT 时需要确保签名有效，可以通过提供密钥来验证。

JWT 通常用于用户身份验证，确保信息在客户端和服务器之间安全传输。