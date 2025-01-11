## 安装

go get github.com/gin-gonic/gin

## gin.Default()

这行代码创建一个带有默认中间件（如日志记录和恢复中间件）的Gin引擎实例，用于处理HTTP请求。

### `GET/PUT/POST/DELETE`

```go
r.GET("/news", func(c *gin.Context) {
    c.String(200, "%v", "i'm news!")
})
r.POST("/add", func(c *gin.Context) {
    c.String(200, "%v", "i'm post!")
})
r.PUT("/update", func(c *gin.Context) {
    c.String(200, "%v", "i'm put!")
})
r.DELETE("/delete", func(c *gin.Context) {
    c.String(200, "%v", "i'm delete!")
})
```

---

### `r.GET`
```go
r.GET(path string, handler gin.HandlerFunc)
```
**说明**：定义一个处理 GET 请求的路由。

**示例**：
```go
r.GET("/hello", func(c *gin.Context) {
    c.String(200, "Hello, World!")
})
```

---

### `c.String`
```go
c.String(code int, format string, values ...interface{})
```
**说明**：返回纯文本响应，格式化输出字符串。

**示例**：
```go
r.GET("/text", func(c *gin.Context) {
    c.String(200, "Hello, %s!", "Gin")
})
```

---

### `c.JSON`
```go
c.JSON(code int, obj interface{})
```
**说明**：返回 JSON 格式的数据，自动序列化对象为 JSON。

**示例**：
```go
r.GET("/json", func(c *gin.Context) {
    c.JSON(200, gin.H{"message": "Hello, JSON!"})
})
```

### `gin.H`

`gin.H` 是一个快捷方式，用于创建 `map[string]interface{}`，方便返回动态 JSON 数据。例如：

```go
r.GET("/dynamic-json", func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"status":  "success",
		"message": "Dynamic JSON response",
	})
})
```

此方法无需预先定义结构体，适用于简单或动态的 JSON 响应。

### `c.JSONP`
```go
c.JSONP(code int, obj interface{})
```
**说明**：返回 JSONP 格式的数据，主要用于跨域请求，自动根据请求中的 `callback` 参数包装成回调函数。

**示例**：
```go
r.GET("/jsonp", func(c *gin.Context) {
    c.JSONP(200, gin.H{"message": "Hello, JSONP!"})
})
```

---

### `c.XML`
```go
c.XML(code int, obj interface{})
```
**说明**：返回 XML 格式的数据，适用于与 XML 服务的通信。

**示例**：
```go
r.GET("/xml", func(c *gin.Context) {
    c.XML(200, gin.H{"status": "success"})
})
```

### `c.Set()`

```go
c.Set(key string, value interface{})
```

用于在当前请求的上下文中存储一个键值对，可以在当前请求的生命周期中共享数据。常用于将数据传递给中间件或不同的处理函数。

## HTML模板渲染

### `r.LoadHTMLGlob`

```go
r := gin.Default()

// 加载 templates 文件夹下的所有 .html 文件作为模板
r.LoadHTMLGlob("templates/*")
```

`LoadHTMLGlob(pattern string)`：使用通配符（如 `"templates/*"`）批量加载模板文件。

`LoadHTMLFiles(files ...string)`：指定多个文件路径来加载特定的模板文件。

### `c.HTML`

```go
c.HTML(code int, name string, obj interface{})
```

**说明**：渲染 HTML 模板并返回，用于生成动态网页内容。

**示例**：

```go
r.LoadHTMLGlob("templates/*")

// 定义一个路由，处理根路径请求
r.GET("/", func(c *gin.Context) {
    // 定义传递给模板的数据
    data := gin.H{
        "Title":   "Hello Gin",
        "Message": "Welcome to Gin framework!",
    }
    // 渲染模板并返回
    c.HTML(200, "index.html", data)
})
```

假设有一个模板文件 `index.html` 放在 `templates` 目录下，内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ .Title }}</title>
</head>
<body>
    <h1>{{ .Message }}</h1>
</body>
</html>
```

Gin 使用 `html/template` 包来渲染模板，支持 Go 的模板语法，例如：

- `{{ .FieldName }}`：访问数据的字段。
- `{{ if .Condition }}`、`{{ else }}`、`{{ end }}`：条件判断。
- `{{ range .Items }}`、`{{ end }}`：循环遍历集合。

## 模板语法

Gin 使用 `html/template` 包的模板语法，支持多种模板操作符和逻辑控制

1. 输出变量

```html
{{ .Name }} <!-- 输出变量 Name 的值 -->
```

2. 条件判断

```html
{{ if .IsAdmin }}Admin用户{{ end }} <!-- 如果 IsAdmin 为 true，则输出 Admin用户 -->
```

3. 带 `else` 的条件判断

```html
{{ if .LoggedIn }}欢迎！{{ else }}请登录{{ end }} <!-- 判断是否登录并输出不同内容 -->
```

4. `range` 循环遍历

```html
{{ range .Items }}{{ . }}{{ end }} <!-- 遍历 Items 集合并输出每个元素 -->
```

5. 带 `index` 的循环

```html
{{ range $index, $element := .Items }}{{ $index }}: {{ $element }}{{ end }} <!-- 输出每个元素和索引 -->
```

6. 访问嵌套字段

```html
{{ .User.Name }} <!-- 访问嵌套字段 User 的 Name 字段 -->
```

7. 定义和使用模板变量

```html
{{ $name := .User.Name }}你好, {{ $name }} <!-- 定义变量 $name 并使用 -->
```

8. 管道操作符 (`|`)

```html
{{ .Title | printf "%s - 官方文档" }} <!-- 通过管道将 Title 的值传入 printf 函数格式化 -->
```

9. 使用 `with` 语句

```html
{{ with .User }}用户名: {{ .Name }}{{ end }} <!-- 使用 User 作为当前上下文 -->
```

10. 转义 HTML

```html
{{ .Content | html }} <!-- 将 Content 作为 HTML 转义输出 -->
```

11. 加载子模板（定义模板块）

```html
{{ define "header" }}<header>Header 内容</header>{{ end }} <!-- 定义 header 模板块 -->
{{ template "header" }} <!-- 使用 header 模板块 -->
```

## 中间件

中间件（Middleware）是指在处理请求和响应时执行的一段逻辑代码，通常用于拦截、修改、增强请求和响应的行为。中间件主要用于进行预处理或后处理，比如身份验证、日志记录、错误处理等。

### `r.Use()`

`r.Use()` 是 Gin 框架中用于给路由添加中间件的函数。中间件是一种在处理请求前后执行的代码，可以用于记录日志、处理错误、验证权限等。

```go
r.Use(middleware1, middleware2, ...)
```

`gin.Logger()` 和 `gin.Recovery()` 是两个内置的 Gin 中间件函数。

### `gin.Logger() `

`gin.Logger()`：记录每次 HTTP 请求的信息（请求路径、状态码、响应时间等）。

### `gin.Recovery()`

`gin.Recovery()`：在程序出现 panic 时进行恢复，防止程序崩溃，返回 500 错误。

### `c.next()`

`c.Next()` 是 Gin 框架中的一个方法，用于控制中间件链中的执行流。它通常在中间件中调用，表示当前中间件执行完毕，接着执行下一个中间件或请求处理函数。

Gin 会按照中间件的**注册顺序依次执行中间件**。每个中间件可以在执行自己的逻辑后，通过 `c.Next()` 控制是否继续执行后续的中间件或路由处理。

- 当 `c.Next()` 被调用时，Gin 会继续执行后续的中间件或路由处理函数。
- 如果中间件中没有调用 `c.Next()`，后续的中间件或处理函数将不会被执行，导致请求处理流程被阻止。

### 自定义中间件

```go
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		// 给Context实例设置一个值
		c.Set("geektutu", "1111")
		// 请求前
		c.Next()
		// 请求后
		latency := time.Since(t)
		log.Print(latency)
	}
}
r.Use(Logger())
```

该中间件 `Logger()` 在请求前记录时间并设置一个键值对；在请求处理完后，计算并打印请求的处理时长。



`Gin` 是一个用 Go 语言编写的 web 框架，提供了非常高效且灵活的路由和中间件机制。`gin.Group` 和 `gin.Bind` 是 Gin 框架中常用的两个功能，分别用于路由分组和请求数据绑定。下面分别介绍它们的作用和用法。

##  `gin.Group`

`gin.Group` 用于创建一个路由分组。路由分组的作用是可以将一组有共同特征的路由进行逻辑上的组织，通常用于路由共享前缀、共享中间件等场景。

- **功能**：为一组路由添加公共的前缀或中间件，使得代码更具组织性和可读性。
- **用法**：通过 `r.Group("/prefix")` 来创建一个新的路由组，所有在这个分组下的路由都会自动带有这个前缀。

#### 示例：

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建 Gin 引擎
	r := gin.Default()

	// 创建一个路由组 /api
	apiGroup := r.Group("/api")
	{
		// 在 /api 路由下添加多个路由
		apiGroup.GET("/users", func(c *gin.Context) {
			c.JSON(200, gin.H{"message": "Get Users"})
		})
		apiGroup.POST("/users", func(c *gin.Context) {
			c.JSON(200, gin.H{"message": "Create User"})
		})
	}

	// 启动服务器
	r.Run(":8080")
}
```

在这个例子中，`/api` 是路由组的公共前缀，所有在 `apiGroup` 下定义的路由都会自动以 `/api` 开头。

##  `c.Bind`

`c.Bind` 用于将请求中的数据绑定到结构体上，通常用于从请求的 `Body` 或 `URL` 中提取数据并进行验证。`c.Bind` 可以绑定 JSON 数据、表单数据、URL 参数等。它通过反射将请求中的数据解析为指定结构体。

- **功能**：根据请求的内容类型（如 JSON、XML、表单数据等），将数据绑定到结构体字段中。
- **用法**：可以使用 `c.Bind()` 或 `c.ShouldBind()` 来绑定数据，`ShouldBind` 会返回绑定的错误（如果有），而 `Bind` 不会返回错误，需要手动检查。

#### 示例：

假设我们有一个用户结构体，表示请求的 JSON 数据结构：

```go
type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}
```

接下来，使用 `gin.Bind` 将 JSON 数据绑定到结构体上：

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func main() {
	// 创建 Gin 引擎
	r := gin.Default()

	// 创建 POST 路由，用于接收 JSON 数据
	r.POST("/user", func(c *gin.Context) {
		var user User
		// 将请求的 JSON 数据绑定到 user 结构体
		if err := c.ShouldBindJSON(&user); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		// 返回绑定后的数据
		c.JSON(http.StatusOK, gin.H{"message": "User created", "user": user})
	})

	// 启动服务器
	r.Run(":8080")
}
```

在这个例子中，当客户端向 `/user` 路由发送 POST 请求并携带 JSON 数据时，`gin.ShouldBindJSON` 会将请求体的 JSON 数据绑定到 `user` 结构体。

### 总结

- **`gin.Group`**：用于创建路由分组，可以为路由分组指定公共的前缀或中间件，简化代码的管理。
- **`gin.Bind`**（或 `gin.ShouldBind`）：用于将请求中的数据（如 JSON、表单等）绑定到 Go 结构体中，方便进行后续的处理。

这两个功能可以帮助你在 Gin 中组织路由和处理请求数据，简化开发流程，提高代码的可读性和可维护性。





## 热加载调试 Hot Reload

Python 的 `Flask`框架，有 *debug* 模式，启动时传入 *debug=True* 就可以热加载(Hot Reload, Live Reload)了。即更改源码，保存后，自动触发更新，浏览器上刷新即可。免去了杀进程、重新启动之苦。

Gin 原生不支持，但有很多额外的库可以支持。例如

- github.com/codegangsta/gin
- github.com/pilu/fresh

这次，我们采用 *github.com/pilu/fresh* 。

```
go get -v -u github.com/pilu/fresh
```

安装好后，只需要将`go run main.go`命令换成`fresh`即可。每次更改源文件，代码将自动重新编译(Auto Compile)。
