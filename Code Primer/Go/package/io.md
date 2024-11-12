Go语言中的`io`包提供了用于基本输入和输出的接口和工具函数，广泛应用于文件读写、网络传输和数据流的处理。它定义了一些核心接口，如`Reader`、`Writer`、`Closer`等，通过这些接口实现了流式数据处理的简单性和灵活性。

### 1. 主要接口

#### 1.1 Reader接口

`Reader`接口用于从数据流中读取数据，定义如下：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

- **Read方法**：读取数据到`p`切片中，返回读取的字节数和错误信息。当返回的错误为`io.EOF`时，表示已到达数据流的末尾。

常用的实现`Reader`接口的类型包括`os.File`、`bytes.Buffer`和`strings.Reader`等。

#### 1.2 Writer接口

`Writer`接口用于向数据流中写入数据，定义如下：

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

- **Write方法**：将数据写入`p`切片，返回写入的字节数和错误信息。

常用的实现`Writer`接口的类型包括`os.File`、`bytes.Buffer`和`net.Conn`等。

#### 1.3 Closer接口

`Closer`接口用于关闭数据流或文件，定义如下：

```go
type Closer interface {
    Close() error
}
```

- **Close方法**：关闭资源并返回错误信息。常见的实现包括文件关闭`os.File.Close`、网络连接关闭`net.Conn.Close`等。

### 2. 组合接口

`io`包还提供了多个组合接口：

- **ReadWriter**：包含`Reader`和`Writer`接口，用于既可以读又可以写的资源。
- **ReadCloser**：包含`Reader`和`Closer`接口，用于可读且可关闭的资源。
- **WriteCloser**：包含`Writer`和`Closer`接口，用于可写且可关闭的资源。
- **ReadWriteCloser**：包含`Reader`、`Writer`和`Closer`接口，用于既可读写又可关闭的资源。

### 3. 工具函数

`io`包中还提供了一些常用的工具函数，简化了对数据流的操作。

#### 3.1 io.Copy

`io.Copy`将数据从一个`Reader`复制到一个`Writer`，直到读取到`EOF`或发生错误。常用于文件复制、网络数据转发等场景。

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

示例：

```go
file, _ := os.Open("input.txt")
defer file.Close()

outputFile, _ := os.Create("output.txt")
defer outputFile.Close()

io.Copy(outputFile, file)
```

#### 3.2 io.CopyN

`io.CopyN`函数类似于`io.Copy`，但只复制指定的字节数`n`。

```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```

#### 3.3 io.ReadFull

`io.ReadFull`确保从`Reader`读取指定的字节数。它会一直读取，直到填满提供的字节切片`p`或发生错误。

```go
func ReadFull(r Reader, buf []byte) (n int, err error)
```

#### 3.4 io.TeeReader

`io.TeeReader`返回一个`Reader`，它会将读取的数据写入`Writer`，但不会影响数据的读取。常用于数据监控或调试场景。

```go
func TeeReader(r Reader, w Writer) Reader
```

示例：

```go
file, _ := os.Open("input.txt")
defer file.Close()

buf := bytes.NewBuffer(nil)
tee := io.TeeReader(file, buf)
io.ReadAll(tee)
fmt.Println(buf.String()) // 输出读取的内容
```

#### 3.5 io.MultiReader 和 io.MultiWriter

- **io.MultiReader**：将多个`Reader`组合成一个`Reader`，按顺序依次读取。
  
  ```go
  func MultiReader(readers ...Reader) Reader
  ```
  
- **io.MultiWriter**：将多个`Writer`组合成一个`Writer`，每次写入会将数据同时写入所有`Writer`中。
  
  ```go
  func MultiWriter(writers ...Writer) Writer
  ```

示例：

```go
r1 := strings.NewReader("Hello, ")
r2 := strings.NewReader("world!")
reader := io.MultiReader(r1, r2)
data, _ := io.ReadAll(reader)
fmt.Println(string(data)) // 输出 "Hello, world!"
```

### 4. 实战示例

以下示例展示了如何将文件内容读取、处理并保存到另一个文件：

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    inputFile, err := os.Open("input.txt")
    if err != nil {
        fmt.Println("打开文件错误:", err)
        return
    }
    defer inputFile.Close()

    outputFile, err := os.Create("output.txt")
    if err != nil {
        fmt.Println("创建文件错误:", err)
        return
    }
    defer outputFile.Close()

    // 使用 io.Copy 将文件内容从输入文件复制到输出文件
    bytesCopied, err := io.Copy(outputFile, inputFile)
    if err != nil {
        fmt.Println("复制文件错误:", err)
        return
    }

    fmt.Printf("成功复制了 %d 字节数据\n", bytesCopied)
}
```

### 总结

`io`包提供了用于流式数据操作的基础接口和工具函数，简化了对数据读写的封装。通过`io.Copy`、`io.TeeReader`等工具函数，可以轻松实现常见的数据流处理需求。配合`io`包的接口和组合接口，可以灵活地实现不同的数据处理流程。