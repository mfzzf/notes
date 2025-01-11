### Redis 相关信息

#### 端口号
6379

#### 持久化方式

1. **RDB**
    - **将内存数据，以二进制写入磁盘**
    - **触发持久化的命令**：
        - **save**：主线程阻塞，生产环境慎用！！！
        - **bgsave**：fork()调用子进程，只有fork()调用创建子进程期间是阻塞的。
    - **自动触发**：
        - **save m n**：在m秒里面，有n个键发生改变，触发自动持久化。
    - **清空 Redis 数据库**：
        - **flushall**
2. **AOF**：文件追加方式
3. **混合持久化方式**
    - 混合持久化是结合了 RDB 和 AOF 的优点，在写入的时候，先把当前的数据以 RDB 的形式写入文件的开头，再将后续的操作命令以 AOF 的格式存入文件，这样既能保证 Redis 重启时的速度，又能减低数据丢失的风险。

#### 配置说明
```bash
# RDB 保存的条件
save 900 1
save 300 10
save 60 10000
# bgsave 失败之后，是否停止持久化数据到磁盘，yes 表示停止持久化，no 表示忽略错误继续写文件。
stop-writes-on-bgsave-error yes
# RDB 文件压缩
rdbcompression yes
# 写入文件和读取文件时是否开启 RDB 文件检查，检查是否有无损坏，如果在启动是检查发现损坏，则停止启动。
rdbchecksum yes
# RDB 文件名
dbfilename dump.rdb
# RDB 文件目录
dir./
```

**① save 参数** 它是用来配置触发 RDB 持久化条件的参数，满足保存条件时将会把数据持久化到硬盘。 默认配置说明如下：

- save 900 1：表示 900 秒内如果至少有 1 个 key 值变化，则把数据持久化到硬盘；
- save 300 10：表示 300 秒内如果至少有 10 个 key 值变化，则把数据持久化到硬盘；
- save 60 10000：表示 60 秒内如果至少有 10000 个 key 值变化，则把数据持久化到硬盘。

**② rdbcompression 参数** 它的默认值是 `yes` 表示开启 RDB 文件压缩，Redis 会采用 LZF 算法进行压缩。如果不想消耗 CPU 性能来进行文件压缩的话，可以设置为关闭此功能，这样的缺点是需要更多的磁盘空间来保存文件。 

**③ rdbchecksum 参数** 它的默认值为 `yes` 表示写入文件和读取文件时是否开启 RDB 文件检查，检查是否有无损坏，如果在启动是检查发现损坏，则停止启动。

配置查询

127.0.0.1:6379> config get dbfilename
1) "dbfilename"
2) "dump.rdb"

127.0.0.1:6379> config get dir
1) "dir"
2) "/data"

配置设置 

config set dir "/usr/data" 用于修改 RDB 的存储目录。

**注意**：修改 Redis 配置文件重启 Redis 服务器设置参数不会丢失，而使用命令修改的方式，在 Redis 重启之后就会丢失。但手动修改 Redis 配置文件，想要立即生效需要重启 Redis 服务器，而命令的方式则不需要重启 Redis 服务器。

RDB优点：快 它是一个紧凑的文件，可以更快的传输到远程服务器进行 Redis 服务恢复；主进程并不会执行磁盘 I/O 等操作；RDB 文件可以更快的重启。

缺点：只能保存某个时间间隔的数据，会丢失一段时间内的 Redis 数据

​	fork子进程的时候，如果数据集很大，fork时间很长

并且如果数据集很大且 CPU 性能不佳，则可能导致 Redis 停止为客户端服务几毫秒甚至一秒钟



AOF

127.0.0.1:6379> config get appendonly
1) "appendonly"
2) "no"

### 禁用持久化

禁用持久化可以提高 Redis 的执行效率，如果对数据丢失不敏感的情况下，可以在连接客户端的情况下，执行 `config set save ""` 命令即可禁用 Redis 的持久化，如下图所示：

![image.png](assets/2020-02-24-122636.png)

如果 Redis 服务器 CPU 占用过高，可能是什么原因导致的？

数据集过大 正在进行fork调用

当子进程（用于 `bgsave`）读取内存数据并写入 RDB 文件时，由于只是读取操作，不会触发写时复制。

### AOF 文件重写

AOF 是通过记录 Redis 的执行命令来持久化（保存）数据

例如，我们增加了一个计数器，并对它做了 99 次修改，如果不做 AOF 重写的话，那么持久化文件中就会有 100 条记录执行命令的信息，而 AOF 重写之后，之后记录一条此计数器最终的结果信息，这样就去除了所有的无效信息。

触发 AOF 文件重写，要满足两个条件，这两个条件也是配置在 Redis 配置文件中的，它们分别：

- auto-aof-rewrite-min-size：允许 AOF 重写的最小文件容量，默认是 64mb 。
- auto-aof-rewrite-percentage：AOF 文件重写的大小比例，默认值是 100，表示 100%，也就是只有当前 AOF 文件，比最后一次（上次）的 AOF 文件大一倍时，才会启动 AOF 文件重写。

127.0.0.1:6379> config get auto-aof-rewrite-min-size
1) "auto-aof-rewrite-min-size"
2) "67108864"

```bash
# 允许 AOF 重写的最小文件容量
auto-aof-rewrite-min-size 64mb
```

127.0.0.1:6379> config get auto-aof-rewrite-percentage

1) "auto-aof-rewrite-percentage"
2) "100"

只有同时满足 auto-aof-rewrite-min-size 和 auto-aof-rewrite-percentage 设置的条件，才会触发 AOF 文件重写

```bash
# 是否开启启动时加载 AOF 文件效验，默认值是 yes，表示尽可能的加载 AOF 文件，忽略错误部分信息，并启动 Redis 服务。
# 如果值为 no，则表示，停止启动 Redis，用户必须手动修复 AOF 文件才能正常启动 Redis 服务。 
aof-load-truncated yes
```

- 如果只开启了 AOF 持久化，Redis 启动时只会加载 AOF 文件（appendonly.aof），进行数据恢复；
- 如果只开启了 RDB 持久化，Redis 启动时只会加载 RDB 文件（dump.rdb），进行数据恢复；
- 如果同时开启了 RDB 和 AOF 持久化，Redis 启动时只会加载 AOF 文件（appendonly.aof），进行数据恢复。

![image.png](assets/2020-02-24-122534.png)

**优先加载 AOF 文件**。这是因为 AOF 文件通常包含比 RDB 文件更新的数据。如果 AOF 文件丢失或损坏，Redis 才会尝试加载 RDB 文件。