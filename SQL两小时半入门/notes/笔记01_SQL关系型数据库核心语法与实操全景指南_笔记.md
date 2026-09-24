# SQL关系型数据库核心语法与实操全景指南

## 数据库体系架构与管理系统

### 持久化介质与管理系统演进

* **数据持久化介质（Persistence Media）**

    * > **定义**：用于承载与保存业务数据并决定数据存取生命周期与读写性能的物理载体。

    * 记忆型介质：容量极小，易受干扰遗忘，交互延迟最低。

    * 平面物理介质：容量可随纸张物理堆叠扩展，缺乏动态索引结构，数据规模膨胀时线性检索复杂度过高。

    * 结构化电子存储：基于磁盘介质与内存缓存，依托多维索引体系，在保障海量数据吞吐的同时实现对数级甚至常数级的检索响应。

* **数据库管理系统（Database Management System，DBMS）**

    * > **定义**：操纵和管理底层物理数据库的系统级基础软件，负责数据的结构化组织、物理存储、高效检索、并发调度与完整性保护。

    * 三层解耦交互拓扑：应用程序不直接扫描物理磁盘文件，而是向 DBMS 发送结构化查询指令；DBMS 解析指令后基于索引结构定位目标并装配返回业务层。

    * 调度机制：

        ```text
        +-------------------+           +-------------------------------------+           +------------------------+
        | 应用程序 / Web 平台 | --------> | 数据库管理系统 (DBMS, 如 MySQL)        | --------> | 物理存储底座 (Database)  |
        | (发起数据读写请求) | <-------- | (基于索引结构秒级检索目标数据并返回) | <-------- | (持久化存放海量数据记录) |
        +-------------------+           +-------------------------------------+           +------------------------+
        ```

    * 指令调度边界：涵盖业务平台的数据检索（查询）、新增（上传）、更新（修改）与物理删除（下架）。

---

## 关系模型与键约束机制

### 关系型与非关系型架构对比

* **关系型数据库管理系统（Relational Database Management System，RDBMS）**

    * > **定义**：基于关系模型建立的数据库系统，将业务数据组织在由行与列构成的二维表格中，并通过公共属性及键约束建立表间关联。

    * 通信规范：统一遵循标准结构化查询语言（SQL）进行指令交互。

    * 典型系统：MySQL、Oracle、PostgreSQL、SQL Server。

* **非关系型数据库管理系统（Non-relational Database Management System，NRDBMS / NoSQL）**

    * > **定义**：泛指所有未采用传统二维表格与外键关联模型组织业务数据的数据库系统统称。

    * 存储形态：涵盖文档型存储（JSON/BSON 格式，如 MongoDB）、图形存储（Graph 拓扑网络，存储节点与边）、键值对存储（Key-Value 内存映射，如 Redis）以及分布式全文搜索引擎（如 Elasticsearch）。

    * 通信规范：缺乏全球统一步调的交互语言，各软件均采用自研专用 API 或专属操作命令集。

| 架构分类 | 组织形态与存储方式 | 通信控制规范 | 核心代表产品 |
| :--- | :--- | :--- | :--- |
| **关系型数据库（RDBMS）** | 二维表格（Table）、主外键关联合约 | 统一遵循标准 SQL 语言交互 | MySQL、Oracle、PostgreSQL、SQL Server |
| **非关系型数据库（NRDBMS）** | 键值（KV）、文档（JSON）、图形拓扑等多态结构 | 各软件自研专用 API 或专用命令集 | MongoDB、Redis、DynamoDB、Elasticsearch |

### 二维表结构与主键约束

* **二维数据表结构（Relational Table Structure）**

    * > **定义**：关系型数据库中用于表达独立实体集合的二维几何网格容器。

    * 行（Row）：横向的单一记录，代表系统中一个具体的业务实体元组。

    * 列（Column / Attribute）：纵向的属性维度，代表业务实体具备的某项特定特征字段。

* **主键（Primary Key，PK）**

    * > **定义**：数据表中用于唯一标识每一行数据记录的一个或一组属性列。

    * 约束条件：主键字段的取值在全表范围内必须绝对唯一，严禁出现重复值，且绝对不允许为 NULL。

    * 字段内联声明语法：在创建数据表字段时直接追加 `PRIMARY KEY` 声明。

        ```sql
        CREATE TABLE `student` (
            `student_id` INT PRIMARY KEY,
            `name` VARCHAR(20),
            `major` VARCHAR(20)
        );
        ```

    * 独立声明语法：在所有属性列定义完成后，通过小括号后置独立声明。

        ```sql
        CREATE TABLE `student` (
            `student_id` INT,
            `name` VARCHAR(20),
            `major` VARCHAR(20),
            PRIMARY KEY (`student_id`)
        );
        ```

### 外键引用与层级关联

* **外键（Foreign Key，FK）**

    * > **定义**：数据表中的特定属性列，其取值必须严格引用并参照另一张表（或自身表）的主键值，用于在离散表之间建立逻辑纽带并保障参照完整性。

    * 引用约束规则：外键有且仅能引用具备绝对唯一性保障的主键列；若引用允许重复的普通列，系统将无法判定具体对应哪一条实体记录。

    * 跨表关联示例拓扑：

        ```text
          员工表 (employee)                                    部门表 (branch)
        +--------+------+----------+--------+                +-----------+-------------+------------+
        | emp_id | name | ...      | branch_id (FK) |  +-------> | branch_id | branch_name | manager_id |
        | (PK)   |      |          |        |        | |  (PK)   |             | (FK)       |
        +--------+------+----------+--------+        | |     +-----------+-------------+------------+
        | 206    | 小黄 | ...      | 1      | --------+ |     | 1         | 研发部      | 206        | -----+
        | 207    | 小绿 | ...      | 2      | --------|-+     | 2         | 行政部      | 207        | ---+ |
        +--------+------+----------+--------+         |       +-----------+-------------+------------+    | |
            ^                                         |                                                   | |
            +-----------------------------------------|---------------------------------------------------+ |
            +-----------------------------------------+-----------------------------------------------------+
        ```

* **自引用外键（Self-referencing Foreign Key）**

    * > **定义**：数据表内部的外键属性直接引用本表自身的主键列，用于在单表内部表达树状管理层级与汇报链条。

    * 运行机理：员工表中的主管编号（`sup_id`）定义为外键并指向自身的工号（`emp_id`）；处于最高层级的首脑由于无上级主管，其 `sup_id` 取值为 NULL。

    * 结构示意：

        ```text
        +--------+------+---------------+
        | emp_id | name | sup_id (FK)   |
        | (PK)   |      | (引用本表主键) |
        +--------+------+---------------+
        | 206    | 小黄 | NULL (无主管) | <-------+
        | 207    | 小绿 | 206           | --------+ (上级是小黄)
        | 208    | 小黑 | 206           | --------+ (上级是小黄)
        | 209    | 小蓝 | 207           | --------+ (上级是小绿，指向 207)
        +--------+------+---------------+
        ```

### 复合主键与主外键重叠

* **复合主键（Composite Primary Key）**

    * > **定义**：由两个或多个属性列联合绑定、共同构成的数据表唯一主键。

    * 判定规则：参与复合主键的单一列允许各自出现重复值，但所有联合构成的多元组组合在全表中必须绝对唯一。

    * 声明方式：必须在建表语句末尾使用独立声明语法，小括号内列出全部联合字段。

        ```sql
        PRIMARY KEY (`emp_id`, `client_id`)
        ```

* **主外键重叠设计（Overlapping Primary and Foreign Key）**

    * > **定义**：数据表中的特定字段同时兼具复合主键构成要素与指向父表主键的外键双重角色。

    * 业务建模形态：在多对多业务关联表（如 `works_with` 销售对接表）中，`emp_id` 既是本表复合主键的一半，又是引用员工表的主键外键；`client_id` 既是复合主键的另一半，又是引用客户表的主键外键。

        ```text
        +--------------+----------------+-------------+
        | emp_id       | client_id      | total_sales |
        | (联合主键 PK / | (联合主键 PK / |             |
        |  员工表 FK)   |  客户表 FK)    |             |
        +--------------+----------------+-------------+
        | 206          | 400            | 55000       |
        | 206          | 401            | 267000      |
        | 207          | 400            | 12000       |
        +--------------+----------------+-------------+
        ```

---

## 数据库定义与表模式管理

### 语法规范与客户端工具

* **SQL 语法书写规约（SQL Syntax Conventions）**

    * > **定义**：人或应用程序与关系型数据库管理系统进行交互时必须遵守的标准词法与语法规则。

    * 关键字大小写规则：语法引擎本身不区分大小写，但工程规范统一使用纯大写英文字母书写保留关键字（如 `CREATE`、`SELECT`、`WHERE`），形成视觉切割。

    * 标识符反引号转义：自定义数据库名、数据表名与属性列名使用纯小写，并强烈建议统一使用一对反引号（\`）包裹，规避与内置保留字冲突（如 ``CREATE DATABASE `database`;``）。

    * 指令闭合符号：每一条独立的 SQL 语句末尾必须以英文半角分号（`;`）闭合。

* **客户端工作区与执行控制（Workbench Execution Controls）**

    * > **定义**：用于连接数据库服务端并分发 SQL 执行指令的可视化图形交互环境。

    * 核心三件套：MySQL Server 8.0（底层持久化引擎）、MySQL Workbench 8.0（图形 GUI 客户端）、MySQL Shell 8.0（高级命令行终端）。

    * 纯黄色闪电：执行编辑器中当前被鼠标选中的代码块；若未选中任何内容，则自顶向下执行编辑区全量脚本。

    * 带光标闪电：仅执行当前文本输入光标所在的那一条单独 SQL 指令。

### 库级生命周期管理

* **数据库创建与检索（CREATE and SHOW DATABASE）**

    * > **定义**：在宿主数据库服务实例中开辟或检索逻辑命名空间的操作。

    * 创建指令：`CREATE DATABASE 库名;`

        ```sql
        CREATE DATABASE `sql_tutorial`;
        ```

    * 检索指令：`SHOW DATABASES;`，输出当前实例挂载的所有数据库列表。

* **工作库切换与物理销毁（USE and DROP DATABASE）**

    * > **定义**：选定当前指令作用上下文或彻底移除指定数据库的操作。

    * 切换工作库：`USE 库名;`，告知引擎后续所有 DDL 与 DML 操作均在目标库内生效。

        ```sql
        USE `sql_tutorial`;
        ```

    * 物理销毁指令：`DROP DATABASE 库名;`，将逻辑库及其名下的所有数据表、视图和索引完全丢弃。

    * > ⚠️ **易错点**：执行 `DROP DATABASE` 属于高危不可逆操作，被删除库内部所有的结构定义与全部业务数据都将被彻底清除。

### 核心物理数据类型

* **数值与文本数据类型（Numeric and String Data Types）**

    * > **定义**：用于规定属性列在磁盘底层所分配的二进制编码结构、存储容量及运算规则的物理类型。

    * 整数型 `INT`：存放标准整数字面量，支持正整数、0 与负整数。

    * 高精度定点小数型 `DECIMAL(M, D)`：$M$（Precision，精度）代表允许包含的最大有效数字总位数；$D$（Scale，标度）代表小数点右侧必须保留的位数。整数位最多为 $M - D$ 位。例如 `DECIMAL(3, 2)` 能够合法存储 `2.33`。

    * 变长文本字符串型 `VARCHAR(M)`：存放可变长度纯文本字符串，$M$ 为该字段允许容纳的字符上限。

* **二进制与时态数据类型（Binary and Temporal Data Types）**

    * > **定义**：用于承载非结构化多媒体媒介或连续时间轨迹的专用数据类型。

    * 二进制大对象 `BLOB`：Binary Large Object，存放无结构的原始二进制数据流（如图片二进制元数据、音频流或文档流）。

    * 纯日期型 `DATE`：存放离散日历日期，固定遵循 `YYYY-MM-DD` 格式。

    * 完整时间戳 `TIMESTAMP`：精准记录包含时、分、秒的时间轨迹（格式形如 `YYYY-MM-DD HH:MM:SS`），主要用于数据行创建与修改审计。

| 数据类型标识符 | 核心物理语义 | 参数要求与参数意义 | 典型数据样例 | 核心适用业务场景 |
| :--- | :--- | :--- | :--- | :--- |
| `INT` | 整数类型 | 无参数 | `1024`, `-50`, `0` | 主键标识符、序列号、计数器 |
| `DECIMAL(M, D)` | 高精度定点小数 | `M`：有效数字总位数；`D`：小数位数 | `DECIMAL(3, 2)` 存 `2.33` | 财务结算、商品金额、学业绩点 |
| `VARCHAR(M)` | 变长文本字符串 | `M`：最大允许字符长度上限 | `VARCHAR(20)` 存 `'小白'` | 用户姓名、登录账号、地址摘要 |
| `BLOB` | 二进制大对象 | 无必需长度参数 | 二进制原始文件流 | 物理图片存储、多媒体二进制流 |
| `DATE` | 离散日历日期 | 无参数（固定 `YYYY-MM-DD`） | `'2021-08-08'` | 出生日期、合同签署日、纪念日 |
| `TIMESTAMP` | 时分秒完整时间戳 | 无参数（含时分秒精度） | `'2021-08-08 14:30:00'` | 数据创建审计、状态流转时间追踪 |

### 表模式创建与动态变更

* **表模式定义与查验（CREATE TABLE and DESCRIBE）**

    * > **定义**：在选定的逻辑数据库内部开辟实体数据表结构并检视其元数据规范。

    * 表创建语法：

        ```sql
        CREATE TABLE `student` (
            `student_id` INT PRIMARY KEY,
            `name` VARCHAR(20),
            `major` VARCHAR(20)
        );
        ```

    * 语法边界：字段定义之间以半角逗号分隔，最后一个字段末尾绝对不能追加逗号。

    * 结构检视指令：`DESCRIBE 表名;`（或缩写为 `DESC 表名;`），输出包含 `Field`、`Type`、`Null`、`Key`、`Default`、`Extra` 维度的元数据信息。

* **表结构动态维护与销毁（ALTER TABLE and DROP TABLE）**

    * > **定义**：对已有数据表的物理列进行热追加、删除或整表物理清除的操作。

    * 追加新字段：`ALTER TABLE 表名 ADD 列名 数据类型;`

        ```sql
        ALTER TABLE `student` ADD `GPA` DECIMAL(3, 2);
        ```

    * 移除已有字段：`ALTER TABLE 表名 DROP COLUMN 列名;`

        ```sql
        ALTER TABLE `student` DROP COLUMN `GPA`;
        ```

    * 物理销毁数据表：`DROP TABLE 表名;`，执行后该表的结构定义与物理数据被彻底抹除。

---

## 数据表完整性约束与操作控制

### 基础数据写入机制

* **全字段值对齐写入（INSERT INTO ... VALUES）**

    * > **定义**：在不指定字段列表的情况下，向数据表录入全新记录的基础指令。

    * 映射规则：`VALUES` 括号内提供的值列表，必须严格按照数据表创建时定义的物理列顺序逐一对应。

        ```sql
        INSERT INTO `student` VALUES (1, '小白', '歷史');
        ```

    * 格式规约：数值直接书写；文本必须使用成对英文引号（推荐统一单引号 `'...'`）包裹；值与值用逗号隔开；整句用分号结尾。

* **显式列名插入与空值占位（Explicit Column Insertion and NULL）**

    * > **定义**：通过显式声明待插入字段清单自主控制传参顺序，或借助占位符表达缺失信息。

    * 属性自主映射：表名后声明属性列括号，`VALUES` 内的数据只需与声明列表严格对齐。

        ```sql
        INSERT INTO `student` (`name`, `major`, `student_id`) VALUES ('小藍', '英語', 4);
        ```

    * 空值占位符 `NULL`：表示“无数据”或“数值缺失”，既不是空字符串 `''`，也不是数值 0。若插入语句中未提及某个允许为空的字段，系统自动将其默认填充为 `NULL`。

### 列级完整性约束配置

* **实体完整性约束（NOT NULL and UNIQUE）**

    * > **定义**：附加在属性列定义后方，强制检验字段数据有效性与全局唯一性的约束规则。

    * 非空约束 `NOT NULL`：强制该字段必须传入有效非空值，拦截任何形式的 `NULL` 写入。

    * 唯一约束 `UNIQUE`：强制该字段在全表范围内数值互不重复；若尝试插入已存在的值，数据库直接抛错拦截。

        ```sql
        CREATE TABLE `student` (
            `student_id` INT,
            `name` VARCHAR(20) NOT NULL,
            `major` VARCHAR(20) UNIQUE,
            PRIMARY KEY (`student_id`)
        );
        ```

* **缺省填充与自增序列（DEFAULT and AUTO_INCREMENT）**

    * > **定义**：用于在数据写入时提供自动值填充与序号单调递增的列级扩展属性。

    * 预设默认值 `DEFAULT`：当插入操作未显式提供该字段数据时，系统自动赋予预设的常量兜底，避免字段退化为 `NULL`。

        ```sql
        `major` VARCHAR(20) DEFAULT '歷史'
        ```

    * 主键序列自增 `AUTO_INCREMENT`：由数据库引擎在上一条记录的基础上自动累加 1 并赋予新行主键，彻底免除人工或应用程序显式传递主键 ID。

        ```sql
        `student_id` INT AUTO_INCREMENT
        ```

### 针对性数据更新与安全模式

* **安全更新模式（Safe Updates Mode）**

    * > **定义**：MySQL 服务端用于拦截未携带主键或唯一索引作为过滤条件的批量修改与删除操作的保护屏障。

    * 配置变更：`SET sql_safe_updates = 0;`，将该变量设为 0 关闭限制，以允许基于非主键条件的更新与删除练习。

* **针对性数据更新（UPDATE ... SET ... WHERE）**

    * > **定义**：修改数据表中已存在记录特定属性取值的数据操纵指令。

    * 单条件与多条件更新：通过 `WHERE` 子句限定变更范围，支持结合 `OR` 运算符扩增影响区间。

        ```sql
        UPDATE `student` SET `major` = '生化' WHERE `major` = '生物' OR `major` = '化學';
        ```

    * 多字段同时更新：在 `SET` 关键字后列出多个赋值表达式，彼此用逗号分隔。

        ```sql
        UPDATE `student` SET `name` = '小灰', `major` = '物理' WHERE `student_id` = 1;
        ```

    * > ⚠️ **易错点**：执行 `UPDATE` 或 `DELETE` 指令时若遗漏 `WHERE` 条件，数据库将无差别修改或清空全表所有记录，引发不可逆的全表数据污染或丢失。

### 物理数据删除与范围控制

* **精确条件物理删除（DELETE FROM ... WHERE）**

    * > **定义**：根据指定条件从数据表中永久抹除实体行记录的操作。

    * 组合条件精准删除：使用逻辑运算符 `AND` 实现多维度交集过滤。

        ```sql
        DELETE FROM `student` WHERE `name` = '小灰' AND `major` = '物理';
        ```

    * 数值比较范围删除：借助比较运算符（`<`、`>`、`<=`、`>=`、`<>`）批量清理满足特定指标区间的记录。

        ```sql
        DELETE FROM `student` WHERE `score` < 60;
        ```

* **全表清空风险与无条件删除（Table Truncation Risk）**

    * > **定义**：执行不带条件判定的删除指令使整张表退化为空表的极端状态。

    * 语法特征：`DELETE FROM 表名;`。该指令会逐行删除表内所有数据元组，保留表结构本身。

---

## 单表条件检索与结果集控制

### 字段投影与去重检索

* **属性投影查询（Column Projection）**

    * > **定义**：在数据检索时显式挑选所需输出的特定属性列，剔除无关字段的机制。

    * 全量投影：`SELECT * FROM 表名;`，提取数据表的全部字段。

    * 定向投影：在 `SELECT` 后罗列目标字段名，多个列之间用逗号隔开，能有效降低网络 I/O 开销与客户端内存消耗。

        ```sql
        SELECT `name`, `major` FROM `student`;
        ```

* **投影去重机制（DISTINCT）**

    * > **定义**：在查询结果集输出前，将指定投影字段组合完全相同的重复数据行予以折叠过滤的修饰符。

    * 作用机理：`DISTINCT` 作用于紧随其后的所有投影字段，仅输出唯一的属性取值组合。

        ```sql
        SELECT DISTINCT `sex` FROM `employee`;
        SELECT DISTINCT `branch_id` FROM `employee`;
        ```

### 结果集排序与数量切片

* **结果集排序（ORDER BY）**

    * > **定义**：按照一个或多个指定字段的取值次序，对查询检索出的结果行执行重新排列的子句。

    * 方向控制：默认缺省为升序排列（`ASC`）；若需从高到低降序排列，必须显式附加 `DESC` 关键字。

    * 多级排序：列出多个排序列，优先按首个字段排序；当首字段取值并列相同时，再依据次级字段进行升序或降序排布。

        ```sql
        SELECT * FROM `student` ORDER BY `score` DESC, `student_id` ASC;
        ```

    * 文本字符排序规则：依据字母序升序排列（如字母 `'F'` 在 `'M'` 之前）。

* **数量切片截取（LIMIT）**

    * > **定义**：强行截断查询结果集，仅提取前指定条数记录的物理截断子句。

    * 极值榜单检索范式：将排序子句与切片子句级联使用，实现提取特定区间 Top-N 数据。

        ```sql
        -- 提取薪资最高的前三名员工
        SELECT * FROM `employee` ORDER BY `salary` DESC LIMIT 3;
        ```

### 逻辑比较与集合匹配

* **布尔逻辑与非等值比对（AND, OR, Not Equal）**

    * > **定义**：在 `WHERE` 子句中通过组合布尔关系与非等值判定实现精准结果行过滤。

    * 非等值比较运算符：SQL 中标准的不等于操作符书写为由小于号和大于号组成的尖括号对 `<>`。

        ```sql
        SELECT * FROM `student` WHERE `score` <> 70;
        ```

    * 逻辑复合连接：`AND` 要求两端条件同时成立，`OR` 仅要求两端条件至少成立其一。

* **集合包含运算（IN）**

    * > **定义**：用于检验指定字段的取值是否落在由一组离散常量构成的候选集合中的成员运算符。

    * 语法优势：替代冗长重复的多重 `OR` 连接表达式，精简 SQL 结构。

        ```sql
        SELECT * FROM `student` WHERE `major` IN ('歷史', '英語', '生物');
        ```

---

## 聚合运算与模式匹配

### 聚合计算与空值处理

* **行数统计与空值过滤（COUNT）**

    * > **定义**：对符合过滤条件的数据记录执行行数累加汇总并返回单个标量计数值的聚合函数。

    * 全行计数：`COUNT(*)` 遍历整张表，统计所有物理存在的记录总行数。

    * 指定列有效计数：`COUNT(列名)` 仅统计该列中取值不为 NULL 的有效非空行数。

        ```sql
        SELECT COUNT(*) FROM `employee`;
        SELECT COUNT(`sup_id`) FROM `employee`;
        ```

    * > ⚠️ **易错点**：`COUNT(*)` 统计整行物理记录总条数（包含取值为 NULL 的行），而 `COUNT(列名)` 仅统计指定列中取值非 NULL 的有效记录条数。

* **数值极值与统计均值（AVG, SUM, MAX, MIN）**

    * > **定义**：针对数值型属性列执行算术平均、累加求和以及极值挖掘的内置聚合函数。

    * 算术均值 `AVG(列名)`：累加数值总和后除以行数，返回平均值。

    * 累加求和 `SUM(列名)`：计算全量记录该列数值的累计总额。

    * 极大值与极小值：`MAX(列名)` 与 `MIN(列名)` 分别输出指定范围内的最大与最小标量值。

        ```sql
        SELECT AVG(`salary`), SUM(`salary`), MAX(`salary`), MIN(`salary`) FROM `employee`;
        ```

### 模糊匹配与通配符规则

* **百分比通配符（Percentage Wildcard, %）**

    * > **定义**：搭配 `LIKE` 操作符使用，在模式串中代表任意数量（零个、一个或多个）的任意字符。

    * 前缀匹配模式：`'254%'` 匹配所有以 `254` 起始的字符串。

    * 后缀匹配模式：`'%335'` 匹配所有以 `335` 结尾的字符串。

    * 任意包含模式：`'%354%'` 匹配在任意内部位置嵌入了 `354` 的字符串。

    * 汉字前缀提取：`'艾%'` 命中所有以汉字“艾”开头的文本（如“艾瑞克”）。

* **下划线通配符（Underscore Wildcard, _）**

    * > **定义**：搭配 `LIKE` 操作符使用，在模式串中具有严格的定长占位特性，每一个下划线仅且必须代表一个字符。

    * 固定格式时态模式匹配：针对标准日期 `YYYY-MM-DD` 结构，使用连续下划线精准占位。

        ```sql
        -- 匹配所有在 12 月份出生的员工（前 5 个下划线占位 4 位年份与 1 个连接符）
        SELECT * FROM `employee` WHERE `birth_date` LIKE '_____12%';
        
        -- 匹配所有在 9 月份出生的员工（月份位按两位补齐 09）
        SELECT * FROM `employee` WHERE `birth_date` LIKE '_____09%';
        ```

---

## 多表集合操作与关联查询

### 多查询结果集合并

* **垂直联合查询（UNION）**

    * > **定义**：将两个或多个独立 `SELECT` 查询的输出结果集垂直拼接为一个统一结果集的集合操作符。

    * 硬性约束条件：

        * 各子查询投影所选取的属性列数必须严格一致。

        * 各对应物理位置的列，其底层数据类型必须相互兼容。

    * 语法调用与链式合并：

        ```sql
        SELECT `name` FROM `employee`
        UNION
        SELECT `client_name` FROM `client`
        UNION
        SELECT `branch_name` FROM `branch`;
        ```

* **联合结果集表头与别名（UNION Field Aliasing）**

    * > **定义**：为多表联合输出规范化重命名表头以消除多源字段语义歧义的技术。

    * 默认命名机制：合并后结果集的表头名称，默认完全继承第一个 `SELECT` 子句所使用的字段名。

    * 别名重命名：使用 `AS` 关键字在首个查询中为输出字段指定中性别名。

        ```sql
        SELECT `emp_id` AS `total_id`, `name` AS `total_name` FROM `employee`
        UNION
        SELECT `client_id`, `client_name` FROM `client`;
        ```

### 表连接匹配与数据保留

* **内连接（INNER JOIN）**

    * > **定义**：寻找两表之间满足 `ON` 关联匹配条件的交集记录，剔除任意一侧无法配对的孤立行。

    * 字段前缀消歧：当关联两表中包含同名属性时，必须使用 `表名.字段名` 格式予以界定。

        ```sql
        SELECT `employee`.`emp_id`, `employee`.`name`, `branch`.`branch_name`
        FROM `employee`
        JOIN `branch` ON `employee`.`emp_id` = `branch`.`manager_id`;
        ```

    * 匹配拓扑：

        ```text
        [ employee 表记录 (emp_id) ]  <─── ON 关联条件 ───>  [ branch 表记录 (manager_id) ]
             206 (小黄)                    相等                   206 (研發部)  ──> 输出匹配行
             207 (小绿)                    相等                   207 (行政部)  ──> 输出匹配行
             208 (小黑)                    相等                   208 (資訊部)  ──> 输出匹配行
             209 (小白)                   无对应                   未匹配        ──> 丢弃
             210 (小蓝)                   无对应                   未匹配        ──> 丢弃
               无                         未匹配                   NULL (偷懒部) ──> 丢弃
        ```

* **外连接（LEFT JOIN and RIGHT JOIN）**

    * > **定义**：在关联条件无法匹配时，强制保留某一侧数据表的全部记录，并在缺失侧自动填充 NULL 的连接方式。

    * 左外连接（`LEFT JOIN`）：以左表为基准全量输出，右表无法匹配项补填 NULL。

        ```sql
        SELECT `employee`.`emp_id`, `employee`.`name`, `branch`.`branch_name`
        FROM `employee`
        LEFT JOIN `branch` ON `employee`.`emp_id` = `branch`.`manager_id`;
        ```

    * 右外连接（`RIGHT JOIN`）：以右表为基准全量输出，左表无法匹配项补填 NULL。

        ```sql
        SELECT `employee`.`emp_id`, `employee`.`name`, `branch`.`branch_name`
        FROM `employee`
        RIGHT JOIN `branch` ON `employee`.`emp_id` = `branch`.`manager_id`;
        ```

| 连接类型 | 核心驱动方向 | 未匹配记录的处理策略 | 适用业务场景 |
| :--- | :--- | :--- | :--- |
| `INNER JOIN` | 双向对称 | 只要任一侧未满足 `ON` 条件即被完全剔除 | 仅提取关系完全确立的核心业务数据 |
| `LEFT JOIN` | 左表主导 | 左表全量保留；右表无匹配项补填 `NULL` | 保留全量实体主体并附加可选关联属性 |
| `RIGHT JOIN` | 右表主导 | 右表全量保留；左表无匹配项补填 `NULL` | 以从属维度为全量基准反查主体信息 |

### 嵌套子查询求解

* **单值标量子查询（Scalar Subquery）**

    * > **定义**：嵌套在外部 SQL 语句内部，且执行后回传单一标量值（1 行 1 列）的内层检索语句。

    * 运算符要求：外部查询可以直接使用标量比较操作符（如 `=`、`>`、`<`）。

        ```sql
        SELECT `name` FROM `employee`
        WHERE `emp_id` = (
            SELECT `manager_id` FROM `branch`
            WHERE `branch_name` = '研發'
        );
        ```

* **多值集合子查询（Multiple-row Subquery）**

    * > **定义**：执行后回传多行记录组成的一维数值集合的内层嵌套检索语句。

    * 匹配规则：外部主查询必须使用集合成员运算符 `IN` 判定属性是否命中内层集合。

        ```sql
        SELECT `name` FROM `employee`
        WHERE `emp_id` IN (
            SELECT `emp_id` FROM `works_with`
            WHERE `total_sales` > 50000
        );
        ```

    * > ⚠️ **易错点**：当内部子查询返回多行多值集合时，外部查询绝对不可使用标量等号 `=`，否则会抛出运行时异常，必须使用集合运算符 `IN` 进行匹配。

---

## 企业级多表建模与外键级联控制

### 循环依赖建表与死锁写入解耦

* **循环依赖建表解耦机制（Circular Dependency Decoupling）**

    * > **定义**：通过分阶段构建无外键基础表与后期表结构动态变更，破除数据表之间双向相互引用建表阻断的工程流程。

    * 步骤一：创建基础员工表（不挂载外键约束）。

        ```sql
        CREATE TABLE `employee` (
            `emp_id` INT PRIMARY KEY,
            `name` VARCHAR(20),
            `birth_date` DATE,
            `sex` VARCHAR(1),
            `salary` INT,
            `branch_id` INT,
            `sup_id` INT
        );
        ```

    * 步骤二：创建部门表并挂载经理外键。

        ```sql
        CREATE TABLE `branch` (
            `branch_id` INT PRIMARY KEY,
            `branch_name` VARCHAR(20),
            `manager_id` INT,
            FOREIGN KEY (`manager_id`) REFERENCES `employee`(`emp_id`) ON DELETE SET NULL
        );
        ```

    * 步骤三：通过 `ALTER TABLE` 动态追加员工表的外键约束。

        ```sql
        ALTER TABLE `employee` 
        ADD FOREIGN KEY (`branch_id`) 
        REFERENCES `branch`(`branch_id`) 
        ON DELETE SET NULL;
        
        ALTER TABLE `employee` 
        ADD FOREIGN KEY (`sup_id`) 
        REFERENCES `employee`(`emp_id`) 
        ON DELETE SET NULL;
        ```

    * 步骤四：独立创建客户表 `client`。

        ```sql
        CREATE TABLE `client` (
            `client_id` INT PRIMARY KEY,
            `client_name` VARCHAR(20),
            `phone` VARCHAR(20)
        );
        ```

    * 步骤五：创建复合主键兼复合外键的 `works_with` 表。

        ```sql
        CREATE TABLE `works_with` (
            `emp_id` INT,
            `client_id` INT,
            `total_sales` INT,
            PRIMARY KEY (`emp_id`, `client_id`),
            FOREIGN KEY (`emp_id`) REFERENCES `employee`(`emp_id`) ON DELETE CASCADE,
            FOREIGN KEY (`client_id`) REFERENCES `client`(`client_id`) ON DELETE CASCADE
        );
        ```

| 表名 | 字段名称 | 数据类型 | 约束属性 | 业务语义 |
| :--- | :--- | :--- | :--- | :--- |
| `employee` | `emp_id` | INT | PRIMARY KEY | 员工工号 |
| | `name` | VARCHAR(20) | - | 员工姓名 |
| | `birth_date`| DATE | - | 出生日期 |
| | `sex` | VARCHAR(1) | - | 性别（'M' / 'F'） |
| | `salary` | INT | - | 薪资 |
| | `branch_id` | INT | FOREIGN KEY | 所属部门代号（参照 branch） |
| | `sup_id` | INT | FOREIGN KEY | 直属主管工号（自引用参照 employee） |
| `branch` | `branch_id` | INT | PRIMARY KEY | 部门编号 |
| | `branch_name`| VARCHAR(20)| - | 部门名称 |
| | `manager_id`| INT | FOREIGN KEY | 部门主管工号（参照 employee） |
| `client` | `client_id` | INT | PRIMARY KEY | 客户编号 |
| | `client_name`| VARCHAR(20)| - | 客户名称 |
| | `phone` | VARCHAR(20)| - | 联系电话 |
| `works_with` | `emp_id` | INT | PRIMARY KEY, FOREIGN KEY | 经办员工工号（参照 employee） |
| | `client_id` | INT | PRIMARY KEY, FOREIGN KEY | 对应客户编号（参照 client） |
| | `total_sales`| INT | - | 销售成交金额 |

* **双向外键写入死锁破解策略（Write Deadlock Resolution Strategy）**

    * > **定义**：利用空值占位分阶段写入再行数据回填的策略，化解双向外键互锁导致的数据录入冲突。

    * 执行时序：

        ```sql
        -- 阶段 1：插入部门，主管字段设为 NULL 占位跳过外键校验
        INSERT INTO `branch` VALUES (1, '研發', NULL);
        INSERT INTO `branch` VALUES (2, '行政', NULL);
        INSERT INTO `branch` VALUES (3, '資訊', NULL);
        
        -- 阶段 2：部门已就绪，全量写入员工数据
        INSERT INTO `employee` VALUES (206, '小黃', '1998-10-08', 'F', 50000, 1, NULL);
        INSERT INTO `employee` VALUES (207, '小綠', '1985-09-16', 'M', 29000, 2, 206);
        INSERT INTO `employee` VALUES (208, '小黑', '2000-12-19', 'M', 35000, 3, 206);
        INSERT INTO `employee` VALUES (209, '小白', '1997-01-22', 'F', 39000, 3, 207);
        INSERT INTO `employee` VALUES (210, '小藍', '1995-03-27', 'M', 84000, 1, 207);
        
        -- 阶段 3：员工已入库，通过 UPDATE 补全部门表主管工号
        UPDATE `branch` SET `manager_id` = 206 WHERE `branch_id` = 1;
        UPDATE `branch` SET `manager_id` = 207 WHERE `branch_id` = 2;
        UPDATE `branch` SET `manager_id` = 208 WHERE `branch_id` = 3;
        
        -- 阶段 4：写入客户数据与销售关联明细
        INSERT INTO `client` VALUES (400, '阿狗', '254354335');
        INSERT INTO `client` VALUES (401, '阿貓', '256334335');
        INSERT INTO `client` VALUES (402, '旺來', '45354335');
        INSERT INTO `client` VALUES (403, '露西', '2234335');
        INSERT INTO `client` VALUES (404, '艾瑞克', '23454335');
        
        INSERT INTO `works_with` VALUES (206, 400, 70000);
        INSERT INTO `works_with` VALUES (207, 401, 26700);
        INSERT INTO `works_with` VALUES (208, 402, 22000);
        INSERT INTO `works_with` VALUES (208, 403, 5400);
        INSERT INTO `works_with` VALUES (210, 404, 34000);
        ```

### 外键删除联动规则与约束边界

* **外键联动删除策略（ON DELETE SET NULL and ON DELETE CASCADE）**

    * > **定义**：当父表主键记录被物理删除时，控制从表关联行生命周期的联动保护机制。

    * 外键置空 `ON DELETE SET NULL`：父表记录被删除时，子表中原指向该记录的外键字段自动被引擎重置为 NULL，子表整行记录得以安全保留。

    * 级联删除 `ON DELETE CASCADE`：父表记录被删除时，子表中所有关联该主键的记录行会被数据库连带彻底抹除。

    * 联动机制对比：

        ```text
        [ 父表 employee ] (执行 DELETE WHERE emp_id = 207)
               │
               ├───> branch.manager_id (配置: ON DELETE SET NULL)
               │     └─> 原指向 207 的外键字段自动被引擎修改为 NULL (部门记录本身保留)
               │
               └───> works_with.emp_id (配置: ON DELETE CASCADE)
                     └─> 原包含 207 的整行销售关联记录被连带彻底删除
        ```

* **主外键冲突约束边界（Primary Key Non-null Boundary）**

    * > **定义**：主键非空约束对从表外键联动置空行为施加的物理限制。

    * 冲突机理：主键与复合主键的核心特性是必须满足全局唯一且强制非空（NOT NULL）。若将参与主键的外键列配置为 `ON DELETE SET NULL`，删除父表记录时引擎尝试写入 NULL，将直接违背主键非空规则。

    * > ⚠️ **易错点**：若子表中的外键同时参与构成了该子表的主键或复合主键，该外键绝对不可配置为 `ON DELETE SET NULL`，因主键强制非空，置空会产生底层逻辑冲突，必须配置为 `ON DELETE CASCADE`。

---

## 应用程序数据库交互与事务管理

### 驱动连接与游标生命周期

* **数据库驱动与连接对象（mysql.connector and Connection）**

    * > **定义**：高级编程语言与关系型数据库服务端建立底层网络 Socket 通信的驱动适配层。

    * 依赖库安装：在终端使用包管理器安装官方支持驱动：

        ```bash
        pip install mysql-connector-python
        ```

    * 连接参数规范：

        * `host`：数据库主机地址（本地传入 `'localhost'` 或 `'127.0.0.1'`）。

        * `port`：监听端口（MySQL 缺省端口为 3306）。

        * `user`：认证账号（通常为 `'root'`）。

        * `password`：鉴权密码。

        * `database`：可选参数，直连指定逻辑库上下文，省去脚本内部显式派发 `USE` 指令。

* **执行游标与数据提取（Cursor and Fetchall）**

    * > **定义**：用于在建立的数据库连接会话内部执行 SQL 语句并遍历结果集的中间抽象管道。

    * 全生命周期拓扑：

        ```text
        [ Python 运行时环境 ]
                 │
                 │  1. mysql.connector.connect(host, port, user, password, database)
                 ▼
        [ 物理数据库连接 Connection ]
                 │
                 │  2. connection.cursor()
                 ▼
        [ 数据库游标 Cursor ]
                 │
                 │  3. cursor.execute("SQL 语句")
                 │
                 ├─── 查询流程 ──> 4. records = cursor.fetchall() (提取并遍历输出)
                 │
                 └─── 写入流程 ──> 4. connection.commit() (事务持久化落盘)
                 │
                 ▼
        [ 5. 显式释放资源: cursor.close() / connection.close() ]
        ```

    * 数据集提取：通过 `cursor.fetchall()` 一次性从网络缓冲区抽取全部匹配行并封装为元组列表供应用程序遍历。

### 数据持久化与事务提交机制

* **写操作会话事务暂存区（DML Transaction Staging Area）**

    * > **定义**：对数据表内容产生实际变更的操作在未正式落盘前所处的临时会话缓冲区。

    * 影响范围：涵盖所有数据操纵语言（DML）指令，包括 `INSERT`、`UPDATE`、`DELETE`。

    * 运行状态：调用 `cursor.execute()` 之后，数据变更仅在当前连接的内存暂存区就绪，未写入底层物理存储文件。

* **显式事务提交（Explicit Transaction Commit）**

    * > **定义**：指示数据库引擎将当前事务暂存区中的全部数据修改操作正式持久化落盘到磁盘文件的系统调用。

    * 规则要求：凡涉及新增、更新、删除操作，必须显式调用 `connection.commit()` 提交事务；若未调用，脚本退出后所有变更将被自动丢弃。

        ```python
        import mysql.connector
        
        connection = mysql.connector.connect(
            host="localhost",
            port=3306,
            user="root",
            password="password",
            database="sql_tutorial"
        )
        cursor = connection.cursor()
        
        # 执行数据变更
        cursor.execute("UPDATE `branch` SET `manager_id` = 206 WHERE `branch_id` = 4")
        # 显式提交事务，使物理数据正式生效
        connection.commit()
        
        cursor.close()
        connection.close()
        ```
