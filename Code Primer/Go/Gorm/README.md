# 基本使用

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

# Callbacks

GORM 的钩子方法（Callbacks）是用于在模型的生命周期中插入自定义逻辑的一种方式。通过这些回调，您可以在模型执行某些操作（如创建、更新、查询和删除）时插入自己的代码逻辑。这些回调方法通常被定义为模型结构体的指针，并在相关操作执行时自动调用。

在 GORM 中，回调方法分为不同的操作阶段，每个阶段都有其对应的回调函数。GORM 提供的常见回调方法如下：

### 1. **创建操作相关回调**：

- **BeforeSave**：在保存（包括创建和更新）之前调用。可以用来做数据验证、修改或日志记录等操作。
- **BeforeCreate**：在创建记录之前调用。通常用于设置字段（如创建时间等）或者进行数据检查。
- **AfterCreate**：在创建记录之后调用。可以用来执行一些后续的操作，比如日志记录或者发送通知等。
- **AfterSave**：在保存（无论是创建还是更新）之后调用。这个回调通常用于对比更新后的数据，执行其他任务等。

### 2. **更新操作相关回调**：

- **BeforeSave**：在保存之前调用，用于处理更新前的数据操作。
- **BeforeUpdate**：在更新记录之前调用。可以在这里做一些针对更新的操作，比如验证数据、更新相关字段等。
- **AfterUpdate**：在更新记录之后调用。可以用于执行一些更新后的操作。
- **AfterSave**：在保存（包括更新）之后调用，通常用于更新后的一些额外处理。

### 3. **删除操作相关回调**：

- **BeforeDelete**：在删除记录之前调用。可以在这里执行一些删除前的清理工作，比如检查依赖项、确认删除操作等。
- **AfterDelete**：在删除记录之后调用。可以用于执行清理操作，如删除与该记录相关的文件、缓存等。

### 4. **查询操作相关回调**：

- **AfterFind**：在查询并找到记录后调用。通常用于在获取到数据后对其进行某些加工或处理。例如，解密数据、转换时间格式等。

### 错误处理：

如果回调函数返回错误，GORM 会中止当前操作并回滚所有更改。这对于确保数据的完整性和一致性非常有用。如果回调没有错误，GORM 将继续执行后续的操作。

### 回调方法定义：

回调方法通常是定义在模型结构体的指针上。例如：

```go
type User struct {
    ID    uint
    Name  string
    Email string
}

func (u *User) BeforeSave(tx *gorm.DB) (err error) {
    // 在保存之前执行的逻辑
    if u.Name == "" {
        return fmt.Errorf("Name cannot be empty")
    }
    return nil
}

func (u *User) AfterSave(tx *gorm.DB) (err error) {
    // 在保存之后执行的逻辑
    fmt.Println("User saved:", u.Name)
    return nil
}
```

在这个例子中，当对 `User` 进行保存操作时，`BeforeSave` 会先执行，检查 `Name` 是否为空；如果为空，则返回错误，阻止保存操作。如果保存成功，`AfterSave` 会被调用，打印出保存的用户信息。

### 小结：

GORM 的回调方法使得在数据库操作过程中，可以轻松插入一些自定义逻辑，提高了应用的灵活性和扩展性。在实际开发中，常用的回调方法包括 `BeforeSave`, `AfterSave`, `BeforeCreate`, `AfterCreate` 等，通过这些钩子可以处理数据验证、数据处理、日志记录等操作。



# 结构体说明

## gorm.DB

```go
type DB struct {
	sync.RWMutex 
    //读写锁
    //作用：用于并发控制，允许多个 goroutine 读取数据，但在写入时会阻塞所有其他的读写操作。这是为了保证多线程环境下的数据库操作安全。
    
	Value        interface{}
    //存储数据库查询的结果，通常是结构体或数据的切片。在进行查询操作时，返回的结果通常存放在这个字段中。
    
	Error        error
    //存储执行数据库操作时产生的错误。如果有错误发生，这个字段会保存错误对象。你可以通过它来检查数据库操作是否成功。
    
	RowsAffected int64
	//表示受影响的行数。执行更新、删除等操作时，该字段会记录操作影响的行数。可以用来判断执行结果是否符合预期。
    
	// single db
	db                SQLCommon
    //存储实现了 SQLCommon 接口的数据库对象，SQLCommon 是 GORM 的数据库接口，用于执行 SQL 查询、更新等操作。它是底层数据库连接的抽象，可以是具体的数据库类型（如 MySQL、PostgreSQL、SQLite 等）。
	blockGlobalUpdate bool
    //这个字段表示是否阻止全局的更新操作。如果为 true，GORM 在执行查询或修改时会阻止全局更新的行为。
    
    
	logMode           logModeValue
    //表示当前的日志模式。GORM 支持不同的日志级别（例如：Silent、Error、Warn、Info 等），可以根据这个字段来控制日志输出的详细程度。
	logger            logger
    //GORM 使用 logger 来记录日志。这个字段允许 GORM 在执行数据库操作时记录详细的日志信息，帮助开发者调试和优化代码。
	search            *search
    //这个字段表示一个搜索结构体，用于帮助 GORM 执行查询时进行条件构建或索引等操作。它通常包含查询条件和排序方式等。
	values            sync.Map
   	//使用 sync.Map 存储一些自定义的键值对。这是为了提高并发性能，sync.Map 是一个并发安全的映射，它允许在多线程环境中安全地读取和写入数据。

	// global db
	parent        *DB
    //指向父 DB 对象。在 GORM 中，数据库操作可以通过父子关系进行继承，如果一个 DB 对象执行了某个操作，它可以将结果传递给它的父 DB 对象。
	callbacks     *Callback
    //作用：用于存储回调函数。GORM 支持在执行某些数据库操作时自动执行回调函数，例如 BeforeCreate、AfterCreate 等。callbacks 允许你在操作前后进行自定义的逻辑处理。

	dialect       Dialect
    //表示当前数据库的方言（Dialect）。GORM 支持多种数据库（如 MySQL、PostgreSQL、SQLite 等），每种数据库有自己的方言。通过 dialect 字段，GORM 可以根据不同的数据库类型调整 SQL 语法和行为。
	singularTable bool
	//一个布尔值，表示是否使用单数表名。在 GORM 中，默认的表名是结构体名称的复数形式。如果这个字段为 true，GORM 会使用单数表名。
	// function to be used to override the creating of a new timestamp
	nowFuncOverride func() time.Time
    //用于覆盖创建时间戳的函数。GORM 会自动管理 CreatedAt 和 UpdatedAt 字段的时间戳，通常使用 time.Now() 生成时间。如果需要自定义时间戳的生成逻辑，可以通过这个字段来实现。
}
```

