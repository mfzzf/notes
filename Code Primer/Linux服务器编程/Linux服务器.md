## Linux 网络编程 API

## 主机字节序、网络字节序

主机字节序通常是小端字节序，即高位字节在高地址处，低位字节在低地址处。

网络字节序通常是大端字节序，即高位字节在低地址处，低位字节在高地址处。

**机器字节序判断**

```cpp
#include <stdio.h>
void byteorder()
{
	union
	{
		short value;
		char union_bytes[ sizeof( short ) ];
	} test;
	test.value = 0x0102;
	if (  ( test.union_bytes[ 0 ] == 1 ) && ( test.union_bytes[ 1 ] == 2 ) )
	{
		printf( "big endian\n" );
	}
	else if ( ( test.union_bytes[ 0 ] == 2 ) && ( test.union_bytes[ 1 ] == 1 ) )
	{
		printf( "little endian\n" );
	}
	else
	{
		printf( "unknown...\n" );
	}
}

int main(){
	byteorder();
	return 0;
}
```

## 通用socket地址

```cpp
#include <bits/socket.h>

struct sockaddr{
    sa_family_t ss_family;
    char sa_data[14];
}
```

### 宏定义

```cpp
PF_UNIX AF_UNIX UNIX本地协议族
PF_INET AF_INET TCP/IPv4协议族
PF_INET6 AF_INET6 TCP/IPv6协议族
```

但是显然这个不是很好用，所以Linux为各个协议族提供了专门的socket地址结构体。

## 专用socket地址

**Unix本地域协议族**

```cpp
#include <sys/un.h>

struct sockaddr_un
{
    sa_family_t sin_family;
    char sun_path[108];
}
```

**TCP/IP协议族**

```cpp
struct sockaddr_in
{
    sa_family_t sin_family;
    u_int16_t sin_port;
    struct in_addr sin_addr;
}

struct in_addr
{
    u_int32_t s_addr;
}
```

**注意：**

所有专用socket类型的变量在实际使用的时候都需要转换为通用的socket地址`sockaddr`。

`(struct sockaddr * )&address;`

## IP地址转换函数

`inet_pton` 和 `inet_ntop` 是用于处理网络地址转换的函数，通常用于将 IP 地址在文本和二进制格式之间转换。

```cpp
#include <arpa/inet.h>
```



### `inet_pton`

将字符串 IP 转换为二进制格式

**定义**：

```c
int inet_pton(int af, const char *src, void *dst);
```

- **参数**：
  - `af`：地址族，通常是 `AF_INET`（IPv4）或 `AF_INET6`（IPv6）。
  - `src`：指向源字符串的指针，表示文本格式的 IP 地址。
  - `dst`：指向目标内存的指针，存储转换后的二进制地址。

- **返回值**：
  - 成功返回 `1`，失败返回 `0`，若输入的地址格式不正确返回 `-1`。

### `inet_ntop`

将二进制 IP 转换回字符串格式

**定义**：
```c
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

- **参数**：
  - `af`：地址族，通常是 `AF_INET` 或 `AF_INET6`。
  - `src`：指向源二进制地址的指针。
  - `dst`：指向目标字符串的指针，存储转换后的文本格式 IP 地址。
  - `size`：目标缓冲区的大小。

- **返回值**：
  - 成功返回指向目标字符串的指针，失败返回 `NULL`。

### 宏定义

```cpp
#include <netinet/in.h>
#define INET_ADDRSTRLEN 16
#define INET6_ADDRSTRLEN 46
```

用于指定函数`inet_ntop`的参数`cnt`的大小。

## 创建socket

```cpp
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
//protocol default 0
```

### DESCRIPTION

`socket()`  creates an endpoint for communication and returns a file descriptor that refers to that endpoint.  The file descriptor returned by a successful call will be the lowest-num‐bered file descriptor not currently open for the process.

The` domain argument` specifies a communication domain; this selects the protocol family which will be used for communication.  These families are defined in `<sys/socket.h>`.  The  for‐mats currently understood by the Linux kernel include:

```cpp
 Name         Purpose                                    Man page
AF_UNIX       Local communication                        unix(7)
AF_INET       IPv4 Internet protocols                    ip(7)
AF_INET6      IPv6 Internet protocols                    ipv6(7)
```

```cpp
The socket has the indicated type, which specifies the communication semantics.  Currently defined types are:

SOCK_STREAM     Provides sequenced, reliable, two-way, connection-based byte streams.  An out-of-band data transmission mechanism may be supported.

SOCK_DGRAM      Supports datagrams (connectionless, unreliable messages of a fixed maximum length).
```

### RETURN VALUE

​       On success, `a file descriptor` for the new socket is returned.  `On error, -1 is returned`, and errno is set appropriately.
