# 笔记 02 关系数据库标准语言 SQL

> 整理自《数据库系统概论》P16~P29 单集精读长文
> 渲染支持：Typora / VS Code Markmap / XMind 一键脑图

```text
关系数据库标准语言 SQL（P16-P29）
├── SQL 总览与三级模式映射 (P16)
│   ├── 语言构成 ──── DDL、DQL、DML、DCL 四类与核心关键字
│   ├── 技术特征 ──── 综合一体、非过程化、面向集合、两种用法
│   └── 模式映射 ──── 基本表→模式，视图→外模式，存储文件→内模式
├── 模式与基本表定义 (P17-P18)
│   ├── 模式对象 ──── CREATE/DROP SCHEMA 与 CASCADE/RESTRICT
│   ├── 建表约束 ──── 数据类型矩阵、列级/表级约束与三表建模
│   └── 结构演进 ──── ALTER TABLE 与 DROP TABLE 依赖级联
├── 索引机制与物理优化 (P19)
│   ├── 结构原理 ──── 全表扫描 O(n) 与 B+ 树 O(log n)
│   ├── 索引分类 ──── 聚集索引唯一性与非聚集索引回表
│   └── 索引管理 ──── CREATE/DROP INDEX 与建索引权衡
├── 单表查询与空值逻辑 (P20-P22)
│   ├── 投影过滤 ──── 目标列表达式、WHERE 谓词与 LIKE/ESCAPE
│   ├── 空值逻辑 ──── NULL 三值逻辑与 IS NULL 判定
│   ├── 排序聚集 ──── ORDER BY 多列排序与五大聚集函数
│   └── 分组过滤 ──── GROUP BY 投影铁律与 HAVING 组过滤
├── 多表连接查询 (P23)
│   ├── 连接语法 ──── 隐式连接与 ANSI JOIN 的等价性
│   ├── 连接类型 ──── 等值、自然、自身与外连接族
│   └── 多表连接 ──── 三表链式连接与 ON/WHERE 分工
├── 嵌套查询与量词 (P24-P26)
│   ├── 执行模型 ──── 不相关子查询一次求值与相关子查询循环
│   ├── 比较谓词 ──── 标量比较子查询与基数违例防范
│   ├── 集合谓词 ──── IN、ANY、ALL 与聚集函数等价变换
│   └── 存在量词 ──── EXISTS 布尔检验与双重 NOT EXISTS
├── 集合查询与派生表 (P27)
│   ├── 并相容性 ──── 列数一致与对应位置类型兼容
│   ├── 集合运算 ──── UNION、UNION ALL、INTERSECT、EXCEPT
│   └── 内联视图 ──── FROM 子句子查询与强制别名
├── 数据更新 DML (P28)
│   ├── 插入数据 ──── INSERT VALUES 与子查询批量灌入
│   ├── 修改删除 ──── UPDATE、DELETE 与 TRUNCATE 辨析
│   └── 约束防御 ──── 实体、参照、用户定义三级校验
└── 视图机制 (P29)
    ├── 定义安全 ──── CREATE VIEW 与 WITH CHECK OPTION
    ├── 视图消解 ──── 查询重写为基本表查询
    ├── 可否更新 ──── 行列子集视图与只读视图类型
    └── 工程价值 ──── 简化、多视角、逻辑独立性、安全隔离
```

---

## 1. SQL 语言构成与三级模式映射

> **一句话主旨**：界定 SQL 的四类语句与四条技术特征，给出基本表、视图、存储文件与三级模式的对应关系。

### 1.1 语言构成

* **结构化查询语言（Structured Query Language，SQL）**
    * > 定义：关系数据库系统的通用标准语言，统一了数据的定义、查询、操纵与控制功能。
    * 1974 年由 Boyce 和 Chamberlin 提出，最初称 SEQUEL，在 IBM 的 System R 实验关系数据库上实现。
    * ANSI 与 ISO 先后将其确立为国际工业标准，迭代版本为 SQL-86、SQL-89、SQL-92、SQL:1999、SQL:2003、SQL:2016。
    * > 来源: P16

* **四种语句子系统**
    * 数据定义语言（DDL）：`CREATE`、`DROP`、`ALTER`，用于定义模式、表、视图、索引。
    * 数据查询语言（DQL）：`SELECT`。
    * 数据操纵语言（DML）：`INSERT`、`UPDATE`、`DELETE`。
    * 数据控制语言（DCL）：`GRANT`、`REVOKE`、`COMMIT`、`ROLLBACK`。
    * > 来源: P16

### 1.2 技术特征

* **综合一体（All-in-one）**
    * 数据定义、操纵、控制的语言风格高度统一，DBA 与开发者在同一会话中自由流转。
    * 支持数据库运行生命周期中动态修改模式（Dynamic Schema Evolution），无需停机重新编译。
    * > 来源: P16

* **高度非过程化（Non-procedural）**
    * 用户只提出“做什么（What to do）”，不指定存取路径，也不了解底层物理文件与树形索引。
    * 复杂存取路径选择与代数优化完全由 DBMS 引擎自动完成。
    * > 来源: P16

* **面向集合的操作方式（Set-at-a-time）**
    * 与传统格式化模型的“一次一记录（Record-at-a-time）”游标循环相对。
    * 查询结果是元组集合，且一次 `INSERT`、`UPDATE`、`DELETE` 的作用对象同样可以是条件过滤出的一组元组。
    * > 来源: P16

* **同一语法提供两种使用方式**
    * 交互式 SQL：在终端与 Navicat、DBeaver、pgAdmin 等工具中直接输入并即时执行。
    * 嵌入式 SQL：嵌入 C/C++、Java、Python、Go 等宿主语言，经游标或 ORM 协同；两种场景语法体系完全一致。
    * > 来源: P16

### 1.3 三级模式映射

* **基本表（Base Table）对应概念模式**
    * 本身独立存在，每个基本表对应一个关系；若干基本表构成的全局逻辑结构即概念模式的落地。
    * > 来源: P16

* **视图（View）对应外模式**
    * 从一个或几个基本表（或其他视图）导出的“虚表”；数据库只存放视图定义，不存放对应数据。
    * > 来源: P16

* **存储文件（Stored File）对应内模式**
    * 逻辑上的基本表在物理层由一个或多个存储文件实现，含记录物理存储格式、页面布局与索引结构。
    * > 来源: P16

---

## 2. 模式与基本表的定义

> **一句话主旨**：给出模式作为命名空间的定义与删除、常用数据类型矩阵，以及内嵌三大完整性约束的建表语法。

### 2.1 模式的定义与删除

* **模式（Schema）**
    * > 定义：数据库对象的组织容器与命名空间（Namespace），一个模式内包含若干基本表、视图与索引。
    * 不同模式可含同名基本表（`HR.Employee` 与 `Finance.Employee` 不冲突）；权限控制可精准绑定到模式级。
    * `CREATE SCHEMA <模式名> AUTHORIZATION <用户名> [子句集];`；省略模式名时默认等于用户名。
    * `AUTHORIZATION` 指定模式所有者；子句集允许定义模式的同时直接创建基本表、视图与授权语句。
    * > 来源: P17

* **删除模式**
    * `DROP SCHEMA <模式名> <CASCADE | RESTRICT>;`
    * `CASCADE`：删除模式的同时连带删除其下全部基本表、视图、索引。
    * `RESTRICT`：仅当模式为空（无任何下属对象）时才允许删除，否则立即报错拒绝执行。
    * > 来源: P17

### 2.2 数据类型与建表约束

* **常用 SQL 标准数据类型**
    * `INT`/`INTEGER` 4 字节带符号整数（约 $-2.1 \times 10^9 \sim 2.1 \times 10^9$）；`SMALLINT` 2 字节（$-32768 \sim 32767$）；`BIGINT` 8 字节大整数。
    * `NUMERIC(p, s)`/`DECIMAL(p, s)` 高精度定点小数，$p$ 为总有效位数（Precision），$s$ 为小数位数（Scale），用于银行金额与财务结算。
    * `CHAR(n)` 定长字符（不足 $n$ 位末尾补空格）；`VARCHAR(n)` 变长字符（按实际内容占用）。
    * `DATE` 格式 `YYYY-MM-DD`；`TIME` 格式 `HH:MM:SS`；`TIMESTAMP` 含日期、时间、毫秒并支持时区。
    * > 来源: P17

* **列级约束与表级约束**
    * > 定义：列级约束紧跟某一列定义，只对该单一列生效（如 `NOT NULL`、`UNIQUE`）；表级约束写在所有列定义完成后的末尾。
    * 当约束涉及多个列（复合主码、外码）时，必须声明为表级约束。
    * 若在多个列后分别写 `PRIMARY KEY`，系统判定为声明多个独立主码而报错（一个表只能有一个主码）；复合主码须写 `PRIMARY KEY (列1, 列2, …)`。
    * > 易错点：创建含外码的表时必须先创建被参照的主表，后创建参照的从表，否则外码引用找不到目标表而报错中断。
    * > 来源: P17

* **经典三表建表要点**
    * `Student` 的单列主码 `PRIMARY KEY` 隐含 `NOT NULL` + `UNIQUE`；`CHECK` 用于枚举与区间（如 `Ssex IN ('男','女')`、`Sage BETWEEN 15 AND 45`）。
    * `Course` 的先修课 `Cpno` 以 `FOREIGN KEY (Cpno) REFERENCES Course(Cno)` 实现表级自参照外码。
    * `SC` 用表级复合主码 `PRIMARY KEY (Sno, Cno)`；对 `Sno` 用 `ON DELETE CASCADE`/`ON UPDATE CASCADE`，对 `Cno` 用 `ON DELETE RESTRICT`。
    * > 来源: P17

---

## 3. 基本表结构的动态演进

> **一句话主旨**：整理 `ALTER TABLE` 的列与约束变更、`DROP TABLE` 的依赖判定，以及 `DELETE` 与 `DROP` 的本质区别。

### 3.1 列与约束的动态修改

* **`ALTER TABLE` 语法框架**

```sql
ALTER TABLE <表名>
    [ ADD [COLUMN] <新列名> <数据类型> [列级完整性约束] ]
    [ DROP [COLUMN] <列名> [CASCADE | RESTRICT] ]
    [ ALTER COLUMN <列名> <新数据类型> ]
    [ ADD CONSTRAINT <约束名> <表级完整性约束> ]
    [ DROP CONSTRAINT <约束名> [CASCADE | RESTRICT] ];
```

* **新增列 `ADD COLUMN`**
    * `ALTER TABLE Student ADD EntranceDate DATE;`
    * > 易错点：表中已存在历史持久化数据时，新增列通常不能声明为 `NOT NULL`（除非同时指定 `DEFAULT`），否则既有记录在该列无值可填，直接违反非空约束导致 DDL 执行失败。
    * > 来源: P18

* **修改列类型 `ALTER COLUMN`**
    * `ALTER TABLE Student ALTER COLUMN Sname VARCHAR(50);`
    * 类型变更通常只允许向前兼容扩容（20 字符扩为 50）；缩小类型或将字符串转数值时，既有超长或不兼容数据会使 DBMS 终止变更以保护数据完整性。
    * > 来源: P18

* **表级约束的增删**
    * `ALTER TABLE Student ADD CONSTRAINT C_Student_Age CHECK (Sage BETWEEN 15 AND 40);`
    * `ALTER TABLE Student DROP CONSTRAINT C_Student_Age;`
    * 显式命名约束可极大提高后续数据库维护与删除的灵活性。
    * > 来源: P18

### 3.2 删除基本表与依赖判定

* **`DROP TABLE <表名> [RESTRICT | CASCADE];`**
    * `RESTRICT`（默认）：删除前扫描数据字典，只要存在外码参照该表主码、视图基于该表定义、或触发器与存储过程依赖该表，立即报错拒绝删除。
    * `CASCADE`：强制销毁该表，连带自动销毁其全部索引、使基于它的视图失效并删除、将参照它的外码约束从数据字典注销。
    * > 来源: P18

* **`DELETE` 与 `DROP` 的区别**
    * `DELETE FROM Student;` 是 DML：只清空全部数据元组，表的模式定义、列结构、索引与约束仍完整保存在数据字典中，之后仍可执行 `INSERT`。
    * `DROP TABLE Student;` 是 DDL：不仅清空全部数据，还把表的元数据结构与模式定义从数据字典彻底抹去。
    * > 来源: P18

* **级联选项的风险**
    * > 易错点：`CASCADE` 具有强隐式级联破坏性，核心主表可能被数十个下游视图与外键弱依赖，一条语句可在不知情下摧毁大量下游关键业务报表与视图，造成不可逆的生产事故。
    * > 来源: P18

---

## 4. 索引机制与物理优化

> **一句话主旨**：说明索引的物理本质与 B+ 树特性，区分聚集与非聚集索引，并给出索引 DDL 与建索引权衡。

### 4.1 索引的物理结构与分类

* **索引（Index）**
    * > 定义：存储在磁盘上的一种特殊辅助数据结构，按指定的排序列组织指针。
    * 无索引时数据库引擎自前向后加载全部数据页逐行比对，称全表扫描（Table Scan），时间复杂度 $O(n)$；建立索引后检索复杂度降至 $O(\log n)$。
    * > 来源: P19

* **B+ 树索引（B+ Tree Index）**
    * 多路平衡查找树，所有叶子结点深度完全相同，树高通常 3~4 层即可支撑数千万级数据，查询单条记录只需 3~4 次磁盘 I/O。
    * 非叶子结点仅存索引键值与指针，不存具体行数据，单页容纳海量分支指针以提高扇出比（Fan-out）。
    * 叶子结点保存全部实际数据记录指针（或真实行记录），并按键值升序用双向链表串联，支撑范围查询（Range Query）与排序输出。
    * 在 Oracle、MySQL InnoDB、PostgreSQL、SQL Server 等主流商用数据库中应用最广泛、最成熟。
    * > 来源: P19

* **聚集索引（Clustered Index）**
    * 表中行的物理存放顺序与索引顺序完全一致，叶子结点就是真实数据页（含全部字段）。
    * > 易错点：一个基本表上最多只能建立一个聚集索引——物理磁盘上的数据行只能有一种物理排布次序。
    * 查询连续范围或分组排序极快，物理相邻数据页可触发顺序预读；对聚集索引列更新或插入乱序新行会触发昂贵的数据行物理位移乃至数据页分裂（Page Split），故频繁变动的属性不适合建聚集索引。
    * > 来源: P19

* **非聚集索引（Non-clustered Index）**
    * 索引逻辑顺序与磁盘物理顺序完全无关，索引表与基本表物理独立；一个基本表可建立多个非聚集索引。
    * 叶子结点仅存索引键值加指向主表数据行的物理寻址指针（Row ID，RID）或主键值。
    * 检索时先查辅助索引取得主键/RID，再回主数据表提取其他字段，该过程称回表查询（Bookmarking Lookup）。
    * > 来源: P19

### 4.2 索引 DDL 与设计权衡

* **建立索引**

```sql
CREATE [UNIQUE] [CLUSTER] INDEX <索引名>
ON <表名> (<列名> [<次序>] [, <列名> [<次序>]] ...);
```

* `UNIQUE` 规定每个键值只对应唯一一条记录；`CLUSTER` 表示建立聚集索引；`<次序>` 取 `ASC`（升序，默认）或 `DESC`（降序）。
* 例：`CREATE UNIQUE CLUSTER INDEX Idx_Student_Sno ON Student(Sno ASC);`
* 例：`CREATE INDEX Idx_SC_SnoCno ON SC(Sno ASC, Cno DESC);`
* > 来源: P19

* **删除索引**
    * `DROP INDEX <索引名> [ON <表名>];`；只删除加速查找的辅助 B+ 树文件，原基本表中的业务数据不受任何损害。
    * > 来源: P19

* **建索引的收益与代价**
    * 收益：极速提升 `SELECT` 读性能、显著降低 `ORDER BY` 排序开销、强化主外码唯一性校验。
    * 代价：额外占用大量磁盘存储空间、拖慢 `INSERT`/`UPDATE`/`DELETE`、每次写数据必须同步维护 B+ 树。
    * 适合建索引：常作 `WHERE` 过滤条件的列、常作 `JOIN` 连接键（如外码）的列、常用于 `GROUP BY` 或 `ORDER BY` 的列。
    * 不适合建索引：极少出现在查询条件中的列、取值极少（低基数，如性别）的列、频繁修改写入的列、仅几行几十行的小字典表。
    * > 来源: P19

---

## 5. 单表查询：投影、过滤与空值逻辑

> **一句话主旨**：给出 `SELECT` 的目标列表达式、`WHERE` 各类谓词与模糊匹配转义，以及 `NULL` 的三值逻辑铁律。

### 5.1 目标列表达式与投影

* **`SELECT` 查询骨架**
    * `SELECT` 目标列表达式对应关系代数的投影（Projection，$\pi$）；`WHERE` 条件表达式对应选择（Selection，$\sigma$）。
    * 单表查询逻辑执行流向：`FROM` 载入基表 → `WHERE` 逐行评估谓词过滤元组 → `SELECT` 提取属性、执行标量计算并处理 `DISTINCT`。
    * > 来源: P20

* **`ALL` 与 `DISTINCT`**
    * `ALL` 为默认，保留结果集中的所有重复元组；`DISTINCT` 强制执行集合去重，消除重复行。
    * 例：`SELECT DISTINCT Sno FROM SC;`
    * > 来源: P20

* **目标列表达式的计算与别名**
    * 目标列不仅可为列名，还可含算术运算、字符串操作与函数映射，并用 `AS` 赋予易读别名。
    * 例：`SELECT Sname AS 姓名, 'Birth Year:' AS 标识标签, 2026 - Sage AS 出生年份, LOWER(Sdept) AS 院系编码小写 FROM Student;`
    * > 来源: P20

### 5.2 `WHERE` 谓词体系

| 查询条件分类 | 谓词符号 / 关键字 | 语法模式示范 |
| :--- | :--- | :--- |
| 比较大小 | `=`, `>`, `<`, `>=`, `<=`, `!=`, `<>` | `WHERE Sage >= 20` |
| 确定闭区间范围 | `BETWEEN ... AND ...` / `NOT BETWEEN` | `WHERE Sage BETWEEN 18 AND 22` |
| 确定有限集合 | `IN (...)` / `NOT IN (...)` | `WHERE Sdept IN ('CS', 'MA', 'IS')` |
| 字符模糊匹配 | `LIKE` / `NOT LIKE` | `WHERE Sname LIKE '张%'` |
| 空值判断 | `IS NULL` / `IS NOT NULL` | `WHERE Grade IS NULL` |
| 多重复合逻辑 | `AND`, `OR`, `NOT`（优先级 NOT > AND > OR） | `WHERE Sdept = 'CS' AND Sage < 20` |

* **谓词组合与书写**
    * 各类谓词通过逻辑联结词 `AND`、`OR`、`NOT` 精确界定候选元组集合，优先级为 `NOT` > `AND` > `OR`。
    * `IN` 与 `BETWEEN ... AND ...` 结合括号使用表达清晰，执行引擎易于命中索引。
    * > 来源: P20

* **字符匹配 `LIKE` 与转义**

```sql
[NOT] LIKE '<匹配串>' [ESCAPE '<换码字符>']
```

* 百分号 `%` 代表任意长度（包括长度为 0）的字符串，`Sname LIKE '刘%'` 可匹配“刘邦”“刘备”甚至单字“刘”。
* 下划线 `_` 代表任意单个字符，`Sname LIKE '欧阳_'` 只匹配恰好 3 个字的名字，不匹配“欧阳震华”。
* 检索串本身含 `%` 或 `_` 时用 `ESCAPE` 声明换码字符，如 `WHERE Cname LIKE 'DB\_%' ESCAPE '\';` 使 `_` 作纯文本字面值。
* > 来源: P20

### 5.3 空值与三值逻辑

* **空值（NULL）**
    * > 定义：代表“未知（Unknown）”“不存在”或“无意义”，不是空字符串 `""`，也不是数字零 `0`。
    * > 易错点：严禁写 `WHERE Grade = NULL`，必须且只能写 `WHERE Grade IS NULL` 或 `WHERE Grade IS NOT NULL`。
    * > 来源: P20

* **三值逻辑（Three-Valued Logic）**
    * 任何数值与 `NULL` 进行算术比较（如 `NULL = 100`、`NULL = NULL`、`NULL > 50`），判定结果一律为 UNKNOWN。
    * `WHERE` 根本法则：只有谓词条件最终计算结果为严格 TRUE 的元组才被选入结果集，结果为 FALSE 或 UNKNOWN 的行均被直接丢弃。
    * `WHERE Grade <> 60` 时成绩为 `NULL` 的行因结果为 UNKNOWN 而被过滤，故永远查不出包含 NULL 成绩的行。
    * > 来源: P20

```text
        AND      TRUE      FALSE     UNKNOWN
       TRUE      TRUE      FALSE     UNKNOWN
       FALSE     FALSE     FALSE     FALSE
       UNKNOWN   UNKNOWN   FALSE     UNKNOWN

        OR       TRUE      FALSE     UNKNOWN
       TRUE      TRUE      TRUE      TRUE
       FALSE     TRUE      FALSE     UNKNOWN
       UNKNOWN   TRUE      UNKNOWN   UNKNOWN
```

---

## 6. 排序与聚集函数

> **一句话主旨**：给出 `ORDER BY` 的多列优先级与空值位阶、五大聚集函数的功能矩阵，以及空值忽略铁律。

### 6.1 结果排序

* **`ORDER BY` 子句**
    * `ORDER BY <列名> [ASC | DESC] [, <列名> [ASC | DESC]] ...`
    * `ASC` 升序为系统默认（数值从小到大、字母 A 到 Z）；`DESC` 降序（数值从大到小、日期由新到旧）。
    * 多属性列层级排序：先按第一个列的值排序，仅当第一个列的值完全相同时，才用第二个列的值进行内部微调。
    * 例：`SELECT Sno, Grade FROM SC WHERE Cno = '1' ORDER BY Grade DESC, Sno ASC;`
    * > 来源: P21

* **排序与空值（NULL）的交互**
    * SQL 标准将 `NULL` 视作具有特殊位阶；多数数据库（Oracle、PostgreSQL）在 `ASC` 时默认将 `NULL` 置于最后（或最前），`DESC` 时相反。
    * 标准 SQL 允许用 `NULLS FIRST` 或 `NULLS LAST` 显式固定空值行在报表中的呈现位置。
    * > 来源: P21

### 6.2 五大聚集函数

| 聚集函数名 | 输入参数形式 | 功能描述与语义规约 | 适用数据类型 |
| :--- | :--- | :--- | :--- |
| `COUNT(*)` | `*` | 统计元组总行数，无论某行是否含 NULL 一律计入 | 任意表 |
| `COUNT(<列名>)` | `[DISTINCT\|ALL] 列` | 统计该列非空值行数，自动忽略 NULL | 任意类型 |
| `SUM(<列名>)` | `[DISTINCT\|ALL] 列` | 计算该列数值总和，自动忽略 NULL | 仅限数值型 |
| `AVG(<列名>)` | `[DISTINCT\|ALL] 列` | 计算该列数值平均值，自动忽略 NULL | 仅限数值型 |
| `MAX(<列名>)` | `[DISTINCT\|ALL] 列` | 求该列最大值，自动忽略 NULL | 数值/字符/日期 |
| `MIN(<列名>)` | `[DISTINCT\|ALL] 列` | 求该列最小值，自动忽略 NULL | 数值/字符/日期 |

* **聚集函数的空值处理铁律**
    * > 易错点：除 `COUNT(*)` 外，其余所有聚集函数在计算时均自动忽略空值（NULL）。
    * 对成绩集合 `[100, 80, NULL]`：`COUNT(*)` 为 3，`COUNT(Grade)` 为 2，`AVG(Grade)` 为 $\frac{100+80}{2} = 90.0$，而非 $\frac{100+80+0}{3} = 60.0$。
    * 若要求缺考者按 0 分计入平均分，须显式转换：`SELECT AVG(COALESCE(Grade, 0)) FROM SC;`
    * > 来源: P21

* **`DISTINCT` 在聚集中的去重效应**
    * `COUNT(DISTINCT Sno)` 统计选修过课程的学生人数，同一学生选多门课只算 1 人。
    * `SUM(DISTINCT Ccredit)` 统计所有不同学分的总和。
    * > 来源: P21

* **`WHERE` 严禁使用聚集函数**
    * > 易错点：`SELECT Sname FROM Student WHERE Sage = MAX(Sage);` 是非法语句；`WHERE` 在聚合前对单行元组逐一评估，扫描某行时全表 `MAX(Sage)` 尚未计算出来。
    * 正确写法须借子查询：`SELECT Sname FROM Student WHERE Sage = (SELECT MAX(Sage) FROM Student);`
    * > 来源: P21

---

## 7. 分组与 `HAVING`

> **一句话主旨**：给出 `GROUP BY` 的分组语义与投影合法性铁律，以及 `HAVING` 组过滤与 `WHERE` 的分工。

### 7.1 `GROUP BY` 分组

* **`GROUP BY <列名1> [, <列名2>] ...`**
    * 系统扫描结果表，将所有在指定分组列上取值完全相同的元组归为同一个组。
    * 一旦执行分组，后续所有聚集函数不再针对全表计算，而是对每一个独立的组分别独立计算。
    * > 来源: P22

* **投影合法性铁律（Single-Value Rule）**
    * > 易错点：含 `GROUP BY` 的查询中，`SELECT` 目标列出现的属性有且仅能属于两类——出现在 `GROUP BY` 中的分组列，或被聚集函数包裹的聚合项。
    * 非法示例：`SELECT Sdept, Sname, AVG(Sage) FROM Student GROUP BY Sdept;`，`Sname` 既不在分组列也未被聚集包裹，标准 SQL 直接报错拦截。
    * 原因：按系分组后每组只输出唯一一行，组内存在张三、李四、王五等多名学生，无法从逻辑上决定填入哪一个 `Sname`，产生多义性。
    * > 来源: P22

### 7.2 `HAVING` 与 `WHERE` 的分工

* **`HAVING` 分组过滤**
    * 需对分组计算出的统计指标施加条件过滤时使用 `HAVING`：`SELECT Cno, COUNT(Sno) AS 选课人数, AVG(Grade) AS 平均分 FROM SC GROUP BY Cno HAVING AVG(Grade) >= 80;`
    * > 来源: P22

* **`WHERE` 与 `HAVING` 对比**

| 比较维度 | WHERE 子句 | HAVING 子句 |
| :--- | :--- | :--- |
| 作用对象 | 单行元组（Row） | 已划分的数据组（Group） |
| 执行时机 | 在 `GROUP BY` 分组之前执行 | 在分组与聚集计算之后执行 |
| 聚集函数支持 | 严禁使用聚集函数 | 高度依赖聚集函数 |
| 性能影响 | 越早执行越好，减少参与后续分组的数据量 | 较后执行，无法减少分组过程本身的计算开销 |

* **完整执行时序**
    * `FROM`（含 `JOIN`）→ `WHERE` 逐行过滤 → `GROUP BY` 分组 → 各组聚集计算 → `HAVING` 组过滤 → `SELECT` 与 `ORDER BY` 输出。
    * 复合条件分组例：`WHERE Grade >= 60` 先过滤，再 `GROUP BY Cno`，最后 `HAVING COUNT(Sno) >= 3 AND AVG(Grade) > 75`。
    * > 来源: P22

---

## 8. 连接查询

> **一句话主旨**：给出隐式连接与 ANSI `JOIN` 语法的等价关系，以及等值、自然、自身与外连接族的匹配规则。

### 8.1 连接语法与等值连接

* **两种等价连接语法**
    * 传统隐式连接（`WHERE` 子句）：`SELECT * FROM A, B WHERE A.id = B.id;`，易因遗漏 `WHERE` 条件导致笛卡尔积灾难。
    * 现代 ANSI 显式连接（`JOIN ... ON`）：`SELECT * FROM A [INNER] JOIN B ON A.id = B.id;`，连接条件置于 `ON`、行过滤置于 `WHERE`，逻辑解耦。
    * > 来源: P23

* **等值连接（Equi-join）**
    * 在连接谓词中用等号关联两张表的同域属性。
    * 例：`SELECT Student.*, SC.* FROM Student INNER JOIN SC ON Student.Sno = SC.Sno;`
    * > 来源: P23

* **自然连接（NATURAL JOIN）**
    * 自动寻找两张表中的所有同名公共列执行等值比对，并在最终输出中自动去除重复的同名列。
    * > 易错点：生产代码推荐显式 `JOIN ... ON` 而非 `NATURAL JOIN`；若两表将来新增同名但业务无关的列（如都新增 `Status` 或 `Remark`），自然连接会自动将其并入连接条件，使查询结果意外变为空集。
    * > 来源: P23

### 8.2 自身连接与外连接

* **自身连接（Self-Join）**
    * 适用场景：一张表的某属性参照本表主码（自参照外码，如课程与先修课、员工与主管）。
    * 核心规约：在 `FROM` 中为同一物理表赋两个不同别名，使其逻辑上分裂为两张互不干扰的虚拟表。
    * 例：`FROM Course C1 LEFT JOIN Course C2 ON C1.Cpno = C2.Cno LEFT JOIN Course C3 ON C2.Cpno = C3.Cno;` 查询二重先修课。
    * > 来源: P23

* **外连接族（Outer Join）**
    * 内连接仅输出两表均成功匹配的元组，未匹配者作为悬浮元组（Dangling Tuple）被丢弃；外连接将其纳入结果集并将缺失方属性补 `NULL`。
    * 左外连接 `LEFT [OUTER] JOIN`：以左表为基准，左表所有行全部保留，右表无匹配则对应字段全部填 `NULL`。
    * 右外连接 `RIGHT [OUTER] JOIN`：以右表为基准全部保留；`A RIGHT JOIN B` 完全等价于 `B LEFT JOIN A`。
    * 全外连接 `FULL [OUTER] JOIN`：左右两表所有悬浮元组全部保留，任何未匹配方均补 `NULL`。
    * > 来源: P23

* **多表连续连接**
    * 连接可跨越三张及以上表，按 `Student INNER JOIN SC ON Student.Sno = SC.Sno INNER JOIN Course ON SC.Cno = Course.Cno` 逐级拼接，行过滤条件统一放 `WHERE`。
    * > 来源: P23

---

## 9. 嵌套查询与量词

> **一句话主旨**：区分相关与不相关子查询的执行模型，整理比较、集合与存在三类谓词的语义及等价变换。

### 9.1 嵌套查询执行模型

* **查询块与子查询**
    * > 定义：一个 `SELECT-FROM-WHERE` 语句称一个查询块（Query Block）；嵌入另一查询块 `WHERE` 或 `HAVING` 中的查询称子查询（Subquery，内层），外部结构称父查询（外层）。
    * > 来源: P24

* **不相关子查询（Uncorrelated Subquery）**
    * 内层子查询完全独立，不依赖父查询任何字段；调度策略为由内向外，内层只执行一次。
    * 执行流程：先独立执行内层产出固定单值或集合 → 将其替换为父查询条件中的字面常量 → 父查询单次扫描输出最终结果。
    * > 来源: P24

* **相关子查询（Correlated Subquery）**
    * 内层子查询的 `WHERE` 条件中引用了外层表的属性；调度策略类似双重 for 循环，外层每流转一行，内层带该行属性值重新执行一次计算。
    * 执行流程：从外层取出第一元组并传入相关属性 → 内层带参执行计算条件真值 → 外层据真值决定是否输出 → 外层游标移到下一元组重复，直至外层扫描完毕。
    * > 来源: P24

### 9.2 比较、集合与存在谓词

* **带比较运算符的子查询**
    * 当确信内层只返回单个标量值（Single Value）时，可在父查询属性与子查询之间用 `=, <>, <, <=, >, >=`。
    * 不相关例：`WHERE Sdept = (SELECT Sdept FROM Student WHERE Sname = '刘晨');`
    * 相关例：`WHERE Grade > (SELECT AVG(Grade) FROM SC y WHERE y.Cno = x.Cno);`
    * > 易错点：若内层返回多行（如重名刘晨分属计算机系与数学系），标量比较抛出 `Subquery returns more than 1 row`；无法排除多行时应改用 `IN` 谓词。
    * > 来源: P24

* **`IN` 谓词与解嵌套**
    * `expr IN (Subquery)` 判断属性值是否属于子查询返回的值集合；三层穿透例：`WHERE Sno IN (SELECT Sno FROM SC WHERE Cno IN (SELECT Cno FROM Course WHERE Cname = '信息系统'))`。
    * 带 `IN` 的嵌套查询逻辑上等价于多表 `INNER JOIN`；优化器会自动将满足条件的 `IN` 嵌套解嵌套（Unnesting）改写为半连接（Semi-Join）或内连接，以利用哈希关联或排序合并算法加速。
    * > 来源: P25

* **`ANY`（`SOME`）与 `ALL` 谓词**
    * 二者不能单独使用，必须紧跟在比较运算符之后（如 `> ANY`、`< ALL`、`= ANY`）。
    * `> ANY` 语义为“大于集合中至少一个值”，等价 `> (SELECT MIN(...) ...)`；`> ALL` 为“大于集合中所有值”，等价 `> (SELECT MAX(...) ...)`。
    * `< ANY` 等价 `< (SELECT MAX(...) ...)`；`< ALL` 等价 `< (SELECT MIN(...) ...)`。
    * `= ANY` 等价于 `IN` 谓词；`<> ALL` 等价于 `NOT IN` 谓词。
    * 工程推荐用 `MAX`/`MIN` 聚集写法：内层只产出单行标量，若 `MAX` 列有 B+ 树索引，可直接 $O(1)$ 树右侧边缘探针命中。
    * > 来源: P25

* **`EXISTS` 存在量词**
    * > 定义：对应数学逻辑中的存在量词（Existential Quantifier，$\exists$）；`EXISTS` 子查询不返回任何实际数据行或列值，只产生布尔真值 TRUE/FALSE。
    * `EXISTS`：内层结果集非空（至少一行）返回 TRUE，空集返回 FALSE；`NOT EXISTS` 逻辑取反。
    * 因只关心有无结果行，内层目标列统一写 `SELECT *` 或 `SELECT 1`，优化器按存在性短路扫描，找到第一条即返回。
    * > 来源: P26

### 9.3 双重 `NOT EXISTS` 与全称量词

* **全称量词 $\forall$ 的反演**
    * SQL 标准并未提供 `FOR ALL` 全称量词关键字，含“全部”“所有”的查询须用德·摩根反演：$\forall x P(x) \iff \neg \exists x (\neg P(x))$。
    * 语义转译：“所有的 $x$ 都满足性质 $P$”等价于“不存在任何一个 $x$，它不满足性质 $P$”。
    * > 来源: P26

* **查询选修了全部课程的学生**
    * 三层结构：外层 `Student S` → 中间层遍历全校 `Course C` → 最内层查 `SC` 中 S 是否选修 C。
    * 中间层抓到哪怕一门 S 没选的课，外层 `NOT EXISTS` 立即为 FALSE；只有全部选中、中间层恒为空集，外层才为 TRUE。
    * > 来源: P26

* **逻辑蕴涵 $p \to q$ 的求解**
    * 等价恒等式 $p \to q \iff \neg p \lor q \iff \neg (p \land \neg q)$，用于“至少选修了某学生全部课程”类问题。
    * 语义转译：“不存在这样一门课程：学生 202601 选修了它，而候选学生 $x$ 却没选修它”。
    * > 来源: P26

---

## 10. 集合查询与派生表

> **一句话主旨**：给出集合运算的并相容性约束、`UNION` 与 `UNION ALL` 的性能差异，以及派生表的强制别名规则。

### 10.1 集合运算

* **并相容性硬性约束**
    * 参与 `UNION`、`INTERSECT`、`EXCEPT` 运算的各 `SELECT` 语句，目标列属性个数必须完全相同。
    * 对应位置的列必须属于可相互隐式转换的数据类型或相同域。
    * > 来源: P27

* **`UNION` 与 `UNION ALL` 的性能分水岭**
    * `UNION` 在拼接两个结果集后自动启动昂贵的去重排序（Sort-Unique）或哈希去重，剔除全部分量相同的多余行。
    * `UNION ALL` 直接简单拼装数据行，绝不做任何去重扫描；已知两集合不可能重复或允许重复统计时必须选用，速度往往快数倍乃至数十倍。
    * > 来源: P27

* **交操作与差操作**
    * `INTERSECT` 仅保留双方共有的重合行，完全等价于关系代数 $\cap$。
    * `EXCEPT` 从第一个查询结果扣除在第二个查询中出现的行；Oracle 中写为 `MINUS`。
    * 例：`SELECT Sno FROM SC WHERE Cno = '1' EXCEPT SELECT Sno FROM SC WHERE Cno = '2';`
    * > 来源: P27

* **`ORDER BY` 放置纪律**
    * > 易错点：整个集合查询中 `ORDER BY` 排序子句只能在最后一个 `SELECT` 语句末尾出现一次，对最终总结果集统一排序；严禁在中间各单体子查询中各自添加独立 `ORDER BY`。
    * > 来源: P27

### 10.2 派生表

* **派生表（Derived Table）**
    * > 定义：直接在 `FROM` 子句中将一个完整子查询作为临时基本表使用，又称内联视图（Inline View）。
    * > 易错点：在 `FROM` 子句中放置派生表时必须且强制为其显式赋予表别名，未指定别名会导致数据库解析器报错。
    * > 来源: P27

* **典型用法：聚集结果的二次连接**
    * 先按学号聚合出选修超 3 门的学生学号与平均成绩，再与 `Student` 连接。
    * `FROM Student S INNER JOIN (SELECT Sno, AVG(Grade) AS AvgGrade FROM SC GROUP BY Sno HAVING COUNT(Cno) > 3) AS AvgSc ON S.Sno = AvgSc.Sno`
    * > 来源: P27

---

## 11. 数据更新

> **一句话主旨**：给出 `INSERT`、`UPDATE`、`DELETE` 的语法与批量写法，以及 DML 触发的三级完整性校验。

### 11.1 三类更新操作

* **插入 `INSERT`**

```sql
INSERT INTO <表名> [(<属性列1> [, <属性列2>] ...)]
VALUES (<常量1> [, <常量2>] ...);
```

* 显式指明列名列表时，列名次序可与建表物理列序不一致，只需 `VALUES` 常量类型与顺序与之严格一一对应。
* 省略列名列表时必须按建表定义的全量属性物理次序提供全部字段值，极易因后续加列导致代码失效。
* 未列出的字段自动填充建表时的 `DEFAULT` 默认值；无默认值且未声明 `NOT NULL` 时系统自动填入 `NULL`。
* > 来源: P28

* **批量插入（`INSERT ... SELECT`）**
    * 将 `SELECT` 抽出的海量结果集一次性批量插入目标表，用于数据清洗、历史归档与数仓聚合。
    * 例：`INSERT INTO Dept_Age (Sdept, AvgAge) SELECT Sdept, AVG(Sage) FROM Student GROUP BY Sdept;`
    * > 来源: P28

* **修改 `UPDATE`**

```sql
UPDATE <表名>
SET <列名> = <表达式> [, <列名> = <表达式>] ...
[WHERE <条件>];
```

* `SET` 子句指定需要更新的列及其赋予的新值或计算表达式，`WHERE` 子句限定哪些元组需要被修改。
* > 易错点：若省略 `WHERE` 子句，则该表中的所有元组都会被同时修改。
* 支持带子查询的批量修改：`UPDATE SC SET Grade = 0 WHERE Sno IN (SELECT Sno FROM Student WHERE Sdept = 'CS');`
* > 来源: P28

* **删除 `DELETE` 与 `TRUNCATE` 辨析**
    * `DELETE FROM <表名> [WHERE <条件>];` 只删除数据元组，表的结构、约束、索引完整保留；省略 `WHERE` 则清空表中所有记录。
    * `DELETE` 是标准 DML，逐行删除并记录完整的事务回滚重做日志（Undo/Redo Log），支持事务回滚（Rollback）。
    * `TRUNCATE` 是 DDL 操作，通过直接释放物理数据页快速清空全表，不可精细过滤，执行极快但无法按行恢复。
    * > 来源: P28

### 11.2 完整性防御

* **DML 三级完整性校验流程**
    * 实体完整性校验：主码是否冲突、主码属性是否被赋 `NULL`；违约立即抛错中断并回滚事务。
    * 参照完整性校验：`INSERT`/`UPDATE` 检查外码值在被参照主表中是否存在；`DELETE` 主表时按级联配置执行 `RESTRICT` 报错、`CASCADE` 连带删除或 `SET NULL` 置空。
    * 用户定义完整性校验：`CHECK` 表达式、`NOT NULL`、`UNIQUE` 是否全部通过。
    * 全部合规后写入重做日志并持久化生效。
    * > 来源: P28

* **参照完整性违约的识别**
    * > 易错点：删除主表某行时报 `integrity constraint ... violated - child record found`，说明从表（如 `SC`）存在参照该学号的外码记录且外码采用 `RESTRICT` 策略，系统严禁删除主表对应行以维护参照完整性。
    * > 来源: P28

---

## 12. 视图机制

> **一句话主旨**：给出视图的定义与安全检查、视图消解算法、可更新性边界及其工程价值。

### 12.1 视图定义与安全

* **视图（View）**
    * > 定义：从一个或几个基本表（或其他视图）导出的虚拟表（Virtual Table）。
    * 数据库中只存放视图定义（保存在数据字典中的一段 `SELECT` 语句文本），不存放视图对应的数据记录；基表数据变动后经视图查询自动同步动态展现。
    * 基本表是物理磁盘上有真实数据文件落盘、占有实际存储空间的“实表”；视图是外模式在 SQL 标准中最直接的工程承载机制。
    * > 来源: P29

* **创建与删除视图**

```sql
CREATE VIEW <视图名> [(<列名> [, <列名>] ...)]
AS <子查询>
[WITH CHECK OPTION];
```

* 必须显式定义视图列名列表的四种场景：目标列是聚集函数或算术表达式；多表连接出现同名属性列；为列启用更符合用户习惯的别名；需要对真实列名脱敏。
* `DROP VIEW <视图名> [CASCADE];` 只从数据字典注销视图定义文本，底层基本表及其业务数据丝毫不受影响。
* > 来源: P29

* **`WITH CHECK OPTION`**
    * > 定义：创建视图时追加该子句后，通过该视图执行 `INSERT`、`UPDATE`、`DELETE` 时 DBMS 会强制自动追加视图定义中的子查询条件。
    * 例：对含 `WHERE Sdept = 'CS'` 且带该子句的视图执行 `SET Sdept = 'MA'`，新值违反准入约束会被立即拦截并抛错；插入 `Sdept = 'IS'` 的行同样被拦截。
    * > 来源: P29

### 12.2 视图消解与可更新性

* **视图消解（View Resolution）**
    * 步骤 1：从系统数据字典提取该视图定义的内层 `SELECT` 语句。
    * 步骤 2：将用户针对视图编写的外层查询与视图本身的子查询进行语义合并与条件拼接。
    * 步骤 3：生成直接针对底层物理基本表的等价综合查询，交给查询优化器编译执行。
    * 例：`SELECT Sno, Sage FROM V_CS_Student WHERE Sage < 20;` 消解为 `SELECT Sno, Sage FROM Student WHERE Sdept = 'CS' AND Sage < 20;`
    * > 来源: P29

* **可更新视图（Updatable Views）**
    * 理论上只有行列子集视图（Row-and-column Subset View）确定可更新：仅来自单一物理基本表，仅经简单选择（行筛选）与投影（列提取）产生，且完整包含原基本表主码。
    * 若视图数据由多行聚合而来或缺少原表主键，修改须转换为对基本表的“逆向映射”，在数学上是不可逆的多对一映射，无法确定修改底层哪一行。
    * > 来源: P29

* **绝对不可更新的视图类型**
    * 包含聚集函数（`SUM`、`AVG`、`MAX`、`MIN`、`COUNT`）的视图。
    * 包含 `GROUP BY` 或 `HAVING` 子句的视图；包含 `DISTINCT` 去重修饰符的视图；包含 `UNION` 等集合操作符的视图。
    * 由多表复杂连接导出且缺少关键外码参照约束的混合视图。
    * > 来源: P29

### 12.3 视图的工程价值

* **查询简化与逻辑独立性**
    * 将冗长的跨多表外连接与嵌套逻辑固化为视图，终端开发者只需 `SELECT * FROM View` 即可使用。
    * 底层基本表因性能优化拆分为两张子表时，可建立同名视图将两张子表联合，既有应用程序保持调用视图，业务代码无需修改。
    * > 来源: P29

* **多视角与安全隔离**
    * 财务部、人事部、市场部可针对同一张底层员工大表，建立各自视角契合的专属视图，各取所需。
    * 在视图中排除敏感薪资、身份证字段实现列级防护，并在 `WHERE` 中限定部门实现行级防护，配合权限系统只授予视图访问权，从物理上杜绝数据越权泄露。
    * > 来源: P29
