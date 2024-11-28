GORM 是一个流行的 Go 语言 ORM（对象关系映射）库，它简化了与数据库交互的过程。GORM 支持多种数据库，包括 MySQL、PostgreSQL、SQLite 和 SQL Server。下面简述一下 GORM 的常见使用方法。

### 1. 安装 GORM

首先，需要安装 GORM 库，可以通过 `go get` 安装：

```bash
go get -u github.com/jinzhu/gorm
```

此外，还需要安装特定数据库的驱动，例如 MySQL：

```bash
go get -u github.com/jinzhu/gorm/dialects/mysql
```

### 2. 配置数据库连接

首先导入 GORM 和数据库驱动库，并配置数据库连接：

```go
package main

import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql" // MySQL 驱动
    "log"
)

var db *gorm.DB
var err error

func main() {
    // 连接到数据库
    db, err = gorm.Open("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // 其他代码...
}
```

### 3. 定义模型（Struct）

GORM 使用结构体（struct）来定义数据库表模型。结构体字段的名称对应数据库表中的字段：

```go
type User struct {
    ID        uint   `gorm:"primary_key"`
    Name      string `gorm:"size:100"`
    Age       int
    CreatedAt time.Time
    UpdatedAt time.Time
}
```

GORM 会自动根据结构体生成数据库表（如果表不存在），表名默认为结构体名称的小写形式。

### 4. 自动迁移（Auto Migration）

通过 GORM 的 `AutoMigrate` 方法，可以自动创建表或修改表的结构：

```go
db.AutoMigrate(&User{})
```

`AutoMigrate` 会根据结构体字段自动创建数据库表，若已有表，则会更新表结构。

### 5. 增加数据（Create）

GORM 提供了方便的创建记录的功能：

```go
user := User{Name: "John Doe", Age: 30}
db.Create(&user)
```

这样会在数据库中插入一条新的 `User` 记录。

### 6. 查询数据（Read）

GORM 支持多种查询方式，包括查询单条记录、查询多个记录：

```go
// 查询单条记录
var user User
db.First(&user, 1) // 查找 ID 为 1 的用户

// 查询所有记录
var users []User
db.Find(&users)

// 使用条件查询
db.Where("name = ?", "John Doe").First(&user)
```

### 7. 更新数据（Update）

更新数据也很简单，GORM 提供了多种方法：

```go
// 更新单个字段
db.Model(&user).Update("Age", 31)

// 更新多个字段
db.Model(&user).Updates(User{Age: 32, Name: "John Updated"})

// 使用 map 更新
db.Model(&user).Updates(map[string]interface{}{"Age": 33, "Name": "John Mapped"})
```

### 8. 删除数据（Delete）

删除记录可以通过 `Delete` 方法进行：

```go
db.Delete(&user, 1) // 删除 ID 为 1 的用户
```

### 9. 事务（Transaction）

GORM 支持事务，可以通过 `Begin`、`Commit` 和 `Rollback` 来处理：

```go
tx := db.Begin()

if err := tx.Create(&user).Error; err != nil {
    tx.Rollback()
    return err
}
tx.Commit()
```

### 10. 关联查询（Relationships）

GORM 支持一对多、多对多等关联查询：

```go
type Order struct {
    ID     uint
    Amount float64
    UserID uint
    User   User // 外键关联
}

db.Create(&Order{Amount: 100.0, UserID: 1})

// 查询带有关联的记录
var orders []Order
db.Preload("User").Find(&orders)
```

### 11. 高级查询（Advanced Querying）

GORM 支持链式调用，可以进行复杂的查询：

```go
db.Where("age > ?", 25).Order("created_at desc").Limit(10).Find(&users)
```

### 12. 使用原生 SQL

如果 GORM 的功能不满足需求，可以直接执行原生 SQL 查询：

```go
var users []User
db.Raw("SELECT * FROM users WHERE age > ?", 25).Scan(&users)
```

### 总结

GORM 是一个非常强大的 ORM 库，提供了很多功能来简化数据库操作。它的核心理念是通过结构体和数据库表之间的映射，使得开发者能以面向对象的方式操作数据库，极大地提高了开发效率。