# Gin 快速入门

## 内容

[TOC]



## 构建标签

### 使用 JSON 替代

Gin 默认使用 `encoding/json` 作为 JSON 包，但你可以通过构建时选择其他标签来替换。

[jsoniter](https://github.com/json-iterator/go)

```sh
go build -tags=jsoniter .
```

[go-json](https://github.com/goccy/go-json)

```sh
go build -tags=go_json .
```

[sonic](https://github.com/bytedance/sonic) （需要确保 CPU 支持 AVX 指令）

```sh
$ go build -tags="sonic avx" .
```

### 构建时不使用 `MsgPack` 渲染功能

Gin 默认启用 `MsgPack` 渲染功能，但你可以通过指定 `nomsgpack` 构建标签来禁用该功能。

```sh
go build -tags=nomsgpack .
```

这样做有助于减小可执行文件的体积。详细信息请参见 [这篇文章](https://github.com/gin-gonic/gin/pull/1852)。

## API 示例

你可以在 [Gin 示例库](https://github.com/gin-gonic/examples) 中找到多个现成的示例。

### 使用 GET、POST、PUT、PATCH、DELETE 和 OPTIONS

```go
func main() {
  // 创建一个带有默认中间件（日志和恢复）的 Gin 路由器
  router := gin.Default()

  router.GET("/someGet", getting)
  router.POST("/somePost", posting)
  router.PUT("/somePut", putting)
  router.DELETE("/someDelete", deleting)
  router.PATCH("/somePatch", patching)
  router.HEAD("/someHead", head)
  router.OPTIONS("/someOptions", options)

  // 默认情况下，Gin 会在 :8080 端口提供服务，除非定义了 PORT 环境变量
  router.Run()
  // router.Run(":3000") 可以指定端口
}
```

### 路径中的参数

```go
func main() {
  router := gin.Default()

  // 这个处理器会匹配 /user/john，但不会匹配 /user/ 或 /user
  router.GET("/user/:name", func(c *gin.Context) {
    name := c.Param("name")
    c.String(http.StatusOK, "Hello %s", name)
  })

  // 这个处理器会匹配 /user/john/ 和 /user/john/send
  // 如果没有其他路由匹配 /user/john，它会重定向到 /user/john/
  router.GET("/user/:name/*action", func(c *gin.Context) {
    name := c.Param("name")
    action := c.Param("action")
    message := name + " is " + action
    c.String(http.StatusOK, message)
  })

  router.POST("/user/:name/*action", func(c *gin.Context) {
    b := c.FullPath() == "/user/:name/*action" // true
    c.String(http.StatusOK, "%t", b)
  })

  // 这个处理器会添加一个新的路由：/user/groups
  // 精确路由总是优先于参数路由，即使它们在定义时的顺序不同
  router.GET("/user/groups", func(c *gin.Context) {
    c.String(http.StatusOK, "The available groups are [...]")
  })

  router.Run(":8080")
}
```

### 查询字符串参数

```go
func main() {
  router := gin.Default()

  // 查询字符串参数通过底层请求对象进行解析。
  // 请求响应一个符合 URL 的匹配：/welcome?firstname=Jane&lastname=Doe
  router.GET("/welcome", func(c *gin.Context) {
    firstname := c.DefaultQuery("firstname", "Guest")
    lastname := c.Query("lastname") // 是 c.Request.URL.Query().Get("lastname") 的快捷方式

    c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
  })
  router.Run(":8080")
}
```

### 多部分表单/Urlencoded 表单

```go
func main() {
  router := gin.Default()

  router.POST("/form_post", func(c *gin.Context) {
    message := c.PostForm("message")
    nick := c.DefaultPostForm("nick", "anonymous")

    c.JSON(http.StatusOK, gin.H{
      "status":  "posted",
      "message": message,
      "nick":    nick,
    })
  })
  router.Run(":8080")
}
```

### 另一个示例：查询 + 表单

```sh
POST /post?id=1234&page=1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=manu&message=this_is_great
```

```go
func main() {
  router := gin.Default()

  router.POST("/post", func(c *gin.Context) {

    id := c.Query("id")
    page := c.DefaultQuery("page", "0")
    name := c.PostForm("name")
    message := c.PostForm("message")

    fmt.Printf("id: %s; page: %s; name: %s; message: %s", id, page, name, message)
  })
  router.Run(":8080")
}
```

```sh
id: 1234; page: 1; name: manu; message: this_is_great
```

### 将 Map 用作查询字符串或表单参数

```sh
POST /post?ids[a]=1234&ids[b]=hello HTTP/1.1
Content-Type: application/x-www-form-urlencoded

names[first]=thinkerou&names[second]=tianou
```

```go
func main() {
  router := gin.Default()

  router.POST("/post", func(c *gin.Context) {

    ids := c.QueryMap("ids")
    names := c.PostFormMap("names")

    fmt.Printf("ids: %v; names: %v", ids, names)
  })
  router.Run(":8080")
}
```

```sh
ids: map[b:hello a:1234]; names: map[second:tianou first:thinkerou]
```

### 文件上传

#### 单个文件

参考问题 [#774](https://github.com/gin-gonic/gin/issues/774) 和详细的 [示例代码](https://github.com/gin-gonic/examples/tree/master/upload-file/single)。

`file.Filename` **不应当**被信任。请参阅 [`Content-Disposition` 在 MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition#Directives) 和 [#1693](https://github.com/gin-gonic/gin/issues/1693)

> 文件名是可选的，不应被应用程序盲目使用：应剥离路径信息，并按照服务器文件系统规则进行转换。

```go
func main() {
  router := gin.Default()
  // 为多部分表单设置较低的内存限制（默认是 32 MiB）
  router.MaxMultipartMemory = 8 << 20  // 8 MiB
  router.POST("/upload", func(c *gin.Context) {
    // 单个文件
    file, _ := c.FormFile("file")
    log.Println(file.Filename)

    // 将文件上传到指定的目标。
    c.SaveUploadedFile(file, dst)

    c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
  })
  router.Run(":8080")
}
```

如何使用 `curl`：

```bash
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```

#### 多个文件

请参阅详细的 [示例代码](https://github.com/gin-gonic/examples/tree/master/upload-file/multiple)。

```go
func main() {
  router := gin.Default()
  // 为多部分表单设置较低的内存限制（默认是 32 MiB）
  router.MaxMultipartMemory = 8 << 20  // 8 MiB
  router.POST("/upload", func(c *gin.Context) {
    // 多部分表单
    form, _ := c.MultipartForm()
    files := form.File["upload[]"]

    for _, file := range files {
      log.Println(file.Filename)

      // 将文件上传到指定的目标。
      c.SaveUploadedFile(file, dst)
    }
    c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
  })
  router.Run(":8080")
}
```

如何使用 `curl`：

```bash
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

### 路由分组

```go
func main() {
  router := gin.Default()

  // 简单分组：v1
  {
    v1 := router.Group("/v1")
    v1.POST("/login", loginEndpoint)
    v1.POST("/submit", submitEndpoint)
    v1.POST("/read", readEndpoint)
  }

  // 简单分组：v2
  {
    v2 := router.Group("/v2")
    v2.POST("/login", loginEndpoint)
    v2.POST("/submit", submitEndpoint)
    v2.POST("/read", readEndpoint)
  }

  router.Run(":8080")
}
```

### 默认没有中间件的空 Gin

使用

```go
r := gin.New()
```

而不是

```go
// 默认情况下附带 Logger 和 Recovery 中间件
r := gin.Default()
```

### 使用中间件

```go
func main() {
  // 创建一个没有任何中间件的路由器
  r := gin.New()

  // 全局中间件
  // Logger 中间件将日志写入 gin.DefaultWriter，即使你设置了 GIN_MODE=release。
  // 默认情况下 gin.DefaultWriter = os.Stdout
  r.Use(gin.Logger())

  // Recovery 中间件从任何 panic 中恢复，如果发生了 panic，它会写入 500 错误。
  r.Use(gin.Recovery())

  // 每个路由的中间件，可以添加任意数量。
  r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

  // 授权分组
  // authorized := r.Group("/", AuthRequired())
  // 和下面的代码效果一样：
  authorized := r.Group("/")
  // 每个分组的中间件！在这个例子中，我们只在 "authorized" 分组中使用自定义创建的
  // AuthRequired() 中间件。
  authorized.Use(AuthRequired())
  {
    authorized.POST("/login", loginEndpoint)
    authorized.POST("/submit", submitEndpoint)
    authorized.POST("/read", readEndpoint)

    // 嵌套分组
    testing := authorized.Group("testing")
    // 访问 0.0.0.0:8080/testing/analytics
    testing.GET("/analytics", analyticsEndpoint)
  }

  // 启动并监听 0.0.0.0:8080
  r.Run(":8080")
}
```

### 自定义恢复行为

```go
func main() {
  // 创建一个没有任何中间件的路由器
  r := gin.New()

  // 全局中间件
  // Logger 中间件将日志写入 gin.DefaultWriter，即使你设置了 GIN_MODE=release。
  // 默认情况下 gin.DefaultWriter = os.Stdout
  r.Use(gin.Logger())

  // Recovery 中间件从任何 panic 中恢复，如果发生了 panic，它会写入 500 错误。
  r.Use(gin.CustomRecovery(func(c *gin.Context, recovered any) {
    if err, ok := recovered.(string); ok {
      c.String(http.StatusInternalServerError, fmt.Sprintf("error: %s", err))
    }
    c.AbortWithStatus(http.StatusInternalServerError)
  }))

  r.GET("/panic", func(c *gin.Context) {
    // panic 一个字符串 -- 自定义的中间件可以将其保存到数据库或反馈给用户
    panic("foo")
  })

  r.GET("/", func(c *gin.Context) {
    c.String(http.StatusOK, "ohai")
  })

  // 启动并监听 0.0.0.0:8080
  r.Run(":8080")
}
```

### 如何写入日志文件

```go
func main() {
  // 禁用控制台颜色，写入文件时不需要控制台颜色。
  gin.DisableConsoleColor()

  // 写入日志到文件。
  f, _ := os.Create("gin.log")
  gin.DefaultWriter = io.MultiWriter(f)

  // 如果你需要同时写入日志到文件和控制台，可以使用以下代码。
  // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

  router := gin.Default()
  router.GET("/ping", func(c *gin.Context) {
      c.String(http.StatusOK, "pong")
  })

  router.Run(":8080")
}
```

### 自定义日志格式

```go
func main() {
  router := gin.New()

  // LoggerWithFormatter 中间件将日志写入 gin.DefaultWriter
  // 默认情况下 gin.DefaultWriter = os.Stdout
  router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {

    // 自定义格式
    return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
        param.ClientIP,
        param.TimeStamp.Format(time.RFC1123),
        param.Method,
        param.Path,
        param.Request.Proto,
        param.StatusCode,
        param.Latency,
        param.Request.UserAgent(),
        param.ErrorMessage,
    )
  }))
  router.Use(gin.Recovery())

  router.GET("/ping", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
  })

  router.Run(":8080")
}
```

**输出示例**

```sh
::1 - [Fri, 07 Dec 2018 17:04:38 JST] "GET /ping HTTP/1.1 200 122.767µs "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36" "
```

### 跳过日志记录

```go
func main() {
  router := gin.New()

  // 通过设置 LoggerConfig 中的 SkipPaths 跳过特定路径的日志记录
  loggerConfig := gin.LoggerConfig{SkipPaths: []string{"/metrics"}}

  // 根据逻辑跳过日志记录，通过设置 LoggerConfig 中的 Skip 函数
  loggerConfig.Skip = func(c *gin.Context) bool {
      // 示例：跳过非服务器端错误
      return c.Writer.Status() < http.StatusInternalServerError
  }

  router.Use(gin.LoggerWithConfig(loggerConfig))
  router.Use(gin.Recovery())

  // 跳过日志记录
  router.GET("/metrics", func(c *gin.Context) {
      c.Status(http.StatusNotImplemented)
  })

  // 跳过日志记录
  router.GET("/ping", func(c *gin.Context) {
      c.String(http.StatusOK, "pong")
  })

  // 不跳过日志记录
  router.GET("/data", func(c *gin.Context) {
    c.Status(http.StatusNotImplemented)
  })

  router.Run(":8080")
}
```

### 控制日志输出颜色

默认情况下，日志在控制台输出时会根据 TTY 自动着色。

**不着色日志：**

```go
func main() {
  // 禁用日志的颜色
  gin.DisableConsoleColor()

  // 创建一个带有默认中间件的 gin 路由器：日志和恢复（无崩溃）中间件
  router := gin.Default()

  router.GET("/ping", func(c *gin.Context) {
      c.String(http.StatusOK, "pong")
  })

  router.Run(":8080")
}
```

**始终着色日志：**

```go
func main() {
  // 强制日志着色
  gin.ForceConsoleColor()

  // 创建一个带有默认中间件的 gin 路由器：日志和恢复（无崩溃）中间件
  router := gin.Default()

  router.GET("/ping", func(c *gin.Context) {
      c.String(http.StatusOK, "pong")
  })

  router.Run(":8080")
}
```

### 模型绑定与验证

要将请求体绑定到某个类型，使用模型绑定。我们目前支持绑定 JSON、XML、YAML、TOML 和标准的表单值（如 `foo=bar&boo=baz`）。

Gin 使用 [**go-playground/validator/v10**](https://github.com/go-playground/validator) 进行验证。关于标签使用的完整文档，请参阅 [这里](https://pkg.go.dev/github.com/go-playground/validator#hdr-Baked_In_Validators_and_Tags)。

请注意，您需要在所有字段上设置相应的绑定标签，例如，当从 JSON 绑定时，设置 `json:"fieldname"`。

Gin 提供了两组绑定方法：

- **必须绑定（Must Bind）**
  - **方法** - `Bind`，`BindJSON`，`BindXML`，`BindQuery`，`BindYAML`，`BindHeader`，`BindTOML`
  - **行为** - 这些方法在内部使用 `MustBindWith`，如果有绑定错误，请求会被中止，并且会返回 `c.AbortWithError(400, err).SetType(ErrorTypeBind)`，这将把响应的状态码设置为 400，并且 `Content-Type` 头会设置为 `text/plain; charset=utf-8`。如果在此之后尝试设置响应代码，将会得到一个警告 `[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422`。如果您希望更好地控制行为，可以使用 `ShouldBind` 等效方法。
  
- **应绑定（Should Bind）**
  - **方法** - `ShouldBind`，`ShouldBindJSON`，`ShouldBindXML`，`ShouldBindQuery`，`ShouldBindYAML`，`ShouldBindHeader`，`ShouldBindTOML`
  - **行为** - 这些方法在内部使用 `ShouldBindWith`，如果发生绑定错误，错误会被返回，开发者需要负责处理该请求和错误。

在使用 `Bind` 方法时，Gin 会根据 `Content-Type` 头来推断绑定器。如果您明确知道要绑定的内容类型，可以直接使用 `MustBindWith` 或 `ShouldBindWith`。

您还可以指定某些字段为必填。如果字段上装饰了 `binding:"required"` 且绑定时该字段为空值，则会返回错误。

```go
// 从 JSON 绑定
type Login struct {
  User     string `form:"user" json:"user" xml:"user" binding:"required"`  // 用户名字段
  Password string `form:"password" json:"password" xml:"password" binding:"required"`  // 密码字段
}

func main() {
  router := gin.Default()

  // JSON 绑定示例 ({"user": "manu", "password": "123"})
  router.POST("/loginJSON", func(c *gin.Context) {
    var json Login
    if err := c.ShouldBindJSON(&json); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if json.User != "manu" || json.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // XML 绑定示例
  //  <?xml version="1.0" encoding="UTF-8"?>
  //  <root>
  //    <user>manu</user>
  //    <password>123</password>
  //  </root>
  router.POST("/loginXML", func(c *gin.Context) {
    var xml Login
    if err := c.ShouldBindXML(&xml); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if xml.User != "manu" || xml.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // 表单绑定示例 (user=manu&password=123)
  router.POST("/loginForm", func(c *gin.Context) {
    var form Login
    // 根据 Content-Type 头来推断使用哪个绑定器
    if err := c.ShouldBind(&form); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if form.User != "manu" || form.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // 启动服务
  router.Run(":8080")
}
```

**请求示例**

```sh
$ curl -v -X POST \
  http://localhost:8080/loginJSON \
  -H 'content-type: application/json' \
  -d '{ "user": "manu" }'
> POST /loginJSON HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.51.0
> Accept: */*
> content-type: application/json
> Content-Length: 18
>
* upload completely sent off: 18 out of 18 bytes
< HTTP/1.1 400 Bad Request
< Content-Type: application/json; charset=utf-8
< Date: Fri, 04 Aug 2017 03:51:31 GMT
< Content-Length: 100
<
{"error":"Key: 'Login.Password' Error:Field validation for 'Password' failed on the 'required' tag"}
```

### 跳过验证

在运行上述示例时，使用上面的 `curl` 命令时返回了错误，因为示例使用了 `binding:"required"` 来验证 `Password` 字段。如果将 `Password` 字段的标签改为 `binding:"-"`，则在再次运行 `curl` 命令时不会返回错误。

### 自定义验证器

还可以注册自定义验证器。请查看 [示例代码](https://github.com/gin-gonic/examples/tree/master/custom-validation/server.go)。

```go
package main

import (
  "net/http"
  "time"

  "github.com/gin-gonic/gin"
  "github.com/gin-gonic/gin/binding"
  "github.com/go-playground/validator/v10"
)

// Booking 包含绑定和验证的数据
type Booking struct {
  CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
  CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn" time_format:"2006-01-02"`
}

// 自定义验证函数：检查日期是否可预定（是否晚于当前时间）
var bookableDate validator.Func = func(fl validator.FieldLevel) bool {
  date, ok := fl.Field().Interface().(time.Time)
  if ok {
    today := time.Now()
    if today.After(date) {
      return false
    }
  }
  return true
}

func main() {
  route := gin.Default()

  // 注册自定义验证器
  if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
    v.RegisterValidation("bookabledate", bookableDate)
  }

  route.GET("/bookable", getBookable)
  route.Run(":8085")
}

func getBookable(c *gin.Context) {
  var b Booking
  if err := c.ShouldBindWith(&b, binding.Query); err == nil {
    c.JSON(http.StatusOK, gin.H{"message": "Booking dates are valid!"})
  } else {
    c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
  }
}
```

**请求示例**

```sh
$ curl "localhost:8085/bookable?check_in=2030-04-16&check_out=2030-04-17"
{"message":"Booking dates are valid!"}

$ curl "localhost:8085/bookable?check_in=2030-03-10&check_out=2030-03-09"
{"error":"Key: 'Booking.CheckOut' Error:Field validation for 'CheckOut' failed on the 'gtfield' tag"}

$ curl "localhost:8085/bookable?check_in=2000-03-09&check_out=2000-03-10"
{"error":"Key: 'Booking.CheckIn' Error:Field validation for 'CheckIn' failed on the 'bookabledate' tag"}
```

[结构体级验证](https://github.com/go-playground/validator/releases/tag/v8.7) 也可以通过这种方式注册。有关更多信息，请参阅 [struct-lvl-validations 示例](https://github.com/gin-gonic/examples/tree/master/struct-lvl-validations)。

### 仅绑定查询字符串

`ShouldBindQuery` 函数仅绑定查询参数，而不绑定 POST 数据。更多信息请参阅 [GitHub issue](https://github.com/gin-gonic/gin/issues/742#issuecomment-315953017)。

```go
package main

import (
  "log"
  "net/http"

  "github.com/gin-gonic/gin"
)

type Person struct {
  Name    string `form:"name"`
  Address string `form:"address"`
}

func main() {
  route := gin.Default()
  route.Any("/testing", startPage)
  route.Run(":8085")
}

func startPage(c *gin.Context) {
  var person Person
  if c.ShouldBindQuery(&person) == nil {
    log.Println("====== 仅绑定查询字符串 ======")
    log.Println(person.Name)
    log.Println(person.Address)
  }
  c.String(http.StatusOK, "Success")
}
```

### 绑定查询字符串或 POST 数据

如果同时使用查询字符串和 POST 数据，Gin 会根据请求的 `content-type` 来决定使用哪个绑定方式。`GET` 请求会优先使用查询字符串绑定，`POST` 请求会根据 `content-type` 来决定是否使用 JSON、XML 或 `form-data`。

```go
package main

import (
  "log"
  "net/http"
  "time"

  "github.com/gin-gonic/gin"
)

type Person struct {
  Name       string    `form:"name"`
  Address    string    `form:"address"`
  Birthday   time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
  CreateTime time.Time `form:"createTime" time_format:"unixNano"`
  UnixTime   time.Time `form:"unixTime" time_format:"unix"`
}

func main() {
  route := gin.Default()
  route.GET("/testing", startPage)
  route.Run(":8085")
}

func startPage(c *gin.Context) {
  var person Person
  // 如果是 `GET` 请求，只会使用 `Form` 绑定引擎（查询字符串绑定）。
  // 如果是 `POST` 请求，首先检查 `content-type` 是否为 `JSON` 或 `XML`，然后使用 `Form`（表单数据）。
  if c.ShouldBind(&person) == nil {
    log.Println(person.Name)
    log.Println(person.Address)
    log.Println(person.Birthday)
    log.Println(person.CreateTime)
    log.Println(person.UnixTime)
  }

  c.String(http.StatusOK, "Success")
}
```

**测试命令：**

```sh
curl -X GET "localhost:8085/testing?name=appleboy&address=xyz&birthday=1992-03-15&createTime=1562400033000000123&unixTime=1562400033"
```

### 绑定默认值（如果未提供）

如果在客户端没有提供字段的值，服务器可以使用默认值进行绑定。可以在 `form` 标签内指定默认值：

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

type Person struct {
	Name      string    `form:"name,default=William"`
	Age       int       `form:"age,default=10"`
	Friends   []string  `form:"friends,default=Will;Bill"`
	Addresses [2]string `form:"addresses,default=foo bar" collection_format:"ssv"`
	LapTimes  []int     `form:"lap_times,default=1;2;3" collection_format:"csv"`
}

func main() {
	g := gin.Default()
	g.POST("/person", func(c *gin.Context) {
		var req Person
		if err := c.ShouldBindQuery(&req); err != nil {
			c.JSON(http.StatusBadRequest, err)
			return
		}
		c.JSON(http.StatusOK, req)
	})
	_ = g.Run("localhost:8080")
}
```

**测试请求：**

```sh
curl -X POST http://localhost:8080/person
```

**返回：**

```json
{
  "Name": "William",
  "Age": 10,
  "Friends": ["Will", "Bill"],
  "Addresses": ["foo", "bar"],
  "LapTimes": [1, 2, 3]
}
```

#### 注意：

对于默认值的 [集合类型](#collection-format-for-arrays)，需要遵循以下规则：

- 由于逗号用于分隔标签选项，逗号不能出现在默认值中，避免产生未定义行为。
- 对于 `multi` 和 `csv` 格式，应该使用分号（`;`）替代逗号来分隔默认值。
- 对于 `multi` 和 `csv` 格式，不能在默认值中使用分号作为值的分隔符。

#### 集合格式说明

| 格式         | 描述                       | 示例                    |
| ------------ | -------------------------- | ----------------------- |
| multi (默认) | 多个参数实例而不是多个值。 | key=foo&key=bar&key=baz |
| csv          | 逗号分隔的值。             | foo,bar,baz             |
| ssv          | 空格分隔的值。             | foo bar baz             |
| tsv          | 制表符分隔的值。           | "foo\tbar\tbaz"         |
| pipes        | 管道符分隔的值。           | foo\|bar\|baz           |

**示例代码：**

```go
package main

import (
	"log"
	"time"
	"github.com/gin-gonic/gin"
)

type Person struct {
	Name       string    `form:"name"`
	Addresses  []string  `form:"addresses" collection_format:"csv"`
	Birthday   time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
	CreateTime time.Time `form:"createTime" time_format:"unixNano"`
	UnixTime   time.Time `form:"unixTime" time_format:"unix"`
}

func main() {
	route := gin.Default()
	route.GET("/testing", startPage)
	route.Run(":8085")
}

func startPage(c *gin.Context) {
	var person Person
	// 如果是 GET 请求，使用 `Form` 引擎（查询字符串绑定）。
	// 如果是 POST 请求，根据 `content-type` 判断是否使用 JSON、XML 或 `form-data`。
	if c.ShouldBind(&person) == nil {
		log.Println(person.Name)
		log.Println(person.Addresses)
		log.Println(person.Birthday)
		log.Println(person.CreateTime)
		log.Println(person.UnixTime)
	}
	c.String(200, "Success")
}
```

**测试请求：**

```sh
curl -X GET "localhost:8085/testing?name=appleboy&addresses=foo,bar&birthday=1992-03-15&createTime=1562400033000000123&unixTime=1562400033"
```

### 绑定 URI 参数

Gin 还支持从 URI 中绑定参数。可以通过在结构体字段上使用 `uri` 标签来指定绑定的字段。

```go
package main

import (
  "net/http"

  "github.com/gin-gonic/gin"
)

type Person struct {
  ID   string `uri:"id" binding:"required,uuid"`
  Name string `uri:"name" binding:"required"`
}

func main() {
  route := gin.Default()
  route.GET("/:name/:id", func(c *gin.Context) {
    var person Person
    if err := c.ShouldBindUri(&person); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"msg": err.Error()})
      return
    }
    c.JSON(http.StatusOK, gin.H{"name": person.Name, "uuid": person.ID})
  })
  route.Run(":8088")
}
```

**测试请求：**

```sh
curl -v localhost:8088/thinkerou/987fbc97-4bed-5078-9f07-9141ba07c9f3
curl -v localhost:8088/thinkerou/not-uuid
```

**返回：**

- 当请求中 `id` 是有效 UUID 时，返回：

```json
{
  "name": "thinkerou",
  "uuid": "987fbc97-4bed-5078-9f07-9141ba07c9f3"
}
```

- 当 `id` 无效时，返回错误：

```json
{
  "msg": "Key: 'Person.ID' Error:Field validation for 'ID' failed on the 'uuid' tag"
}
```

### 绑定自定义的 Unmarshaler

```go
package main

import (
  "github.com/gin-gonic/gin"
  "strings"
)

type Birthday string

func (b *Birthday) UnmarshalParam(param string) error {
  *b = Birthday(strings.Replace(param, "-", "/", -1))
  return nil
}

func main() {
  route := gin.Default()
  var request struct {
    Birthday Birthday `form:"birthday"`
  }
  route.GET("/test", func(ctx *gin.Context) {
    _ = ctx.BindQuery(&request)
    ctx.JSON(200, request.Birthday)
  })
  route.Run(":8088")
}
```

测试命令：

```sh
curl 'localhost:8088/test?birthday=2000-01-01'
```

结果：

```sh
"2000/01/01"
```

### 绑定请求头（Header）

```go
package main

import (
  "fmt"
  "net/http"

  "github.com/gin-gonic/gin"
)

type testHeader struct {
  Rate   int    `header:"Rate"`
  Domain string `header:"Domain"`
}

func main() {
  r := gin.Default()
  r.GET("/", func(c *gin.Context) {
    h := testHeader{}

    if err := c.ShouldBindHeader(&h); err != nil {
      c.JSON(http.StatusOK, err)
    }

    fmt.Printf("%#v\n", h)
    c.JSON(http.StatusOK, gin.H{"Rate": h.Rate, "Domain": h.Domain})
  })

  r.Run()

// 客户端
// curl -H "rate:300" -H "domain:music" 127.0.0.1:8080/
// 输出
// {"Domain":"music","Rate":300}
}
```

### 绑定 HTML 表单中的复选框（Checkboxes）

查看详细信息：[详细信息链接](https://github.com/gin-gonic/gin/issues/129#issuecomment-124260092)

**main.go**

```go
...

type myForm struct {
    Colors []string `form:"colors[]"`
}

...

func formHandler(c *gin.Context) {
    var fakeForm myForm
    c.ShouldBind(&fakeForm)
    c.JSON(http.StatusOK, gin.H{"color": fakeForm.Colors})
}

...

```

**form.html**

```html
<form action="/" method="POST">
    <p>Check some colors</p>
    <label for="red">Red</label>
    <input type="checkbox" name="colors[]" value="red" id="red">
    <label for="green">Green</label>
    <input type="checkbox" name="colors[]" value="green" id="green">
    <label for="blue">Blue</label>
    <input type="checkbox" name="colors[]" value="blue" id="blue">
    <input type="submit">
</form>
```

**结果：**

```json
{"color":["red","green","blue"]}
```

### Multipart/Urlencoded 绑定

```go
type ProfileForm struct {
  Name   string                `form:"name" binding:"required"`
  Avatar *multipart.FileHeader `form:"avatar" binding:"required"`

  // 或者对于多个文件
  // Avatars []*multipart.FileHeader `form:"avatar" binding:"required"`
}

func main() {
  router := gin.Default()
  router.POST("/profile", func(c *gin.Context) {
    // 你可以通过显式声明来绑定 multipart 表单：
    // c.ShouldBindWith(&form, binding.Form)
    // 或者你可以简单地使用自动绑定与 ShouldBind 方法：
    var form ProfileForm
    // 在这种情况下，将自动选择合适的绑定方式
    if err := c.ShouldBind(&form); err != nil {
      c.String(http.StatusBadRequest, "bad request")
      return
    }

    err := c.SaveUploadedFile(form.Avatar, form.Avatar.Filename)
    if err != nil {
      c.String(http.StatusInternalServerError, "unknown error")
      return
    }

    // db.Save(&form)

    c.String(http.StatusOK, "ok")
  })
  router.Run(":8080")
}
```

测试命令：

```sh
curl -X POST -v --form name=user --form "avatar=@./avatar.png" http://localhost:8080/profile
```

### XML、JSON、YAML、TOML 和 ProtoBuf 渲染

```go
func main() {
  r := gin.Default()

  // gin.H 是 map[string]any 的快捷方式
  r.GET("/someJSON", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
  })

  r.GET("/moreJSON", func(c *gin.Context) {
    // 你也可以使用结构体
    var msg struct {
      Name    string `json:"user"`
      Message string
      Number  int
    }
    msg.Name = "Lena"
    msg.Message = "hey"
    msg.Number = 123
    // 注意 msg.Name 变成了 "user"
    // 输出将是： {"user": "Lena", "Message": "hey", "Number": 123}
    c.JSON(http.StatusOK, msg)
  })

  r.GET("/someXML", func(c *gin.Context) {
    c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
  })

  r.GET("/someYAML", func(c *gin.Context) {
    c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
  })

  r.GET("/someTOML", func(c *gin.Context) {
    c.TOML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
  })

  r.GET("/someProtoBuf", func(c *gin.Context) {
    reps := []int64{int64(1), int64(2)}
    label := "test"
    // protobuf 的具体定义在 testdata/protoexample 文件中
    data := &protoexample.Test{
      Label: &label,
      Reps:  reps,
    }
    // 注意，data 会作为二进制数据返回
    // 输出 protobuf 序列化数据
    c.ProtoBuf(http.StatusOK, data)
  })

  // 启动监听：0.0.0.0:8080
  r.Run(":8080")
}
```

#### SecureJSON

使用 SecureJSON 防止 JSON 劫持。如果响应的结构体为数组，默认会在响应体前面加上 `"while(1),"`。

```go
func main() {
  r := gin.Default()

  // 你也可以设置自己的 secure json 前缀
  // r.SecureJsonPrefix(")]}',\n")

  r.GET("/someJSON", func(c *gin.Context) {
    names := []string{"lena", "austin", "foo"}

    // 输出将是： while(1);["lena","austin","foo"]
    c.SecureJSON(http.StatusOK, names)
  })

  // 启动监听：0.0.0.0:8080
  r.Run(":8080")
}
```

#### JSONP

使用 JSONP 从不同域的服务器请求数据。如果查询参数中存在 callback，响应体将包含该回调函数。

```go
func main() {
  r := gin.Default()

  r.GET("/JSONP", func(c *gin.Context) {
    data := gin.H{
      "foo": "bar",
    }

    // callback 是 x
    // 输出将是： x({\"foo\":\"bar\"})
    c.JSONP(http.StatusOK, data)
  })

  // 启动监听：0.0.0.0:8080
  r.Run(":8080")

        // 客户端
        // curl http://127.0.0.1:8080/JSONP?callback=x
}
```

#### AsciiJSON

使用 AsciiJSON 生成仅包含 ASCII 的 JSON，并转义所有非 ASCII 字符。

```go
func main() {
  r := gin.Default()

  r.GET("/someJSON", func(c *gin.Context) {
    data := gin.H{
      "lang": "GO语言",
      "tag":  "<br>",
    }

    // 输出将是： {"lang":"GO\u8bed\u8a00","tag":"\u003cbr\u003e"}
    c.AsciiJSON(http.StatusOK, data)
  })

  // 启动监听：0.0.0.0:8080
  r.Run(":8080")
}
```

#### PureJSON

通常，JSON 会将特殊的 HTML 字符替换为它们的 Unicode 实体，例如 `<` 会变成 `\u003c`。如果你希望直接编码这些字符，可以使用 PureJSON。

```go
func main() {
  r := gin.Default()

  // 返回 Unicode 实体
  r.GET("/json", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
      "html": "<b>Hello, world!</b>",
    })
  })

  // 返回字面值字符
  r.GET("/purejson", func(c *gin.Context) {
    c.PureJSON(http.StatusOK, gin.H{
      "html": "<b>Hello, world!</b>",
    })
  })

  // 启动监听：0.0.0.0:8080
  r.Run(":8080")
}
```

### 提供静态文件

```go
func main() {
  router := gin.Default()
  router.Static("/assets", "./assets")
  router.StaticFS("/more_static", http.Dir("my_file_system"))
  router.StaticFile("/favicon.ico", "./resources/favicon.ico")
  router.StaticFileFS("/more_favicon.ico", "more_favicon.ico", http.Dir("my_file_system"))

  // 启动监听：0.0.0.0:8080
  router.Run(":8080")
}
```

### 从文件提供数据

```go
func main() {
  router := gin.Default()

  router.GET("/local/file", func(c *gin.Context) {
    c.File("local/file.go")
  })

  var fs http.FileSystem = // ...
  router.GET("/fs/file", func(c *gin.Context) {
    c.FileFromFS("fs/file.go", fs)
  })
}
```

### 从读取器提供数据

```go
func main() {
  router := gin.Default()
  router.GET("/someDataFromReader", func(c *gin.Context) {
    response, err := http.Get("https://raw.githubusercontent.com/gin-gonic/logo/master/color.png")
    if err != nil || response.StatusCode != http.StatusOK {
      c.Status(http.StatusServiceUnavailable)
      return
    }

    reader := response.Body
    defer reader.Close()
    contentLength := response.ContentLength
    contentType := response.Header.Get("Content-Type")

    extraHeaders := map[string]string{
      "Content-Disposition": `attachment; filename="gopher.png"`,
    }

    c.DataFromReader(http.StatusOK, contentLength, contentType, reader, extraHeaders)
  })
  router.Run(":8080")
}
```

### HTML 渲染

使用 `LoadHTMLGlob()` 或 `LoadHTMLFiles()` 加载模板。

```go
func main() {
  router := gin.Default()
  router.LoadHTMLGlob("templates/*")
  // router.LoadHTMLFiles("templates/template1.html", "templates/template2.html")
  router.GET("/index", func(c *gin.Context) {
    c.HTML(http.StatusOK, "index.tmpl", gin.H{
      "title": "Main website",
    })
  })
  router.Run(":8080")
}
```

**templates/index.tmpl**

```html
<html>
  <h1>
    {{ .title }}
  </h1>
</html>
```

使用不同目录中相同名称的模板

```go
func main() {
  router := gin.Default()
  router.LoadHTMLGlob("templates/**/*")
  router.GET("/posts/index", func(c *gin.Context) {
    c.HTML(http.StatusOK, "posts/index.tmpl", gin.H{
      "title": "Posts",
    })
  })
  router.GET("/users/index", func(c *gin.Context) {
    c.HTML(http.StatusOK, "users/index.tmpl", gin.H{
      "title": "Users",
    })
  })
  router.Run(":8080")
}
```

**templates/posts

/index.tmpl**

```html
{{ define "posts/index.tmpl" }}
<html><h1>
  {{ .title }}
</h1>
<p>Using posts/index.tmpl</p>
</html>
{{ end }}
```

**templates/users/index.tmpl**

```html
{{ define "users/index.tmpl" }}
<html><h1>
  {{ .title }}
</h1>
<p>Using users/index.tmpl</p>
</html>
{{ end }}
```

#### 自定义模板渲染器

你也可以使用自定义的 HTML 模板渲染器。

```go
import "html/template"

func main() {
  router := gin.Default()
  html := template.Must(template.ParseFiles("file1", "file2"))
  router.SetHTMLTemplate(html)
  router.Run(":8080")
}
```

#### 自定义定界符

你可以使用自定义定界符来渲染模板。

```go
r := gin.Default()
r.Delims("{[{", "}]}") // 设置自定义的定界符
r.LoadHTMLGlob("/path/to/templates")
```

#### 自定义模板函数

你可以自定义模板函数，如下所示：

```go
import (
  "fmt"
  "html/template"
  "net/http"
  "time"

  "github.com/gin-gonic/gin"
)

func formatAsDate(t time.Time) string {
  year, month, day := t.Date()
  return fmt.Sprintf("%d/%02d/%02d", year, month, day)
}

func main() {
  router := gin.Default()
  router.Delims("{[{", "}]}") // 自定义定界符
  router.SetFuncMap(template.FuncMap{
      "formatAsDate": formatAsDate,
  })
  router.LoadHTMLFiles("./testdata/template/raw.tmpl")

  router.GET("/raw", func(c *gin.Context) {
      c.HTML(http.StatusOK, "raw.tmpl", gin.H{
          "now": time.Date(2017, 07, 01, 0, 0, 0, 0, time.UTC),
      })
  })

  router.Run(":8080")
}
```

**raw.tmpl**

```html
Date: {[{.now | formatAsDate}]}
```

结果：

```sh
Date: 2017/07/01
```

### 多模板

Gin 默认只支持使用一个 `html.Template`。可以参考 [多模板渲染](https://github.com/gin-contrib/multitemplate) 来使用 Go 1.6 版本的 `block template` 等功能。

### 重定向

发起 HTTP 重定向很简单，支持内部和外部地址。

```go
r.GET("/test", func(c *gin.Context) {
  c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})
```

从 POST 请求发起 HTTP 重定向，参考问题：[#444](https://github.com/gin-gonic/gin/issues/444)

```go
r.POST("/test", func(c *gin.Context) {
  c.Redirect(http.StatusFound, "/foo")
})
```

路由重定向，使用 `HandleContext`：

```go
r.GET("/test", func(c *gin.Context) {
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"hello": "world"})
})
```

### 自定义中间件

```go
func Logger() gin.HandlerFunc {
  return func(c *gin.Context) {
    t := time.Now()

    // 设置示例变量
    c.Set("example", "12345")

    // 请求前操作

    c.Next()

    // 请求后操作
    latency := time.Since(t)
    log.Print(latency)

    // 获取返回的状态
    status := c.Writer.Status()
    log.Println(status)
  }
}

func main() {
  r := gin.New()
  r.Use(Logger())

  r.GET("/test", func(c *gin.Context) {
    example := c.MustGet("example").(string)

    // 输出: "12345"
    log.Println(example)
  })

  // 在 0.0.0.0:8080 上监听并服务
  r.Run(":8080")
}
```

### 使用 BasicAuth() 中间件

```go
// 模拟一些私人数据
var secrets = gin.H{
  "foo":    gin.H{"email": "foo@bar.com", "phone": "123433"},
  "austin": gin.H{"email": "austin@example.com", "phone": "666"},
  "lena":   gin.H{"email": "lena@guapa.com", "phone": "523443"},
}

func main() {
  r := gin.Default()

  // 使用 gin.BasicAuth() 中间件的分组
  // gin.Accounts 是 map[string]string 的快捷方式
  authorized := r.Group("/admin", gin.BasicAuth(gin.Accounts{
    "foo":    "bar",
    "austin": "1234",
    "lena":   "hello2",
    "manu":   "4321",
  }))

  // /admin/secrets 路径
  // 访问 "localhost:8080/admin/secrets"
  authorized.GET("/secrets", func(c *gin.Context) {
    // 获取用户，这是 BasicAuth 中间件设置的
    user := c.MustGet(gin.AuthUserKey).(string)
    if secret, ok := secrets[user]; ok {
      c.JSON(http.StatusOK, gin.H{"user": user, "secret": secret})
    } else {
      c.JSON(http.StatusOK, gin.H{"user": user, "secret": "没有秘密 :("})
    }
  })

  // 在 0.0.0.0:8080 上监听并服务
  r.Run(":8080")
}
```

### 中间件内使用 Goroutines

当在中间件或处理程序内启动新的 Goroutines 时，**不应**直接使用原始的上下文，需要使用只读副本。

```go
func main() {
  r := gin.Default()

  r.GET("/long_async", func(c *gin.Context) {
    // 创建副本，在 Goroutine 中使用
    cCp := c.Copy()
    go func() {
      // 模拟一个耗时任务，5 秒
      time.Sleep(5 * time.Second)

      // 使用复制的上下文 "cCp"，非常重要
      log.Println("完成! 路径：" + cCp.Request.URL.Path)
    }()
  })

  r.GET("/long_sync", func(c *gin.Context) {
    // 模拟一个耗时任务，5 秒
    time.Sleep(5 * time.Second)

    // 因为没有使用 Goroutine，可以直接使用原始上下文
    log.Println("完成! 路径：" + c.Request.URL.Path)
  })

  // 在 0.0.0.0:8080 上监听并服务
  r.Run(":8080")
}
```

### 自定义 HTTP 配置

直接使用 `http.ListenAndServe()`：

```go
func main() {
  router := gin.Default()
  http.ListenAndServe(":8080", router)
}
```

或者：

```go
func main() {
  router := gin.Default()

  s := &http.Server{
    Addr:           ":8080",
    Handler:        router,
    ReadTimeout:    10 * time.Second,
    WriteTimeout:   10 * time.Second,
    MaxHeaderBytes: 1 << 20,
  }
  s.ListenAndServe()
}
```

### 支持 Let's Encrypt

下面是一个 1 行代码使用 Let's Encrypt 配置 HTTPS 服务器的示例。

```go
package main

import (
  "log"
  "net/http"

  "github.com/gin-gonic/autotls"
  "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()

  // Ping 请求处理
  r.GET("/ping", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
  })

  log.Fatal(autotls.Run(r, "example1.com", "example2.com"))
}
```

下面是一个使用自定义自动证书管理器的示例。

```go
package main

import (
  "log"
  "net/http"

  "github.com/gin-gonic/autotls"
  "github.com/gin-gonic/gin"
  "golang.org/x/crypto/acme/autocert"
)

func main() {
  r := gin.Default()

  // Ping 请求处理
  r.GET("/ping", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
  })

  m := autocert.Manager{
    Prompt:     autocert.AcceptTOS,
    HostPolicy: autocert.HostWhitelist("example1.com", "example2.com"),
    Cache:      autocert.DirCache("/var/www/.cache"),
  }

  log.Fatal(autotls.RunWithManager(r, &m))
}
```

### 使用 Gin 运行多个服务

可以参考 [这个问题](https://github.com/gin-gonic/gin/issues/346)，尝试以下示例：

```go
package main

import (
  "log"
  "net/http"
  "time"

  "github.com/gin-gonic/gin"
  "golang.org/x/sync/errgroup"
)

var (
  g errgroup.Group
)

func router01() http.Handler {
  e := gin.New()
  e.Use(gin.Recovery())
  e.GET("/", func(c *gin.Context) {
    c.JSON(
      http.StatusOK,
      gin.H{
        "code":  http.StatusOK,
        "error": "Welcome server 01",
      },
    )
  })

  return e
}

func router02() http.Handler {
  e := gin.New()
  e.Use(gin.Recovery())
  e.GET("/", func(c *gin.Context) {
    c.JSON(
      http.StatusOK,
      gin.H{
        "code":  http.StatusOK,
        "error": "Welcome server 02",
      },
    )
  })

  return e
}

func main() {
  server01 := &http.Server{
    Addr:         ":8080",
    Handler:      router01(),
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
  }

  server02 := &http.Server{
    Addr:         ":8081",
    Handler:      router02(),
    ReadTimeout:  5 * time.Second,
    WriteTimeout: 10 * time.Second,
  }

  g.Go(func() error {
    err := server01.ListenAndServe()
    if err != nil && err != http.ErrServerClosed {
      log.Fatal(err)
    }
    return err
  })

  g.Go(func() error {
    err := server02.ListenAndServe()
    if err != nil && err != http.ErrServerClosed {
      log.Fatal(err)
    }
    return err
  })

  if err := g.Wait(); err != nil {
    log.Fatal(err)
  }
}
```

### 优雅关闭或重启

有几种方法可以实现优雅的关闭或重启。你可以使用专门为此目的设计的第三方包，或者使用 Go 内置的功能和方法手动实现。

#### 使用第三方包

我们可以使用 [fvbock/endless](https://github.com/fvbock/endless) 来替代默认的 `ListenAndServe`。更多细节请参考问题 [#296](https://github.com/gin-gonic/gin/issues/296)。

```go
router := gin.Default()
router.GET("/", handler)
// [...]
// 使用 endless 包提供的 ListenAndServe
endless.ListenAndServe(":4242", router)
```

其他替代方案：

* [grace](https://github.com/facebookgo/grace): 为 Go 服务器提供优雅的重启和零停机部署。
* [graceful](https://github.com/tylerb/graceful): 一个 Go 包，支持 http.Handler 服务器的优雅关闭。
* [manners](https://github.com/braintree/manners): 一个礼貌的 Go HTTP 服务器，支持优雅关闭。

#### 手动实现

如果你使用的是 Go 1.8 或更高版本，可能不需要使用这些库。可以考虑使用 `http.Server` 自带的 [Shutdown()](https://pkg.go.dev/net/http#Server.Shutdown) 方法进行优雅关闭。以下是其用法示例，更多 Gin 的优雅关闭示例可以参考 [这里](https://github.com/gin-gonic/examples/tree/master/graceful-shutdown)。

```go
// +build go1.8

package main

import (
  "context"
  "log"
  "net/http"
  "os"
  "os/signal"
  "syscall"
  "time"

  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  router.GET("/", func(c *gin.Context) {
    time.Sleep(5 * time.Second)
    c.String(http.StatusOK, "Welcome Gin Server")
  })

  srv := &http.Server{
    Addr:    ":8080",
    Handler: router,
  }

  // 在 goroutine 中启动服务器，避免阻塞下面的优雅关闭处理
  go func() {
    if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
      log.Printf("listen: %s\n", err)
    }
  }()

  // 等待中断信号，优雅关闭服务器，设置超时时间为 5 秒
  quit := make(chan os.Signal)
  signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
  <-quit
  log.Println("Shutting down server...")

  // 使用上下文设置 5 秒的超时时间来告知服务器当前正在处理的请求
  ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
  defer cancel()

  if err := srv.Shutdown(ctx); err != nil {
    log.Fatal("Server forced to shutdown:", err)
  }

  log.Println("Server exiting")
}
```

### 使用模板构建单一二进制文件

你可以通过使用 [embed](https://pkg.go.dev/embed) 包，将一个包含模板的服务器构建成一个单一的二进制文件。

```go
package main

import (
  "embed"
  "html/template"
  "net/http"

  "github.com/gin-gonic/gin"
)

//go:embed assets/* templates/*
var f embed.FS

func main() {
  router := gin.Default()
  templ := template.Must(template.New("").ParseFS(f, "templates/*.tmpl", "templates/foo/*.tmpl"))
  router.SetHTMLTemplate(templ)

  // 示例：/public/assets/images/example.png
  router.StaticFS("/public", http.FS(f))

  router.GET("/", func(c *gin.Context) {
    c.HTML(http.StatusOK, "index.tmpl", gin.H{
      "title": "Main website",
    })
  })

  router.GET("/foo", func(c *gin.Context) {
    c.HTML(http.StatusOK, "bar.tmpl", gin.H{
      "title": "Foo website",
    })
  })

  router.GET("favicon.ico", func(c *gin.Context) {
    file, _ := f.ReadFile("assets/favicon.ico")
    c.Data(
      http.StatusOK,
      "image/x-icon",
      file,
    )
  })

  router.Run(":8080")
}
```

查看完整示例，访问 `https://github.com/gin-gonic/examples/tree/master/assets-in-binary/example02` 目录。

### 将表单数据请求与自定义结构体绑定

以下示例展示了如何使用自定义结构体进行绑定：

```go
type StructA struct {
    FieldA string `form:"field_a"`
}

type StructB struct {
    NestedStruct StructA
    FieldB string `form:"field_b"`
}

type StructC struct {
    NestedStructPointer *StructA
    FieldC string `form:"field_c"`
}

type StructD struct {
    NestedAnonyStruct struct {
        FieldX string `form:"field_x"`
    }
    FieldD string `form:"field_d"`
}

func GetDataB(c *gin.Context) {
    var b StructB
    c.Bind(&b)
    c.JSON(http.StatusOK, gin.H{
        "a": b.NestedStruct,
        "b": b.FieldB,
    })
}

func GetDataC(c *gin.Context) {
    var b StructC
    c.Bind(&b)
    c.JSON(http.StatusOK, gin.H{
        "a": b.NestedStructPointer,
        "c": b.FieldC,
    })
}

func GetDataD(c *gin.Context) {
    var b StructD
    c.Bind(&b)
    c.JSON(http.StatusOK, gin.H{
        "x": b.NestedAnonyStruct,
        "d": b.FieldD,
    })
}

func main() {
    r := gin.Default()
    r.GET("/getb", GetDataB)
    r.GET("/getc", GetDataC)
    r.GET("/getd", GetDataD)

    r.Run()
}
```

使用 `curl` 命令的结果：

```sh
$ curl "http://localhost:8080/getb?field_a=hello&field_b=world"
{"a":{"FieldA":"hello"},"b":"world"}
$ curl "http://localhost:8080/getc?field_a=hello&field_c=world"
{"a":{"FieldA":"hello"},"c":"world"}
$ curl "http://localhost:8080/getd?field_x=hello&field_d=world"
{"d":"world","x":{"FieldX":"hello"}}
```

### 尝试将请求体绑定到不同的结构体

通常情况下，绑定请求体的方法会消耗 `c.Request.Body`，并且不能多次调用。

```go
type formA struct {
  Foo string `json:"foo" xml:"foo" binding:"required"`
}

type formB struct {
  Bar string `json:"bar" xml:"bar" binding:"required"`
}

func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // c.ShouldBind 会消耗 c.Request.Body，不能重复使用
  if errA := c.ShouldBind(&objA); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // 这里总是会出错，因为此时 c.Request.Body 已经是 EOF 了
  } else if errB := c.ShouldBind(&objB); errB == nil {
    c.String(http.StatusOK, `the body should be formB`)
  } else {
    ...
  }
}
```

为了解决这个问题，你可以使用 `c.ShouldBindBodyWith` 或者简化的快捷方式。

- `c.ShouldBindBodyWithJSON` 是 `c.ShouldBindBodyWith(obj, binding.JSON)` 的快捷方式。
- `c.ShouldBindBodyWithXML` 是 `c.ShouldBindBodyWith(obj, binding.XML)` 的快捷方式。
- `c.ShouldBindBodyWithYAML` 是 `c.ShouldBindBodyWith(obj, binding.YAML)` 的快捷方式。
- `c.ShouldBindBodyWithTOML` 是 `c.ShouldBindBodyWith(obj, binding.TOML)` 的快捷方式。

```go
func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // 这会读取 c.Request.Body 并将结果存储到上下文中
  if errA := c.ShouldBindBodyWith(&objA, binding.Form); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // 这时，它会重用存储在上下文中的 body
  } else if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
    c.String(http.StatusOK, `the body should be formB JSON`)
  // 还可以接受其他格式
  } else if errB2 := c.ShouldBindBodyWithXML(&objB); errB2 == nil {
    c.String(http.StatusOK, `the body should be formB XML`)
  } else {
    ...
  }
}
```

1. `c.ShouldBindBodyWith` 会在绑定之前将请求体存储到上下文中。这个方法对性能有轻微的影响，因此如果你只需要一次绑定，可以避免使用它。
2. 这个功能只对某些格式有用 —— `JSON`、`XML`、`MsgPack`、`ProtoBuf` 等。对于其他格式，`Query`、`Form`、`FormPost`、`FormMultipart` 可以多次调用 `c.ShouldBind()`，对性能不会造成影响（参见 [#1341](https://github.com/gin-gonic/gin/pull/1341)）。

### 使用自定义结构体和自定义标签绑定表单数据请求

```go
const (
  customerTag = "url"
  defaultMemory = 32 << 20
)

type customerBinding struct {}

func (customerBinding) Name() string {
  return "form"
}

func (customerBinding) Bind(req *http.Request, obj any) error {
  if err := req.ParseForm(); err != nil {
    return err
  }
  if err := req.ParseMultipartForm(defaultMemory); err != nil {
    if err != http.ErrNotMultipart {
      return err
    }
  }
  if err := binding.MapFormWithTag(obj, req.Form, customerTag); err != nil {
    return err
  }
  return validate(obj)
}

func validate(obj any) error {
  if binding.Validator == nil {
    return nil
  }
  return binding.Validator.ValidateStruct(obj)
}

// 现在我们可以这么做!!!
// FormA 是外部类型，我们不能修改它的标签
type FormA struct {
  FieldA string `url:"field_a"`
}

func ListHandler(s *Service) func(ctx *gin.Context) {
  return func(ctx *gin.Context) {
    var urlBinding = customerBinding{}
    var opt FormA
    err := ctx.MustBindWith(&opt, urlBinding)
    if err != nil {
      ...
    }
    ...
  }
}
```

### HTTP/2 服务器推送

`http.Pusher` 仅在 **Go1.8+** 中受支持。有关详细信息，请参见 [Go 博客](https://go.dev/blog/h2push)。

```go
package main

import (
  "html/template"
  "log"
  "net/http"

  "github.com/gin-gonic/gin"
)

var html = template.Must(template.New("https").Parse(`
<html>
<head>
  <title>Https Test</title>
  <script src="/assets/app.js"></script>
</head>
<body>
  <h1 style="color:red;">Welcome, Ginner!</h1>
</body>
</html>
`))

func main() {
  r := gin.Default()
  r.Static("/assets", "./assets")
  r.SetHTMLTemplate(html)

  r.GET("/", func(c *gin.Context) {
    if pusher := c.Writer.Pusher(); pusher != nil {
      // 使用 pusher.Push() 进行服务器推送
      if err := pusher.Push("/assets/app.js", nil); err != nil {
        log.Printf("Failed to push: %v", err)
      }
    }
    c.HTML(http.StatusOK, "https", gin.H{
      "status": "success",
    })
  })

  // 使用 HTTPS 监听并服务于 https://127.0.0.1:8080
  r.RunTLS(":8080", "./testdata/server.pem", "./testdata/server.key")
}
```

### 定义路由日志格式

默认的路由日志格式为：

```sh
[GIN-debug] POST   /foo                      --> main.main.func1 (3 handlers)
[GIN-debug] GET    /bar                      --> main.main.func2 (3 handlers)
[GIN-debug] GET    /status                   --> main.main.func3 (3 handlers)
```

如果你希望以指定格式（例如 JSON、键值对或其他格式）记录这些信息，可以使用 `gin.DebugPrintRouteFunc` 来定义日志格式。以下示例中，我们使用标准日志包记录所有路由，但你可以根据需要使用其他日志工具。

```go
import (
  "log"
  "net/http"

  "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
    log.Printf("endpoint %v %v %v %v\n", httpMethod, absolutePath, handlerName, nuHandlers)
  }

  r.POST("/foo", func(c *gin.Context) {
    c.JSON(http.StatusOK, "foo")
  })

  r.GET("/bar", func(c *gin.Context) {
    c.JSON(http.StatusOK, "bar")
  })

  r.GET("/status", func(c *gin.Context) {
    c.JSON(http.StatusOK, "ok")
  })

  // 在 http://0.0.0.0:8080 上监听并服务
  r.Run()
}
```

### 设置和获取 Cookie

```go
import (
  "fmt"

  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()

  router.GET("/cookie", func(c *gin.Context) {

      cookie, err := c.Cookie("gin_cookie")

      if err != nil {
          cookie = "NotSet"
          c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
      }

      fmt.Printf("Cookie 值: %s \n", cookie)
  })

  router.Run()
}
```

### 不信任所有代理

Gin 允许你指定哪些请求头包含真实客户端 IP（如果有），并指定你信任的代理（或直接客户端）来指定这些请求头。

使用 `SetTrustedProxies()` 函数在 `gin.Engine` 上指定你信任的网络地址或网络 CIDR。可以是 IPv4 地址、IPv4 CIDR、IPv6 地址或 IPv6 CIDR。

**注意：** 默认情况下，Gin 会信任所有代理，如果没有使用上面的函数指定受信任的代理，这是 **不安全** 的。与此同时，如果你不使用任何代理，可以通过使用 `Engine.SetTrustedProxies(nil)` 来禁用此功能，然后 `Context.ClientIP()` 会直接返回远程地址，从而避免不必要的计算。

```go
import (
  "fmt"

  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  router.SetTrustedProxies([]string{"192.168.1.2"})

  router.GET("/", func(c *gin.Context) {
    // 如果客户端是 192.168.1.2，使用 X-Forwarded-For
    // 请求头从可信部分推断出原始客户端 IP。
    // 否则，直接返回客户端 IP
    fmt.Printf("ClientIP: %s\n", c.ClientIP())
  })
  router.Run()
}
```

**注意：** 如果你使用了 CDN 服务，可以设置 `Engine.TrustedPlatform` 来跳过 `TrustedProxies` 检查，`TrustedPlatform` 的优先级高于 `TrustedProxies`。以下是示例：

```go
import (
  "fmt"

  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  // 使用预定义的 gin.PlatformXXX
  // Google App Engine
  router.TrustedPlatform = gin.PlatformGoogleAppEngine
  // Cloudflare
  router.TrustedPlatform = gin.PlatformCloudflare
  // Fly.io
  router.TrustedPlatform = gin.PlatformFlyIO
  // 或者，你可以设置你自己的受信请求头。确保 CDN 阻止用户传递这个头！例如，如果 CDN
  // 将客户端 IP 存放在 X-CDN-Client-IP 中：
  router.TrustedPlatform = "X-CDN-Client-IP"

  router.GET("/", func(c *gin.Context) {
    // 如果设置了 TrustedPlatform，ClientIP() 会解析
    // 对应的请求头并直接返回 IP
    fmt.Printf("ClientIP: %s\n", c.ClientIP())
  })
  router.Run()
}
```

### 测试

使用 `net/http/httptest` 包是进行 HTTP 测试的首选方式。

```go
package main

import (
  "net/http"

  "github.com/gin-gonic/gin"
)

func setupRouter() *gin.Engine {
  r := gin.Default()
  r.GET("/ping", func(c *gin.Context) {
    c.String(http.StatusOK, "pong")
  })
  return r
}

func main() {
  r := setupRouter()
  r.Run(":8080")
}
```

上述代码的测试：

```go
package main

import (
  "net/http"
  "net/http/httptest"
  "testing"

  "github.com/stretchr/testify/assert"
)

func TestPingRoute(t *testing.T) {
  router := setupRouter()

  w := httptest.NewRecorder()
  req, _ := http.NewRequest(http.MethodGet, "/ping", nil)
  router.ServeHTTP(w, req)

  assert.Equal(t, http.StatusOK, w.Code)
  assert.Equal(t, "pong", w.Body.String())
}
```