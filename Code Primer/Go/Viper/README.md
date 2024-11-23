### 什么是 Viper？

Viper 是适用于 Go 应用程序（包括 Twelve-Factor App）的完整配置解决方案。它被设计为在应用程序中工作，并且可以处理所有类型的配置需求和格式。它支持

- 设置默认值
- 可以读取 JSON，TOML，YAML，HCL，envfile 和 Java properties 格式的配置文件
- 实时监控和重新读取配置文件（可选）
- 读取环境变量中的配置信息
- 读取远程配置系统（etcd 或 Consul）中的配置信息，并监控配置信息发生改变
- 读取命令行参数中的配置信息
- 读取 buffer 中的配置信息
- 显式设置配置项

可以将 Viper 视为满足您所有应用程序配置需求的注册表。

### 为什么使用 Viper？

在构建现代应用程序时，您无需担心配置文件格式；您想专注于构建出色的软件。Viper 的出现就是为了在这方面给您提供帮助。

Viper 为您执行以下操作：

1. 查找，加载和反序列化 JSON，TOML，YAML，HCL，INI，envfile 或 Java properties 格式的配置文件。
2. 提供一种机制来为您的不同配置选项设置默认值。
3. 提供一种机制来通过命令行参数覆盖指定的选项的值。
4. 提供别名系统，以在不会破坏现有代码的情况下轻松重命名参数。
5. 用户提供了与默认值相同的命令行或配置文件时，可以容易地于区分它们的区别。

Viper 使用以下优先顺序。每个项目优先于其下面的项目：

- 显式调用 Set 方法设置值
- flag（命令行参数）
- env（环境变量）
- config（配置文件）
- key/value 存储
- 默认值

**重要**：Viper 配置项的 Key 不区分大小写。正在讨论是否设置为可选项。

### 怎么将配置项写入 Viper？

#### 安装

```go
go get github.com/spf13/viper
```

#### 建立默认值

一个好的配置系统应该支持默认值。默认值对于 Key 不是必须的，但是如果未通过配置文件，环境变量，远程配置或标志（flag）设置 Key 的值，那么 Key 的默认值很有用。

#### 示例：

```go
viper.SetDefault("ContentDir", "content")
viper.SetDefault("LayoutDir", "layouts")
viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categor
```

#### 读取配置文件

Viper 需要最少的配置，以便它知道在哪里寻找配置文件。Viper 支持JSON，TOML，YAML，HCL，INI，envfile 和 Java Properties 格式的文件。Viper 可以搜索多个路径，但是当前单个 Viper 实例仅支持单个配置文件。Viper 不会默认使用任何配置搜索路径，而会将默认决定留给应用程序。

下面是如何使用 Viper 搜索和读取配置文件的示例。不需要任何特定路径，但至少需要提供一个配置文件的预期路径（见代码 3-5 行）。

```go
viper.SetConfigName("config") // name of config file (without extension)
viper.SetConfigType("yaml") // REQUIRED if the config file does not have the extension in the name
viper.AddConfigPath("/etc/appname/")   // path to look for the config file in
viper.AddConfigPath("$HOME/.appname")  // call multiple times to add many search paths
viper.AddConfigPath(".")               // optionally look for config in the working directory
err := viper.ReadInConfig() // Find and read the config file
if err != nil { // Handle errors reading the config file
  panic(fmt.Errorf("Fatal error config file: %s \n", err))
}
```

您可以处理未找到配置文件的特定情况，如下所示：

```go
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // Config file not found; ignore error if desired
    } else {
        // Config file was found but another error was produced
    }
}

// Config file found and successfully parsed
```

**注意** [自 1.6]：您也可以有一个没有扩展名的文件， 并以编程方式指定格式。对于位于用户 $HOME 目录中的配置文件，没有任何扩展名，如 .bashrc

#### 写入配置文件

从配置文件中读取文件很有用，但有时您希望存储运行时所做的所有修改。为此，有一堆命令可用，每个命令都有自己的用途：

- WriteConfig - 将当前 viper 配置写入预定义路径并覆盖（如果存在）。如果没有预定义的路径，则返回错误。
- SafeWriteConfig - 将当前 viper 配置写入预定义路径。如果没有预定义的路径，则返回错误。如果存在，不会覆盖当前配置文件。
- WriteConfigAs - 将当前 viper 配置写入给定的文件路径。将覆盖给定的文件（如果存在）。
- SafeWriteConfigAs - 将当前 viper 配置写入给定的文件路径。如果存在，不会覆盖给定文件。

根据经验，所有标有 safe 标记的方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。

一个小示例：

```go
viper.WriteConfig() // writes current config to predefined path set by 'viper.AddConfigPath()' and 'viper.SetConfigName'
viper.SafeWriteConfig()
viper.WriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.config") // will error since it has already been written
viper.SafeWriteConfigAs("/path/to/my/.other_config")
```

#### 监控和重新读取配置文件

Viper 支持在运行时让应用程序实时读取配置文件的能力。

需要重新启动服务器才能使配置生效的日子已经一去不复返了，viper 支持的应用程序可以在运行时读取对配置文件的更新，并且不会错过任何更新。

只需告诉 viper 实例 watchConfig。您可以为 Viper 提供一个回调函数，在每次发生更改时运行的该函数。

请确保在调用 WatchConfig() 之前添加了所有配置路径

```go
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
  fmt.Println("Config file changed:", e.Name)
})
```

#### 从 io.Reader 读取配置

Viper 预定义许多配置源（如文件、环境变量、命令行参数和远程 K/V 存储，但您不受他们的约束。您还可以实现自己所需的配置源，并提供给 viper。

```go
viper.SetConfigType("yaml") // or viper.SetConfigType("YAML")

// any approach to require this configuration into your program.
var yamlExample = []byte(`
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
  jacket: leather
  trousers: denim
age: 35
eyes : brown
beard: true
`)

viper.ReadConfig(bytes.NewBuffer(yamlExample))

viper.Get("name") // this would be "steve"
```

#### 覆盖设置

这些可能是来自命令行参数，也可以来自您自己的应用程序逻辑。

```go
viper.Set("Verbose", true)
viper.Set("LogFile", LogFile)
```

#### 注册和使用别名

别名允许由多个键引用单个值

```go
viper.RegisterAlias("loud", "Verbose")

viper.Set("verbose", true) // same result as next line
viper.Set("loud", true)   // same result as prior line

viper.GetBool("loud") // true
viper.GetBool("verbose") // true
```

#### 使用环境变量

Viper 完全支持环境变量。这使 Twelve-Factor App 开箱即用。有五种方法可以帮助使用 ENV：

- AutomaticEnv()
- BindEnv(string…) : error
- SetEnvPrefix(string)
- SetEnvKeyReplacer(string…) *strings.Replacer
- AllowEmptyEnv(bool)

使用 ENV 变量时，必须认识到 Viper 将 ENV 变量视为对大小敏感。

Viper 提供了一种机制，用于尝试确保 ENV 变量是唯一的。通过使用 SetEnvPrefix，您可以告诉 Viper 在从环境变量读取时使用前缀。BindEnv 和AutomaticEnv 都将使用前缀。

BindEnv 采用一个或多个参数。第一个参数是键名称，其余参数是要绑定到此键的环境变量的名称。如果提供了多个，它们将按指定顺序优先。环境变量的名称是大小写敏感。如果未提供 ENV 变量名称，则 Viper 将自动假定 ENV 变量与以下格式匹配：前缀 + “_” + 所有 CAPS 中的键名称。当您显式提供 ENV 变量名称（第二个参数）时，它不会自动添加前缀。例如，如果第二个参数为”id”，Viper 将查找 ENV 变量”ID”。

使用 ENV 变量时，需要注意的一个重要问题是每次访问该值时都会重新读取该值。调用 BindEnv 时，viper 不会固定该值。

AutomaticEnv 是一个强大的帮助器，尤其是当与SerenvPrefix 结合。调用时，viper 将会在发出 viper.Get 请求时，随时检查环境变量。它将应用以下规则。如果使用 EnvPrefix 设置了前缀，它将检查一个环境变量的名称是否与键匹配。

SetEnvKeyReplacer 允许您使用 strings.Replacer 对象将 Env 键在一定程度上重写。如果您想要使用 - 或者其它符号在 Get() 调用中，但希望环境变量使用 _ 分隔符，这非常有用。使用它的示例可以在 viper_test.go 中找到。

或者，您也可以将 EnvKeyReplacer 与 NewWithOptions 工厂函数一起使用。与 SetEnvKeyReplacer 不同，它接受 StringReplacer 接口，允许您编写自定义字符串替换逻辑。

默认情况下，空环境变量被视为未设置，并将回退到下一个配置源。若要将空环境变量视为已设置，请使用 AllowEmptyEnv 方法。

环境变量-示例代码：

```go
SetEnvPrefix("spf") // will be uppercased automatically
BindEnv("id")

os.Setenv("SPF_ID", "13") // typically done outside of the app

id := Get("id") // 13
```

#### 使用 Flags

Viper 能够绑定到 flags。具体来说，viper 支持 Cobra 库中使用的 Pflags。

与 BindEnv 一样，在调用绑定方法时，不会设置该值，而是在访问绑定方法时设置该值。这意味着您可以尽早绑定，即使在 init() 函数中。

对于单个 Flag，BindPFlag() 方法提供此功能。

例如：

```go
serverCmd.Flags().Int("port", 1138, "Port to run Application server on")
viper.BindPFlag("port", serverCmd.Flags().Lookup("port"))
```

您还可以绑定一组现有的 pflags（pflag.FlagSet）：

例如：

```go
pflag.Int("flagname", 1234, "help message for flagname")

pflag.Parse()
viper.BindPFlags(pflag.CommandLine)

i := viper.GetInt("flagname") // retrieve values from viper instead of pflag
```

在 Viper 中使用 pflag 并不阻碍其他包中使用标准库中的 flag 包。pflag 包可以通过导入这些 flags 来处理为 flag 包定义的 flags。这是通过调用一个pflag 包提供的便利函数 AddGoFlagSet() 实现的。

例如：

```go
package main

import (
  "flag"
  "github.com/spf13/pflag"
)

func main() {

  // using standard library "flag" package
  flag.Int("flagname", 1234, "help message for flagname")

  pflag.CommandLine.AddGoFlagSet(flag.CommandLine)
  pflag.Parse()
  viper.BindPFlags(pflag.CommandLine)

  i := viper.GetInt("flagname") // retrieve value from viper

  ...
}
```

#### flag 接口

如果您不使用 Pflags，Viper 提供两个 Go 接口来绑定其他 flag 系统。

FlagValue 表示单个 flag。这是一个说明如何实现此接口的非常简单的示例：

```go
type myFlag struct {}
func (f myFlag) HasChanged() bool { return false }
func (f myFlag) Name() string { return "my-flag-name" }
func (f myFlag) ValueString() string { return "my-flag-value" }
func (f myFlag) ValueType() string { return "string" }
```

一旦您的 flag 实现此接口，您只需告诉 Viper 将其绑定：

```go
viper.BindFlagValue("my-flag-name", myFlag{})
```

FlagValueSet 表示一组 flags。这是一个说明如何实现此接口的非常简单的示例：

```go
type myFlagSet struct {
  flags []myFlag
}

func (f myFlagSet) VisitAll(fn func(FlagValue)) {
  for _, flag := range flags {
    fn(flag)
  }
}
```

一旦您的 flag 集合实现此接口，您只需告诉 Viper 绑定它：

```go
fSet := myFlagSet{
  flags: []myFlag{myFlag{}, myFlag{}},
}
viper.BindFlagValues("my-flags", fSet)
```

#### 远程 Key/Value 存储支持

若要在 Viper 中启用远程支持，请对 viper/remote 包进行空白导入：

```go
import _ "github.com/spf13/viper/remote"
```

Viper 将读取从 Key/Value 存储（例如 etcd 或 Consul ）中的路径检索到的配置字符串（如JSON，TOML，YAML，HCL 或 envfile）。这些值优先级高于默认值，但会被从磁盘，命令行参数（flag）或环境变量检索的配置值覆盖。

Viper 使用 crypt 从 K / V 存储中检索配置，这意味着如果您具有正确的 gpg 密钥，您可以将配置值加密后存储，并可以自动将其解密。加密是可选的。

您可以将远程配置与本地配置结合使用，也可以独立使用。

crypt 有一个命令行帮助程序，您可以用来将配置放入 K / V 存储中。crypt 默认使用在 [http://127.0.0.1:4001](http://127.0.0.1:4001/) 上的 etcd。

```go
$ go get github.com/bketelsen/crypt/bin/crypt
$ crypt set -plaintext /config/hugo.json /Users/hugo/settings/config.json
```

确认已设置值：

```go
$ crypt get -plaintext /config/hugo.json
```

有关如何设置加密值或如何使用 Consul 的示例，请参见 crypt 文档。
https://github.com/bketelsen/crypt

#### 远程 Key/Value 存储示例 - 未加密

etcd

```go
viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json")
viper.SetConfigType("json") // because there is no file extension in a stream of bytes, supported extensions are "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```

Consul
您需要使用具有所需配置的 JSON 值设置 Consul 存储中的 key。例如，创建一个具有 JSON 值得 Consul key/value 存储的 key MY_CONSUL_KEY。

```go
{
    "port": 8080,
    "hostname": "myhostname.com"
}
viper.AddRemoteProvider("consul", "localhost:8500", "MY_CONSUL_KEY")
viper.SetConfigType("json") // Need to explicitly set this to json
err := viper.ReadRemoteConfig()

fmt.Println(viper.Get("port")) // 8080
fmt.Println(viper.Get("hostname")) // myhostname.com
```

Firestore

```go
viper.AddRemoteProvider("firestore", "google-cloud-project-id", "collection/document")
viper.SetConfigType("json") // Config's format: "json", "toml", "yaml", "yml"
err := viper.ReadRemoteConfig()
```

当然，您也可以使用 SecureRemoteProvider

#### 远程 Key/Value 存储示例 - 加密

```go
viper.AddSecureRemoteProvider("etcd","http://127.0.0.1:4001","/config/hugo.json","/etc/secrets/mykeyring.gpg")
viper.SetConfigType("json") // because there is no file extension in a stream of bytes,  supported extensions are "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```

#### 监控 etcd 中的更改 - 未加密

```go
// alternatively, you can create a new viper instance.
var runtime_viper = viper.New()

runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")
runtime_viper.SetConfigType("yaml") // because there is no file extension in a stream of bytes, supported extensions are "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"

// read from remote config the first time.
err := runtime_viper.ReadRemoteConfig()

// unmarshal config
runtime_viper.Unmarshal(&runtime_conf)

// open a goroutine to watch remote changes forever
go func(){
  for {
      time.Sleep(time.Second * 5) // delay after each request

      // currently, only tested with etcd support
      err := runtime_viper.WatchRemoteConfig()
      if err != nil {
          log.Errorf("unable to read remote config: %v", err)
          continue
      }

      // unmarshal new config into our runtime config struct. you can also use channel
      // to implement a signal to notify the system of the changes
      runtime_viper.Unmarshal(&runtime_conf)
  }
}()
```

### 怎么在 Viper 中获取配置项？

在 Viper 中，有几种根据值的类型获取值的方法。存在以下功能和方法：

- Get(key string) : interface{}
- GetBool(key string) : bool
- GetFloat64(key string) : float64
- GetInt(key string) : int
- GetIntSlice(key string) : []int
- GetString(key string) : string
- GetStringMap(key string) : map[string]interface{}
- GetStringMapString(key string) : map[string]string
- GetStringSlice(key string) : []string
- GetTime(key string) : time.Time
- GetDuration(key string) : time.Duration
- IsSet(key string) : bool
- AllSettings() : map[string]interface{}

认识到的一件重要事情是，每个 Get 函数如果找不到值，它将返回零值。为了检查给定键是否存在，提供了 IsSet() 方法。

例如：

```go
viper.GetString("logfile") // case-insensitive Setting & Getting
if viper.GetBool("verbose") {
    fmt.Println("verbose enabled")
}
```

#### 访问嵌套键

访问器方法还接受深度嵌套键的格式化路径。例如，如果加载了以下JSON文件：

```go
{
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}
```

Viper 可以通过传递「.」分隔键的路径来访问嵌套字段：

```go
GetString("datastore.metric.host") // (returns "127.0.0.1")
```

遵守上面建立的优先级规则；搜索路径将遍历其余配置注册表，直到找到为止。

例如，在给定此配置文件的情况下，datastore.metric.host 和 datastore.metric.port 均已定义（并且可以被覆盖）。如果另外在默认设置中定义了 datastore.metric.protocol，Viper 也会找到它。

但是，如果 datastore.metric 被直接赋值覆盖（通过 flag，环境变量，Set() 方法等），则 datastore.metric 的所有子键也都变为未定义状态，它们被较高的优先级配置遮蔽（shadowed）了。

Viper 可以使用路径中的数字访问数组索引。例如：

```go
{
    "host": {
        "address": "localhost",
        "ports": [
            5799,
            6029
        ]
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}

GetInt("host.ports.1") // returns 6029
```

最后，如果存在与分隔的键路径匹配的键，则将返回其值。例如

```go
{
    "datastore.metric.host": "0.0.0.0",
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}

GetString("datastore.metric.host") // returns "0.0.0.0"
```

#### 提取子树

在开发可重用模块时，提取配置的子集并传递给模块通常很有用。这样，模块可以实例化一次，就获取到不同的配置。

例如，应用程序可能出于不同的目的使用多个不同的缓存存储：

```go
cache:
  cache1:
    max-items: 100
    item-size: 64
  cache2:
    max-items: 200
    item-size: 80
```

我们可以将缓存名称传递给模块（例如 NewCache(“缓存1”)，但访问配置键需要奇怪的串联，并且与全局配置的分离更少。

因此，与其这样做，我们不要将 Viper 实例传递给表示配置子集的构造函数：

```go
cache1Config := viper.Sub("cache.cache1")
if cache1Config == nil { // Sub returns nil if the key cannot be found
    panic("cache configuration not found")
}

cache1 := NewCache(cache1Config)
```

注意：始终检查 Sub 的返回值。如果找不到 Key，则返回 nil。

在内部，NewCache 函数可以直接处理 max-items 和 item-size 的键：

```go
func NewCache(v *Viper) *Cache {
    return &Cache{
        MaxItems: v.GetInt("max-items"),
        ItemSize: v.GetInt("item-size"),
    }
}
```

生成的代码易于测试，因为它与主配置结构分离，并且更易于重用（出于同样的原因）。

#### 反序列化

您还可以选择将所有值或特定值解析到 struct、map 和 etc。

有两种方法可以做到这一点：

- Unmarshal(rawVal interface{}) : error
- UnmarshalKey(key string, rawVal interface{}) : error

例如：

```go
type config struct {
  Port int
  Name string
  PathMap string `mapstructure:"path_map"`
}

var C config

err := viper.Unmarshal(&C)
if err != nil {
  t.Fatalf("unable to decode into struct, %v", err)
}
```

如果要解析 Key 本身包含「.」（默认键分隔符）的配置，必须更改分隔符：

```go
v := viper.NewWithOptions(viper.KeyDelimiter("::"))

v.SetDefault("chart::values", map[string]interface{}{
    "ingress": map[string]interface{}{
        "annotations": map[string]interface{}{
            "traefik.frontend.rule.type":                 "PathPrefix",
            "traefik.ingress.kubernetes.io/ssl-redirect": "true",
        },
    },
})

type config struct {
  Chart struct{
        Values map[string]interface{}
    }
}

var C config

v.Unmarshal(&C)
```

Viper 还支持解析到嵌入结构体：

```go
/*
Example config:

module:
    enabled: true
    token: 89h3f98hbwf987h3f98wenf89ehf
*/
type config struct {
  Module struct {
    Enabled bool

    moduleConfig `mapstructure:",squash"`
  }
}

// moduleConfig could be in a module specific package
type moduleConfig struct {
  Token string
}

var C config

err := viper.Unmarshal(&C)
if err != nil {
  t.Fatalf("unable to decode into struct, %v", err)
}
```

Viper 在内部使用
github.com/mitchellh/mapstructure 解析值，默认情况下使用 mapstructure tag。

#### 序列化为字符串

您可能需要将 viper 中保存的所有设置序列化到字符串中，而不是将它们写入文件。您可以将您最喜爱的格式的序列化程序与 AllSettings() 返回的配置一起使用。

```go
import (
    yaml "gopkg.in/yaml.v2"
    // ...
)

func yamlStringSettings() string {
    c := viper.AllSettings()
    bs, err := yaml.Marshal(c)
    if err != nil {
        log.Fatalf("unable to marshal config to YAML: %v", err)
    }
    return string(bs)
}
```

### 使用单个 Viper 实例，还是使用多个 Viper 实例？

Viper 可以开箱即用。无需配置或初始化，就可以使用 Viper。由于大多数应用程序都希望使用单个中央存储库进行配置，因此 viper 包提供了此功能。它类似于单例模式。

在上面的所有示例中，他们都以单例模式风格演示了使用 Viper 的使用方法。

#### 使用多个 Viper 实例

您还可以创建许多不同的 Viper 实例，供应用程序使用。每个都有其独特的配置和值集。每个都可以从不同的配置文件、Key/Value 存储等读取。Viper 包支持的所有函数都镜像为 Viper 上的方法。

例如：

```go
x := viper.New()
y := viper.New()

x.SetDefault("ContentDir", "content")
y.SetDefault("ContentDir", "foobar")

//...
```

当使用多个 Viper 时，由用户管理不同的 Viper。

### 使用 Viper 读取配置文件的模拟示例

使用 Viper 读取配置文件的模拟示例

```go
.
├── configs
│   └── config.yaml
├── go.mod
├── go.sum
└── main.go
```

配置文件：
configs/config.yaml

```go
Server:
  RunMode: debug
  HttpPort: 8080
  ReadTimeout: 60
  WriteTimeout: 60
```

使用 Viper 读取配置文件中的内容，并解码到 struct 中：
main.go

```go
type ServerSetting struct {
  RunMode      string
  HttpPort     string
  ReadTimeout  time.Duration
  WriteTimeout time.Duration
}

var server ServerSetting

func main() {
  // 实例化 viper 实例
  vp := viper.New()
  // 设置配置文件名称
  vp.SetConfigName("config")
  // 设置配置文件类型
  vp.SetConfigType("yaml")
  // 添加配置文件路径
  vp.AddConfigPath("configs/")
  // 读取配置文件内容
  err := vp.ReadInConfig()
  if err != nil {
    panic(fmt.Errorf("Fatal error config file: %s\n", err))
  }
  //  解码 key 到 struct
  err = vp.UnmarshalKey("Server", &server)
  if err != nil {
    panic(fmt.Errorf("unable to decode into struct, %v", err))
  }
  // 打印
  fmt.Printf("Server: %+v\n", server)
}
```

### 总结

本文是 Viper 开源库的 README 的中文翻译，文章内容介绍了什么是 Viper，Viper 包含哪些功能和 Viper 管理配置信息的不同方式的使用方法，以及不同方式之间的优先级顺序。翻译内容难免有不准确的地方，建议对照英文原稿阅读。文章结尾，还给出了一个使用 Viper 读取配置文件的模拟示例。截止发稿，Viper 的最新版本为 v1.7.1，并且作者目前正在收集使用反馈，为开发 v2.0 做准备。