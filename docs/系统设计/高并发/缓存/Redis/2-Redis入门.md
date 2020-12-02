# 💫 Redis 入门

---

## 1. 概述

**Redis（Remote Dictionary Server )，即远程字典服务**：是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、**Key-Value 数据库**，并提供多种语言的API。与传统数据库不同的是 **Redis 的数据是存在内存中的** ，也就是它是内存数据库，所以读写速度非常快，因此 **Redis 被广泛应用于缓存方向**。

> 🎉 免费和开源！是当下最热门的 NoSQL 技术之一，也被人们称之为**结构化数据库**。
>
> 🏠 官网 [http://www.redis.cn/documentation.html](http://www.redis.cn/documentation.html)

🆚 **Redis 与其他 key - value 缓存产品相比有以下三个特点**：

- Redis 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis 不仅仅支持简单的 key-value 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。
- Redis支持数据的备份，即 **master-slave 主从模式**的数据备份。

👍 **Redis 优势**：

- **性能极高** – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- **丰富的数据类型** – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- **原子** – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- **丰富的特性** – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## 2. Windows 安装

📥 下载地址：[https://github.com/MicrosoftArchive/redis/releases](https://github.com/MicrosoftArchive/redis/releases)

![](https://gitee.com/veal98/images/raw/master/img/20200716201224.png)

🚨 停更很久了，因为 **Redis 都是推荐在 Linux 服务器上搭建**。

具体安装教程见此：👉 [Redis在windows下安装过程](https://www.cnblogs.com/M-LittleBird/p/5902850.html)

## 3. Linux 安装（Ubuntu）

> 😒 在打开虚拟机的时候遇到了 BUG，`drm:vmw_host_log [vmwgfx]] *ERROR* Failed to send host log message`，参考本篇文章解决：[https://blog.csdn.net/mychangee/article/details/104954262](https://blog.csdn.net/mychangee/article/details/104954262)

🏃‍ Ununtu 下安装步骤：

在 Ubuntu 系统安装 Redis 可以使用以下命令:

```
$sudo apt-get update
$sudo apt-get install redis-server
```

如果报错`无法获得锁`，可用如下方法解决：

![](https://gitee.com/veal98/images/raw/master/img/20200716205258.png)

**启动 Redis**

```
$ redis-server
```

![](https://gitee.com/veal98/images/raw/master/img/20200716205800.png)

**查看 redis 是否启动？**

```
$ redis-cli
```

以上命令将打开以下终端：

```
redis 127.0.0.1:6379>
```

127.0.0.1 是本机 IP ，6379 是 redis 服务端口。现在我们输入 PING 命令。

```
redis 127.0.0.1:6379> ping
PONG
```

![](https://gitee.com/veal98/images/raw/master/img/20200716205816.png)

以上说明我们已经成功安装了 redis 🎉。

## 4. 测试性能

`redis-benchmark` 是一个官方自带的压力（性能）测试工具。

redis 性能测试工具可选参数如下：

![](https://gitee.com/veal98/images/raw/master/img/20200716210135.png)

我们来简单测试下：

```powershell
# 测试：100个并发连接 100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

<img src="https://gitee.com/veal98/images/raw/master/img/20200716212127.png" style="zoom:88%;" />

🔎 这些字段的含义：

<img src="https://gitee.com/veal98/images/raw/master/img/20200716212342.png" style="zoom:88%;" />

## 5. 基础知识

**redis 默认有 16 个数据库，默认使用的是第0个**

可以使用 `select `进行切换数据库：

<img src="https://gitee.com/veal98/images/raw/master/img/20200716212654.png" style="zoom: 88%;" />

查看数据库所有的 `key`：

<img src="https://gitee.com/veal98/images/raw/master/img/20200716212821.png" style="zoom:88%;" />

可以看到， `4）name` 便是我们添加的 `key`。

清除当前数据库 `flushdb`

清除全部数据库的内容 `FLUSHALL`

<br>

Redis 是很快的，官方表示，**Redis 是基于内存操作，CPU 不是 Redis 的性能瓶颈，Redis 的瓶颈是根据**
**机器的内存和网络带宽**，既然可以使用单线程来实现，所以 **Redis 是基于单线程的**。

**Redis 将所有的数据全部放在内存中**，如果使用多线程，那么 CPU 的上下文切换是一个耗时的操作。对于内存系统来说，如果**没有上下文切换，多次读写都是在一个CPU上的**，那么效率就是最高的。Redis 便基于此。

> 💡 存取速度：CPU > 内存 > 外存（硬盘）

## 📚 References

- [【狂神说Java】Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB?from=search&seid=3325634079268895938)
- [Redis 安装 - 菜鸟教程](https://www.runoob.com/redis/redis-install.html)