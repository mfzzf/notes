`bufio`包是 Go 语言中用于高效处理 I/O 操作的一个重要包。它通过提供缓冲区，减少了频繁的系统调用，特别适合对文件、网络连接等数据源进行高效的批量读写。它内含多种类型和方法，可帮助我们高效地处理流式数据。下面我们将详细介绍 `bufio` 包中的核心类型和方法。

### 1. `bufio` 包中的核心类型

#### 1.1 `bufio.Reader`

`bufio.Reader` 是对一个 `io.Reader` 进行封装，使用缓冲来提高读取效率。

- **结构体定义**：

  ```go
  type Reader struct {
      // 内部实现，封装了一个 io.Reader
      // 使用一个缓冲区对数据进行缓存
  }
  ```

- **初始化**：

  创建一个 `bufio.Reader` 通常使用 `NewReader` 函数：

  ```go
  func NewReader(rd io.Reader) *Reader
  ```

- **常用方法**：

  - `Read(p []byte) (n int, err error)`: 读取 `len(p)` 字节到 `p` 中。若缓冲区为空，会从 `io.Reader` 中读取数据。
  
  - `ReadByte() (byte, error)`: 读取一个字节。若缓冲区为空，则读取一个数据块填充缓冲区。
  
  - `UnreadByte() error`: 撤回上一次读取的字节，回退一个字节。这在多次读取后发现需要回退时非常有用。
  
  - `Peek(n int) ([]byte, error)`: 查看但不消费接下来的 `n` 个字节。如果缓存中字节不足 `n`，返回 `ErrBufferFull`。
  
  - `ReadLine() (line []byte, isPrefix bool, err error)`: 逐行读取，返回一行数据，若 `isPrefix` 为 `true`，表示该行未读完。
  
  - `ReadString(delim byte) (string, error)`: 读取直到遇到 `delim` 为止，并返回一个字符串。
  
  - `ReadBytes(delim byte) ([]byte, error)`: 读取直到遇到 `delim` 为止，并返回字节切片。

#### 示例：`bufio.Reader` 读取文件

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("打开文件失败:", err)
        return
    }
    defer file.Close()

    reader := bufio.NewReader(file)

    // 使用 ReadString 方法按行读取
    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            break
        }
        fmt.Print(line)
    }
}
```

#### 1.2 `bufio.Writer`

`bufio.Writer` 对一个 `io.Writer` 进行封装，使用缓冲区批量写入以提升性能。适用于网络、文件等流式写入场景。

- **结构体定义**：

  ```go
  type Writer struct {
      // 内部实现，封装了一个 io.Writer
      // 使用一个缓冲区对数据进行缓存
  }
  ```

- **初始化**：

  创建一个 `bufio.Writer` 通常使用 `NewWriter` 函数：

  ```go
  func NewWriter(w io.Writer) *Writer
  ```

- **常用方法**：

  - `Write(p []byte) (n int, err error)`: 将字节写入缓冲区。缓冲区满时才会将数据刷新到 `io.Writer` 中。
  
  - `WriteString(s string) (int, error)`: 将字符串写入缓冲区。
  
  - `WriteByte(c byte) error`: 写入一个字节。
  
  - `Flush() error`: 将缓冲区内容刷新到 `io.Writer` 中。这在关闭文件或完成写入后需要调用，以确保数据写出。

#### 示例：`bufio.Writer` 写入文件

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    file, err := os.Create("output.txt")
    if err != nil {
        fmt.Println("创建文件失败:", err)
        return
    }
    defer file.Close()

    writer := bufio.NewWriter(file)

    writer.WriteString("Hello, ")
    writer.WriteString("World!\n")

    // 重要：确保将缓冲区内容写入文件
    writer.Flush()
}
```

#### 1.3 `bufio.Scanner`

`bufio.Scanner` 是一个用于逐行或按定界符分割的简便读取工具，适合快速处理分行或分块的数据。它的缓冲区大小固定，适合轻量级的文本数据处理。

- **初始化**：

  创建一个 `bufio.Scanner` 通常使用 `NewScanner` 函数：

  ```go
  func NewScanner(r io.Reader) *Scanner
  ```

- **常用方法**：

  - `Scan() bool`: 每次调用读取一行或一个分隔段。使用循环 `for scanner.Scan()` 可以逐行读取。
  
  - `Text() string`: 返回最近读取的文本，适合文本行的读取。
  
  - `Bytes() []byte`: 返回最近读取的字节切片，适合字节数据处理。
  
  - `Split(splitFunc SplitFunc)`: 设置分隔函数，如 `ScanLines`、`ScanWords` 等。

#### 示例：`bufio.Scanner` 逐行读取文件

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("example.txt")
    if err != nil {
        fmt.Println("打开文件失败:", err)
        return
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)

    for scanner.Scan() {
        line := scanner.Text()
        fmt.Println(line)
    }

    if err := scanner.Err(); err != nil {
        fmt.Println("读取文件错误:", err)
    }
}
```

### 2. 缓冲区大小控制

`bufio` 默认缓冲区大小为 4096 字节（4KB），可以通过 `NewReaderSize`、`NewWriterSize` 或 `scanner.Buffer` 进行调整。

例如：

```go
reader := bufio.NewReaderSize(file, 8192) // 8KB 缓冲区
writer := bufio.NewWriterSize(file, 8192)
```

`scanner.Buffer` 用于设置 `Scanner` 的缓冲区大小：

```go
scanner := bufio.NewScanner(file)
buf := make([]byte, 4096) // 设置 4KB 缓冲区
scanner.Buffer(buf, 4096)
```

### 3. `bufio` 包使用的注意事项

- **选择合适的缓冲区大小**：默认缓冲区适合大多数场景，但可根据需求调整缓冲区大小。
  
- **记得调用 `Flush`**：`bufio.Writer` 写入数据时，最后调用 `Flush()` 将缓冲区内容写入底层 `io.Writer`。
  
- **错误处理**：`bufio.Scanner` 不会返回错误，但 `scanner.Err()` 方法在读取过程中出错时会返回错误信息。

### 总结

`bufio` 包通过提供 `Reader`、`Writer` 和 `Scanner`，在 Go 中实现了高效的数据流读写。利用缓冲区减少系统调用次数和 I/O 开销，非常适合网络、文件和内存中的批量数据处理需求。掌握 `bufio` 包中的这些基本操作将为你在 Go 语言中进行高效 I/O 操作奠定坚实的基础。