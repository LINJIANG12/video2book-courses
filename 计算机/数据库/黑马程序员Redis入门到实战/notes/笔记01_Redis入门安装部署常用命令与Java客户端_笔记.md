# Redis 入门：安装部署、常用命令与 Java 客户端

## Redis 与 NoSQL 的定位

### 键值型数据库

* > **定义**：内存中的数据以 key-value 对的形式组织，key 用来定位，value 就是这条数据本身

* Redis 没有表，也没有约束，存的都是键值对

* value 可以是字符串，也可以是 List、有序集合、无序集合、哈希表等结构；value 结构丰富、功能丰富，是 Redis 在企业里用得比较多的一个重要原因

* 把用户拆成 id、name、age 一个个键值对存会让数据显得松散，本质是一个用户却被打散了。

* 常见做法是把一个用户的多个字段组装成一个 JSON 字符串作为 value，key 用用户的 id；value 变长变复杂，但它依然是字符串

* > **提示**：Redis 的 key 一般就是一个 String 字符串，值的类型却是多种多样

### NoSQL 的四种常见形态

* **键值型（Key-Value）**：代表就是 Redis；key 可自定义，value 类型也可自定义，约束相对小很多

* **文档型（Document）**：一条数据就是一条 JSON 文档，对应一行数据；字段可以任意，插入一条是 id、name、age，再插一条改成 id、username 也没问题，字段约束非常松散

* **列类型（宽列模型）**：典型代表是 HBase

* **图类型（Graph）**：存入的每条数据看成一个节点，节点之间有联系（如师生关系、朋友关系）就叫图；一般不做社交类应用时用得非常少

* 四种形态的共同点是数据结构都没有严格要求、比较松散；后续加一个字段、少一个字段都可以，影响相对较小

* > **注意**：说 NoSQL「非结构化」，并不是说它完全没有约束，约束强不强取决于你用的是哪一种 NoSQL 数据库

### 结构化与非结构化

* > **定义**：结构化（Structure）指数据有固定的格式要求，这些要求由表和表的约束来定义

* 约束举例：给 id 加主键约束、给 name 加 unique 唯一约束、给 age 加 unsigned 无符号约束

* 还可以定义数据类型与长度：id 为 BIGINT 长度 20 以内、name 为 VARCHAR 长度 32 以内、age 为 INT 长度 3 以内，这样能节省一定的内存空间

* 约束一旦定义好，表结构就固定了，插入的数据必须严格遵循，数据库也会做校验，不符合就报错、不允许插入

* > **注意**：表结构一般不建议随便修改，最好在项目设计之初就定义好；数据达到上千万规模时改某个字段可能导致表被锁很长一段时间不可用，且表变了业务往往也要跟着变

* NoSQL 存储的是非结构化数据，对数据的结构没有非常严格的约束

### 数据关联性

* > **定义**：关系型（Relational）指数据与数据之间往往有关联，关联靠外键建立，并由数据库自己维护与检查

* 举例：订单表记录 user id 与 goods id，分别外键关联用户表和商品表的主键，两张表因此产生关系

* 外键关系建立后，删除一个用户或商品就不再被允许，数据库会认为它在别的表里有关联并做检查

* 外键关联的好处是节省数据存储空间：订单里只关联一个用户 id，而不是把完整用户信息保存一份

* NoSQL 没有外键，数据与数据之间不直接维护关联

* NoSQL 里维护「一个用户下了多少单、每个单里有什么商品」的常见做法是 JSON 嵌套：文档本身代表一个用户，下面挂 orders 数组，每项是一个订单，订单里再记录 item 商品信息

* 嵌套形式的第一个特点是没有关联：没有表，也没有表与表之间的外键关系

* > **易错点**：嵌套形式的第二个特点是数据重复——张三买了荣耀 6，李四也可以买，这件商品的信息就要在多个用户的文档里各存一份

* 若改成只记录一个商品 id、再单独存一份商品文档，这层关系就只能靠程序员用业务逻辑自己维护，因为数据库本身不会维护表与表之间的关联

### 查询方式

* 关系型数据库基于 SQL 查询，语法格式固定：`SELECT 字段 FROM 表 WHERE 条件`

* > **提示**：语法固定带来的好处是只要是关系型数据库就能用相同的语句查询，MySQL、Oracle 通用

* NoSQL 是非 SQL 查询，没有固定语法、格式不统一

```text
Redis         GET key                    —— 像命令，一个命令就够，key 是字符串
MongoDB       db.user.find({ id: 1 })    —— 像函数调用
ElasticSearch GET /users/1               —— 像 HTTP 的 RESTful 请求
```

* NoSQL 查询方式的优点是相对简单、没有复杂语法需要学习；缺点是不统一，每个不同的库都要学各自的语法

### 事务

* > **定义**：事务必须满足 **ACID** 特性——原子性、一致性、隔离性等。

* 关系型数据库底层都可以帮我们实现 ACID，可以认为所有关系数据库都是满足 ACID 的。

* NoSQL 要么没有事务，要么无法满足事务的强一致性，只能做一些基本的一致性

* > **定义**：无法满足或无法全部满足 ACID 时，称之为 **BASE**，即基本的一种满足

* 对数据安全性要求较高、要满足 ACID 时，应优先选择关系型数据库而不是 NoSQL

### 存储方式与扩展性

* 关系型数据库大多采用磁盘存储，核心数据都在磁盘上

* NoSQL 大多将数据存在内存里，好处是查询性能非常高，对性能要求较高的场景可以使用

* > **定义**：垂直扩展——关系型数据库设计之初一般没考虑分布式数据分片，数据存本机，只能靠提升这台机器的性能提升能力

* MySQL 的主从只是让机器数量提升、读写性能提升，主和从存储的数据一模一样，数据存储总量没变，主从本质上只是备份

* > **定义**：水平扩展——插入数据时基于 id 或唯一标识做哈希运算，根据哈希结果决定这条数据落在哪个节点上，天然支持数据拆分

```text
关系型：垂直扩展 —— 只能给这一台升配
   ┌────────┐
   │ 数据库 │
   └────────┘
        ▲
   加 CPU / 加内存

非关系型：水平扩展 —— 加节点就行
   ┌────┐    ┌────┐    ┌────┐
   │ N1 │    │ N2 │    │ N3 │
   └────┘    └────┘    └────┘
     插入时按 id 或唯一标识哈希，结果决定落在哪个节点
```

* MySQL 默认不支持水平扩展，可基于第三方组件实现分库；但引入第三方组件会对性能造成影响，开发时要考虑的问题更多、复杂度增加

### 六个维度的综合对比

| 对比维度 | 关系型数据库 | 非关系型数据库 |
| :--- | :--- | :--- |
| 数据结构 | 结构化，有固定的表结构与约束 | 非结构化，常见键值、文档、宽列、图四种形态 |
| 数据关联性 | 有关联，靠外键由数据库维护 | 无关联，关联关系要靠程序员用业务逻辑自己维护 |
| 查询方式 | 基于 SQL 语句，语法严格且通用 | 语法不统一，有的像命令、有的像函数、有的像请求 |
| 事务 | 满足 ACID | 不一定能满足，或只做到 BASE |
| 存储方式 | 大多磁盘存储 | 大多内存存储，查询性能高 |
| 扩展性 | 垂直扩展 | 水平扩展，天然支持数据拆分 |

### 选型

* 业务数据结构相对固定、将来不会怎么变更，或对安全性、一致性要求较高（比如下单的订单数据），建议使用关系型数据库

* 数据结构不固定、经常可能有灵活变更，对一致性和 ACID 要求不高、但对性能要求较高，适合使用 NoSQL

* 实际开发中两者结合使用：订单数据用关系型数据库存储，同时可以冗余地把部分订单数据放到 NoSQL 里提升查询效率

---

## Redis 的安装

### 运行环境

* Redis 基于 Linux 服务器部署，版本选择 CentOS 7，建议保持一致

* 选择 Linux 的理由之一：大多数企业在做项目部署时用的都是 Linux 服务器

* 选择 Linux 的理由之二：Redis 的作者根本就没有编写 Windows 版的 Redis

* > **易错点**：网上找到的 Windows 版 Redis 并不是官方提供的，而是由微软自己编译出来的。

* 使用的 Redis 版本是 6.2.6

* > **提示**：云服务器也能装，但网速是一个考验，且安全性设置多（比如防火墙不能关），操作起来反而更麻烦，建议在本地准备一台虚拟机

### 依赖与编译安装

* Redis 基于 C 语言编写，安装前必须先安装 gcc 依赖

```bash
yum install -y gcc
```

* 安装包上传到 `/usr/local/src` 目录，这个目录一般情况下都是用来放安装文件的。

```bash
cd /usr/local/src
tar -zxvf redis-6.2.6.tar.gz
```

* 解压后进入 Redis 的安装目录，运行 `make && make install`，其中 `make` 是编译，`make install` 是安装

```bash
cd redis-6.2.6
make && make install
```

### 安装产物

* 默认安装路径是 `/usr/local/bin`，这些命令已经加入了环境变量，不必待在当前目录，在任意地方都可以直接运行

* > **提示**：敲 `redis-server` 打到一半按 Tab 键，如果能自动补全，就证明它已经在环境变量里了。

| 文件 | 作用 |
| :--- | :--- |
| `redis-server` | Redis 的服务端，运行这个脚本就能启动 |
| `redis-cli` | Redis 的命令行客户端，比较常用 |
| `redis-sentinel` | Redis 的哨兵 |

* 看到这三个文件在，就证明安装已经成功

* 官网除下载外还有三个去处：Commands 包含 Redis 的所有命令及其作用，是最官方、最准确的文档，也是学习 Redis 最重要的途径；Clients 是官方提供的各种语言的客户端；Documentation 是官方帮助文档，涵盖命令列表、pipeline 管道模式、发布订阅等相对高级的使用方式

---

## Redis 的启动方式

### 前台启动

```bash
redis-server
```

* 回车后会弹出一个 Redis 的日志界面，显示版本 6.2.6、端口 6379、本次的进程 ID、官方网站以及一个 Redis 的 logo

* > **易错点**：这种启动方式叫前台启动，界面会一直卡在这里，要连接必须重新打开一个窗口；当前窗口一关，Redis 也就被停止了，是一种不友好的方式

### 指定配置文件启动

* 配置文件的默认位置在 Redis 的安装目录下，叫 `redis.conf`

* > **提示**：修改之前最好提前备份，万一改错将来还能恢复：`cp redis.conf redis.conf.bck`

* 想在启动时指定配置文件，只需在命令后面跟上配置文件名称即可

```bash
redis-server redis.conf
```

* 这种方式启动后没有任何日志输出，因为它已经变成后台运行

* 用 `ps -ef | grep redis` 查看是否有进程在运行；要停止就用 `kill` 把这个进程杀掉

### 开机自启

* 需要自己编写一个系统服务文件，把 Redis 加入到操作系统的服务当中，这样以后它就能开机自启

* > **注意**：服务文件的内容最好拷贝现成的，不要自己去写

* 服务文件里最关键的是启动命令那一行：前面是 `redis-server` 的安装位置，后面是配置文件的目录，它会用这个命令去启动并指定这个配置文件

```text
ExecStart=/usr/local/bin/redis-server /usr/local/redis-6.2.6/redis.conf
```

```bash
vim /etc/systemd/system/redis.service
systemctl daemon-reload
```

| 命令 | 作用 |
| :--- | :--- |
| `systemctl start redis` | 启动 Redis |
| `systemctl stop redis` | 停止 Redis |
| `systemctl restart redis` | 重启 Redis |
| `systemctl status redis` | 查看 Redis 的运行状态 |
| `systemctl enable redis` | 设置开机自启 |

* 被 systemd 管理还不等于开机自启，只是能通过这些命令控制 Redis；`stop` 之后状态会变成 `dead`，需要额外执行 `systemctl enable redis` 才是开机自启

---

## 配置文件关键项

### 必改的三项

| 配置项 | 默认值 | 改成 | 说明 |
| :--- | :--- | :--- | :--- |
| `bind` | `127.0.0.1` | `0.0.0.0` | 端口监听的地址，不是允许访问的地址 |
| `daemonize` | `no` | `yes` | 是否以守护进程在后台运行 |
| `requirepass` | 注释状态 | 自己的密码 | 访问 Redis 的密码 |

* `bind` 是端口监听的地址，不能说是允许访问的地址；监听 `127.0.0.1` 意味着只有本地访问才允许，从外界访问都会被拒绝

* 改成 `0.0.0.0` 意味着在任意 IP 地址都能访问这台 Redis；测试阶段建议这么改，生产环境下肯定还是用默认值

* > **提示**：修改时把默认的 `127.0.0.1` 那一行注释掉，再新加一个 `0.0.0.0` 进去

* `protected-mode` 保护模式默认开启，一般先不用管，密码校验等操作 Redis 会替我们去做

* `daemonize` 控制的就是前台运行还是后台运行，默认值 `no`，改成 `yes` 就变成守护进程、后台运行

* > **提示**：配置项不好找时用搜索，在 vim 里输入 `/daemonize` 就能直接定位过去

* `requirepass` 默认是注释掉的；既然把 Redis 改成了任意人都能访问，在没有密码的情况下就等于裸奔

* > **注意**：Redis 有些命令可能存在漏洞，如果任何人都能访问，它可能在你电脑上跑一些脚本，机器有可能变成肉鸡或矿机，所以一定要给 Redis 设置密码

### 可选配置

| 配置项 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `port` | `6379` | 一般就用默认，除非端口被占用 |
| `dir` | `.` | 工作目录，`.` 代表当前目录 |
| `databases` | `16` | 数据库数量，编号 0 到 15 |
| `maxmemory` | 不限 | 最大可用内存，例如 `512mb` |
| `logfile` | 空 | 日志文件，默认为空即不记录日志 |

* `dir` 的默认 `.` 代表当前的意思：你在哪里启动或运行 Redis 命令，哪里就是工作目录

* `databases` 默认 16 代表有 16 个库，编号从 0 到 15；改成 1 就代表将来只用一个库，数量可在 1 到 16 之间选择

* > **易错点**：MySQL 的数据库可以随便创建，Redis 的库是固定的、由它自己创建好，数量可以调但库的名字不能改

* `logfile` 默认为空即不记录日志，将来出了错误也不知道原因，可以给它指定一个名称如 `redis.log`

* > **注意**：`logfile` 只写了文件名没有写路径，那么日志文件会产生在 `dir` 也就是运行 Redis 命令时的那个目录；这两个配置结合才是真正的存储位置

---

## 连接 Redis

### redis-cli 的语法与选项

```bash
redis-cli [option] [command]
```

| 选项 | 含义 | 不指定时的默认值 |
| :--- | :--- | :--- |
| `-h` | 指定 Redis 的 IP 地址 | `127.0.0.1`，代表本机 |
| `-p` | 指定端口 | `6379`，正是 Redis 的默认端口 |
| `-a` | 设置访问 Redis 时的密码 | 无 |

* 这些选项都可以不指定，不指定就是默认；指定了它就按你指定的去连接

* `command` 是连上之后要执行的命令；一般情况下不输入，因为希望先连上，之后会进入持续的交互式控制台，输入命令返回、再输入再返回

### 带密码的两种连法

* 不指定密码直接连上后打 `ping`，会返回 `(error) NOAUTH Authentication required.`，因为没有权限、需要密码

* 方式一：连接时加 `-a 密码`，能成功进入，`ping` 返回 `PONG`；但会有警告，说利用 `-a` 参数来指定密码太危险

* 方式二：连接时不指定密码，连上以后用 `auth` 命令校验；Redis 没有用户名只有密码，直接写密码即可。

```text
127.0.0.1:6379> auth 123321
OK
127.0.0.1:6379> ping
PONG
```

### 库的选择

* Redis 默认就有 16 个库，从 0 到 15；不同库之间尽管 key 一样，值是互不干扰的。

* 命令行里用 `select 编号` 切库

```text
127.0.0.1:6379> select 1
OK
127.0.0.1:6379> get name
"rose"
```

### 图形化桌面客户端

* 图形化客户端并不是官方提供的，而是 GitHub 上有开发者编写并开源的。

* > **注意**：它提供的仅仅是桌面客户端的源码，而没有现成的包，要用就得自己去编译；不想编译可以订阅作者的付费服务，图省事也可以使用别人帮你自动编译好的免费版本

* 连接设置里：连接名称随意起；IP 地址处填虚拟机的 IP；还有一处填密码，不填密码连不上

* 连上以后会显示 16 个库，从 0 到 15；库名是固定的不能改，只能去配置库的数量

* > **提示**：日常情况下建议使用图形化界面，很方便；但初学 Redis、想要学习里面的命令，还是建议通过命令行，因为在命令行里可以让你更加熟悉每一个命令

---

## 数据类型与命令查询

### 五种基本类型

| 类型 | 值是什么 | 特点 |
| :--- | :--- | :--- |
| String | 普通字符串 | 最基础的值类型 |
| Hash | 哈希表 | 值不是一个普通字符串，这里用字符串的形式来描述，但本质其实是个哈希表 |
| List | 有序集合 | 本质是一个链表 |
| Set | 无序集合 | 不能重复 |
| Sorted Set | 有序集合 | 可排序的集合，当然也不能重复 |

* Redis 里 key 一般就是 String，值却五花八门，光列出来的就有八种之多，而这还不是全部

* 除这八种以外 Redis 还有很多其他类型，用来实现其他特殊功能，比如消息队列

### 三种特殊类型

* `GEO`：是一个地理坐标，存的是经度和纬度

* `Map`：一种特殊的按 value 去进行存储的方式

* `HyperLogLog`：也是一种特殊的按 value 去进行存储的方式

* > **提示**：这三种往往用到一些特殊的数据统计，使用场景比较单一，通常连同它的操作和使用场景一起在实战阶段学习

* > **定义**：特殊类型的底层本质其实都是字符串，只不过是在基本类型的基础上做了特殊处理来实现特殊的功能。

### 学习路径

* 面对如此多的数据类型会产生两个问题：该怎么操作这些不同的数据类型，以及什么时候该使用哪一种

* 按照学习规律先弄懂怎么用，把每种数据结构的操作弄熟、特征弄透，再去分析什么时候用会更容易理解

* > **提示**：带着项目需求去分析该什么时候用，比直白地告诉你结论要好很多

### 怎么查命令

* Redis 里所有的命令都是分组的，学习时也分组一组一组来学

* 官网上用 `Filter by group` 按组分类过滤：`all` 是所有命令，选 `string` 就过滤出 String 相关的命令，选 `hash` 就看到哈希表有关的命令

* 命令行里输入 `help`，`help @组名` 就能看该组的帮助：`help @generic` 是不分数据结构的通用指令，`help @string`、`help @list`、`help @set`、`help @hash` 等分别对应各数据结构

* 帮助文档会告诉你命令的名称、参数、说明命令的摘要，还有从什么版本开始

* > **提示**：命令不用死记硬背，参考文档学习，忘了也没关系，关键是要会通过查询的方式自己去使用

---

## 通用命令

### keys

```text
127.0.0.1:6379> help keys
KEYS pattern
summary: Find all keys matching the given pattern.
group: generic
```

* 作用是查看符合模板（pattern）的所有 key

* > **定义**：pattern 是 Redis 内置的一种匹配表达式，它不是正则。

* 通配符：`*` 代表多个（任意）字符，`?` 代表一个字符

```text
127.0.0.1:6379> keys *
1) "age"
2) "name"
```

* > **易错点**：`keys` 既然基于通配符搜索，底层一定有模糊查询机制，效率肯定不高；数据量达到数百万、上千万甚至更多时，模糊匹配会给服务器带来巨大负担，可能搜很长时间

* > **注意**：Redis 是单线程的，搜索的这一段时间内无法执行其他命令，等于整个 Redis 服务被阻塞，所以生产环境下不建议使用 `keys` 去查询

* 集群模式下有主有从时，在从节点上做倒还可以；千万不要在主节点去做，因为会阻塞所有的请求

### del

* 作用是删除一个或多个指定的 key，要删谁就跟上谁

* > **提示**：帮助文档里 `DEL key [key ...]` 的中括号表示它可以接受多个 key，传一个就删一个，传多个就删多个

```text
127.0.0.1:6379> del k1 k2 k3 k4
(integer) 3
```

* > **定义**：`del` 的返回值代表实际删除的 key 的数量；上面 `k4` 不存在，所以返回值是 3

### exists

* 作用是判断一个 key 是否存在，后面同样可以跟一个或多个 key

* 存在返回 `(integer) 1`，不存在返回 `(integer) 0`

### expire 与 ttl

* > **定义**：存活时间（Time To Live，TTL）即 key 的有效期

* `EXPIRE key seconds` 给一个 key 设置有效期，单位是秒；有效期到期时该 key 会被自动删除

* `PEXPIRE key milliseconds` 的时间单位是毫秒

* > **易错点**：`EXPIRE` 的单位是秒、`PEXPIRE` 的单位是毫秒，这两个千万别记混

* 需要有效期的原因：Redis 基于内存存储，如果永远往里插入而从来不做删除清理，假以时日存储的数据越来越多，总有一天内存可能会被占满

* 举例：短信验证码一般的存在时间是五分钟，就给它设置五分钟有效期，到期以后自动删除，这样可以节省 Redis 的内存空间

* `TTL key` 查看一个 key 的剩余有效期

| 返回值 | 含义 |
| :--- | :--- |
| `-2` | key 已经不存在了，说明它是被自动删除的 |
| `-1` | key 存在，但永久有效，不会被自动删除 |
| `0` 及以上 | 距离过期还剩多少秒 |

* > **提示**：一般情况下建议在往 Redis 里存入数据时最好都设置一个有效期

---

## String 类型

### 本质与编码

* String 的值是 Redis 里最简单的一种数据类型

* 表现形式可以是普通字符串（如 `hello world`）、整型（如 `10`）、浮点型（如 `2.5`）

* > **定义**：不论表现形式如何，底层都是用字节数组去存储，区别在编码——数值类型会把数字直接转为二进制的形式作为字节去存储

* 一个字节就能表示很大的数字，所以数值编码更节省空间；字符串只能把字符转成对应的字节码再去存储，相对占用内存更多

* String 的表现形式不一样、底层编码不一样，但本质都是字节数组

* > **注意**：String 的最大上限不能超过 512 MB，所以一般情况下不会往 String 里面存储太多的数据，最多存一个图片的地址

### 单个与批量存取

```text
SET key value [EX seconds] [NX]
MSET key value [key value ...]
```

* `SET` 是添加或者修改：如果 key 不存在那就是添加，如果已经存在那就是修改

* 多次 `SET` 同一个 key 可以看到值被覆盖掉了，所以增或者改是一体的。

* `GET` 取单个值；`MSET` 批量添加，接收多组 key value；`MGET` 批量获取，一次性返回多个值

* > **提示**：Redis 里多个值形成的数组以 `1)`、`2)`、`3)` 这样的形式返回

```text
> MSET name 杰克 age 18 k1 v1 k2 v2
OK
> MGET name age k1 k2
1) "杰克"
2) "18"
3) "v1"
4) "v2"
```

### 数值自增与自减

* `INCR key` 让一个整型的 key 自增一，返回值是自增后的值

* `INCRBY key increment` 可以让步长自己指定，把 increment 写成负数就实现了自减的效果

* > **提示**：虽然事实上有专门的 `DECR` 自减命令，但一般使用时都用 `INCR` 并给成正负即可。

* `INCRBYFLOAT key increment` 让浮点型的数字做增长

* > **注意**：浮点数据没有默认增长，必须指定增长的步长

```text
> SET score 10.1
OK
> INCRBYFLOAT score 0.5
"10.6"
```

### 两个组合命令

* > **定义**：`SETNX key value` 添加一个键值对的前提是这个 key 不存在，否则不执行，所以它才是真正的新增功能，只有新增效果

* `SETNX` 返回 `(integer) 1` 表示插入成功；返回 `(integer) 0` 表示 key 已存在、值没有发生改变

* > **易错点**：`SETNX` 其实是组合命令——先是 `SET` 再加一个 `NX`；`NX` 本来就是 `SET` 的一个可选参数，所以 `SET name 王五 NX` 与 `SETNX name 王五` 的效果一样

* > **定义**：`SETEX key seconds value` 添加一个 key 的同时设置有效期，是把 `SET` 与 `EXPIRE` 合二为一

```text
> SETEX name 10 杰克
OK
> TTL name
(integer) 10
```

* `SETEX` 同样可以写成在 `SET` 后面加 `EX seconds` 来指定，效果是一样的。

### 命令分组记忆

| 分组 | 命令 | 说明 |
| :--- | :--- | :--- |
| 基本存取 | `SET`、`GET`、`MSET`、`MGET` | 单个增、单个查、批量增、批量查；增的操作具备修改功能，存在就是修改、不存在就是新增 |
| 数值操作 | `INCR`、`INCRBY`、`INCRBYFLOAT`、`DECR` | 专门操作数值类型，做自增或自减 |
| 组合命令 | `SETNX`、`SETEX` | 前者只在 key 不存在时才加，后者是添加的同时设置有效期 |

* 删除不属于 String 专属，用的是通用命令 `DEL`

### key 的层级结构

* Redis 是键值型数据库，键一定要求唯一，所以大多数情况下会以数据的 id 来作为 key 形成唯一标识

* Redis 里没有 MySQL 中 table 的概念，没有表，所有数据都存在一起；用户 id 为 1、商品 id 也恰好为 1 时就会产生冲突

* > **定义**：Redis 的 key 允许多个单词拼在一起形成层级结构，单词之间用冒号隔开

```text
项目名:业务名:数据类型:唯一标识
```

* > **提示**：这个格式不是固定的，公司里有自己的 key 标准就按照公司的标准去做

* 举例：`黑马:user:1` 与 `黑马:product:1`——一级项目名、二级数据类型、三级 id，两者前缀不同就不会产生冲突

* key 定义好以后，值是把 Java 对象序列化成的 JSON 字符串；Java 对象里有多少字段，JSON 里就有多少字段

```text
黑马
├── user
│   ├── 1
│   └── 2
└── product
    ├── 1
    └── 2
```

* 图形化客户端会把这种 key 自动折叠成层级树，看起来就像实际存了多层目录，其实是从未单独插入过任何中间层的 key

* > **提示**：这种 key 的层级存储可以避免 id 相同时的冲突，并且让数据分离看起来比较优雅，是使用 Redis 的一个最佳实践；不仅 String，其他数据类型的 key 都可以这样做

---

## Hash 类型

### 结构与价值

* > **定义**：Hash 类型的值是一个无序字典，其实就是一个哈希表，跟 Java 中的 `HashMap` 比较类似

* String 存对象的弊端：存取的是一个字符串，不管里面有两个字段还是十个字段，它都是一个字符串；想单独对某个字段做修改是无法进行的，要么把整个字符串覆盖掉，要么只能删掉重来

* Hash 的 key 与 String 没什么差异，差异在 value：它的 value 又分成两部分，一部分叫 `field`（也有人称它为哈希 key），另外一部分是 value

```text
String：key ──► "id=1, name=杰克, age=20, ..."

Hash：  key ──► ┌────────┬───────┐
                │ field  │ value │
                ├────────┼───────┤
                │ name   │ Lucy  │
                │ age    │ 17    │
                └────────┴───────┘
```

* 把一个用户里的 name、age 等拆成独立的字段值，用户有十个字段这个 key 里就可以有十个 field

* > **提示**：每个字段都能独立表示、独立修改，修改某一个字段对其他字段没有任何影响，这就是 Hash 相对于 String 更灵活的优势

### 单字段与批量存取

* Hash 的常见命令可以对照 String 来学：把 String 命令前面加上 H 就变成哈希命令——`SET` 变 `HSET`、`GET` 变 `HGET`、`MSET` 变 `HMSET`、`MGET` 变 `HMGET`、`SETNX` 变 `HSETNX`、`INCRBY` 变 `HINCRBY`

* 因为多出了一个 field，命令里就多了一个参数：`HSET key field value`

* 在同一个 key 里插入不同的 field，`HGET key field` 取单个字段；对同一个 key 再 `HSET` 一次就能修改单个字段

* `HMSET key field value [field value ...]` 一次给多个字段赋值；`HMGET key field [field ...]` 一次取多个字段的值

* 一个对象多一个字段、少一个字段都没问题，Redis 并不要求所有对象的字段一致

```text
> HMSET 黑马:user:4 name lily age 20 sex man
OK
> HMGET 黑马:user:4 name age sex
1) "lily"
2) "20"
3) "man"
```

### 全量获取

* `HGETALL key`：获取该哈希 key 中所有的字段名和字段值，一个 key 一个 value 紧挨着依次返回，不需要知道字段

* `HKEYS key`：获取一个 key 中所有的字段名

* `HVALUES key`：获取一个 key 中所有的 value

* > **提示**：三者可以对应 Java `HashMap` 里的 entry 键值对集合、keys 集合、values 集合

### 字段自增与 HSETNX

* `HINCRBY key increment field`：让哈希里某个字段的值增长，increment 给成负数就是自减

* `HSETNX key field value`：判断值不存在才执行、已存在就不执行

* > **易错点**：`HSETNX` 判断的不是 key 是否存在，而是某一个 field 是否存在；而 String 的 `SETNX` 判断的是当前 key 是否存在。

* > **提示**：Hash 类型的常见命令当成 Java 里的 `Map` 去操作就行了。

---

## List 类型

### 特征

* > **定义**：List 的值与 Java 中的 `LinkedList` 比较类似，底层可以看作是一个双向链表的结构

* 双向链表最大的特点是支持正向检索和反向检索，从头到尾或者从尾到头都可以。

* 有序：顺序跟插入的顺序有关

* 元素可以重复：它不会去检查元素是否一致

* 插入和删除速度快：插入和删除只是改变了链表中节点的指向

* 查询速度相对差一些：只能通过逐个节点遍历的方式查询，比传统的数组类型稍微差一点

* 适用场景：保存对顺序有要求的数据，例如朋友圈的点赞列表（谁先点赞谁后点赞）、评论列表

### PUSH 与 POP

* > **定义**：核心操作只有两个——`PUSH` 是插入，`POP` 是移除并返回

* `L` 是 left 即左侧，可以理解成队首；`R` 是右侧，可以理解成队尾

* > **易错点**：元素落在什么位置只跟插入的位置有关，跟元素本身的字母顺序无关，不会因为字母靠后就被排到后面

```text
最初：  [ a , b ]

LPUSH c  →  [ c , a , b ]
LPUSH d  →  [ d , c , a , b ]
RPUSH e  →  [ d , c , a , b , e ]

LPOP     →  取出 d
RPOP     →  取出 e
```

* `LPUSH users 1 2 3` 之后顺序是 `3 2 1`：先推 1，1 此时在最左边；再推 2，2 就跑到 1 的左边；再推 3，3 就跑到了最左侧

* `RPUSH` 恰好相反，后来的元素依次排在右侧

* > **提示**：不需要死记硬背，在脑海中想象列表的样子，`LPUSH` 就从左边插、`RPUSH` 就从右边插，大概就能推断出存进去的是谁、取出来的是谁

### 按角标取一段

* `LRANGE key start stop` 返回一段角标范围内的所有元素，先指定 key 再指定开始与结束

* > **易错点**：返回结果前面 `1)`、`2)` 这样的编号不是角标，角标是从 0 开始另算的。

```text
> LPUSH users 1 2 3
(integer) 3
> LPOP users 1
"3"
> RPOP users 1
"6"
> LRANGE users 1 2
1) "1"
2) "4"
```

### 阻塞式获取

* `LPOP` 与 `RPOP` 在列表中没有任何元素时会直接返回一个 nil，告诉你没有

* > **定义**：`BLPOP` 与 `BRPOP` 前面的 `B` 代表阻塞（block），效果与 `LPOP`、`RPOP` 一样，但列表为空时会等待一段时间，有点像阻塞队列的效果

```text
BLPOP key [key ...] timeout
BRPOP key [key ...] timeout
```

* timeout 是等待的秒数：写 10 就是等 10 秒，写 100 就是等 100 秒，等不到就结束

* 等待期间另一个客户端 `LPUSH` 进来元素，阻塞的一侧会立刻拿到元素，并且会告诉你花了多长时间才拿到

* > **注意**：`BLPOP` 必须传足参数，只给 key 不给 timeout 会报 `ERR wrong number of arguments for 'blpop' command`

### 模拟栈、队列与阻塞队列

* > **定义**：栈的特点是先进后出，入口与出口在同一侧；队列的特点是先进先出，入口与出口不在同一侧

* 类比：喝酒喝大了吐了是先进后出——用嘴巴喝进去、用嘴巴吐出来，同一个口，所以是栈；没吐而是去排队排出来是先进先出——喝与排不是一个口，所以是队列

```text
栈（先进后出）：入口与出口在同一侧
    LPUSH + LPOP
    RPUSH + RPOP

队列（先进先出）：入口与出口在不同侧
    LPUSH + RPOP
    RPUSH + LPOP

阻塞队列：先保证是个队列，再把"取"的那一步换成阻塞版本
    LPUSH + BRPOP
    RPUSH + BLPOP
```

* > **提示**：只要在同一边就是栈，只要不在同一边就是队列；阻塞队列则是在队列基础上把 `LPOP`/`RPOP` 换成 `BLPOP`/`BRPOP`

---

## Set 类型

### 底层与特征

* > **定义**：Set 可以看作是一个 value 恒为 null 的哈希表——底层仍是哈希表，只是这一次不关心 value 了，只关心它的 key

* 这一点和 Java 的 `HashSet` 由 `HashMap` 实现是同一个思路

```text
   insert "tom"
        │
        ├── ① hash("tom") 算出插入角标
        └── ② 按角标落进哈希表，value 恒为 null

   ┌───────────────────────┐
   │  k = "tom"  →  null   │  同一个 k 再来一次 → 互相覆盖
   └───────────────────────┘
```

* 无序：每一个插入的元素都会用 hash 算法计算它插入的角标，因此存储顺序与插入顺序无关

* 元素不可重复：相同元素会互相覆盖

* 查找速度比较快：根据哈希表来做查找，时间复杂度比较低

* 比 Java 的 Set 多了交集、并集、差集等特殊的集合运算功能。

* > **提示**：集合运算才是 Set 类型的价值所在，好友列表、共同好友、关注列表这些做起来非常方便，在社交型应用中 Set 的使用比较广泛

### 单个集合的操作

| 命令 | 作用 |
| :--- | :--- |
| `SADD key member [member ...]` | 往集合中添加元素；`key` 是集合名称，`member` 是要插入的元素，一次可插一个也可插多个 |
| `SREM key member [member ...]` | 移除元素（remove 的缩写），同样支持移除一个或多个 |
| `SCARD key` | 返回集合中元素的总个数，就是一个计数 |
| `SISMEMBER key member` | 判断元素在不在当前集合里，有点像 Java 里的 `contains` |
| `SMEMBERS key` | 返回集合中的所有成员 |

```text
SADD s1 a
SADD s1 b c d
SMEMBERS s1
SREM s1 a
SISMEMBER s1 a
SCARD s1
```

### 集合间的交集、差集、并集

```text
s1 = { a, b, c }        s2 = { b, c, d }

SINTER   交集  s1 ∩ s2  =  { b, c }          两边都有的
SDIFF    差集  s1 - s2  =  { a }             s1 里有、s2 里没有的
SUNION   并集  s1 ∪ s2  =  { a, b, c, d }   合在一起，重复只记一次
```

* `SINTER key1 key2` 求的是两个集合的交集，即两边都有的元素

* `SDIFF key1 key2` 代表的是差距，看第一个 key 里有、第二个 key 里没有的。

* `SUNION key1 key2` 把所有元素合并在一起；因为 Set 不能重复，合并时重复元素只会记录一次

* > **易错点**：`SDIFF` 的第一个 key 是被减数，`SDIFF s1 s2` 和 `SDIFF s2 s1` 算的是两回事，把 key 的顺序写反结果就反了。

### 好友列表案例

* 数据用 `SADD zs lisi wangwu zhaoliu` 与 `SADD ls wangwu mazi ergou` 存两份好友列表

| 需求 | 命令 |
| :--- | :--- |
| 计算张三有几个好友 | `SCARD zs` |
| 计算张三和李四的共同好友 | `SINTER zs ls` |
| 哪些人是张三的好友却不是李四的 | `SDIFF zs ls` |
| 张三和李四的好友总共有哪些人 | `SUNION zs ls` |
| 判断李四是不是张三的好友 | `SISMEMBER zs lisi` |
| 判断张三是不是李四的好友 | `SISMEMBER ls zs` |

* 好友关系是单向记录的：张三的好友里有李四，李四的好友里却可能没有张三，两个方向要分别判断

* 把某个好友从列表删除用 `SREM zs lisi`

---

## SortedSet 类型

### 底层与特征

* Java 里可排序的 Set 是 `TreeSet`，底层由红黑树实现，且排序时需要你自己定义排序方法

* Redis 的 SortedSet 从功能上与之类似——都是可排序的集合、元素唯一，但底层数据结构上的差别很大

* > **定义**：SortedSet = Set + score，每个成员都带一个分数，排序就按这个固定的 score 值来，不需要自定义排序方法

```text
                ZADD key score member ...
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
   ┌─────────────────────┐  ┌─────────────────────┐
   │  skip list          │  │  hash table         │
   │  按 score 维护顺序  │  │  key   = member     │
   │  TopN 走这里        │  │  value = score      │
   └─────────────────────┘  └─────────────────────┘
              │                       │
              └── 负责排序            └── 负责按元素查分
```

* 跳表的作用是用来做排序；哈希表中 key 是元素，这样实现了 set 的效果，值是对应的 score 分数

* 特性：可排序、元素不能重复、查询速度快（哈希表保证查询，跳表还能进一步增加查询速度）

* 常见用途：实现排行榜，比如 TopN、Top10 这样的效果

### 命令清单

| 命令 | 作用 | 说明 |
| :--- | :--- | :--- |
| `ZADD key score member [...]` | 新增元素 | 除 key 和 member 还要带上分数，**分数在前、元素在后**，后面可串多组批量添加 |
| `ZREM key member` | 移除元素 | 先指定集合名再给元素 |
| `ZSCORE key member` | 查询某个元素的分数 | |
| `ZRANK key member` | 获取排名 | 排序默认根据分数来，排名从 0 开始 |
| `ZCARD key` | 获取元素总个数 | 一个计数 |
| `ZCOUNT key min max` | 统计**分数区间**内的个数 | min/max 是分数值 |
| `ZINCRBY key increment member` | 分数自增或自减 | increment 写成负数就是减分 |
| `ZRANGE key start stop` | 按**排名区间**取元素 | min/max 是名次角标，从 0 开始 |
| `ZRANGEBYSCORE key min max` | 按**分数区间**取元素 | min/max 是分数的最小值与最大值 |

* > **易错点**：`ZRANGE` 和 `ZRANGEBYSCORE` 长得像但拿的东西完全不同——前者的 min/max 是**名次角标**，后者的 min/max 是**分数值**；`ZRANGE 0 9` 是按得分排序后的前十名元素，`ZRANGEBYSCORE 0 80` 是 80 分以内的所有元素

* > **易错点**：计数那边同理，`ZCARD` 是全量个数，`ZCOUNT` 才是按分数区间统计；`ZCOUNT` 与 `ZRANGEBYSCORE` 给的都是分数区间，但前者出数量、后者出具体元素

### 降序排名的 REV 后缀

* 所有与排名有关的命令默认都是升序排名，按分数升序

* 要降序排名，就在这条命令的 `Z` 后面添加 `REV`（reverse）

* `ZRANGE` 加 `REV` 变 `ZREVRANGE`，`ZRANK` 加 `REV` 变 `ZREVRANK`

* 基本上大部分跟排名有关的命令都支持这个 `REV`

### 排行榜案例

* 批量写入：`ZADD stu 85 jack 89 lucy 82 rose 95 tom 78 jerry 92 amy 76 miles`，key 是 `stu`，分数在前元素在后

* 图形化客户端默认展示出来的数据就是升序的，插入是乱的、取出来是排好序的。

* 删除元素：`ZREM stu tom`

* 升序排名：`ZRANK stu rose` 返回 2，因为在升序列表里 rose 排第 3 位、而排名从 0 开始

* 降序排名：`ZREVRANK stu rose` 返回 3

* > **注意**：业务里习惯的排名是从 1 开始，所以功能涉及排名时一定要在获取到的结果上加个一

* 80 分以下的人数：`ZCOUNT stu 0 80` 返回 2；`ZCARD stu` 返回的是全部元素个数

* 加分：`ZINCRBY stu 2 amy`，92 变成 94；把 2 改成负数就是减分

* 前三名用 `ZREVRANGE stu 0 2`，后三名用 `ZRANGE stu 0 2`

* 80 分以下的具体学生用 `ZRANGEBYSCORE stu 0 80`

---

## Java 客户端选型

* Redis 官网提供了各种编程语言的客户端，推荐的客户端会带一个标记，那个小点表明这个客户端最近也持续在做更新；Java 客户端里排在前三的都属于推荐之列

### Jedis

* 名字其实就是 Java Redis 的组成单词

* 最大的特点是里面的方法都以 Redis 的命令作为方法名称：Redis 有 `SET` 它就有一个 `set`，有 `get` 它就有一个 `get`，有 `mset` 它就有一个 `mset`

* 学习成本比较低，命令知道了方法也就会了，简单实用；刚推出就得到广泛使用，至今依然有很多公司在使用

* > **易错点**：Jedis 实例是线程不安全的，多线程并发运行有线程安全问题；多线程使用时必须为每一个线程创建独立的连接，必须配合连接池使用

### Lettuce

* 底层实现基于 Netty，Netty 是一个高性能的网络编程框架

* 支持同步或异步的连接，是一种响应式编程的方式，也是现在 Spring 里推荐大家使用的编程方式

* 线程安全，并且对 Redis 的哨兵模式、集群模式都有非常好的支持

* 响应式、异步编程的吞吐能力也比较高，因此 Spring 官方默认兼容的就是 Lettuce 客户端

### Redisson

* 特点不在于对 Redis 的基本操作，而在于它底层基于 Redis 实现了一系列分布式的、可伸缩的 Java 工具

* Java 里的 Map、集合、队列、锁、信号量、原子整形等基本类都是单机的，如果在分布式环境下往往就失去作用

* Redisson 基于 Redis 重新实现了这一批东西，使它们可以在分布式环境下同样能够使用

* 在分布式环境下有这类需求时可以直接使用，不用自己造轮子

### 三者对比

| 客户端 | 最大特点 | 什么时候会选它 |
| :--- | :--- | :--- |
| Jedis | 方法名就是 Redis 命令名，学习成本低 | 常规业务逻辑，简单直接；实例线程不安全，多线程必须配连接池 |
| Lettuce | 基于 Netty，同步/异步、响应式编程 | 与 Spring 现在的编程模型结合得好、吞吐高，Spring 官方默认兼容 |
| Redisson | 基于 Redis 重写了一批分布式的 Java 工具 | 需要在分布式环境下使用这些能力时，不用自己造轮子 |

* > **提示**：Spring Data Redis 底层可以兼容 Jedis 和 Lettuce，它定义了一套 API，这套 API 底层既可以用 Jedis 实现也可以用 Lettuce 实现，学了它就等于两个都会

---

## Jedis 的使用

### 依赖与连接

* 依赖是 `redis.clients` 的 `jedis`，版本 3.7.0；另外引入 JUnit 以便基于单元测试测代码

```xml
<dependencies>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.7.0</version>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

* 建立连接其实就是 `new Jedis`，第一个参数是 IP 地址，第二个是端口号，效果类似 `redis-cli -h` 与 `-p`

* 建连接之后还没有权限，需要 `auth` 设置密码才有访问它的权利

* `select` 可以选择库，不选择的话默认也是 0 号库，总共有 16 个库

```java
@BeforeAll
static void setUp() {
    jedis = new Jedis("192.168.150.101", 6379);
    jedis.auth("123321");
    jedis.select(0);
}
```

### 读写操作

* 方法名就是命令名：`jedis.set("name", "杰克")` 就像在命令行里输入 `SET name 杰克`；`jedis.get("name")` 得到的就是值本身

* `setex`、`setnx` 等命令对应的方法也都存在

* 哈希用 `hset(key, field, value)` 存单个字段，多次调用就能在同一个 key 下插入多个不同的 field

* `hmset` 可以一次传入多个字段值，参数用 Map 承载

* `hget` 是一个字段一个字段取，`hgetAll` 一次性把整个 Map 取出来

```java
@Test
void testHash() {
    jedis.hset("user1", "name", "杰克");
    jedis.hset("user1", "age", "21");

    Map<String, String> all = jedis.hgetAll("user1");
    System.out.println(all);
}
```

### 释放资源

* 最后一步是释放资源，也就是 `close` 关闭

* > **易错点**：这里要做健壮性判断，如果前面已经抛异常直接走到这，`jedis` 还没初始化，直接关有空指针的风险

```java
@AfterAll
static void tearDown() {
    if (jedis != null) {
        jedis.close();
    }
}
```

* 四步流程：引入依赖 → 建立连接（指定 IP、端口、密码、选择库）→ 用 Jedis 操作（方法名就是命令名）→ 释放资源，最后一步千万不能忘

### Jedis 连接池

* 频繁地创建和销毁 Jedis 对象有很大的性能损耗，官方基于 Apache Commons Pool 实现了 Jedis 连接池

* 通常把 `JedisPool` 定义成工具类的静态成员变量，再通过静态代码块初始化，JVM 里就一个

```java
public class JedisConnectionFactory {

    private static final JedisPool JPL;

    static {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(8);
        poolConfig.setMaxIdle(8);
        poolConfig.setMinIdle(0);
        poolConfig.setMaxWait(Duration.ofMillis(1000));

        JPL = new JedisPool(poolConfig, "192.168.150.101", 6379, 1000, "123321");
    }

    public static Jedis getJedis() {
        return JPL.getResource();
    }
}
```

* 构造函数里前一半是连接池的配置，后一半是连接的参数——IP、端口、超时时间、密码

| 参数 | 含义 | 示例值 |
| :--- | :--- | :--- |
| `maxTotal` | 最大连接数，池里最多允许创建这么多个连接，再多就不行了 | 8 |
| `maxIdle` | 最大空闲连接，即便没人访问也最多预备这么多个，一旦有人来就可以直接用 | 8 |
| `minIdle` | 最小空闲连接，超过一段时间一直没人访问，空闲连接会被释放直到达到这个数 | 0 |
| `maxWait` | 池里没有连接可用时最多等多久；默认值 -1 表示无限制等待，一直等到有新的空闲连接为止 | 1000ms，超时就报错 |

* 提供一个静态方法暴露给外部，任何地方调用它都是从池子里 `getResource`

* > **易错点**：从池里取出的 Jedis 调用 `close` 时，因为存在连接池，底层并不会真的关闭而是 `returnResource` 归还——连接还到连接池里，而不是把它销毁

---

## Spring Data Redis

### 定位

* Spring Data 是 Spring 里专门做数据操作的模块，旗下还有 JDBC、JPA、MongoDB、Cassandra、Solr、Elasticsearch、Neo4j 等，Spring Data Redis 是其中之一

* 目前最新版本是 2.6，官方支持到次年同期约一年左右，能放心用很长时间；企业里用的往往落后一点，比如 2.3、2.4，但整体使用差异不大

* > **提示**：新版本不等于生产版本，企业卡在旧版本上是常态，别为了追新而追新

### 它替我们做了什么

* Spring 从来不重复造轮子，它做的是**整合**：Spring Data Redis 底层整合了 Lettuce 和 Jedis 这两个客户端，并提供 `RedisTemplate` 作为统一标准的 API

* > **易错点**：注意是整合，不是抄袭——把别人的东西拿过来再封装，在 Spring 那儿叫整合

* 这套思路与 `JdbcTemplate` 封装数据库操作一模一样，只是 `RedisTemplate` 封装的是对 Redis 的各种操作

* 额外支持：发布订阅模型、哨兵、集群，以及基于 Lettuce 实现的响应式编程——结合 Spring WebFlux 做响应式编程时用它再好不过

* Redis Collection：把 JDK 里的各种集合基于 Redis 重新实现了一遍，比如队列、链表等；这样做出来的实现是分布式的、跨系统的。

```text
        你的 Service 代码
               │
               ▼
    ┌────────────────────────┐
    │     RedisTemplate      │  统一 API，按数据结构分组
    └────────────────────────┘
        │                 │
        ▼                 ▼
   Lettuce 驱动       Jedis 驱动        二选一，默认 Lettuce
        └────────┬────────┘
                 ▼
       commons-pool2 连接池
                 ▼
            Redis 服务端
```

### 序列化与反序列化支持

* Jedis 的 `set` 参数是字符串或字节数组，存一个复杂的 Java 对象必须手动对它做序列化、变成字符串或字节

* Spring Data Redis 内部支持基于 JDK、Jackson、字符串等方式做序列化，写入 Redis；也支持反序列化，从 Redis 里读到的字节再变成 Java 对象或字符串

* > **定义**：序列化 = Java 对象转成字节写进去；反序列化 = Redis 里的字节变成 Java 对象读出来

### opsForXxx 分组

* Jedis 把 Redis 上百个命令封装成上百个方法，学习成本低但类显得比较臃肿

* Redis 官方的命令本来就是分组的——通用命令、操作字符串的、操作哈希的、操作 List 的、操作 Set 的，`RedisTemplate` 做了同样的事

| 调用 | 拿到的对象 | 对应的数据结构 |
| :--- | :--- | :--- |
| `opsForValue()` | `ValueOperations` | 字符串 |
| `opsForHash()` | `HashOperations` | 哈希 |
| `opsForList()` | `ListOperations` | List |
| `opsForSet()` | `SetOperations` | Set |
| `opsForZSet()` | `ZSetOperations` | Sorted Set |

* 这些 API 的名字都是 `opsFor` 开头，返回值都是对应的 Operation 对象，里面封装的就是该数据结构的各种操作

* > **提示**：`RedisTemplate` 这个类本身上封装的是一些通用的、或者说比较特殊的命令，直接利用它去调用就好

### 快速入门三步

* 第一步引入依赖，一共两个：`spring-boot-starter-data-redis` 与连接池 `commons-pool2`；因为不管是 Jedis 还是 Lettuce，底层都会基于 commons-pool2 实现连接池效果

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

* 第二步配置连接信息；Spring Boot 已经默认整合并做了自动装配，不用自己写连接池代码

```yaml
spring:
  redis:
    host: 192.168.150.101
    port: 6379
    password: 123321
    database: 0
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: 1000ms
```

* > **提示**：配置项记不住也没关系，在配置文件里输入 `spring.redis`，所有的提示就都有了，照着写就行

* `max-active` 是最大连接数，`max-idle` 是最大空闲连接，`min-idle` 是最小空闲连接，`max-wait` 是等待时长

* > **易错点**：连接池配置有两套，一套 Jedis 一套 Lettuce，选哪种就是在选底层实现；`spring-boot-starter-data-redis` 默认引入的是 Lettuce 客户端，要用 Jedis 还得在 pom 里额外引入 Jedis 的依赖

* > **易错点**：一定要手动配置 `lettuce.pool`，连接池配置才会生效，否则它不会生效

* 第三步注入并写测试；`@Autowired` 自动装配，这些类都不用我们自己创建

```java
@SpringBootTest
class RedisTemplateTest {

    @Autowired
    private RedisTemplate<String, Object> template;

    @Test
    void testString() {
        template.opsForValue().set("name", "胡歌");

        String name = (String) template.opsForValue().get("name");
        System.out.println(name);
    }
}
```

* > **提示**：`set` 的 key 和 value 并没有要求必须传字符串，传 Object 也可以接受，因为底层有一个自动的序列化机制帮你处理

### opsForHash

* `opsForHash()` 拿到的就是对哈希有关的操作；Spring 里的方法名并不是命令名

* > **定义**：`put(key, hashKey, value)` 相当于 `HSET key field value`——它认为你玩的是哈希表，哈希表不就是 Java 的 HashMap 吗，于是干脆叫 `put`

* `putAll(key, map)` 有点像 `HMSET`，指定一个 key 和多个字段值，字段值形成一个 Map

* `get(key, hashKey)` 取一个字段；`entries(key)` 取所有的键值对，对应 Java 的 `entrySet`

* `keys(key)` 取所有的字段名，`values(key)` 取所有的值

```java
@Test
void testHash() {
    stringTemplate.opsForHash().put("user:400", "name", "胡歌");
    stringTemplate.opsForHash().put("user:400", "age", "21");

    Map<String, String> map = new HashMap<>();
    map.put("name", "胡歌");
    map.put("age", "21");
    stringTemplate.opsForHash().putAll("user:400", map);

    Object name = stringTemplate.opsForHash().get("user:400", "name");
    Map<Object, Object> result = stringTemplate.opsForHash().entries("user:400");

    stringTemplate.opsForHash().keys("user:400");
    stringTemplate.opsForHash().values("user:400");
}
```

---

## 序列化器的选择与取舍

### 默认的 JDK 序列化

* 现象：明明执行了 `set("name", "胡歌")`，在客户端里 `GET name` 取到的却是旧值；`keys *` 里除了 `name`，还多了一个看不懂的长 key

* 原因：`RedisTemplate` 的 `set` 方法接收的参数并不是字符串而是 Object，在进入 `set` 之前传进来的值就已经被装饰成字节了，key 也一样

```text
   opsForValue().set("name", "胡歌")
            │
            ▼
   RedisTemplate.set()          参数是 Object，不是 String
            │  key 与 value 双双都要过序列化
            ▼
   valueSerializer.serialize()
            │
            ▼
   JdkSerializationRedisSerializer
            │
            ▼
   SerializationUtils → JDK 序列化
     └─ new ByteArrayOutputStream()    缓冲
     └─ new ObjectOutputStream(buf)
     └─ oos.writeObject(obj)           把 Java 对象转成字节
            │
            ▼
      byte[]  ──►  写进 Redis
```

* RedisTemplate 里有四个序列化器，利用它存入的一切数据最终都会作用于这四个，取决于你的数据结构；普通字符串只用到前两个

| 序列化器 | 管的是谁 |
| :--- | :--- |
| `keySerializer` | key 的序列化器 |
| `valueSerializer` | value 的序列化器 |
| `hashKeySerializer` | 哈希结构里面的字段名 |
| `hashValueSerializer` | 哈希结构里面的字段值 |

* 这四个默认为 null，在没有给它们定义的情况下会创建一个默认的序列化器，而默认序列化器就是 JDK 的序列化器

* > **注意**：JDK 序列化器会把你的 key 和 value 一起序列化成字节，key 也被序列化，这才是「明明写了 name 却找不到 name」的真正原因

### JDK 序列化的两大问题

* 可读性差：值被剁碎成一长串看不懂的东西，在客户端里认不出存的是什么

* 还会出现 bug：以为把 `name` 改了，结果 `name` 没改，而是 set 了一个新的东西进去

* 内存占用较大：明明只是一个短字符串，却被序列化成很长的字节

* 期望的效果是所见即所得——我写的是什么，你就存什么

### 可选的序列化器

| 序列化器 | 干什么 | 用在哪 |
| :--- | :--- | :--- |
| `JdkSerializationRedisSerializer` | JDK 默认序列化，内部走 `ObjectOutputStream` | 最不好用的一种，不希望用它 |
| `StringRedisSerializer` | 专门用来处理字符串 | key、hashKey |
| `GenericJackson2JsonRedisSerializer` | 把对象转成 JSON 字符串 | value、hashValue |

* 字符串要想转成字节写入 Redis，其实只需要简单地 `getBytes` 就行了，没必要利用 JDK 去序列化；`StringRedisSerializer` 做的就是这件事，只不过底层的编码（比如 UTF-8）你可以控制

* > **提示**：一般情况下 key 是字符串、只有值可能是对象，所以 key 一般就用 `StringRedisSerializer`，值建议用 JSON 序列化器

### 自定义 RedisTemplate

* `RedisTemplate` 是允许你修改的，只要把它的序列化对象换掉就行

* 定义一个 Bean 叫 `redisTemplate`，泛型 `<String, Object>` 意味着默认 key 永远是 String、value 是 Object；把它 new 出来，注入连接工厂并塞进去，然后开始设置序列化工具

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> tm = new RedisTemplate<>();

        tm.setConnectionFactory(connectionFactory);

        tm.setKeySerializer(RedisSerializer.string());
        tm.setHashKeySerializer(RedisSerializer.string());

        tm.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        tm.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());

        return tm;
    }
}
```

* > **提示**：连接工厂不需要我们自己创建，Spring Boot 会自动创建，注入进来即可；`RedisSerializer.string()` 返回的就是一个以 UTF-8 作为编码的 `StringRedisSerializer` 常量，不用自己去 new

* > **易错点**：改用 Jackson 后如果缺少 Jackson 处理的工具类，会报类找不到的错误；平时开发时 Spring Boot 自带这个依赖不用操心，没有引入 Spring MVC 时才需要手动引入

* 换完之后 `name` 在客户端里就是一个干干净净的普通字符串，所见即所得

### 自动反序列化的代价

* 存一个对象进去，客户端里看到的是 JSON 风格的值；取的时候还能自动把 JSON 反序列化成 Java 对象

* 原理：写入 JSON 的同时它还会写出一个 `class` 属性，对应的就是这个 User 类的字节码名称；反序列化时读到这个类名字，才知道要转成哪个类型的对象

* 问题是这个字节码本身占用的内存空间甚至比数据本身还要长；当成百上千上万个对象要存时会多余占用非常多空间，大型项目中无法接受

* > **注意**：不写这个 class 就没有办法实现自动反序列化——序列化时直接把对象转字符串写就行，但反序列化时它无法知道这个 JSON 串要转成哪个类型的对象

* > **定义**：class 字段是自动反序列化的前提，二者绑定——有它才有便利，没它就没有便利但省内存

### StringRedisTemplate 方案

* 想省内存就不能用 JSON 序列化器处理 value，而要统一使用字符串序列化器；这样要求 key 和 value 都只能是字符串，不能再直接存 Java 对象

* > **定义**：`StringRedisTemplate` 是 Spring 提供的一个类，它的 key 和 value 的序列化方式默认就是字符串方式

* 用了它就省去了自定义 `RedisConfig` 的复杂配置，以前那一套不需要了。

* 要存 Java 对象时必须手动完成序列化与反序列化，通常用 `ObjectMapper`（Spring MVC 里默认使用的 JSON 处理工具），也可以换成 FastJSON 等熟悉的工具

```java
@SpringBootTest
class RedisStringTemplateTest {

    @Autowired
    private StringRedisTemplate stringTemplate;

    private static final ObjectMapper MAPPER = new ObjectMapper();

    @Test
    void testSaveObject() throws JsonProcessingException {
        User user = new User("胡歌", 21);
        String json = MAPPER.writeValueAsString(user);
        stringTemplate.opsForValue().set("user:200", json);
    }

    @Test
    void testGetObject() throws JsonProcessingException {
        String json = stringTemplate.opsForValue().get("user:200");
        User user = MAPPER.readValue(json, User.class);
        System.out.println(user);
    }
}
```

* > **易错点**：注意 key 是 `user:200`，存进去的也必须用 `user:200` 去取

* > **提示**：取出来的一定是一个 JSON 字符串，不能强转，因为里面没有字节码就没人帮你自动处理；而程序员自己知道取出来的是哪个类型，手动告诉它即可。

* 把 JSON 的处理封装成工具类之后，这几行代码也可以省掉，这是推荐使用的方案

### 两种方案对比

| | 自定义 RedisTemplate + JSON 序列化器 | 使用 StringRedisTemplate |
| :--- | :--- | :--- |
| 写 | 直接 `set` 对象 | 先 `writeValueAsString` 转 JSON 再 `set` |
| 读 | 直接 `get`，自动反序列化 | 先 `get` 拿字符串，再 `readValue` 转对象 |
| 内存 | 会占用额外空间记录那个类的字节码 | 纯 JSON 字符串，省空间 |
| 配置 | 需要自己写 RedisConfig | 省去了自定义的过程 |

* 方案一在做 JSON 的序列化和反序列化时会自动帮我们处理，不用你管，弊端是占用额外的内存空间记录类的字节码

* 方案二省去了自定义的过程，麻烦点在于每次存数据要手动把对象序列化为 JSON、每次读数据要手动做反序列化

* > **提示**：两者各有优缺点，没有最好的，需要根据自己的需求或者说自己的喜好去选择
