# 代码生成器源码与 Velocity 模板

## 生成请求链路

### 生成页入口与 handleGenTable

* **生成按钮的两个入口**

    * > **定义**：页面上有两个生成按钮——一个在左上角负责批量生成，一个在每行业务表右侧负责单表生成，但两者调用的是同一个 `handleGenTable` 方法与同一套后端接口。

    * 生成按钮位于生成页 `index` 视图组件内，全局搜索该组件即可定位到按钮绑定与点击事件。

    * 搜到第二个按钮同样调 `handleGenTable`，这不是找错代码。

* **handleGenTable（row, type）的四步**

    * 第一步取表名：单表（`type === 0`）取当前行的 `row.tableName` 组成单元素数组，批量则取 `tableNames.value`，再用 `join(",")` 拼成逗号分隔字符串。

    * 第二步判空：`tableName` 为空时弹出「请选择要生成的数据」并 `return` 中断方法，与导入功能的空值校验写法一致。

    * 第三步区分出口：`type` 决定是自定义路径（代码直接落到指定路径下方）还是走默认下载器。

    * 第四步发起请求：`download("/tool/gen/batchGenCode?tableName=" + tableName, "ruoyi.zip")`。

    * > **易错点**：发出去的是 **GET 请求**，表名拼在 URL 查询参数后面，多张表用逗号分隔；浏览器 F12 里能直接看到路径 `batchGenCode` 后带着全部表名和 zip 下载过程。

### download.js 下载器

* **抽取好的下载文件方法（download(url, name)）**

    * > **定义**：一个把后端地址与目标文件名交给浏览器另存为的通用下载方法，签名只有两个参数——`url` 是后端地址，`name` 是下载后的文件名及扩展名。

    * 处在前端目录结构的 `plugins` 插件目录下。

    * 拼 `baseURL + url` 得到完整后端请求路径。

    * 用 `ElMessage.loading("正在下载数据，请稍候")` 弹出下载中提示。

    * 通过 axios 发 **get** 请求，头部携带 `Authorization: "Bearer " + getToken()`，`responseType` 设为 `blob`，拿二进制流存盘。

### batchGenCode 入口方法

* **批量生成入口（batchGenCode(response, tableName)）**

    * > **定义**：代码生成模块 Controller 上的 `batchGenCode` 方法，负责处理前端请求、执行代码生成逻辑，并把 zip 压缩包响应回客户端。

    * `response`：作为响应流，返回下载文件。

    * `tableName`：接收前端传来的逗号分隔表名字符串。

    * Controller 只做两件事：`Convert.toStrArray(tableName)` 把逗号串转成字符串数组；再调 `genTableService.generatorCode(tableNames)` 拿到字节数组。

    * 生成字节数组才是关键一步，Controller 与前端都只是在搬字节。

### Service 第一层：字节流与 zip 流

* **generatorCode(String[] tableNames)**

    * > **定义**：Service 里的同名方法共两层，这一层只负责把字节数组灌进 zip 流，不认识任何一张表长什么样。

    * 第一步 `new ByteArrayOutputStream bos`，用来存 zip 的压缩数据。

    * 第二步 `new ZipOutputStream zos(bos)`，把生成的代码压成 zip，便于网络传输。

    * 第三步遍历表名数组，每张表调一次 `generatorCode(tableName, zos)`，生成结果全部交给同一个 zip 输出流。

    * 第四步 `zos.finish()` 释放流，再 `bos.toByteArray()` 返回整个 zip 的字节数组。

### Service 第二层：单表生成的六步

* **generatorCode(单张表名, zip)**

    * > **定义**：整条链路里真正干活的方法，把一张表变成 zip 中的若干代码文件；整句翻译是「初始化模板引擎 → 设变量 → 加载模板 → 渲染模板 → 写进 zip」。

    * 业务表的基本信息对象内部**也包含了该表的字段信息**，是一个 `List` 集合存放该表的列信息。

    * 是子表时才会去给主表关联子表信息；单表操作时这一步不执行该分支。

    * 树形模板类型会额外加载树形数据相关信息，子表会在这里加载子表相关信息。

    * 第 5 步拿不到模板时直接结束，不进入渲染。

* **第一步：查表基本信息与字段信息**

    * 按方法签名 ctrl 跳进实现，实现里是一条 SQL：先查业务的基本信息表，再关联查询业务的字段信息表，查询条件是指定表名。

    * 返回的所有数据被封装到实体类中，拿到基本信息对象后当前业务表的所有数据都可获取。

* **第二步：判断是不是主子表**

    * 拿到表信息后先判断它是否为子表信息。

    * 是主子表则给主表关联子表信息；普通单表不执行这部分。

    * 这个判断在后面的模板加载处还会再用一次。

* **第三步：标记主键列**

    * > **定义**：`isPk` 是列信息上的主键标记，逻辑是遍历主表的列，命中主键列就 `set` 一下，告诉业务表哪个字段是主键。

    * 当前主表还有子表时规则相同：从子表里找主键列设置，找不到就把子表第一列设为主键列。

    * > **易错点**：主表**没有**主键列时会默认把第一个列当作主键列，这是数据库 InnoDB 引擎的默认设计，不用额外处理。

* **第四步：装配 Velocity 上下文对象**

    * > **定义**：模板上下文对象负责变量的填充——把数据库查询到的表信息全部设置成模板变量，Velocity 再把它们合并进模板文件。

    * 先通过业务表对象取模块名、业务名、包名、模板类别、功能名。

    * 模板类别共三种：单表的 CRUD、树表、主子表。

    * 前端类型共两种：vue2 与 vue3；默认用 vue2，若指定 element-plus 则改为 vue3。

    * 数据库中没有功能名时，会填充字符串「请填写功能名」。

    * 随后把类名、类名首字母小写、模块名、业务名首字母大写、业务名称、基础包名、包名、作者、生成时间、组件列、导入列的类型、权限列信息、表信息等**全部数据**塞进上下文对象。

* **第五步：加载模板列表**

    * 加载时传两个参数——模板类型与前端类型。

    * 后端模板路径对应代码生成模块 `resources` 目录下的 `vm` 目录，所有文件交给模板引擎对象加载。

    * 依次添加：Java 相关代码模板、数据库 SQL 文件、对应 XML，以及前端 api 模板文件路径。

    * 加载完毕后封装成集合返回上一层。

    * 加载的模板随模板类型不同而不同：

| 模板类型 | 页面上的说法 | 加载的模板 |
| --- | --- | --- |
| 单表 | CRUD | 只加载普通的 CRUD 模板 |
| 树表 | tree | 加载 `index_tree.java.vm` 这类带树结构的模板 |
| 子表 | sub | 加载主子表相关的模板 |

* **第六步：渲染模板并写入 zip**

    * > **定义**：渲染模板就是把准备好的模板变量**合并**到模板文件中的过程，通过加载好的模板变量结合字符输出流写入模板文件，从而生成代码文件。

    * 遍历模板列表，逐个执行合并渲染，再把生成好的代码文件添加到 zip 的字节输出流。

    * 循环直到所有模板都填充完毕，这个代码才执行结束。

* **一次生成请求的完整旅程**

    ```text
    【浏览器】勾选业务表 → 点「生成」
       handleGenTable(row, type)
            │  GET /tool/gen/batchGenCode?tableName=tb_a,tb_b,tb_c  + 令牌
            ▼
    【Controller】batchGenCode(response, tableName)
            │  tableName 逗号串 → String[]
            ▼
    【Service 第 1 层】generatorCode(tableNames)
            │  new ByteArrayOutputStream()  ← 存 zip 的字节
            │  new ZipOutputStream(bos)    ← 压成 zip
            │  for (表名 : 数组) → 逐张表生成
            ▼
    【Service 第 2 层】generatorCode(单张表名, zip)
            │  1. 查表基本信息 + 字段信息
            │  2. 是子表 → 再挂上子表信息
            │  3. 标记主键列
            │  4. 建 VelocityContext，塞入所有模板变量
            │  5. getTemplate(模板类型, 前端类型) → 没加载到就结束
            │  6. 遍历模板：merge(context, writer) → 写进 zip
            ▼
    【Controller 收尾】response.reset() → 响应头 → writeBytes(...)
            ▼
    【浏览器】download() → 另存为 ruoyi.zip → 解压得到前后端代码
    ```

### 响应流收尾与响应头顺序

* **writeZipResponse**

    * Controller 拿到 zip 字节数组后先 `response.reset()`，防止头部信息冲突，再设置响应头。

    * 全部设置完毕后，用 IO 工具类把字节数组写入 `response` 的字节输出流，返回客户端实现下载。

    * 前端拿到 zip 点保存、解压，就得到前后端代码。

* **响应头设置的固定顺序**

| 顺序 | 设置项 | 作用 |
| --- | --- | --- |
| 1 | `Access-Control-Allow-Origin` | 允许跨域访问 |
| 2 | `Access-Control-Expose-Headers` | 允许 JS 读取指定的响应头 |
| 3 | `Content-disposition` | 指定文件的下载名称 |
| 4 | `Content-Length` | 指定文件的大小长度 |
| 5 | `Content-Type` | 指定文件的 MIME 类型 |
| 6 | `Character-Encoding` | 指定字符集编码格式 |

## 生成器默认值配置

* **代码生成器配置文件（generator.properties）**

    * > **定义**：代码生成模块自带的配置文件，存放导入新表时自动套用的一组默认值，是改造代码生成器的入口。

    * 这些默认值不在前端表单里；表结构导入那一刻由代码生成模块读取配置文件并写入业务表记录。

* **四个默认值开关**

| 配置项 | 改成的值 | 作用 |
| --- | --- | --- |
| `author` | `it黑马` | 生成代码头部的作者署名 |
| `packageName` | `com.dk.manager` | 新表默认的生成包路径 |
| `autoRemovePre` | `true` | 开启自动去除表前缀的功能 |
| `removeTablePrefix` | `tb_` | 要去除的表前缀 |

    * 配置内容形如：

      ```properties
      # 代码生成默认值，导入新表时自动套用

      # 作者
      author=it黑马

      # 默认的生成包路径
      packageName=com.dk.manager

      # 是否自动去除表前缀
      autoRemovePre=true

      # 要去除的表前缀，多个以逗号分隔
      removeTablePrefix=tb_
      ```

    * 改完按 `Ctrl + F9` 热部署更新，再回浏览器测试。

    * > **易错点**：要去除多个前缀时，它们之间以**逗号分隔**，别写成空格。

* **改完必须重新导入才生效**

    * 已导入表的默认值是在**导入那一刻**从配置文件读出并存库的，改配置不会影响库里已存的老值。

    * 所以必须先把旧表**删除掉，重新完成表结构的导入**，才会读取最新配置。

    * 重新导入后再翻到第二页点编辑，可以看到实体类前缀 `tb_` 已消失、作者变为 `it黑马`，生成信息里包路径与模块名都变成 `manager`。

## Velocity 模板引擎

* **Velocity（模板引擎）**

    * > **定义**：基于 Java 语言的模板引擎，允许用特定语法在模板中嵌入 Java 的对象数据，实现界面设计与业务逻辑相分离，让界面设计更灵活、代码更易维护。

    * 「界面设计」即**模板**，是固定的文本结构；「业务逻辑」即**数据模型**，是动态变化的数据。

    * 它的作用是把动态数据**合并**到模板当中，然后输出到指定格式的文件中。

    * 电子发票的比喻：发票格式固定属于模板，每次开票人信息不同属于数据模型，Velocity 把实际开票人信息填进含占位符的模板，生成指定人的发票。

    * Java 中可选模板引擎还有 FreeMarker、Thymeleaf 等，若依框架选择了 Velocity。

* **Velocity 的三个应用场景**

    * 生成动态变化的 web 界面：按传入的数据模型渲染不同页面效果，这在前后端未分离的项目中用得多，现在比较少了。

    * 根据模板生成 Java 源码：这是若依代码生成器的核心能力之一。

    * 把动态页面转换为静态页，提高网页加载速度和性能，热点新闻页、商品详情页常用这种静态化处理。

* **四步固定流程**

    * > **定义**：一份完整的 Velocity 渲染代码由四个小步骤组成，其中第 1、3、4 步基本固定，**第 2 步「准备数据模型」才是重点**，不同模板只需换数据模型。

    * 第一步初始化模板引擎。

    * 第二步准备模板的数据模型，即上下文对象。

    * 第三步读取模板。

    * 第四步渲染模板，也就是合并输出。

* **初始化配置 VelocityInitializer**

    * > **定义**：若依在 util 包下提供的工具类，把模板引擎初始化步骤抽成一个静态方法 `initVelocity`，直接调用即可完成 `.vm` 的初始化。

    * `file.resource.loader.path` 设为 `classpath:`，加载类路径下方所有以 `.vm` 结尾的文件。

    * `file.resource.loader.class` 设为 `org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader`。

    * `input.encoding` 与 `output.encoding` 均设为 `UTF-8`，指定模板文件的字符集。

    * 最后 `Velocity.init(p)` 完成模板引擎的初始化。

* **模板读取与输出位置**

    * 初始化时已默认定位到 classpath 类路径，所以读取模板直接写 `vm/index.html.vm`，再补上 `UTF-8`。

    * `Velocity.getTemplate` 有多个重载，写代码时 IDEA 会弹出提示，要选**带模板名称 + 字符集编码**的那个。

    * `merge` 的第二个参数 IDEA 默认给的是 `System.out`，要输出成静态 HTML 页面必须自己 `new FileWriter`，把位置指到指定目录（如 `D:/workspace/index.html`）。

    * `new FileWriter` 会抛异常，直接给 `main` 方法加 `throws Exception` 抛出去。

    * > **易错点**：`FileWriter` 一定要做关流操作，不然生成的页面是**空白**的。

* **入门案例与测试代码位置**

    * 三步操作：先把固定页面里的「加油同学」替换成 `${message}` 占位符；再执行四步代码完成数据填充；最后运行并打开输出页面看动态数据。

    * 模板后缀统一用 `.vm`，即使内容是 HTML 页面也建 `index.html.vm`。

    * 模板放在 `src/main` 下会跟业务代码一起打包上线，不符合预期，所以另建 `test/java` 放测试类、`test/resources` 存代码模板，Maven 结构下这两处不会上线打包。

    * 测试类包名 `com.dk.test`，类内写 `main` 方法，第一步直接调 `VelocityInitializer.initVelocity()`。

    * 把 `context.put` 里的值改掉重新执行，输出页面随之变化，这就是模板与数据模型相分离的效果：模板无动态数据可随时改，改完由后端重新加载动态数据生成新页面。

* **从表信息到模板变量**

    ```text
    业务表 sys_biz_order（表名、表注释、作者，以及每一列的
    字段名、Java 类型、是否列表展示、是否父类属性、注释、Excel 日期格式）
            │
            │  代码生成器读取表信息 + 字段信息
            ▼
    上下文对象 Context
      packageName        基础包名
      moduleName         模块名
      businessName       业务名
      className          大写类名
      author / date      注释里的作者与生成时间
      table / columns    表信息与字段信息
            │
            │  Velocity 渲染 ${变量} 与 # 指令
            ▼
      domain.java.vm  ─────────►  Order.java（实体类）
      controller.java.vm ─────►  OrderController.java（控制器）
    ```

## Velocity 基础语法

* **变量定义指令 #set**

    * > **定义**：`#set` 用于在模板界面直接声明变量，形式是一对圆括号，左边是变量名 `$name`，右边是该变量的值。

    * 写法：`#set($name = "velocity")`。

    * 除了在 Java 代码中定义数据模型，还可以在模板里用 `#set` 声明。

* **两种取值写法**

    * > **定义**：取值有两种等价写法——直接 `$` 后跟变量名，以及 `$` 加花括号包裹变量名 `${name}`。

    * 两种写法单独取值结果相同，都能取到 `velocity`。

    * 简单语法可以快速引入变量值，但**涉及字符串拼接就不适用**。

    * 拼接场景举例：定义 `$column = "order"` 后，`$columnService` 会把 `$columnService` 当成普通文本原样输出，而 `${column}Service` 才输出 `orderService`。

    * > **结论**：**单独使用时推荐简单语法**（语法简单）；**涉及字符串拼接时推荐标准语法 `${}`**。

    * > **易错点**：IDEA 在写左花括号时会自动在右侧补一个右花括号，要记得删掉多余部分，否则渲染出来会多出一个 `}`。

* **模板注释**

    * `##` 开头的一行是 Velocity 的模板注释，页面上不会输出。

    * 注释可以只写 `## 定义变量`，也可用 `#* ... *#` 块注释形式。

    * IDEA 里可以用快捷键 `Alt + /` 生成注释。

* **对象变量的属性访问**

    * > **定义**：给对象变量赋值后，模板中用 `$对象名.属性名` 逐层取到具体属性。

    * 示例对象有两个属性 `id` 与 `regionName`，模板里写 `$region`、`$region.id`、`$region.regionName`。

    * 直接写 `$region` 时底层走 `toString` 方法，打印出对象的 id 与区域名称。

    * 对象变量的构造可用 Lombok：`@Data` 管 get/set，`@NoArgsConstructor` 管无参构造，`@AllArgsConstructor` 管全参构造。

    * > **提示**：模板里的**变量名**是 `put`/`setVariable` 时左侧的名称，跟右侧传进去的对象**没有任何关系**；这里写 `a` 模板里就写 `$a`，再去点实体中的属性。

    * > **提示**：提示符由通义灵码生成，建议**手写**一遍以巩固基础语法。

* **集合定义与 foreach 循环**

    * > **定义**：`#foreach($item in $list) ... #end` 是 Velocity 的循环指令，`$item` 是本次循环拿到的元素值，`$list` 是被遍历的集合。

    * 集合用 `#set` 定义，方括号里把元素用逗号隔开：`#set($forList = ["春", "夏", "秋", "冬"])`。

    * 写法与 Java 相同，先写 `in` 关键字，左侧是元素变量 `$item`，循环体里 `${item}<br/>` 输出。

    * 循环体以 `#end` 结束。

* **循环序号 foreach.count 与 foreach.index**

    * 给元素配序号时在 `$item` 前补中括号，序号值动态获取：`[${foreach.count}]${item}<br/>`。

    * > **易错点**：`${foreach.count}` 是**从 1 开始**的，输出 `[1]春[2]夏[3]秋[4]冬`。

    * 想显示索引就把 `count` 改为 `index`，`${foreach.index}` 是**从 0 开始**的。

    * 给人看的列表编号用 `count`，要跟数组下标、跟后端传过来的位置对齐就用 `index`。

    * 建议在模板里留一行中文注释写明「count 从一开始，index 从零开始」，下次回来改时不容易搞混。

* **遍历对象集合**

    * > **定义**：集合元素是对象时，循环体内继续用点号取属性，如 `${region.id} ${region.name}<br/>`。

    * 数据模型侧用 List 集合的静态方法 `of` 创建并放进上下文：`context.setVariable("regionList", regionList)`。

    * 多个 `<br/>` 会让每个元素各占一行，去掉第一个即可让 id 与名称同行显示。

* **JDK 版本不兼容坑（List.of）**

    * > **易错点**：`List.of()` 是 **JDK 1.9 之后**才添加的，而若依父工程 `pom.xml` 里 `<java.version>` 锁死为 `jdk 1.8`。工程结构里开发环境虽是 JDK 11，但那只是**编译环境**，运行时已锁定 1.8，所以编译期不报错、跑到运行期就报「找不到该符号」。

    * 解决方案：把 `pom.xml` 里的 `<java.version>` 从 `1.8` 改为 `11`，改完第一次执行需要重新编译。

    * 不想动 JDK 版本时，换成 `new ArrayList<>()` 之后再 `add` 这两个对象也完全可以。

* **条件判断指令 #if**

    * > **定义**：`#if / #elseif / #else / #end` 用于按条件决定是否输出特定内容，相当于 Java 的 `if...else if...else`，只是每个关键字前加井号，末尾加 `#end`。

    * 成绩示例：`#set($score = 80)` 后，大于等于 80 输出「优秀」，`#elseif` 判断大于等于 60 输出「及格」，`#else` 输出「不及格」。

    * 井号 `if` 后直接判断 `$score`；写完格式化让 if 语句左侧对齐。

    * > **提示**：`#elseif` 是一个**整体**，中间没有空格也没有多余的井号；IDEA 会补全，但第一次练建议手敲一遍。

    * 修改成绩变量值重新执行刷新，输出随之切换，说明判断确实生效。

* **对象的非空判断与逻辑运算符**

    * > **定义**：直接判断对象代表该对象不为空时才执行分支，不需要写「不等于空」。

    * `#if($user)` 输出欢迎语，`#if(!$user)` 在对象为空时才执行，逻辑取反只需在前方加感叹号。

    * 条件判断中 Velocity 也支持 `&&`、`||`、`!` 三种逻辑运算符，与 Java 一致。

## 代码模板走读

* **单表 / 主子表与树表的模板差异**

| | 单表 / 主子表 | 树表 |
| --- | --- | --- |
| 实体类继承 | `BaseEntity` | `TreeEntity` |
| 是否导入 `TableDataInfo` | 是 | 否 |
| 列表接口返回 | `TableDataInfo`（带分页） | `AjaxResult`（List 全量数据） |
| 树形结构谁生成 | — | 前端 |

* **变量来源对照**

    * 模板里的变量不是凭空来的：生成代码时 Velocity 工具类已经把业务表的基本信息和字段信息全部放到了上下文对象中。

    * 读模板时几乎每一行都能在这条链路上找到来源——表信息与字段信息进入上下文对象，再由 `${变量}` 与 `#` 指令渲染成实体类和控制器。

### 实体类模板走读

* **第一行包路径与导包**

    * 第一行取上下文里的 `$packageName`（当前项目基础包名，形如 `com.dk.manager`），后面拼固定的 `.domain`，就是实体类的包路径。

    * 中间导包按列的类型来做：`Long`、`String` 都在 `java.lang` 包下，而 `java.lang` 是默认无需导入的，所以这类导包被简化掉了。

    * 有其他特殊类型时才进行导入；另外还导入了 Apache commons-lang 的包（用于生成对象的 toString 方法）和项目自定义的 Excel 注解包。

    * 接着是一个 `#if` 判断：单表 CRUD 或主子表让类默认继承 `BaseEntity`，树表则继承 `TreeEntity`，导包这里也有对应判断。订单实体导入的是 `BaseEntity`。

* **类注释与类定义**

    * 生成实体类注释时的变量对应关系：`${className}` 对应类名，`${businessName}` 之类对应业务名（如订单管理），`${tableName}` 对应表名，`${author}` 是配置文件里指定的作者，还有代码生成时间也通过变量获取。

    * 注释之后还有一个判断：单表或主子表就设置 `baseEntity` 相关的变量，树表则设置 `TreeEntity` 变量，它正是下方需要继承的。

    * 再指定类名（大写的 `Order`），然后定义类的序列化版本号，固定值是 `1`。

* **字段属性遍历**

    * > **定义**：要获取当前业务表所有列的字段信息，需要做一个 `#foreach` 遍历，把订单表所有字段属性在这里面生成出来。

    * 遍历时第一个判断是**如果这不是父类的属性才生成**——每个实体类都继承 `BaseEntity`，父类已有创建人、创建时间、更新人、更新时间、备注说明这五个信息，子类没必要重复生成，命中这五个字段就跳过。

    * 非父类字段先生成属性注释，如「主键」「订单编号」。

    * 属性定义部分跟上 `private` 关键字，取该属性的 Java 类型（`Long`、`String`）和字段名称（`javaField`，如 `id`、`orderNumber`）。

    * 再往下还有一个判断：如果当前表有子表，需要在类里定义一个子表集合来表示主子表关系、存储子表数据；订单表是单表操作，这部分不包含。

* **列表展示字段的 Excel 注解与日期格式**

    * 第二个判断是**如果该字段是在列表展示的**才生成对应注解：订单编号在列表展示，要生成它的注解；机器编码不需要在页面列表展示，注解就不生成。

    * 日期类型要转换为指定格式的字符串（年月日），解析 Excel 时同样转换为年月日格式。

    * 模板片段：

      ```velocity
      #if($column.isList())
      @Excel(name = "${column.columnComment}", dateFormat = "yyyy-MM-dd", parseFormat = ["yyyy-MM-dd"])
      #end
      ```

* **get/set 与 toString**

    * 生成 get 和 set 方法也是一个 `#foreach` 循环过程，逐个属性生成 id、订单编号、设备编号等的 get/set，与订单实体类完全对得上。

    * 有子表数据时同样会生成 List 集合的 get/set 方法。

    * 最后生成 toString 方法，`return` 一个 `ToStringBuilder` 字符串拼接对象，通过 `#foreach` 取所有字段属性名，再通过它们的 get 方法取具体值：

      ```velocity
      @Override
      public String toString() {
          return new ToStringBuilder(this)
              .append("id", getId())
              .append("orderNumber", getOrderNumber())
              .append("machineCode", getMachineCode())
              .toString();
      }
      ```

### 控制器模板走读

* **导包与类定义**

    * 第一行同样是获取当前模板的包路径，下面导入项目的 Java 类库：`List` 集合、`AjaxResult`、`TableDataInfo`、权限控制、`@Log` 自动注入，以及 Rest 风格请求方式、`@PathVariable`、`@RequestBody`、`@RequestMapping`、`@RestController` 等。

    * 导包处有一个判断：单表操作或主子表操作需要返回分页对象 `TableDataInfo`；树表**不需要做导包**，因为树表查询的是所有数据、由前端生成树形结构，不包含后端分页功能。

    * 类注释的作者、时间与实体类模板完全一样。

    * 类定义形如：

      ```java
      @RestController
      @RequestMapping("${moduleName}/${businessName}")
      public class ${className}Controller extends BaseController
      ```

    * `@RestController` 是固定的；`@RequestMapping` 里 `${moduleName}` 对应模块名，后面接业务名，中间做拼接。

    * `${className}` 是大写的，对应大写的 `Order`；Service 依赖注入是大写类型加小写对象名，所以声明变量时用的是小写的 `className`。

* **方法注释与权限注解**

    * `${businessName}` 对应订单管理；权限注解为 `@PreAuthorize("@ss.hasPermi('${moduleName}:${businessName}:list')")`，其中 `list` 是固定的。

* **列表方法的返回值分支**

    * 单表或主子表的返回值是 `TableDataInfo`，即分页对象。

    * 树表的返回值是标准的 `AjaxResult`，查询 List 集合直接返回前端，由前端生成树表结构。

    * 查询列表之后的模板依次是导出方法、按 id 查询、新增、修改、删除，与订单 controller 中这几个方法一一对应。

    * 有了这套固定模板，将来导入商品、设备时只需替换变量名，就能快速生成控制器框架代码，保证代码一致性与可维护性。

## Lombok 集成改造

* **改造目标与依赖**

    * > **定义**：改造 domain 实体类模板，通过集成 Lombok 添加注解，由注解自动生成标准 Java 属性方法，从而**删除冗余的 get/set 与 toString**。

    * 若依默认并没有提供 Lombok 依赖，需确保项目已添加依赖坐标——在 common 模块的 `pom.xml` 里可以确认，且项目自定义的 DTO 和 VO 类已在使用相关注解。

* **类上的四个注解**

    * 位置在模板中 `public class` 的上方：

      ```java
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      @Builder
      public class ${className} extends ${entity}
      {
      ```

* **四个注解的分工**

| 注解 | 作用 |
| --- | --- |
| `@Data` | 自动生成 getter、setter 以及 toString 方法 |
| `@NoArgsConstructor` | 生成无参构造方法，方便实体类操作 |
| `@AllArgsConstructor` | 生成全参构造方法 |
| `@Builder` | 简化对象的创建，快速完成对象属性的复制操作 |

* **导包必须手动检查**

    * 写完注解后要逐个对一遍导包：`builder` 没问题、`data` 没问题、无参构造没问题，缺的是**全参构造**这条。

    * > **易错点**：当前代码模板是 `.vm` 而不是 `.java`，IDEA 在导包时很可能出现遗漏，四个注解要对应四条导包操作。

    * 为让导包结构更清晰，可把 lombok 的导包先剪切走、删掉多余空格，在最下方加一行注释「导入 lombok 的相关注解」，再放回原位置。

* **删除被注解接管的代码**

    * 删掉生成 get/set 方法那一整段——它从第 75 行附近注释「为每个属性生成 get and set 方法」的 `#foreach` 开始，到 `#foreach` 结束为止。

    * 子表集合的 get/set 方法也能用注解生成，同样删掉。

    * 最下方重写的 toString 方法一并删除。

    * Apache commons 的导包是为 toString 服务的，toString 删掉后这两条导包也要一起删。

    * > **易错点**：删除时**最后一个花括号是当前类的结束符，不能误删**；只删从 toString 开始到类结束花括号之前的部分，保留最后一个花括号。

    * > **易错点**：Lombok 这几个注解**大小写敏感**，`AllArgsConstructor` 里两个 `a` 都得大写；粘贴生成的实体类报「找不到该符号」时先回头看模板，把小写改成大写再 `Ctrl+F9` 热部署。

* **验证方式**

    * `Ctrl+F9` 热更新成功后回浏览器，对订单表点预览，看类上方是否已增加 Lombok 注解、导包是否完整、冗余 get/set 与 toString 是否消失。

    * 再把生成的实体类拷到对应业务模块的 domain 包，全选删除原文件后 `Ctrl+V` 替换，`Ctrl+F9` 热部署。

    * 访问订单管理界面，查询功能正常即证明实体类模板改造成功；此后每次生成代码都会自动支持 Lombok。

## Swagger 集成改造

* **改造目标**

    * > **定义**：Swagger 是 API 文档生成工具，能自动生成在线接口文档；目标是改造 controller 模板，为类和类中方法添加注解，自动产生 API 文档。

    * 若依项目已集成 Swagger 相关坐标，这一步直接跳过，只需改造 control 模板。

* **类上加 @Api**

    * 在类的位置（往上拉到第 33 号附近）添加 `@Api`，它有一个 `tags` 属性，用于 Swagger 文档中的**分类显示**。

    * 取值不用写死的「订单管理」，而是用注释里的类名变量拼一个 `Controller` 后缀，保证见名知意：

      ```java
      /**
       * ${moduleName}Controller
       *
       * @author ${author}
       * @date ${date}
       */
      @Api(tags = "订单管理Controller")
      @RestController
      @RequestMapping("${moduleName}/${businessName}")
      public class ${className}Controller extends BaseController
      ```

    * 换一张表它就自动变成「商品管理Controller」。

* **每个方法上加 @ApiOperation**

    * `@ApiOperation` 用它的 `value` 属性描述方法功能，同样取方法的注释值：

      ```java
      @ApiOperation(value = "查询订单管理列表")
      @GetMapping("/list")
      public TableDataInfo list(Order order)
      ```

    * 从第一个方法复制到其余方法并各自改注释：导出方法改为「导出订单管理列表」，查询详情改为「获取订单详细信息」，新增、修改、删除按同法处理。

* **AjaxResult 改 R 才能展示返回结构**

    * > **易错点**：`AjaxResult` 默认继承了 `HashMap`，**Swagger 对它不兼容**，展开不了里面的数据，因此接口文档看不到返回结果的详细信息。

    * 解决办法是直接改为 `R` 类型，这是一种通用返回类型，可以包含各种不同的数据结构。

    * 改造范围是模板里需要返回结果的那几处——查询列表（树表分支）、按 id 查询、新增、修改、删除等方法，逐个从 `AjaxResult` 换成 `R`，并把 `AjaxResult.success(...)` 的构造方式改成 `R.ok(...)` 这一类写法。

    * Swagger 解析返回类型时需要拿到具体的泛型结构才能展开字段。

* **验证方式**

    * `Ctrl+F9` 热部署后点预览，检查 control 部分：类上方有 `@Api` 且 `tags` 为「订单管理Controller」作为文档分类名，每个方法上方有 `@ApiOperation` 描述功能。

    * 把最新代码拷贝到项目对应 controller，全选删掉原代码后替换，再 `Ctrl+F9` 热部署。

    * 先访问订单管理页面确认查询与重置正常，再打开「系统接口」页面看订单管理分类是否正常显示、该模块是否列出新增、修改、导出、查询、删除以及获取若依标准代码这几个接口。

    * 看到在线接口文档界面即表示 Swagger 集成完成，此后生成代码自动带 Swagger 注解，可直接访问该界面做后端功能测试和前后端联调。