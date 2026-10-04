# Redis 单线程模型与 RESP 通信协议

## Redis 单线程为何能扛住高并发

### I/O 多路复用与 ae 封装层

* **I/O 多路复用（I/O Multiplexing）**

    * > **定义**：一个线程同时盯着成千上万个 fd，只要其中某一个或几个可读、可写，就把注意力挪过去处理，其余的一律不管。

    * 线程从「忙等」变成「等通知」，CPU 利用率随之提高。

    * 具体实现由操作系统提供，不同系统的实现并不相同。

| 实现 | 适用环境 |
| :--- | :--- |
| `epoll` | Linux 系统下的实现方案 |
| `poll` | 另一些操作系统上作为底层的实现方案 |
| `kqueue` | Unix 及其衍生版本，例如 macOS 底层就是它 |
| `select` | 跨平台方案，基本上大部分操作系统都支持 |

* **ae 封装层（ae.c）**

    * > **定义**：Redis 在各操作系统多路复用实现之上做的二次封装，对外暴露统一 API，让上层业务代码对操作系统差异无感知。

    * 对应的封装文件是 `ae_epoll.c`、`ae_kqueue.c`、`ae_select.c` 等。

    * 编译期判断当前环境：声明 `have_epoll`、`have_kqueue` 等变量逐个判断，支持哪个就引入哪个文件，全都不支持就选 `select`。

    * 不管最终选中哪个实现，对外的 API 都不变，上层只认统一的 `aeApiXxx` 接口。

    * 定位类比：像 Java 中接口与它的多种实现，引入哪个文件就触发哪个文件里的函数。

```text
Redis 业务代码
      │  只认统一 API，不管底下是谁
      ▼
┌──────────────────────────────────────┐
│   ae.c  编译期判断当前操作系统环境      │
│   have_epoll / have_kqueue / have_poll │
└──────────────────────────────────────┘
     │            │            │
     ▼            ▼            ▼
ae_epoll.c    ae_kqueue.c   ae_select.c
     │            │            │
     └────────────┴────────────┘
                  ▼
  真正的 epoll_wait / kqueue / poll / select 系统调用
```

### ae 的三个核心 API

| Redis 统一 API | 等价的 epoll 函数 | 作用 |
| :--- | :--- | :--- |
| `aeApiCreate()` | `epoll_create()` | 在内核里创建实例：一棵红黑树记录所有要监听的 fd，一条链表记录就绪的 fd |
| `aeApiAddEvent(fd, event)` | `epoll_ctl()` | 把某个 fd 挂到实例上，监听它的读事件或写事件；新增对应 `ADD`，删除则把指令改成 `DEL` |
| `aeApiPoll(timeout)` | `epoll_wait()` | 带超时时间等待 fd 就绪，返回值是就绪 fd 的数量 |

* **aeApiCreate**

    * 作用是在内核中创建 epoll 实例，实例内部一棵红黑树记录「要盯着谁」，一条链表记录「谁已经准备好了」。

    * > **提示**：红黑树是 epoll 高效的根基，让「往几万个 fd 的集合里增删查」维持在 $O(\log n)$，而不是每次扫一遍整个数组。

* **aeApiAddEvent**

    * 名称里的 `A` 就是 add（新增），做的是把一个 fd 注册到 epoll 实例上，监听其读事件或写事件。

* **aeApiPoll**

    * 参数是超时时间，等待 fd 就绪后返回；返回 0 表示没有就绪，大于 0 表示有就绪。

* > **易错点**：要记的是 `aeApiXxx` 这组名字本身而不是 `epoll_create`，因为源码里到处走的都是 `aeApiXxx`，看到 `aeApiCreate` 要立刻知道它等价于 `epoll_create`。

### 三个 API 串起来的一条完整链路

* 整体流程只有三步：建实例、注册 fd、等就绪；第三步是循环，永远在等。

```text
server 启动
    │
    ├─(1) aeApiCreate()
    │        └─► 内核里建好一棵红黑树 + 一条就绪链表
    │
    ├─(2) 创建 server socket
    │        └─► 拿到 server fd
    │        └─ aeApiAddEvent(server fd, 读事件)
    │              └─► 从此 server fd 可读 = 有客户端连上来了
    │
    └─(3) aeApiPoll(timeout)
              │
              ├── server fd 就绪 ──► 有新客户端连上来
              │        ├─ accept() 拿到客户端 socket fd
              │        └─ aeApiAddEvent(客户端 fd, 读事件)
              │
              └── 客户端 fd 就绪 ──► 客户端发来请求
                       ├─ 读请求、解析请求、执行命令
                       ├─ 结果写进输出缓冲区并排队
                       └─ 客户端 fd 可写 ──► 把结果真的写回去
```

* **server fd 可读的语义**：凡是有客户端连接进来，server fd 就会可读，这就是「有读事件了」。

* 随着循环进行，连上来的客户端越来越多，被监听的 fd 越来越多，就绪的就不再只是 server socket，也可能是客户端 socket 可读。

* **结论**：整体就是监听 server socket、监听客户端 socket，再监听里面发生的各种事件；server socket 的事是接新客户端并再次监听，普通客户端的事是读请求、处理响应。

---

## 服务启动：初始化阶段的三步

* **入口位置**：Redis 服务启动的入口是 `server.c` 里的 `main` 函数，与 Java 类似，作为整个服务的入口最先执行。

* **initServer 的核心三步**

```c
/* initServer() 的核心三步 */
createEventLoop();             /* 创建事件循环 */
listenToPort(...);             /* 创建 server socket */
createSocketAcceptHandler();   /* 监听 + 绑定处理器 */
```

* **事件循环（event loop）**

    * > **定义**：事件循环就等同于 epoll 实例，也就是那个 I/O 多路复用程序；在 Redis 里把它叫 event loop。

    * `createEventLoop` 底层调 `aeApiCreate`，执行完内核里就准备好一棵红黑树和一条链表。

* **listenToPort**

    * > **易错点**：这个名字有误导性——它不是「监听端口」的意思，本质就是创建 server socket。

    * 两个参数：端口读 `server.port`，配置文件里可配，默认 `6379`；IP 地址同样可在配置文件里配（`bind`），不配默认 `127.0.0.1`，即只监听本地地址。

    * 执行完 server socket 创建出来，并得到对应的 fd。

* **createSocketAcceptHandler**

    * 干两件事：一是调 `aeApiAddEvent` 把 `server.ipfd`（server socket 的 fd）添加到 event loop 上开始监听；二是提前设置好处理器 `acceptTcpHandler`（接收 TCP 请求的处理器）。

    * > **提示**：这里的思想是「提前把处理代码写好」，等于一种回调——先定义在 event loop 上，将来事件真的发生就直接调用，而不是等事件发生再临时去写处理代码。

* 三步走完的状态：实例创建完、server socket 创建完、监听完成、处理器提前定义好，只差事件就绪。

---

## 事件循环与 beforeSleep 钩子

* **beforeSleep（aeSetBeforeSleepEventHandler）**

    * > **定义**：在调用 `aeApiPoll` 之前执行的一个钩子函数；因为调 `wait` 后线程很可能休眠等待 fd 就绪，所以把准备工作放在「sleep 之前」做。

* **aeMain**

    * `aeMain` 才是真正开始等待 fd 的地方，内部是 `while (!stop)` 的死循环，不断处理事件，循环体内调 `aeProcessEvents`。

```c
void aeMain(aeEventLoop *eventLoop) {
    int stop = 0;                 /* 默认 0 == false，代表不用停 */
    while (!stop) {
        aeProcessEvents(eventLoop, AE_ALL_EVENTS | AE_CALL_BEFORE_SLEEP);
    }
}
```

* **aeProcessEvents 的三步骨架**

```c
void aeProcessEvents(aeEventLoop *eventLoop, int flags) {
    /* 第一步：wait 之前先做准备 */
    if (flags & AE_CALL_BEFORE_SLEEP) {
        void (*func)(void) = eventLoop->beforeSleep;
        if (func) func();
    }

    /* 第二步：真正去等待，返回就绪 fd 的数量 */
    numevents = aeApiPoll(eventLoop, timeout, ...);

    /* 第三步：numevents == 0 表示没有就绪，本轮直接跳过 */
    for (i = 0; i < numevents; i++) {
        /* 调用这个事件对应的处理器（回调函数） */
    }
}
```

    * 流程是：先调 beforeSleep，再调 `aeApiPoll` 去 wait，按返回的就绪数量遍历，循环体内调用该事件对应的处理器（即回调函数）。

    * 初始化刚完成时 event loop 里只注册了 server socket 一个 fd，因此就绪必然是 server socket 就绪，被调用的处理器就是 `acceptTcpHandler`。

---

## 三类事件与三个处理器

| 事件类型 | 派发给哪个处理器 | 这个处理器干什么 |
| :--- | :--- | :--- |
| server socket 可读 | `acceptTcpHandler` | 接收新客户端请求，拿到 fd 并注册到 event loop 上 |
| 客户端 socket 可读 | `readQueryFromClient` | 把请求参数读到输入缓冲区、解析成命令并执行，结果放入输出缓冲区、排队等待输出，并通过 `beforeSleep` 触发写事件 |
| 客户端 socket 可写 | `sendReplyToClient` | 把输出缓冲区里的数据一个一个写到客户端 socket |

* **结论**：整个 Redis 网络模型就是 I/O 多路复用加事件派发——不断监听 server socket 与客户端 socket，再把三类事件派发给各自的处理器。

### 接收新连接：acceptTcpHandler

* 触发后干两件事：`accept` 接收客户端 socket 得到其 fd；把这个 fd 注册到 event loop 上再次开始监听。

* 接收后会创建 `connection`（对客户端连接的封装与抽象），并把客户端 socket 的 fd 赋值给它。

* `connSetReadHandler` 与 `createSocketAcceptHandler` 类似，都会注册一个 fd 到实例上，只是这里注册的是客户端 socket。

    * 内部两件事：`aeApiAddEvent` 把客户端 fd 注册上去，并绑定读处理器 `readQueryFromClient`。

    * 叫「读处理器」是因为客户端 socket 发生的事就是发请求过来，处理请求就是要读数据。

* 随着循环推进，event loop 里被监听的 fd 从最初的 server socket 一个，随着客户端接入越来越多。

* > **易错点**：`acceptTcpHandler` 与 `readQueryFromClient` 都是处理读事件的处理器，别搞混——一个处理 server socket 上的读事件，一个处理客户端 socket 上的读事件。

### 读取并解析请求：readQueryFromClient

* **client 实例**：每一个客户端 socket 只要与服务器连接，Redis 就把它封装成一个 client 实例，客户端的所有信息（包括它发来的命令参数）都在这个 client 里。

* **输入缓冲区（query buffer）**

    * > **定义**：client 里的 `c->querybuf` 就是输入缓冲区，用来缓存客户端请求参数。

    * `connRead` 从 connection 里把数据读到 query buffer；客户端传过来的请求是以字节形式传递的。

* **processInputBuffer**：把缓冲区中的字节转成字符串，以空格隔开，封装成一个一个 sds 对象，保存到 argv 数组。

    * 例如 `set name 胡歌` 会变成三个 sds 字符串存在数组里：`set`、`name`、`胡歌`；数组中放的是所有的命令参数。

* **执行链路**

```text
conn  --read-->  querybuf  --parse-->  argv[]  --exec-->  processCommand
              (input buffer)            (3 x sds)   |
                                                +-- lookupCommand : name -> command func
                                                +-- cmd->proc()    : really execute it
                                                +-- addReply()     : result -> output buffer
```

### 写出响应：sendReplyToClient

* **触发方式**：`beforeSleep` 里用迭代器 `listGetIterator` 从头往后遍历 `server.clients_pending_write`，逐个取出待写客户端并调 `connSetWriteHandler` 绑定写处理器 `sendReplyToClient`。

* > **易错点**：客户端 socket 之前已经监听过，这里再监听一次并不重复——之前监听的是**读**事件（客户端有请求来了），这里监听的是**写**事件（要把缓冲区数据写回 socket）。

* 客户端 socket 可写触发就绪后，`sendReplyToClient` 把缓冲区中的数据取出来，一个一个写到客户端 socket，客户端才真正拿到结果。

* **闭环总结**：客户端连接 → server fd 可读交给读处理器 → 读请求、解析成命令、执行、结果进输出缓冲区 → `beforeSleep` 触发写事件 → 写处理器把数据写回 socket。

```text
客户端 connect
     │
     ▼
server fd 可读  ──► acceptTcpHandler ──► accept + 注册客户端 fd
     │
客户端 fd 可读  ──► readQueryFromClient
     │                read -> querybuf -> parse -> exec
     │                addReply -> output buffer -> pending queue
     │                beforeSleep -> connSetWriteHandler
     ▼
客户端 fd 可写  ──► sendReplyToClient ──► buffer 写回 socket
     │
     ▼
客户端拿到结果
```

---

## 命令的查找、执行与输出缓冲区

* **命令与函数的映射**：Redis 内部所有命令都有对应的 command 函数（如 `getCommand`、`setCommand`、`pingCommand`），映射关系放在一个字典里，字典的 key 是命令名称，值是对应的 command 函数。

* **lookupCommand**：从 argv 数组取 0 号元素（命令名称字符串），按名称到字典中找到对应的 command 函数，得到 `cmd`。

* **cmd->proc**：`proc` 是 process 的缩写，表示真正执行该命令函数；`ping` 执行完返回 PONG，`set` 执行完返回 OK，不同命令有不同响应。

* **处理命令的三步**：找命令（`lookupCommand`）→ 执行命令（`cmd->proc`）→ 返回结果（`addReply`）。

* **输出缓冲区的两种形态**

    * > **定义**：`c->buf` 与 `c->reply` 都在客户端里面，都属于**输出缓冲区**；`addReply` 并不是真的把数据写给客户端 socket，只是把结果写进这两个之一。

    * `addReplyToBuffer` 先写 `c->buf`；buf 是一个字节数组，有上限，写满后不往 buf 写，改挂到 `c->reply` 链表——链表理论上没有上限。

* **clients_pending_write 队列**：`addReply` 之后调 `listAddNodeHead` 把客户端加到 `server.clients_pending_write`，这是服务器里定义好的固定队列，用来存放那些等待被写的客户端。

* > **注意**：`readQueryFromClient` 结束时结果并没有真的写出，只是进了输出缓冲区并排进了待写队列，真正写出要等写事件触发。

```c
void readQueryFromClient(connection *conn) {
    client *c = connGetData(conn);

    connRead(c->conn, c->querybuf, ...);  /* 请求字节 --> 输入缓冲区 */
    processInputBuffer(c);                 /* 字节 --> sds --> argv[] */
    processCommand(c);                    /* 找命令 -> 执行 -> 生成响应 */
}

void processCommand(client *c) {
    lookupCommand(c->argv, c->argc);      /* argv[0] --> command 函数 */
    c->cmd->proc(c);                      /* 真正执行命令 */
    addReply(c, ...);                     /* 结果进输出缓冲区 */
}

void addReply(client *c, robj *obj) {
    if (!addReplyToBuffer(c, obj)) {
        /* buf 写满了 --> 不往 buf 写，改挂到 c->reply 链表 */
        addReplyToList(c, obj);
    }
    listAddNodeHead(server.clients_pending_write, c);  /* 排队，还没写出去 */
}
```

---

## 单线程模型的性能瓶颈

* **不是瓶颈的地方**

    * I/O 多路复用机制本身不是瓶颈：它只是接收请求、注册事件，速度非常快。

    * 接收应答也不是瓶颈：同样只是注册一下。

    * 命令执行不是瓶颈：Redis 执行命令是纯内存操作，耗时一般在微秒级别。

    * 结果放进缓冲区、排队也不是瓶颈：都是缓存操作，速度很快。

* **真正的瓶颈是两处网络 I/O**

    * 一是从客户端连接读 socket 并解析：读 socket 的速度受网络、带宽影响，效率相对低；主线程可能一直等到数据读完才能继续执行命令。

    * 二是触发写事件后往客户端 socket 写：同样是网络 I/O，受带宽与网络状态影响。

* **结论**：传统单线程模型下影响性能的不是命令处理和事件监听，而是 I/O——影响性能的永远是 I/O，如 MySQL 里影响性能的是对磁盘的读写，这里则是网络读写。

---

## Redis 6 的多线程改造

* **多线程只接管两块**：命令解析（读取并解析请求）与响应结果的输出（写回客户端）。

```text
Redis 6 之前
----------------------------------------------
主线程 |  read+parse  |  execute  |  write   |
       ^  网络 I/O，瓶颈  ^ 很快    ^ 网络 I/O，瓶颈

Redis 6 之后
----------------------------------------------
          +-- IO thread 1 --+
          +-- IO thread 2 --+  read + parse (in parallel)
主线程    +-- IO thread N --+  write to client (in parallel)
          |  execute        |  由主线程逐个串行执行
----------------------------------------------
```

* **读侧**：高并发下读事件非常多，主线程以轮询形式把多个不同客户端分发给不同线程，由它们并行读取 socket 数据、解析成命令并找到对应命令。

* **执行侧**：真正执行命令仍由主线程一个人完成，命令依然一个个执行，是线程安全的串行执行。

* **写侧**：写事件触发后同样开启多线程，由它们从待写队列里取客户端并写出。

* **效果**：这两块多线程可以大幅提高 Redis 处理客户端的速度，减少网络 I/O 带来的性能影响。

* **结论**：单次请求的响应时间并没有缩短，但整体吞吐量相对单线程模式有不小的提升。

---

## RESP 协议的定位与版本

* **通信协议**

    * > **定义**：客户端与服务端之间相互能看懂对方所发信息的规范；客户端发出的命令和服务端返回的结果都要遵循它。

    * 存在理由：客户端发命令若毫无章法，服务端就解析不出来、无法执行；服务端返回结果若毫无章法，客户端也看不懂。

* **RESP（Redis Serialization Protocol）**

    * > **定义**：Redis 序列化协议，是 Redis 采用的通信协议。

    * Redis 是 C/S 架构：启动起来的 Redis 是服务端，命令行 `redis-cli`、Java 客户端 Jedis、桌面客户端等都属于客户端。

    * 客户端形式不同，但与 Redis 通信的基本流程一致：客户端发一条命令 → 服务端接收、解析、执行、返回结果。

* **版本演进**

    * 该协议在 Redis 1.2 版本就已引入，当时还不太标准。

    * Redis 2.0 时代定为标准，称为 **RESP2**，并一直沿用至今。

    * Redis 6 中把 RESP2 升级为 **RESP3**：变化很大，两者之间不太兼容；RESP3 增加了很多不同的数据类型，并支持 Redis 6 的客户端缓存新特性，RESP2 则不支持。

    * Redis 6 默认使用的依然是 RESP2，学习中也以 RESP2 为准。

---

## RESP 的五种数据类型

* 不管是客户端发送的命令还是服务端返回的响应，传输的数据都必须是五种类型之一；类型由传输数据的**首个字节**标识。

| 类型 | 首字节 | 完整格式 | 一句话说明 |
| :--- | :--- | :--- | :--- |
| 简单字符串 Simple String | `+` | `+<字符串>\r\n` | 结尾换行是结束标志，内容里不能出现 `\r\n` |
| 错误 Error | `-` | `-<异常类型> <详细信息>\r\n` | 只有服务端响应时才会出现 |
| 整数 Integer | `:` | `:<数字>\r\n` | 中间跟的是数字格式的字符串 |
| 多行字符串 Bulk String | `$` | `$<字节长度>\r\n<字符串内容>\r\n` | 按长度读，二进制安全 |
| 数组 Array | `*` | `*<元素个数>\r\n<元素>...` | 元素类型任意，甚至可以嵌套自己 |

* **简单字符串**

    * 格式示例：`+OK\r\n`；末尾的 `\r\n` 是换行符，也是结束标志。

    * 读取方式：从 `+` 往后一直读，碰到 `\r\n` 就结束。

    * > **易错点**：若字符串本身包含 `\r\n`（内容分好几行），读到第一个 `\r\n` 就结束只能读到部分数据，因此单行字符串不允许在字符串里出现换行符，它是**二进制不安全**的，只能是普通字符串。

    * 常见用途：服务端返回的 OK、PONG 这类不含特殊字符的普通字符串结果。

* **错误（Error）**

    * 首字节是减号，其余与简单字符串一致：以 `\r\n` 结尾，中间是异常信息字符串。

    * 格式示例：`-ERR unknown command 'hset'\r\n`；前面是异常类型，中间一个空格，后面是异常的详细信息。

    * 一般情况下**仅服务端响应时出现**，客户端发请求不会发这种类型。

    * Redis 内部把异常分得非常多种，除 ERR 外还有很多别的类型；自己写客户端解析时完全可以根据不同异常类型做不同处理。

* **整数（Integer）**

    * 首字节是冒号，后面是数字格式的字符串，同样以 `\r\n` 结尾，格式示例：`:1\r\n`。

    * 这三种类型整体格式差别不大，只是首字节不同，简单字符串是 `+`、错误是 `-`、整数是 `:`。

* **多行字符串（Bulk String）**

    * 首字节是 `$`，允许字符串内部含有换行符这类特殊字符、允许有多行，因此不能以 `\r\n` 作为结束标识。

    * 读取方式借鉴 SDS：不是靠结束标识，而是**记录字符串占用的字节数**，标识多少就读多少字节，不管中间是什么，从而确保二进制安全。

    * 格式示例 `$5\r\nhello\r\n`：`$` 标记类型，紧跟着的 `5` 是字节大小，后面的 `\r\n` 表示记录长度的部分结束，再往下才是真正的字符串内容。

    * 长度字段的特殊值：正数表示字节大小；`0` 表示空字符串，格式为 `$0\r\n\r\n`；`-1` 表示这个字符串根本不存在（例如按 key 找不到），格式为 `$-1\r\n`。

    * > **提示**：解析时先判断长度——0 和 -1 都可以认为「没有」，不必处理；只有正整数才去读内容。

* **数组（Array）**

    * 首字节是 `*`，后面跟上数组元素个数；元素可以是上述四种中的任意一种，甚至可以是数组自身，因此类型不确定、可以嵌套。

    * 命令示例（数组里装的是 `SET name 胡歌`）：

```text
*3\r\n
$3\r\n
set\r\n
$4\r\n
name\r\n
$6\r\n
胡歌\r\n
```

    * 读法：看到 `*` 后跟 `3`，就往下读三个数据；先读到 `$` 知道是多行字符串，长度 3 读出 `set`，再读到一个长度 4 的 `name`，第三个长度 6 读出 `胡歌`。

    * > **易错点**：长度是**字节数**不是字符数，中文一个字在 UTF-8 中占 3 字节，`胡歌` 两个字必须写 6；若按字符数写成 2，客户端读出来的数据会缺一截。

    * 嵌套示例（最外层三个元素：整数 `1`、长度 5 的多行字符串 `hello`、一个含 `h` 与 `hi` 两元素的内层数组）：

```text
*3\r\n
:1\r\n
$5\r\n
hello\r\n
*2\r\n
$1\r\n
h\r\n
$2\r\n
hi\r\n
```

---

## 客户端与服务端各发什么

* 客户端发命令时命令往往分成几段，因此**多数情况下发送的是数组格式**，且数组中往往都是字符串。

* 服务端响应的数据格式多种多样，五种都可能出现。

| 操作 | 返回的可能是哪种 RESP 类型 |
| :--- | :--- |
| 执行 `SET` | 简单字符串，如 `+OK\r\n` |
| 服务端执行出错 | 错误，如 `-ERR ...\r\n` |
| 执行 `INCR` | 整数，如 `:1\r\n` |
| 执行 `GET` | 多行字符串，为了确保二进制安全 |
| 执行 `MGET` | 数组，因为有多个不同的结果 |

---

## 基于 Socket 自己实现客户端

### 建连接与取两条流

* 与 Redis 交互使用 TCP：客户端这边是 `Socket`，Redis 那边是 Server Socket；建连接后发请求用**输出流**、读响应用**输入流**。

    * > **提示**：建议一开始就提前把两条流拿到手，后面直接用，不用反复获取。

* 三段式流程：

```text
① 建连接   new Socket(host, port)
② 收发     输出流 → 发一条 SET       输入流 → 读回响应
③ 释放     finally 里关掉 reader / writer / socket
```

* `Socket` 需要 `host` 与 `port` 两个参数；为了能在 `finally` 里释放，一般把 `Socket` 定义成静态成员变量而不是局部变量。

* > **注意**：真实场景中 host 与 port 最好可配置，不要写死在代码里，这里定义成参数即可。

* **输出流不直接用字节流**：字节流只能输出字节、比较麻烦，改用 `PrintWriter` 字符流，好处是可以直接按行输出，不用自己拼换行符。

    * `PrintWriter` 内部接收的其实还是字节流，可以自己定义转换流 `OutputStreamWriter` 把字节流传给它并指定 UTF-8 编码。

* **输入流用 BufferedReader 的取舍**：字符流按行读很方便，因为响应里有很多换行符；但多行字符串是二进制安全的，用 `readLine()` 一行一行读很可能读不全，**理论上应该用字节流读**，此处为了省事先用字符流。

* **释放连接**：只要不为 `null` 就逐个 `close()`（reader、writer、socket），放在 `finally` 里；释放过程中的异常此处仅打印不处理。

```java
public class RedisClient {

    private static Socket s;
    private static PrintWriter writer;
    private static BufferedReader reader;

    public static void main(String[] args) throws IOException {
        String host = "192.168.6.101";
        int port = 6379;

        // 建连接
        s = new Socket(host, port);

        // 拿流
        writer = new PrintWriter(
                new OutputStreamWriter(s.getOutputStream(), StandardCharsets.UTF_8));
        reader = new BufferedReader(
                new InputStreamReader(s.getInputStream(), StandardCharsets.UTF_8));

        // 中间：发请求、解析响应
        sendRequest();
        System.out.println(handleResponse());

        // 释放
        try {
            if (reader != null) reader.close();
            if (writer != null) writer.close();
            if (s != null) s.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void sendRequest() throws IOException { }

    private static Object handleResponse() throws IOException { return null; }
}
```

### 按 RESP 格式发送请求

* 发一条 `set name 胡歌` 时，前两行 `*3` 与 `$3` 合起来代表「一个 3 元素的数组」，不是普通字符串。

* 用 `println` 逐行输出即可自带换行符，不必手写 `\r\n`。

```java
writer.println("*3");
writer.println("$3");
writer.println("set");
writer.println("$4");
writer.println("name");
writer.println("$6");
writer.println("胡歌");

// 最后给他刷出去
writer.flush();
```

* > **易错点**：千万不要忘了 `writer.flush()`，刷新之后命令才真正发到 Redis。

* **通用化改造**：规律是数组大小取决于元素个数，元素长度是字符串本身的字节大小，循环体内永远是「先写长度，再写内容」；用可变参数接收任意段数的命令。

```java
private static void sendRequest(String... args) throws IOException {
    writer.println("*" + args.length);
    for (String arg : args) {
        writer.println("$" + arg.getBytes(StandardCharsets.UTF_8).length);
        writer.println(arg);
    }
    writer.flush();
}
```

    * > **易错点**：这里的长度不是字符串长度，而是转成字节之后的字节大小——`胡歌` 字符串长度是 2，字节大小是 6。

### 解析响应：按首字节分支

* 判断方式：类型标识在首字节，所以第一步是读首字节；用的是字符流，而这些特殊标识一个字符刚好一个字节，可以直接 `read()` 读单个字符。

* > **易错点**：`switch` 分支里的字符不能用双引号，要用单引号；这种单字符的协议标识最好定义成常量放在单独的常量类里，写死容易出错；`default` 分支表示不是五种之一，抛异常提示未知类型或数据格式不正确。

```java
char prefix = (char) reader.read();
switch (prefix) {
    case '+': ...
    case '-': ...
    case ':': ...
    case '$': ...
    case '*': ...
    default: ...
}
```

* **三种一次读完的简单类型**

    * `+` 单行字符串：`readLine()` 读一行即可，直接 `return`。

    * `-` 错误：同样读一行，但内容是异常信息，把它作为异常抛出去。

    * `:` 数字：`readLine()` 读一行后还要转成数字。

    * > **提示**：建议转成 `long` 而不是 `int`，`int` 有可能太小、超过最大值；把字符串交给 `Long.parseLong()` 会自动转成 `long` 返回。

```java
case '+':
    return reader.readLine();
case '-':
    throw new RuntimeException(reader.readLine());
case ':':
    return Long.parseLong(reader.readLine());
```

* **多行字符串：先读长度再决定读什么**

    * `$` 前缀已经被 `read()` 取走，所以此时 `readLine()` 读到的刚好是长度数字，用 `Integer.parseInt` 转成 int。

    * 长度有三种情况：`-1` 表示不存在，返回 `null`；`0` 表示空字符串（与 `-1` 不一样）；正数才是字符串长度。

    * > **注意**：既然是二进制安全的，正确做法应是读 N 个**字节**作为结果，这样中间有换行符也没关系；但这里只剩字符流、没法再读字节，于是简化假设没有特殊字符、只读一行。

```java
case '$':
    int len = Integer.parseInt(reader.readLine());
    if (len < 0) return null;
    if (len == 0) return "";
    // 偷懒：假设没有特殊字符，直接读一行
    return reader.readLine();
```

* **数组：靠递归读回来**

    * `*` 后面的那一行 `readLine()` 读到元素个数，同样 `Integer.parseInt` 处理；`size <= 0` 直接返回 `null`，大于 0 才去读元素。

    * 每个元素都是一个完整数据，可能是五种中的任意一种（包括数组），所以循环调用 `handleResponse()` 即可，等于一次**递归**。

    * > **提示**：读出来的元素类型不确定，集合要用 `List<Object>`。

```java
case '*':
    int size = Integer.parseInt(reader.readLine());
    if (size <= 0) return null;
    List<Object> list = new ArrayList<>(size);
    for (int i = 0; i < size; i++) {
        list.add(handleResponse());   // 递归：每个元素再来一遍五种判断
    }
    return list;
default:
    throw new RuntimeException("未知类型，数据格式不正确");
```

* **跑起来要先授权**：直接发命令会得到 `NOAUTH Authentication required`——解析到的正是一个 Error 类型响应，说明解析逻辑是对的。

    * 需要先发 `auth <密码>` 再发业务命令；发几次请求就要解析几次响应，不能只解析一次。

```java
sendRequest("auth", "123321");
System.out.println(handleResponse());

sendRequest("set", "name", "胡歌");
System.out.println(handleResponse());
```

    * 通用化之后可发任意段数命令，`get name` 传一个参数即可，`MGET` 可一次传多个 key 并一次性取回全部结果。

---

## 过期时间的记录方式

* **redisDb 的两个核心字典**

    * `redisDb` 是 Redis 数据库的结构体，默认有 0 到 15 共 16 个库，每一个库就是一个 `redisDb` 结构体，`id` 是数据库标识。

    * `dict` 是键空间（keyspace），保存库里**所有** key-value；`expires` 只保存**设置了 TTL 的那些 key**。

```text
redisDb（0 号库）
│
├─ id = 0                          数据库标识，0~15
│
├─ dict  ── 键空间 keyspace
│    保存库里【所有】key-value
│      key  : 指向 redisObject 的指针（如 name）
│      value: 指向 redisObject 的指针（如 "胡歌"）
│
└─ expires
     只保存【设置了 TTL】的 key
      key  : 和上面 dict 的 key 一致
      value: 一个数字，存活时间（TTL）
```

    * `dict` 的类型也是 `dict`，可以看作哈希表；key 和 value 最终都是 `redisObject`，只是类型不同。

    * > **易错点**：`dict` 里存的不是真正的 key 本身，而是 `redisObject` 对应的**内存地址指针**。

    * `expires` 的键与 `dict` 的键一致，值不是 value 而是该 key 对应的 TTL 存活时间，两者恰好形成键值映射。

| key | `dict`（keyspace，一定有） | `expires`（设了 TTL 才有） |
| :--- | :--- | :--- |
| `name`（设了 5s） | 有 → 指向字符串对象 | 有 → 存活时间 5000 |
| `age`（没设过期时间） | 有 → 指向字符串对象 | **没有** |

* **判定方式**：想知道某个 key 有没有过期，只要到 `expires` 字典里按 key 查出 TTL 再做时间判断即可。

* **其余属性**：`redisDb` 里还有 `blocking keys`、`ready keys`、`watched keys` 等 dict 类型属性，用来记录具有特殊功能的 key（如乐观锁监视的 key、阻塞式操作等待的 key）；另有一个记录所有过期 key 平均 TTL 的属性，用于统计。

* **结论**：并不是每个 key 都有过期时间，因此 `expires` 的元素个数小于等于 `dict`；`set` 一个未设过期时间的键值对只存在于第一个字典，设置了过期时间则两个字典里都有。

---

## 过期键的删除策略

* **立即删除不可行**：要做到到期那一刻立刻删除，必须监视每一个 key 并给每个 key 设定时器；key 达到数十万甚至数百万时，定时器会给 CPU 带来极大压力，严重影响 Redis 服务本身的性能，无法接受。

* 实际采用的是**惰性删除**与**周期删除**两种。

### 惰性删除

* > **定义**：不在 key 到期后立即删除，而是当**访问它**的时候再判断有没有到期，到期了才删。

* 触发时机是增删改查任意一种操作；不论哪种操作都要**先找到 key** 再检查，所以判断挂在查到 key 的那一刻。

* 代码上读写各有 `lookupKeyRead` 与 `lookupKeyWrite`，二者共同点是都会先执行 `expireIfNeeded`，只有检查完过期没问题才真正找 key 并返回。

* `expireIfNeeded` 的逻辑很简单：判断这个 key 是否过期，过期就 `delete` 掉，判断依据是到 `expires` 字典里查并做时间比较。

* > **易错点**：惰性删除有漏洞——一个 key 已过期但很长时间没人访问，就不会被检查、也不会被删除；极端情况下大量过期 key 可能永远不被删除。

### 周期删除

* > **定义**：一个定时任务，周期性地**抽样**部分过期的 key 执行删除。

* > **注意**：总共就**一个**定时任务，不是给每个 key 各建一个；也不是把数据库里的 key 全部检查一遍（数百万上千万根本检查不过来），而是抽样一批看是否过期。

* 抽样会随着任务推进不断遍历数据库中不同的 key，直到把所有 key 都遍历一遍，因此可以确保一个 key 只要过期早晚会被抽到、早晚被删掉。

* **定时任务的建立**：`initServer` 中通过 `createTimeEvent` 创建定时器，第一次在 1ms 后执行（初始化后先立即执行一次），执行的任务是 `serverCron`。

* **执行间隔**：`serverCron` 返回值即下次执行的间隔，取决于 `server.hz`（赫兹），默认 10。

    * $$\text{下次执行间隔} = 1000 \div \text{server.hz} = 1000 \div 10 = 100\ \text{ms}$$

    * `server.hz` 可以配置；除初始化后那一次立即执行外，之后每隔 100ms 执行一次。

* **serverCron 里的两件事**

    * 一是 get 时钟再 set 给 `server.eloClock`：这是 Redis 服务器内部维护的时钟，记录以**微秒**为单位的当前时间，每隔一个周期（100ms）记录一次，数值不断变化。

    * 二是 `databasesCron`：真正执行数据库清理，内部调用 `activeExpireCycle`（激活一个清理循环），函数内部有循环，不是做一次就完。

* **抽样的执行流程**：遍历 16 个 db → 遍历字典数组里的 bucket（数组角标，每个角标上是一条链表）→ 分批遍历，进度记录到全局变量以便下次接着执行 → 凑够 20 个 key 就逐个判断是否过期，过期删掉、没过期跳过。

```text
遍历 16 个 db
  └─ 遍历字典数组里的 bucket（数组角标）
       ├─ 从 bucket 里抽 key，不够 20 个就换下一个 bucket
       ├─ 逐个判断是否过期，过期的删掉，没过期的跳过
       ├─ 耗时 >= 25ms ？      ──是──► 结束
       ├─ 过期比例 <= 10% ？   ──是──► 结束
       └─ 都不满足 ──► 再抽一轮，循环重复
```

    * 后台有全局统计，统计过期 key 数量与总 key 数量的比例；删得越多，过期 key 的比例越低。

    * **继续抽样的条件是两个都不满足**：耗时未达上限**且**过期比例仍大于 10%，才会再抽一轮；任一满足就结束。

### slow 模式与 fast 模式

* 两种模式业务流程完全一样——同样遍历 db、遍历 bucket、抽样判断、再做时间与比例判断；差别只在执行周期和单次执行时长。

| 对比项 | `slow` 模式 | `fast` 模式 |
| :--- | :--- | :--- |
| 挂在哪儿 | 定时任务 `serverCron` | 事件循环每轮的 `beforeSleep` |
| 执行间隔 | 每 100ms 一次（受 `server.hz` 影响，默认 10） | 每次循环都来，但两次间隔不得小于 2ms |
| 单次时长上限 | 不超过 25ms（周期的 25%） | 不超过 1ms |
| 提前结束条件 | 耗时 ≥ 25ms 或过期比例 ≤ 10% | 耗时 ≥ 1ms 或过期比例 ≤ 10% |
| 清理力度 | 一次多抽点，清理更彻底 | 每次轻一点，抽到即止 |

* **slow 模式**：一秒最多执行十次，周期 100ms；但 100ms 并非全部用于清理，真正清理的耗时不能超过周期的 25%。

    * $$\text{单次清理时长上限} = \text{周期} \times 25\% = 100\ \text{ms} \times 0.25 = 25\ \text{ms}$$

    * 剩余时间什么都不做，直接跳过，确保执行频率固定、不超过每秒十次，不对主线程造成太多影响。

    * 25ms 是**最多**，不是每次都跑满；20 个 key 的判断通常很快，快的时候几毫秒甚至几微秒就结束。

* **fast 模式**：每次执行完都要判断与上次执行的间隔是否超过 2ms，不足 2ms 就跳过；单次清理耗时要求不超过 1ms，1ms 内可能进行多轮，但两个条件任一不满足同样结束。

* **在事件循环里的位置**

```text
initServer()
  建 server socket、建事件循环、注册事件源、注册定时任务 serverCron
        │
        ▼
aeMain()  ── while (true) 不断转
        │
        ├─► beforeSleep()   ──► activeExpireCycle(FAST)
        │       每轮都跑，单次不超过 1ms
        │
        ├─► aeApiPoll()     ──► 等事件就绪 → 处理 IO（执行 Redis 命令）
        │
        └─► serverCron()    ──► activeExpireCycle(SLOW)
                先判断"到点没"，默认 100ms 一次，到了才跑，最多 25ms
```

    * 外循环频率非常高，可能 1ms 循环一次，快的时候几十微秒、几百微秒就循环一次；若每次循环都直接调 `serverCron`，其执行频率会变得非常高，所以每次执行前要先检查时间，确保 `serverCron` 每隔 100ms 才执行一次。

    * 而 `fast` 模式每次循环都会调，while 循环的频率是多少它就执行多少次。

    * `slow` 模式耗时可能达几十毫秒，若每轮都执行会严重阻塞主线程，所以低频执行；`fast` 模式耗时一般非常短（几百微秒甚至几十微秒），所以高频执行。

* **结论**：`slow` 属于低频、长时长的清理，清理效果更好、清理的 key 更多；`fast` 属于高频、少量清理，每次轻一点、最长不超过 1ms；两者场景不同，最终目的都是在不阻塞主线程的前提下尽可能多地清理过期 key。
