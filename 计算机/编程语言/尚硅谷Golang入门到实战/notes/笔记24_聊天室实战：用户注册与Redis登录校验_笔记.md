# 聊天室实战：用户注册与 Redis 登录校验

## Redis 校验的动机与拆步骤

* **登录不能写死**

    * > **定义**：直接把用户 ID 和密码写死在判断条件里在真实开发中不可能，登录必须到 Redis 里验证。

    * 第一步：先手动往 Redis 里添加一个测试用户，然后让登录成功。

    * 第二步：登录跑通之后再完成一个真实的注册。

    * 后面这一步才是真正的落库，前面那步只是为了先打通链路。

* **两步的关系**

    * > **结论**：只要这两步打通，后台这一大块内容就通了。

    * 协议定制、框架改进这些前期工作都在完成登录那一块干掉了，后面每加一个功能基本就是照着套路写。

---

## 用户数据的存储结构

* **选 hash 的理由**

    * > **定义**：用一个 hash 存用户，key 是用户 ID，value 是序列化后的用户信息。

    * 需要挑一种相对高效而且 key 不能重复的数据结构。

    * list 队列直接往里扔东西，看不出有什么效果，不合适；hash 比较合理。

    * 查用户时只要通过 hash 名 `users` 找到对应字段（也就是 key），有值说明用户存在，没有就说明不存在。

```text
users （hash）
 ├─ "100" ──▶ "{\"userId\":100,\"userPwd\":\"123456\",\"userName\":\"Scott\"}"
 └─ "200" ──▶ "{\"userId\":200,\"userPwd\":\"abc123\",\"userName\":\"Kai\"}"
```

* **用户字符串从登录的 Data 里抄**

    * > **定义**：手动添加用户时，用户字符串的格式直接照抄客户端登录时发过来的 `Data` 内容。

    * 登录时 `userProcess` 发出的那个结构化字符串，就是待保存的用户字符串：

    ```text
    {"userId":100,"userPwd":"123456","userName":"Scott"}
    ```

    * 以这个结构作为 User 结构体的字段设计依据。

* **hset / hget 命令**

    ```bash
    hset users 100 "{\"userId\":100,\"userPwd\":\"123456\",\"userName\":\"Scott\"}"
    hget users 100
    ```

    * `hset` 重复执行相当于修改，重新添加时会覆盖原来的值。

    * 第一次 `hget` 能拿到 ID 和密码但名字是空的，是因为命令里没带 `userName` 字段，需要再 `hset` 一次补上名字。

    * > **易错点**：⚠️ 字段名如果**是数字就千万不要用双引号引起来**，硬加时引起来后果很严重，而且这个错误特别不好调。

---

## model 层

* **model 层要加的三个文件**

    * > **定义**：服务器端此前没有描述用户的数据结构，所以要加一层 model 层，model 相当于我们的数据。

    * `user.go`：定义 User 结构体。

    * `userDao.go`：专门操作 User 的数据访问对象（Data Access Object），有些地方也叫 manager。

    * `error.go`：自定义错误信息，因为登录、注册的错误种类将来会比较多。

    * 另外可以再开一个空的 `service` 包占位，目前用不上。

* **User 结构体的字段**

    ```go
    package model

    // 定义一个用户的结构体
    type User struct {
        UserId   int    `json:"userId"`
        UserPwd  string `json:"userPwd"`
        UserName string `json:"userName"`
    }
    ```

    * 至少要有用户 ID、用户密码、用户名三项，这是最基本也最迫切需要的。

    * 还可以加性别、电子邮箱、对应哪些好友等等，扩展性很好。

* **tag 不是装饰品**

    * > **结论**：JSON 串的 key 必须和结构体字段上的 `json:"..."` tag 名保持一致，否则序列化、反序列化一定失败。

    * 因为往 Redis 里存的 User 序列化出来的 key 是小写的，结构体不加 tag 时反序列化根本不会成功。

* **自定义错误**

    ```go
    package model

    import "errors"

    // 根据业务逻辑的需要，自定义一些错误
    var (
        ERROR_USER_NOT_EXISTS = errors.New("用户不存在")
        ERROR_USER_EXISTS     = errors.New("用户已经存在")
        ERROR_USER_PWD        = errors.New("密码不正确")
    )
    ```

    * 实际开发中通常把错误统一自定义好管理，不会每次都把错误信息散写在程序各处，那样以后要统一改提示信息得挨个地方去找，太 low。

    * 三个错误各有用处：登录查不到人是用户不存在；注册时 ID 已被占用是用户已存在；密码比对不上是密码不正确。

---

## Dao 与 utils 的分工

* **各管一条线**

    * > **定义**：客户端和服务器端之间的交互走 utils，处理器和数据库之间的交互走 Dao。

    * `UserProcess`、`smsProcess` 要和 Redis 打交道时用 Dao 完成数据库操作。

    * 总结成一句话：**跟后台操作用 UserDao，跟前台操作用 utils 里的方法。**

* **为什么现在还没有 service 层**

    * 标准结构里控制层先调 service 层，service 层再调 model 层，这一层不能省。

    * 但目前项目比较简单、业务功能很清晰，一个 DAO 就能处理完，所以暂时不写 service 层。

    * > **定义**：当业务很复杂、service 之间会相互调用，或者一个 service 的业务要调用多个 DAO 来完成时，才会有 service 层。

    * 以后业务复杂了可以加上这一层，现在先建一个空的 `service` 包留着。

---

## 连接池

* **两种连 Redis 的方式**

    * > **定义**：一次操作就连接一次，效率明显偏低；更好的做法是事先初始化好连接池，操作时取出一条连接，用完再放回去，效率大大提升。

    * 连接池由**服务器自己维护**，不可能由 Redis 数据库维护——Redis 是谁创建的当然就是谁在维护。

    * 画图时把连接池放在 model 和 Redis 之间更合理，但它仍然属于服务器这一侧的范畴。

* **redis.go 的归属**

    * 连接池用一个文件 `redis.go` 来维护，这个文件属于初始化内容，直接放在 `main` 包里。

    * 放 `model` 或 `main` 都可以，看实际需求。

* **UserDao 必须持有 pool 字段**

    * > **结论**：既然 UserDao 要操作 Redis，它至少要有一个字段能很轻松地拿到连接池，直接把 pool 作为它的字段。

    ```go
    type UserDao struct {
        pool *redis.Pool
    }
    ```

    * 全局的 pool 放在 `redis.go` 里比较合理，不要放在 `userDao.go` 里。

---

## GetUserById：先按 ID 把人捞出来

* **方法签名**

    ```go
    func (this *UserDao) GetUserById(conn redis.Conn, id int) (user *User, err error)
    ```

    * 它是 Dao 层最基本的能力：给一个 ID，返回一个 User 实例；没有就返回一个 error。

    * 连接既可以由这里传进来，也可以由别的地方拿到再传，所以先在参数里留着。

    * 长文提前留了个问题：这个连接最终是从 pool 里塞过来的，写到登录函数那一层就清楚了。

* **redis.Conn 不是 net.Conn**

    * > **易错点**：⚠️ 这里的连接是 Redis 的 `redis.Conn`，不是网络编程里那个 `net.Conn`，两者完全不是一回事。

    * `redis.Dial` 拿到的连接类型是 redis 包里的 Conn，和 net 包那个连接不是同一个东西。

* **查询语句**

    ```go
    res, err := redis.String(conn.Do("HGet", "users", id))
    if err != nil {
        if err == redis.ErrNil {
            err = ERROR_USER_NOT_EXISTS
        }
        return
    }
    ```

    * 用 `conn.Do("HGet", "users", id)` 查询，返回的是字符串，所以要用 `redis.String` 包起来。

    * > **定义**：`redis.ErrNil` 代表没有查到，也就是这个 ID 在 hash 里不存在。

    * 命中 `redis.ErrNil` 时直接把 err 赋成已经定义好的 `ERROR_USER_NOT_EXISTS` 再返回。

* **必须先反序列化才能比对密码**

    ```go
    user = &User{}
    err = json.Unmarshal([]byte(res), user)
    if err != nil {
        fmt.Println("json.Unmarshal err=", err)
        return
    }
    return
    ```

    * `res` 取出来就是 JSON 字符串的样子，这个东西本身没有用，必须先反序列化成 User 实例才能取出 ID 和密码。

    * > **注意**：传给 `json.Unmarshal` 的应该是一个具体的、已经实例化的东西，不能传一个空指针，所以先在函数体里 `user = &User{}` 创建出这块空间，再把它传进去。

    * 函数末尾记得补一个 `return`，因为担心前面的 `if` 分支没进去。

    * 别忘了引 `encoding/json` 包。

---

## Dao 的 Login

* **Login 的签名与语义**

    ```go
    func (this *UserDao) Login(userId int, userPwd string) (user *User, err error)
    ```

    * > **定义**：用户存在且密码正确时返回一个 User 实例且 err 为 nil；有错时 user 是 nil，返回对应的错误信息。

    * 这个函数要被别的地方调用，所以首字母必须大写。

* **从池里借一根连接**

    ```go
    func (this *UserDao) Login(userId int, userPwd string) (user *User, err error) {
        conn := this.pool.Get()
        defer conn.Close()

        user, err = this.GetUserById(conn, userId)
        if err != nil {
            return
        }

        if user.UserPwd != userPwd {
            err = ERROR_USER_PWD
            return
        }

        return
    }
    ```

    * 第一步就是 `this.pool.Get()` 从连接池取出一条连接，取到马上 `defer conn.Close()`，用完才关闭。

    * 拿到连接后调用自己的 `GetUserById(conn, userId)`，把连接和 ID 都传进去。

* **两道判断对应两种失败**

    * 第一道：`err != nil` 就直接 return，这个错误要么是用户不存在，要么是反序列化出错，不用细分。

    * 第二道：拿到 User **不代表用户合法**，因为可能 ID 对而密码错，所以要比对 `user.UserPwd` 和传进来的密码。

    * 不匹配就赋 `ERROR_USER_PWD` 并 return。

    * 目前确定的就是这两种错误，还有可能是内部信息存错导致的未知错误，不好归类。

---

## NewUserDao 工厂方法

* **工厂模式注入 pool**

    ```go
    func NewUserDao(pool *redis.Pool) *UserDao {
        userDao = &UserDao{
            pool: pool,
        }
        return
    }
    ```

    * 只声明 pool 是 Redis 类型而没做初始化，直接拿来用是不行的，所以用工厂模式创建实例。

    * 它是一个函数而不是 `UserDao` 的方法，名字叫 `NewUserDao`。

    * > **定义**：必须给工厂方法传一个 pool 进来，返回值是指针类型 `*UserDao`。

    * pool 从哪来不用在这个函数里操心，只要保证服务器运行时它已经被初始化好即可；连接池肯定要事先创建好，不可能是用到才创建。

* **整个系统只要一个 Dao**

    * > **结论**：整个操作中只要有一个 UserDao 就够了，希望服务器一启动它就有了实例，需要用的时候直接拿，做成全局的。

    * 不要每次要连数据库才去创建一个 UserDao，那样连接池的优势很难发挥出来。

---

## initPool 与初始化的顺序

* **initPool 的三个参数**

    ```go
    func initPool(address string, maxIdle, maxActive int, idleTimeout time.Duration) {
        pool = redis.NewPool(&redis.Options{
            Address:     address,
            Idle:        maxIdle,
            MaxActive:   maxActive,
            IdleTimeout: idleTimeout,
        })
    }
    ```

    * `address` 是 Redis 的地址；`maxIdle` 是最大的空闲数；`maxActive` 是最大的连接数；`idleTimeout` 是最大的空闲时间。

    * 三个参数里的时间要用 `time.Duration` 类型，比裸写更精准。

    * > **易错点**：⚠️ 空闲时间不能直接填 `100` 这种数字，100 不是一个时间类型，不知道代表多长时间，先前那种写法是有问题的。

    * `redis` 包名特别长，直接从之前写的代码里拷过来即可。

* **为什么叫 initPool 而不是 init**

    * > **定义**：所有初始化工作统一放到 `main` 里一起调用，便于管理，不然东放一点西放一点，自己也不知道初始化了什么。

    * > **注意**：`initPool` **不会自动调用**，必须手动调用它。

    * 之所以拆出这么多个参数，是为了将来这些值可以从配置文件读取，不用每次改源代码。

* **调用时的取值**

    ```go
    func main() {
        initPool("localhost:6379", 16, 0, 300*time.Second)
        // ...
    }
    ```

    * 端口 6379；最大空闲连接数给 16；最大连接数给 0 表示不限制，也可以写个具体值；最长空闲时间给 300 秒。

    * 连接池数量也可以写成 32 根或 8 根，由自己决定；服务器一开启就初始化连接池。

* **initUserDao 与 MyUserDao**

    ```go
    func initUserDao() {
        MyUserDao = model.NewUserDao(pool)
    }
    ```

    * `MyUserDao` 是一个全局的指针变量，命名上和类型上加以区别。

    * 此刻它还没被实例化，实例化同样放在主文件里，希望服务器一启动它就有了。

    * `pool` 本身就是 `redis.go` 里定义并初始化过的全局变量，直接传进去就行，不需要参数传递。

* **顺序：先 pool 后 Dao**

    * > **注意**：⚠️ **一定要先初始化 pool，再初始化 UserDao**，也就是 `initUserDao` 的调用必须放在 `initPool` 之后。

    * 原因很直接：`initUserDao` 依赖 pool，如果先调它，那时 pool 还是空的，仍然得不到一个可用的 `MyUserDao`。

    * 依赖链是 `initUserDao` 依赖 `pool`，`pool` 依赖 `initPool` 去做真正的初始化。

    * 担心顺序被别人改乱的话，可以把两句一起塞进一个 `func init()` 里，这样代码更干净，也顺便用上了 `init` 这个知识点。

---

## 打通 UserProcess 与数据库

* **替换掉写死的判断**

    * > **定义**：把 UserProcess 里原来「ID 等于 100 且密码等于 123456」那段硬编码换掉，直接调用 Dao 的 Login。

    ```go
    user, err := MyUserDao.Login(loginMsg.UserId, loginMsg.UserPwd)
    if err != nil {
        return
    }
    ```

    * 之所以连创建都不用创建，是因为 `initUserDao` 已经把 `MyUserDao` 准备好了，拿来即用。

    * 用户名和密码本来就能从 `LoginMes` 里取到，原先是跟 100 比较，现在直接传进去。

    * 无错误时返回的码仍然是 200，这一部分没有改动。

* **先打出来 User**

    * > **结论**：编译器提示 User 没用到，但 User 显然非常有用——服务器端将来要拿到登录用户的全部信息，不可能只要 ID 和密码，一定要拿结构体。

    * 现在只是暂时没用到，先在后台打印一条「某某人登录成功了」。

    * 后面可以把它展现在服务器维护列表里，比如管理员打开列表看到谁在线、一点就能踢掉，那时这个 User 就非常有用了。

* **验证结果**

    * 客户端没有改动，不需要重新编译，只需要重新编译服务器端；前提是 Redis 处于运行状态，关着肯定要报错。

    * 用 100 / 123456 登录，后台打印出 100 123456 SCOTT 登录成功，说明这个 User 确实是从 Redis 里出来的。

    * 用 900 这个不存在的 ID 登录，提示「用户不存在，请注册再使用」，提示准确。

    * 手动往 Redis 里加的 900 号用户可以登录成功，但用户名取出来是乱码——这是终端显示的问题，不是数据的问题；通过程序注册添加的用户则没有这个问题。

---

## 让自定义错误真正被用上

* **别把错误信息写死在业务里**

    * > **结论**：已经定义好的错误信息却不用、自己傻乎乎写字符串，很不划算，而且以后统一改提示信息要挨个地方改，维护起来很痛苦。

    * 判断 `err` 等于哪个自定义错误，就说明是哪种情况，直接复用它。

    * 提示文案不用写在业务代码里，直接从 `err` 里取——它有 `Error()` 方法能把文字取出来，以后要改文案只改错误定义那一处。

    * 不同情况给不同的码值会更漂亮：用户不存在、密码不正确、未知错误各用一个。

    * 密码不对一般是 403（Forbidden 的意思）更贴切；未知错误可以写 505，也可以写成服务器内部错误，仁者见仁。

* **客户端写死的 500 该退休了**

    * > **结论**：不需要再判断错误码是不是 500，因为 `err` 里已经包含了所有信息；直接一个 `else` 分支，**码只要不等于 200 就都是错的**。

    * 错误信息直接打从 `Login` 返回的 `err` 里带出来的文字就行，一句话的事。

    * 之前的错误表现为「通讯端出错了」，原因就是客户端写死了「你一定是一个 500」，而实际返回的不是 500 就直接走了。

    * 改完之后：ID 存在密码错误提示「密码不正确」，ID 不存在提示「用户不存在，请重新注册」，两边都通。

---

## 注册的消息协议

* **message 包新增两个类型**

    * > **定义**：注册既然是一种消息，就要在 message 包里新增一个发送类型和一个回送类型，注册可以完全照着登录的写法来。

    * 发送类型直接把整个 `User` 结构体塞进去：`MessageType: "register", Data: user`。

    * > **易错点**：⚠️ 早期登录是一个个 `int`、`string` 字段往里写；现在既然已经有 `User` 结构体了，就不要再一个字段一个字段地加。

    * 好处很实际：将来 User 的字段增加时不用改一堆地方。

    * 当初之所以那么写，是因为那时还没有 User 结构体。

* **User 放到共用的目录**

    * 把 `user.go` 复制到 `common/message` 包下，包名改成 `message`，这样客户端也能用到这个 User 定义。

    * 公用的部分写在这里可以，各拷一份也行；长文采用的是放共用目录这个更简洁的方式。

    * > **提示**：字段名和类型同名不会报错，`User` 作为字段名和 `User` 作为类型是可以区分的。

* **注册响应类型**

    ```go
    type RegisterResMes struct {
        Code int    `json:"code"`
        Msg  string `json:"msg"`
    }
    ```

    * 形状和登录响应几乎一样，都有 Code 和错误描述。

    * 状态码定为：**200 表示注册成功，400 表示该用户已经被占用了**。

    * 两者其实可以共用，但从扩展性考虑还是分开写——保不齐 Register 将来会多一个字段。

---

## 客户端的注册流程

* **接收用户输入**

    * 用户 ID：提示「请输入用户的ID」，用 `fmt.Scanf` 读，因为是整数所以用 `%d\n`。

    * 用户密码：提示输入密码，前面已定义过全局变量，直接用它。

    * 用户名字：也就是昵称，前面没有这个变量，先增加一个 string 类型的 `userName` 来接收。

    * > **提示**：真实产品里用户 ID 最好由后面自动分配，这里让用户自己输入，是为了让大家体验一下用户 ID 存在时是怎么返回的。

* **造一个 UserProcess 交给它去办**

    * 套路已经成型：创建一个 `UserProcess`，让它去完成注册，这一步不需要再动脑筋。

* **Register 方法签名**

    ```go
    func (this *UserProcess) Register(userId int, userPwd string, userName string) (err error)
    ```

    * 登录填两个信息，注册填三个信息，多出来的就是 `userName`。

    * 签名太长可以换行写，返回的错误信息该怎么返回还怎么返回。

* **必须改的几处**

    * 消息类型改成 `RegisterMesType`。

    * 变量名改成 `registerMes`，类型是 `RegisterMes`。

    * > **易错点**：⚠️ 里面的字段多了一层 User，不能再写 `registerMes.UserId, registerMes.UserPwd`，必须写成 `registerMes.User.UserId, registerMes.User.UserPwd, registerMes.User.UserName`。

    * `userName` 那一处也容易漏改，不改待会儿就会错。

* **序列化、发送、收包**

    * 先序列化 `registerMes` 塞进 `Data`，再对整个 Message 序列化，这些将来都可以再封装成小函数。

    * 第七步发长度的老规矩照抄；后面那一整段 write 其实已经有 `WritePkg` 可以用，创建 `transfer` 实例直接调就行。

    ```go
    err = tf.WritePkg(data)
    if err != nil {
        fmt.Println("注册发送信息出错 err=", err)
        return
    }
    ```

    * `WritePkg` 要的就是 byte 切片，`data` 本来已经是字节切片，不用转换。

    * 发完要等对方回一个包回来，用 `tf` 再 `ReadPkg` 读，读到的 message 应该就是 `RegisterResMes`，因为有发必有回。

* **判断结果后直接退出**

    * 注册这一版的做法是简单处理：注册完就退出，重新登录。

    * 先把 message 里的 `Data` 反序列化成 `registerResMes`，区分一下而不是直接复用登录的类型。

    * 等于 200 就提示「注册成功，可以重新登录」；不等于 200 说明注册有各种信息错误，把错误信息打出来。

    * > **易错点**：⚠️ `Register` 是被 `main.go` 调用的，外层是死循环菜单，退不出去，所以在 `main` 里 `os.Exit(0)` 直接退，成功失败都不玩了，让你重新登录。

    * 直接写 `os.Exit(0)` 编译器会报「少了一个 return」——因为成功或失败两条路都退不出去了，正好用 `os.Exit` 让编译器接受。

    * 要先把 `os` 包引进去。

* **但这条线还没接完**

    * 客户端这边写完还没用，因为服务器端没有把发过来的用户入库，这条线还没被处理。

```text
客户端                        服务器端
  |                              |
  |  ① Scanf 收 ID/密码/昵称       |
  |                              |
  |  ② Register(userId,pwd,name) |
  |     序列化 RegisterMes         |
  |     塞进 Message.Data         |
  |     再序列化整个 Message       |
  |                              |
  |  ③ 先发长度，再发 data        |---->
  |     tf.WritePkg(data)         |
  |                              |
  |        等待服务器入库并回应     |
  |                              |
  |  ④ tf.ReadPkg() 读回包   <----|
  |     反序列化 Data 得到        |
  |     RegisterResMes            |
  |                              |
  |  ⑤ Code==200 ? 成功 : 打错    |
  |     一律 os.Exit(0) 重新登录   |
```

* 注册请求和登录走的是一模一样的通信骨架：组包、序列化、先长度后数据、发出、等回包；区别只有消息类型从登录换成注册，以及 `Data` 里装的是一个 `User` 结构体而不是散字段。

---

## 服务端的注册分派

* **分派链路已经很简单了**

    * `main.go` 监听并拿到连接 → 交给 process → process 交给总控 → 总控读包后交给 `ServerProcessMes` → 它按消息类型分别处理。

    * 已经在 message 里加了 `RegisterMesType`，就在总控里加一个对应的 case，让 `UserProcess` 去完成注册。

    * 复制一份登录的处理代码改成调用注册即可，方法还没有写，就叫 `ServerProcess.Register`。

    * > **结论**：写一个东西时只需要找到对应文件往里写函数，调用就自然出来了，原先很费劲的地方现在轻松了很多。

* **UserProcess 实例只能写在 case 里**

    * > **易错点**：⚠️ 不要为了少写几行把创建 `UserProcess` 的代码提到 switch 外面，那样做是错的。

    * 现在恰好登录和注册都是 `UserProcess` 做的，看起来提到外面更简洁；但将来来第三、第四、第五种类型，如果用的不是 `UserProcess`，提到外面就白创建了。

    * 所以仍然要做成 case 里面的局部变量。

---

## UserDao 的 Register

* **从 message 里取 Data 反序列化**

    * 注册要传的已经不是单个字段了，`message.Data` 就是 `RegisterMes`，里面有一个 `User`。

    * 要反序列化出来拿到 `User` 结构体，因为得靠它验证这个 ID 在服务器里是不是已经有了。

    * 从 message 取和不取都行，因为 `User` 本来就已经在手里了，语义是一回事。

    * > **定义**：注册时传的是 `User` 的指针。

* **判重方向：没报错才是已存在**

    ```go
      user, err = this.GetUserById(conn, userId)
    ```

    * > **结论**：`GetUserById` 没报错反而说明用户**已经存在**了，因为 ID 能在 Redis 里查到；报错才说明可以继续注册。

    * > **易错点**：⚠️ 这一段的判断方向特别容易搞反，「get 没报错」意味着用户已存在，「get 报错」才意味着可以继续注册。

    * 查的时候取的是 User 里的 `UserId` 作为查询条件，因为调用方已经把 ID 传过来了。

    * `err == nil` 就赋一个「用户已存在」的错误信息返回；确实有错误说明该用户还没注册过、ID 还没进 Redis，可以往下走去入库。

    * get 出来的用户还要再反序列化一次，拿到 User 实例才能继续。

* **序列化与入库**

    * 把这个 User 序列化成字符串 `data`，这一步是**序列化**（把结构体换成字符串），出错就打印并 return。

    * 入库就是再调 `conn.Do`，用 `HSet` 写进 `users` 这个 hash，key 取 user 的 `UserId`，值是刚序列化出来的 `data`。

    * 因为是字节所以要转成 `string`；判断 `err` 不为 nil 时提示「保存注册用户出错」，最后补一个 return。

    * `HSet` 里取 UserId 本来就是 int，再转一次是多余的，可以去掉。

    * > **提示**：`users` 这个 key 最好做成全局常量，长文这里没有做常量，属于不够好的地方。

* **两个包里的 User 不是同一个类型**

    * > **易错点**：⚠️ 结构体长得再像，只要定义在不同的包里，Go 眼里就是两个不同的类型。

    * 现象是编译报「不能使用这个类型，因为它是个指针」；解决办法是在 Dao 里按原先的思路用 `message` 包里的 User，并把 message 包 import 进来。

    * 另一处还要注意该带指针的地方老老实实带上指针。

---

## 注册的响应与状态码

* **照猫画虎地写响应**

    * 第一步从 message 里取出 `RegisterMessage`，把 `Data` 部分反序列化交给它。

    * 第二步声明一个 response message，因为客户端还在等着听注册到底成没成功。

    * 照抄登录的响应代码，改类型、改变量名即可。

* **三个状态码**

    | 状态码 | 含义 |
    | :--- | :--- |
    | 200 | 注册成功，此时错误信息不用给 |
    | 505 | 用户已存在，错误信息直接从自定义错误里取 |
    | 506 | 其它未知错误，提示「注册时发生未知错误」 |

    * 用户已存在这一支的文案不用自己写，因为自定义错误本身就是 error，可以直接取出它自己的错误描述。

    * > **提示**：500 用过了，所以另外挑了 505、506；判定标准很简单，**只要不是 200 就都是错**，具体数值之后可以再统一规划。

* **是序列化不是反序列化**

    * > **易错点**：⚠️ 要把 `RegisterResponse` 的信息进行**序列化**（组包前的编码），得到 `data`；不是反序列化。

    * 拿到 `data` 后不要忘了把它塞给 response，因为现在只是类型有了，还要组合到要返回的那个 Message 类型里。

    * 转成字符串交给总的 response message，对它再序列化一次，然后用 `transfer` 发出去即可。

---

## 注册链路的改动清单

| 端 | 位置 | 改动 |
| :--- | :--- | :--- |
| 公共 | `common/message/user.go` | 增加该文件，把 `User` 结构体放进来 |
| 公共 | `common/message/message.go` | 增加注册和注册响应两个新消息类型 |
| 客户端 | `client/process/userProcess.go` | 增加 `Register` 方法，完成请求注册的任务 |
| 客户端 | `client/main.go` | 调用 `Register`，传 `userId`、用户密码、用户名字 |
| 服务端 | `server/model/userDao.go` | 增加 `Register` 方法，处理注册的落库 |
| 服务端 | `server/process/userProcess.go` | 增加 `Register` 方法，处理注册 |
| 服务端 | `server/main/processor.go` | 总控里调用了注册请求 |

```text
客户端 Register                 总控 case                 UserProcess            UserDao
      |                            |                          |                    |
      |  RegisterMes(注册包)  --->  |                          |                    |
      |                            | 读包，按 Type 分发         |                    |
      |                            |--- case RegisterMes ---->  |                    |
      |                            |   （局部 new，不提到外层）   |                    |
      |                            |                          |  反序列化 Data      |
      |                            |                          |  得到 User 指针 ---->|
      |                            |                          |                    |  GET users <UserId>
      |                            |                          |                    |   err==nil -> 存在
      |                            |                          |                    |   err!=nil -> 继续
      |                            |                          |                    |  Marshal(User)
      |                            |                          |                    |   HSET users <id> <data>
      |                            |                          |                    |
      |                            |                          |  505 已存在 / 506 未知 / 200 成功
      |                            |                          |--- RegisterResMes -->|
      |  <--- RegisterResMes ------|--------------------------|                    |
```

* 注册请求从总控的 case 进入，`UserProcess` 只做编排，真正的判重与落库在 `UserDao`。

* 联调实测：注册 680 号、密码 abc123、昵称 Jackie，提示注册成功；到 Redis 里 `hget users 680` 能查到这个用户；再用 680 / abc123 登录也成功；再次注册 680 会提示「用户已存在」。

---

## 登录时返回在线用户列表

* **这个功能为什么特别烧脑**

    * > **定义**：烧脑的意思是写着写着突然感觉 CPU 和内存不够用了——因为原来写的是单机版，现在变成多核了。

    * 描述听起来很简单，实际上是这一节特别费劲的部分；想通了会觉得很爽，想不通可能调好几天都调不出来。

* **维护在线列表真正要抓的是连接**

    * > **结论**：维护在线列表的目的不是维护用户的 ID 和信息本身——数据库里有信息你还维护这个干什么，**最主要是想办法维护服务器跟各个客户端的连接，关键就是跟连接要抓住**。

    * 连接忘了就啥事都干不了，光有用户信息没有用。

    * 有了连接才可以群发、点对点私聊，甚至发图片、视频、声音。

* **绕一圈回到 UserProcess 的 Conn**

    * 服务器端 `UserProcess` 本身就维护了一个 `Conn`，所以 map 的 value 用 `*UserProcess` 指针即可。

    * key 是用户 ID（int），value 是这个用户对应的 `*UserProcess` 指针。

    * 用指针而不是拷贝结构体，就是因为要拿到 `UserProcess` 里那个 `Conn`。

```text
        onlineUsers  map[int]*UserProcess
        ┌────────┬──────────────────────────┐
        │  key   │  value（指针）            │
        ├────────┼──────────────────────────┤
        │  100   │ ──▶ UserProcess{ Conn, … }│
        │  200   │ ──▶ UserProcess{ Conn, … }│
        │  300   │ ──▶ UserProcess{ Conn, … }│
        │  400   │ ──▶ UserProcess{ Conn, … }│
        └────────┴──────────────────────────┘
            ▲
            │  key = 用户 ID（int）
            └──▶ value 抓的是 Conn，不是信息
```

* **userMgr 的增删改查**

    * 新建一个 `userMgr.go`（后续也写作 `userManager.go`）放在处理层，也就是控制层。

    * 它维护一个叫 `onlineUsers` 的 map，map 比切片用起来方便。

    * > **定义**：在线列表要有增、删、改、查全部，因为用户可能离线，离线就得把它从列表里干掉。

    * 各客户端的连接彼此不同，所以必须由每个 `UserProcess` 各自持有自己的连接。

* **在 LoginResMes 里加一个切片**

    * 现在的 `LoginResMes` 只有 code 和 error 两个字段，不够用。

    * > **定义**：在 `LoginResMes` 结构体里增加一个 `Users []int` 字段，把当前在线用户的 ID 切片一起返回。

    ```go
    Users []int
    ```

    * 有多少个用户就遍历起来返回，客户端解析一下就能看到，登录那一刻一并返回。

* **这一版还有缺陷**

    * > **易错点**：⚠️ 这一版只有在**登录那一瞬间**才能拿到列表；登录完之后再有新用户登录，就不知道对方上线了。

    * 补这个缺口要靠后台那个偷偷跟服务器交互的协程，先把第一步做完。