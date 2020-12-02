# 📰 Redis 发布订阅

---

## 1. 概述

Redis 发布订阅 (pub/sub) 是一种**消息通信模式**：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。

订阅/发布消息图：

<img src="https://gitee.com/veal98/images/raw/master/img/20200724092858.png" style="zoom: 36%;" />

👨‍💼 三大角色：

- 消息发布者
- 频道 channel
- 消息订阅者

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的
关系：

![](https://gitee.com/veal98/images/raw/master/img/20200724093026.png)

当有新消息通过 `PUBLISH `命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户
端：

![](https://gitee.com/veal98/images/raw/master/img/20200724093107.png)

## 2. Redis 发布订阅命令

下表列出了 redis 发布订阅常用命令：

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | `PSUBSCRIBE pattern [pattern ...]` 订阅一个或多个符合给定模式的频道。 |
| 2    | `PUBSUB subcommand [argument [argument ...]]` 查看订阅与发布系统状态。 |
| 3    | `PUBLISH channel message` 将信息发送到指定的频道。           |
| 4    | `PUNSUBSCRIBE [pattern [pattern ...]]` 退订所有给定模式的频道。 |
| 5    | `SUBSCRIBE channel channel [...]` 订阅给定的一个或多个频道的信息。 |
| 6    | `UNSUBSCRIBE [channel [channel ...]]` 指退订给定的频道。     |

这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等。

## 3. 实例

以下实例演示了发布订阅是如何工作的。在我们实例中我们创建了订阅频道名为 **redisChat**

订阅端：

```powershell
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
# 等待读取推送的信息
```

现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。

发布端：

```powershell
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message" # 消息
2) "redisChat" # 哪个频道的消息
3) "Redis is a great caching technique" # 消息的具体内容
1) "message"
2) "redisChat"
3) "Learn redis"
```

## 4. 原理

Redis 底层是使用 C 实现的，通过分析 Redis 源码里的 `pubsub.c` 文件，了解发布和订阅机制的底层实现，加深对 Redis 的理解。

Redis 通过 `PUBLISH `、 `SUBSCRIBE `等命令实现了订阅与发布模式， 这个功能提供两种信息机制， 分别是订阅/发布到频道和订阅/发布到模式， 下文先讨论订阅/发布到频道的实现， 再讨论订阅/发布到模式的实现。

### ① 订阅频道

每个 Redis 服务器进程都维持着一个表示服务器状态的 `redis.h/redisServer` 结构， 结构的 `pubsub_channels` 属性是一个字典， **这个字典就用于保存订阅频道的信息**：

```c
struct redisServer {
    // ...
    dict *pubsub_channels;
    // ...
};
```

⭐ 其中，**字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端**。

比如说，在下图展示的这个 `pubsub_channels` 示例中， `client2` 、 `client5` 和 `client1` 就订阅了 `channel1` ， 而其他频道也分别被别的客户端所订阅：

<img src="https://gitee.com/veal98/images/raw/master/img/20200724095553.png" style="zoom:80%;" />

当客户端调用 `SUBSCRIBE` 命令时， 程序就将客户端和要订阅的频道在 `pubsub_channels` 字典中关联起来。

举个例子，如果客户端 `client10086` 执行命令 `SUBSCRIBE channel1 channel2 channel3` ，那么前面展示的 `pubsub_channels` 将变成下面这个样子：

![](https://gitee.com/veal98/images/raw/master/img/20200724095636.png)

`SUBSCRIBE `命令的行为可以用伪代码表示如下：

```python
def SUBSCRIBE(client, channels):

    # 遍历所有输入频道
    for channel in channels:

        # 将客户端添加到链表的末尾
        redisServer.pubsub_channels[channel].append(client)
```

通过 `pubsub_channels` 字典， 程序只要检查某个频道是否为字典的键， 就可以知道该频道是否正在被客户端订阅； 只要取出某个键的值， 就可以得到所有订阅该频道的客户端的信息。

### ② 发送信息到频道

了解了 `pubsub_channels` 字典的结构之后， 解释 [PUBLISH](http://redis.readthedocs.org/en/latest/pub_sub/publish.html#publish) 命令的实现就非常简单了： 当调用 `PUBLISH channel message` 命令， 程序首先根据 `channel` 定位到字典的键， 然后将信息发送给字典值链表中的所有客户端。

比如说，对于以下这个 `pubsub_channels` 实例， 如果某个客户端执行命令 `PUBLISH channel1 "hello moto"` ，那么 `client2` 、 `client5` 和 `client1` 三个客户端都将接收到 `"hello moto"` 信息：

<img src="https://gitee.com/veal98/images/raw/master/img/20200724110749.png" style="zoom:80%;" />

`PUBLISH `命令的实现可以用以下伪代码来描述：

```python
def PUBLISH(channel, message):

    # 遍历所有订阅频道 channel 的客户端
    for client in server.pubsub_channels[channel]:

        # 将信息发送给它们
        send_message(client, message)
```

### ③ 退订频道

使用 `UNSUBSCRIBE `命令可以退订指定的频道， 这个命令执行的是订阅的反操作： 它从 `pubsub_channels` 字典的给定频道（键）中， 删除关于当前客户端的信息， 这样被退订频道的信息就不会再发送给这个客户端。

### ④ 模式的订阅与信息发送

当使用 `PUBLISH `命令发送信息到某个频道时， 不仅所有订阅该频道的客户端会收到信息， 如果有某个/某些模式和这个频道匹配的话， 那么所有订阅这个/这些频道的客户端也同样会收到信息。

下图展示了一个带有频道和模式的例子， 其中 `tweet.shop.*` 模式匹配了 `tweet.shop.kindle` 频道和 `tweet.shop.ipad` 频道， 并且有不同的客户端分别订阅它们三个：

![](https://gitee.com/veal98/images/raw/master/img/20200724111256.png)

当有信息发送到 `tweet.shop.kindle` 频道时， 信息除了发送给 `clientX` 和 `clientY` 之外， 还会发送给订阅 `tweet.shop.*` 模式的 `client123` 和 `client256` ：

![](https://gitee.com/veal98/images/raw/master/img/20200724111515.png)

另一方面， 如果接收到信息的是频道 `tweet.shop.ipad` ， 那么 `client123` 和 `client256` 同样会收到信息：

![](https://gitee.com/veal98/images/raw/master/img/20200724111533.png)

### ⑤ 订阅模式

`redisServer.pubsub_patterns` 属性是一个**链表**，链表中保存着所有和模式相关的信息：

```c
struct redisServer {
    // ...
    list *pubsub_patterns;
    // ...
};
```

链表中的每个节点都包含一个 `redis.h/pubsubPattern` 结构：

```c
typedef struct pubsubPattern {
    redisClient *client;
    robj *pattern;
} pubsubPattern;
```

`client` 属性保存着订阅模式的客户端，而 `pattern` 属性则保存着被订阅的模式。

每**当调用 `PSUBSCRIBE` 命令订阅一个模式时， 程序就创建一个包含客户端信息和被订阅模式的 `pubsubPattern` 结构， 并将该结构添加到 `redisServer.pubsub_patterns` 链表中**。

作为例子，下图展示了一个包含两个模式的 `pubsub_patterns` 链表， 其中 `client123` 和 `client256` 都正在订阅 `tweet.shop.*` 模式：

<img src="https://gitee.com/veal98/images/raw/master/img/20200724111849.png" style="zoom:80%;" />

如果这时客户端 `client10086` 执行 `PSUBSCRIBE broadcast.list.*` ， 那么 `pubsub_patterns` 链表将被更新成这样：

![](https://gitee.com/veal98/images/raw/master/img/20200724111930.png)

通过遍历整个 `pubsub_patterns` 链表，程序可以检查所有正在被订阅的模式，以及订阅这些模式的客户端。

### ⑥ 发送信息到模式

发送信息到模式的工作也是由 `PUBLISH `命令进行的， 在前面讲解频道的时候， 我们给出了这样一段伪代码， 说它定义了 `PUBLISH `命令的行为：

```python
def PUBLISH(channel, message):

    # 遍历所有订阅频道 channel 的客户端
    for client in server.pubsub_channels[channel]:

        # 将信息发送给它们
        send_message(client, message)
```

但是，这段伪代码并没有完整描述 `PUBLISH` 命令的行为， 因为 `PUBLISH `除了将 `message` 发送到所有订阅 `channel` 的客户端之外， 它还会将 `channel` 和 `pubsub_patterns` 中的模式进行对比， **如果 `channel` 和某个模式匹配的话， 那么也将 `message` 发送到订阅那个模式的客户端**。

完整描述 `PUBLISH `功能的伪代码定于如下：

```python
def PUBLISH(channel, message):

    # 遍历所有订阅频道 channel 的客户端
    for client in server.pubsub_channels[channel]:

        # 将信息发送给它们
        send_message(client, message)

    # 取出所有模式，以及订阅模式的客户端
    for pattern, client in server.pubsub_patterns:

        # 如果 channel 和模式匹配
        if match(channel, pattern):

            # 那么也将信息发给订阅这个模式的客户端
            send_message(client, message)
```

举个例子，如果 Redis 服务器的 `pubsub_patterns` 状态如下：

![](https://gitee.com/veal98/images/raw/master/img/20200724112201.png)

那么当某个客户端发送信息 `"Amazon Kindle, $69."` 到 `tweet.shop.kindle` 频道时， 除了所有订阅了 `tweet.shop.kindle` 频道的客户端会收到信息之外， 客户端 `client123` 和 `client256` 也同样会收到信息， 因为这两个客户端订阅的 `tweet.shop.*` 模式和 `tweet.shop.kindle` 频道匹配。

### ⑦  退订模式

使用 `PUNSUBSCRIBE `命令可以退订指定的模式， 这个命令执行的是订阅模式的反操作： 程序会删除 `redisServer.pubsub_patterns` 链表中， 所有和被退订模式相关联的 `pubsubPattern` 结构， 这样客户端就不会再收到和模式相匹配的频道发来的信息。

## 📚 References

- [【狂神说Java】Redis最新超详细版教程通俗易懂](https://www.bilibili.com/video/BV1S54y1R7SB?from=search&seid=3325634079268895938)

- [Redis 设计与实现（第一版）](https://redisbook.readthedocs.io/en/latest/index.html)

- [菜鸟教程 — Redis 发布订阅](https://www.runoob.com/redis/redis-pub-sub.html)

  


