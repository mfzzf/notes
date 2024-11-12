`net` 包是 Go 语言标准库中提供的一组用于网络编程的函数和类型集合，它支持多种网络协议的实现，主要包括 TCP、UDP、IP 地址、域名解析等功能。通过 `net` 包，开发者可以轻松地进行客户端与服务器之间的通信，管理连接、解析地址以及实现不同协议的操作。

下面是对 `net` 包的详细介绍，涵盖其常用功能和用法。

### 1. 地址解析

#### 1.1 解析 IP 地址

- **`net.ParseIP`**：解析字符串形式的 IP 地址并返回 `net.IP` 类型。

  ```go
  ip := net.ParseIP("192.168.1.1")
  fmt.Println(ip) // 192.168.1.1
  ```

- **`net.ParseCIDR`**：解析一个 IP 地址与子网掩码的 CIDR 表示法，并返回 `net.IP` 和 `*net.IPNet` 类型。

  ```go
  ip, network, err := net.ParseCIDR("192.168.1.0/24")
  fmt.Println(ip)         // 192.168.1.0
  fmt.Println(network)    // 192.168.1.0/24
  fmt.Println(err)        // nil
  ```

#### 1.2 解析主机名

- **`net.LookupHost`**：解析给定的主机名，返回 IP 地址列表。

  ```go
  ips, err := net.LookupHost("www.google.com")
  if err != nil {
      fmt.Println("Error:", err)
  } else {
      fmt.Println("IPs:", ips)
  }
  ```

- **`net.LookupIP`**：解析主机名并返回 `[]net.IP` 类型的 IP 地址。

  ```go
  ips, err := net.LookupIP("google.com")
  if err != nil {
      fmt.Println("Error:", err)
  } else {
      fmt.Println("IPs:", ips)
  }
  ```

- **`net.LookupAddr`**：根据 IP 地址反向查找主机名。

  ```go
  names, err := net.LookupAddr("8.8.8.8")
  if err != nil {
      fmt.Println("Error:", err)
  } else {
      fmt.Println("Names:", names)
  }
  ```

### 2. 网络连接与监听

#### 2.1 TCP 连接

- **`net.Dial`**：创建一个网络连接，支持指定协议（如 TCP 或 UDP），以及地址和端口。

  ```go
  conn, err := net.Dial("tcp", "golang.org:80")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()
  fmt.Println("Connected to golang.org")
  ```

- **`net.DialTCP`**：建立一个 TCP 连接并返回 `*net.TCPConn`。

  ```go
  addr, err := net.ResolveTCPAddr("tcp", "golang.org:80")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  conn, err := net.DialTCP("tcp", nil, addr)
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()
  fmt.Println("Connected to golang.org via TCP")
  ```

#### 2.2 UDP 连接

- **`net.DialUDP`**：创建一个 UDP 连接。

  ```go
  addr, err := net.ResolveUDPAddr("udp", "localhost:9999")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  conn, err := net.DialUDP("udp", nil, addr)
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()
  fmt.Println("Connected to UDP server")
  ```

#### 2.3 TCP 监听

- **`net.Listen`**：在指定的地址和端口上监听传入的连接。

  ```go
  listener, err := net.Listen("tcp", ":8080")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer listener.Close()
  fmt.Println("Listening on port 8080")
  ```

- **`net.ListenTCP`**：在 TCP 地址上监听连接。

  ```go
  addr, err := net.ResolveTCPAddr("tcp", ":8080")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  listener, err := net.ListenTCP("tcp", addr)
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer listener.Close()
  fmt.Println("Listening on port 8080")
  ```

#### 2.4 接收连接

- **`listener.Accept`**：接受传入的连接并返回一个新的连接对象。

  ```go
  conn, err := listener.Accept()
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  fmt.Println("Connection received")
  ```

### 3. 网络 I/O

#### 3.1 读取与写入数据

- **`conn.Read`**：从连接中读取数据。

  ```go
  buffer := make([]byte, 1024)
  n, err := conn.Read(buffer)
  if err != nil {
      fmt.Println("Error reading:", err)
  }
  fmt.Println("Received data:", string(buffer[:n]))
  ```

- **`conn.Write`**：向连接写入数据。

  ```go
  message := "Hello, Go!"
  _, err := conn.Write([]byte(message))
  if err != nil {
      fmt.Println("Error writing:", err)
  }
  ```

#### 3.2 设置超时

- **`conn.SetDeadline`**：设置连接的读取/写入超时时间。

  ```go
  conn.SetDeadline(time.Now().Add(5 * time.Second))
  ```

- **`conn.SetReadDeadline`** 和 **`conn.SetWriteDeadline`**：分别设置读超时和写超时。

  ```go
  conn.SetReadDeadline(time.Now().Add(5 * time.Second))
  ```

### 4. 高级功能

#### 4.1 TCP 客户端和服务器

- **TCP 服务器**：通过监听 TCP 连接，接收和处理客户端请求。

  ```go
  listener, err := net.Listen("tcp", ":8080")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer listener.Close()
  
  for {
      conn, err := listener.Accept()
      if err != nil {
          fmt.Println("Error:", err)
          continue
      }
      go handleRequest(conn)
  }
  
  func handleRequest(conn net.Conn) {
      defer conn.Close()
      buffer := make([]byte, 1024)
      n, err := conn.Read(buffer)
      if err != nil {
          fmt.Println("Error:", err)
          return
      }
      fmt.Println("Received data:", string(buffer[:n]))
      conn.Write([]byte("Hello, client"))
  }
  ```

#### 4.2 TCP 客户端

- **TCP 客户端**：与 TCP 服务器建立连接并交换数据。

  ```go
  conn, err := net.Dial("tcp", "localhost:8080")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()
  
  conn.Write([]byte("Hello, server"))
  buffer := make([]byte, 1024)
  n, err := conn.Read(buffer)
  if err != nil {
      fmt.Println("Error reading:", err)
  }
  fmt.Println("Server response:", string(buffer[:n]))
  ```

#### 4.3 UDP 通信

- **UDP 服务器**：接收来自客户端的数据包。

  ```go
  addr, err := net.ResolveUDPAddr("udp", ":9999")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  conn, err := net.ListenUDP("udp", addr)
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()

  buffer := make([]byte, 1024)
  n, clientAddr, err := conn.ReadFromUDP(buffer)
  if err != nil {
      fmt.Println("Error reading:", err)
  }
  fmt.Println("Received data:", string(buffer[:n]), "from", clientAddr)
  ```

- **UDP 客户端**：向 UDP 服务器发送数据。

  ```go
  addr, err := net.ResolveUDPAddr("udp", "localhost:9999")
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  conn, err := net.DialUDP("udp", nil, addr)
  if err != nil {
      fmt.Println("Error:", err)
      return
  }
  defer conn.Close()
  
  conn.Write([]byte("Hello, UDP server"))
  ```

### 5. 总结

`net` 包是 Go 中用于处理网络编程的基础工具，提供了多种网络协议的实现，方便开发者进行网络通信、地址解析、连接管理等操作。无论是进行简单的客户端-服务器通信，还是实现更复杂的网络协议，`net

` 包都能提供高效且简洁的解决方案。