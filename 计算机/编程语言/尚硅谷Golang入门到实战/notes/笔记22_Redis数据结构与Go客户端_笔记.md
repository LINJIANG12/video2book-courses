# Redis 数据结构与 Go 客户端

## 启动与连接

* **启动服务端**

    * 直接双击 `redis-server.exe` 启动，界面会显示 `6379` 和 `The server is now ready to accept connections on port 6379`。

    * > **注意**：这个服务启动之后就不要关闭，客户端也同时双击 `redis-cli.exe` 连上；连接是双向的，关掉任何一边都玩不转。

    * `redis-cli` 已经把要操作的端口配好了，双击就相当于启动一个 `client.go`，机制上它内部也在建立连接。

    * Linux 下面直接 `./` 加执行名执行即可。

    * 先用 `redis-cli` 把基本指令学完，再用 Go 去连，顺序上是顺理成章的。

* **指令去哪查**

    * 站点 `redisdoc.com` 上按结构分好类写全了指令，用 Ctrl+F 快速查找。

    | 想做什么 | 查哪一栏 | 常见的几条 |
    | :--- | :--- | :--- |
    | 操作 Key 本身 | Key 的操作 | — |
    | 存取 key-value | 字符串 | `SET`、`GET`、`APPEND`、自增、`MGET`、`MSET` |
    | 以哈希结构存取 | Hash | `HSET`、`HGET`、`HDEL`、`HLEN` |
    | 以列表结构存取 | List | — |
    | 存取集合 | Set | — |
    | 存取有序集合 | Sorted Set | 一般以 Z 开头，写的是 `ZSET` |

    * > **结论**：数据只有五种——String（字符串）、Hash、List、Set 和有序集合（Sorted Set）；发布订阅、事务、脚本、连接、切数据库这些属于其它服务类功能。

    * 指令不需要背，把原理搞清楚即可；放在 Java 语境下讲一般要讲两到三天，这里因为是 Go，重点只解决怎么存和怎么取。

## 数据库与通用指令

* **16 个数据库**

    * Redis 启动后默认有 16 个数据库，用编号标识，0 到 15，一共 16 个。

    * 可以把内存想象成 16 份，各数据库之间内容互相独立，默认在 0 号库里操作。

    * `SELECT 1` 切到 1 号库，`SELECT 0` 切回 0 号库；切到空库再 `get k1` 就拿不到东西，切回 0 号才能拿到。

* **添加与取出**

    ```text
       客户端 (redis-cli 或 Go 代码)
                |
                |  set k1 "hello"
                |  ① 指令通过网络发给 Redis
                v
       +----------------------------------+
       |        Redis 核心组件            |
       |  ② 解析：哦，是 SET，字符串类型   |
       +----------------+-----------------+
                        v
       +----------------------------------+
       |     内存中的 0 号数据库          |
       |        k1  ──►  "hello"          |
       +----------------------------------+

       取的时候反过来：
       get k1 ──► 核心组件解析出要 k1
              ──► 在内存里找到 k1
              ──► 把 "hello" 返回给你
    ```

    * `SET` 后面带 key、value，中括号里的部分代表超时时间等附加信息，不需要可以先不管。

    * `GET` 把 key 的名字写进去就能取到值。

* **计数与清空**

    * `DBSIZE` 查看当前数据库一共有多少对 key-value，默认针对当前库；设好一对时返回 1，再加一对就返回 2。

    * `FLUSHDB` 清空当前数据库，`FLUSHALL` 清空 16 个数据库的全部数据，区别就在这里。

## 字符串 String

* **基本形态**

    * > **定义**：字符串是 Redis 最基本的数据类型，一个 key 对应一个 value。

    * 类似于 Go 里用变量名标示一个字符串，`str1 := "hello"` 里 key 相当于 `str1`，值相当于赋给它的内容。

    * Redis 的字符串是二进制安全的，既能存普通文本也能存图片；理论上把图片读成 byte 切片存进去再读出来重写回去就还原了。

    * > **提示**：一般没人真把大图片、大电影存进去，因为内存本身非常珍贵；一条电影 10 个 G 一下就把内存撑满了。

    * > **结论**：Redis 字符串最大是 512M，这是设计层面的限制，指的是**一个**字符串的上限，两个字符串就是两倍。

    * `SET` 如果键存在就相当于修改，不存在就相当于添加；`DEL` 删除，删完再 `GET` 会返回 `nil`。

    * > **提示**：控制台里存中文显示成乱码是因为控制台编码是 GBK，用程序读出来是正确编码，不需要担心。

## 字符串的批量与过期

* **`SETEX`**

    * 用来做限时需求：留言只保留 30 秒、用户 1 分钟不操作就退出、定时任务到点销毁。

    * `ex` 是 expire 的缩写，意思就是超时、废除；`SETEX key 秒 值`，比如 `setex message01 10 hello,world`。

    * 设置 10 秒后再 `get message01` 还能取到，等一会儿再取就变成 `(nil)`。

    * 销毁由核心组件不停扫描完成，它发现超时到了就把数据自动销毁。

    * > **结论**：需要定时、定时器、定时任务时可以用它来做。

* **`MSET` 与 `MGET`**

    * `mset key value key value` 一次性设置多个 key-value，类比 Go 里一次性声明多个变量。

    * `mget key1 key2` 一次性取回多个值，完全也可以用 `get` 一个一个取。

    * > **结论**：对应到 Go 里返回的是一个切片或数组，遍历一遍就能拿到全部值，思路完全可以类比迁移。

## 哈希 Hash

* **结构与适用场景**

    * 名称来历是哈希这个人搞了一个算法，后来用他的名字命名该算法。

    * > **定义**：Redis 哈希是一个 string 类型的 field 和 value 的映射表，即一个键值对的集合。

    * 类似于 Go 里的 `map`：哈希的 key 相当于 map 的名字，里面的 field 和 value 相当于 map 里一个个键值对。

    * 特别适合用于存储对象，也就是 Go 里用结构体实例体现出来的对象；单个基本数据类型描述不了综合信息，所以才有了 map，Redis 的设计者做了同样的事。

* **基本存取**

    * 存哈希用 `hset`，前面的 `h` 代表 hash。

    ```bash
    hset user1 name Smith
    hset user1 age 30
    hset user1 job "Golang coder"
    ```

    * 取哈希用 `hget`，传哈希名和字段名即可。

    * > **易错点**：字段值里如果有空格必须用引号引起来，否则语法报错；没有空格可以省略双引号。

* **批量与判断**

    | 场景 | 指令 |
    | :--- | :--- |
    | 一次性设置多个 field | `hmset key field1 v1 field2 v2 ...` |
    | 一次性获取指定的 field | `hmget key field1 field2 ...` |
    | 获取全部 field | `hgetall key` |
    | 删除指定 field | `hdel key field` |
    | 统计 field 的个数 | `hlen key` |
    | 判断 field 是否存在 | `hexists key field` |

    * `hmset user2 name Jerry age 110 job "Java coder"` 把三句合并成一句，`hmget user2 name age job` 再一次性取回。

    * `hgetall` 按存放顺序返回，每次取都是稳定的，不会乱。

    * `hlen` 统计字段个数；`hexists` 判断字段在不在，返回有或为 0。

    * `hdel` 删除指定字段。

    * > **结论**：这些指令的返回反映到 Go 里应该是切片或数组。

* **字段与字符串的关系**

    * 哈希的 field 不能重复。

    * > **易错点**：写进去的是整数也没用，等存到 Redis 里它都变成字符串了，取回来年龄也变成了字符串；所有数据的本质都是字符串，JSON 序列化之后也一样。

## 列表 List

* **结构与特点**

    * > **定义**：List 翻译成中文就是列表，是简单的字符串列表，按插入顺序排序。

    * 可以在头部添加也可以在尾部添加，像一根两头都没封口的管道。

    * > **结论**：list 的本质是一个链表，后面的数据结构课程会学。

    * 两个特点：元素有序；元素的值可以重复。

* **插入方向与取出顺序**

    ```bash
    lpush city beijing shanghai tianjin
    lrange city 0 -1
    ```

    * `lpush` 从管道左边往里扔，先扔的 beijing 会被后面的挤到后面，所以越晚扔的越靠左；取出来是天津、上海、北京，与存放顺序刚好相反。

    * `rpush` 从右边追加，先加的就在前面，再 `lrange` 取出来是先加的在前、后加的在后。

    ```text
       从左边加入： lp          从右边加入： rp

            ┌───┐                    ┌───┐
            │ C │                    │ A │
            └─┬─┘                    └─┬─┘
              │                        │
            ┌─┴─┐                    ┌─┴─┐
            │ B │                    │ B │
            └─┬─┘                    └─┬─┘
              │                        │
            ┌─┴─┐                    ┌─┴─┐
            │ A │                    │ C │
            └───┘                    └─┬─┘
                                       │
                                     ┌─┴─┐
                                     │ D │
                                     └─┬─┘
                                       │
                                     ┌─┴─┐
                                     │ E │
                                     └───┘

       取的时候默认从左边开始取
    ```

    * 链表里的指向永远是从前面的元素指向后面的元素，不会反过来；从尾部加入也是继续往后形成指向，不会改变方向。

* **范围语法**

    * 基本语法是 `key start stop`，返回指定区间内的元素。

    * 下标都以 0 为底，0 表示列表第一个元素，1 表示第二个，依次类推。

    * 也可以用负数下标：`-1` 代表最后一个，`-2` 代表倒数第二个，`lrange key 0 -1` 就是从第一个取到最后一个。

    * 想截取第一个到第二个就写 `0 1`。

* **弹出与删除**

    | 指令 | 行为 |
    | :--- | :--- |
    | `lrange` | 只取不移走，数据仍留在链表里 |
    | `lpop` | 从最左边取出并移走，弹出一个数据 |
    | `rpop` | 从右边弹出一个数据 |
    | `delete` | 直接把 key 干掉，整条链表消失 |

    * 例：`lpush herolist aaa bbb ccc` 之后 `lpop herolist` 弹出的是 ccc，剩下的 bbb、aaa；再 `rpush herolist ddd eee`，`lrange herolist 0 -1` 的顺序是 ccc、bbb、aaa、ddd、eee。

    * 链表由一个 key 值指向，所以想彻底干掉只要 delete 掉这个 key；之后再遍历会提示 `empty list or set`。

    * > **结论**：这个提示说明它可能是 list 也可能是 set，set 的遍历方式与之很像。

    * > **提示**：这类结构很实用，比如展示用户最近浏览的前十个商品有这种结构就非常轻松，没有的话还得按时间排序、动数据库。

* **使用细节**

    * `LINDEX` 按索引下标获取元素，从左边开始编号，从左到右、从 0 开始。

    * `LLEN` 返回当前 List 的长度；key 不存在时被解释为一个空列表，返回 0。

    * 数据可以从左边插也可以从右边插，按需求来。

    * 把所有值都 pop 完之后，对应的键也就没有了，List 自动消失。

## 集合 Set

* **特点与适用**

    * > **定义**：Redis 的 Set 是 string 类型的集合，默认无序，底层是 hash table 数据结构。

    * 两个特点：元素没有顺序；元素的值不可以重复。

    * > **易错点**：放进一个 aaa 再放 bbb 时，不重复这一点体现在同值再放会返回 0 表示没加进去。

    * 适用场景：存放人名但要求名字不重复；存放电子邮件，现实中两个相同邮箱邮件根本发不出去，所以用 Set 特别合适。

* **四条指令**

    | 场景 | 指令 |
    | :--- | :--- |
    | 往集合里添加元素 | `sadd key element [element ...]` |
    | 取出集合中所有元素 | `smembers key` |
    | 判断某元素是不是成员 | `sismember key element` |
    | 删除指定的值 | `srem key element` |

    * `sadd emails tom@sohu.com jacky@qq.com` 再 `sadd emails kkk@uu.com yyy@sohu.com`，`smembers emails` 取出来的顺序明显没有规律，后面加的 kkk 跑到了前面，证明无序。

    * 重复 `sadd` 一个已存在的元素返回 0，表示没有加进去。

    * `sismember` 判断成员，有返回 1、没有返回 0。

    * `srem` 就是 remove 的简写，删除成功返回 1、删除不成功返回 0；修改等于继续用 add。

## 课堂练习走法

* **Hash 题：学生信息增删改查**

    ```bash
    hmset stu1 name tom age 18 score 99 address beijing
    hgetall stu1
    hmget stu1 name score
    hlen stu1
    hexists stu1 score
    hdel stu1 address
    hgetall stu1
    ```

    * `hexists` 正好可以用来判断成绩或地址有没有填。

* **List 题：中国四大名著按价值排名**

    ```bash
    rpush classics 红楼梦 三国演义 西游记 水浒传
    lrange classics 0 -1
    ```

    * 必须用 `rpush`：因为 `lrange` 从左边开始取，要排出从高到低的顺序，就得让《红楼梦》出现在最左边。

    * 换成 `lpush classics 红楼梦 三国演义 西游记 水浒传`，取出来正好是反过来的《水浒传》《西游记》《三国演义》《红楼梦》。

## Go 客户端：redigo 安装

* **为什么要装第三方库**

    * > **结论**：因为要用到操作 Redis 的 API，而 Redis 不是 Go 语言本身的组件——Redis 是 Redis，Go 是 Go，所以要装一个 Redis 开发的开源包。

* **安装步骤**

    ```bash
    go get github.com/garyburd/redigo/redis
    ```

    * 这条指令要在自己的 GOPATH 路径下执行，不能在别的地方执行。

    * > **注意**：执行之前一定要确认已经安装 Git，装 Git 时要配一下它安装到哪个目录，写到这个 `bin` 目录。

    * `git --version` 能显示版本信息就说明 Git 装好了。

    * 装完之后 `src` 目录下会多出一个 `github.com` 目录，里面就是操作 Redis 的各种函数。

    * 也可以把整个 `github.com` 目录打包发给同学，直接解压到 `src` 下使用，但必须保证路径就在 `src` 下、起始目录是 `github.com`。

    * 进到 GOPATH 目录执行 `go get`，没有报错就是成功。

## Go 连接 Redis

* **`redis.Dial`**

    ```go
    package main

    import (
    	"fmt"

    	"github.com/garyburd/redigo/redis"
    )

    func main() {
    	conn, err := redis.Dial("tcp", "127.0.0.1:6379")
    	if err != nil {
    		fmt.Println("redis Dial err=", err)
    		return
    	}
    	defer conn.Close()

    	fmt.Printf("connect success, conn=%v\n", conn)
    }
    ```

    * 和原先写 TCP 非常相似：`Dial` 拨号，第一个参数是网络类型，第二个是地址，端口默认 6379。

    * 要连别的 IP 就把 IP 写清楚；远程连接要写对方地址，具体问题具体分析。

    * 返回 `conn` 和 error，error 不为 nil 就是拨号失败，连不成功就直接走人。

    * > **提示**：打印出来的那一长串是套接字 conn 的结构体字段，里面有指针之类的信息，看着乱但说明连上了。

* **看源码代替查手册**

    * 网上手册比较少，看源码一样能看懂；包里最重要的文件是 `redis.go`，`Dial` 实际定义在 `conn.go` 里。

    * > **定义**：`Dial connects to the Redis server`，先传 network 和地址，后面是一些选项，最后返回 Conn 和 error。

    * `conn` 是一个结构体，里面有很多字段，同时绑定了很多方法：`Close`、`writeLen`、按字节写等。

    * > **结论**：最重要的方法是 `Do`，可以写命令行，返回的信息是**空接口类型**，也就是可以返回任意数据类型。

## 用 Do 执行指令

* **Set 与 Get**

    ```go
    _, err = conn.Do("Set", "name", "tomjerry")
    if err != nil {
    	fmt.Println("Do Set err=", err)
    	return
    }

    r, err := conn.Do("Get", "name")
    if err != nil {
    	fmt.Println("Do Get err=", err)
    	return
    }
    fmt.Printf("name = %v\n", r)
    ```

    * `Do` 的第一个参数是命令名，后面是一堆可变参数，最后返回任意类型；指令大小写无所谓，一般喜欢写首字母大写。

    * 添加数据时一般忽略返回的结果，只要 error。

    * > **结论**：`Do` 就是把客户端里敲过的那些指令搬过来，只不过客户端用空格分隔参数，Go 这里用逗号分隔。

* **返回值的类型转换**

    * 返回的 `r` 是 `interface{}` 空接口，直接输出看不出内容，要按实际情况转成对应类型。

    * 用类型断言 `r.(string)` 转字符串，注意断言里是小写的 `string`。

    * > **易错点**：故意把值改成中文「汤姆猫猫」时，类型断言会报接口转换出错，所以不要这么转。

    * 更好的做法是用 redis 包自带的转换方法，一步到位：

    ```go
    r, err := redis.String(conn.Do("Get", "name"))
    ```

    * > **结论**：转成字符串就用 `String`，是 int 就用 `Int`，是 float 就用 `Float`，要转多个就加个 s 用 `Strings`。

    * redis 文档里的 `Go Type` 一节列了 `string`、`int`、`float64`、`bool` 等类型；`Do` 会在必要时把参数转换为二进制，`error`、`integer`、`simple` 这些类型都支持转换。

    * 转成 string 之后可以用反射做进一步处理，但一般没必要，需要的结构就足够了。

    * ⚠️ `defer conn.Close()` 一定要写，不关是非常危险的操作，做服务器时可能瞬间爆棚。

## Go 操作哈希

* **单个字段读写**

    ```go
    _, err = conn.Do("HSet", "user01", "name", "john")
    _, err = conn.Do("HSet", "user01", "age", 18)

    r1, err := conn.Do("HGet", "user01", "name")
    r2, err := conn.Do("HGet", "user01", "age")

    name := r1.(string)
    age := r2.(int)
    fmt.Printf("name = %v, age = %v\n", name, age)
    ```

    * 操作哈希只需把 `Do` 的指令从 `Set` 换成 `HSet`，然后按哈希的数据组织形式往里扔。

    * `HSet` 是单个字段单个字段地赋值，取的时候也必须写清是哪个哈希里的哪个字段。

    * 取到的两个结果要分成 `r1`、`r2` 两个变量，不然容易混。

    * 想一次性全取可以用 `HGetAll`，返回一个可转成集合的结果，但麻烦在于还得一个个再转一次。

* **批量读写**

    ```go
    _, err = conn.Do("HMSet", "user02", "name", "john", "age", 19)

    r, err := redis.Strings(conn.Do("HMGet", "user02", "name", "age"))
    if err != nil {
    	fmt.Println("HMGet err=", err)
    	return
    }
    for i, v := range r {
    	fmt.Printf("r[%d] = %s\n", i, v)
    }
    ```

    * `HMSet` 一次性给多个 field 赋值；`HMGet` 取多个字段，取的时候要写清要取哪些信息。

    * 返回多个值要转成切片或数组，取的时候用遍历的方式，Go 里它类似切片或者 map 这种东西。

    * 多个用户就是多个 `for` 循环而已。

* **EXPIRE**

    * 设置有效时间就是在 `Do` 里写 `EXPIRE` 后面跟 key 和时长，比如给 `name` 这个 key 设 10 秒钟；也可以给 Hash 对应的 key 设时长。

    * List 的操作指令跟前面一样，手册里有大量；用好之后海量用户即时通讯就可以做起来了。

## 连接池

* **为什么要连接池**

    * 传统写法是 `redis.Dial` 拨一次号、用完再 `Close`，这种方式效率有时候比较低。

    ```text
       传统方式：每次都重新来一遍

       Go 程序 ──Dial──▶ Redis   （Redis 临时创建一条连接，耗费资源）
            │
            └──Close──▶            （用完就扔，下一次再重新建）

       客户端一多，每次都要重新建，重复劳动


       连接池：事先把连接建好放着

            ┌─────────────────────────────┐
            │         连接池 pool         │
            │  ┌───┐  ┌───┐  ┌───┐  ┌───┐│
            │  │ 1 │  │ 2 │  │ 3 │  │ 4 ││   连接提前建好，不关闭
            │  └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘│
            └──┼────────┼────────┼────────┼──┘
               │        │        │        │
       Go 程序─Get─┘  ─Get─┘   ─Get─┘   ─Get─┘      取一个用一下
          归还 ◀────归还 ◀────归还 ◀────归还       用完放回池中
    ```

    * 服务器为每个客户端临时创建一条连接再返回，是很耗费资源的。

    * 连接池的做法是上来先分配好一批可用连接，放在池子里不关闭；创建多少由程序员按项目规模决定。

    * 需要操作时直接从池子里取一条，用完不要再关，它会自动回到池子里继续和数据库保持连接。

    * > **结论**：事先初始化一定数量的连接放入连接池，需要时取出、用完回放，可以节省临时获取连接的时间，提高效率；连接池由 Go 的连接池技术负责维护。

    * 类比打电话：接通后不挂断，一直保持连接不占线，别人想用也不用重新拨号。

* **关键参数**

    | 参数 | 含义 |
    | :--- | :--- |
    | `MaxIdle` | 最大空闲连接数 |
    | `MaxActive` | 和数据库的最大连接数，0 表示没有限制 |
    | `IdleTimeout` | 最大空闲时间 |
    | `Dial` | 初始化连接的代码，指定连接哪个 IP 的哪个 Redis |
    | `Get()` | 从连接池中取出一个连接 |
    | `Close()` | 关闭连接池 |

    * `MaxIdle` 是最大空闲连接数，比如设成 8 表示池子里最多放 8 个空闲连接。

    * `MaxActive` 和最大空闲数不一样，它指通过连接池最多能和数据库发多少个连接，因为还有并发的问题。

    * > **提示**：写 0 只是连接池技术本身不去限制，实际能不能达到还跟操作系统有关；如果你自己把 `MaxIdle` 设成 8，最大连接数也就是 8 了。

    * `IdleTimeout` 是最大空闲时间，一个连接放回池后 100 秒内没人再用过就达到最大空闲；连接池有自动生长的机制，达到峰值时会自动增长。

    * `Dial` 指定要连哪个 IP 的哪个 Redis，`localhost` 代表本地，端口号也要写清楚。

    * ⚠️ `Close` 一旦关闭连接池，就不能再从里面取连接了。

    * `pool.go` 里 `Pool` 是个结构体，所以 `redis.Pool` 实际返回的是它的指针；字段还包括 `Wait`、`MaxConnLifetime` 等，`IdleTimeout` 用的是 `time` 包里的 `Duration`。

    * 标准配置里 8 和空闲时间都可以按项目大小往上调，性能不够时把数据量适当调大即可。

* **代码用法**

    ```go
    package main

    import (
    	"fmt"

    	"github.com/garyburd/redigo/redis"
    )

    // 第一步：定义一个全局的 pool，任何一个地方都可以使用到
    var pool *redis.Pool

    // 当启动程序时，就初始化连接池（init 在 main 之前执行）
    func init() {
    	pool = &redis.Pool{
    		MaxIdle:     8,   // 最大空闲连接数
    		MaxActive:   0,   // 表示和数据库的最大连接数，0 表示没有限制
    		IdleTimeout: 100, // 最大空闲时间
    		Dial: func() (redis.Conn, error) { // 初始化连接，连接哪个 ip 的 redis
    			return redis.Dial("tcp", "localhost:6379")
    		},
    	}
    }

    func main() {
    	// 第二步：从 pool 里取出一个连接
    	conn := pool.Get()
    	defer conn.Close()

    	// 往 Redis 里放一个东西
    	_, err := conn.Do("Set", "name", "汤姆猫")
    	if err != nil {
    		fmt.Println("conn.Do err=", err)
    		return
    	}

    	// 再取出来
    	r, err := redis.String(conn.Do("Get", "name"))
    	if err != nil {
    		fmt.Println("conn.Do err=", err)
    		return
    	}
    	fmt.Println("r=", r)
    }
    ```

    * 用连接池必须满足两点：程序运行前先初始化 pool，可以把初始化放在 `init` 里；然后才去使用。

    * `pool` 必须定义成 `*redis.Pool` 指针，因为它的 `Get` 方法是跟指针类型关联的，不是指针类型就用不了。

    * `Get` 之后返回的是一个 `Conn`。

    * > **易错点**：`defer conn.Close()` 依然要有，不然用完不关闭连接就回不到池子里，池子只会越用越少，别人就都拿不到连接了。

    * `conn.Do` 放东西时会返回结果和一个 error，结果一般不要，error 必须接住判断，取不出来就 return。

* **关闭连接池的坑**

    * > **易错点**：`pool.Close()` 之后调 `pool.Get()` 不会立刻报错，拿到的 `conn` 甚至看起来是正常的，真正的错误要等到用它去 `Do` 的时候才爆出来，提示 `redigo: "get on closed pool"`。

    * 猜测是因为这个 conn 只是指向连接池的一个引用，`Do` 的时候才正式去取连接。

    * 调试时很容易被这种"半死不活"的连接绕进去，所以连接池一定不要关闭，关闭了就没法玩了。

## go-redis 方案

* **另一种客户端**

    * 除了前面讲的方案，网上还有一种叫 `go-redis` 的方式，思路一样：先装插件，再引入。

    * 它创建连接时可以指定地址、密码和用哪个数据库，字符串、List、Set、Hash 的操作都有对应的指令和函数。

    * > **结论**：区别只在于它把指令直接当作函数名来用，前面那套是把指令写进 `Do` 的参数里；本质还是要学会指令本身。

    * 项目经理要求用 `go-redis` 就用它，没有要求用前面那套也行，本质上区别不大；不管用哪个，底层的原理才是要搞清楚的。