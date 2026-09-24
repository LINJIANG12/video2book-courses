# SQL 语言：数据定义、查询、更新与视图

## SQL 语言定位

### SQL（Structured Query Language）

* **定义**：SQL 是通用的、功能极强的关系数据库语言，用来对现实世界中的数据进行定义、操作和控制。
* **标准属性**：SQL 是关系数据库领域的行业标准。统一标准使产业链可以围绕同一规范集中生产，降低产品成本，推动应用普及和技术创新。
* **理论对应**：数据定义、数据操纵和数据控制分别由 DDL、DML、DCL 承担，关系代数是这些功能背后的理论基础。
* **综合统一**：SQL 覆盖数据库生命周期中的主要活动，包括定义与修改框架、装入数据、增删改查、重构、维护、安全性、完整性和并发访问等。
* **高度非过程化**：使用者说明“要做什么”，不必说明数据沿什么存取路径被找到；数据库管理系统负责把要求转换为关系运算并完成处理。
* **面向集合**：SQL 的操作对象是表，一条语句可以处理一批记录，不必像过程化语言那样逐行定义循环。
* **与高级语言互补**：高级语言擅长描述执行过程，SQL 擅长处理集合式的大规模数据；SQL 可以嵌入 C、Java 或 C++，由程序传入条件并接收结果。
* **作用范围**：SQL 可以作用于基本表和视图，并能连接上层应用程序；存储文件由操作系统和数据库管理系统负责，不属于 SQL 直接管理的范围。

### SQL 的三类功能

* **数据定义语言（Data Definition Language，DDL）**
    * 负责数据库框架的定义、修改和删除，回答“有几张表、每张表长什么样、表与表如何连接”。
* **数据操纵语言（Data Manipulation Language，DML）**
    * 负责数据的增加、删除、修改和查询，简称增、删、改、查。
* **数据控制语言（Data Control Language，DCL）**
    * 负责访问权限以及完整性、安全性等控制；`GRANT` 授予权限，`REVOKE` 收回已经授予的权限。

### 九个核心动词

* **DDL 动词**
    * `CREATE` 创建模式、基本表、视图或索引。
    * `DROP` 删除模式、基本表、视图或索引。
    * `ALTER` 修改基本表的列或完整性约束；索引通常只允许改名，模式和视图通常不提供一般意义上的直接结构修改。
* **DML 动词**
    * `INSERT` 增加数据，`UPDATE` 修改数据，`DELETE` 删除数据，`SELECT` 查询数据。
* **DCL 动词**
    * `GRANT` 把某项权限授予指定用户或用户类别，`REVOKE` 收回已经授予的权限。

## 数据定义对象

### 模式（Schema）

* **定义**：模式是为某个用户建立的数据库整体命名空间，用来容纳基本表、视图、索引和授权定义。
* **创建形式**：`CREATE SCHEMA S_T AUTHORIZATION WANG;` 为用户 `WANG` 建立名为 `S_T` 的模式。
* **省略模式名**：`CREATE SCHEMA AUTHORIZATION WANG;` 时，模式名默认采用用户名。
* **定义的实质**：创建模式不是向模式中填入数据，而是先开出数据库命名空间，为后续对象提供归属。
* **删除形式**：`DROP SCHEMA S_T RESTRICT;` 或 `DROP SCHEMA S_T CASCADE;`。
* **删除边界**：`RESTRICT` 先检查模式中是否已有依赖对象，存在时拒绝删除；`CASCADE` 将模式内部依赖对象一起删除。

### 基本表（Table）

* **定义**：基本表是数据库框架中独立存在、实际存放数据的基本数据表。
* **创建要素**：`CREATE TABLE` 的基本结构包括表名、列名及数据类型、列级完整性约束和表级完整性约束。
* **列级约束**：针对某一列，例如该列不能为空或取值必须唯一。
* **表级约束**：针对完整表，例如哪些列构成主码、哪些列是外码以及外码引用哪张表的哪一列。
* **数据类型**：字符串长度应由业务需要决定；即使学号只包含数字，也可以使用字符串类型，长度过大会消耗额外存储资源。
* **主码含义**：`PRIMARY KEY` 从结果上等价于“非空且唯一”；在已有数据上建立主码时，现有数据也必须满足该要求。
* **示例结构**：

```sql
CREATE TABLE Student (
    Sno   CHAR(10) PRIMARY KEY,
    Sname CHAR(20) UNIQUE,
    Ssex  CHAR(2),
    Sage  INT,
    Sdept CHAR(20)
);
```

### 主码、外码与联合主码

* **单列主码**
    * `Sno CHAR(10) PRIMARY KEY` 把主码约束直接写在学号列定义后面，作用是让学号成为唯一标识。
* **联合主码（Composite Key）**
    * 多个列共同构成主码，约束条件作用于这些列的组合，而不是分别作用于每一列。
    * 选课表中同一学生可以选多门课，同一课程也可以被多个学生选择，因此学号与课程号的组合才构成主码；同一学生不能重复选择同一课程。
    * 正确写法是把多列放入同一个 `PRIMARY KEY` 约束中：

```sql
CREATE TABLE SC (
    Sno   CHAR(10),
    Cno   CHAR(4),
    Grade INT,
    PRIMARY KEY (Sno, Cno),
    FOREIGN KEY (Sno) REFERENCES Student(Sno),
    FOREIGN KEY (Cno) REFERENCES Course(Cno)
);
```

* **外码（Foreign Key）**
    * 外码声明要说明三件事：哪一列是外码、引用哪张表、引用对方哪一列。
    * 课程表的先修课程号可以引用同一张表的课程号：

```sql
CREATE TABLE Course (
    Cno     CHAR(4) PRIMARY KEY,
    Cname   CHAR(20),
    Cpno    CHAR(4),
    Ccredit INT,
    FOREIGN KEY (Cpno) REFERENCES Course(Cno)
);
```

* **依赖关系**
    * `SC.Sno` 来自 `Student.Sno`，`SC.Cno` 来自 `Course.Cno`，两列分别承担外码角色。

### 模式与表的组织方式

* **创建表时明确模式**
    * 在 `CREATE TABLE` 中明确写出所属模式，编写时即使发现原计划漏表，也可以把新增表归入指定模式，方式灵活。
* **创建模式时同时创建表**
    * 把表的定义紧接在创建模式的语句中，归属关系清楚，适合一开始已经确定完整结构的情况；缺点是必须预先想清楚所有表。
* **通过搜索路径确定模式**
    * 使用类似 `SET search_path TO` 的方式连接搜索路径，再执行 `CREATE TABLE`，由当前搜索路径确定表的归属。
* **使用边界**
    * 掌握前两种方式已经足以应对大多数需要明确表达数据库组织结构的场景。

### 修改基本表

* **修改对象**
    * `ALTER TABLE` 必须先指出被修改的表，再描述修改内容；可以新增列、增加或删除表级完整性约束、删除列、修改列名或修改数据类型。
* **新增列**
    * 新增列仍要明确列名、数据类型和约束条件：

```sql
ALTER TABLE Student
ADD Enrolldate DATE;
```

    * 无论原表是否有数据，新增加的列一开始都是空值，原有数据不会自动补出入学时间。
* **修改数据类型**
    * 只要说明目标列要改成什么类型，不必另写过程化的数据搬运逻辑：

```sql
ALTER TABLE Student
ALTER COLUMN Sage INT;
```

* **增加与删除约束**
    * 增加唯一约束：

```sql
ALTER TABLE Course
ADD UNIQUE (Cname);
```

    * 删除表级约束时使用实际约束名：

```sql
ALTER TABLE Course
DROP CONSTRAINT 约束名;
```

    * `约束名` 是数据库中该约束的实际标识，不是字面量，也不是所有数据库系统都接受的固定写法。
* **删除列**
    * 删除列同样要面对 `CASCADE` 与 `RESTRICT` 的依赖处理选择。

### 删除基本表

* **删除形式**：`DROP TABLE Student;` 用于删除基本表本身。
* **限制性删除**
    * `DROP TABLE Student RESTRICT;` 如果表上已有视图、索引等依赖对象，系统会拒绝删除并提示先清理依赖。
* **级联删除**
    * `DROP TABLE Student CASCADE;` 将表及相关依赖对象一起删除，系统可能同时给出说明信息。
* **共同含义**
    * `CASCADE` 与 `RESTRICT` 表示删除时对依赖对象采取什么策略，不仅用于删除模式，也用于删除基本表和视图。
* **系统边界**
    * 不同数据库管理软件对两种选项的具体处理可能有差异，使用时应查询对应软件的手册。

### 索引（Index）

* **定义与作用**
    * 索引是为加快查询建立的辅助结构；给定学号后，系统可以沿索引定位记录，而不必逐行检查整张表。
* **常见类型**
    * 常见索引包括顺序文件索引、B+ 树索引、散列或哈希索引以及位图索引。
    * B+ 树是二叉树的一种特殊形式，同层节点左侧值小于当前节点、右侧值大于当前节点，查找小于当前值时向左侧继续，并具有动态平衡特点。
    * 哈希索引用哈希函数把数据分散到不同的“堆”中，查询时先定位相应的堆，通常查找速度较快。
* **可选属性**
    * `UNIQUE` 表示建立唯一索引；`CLUSTER` 表示建立聚簇索引；二者不写时使用默认索引形式。
    * `ASC` 表示升序，`DESC` 表示降序；不写排序方向时默认升序。
* **建立形式**

```sql
CREATE [UNIQUE] [CLUSTER] INDEX 索引名
ON 表名 (列名 [ASC | DESC], ...);
```

    * 索引可以建在一列或多列上，多列之间用逗号隔开，排序方向可以逐列指定：

```sql
CREATE UNIQUE INDEX SC_Sno
    ON SC (Sno ASC, Cno DESC);

CREATE UNIQUE INDEX Student_Sno
    ON Student (Sno ASC);

CREATE UNIQUE INDEX Course_Cno
    ON Course (Cno ASC);
```

* **维护与使用**
    * 数据库管理员和建表者可以建立索引，数据库管理员与数据库管理系统负责维护，索引建立后大家都可以使用。
* **修改与删除**
    * 索引通常只允许改名：

```sql
ALTER INDEX 索引名 RENAME TO 新索引名;
```

    * 删除索引用 `DROP INDEX 索引名;`。
* **命名约定**
    * 把索引命名为“表名_列名”只是便于阅读的约定，不是数据库强加的规定；其他不重复的名字也可以表达同样作用。

### 数据字典

* **定义**：数据字典把数据项、数据结构、数据流等内容组织成条目，并纳入系统开发文档。
* **作用**：它不是运行数据本身，而是代码旁边的说明，使没有参与原开发的人员也能理解 `Student(Sno)` 等对象、字段、数据和约束的含义。
* **关系**：数据字典与程序开发文档紧密相关，需要和程序一起交付。

## 单表查询

### 查询三要素

* **基本骨架**
    * 单表查询常见的基本形式是 `SELECT ... FROM ... WHERE ...`；`SELECT` 负责输出，`WHERE` 负责行条件，`FROM` 负责确定查询范围。
* **输入、输出与分析**
    * 拿到题目先问“输出是什么、输入条件是什么、分析范围在哪里”，分别对应 `SELECT`、`WHERE`、`FROM`。
    * 查询范围可以是表、视图或派生表；没有筛选条件时可以省略 `WHERE`。
* **职责分工**

| 语句 | 作用 | 具体范围 |
| :--- | :--- | :--- |
| `SELECT` | 指定输出 | 属性、计算表达式、常量、函数结果或输出别名 |
| `FROM` | 指定查询对象 | 表、视图或派生表 |
| `WHERE` | 筛选查询中的行 | 当前查询范围内的记录 |
| `GROUP BY` | 按指定列分组 | 按分组键形成组 |
| `HAVING` | 筛选分组结果 | 分组和聚集计算后的组 |
| `ORDER BY` | 排序最终结果 | 指定排序列和方向 |

### SELECT 的输出形式

* **属性列**
    * 直接列出属性即可输出对应列；没有筛选条件时，`FROM` 后不写 `WHERE`：

```sql
SELECT Sno, Sname
FROM Student;
```

* **全体属性**
    * `SELECT *` 表示输出当前查询范围内的全部属性；配合 `WHERE` 时表示输出所有满足条件的完整记录：

```sql
SELECT *
FROM Student;
```

* **计算表达式**
    * 输出表达式不要求原表中存在对应属性，例如根据当前年份和年龄计算出生年份：

```sql
SELECT Sname, 2022 - Sage
FROM Student;
```

* **常量与函数**
    * `SELECT` 可以直接输出常量，也可以调用函数，例如用 `LOWER(Sdept)` 将所在系转成小写。
* **输出别名**
    * `AS` 后面给出的名称会成为结果表头，可以修饰属性、表达式和函数结果：

```sql
SELECT Sname AS NAME,
       2014 - Sage AS BIRTH,
       LOWER(Sdept) AS "Dept Name"
FROM Student;
```

* **字符串常量**
    * 不带引号的 `Sname`、`Sdept` 是属性名，带单引号的 `'Year of Birth'` 是固定常量；字符串常量必须放在单引号中。

### WHERE 与结果去重

* **行筛选**
    * `WHERE` 负责选择表中的若干行，条件必须针对当前查询范围内的数据。
* **重复的来源**
    * 选课表中一个学生可能选多门课，直接查询 `SC.Sno` 会重复输出学号。
* **DISTINCT 与 ALL**
    * 默认情况下相当于 `ALL`，保留重复值；`DISTINCT` 去掉重复结果：

```sql
SELECT DISTINCT Sno
FROM SC;
```

* **条件类别**
    * 比较条件使用 `=、<、>、<=、>=`；范围条件使用 `BETWEEN ... AND ...` 或 `NOT BETWEEN ... AND ...`；集合条件使用 `IN` 或 `NOT IN`；字符串匹配使用 `LIKE` 或 `NOT LIKE`；空值判断使用 `IS NULL` 或 `IS NOT NULL`；多重条件使用 `AND`、`OR`、`NOT`。
* **范围边界**
    * `BETWEEN` 的两个端点包含在内，例如 `Sage BETWEEN 20 AND 23` 包括 20 和 23。
* **集合条件**

```sql
SELECT Sname, Ssex
FROM Student
WHERE Sdept IN ('CS', 'MA', 'IS');
```

    * `IN` 表示字段值属于括号中列出的集合，也可以改写为若干个等号条件用 `OR` 连接。

### LIKE 字符串匹配

* **通配符**
    * `%` 代表任意长度的字符串，包括零个字符。
    * `_` 代表任意一个字符。
* **模式含义**

```text
A%B   → 以 A 开头、以 B 结尾的任意长度字符串
A_B   → 以 A 开头、以 B 结尾、长度为 3 的字符串
```

    * `A_B` 可以匹配 `ACB`、`ADB`、`AEB`，不能匹配 `AABB`，因为后者中间不是一个字符。
* **前缀、长度与位置匹配**
    * `Sname LIKE '刘%'` 表示以“刘”开头，后面长度任意。
    * `Sname LIKE '欧阳_'` 表示姓“欧阳”且全名恰好三个汉字；`Sname LIKE '欧阳%'` 只限定姓氏，不限定名字长度。
    * `Sname LIKE '_阳%'` 表示第一个字未知、第二个字固定为“阳”、第三个字开始长度未知。
* **取反匹配**
    * `NOT LIKE` 用于查找不满足模式的字符串，例如 `Sname NOT LIKE '刘%'` 查询不姓刘的同学。
* **转义通配符**
    * 当字符串本身含有 `_` 或 `%` 时，用 `ESCAPE` 指定转义字符，使特殊符号按字面意义匹配：

```sql
SELECT Cno, Ccredit
FROM Course
WHERE Cname LIKE 'DB\_Design' ESCAPE '\';
```

    * `DB\_Design` 中的 `\_` 表示字面下划线；不转义时 `_` 会匹配任意一个字符。
* **复杂模式**

```sql
SELECT *
FROM Course
WHERE Cname LIKE 'DB\_%i__' ESCAPE '\';
```

    * `DB\_` 是固定前缀，中间 `%` 表示长度未知，`i` 固定为倒数第三个字符，末尾两个 `_` 表示最后两位各为一个未知字符。

### 空值判断与多重条件

* **空值判断**
    * 判断空值使用 `IS NULL` 或 `IS NOT NULL`：

```sql
SELECT *
FROM Student
WHERE Sage IS NULL;
```

    * 非空年龄使用 `WHERE Sage IS NOT NULL`。
* **条件组合**
    * `AND`、`OR`、`NOT` 用于组合多个条件：

```sql
SELECT Sname
FROM Student
WHERE Sdept = 'CS'
  AND Sage < 20;
```

* **优先级**
    * `AND` 的优先级高于 `OR`；要改变组合关系时使用括号明确表达。

### ORDER BY 排序

* **排序字段与方向**
    * `ORDER BY` 决定结果按哪一列排序；`ASC` 表示升序，`DESC` 表示降序，不写方向时默认升序。
* **示例**

```sql
SELECT Sno, Grade
FROM SC
WHERE Cno = 3
ORDER BY Grade DESC;
```

* **多关键字排序**
    * 第一个排序关键字决定主要顺序，后续关键字只在前面排序值相同时继续比较：

```sql
SELECT *
FROM Student
ORDER BY Sdept ASC, Sage DESC;
```

* **空值排序**
    * 空值参与排序时显示次序由具体系统决定，不能把它当作普通数值比较；日常可用“空值跟大头”记忆，但实际仍应按系统规则处理。

## 聚集、分组与组筛选

### 聚集函数

* **定义与作用**
    * 聚集函数用于统计，常见功能包括统计行数、统计值的个数、统计列数、求最大值、最小值和平均值。
* **整体统计**

```sql
SELECT COUNT(*)
FROM Student;
```

    * `COUNT(*)` 统计所有行，因此得到学生总人数。
* **去重统计**

```sql
SELECT COUNT(DISTINCT Sno)
FROM SC;
```

    * 一个学生可能选多门课，直接 `COUNT(Sno)` 会重复计算选课记录；`COUNT(DISTINCT Sno)` 先对学号去重，再计数。
* **平均、最大和最小**

```sql
SELECT AVG(Grade)
FROM SC
WHERE Cno = 1;

SELECT MAX(Grade)
FROM SC
WHERE Cno = 1;

SELECT MIN(Grade)
FROM SC
WHERE Cno = 1;
```

    * `WHERE Cno = 1` 先留下 1 号课程记录，聚集函数再对留下的结果计算。
* **作用范围**
    * 没有 `GROUP BY` 时，聚集函数作用于整个查询结果；使用 `GROUP BY` 后，聚集函数分别作用于每一个组。

### GROUP BY 分组

* **定义**
    * `GROUP BY` 按指定列的值分组，通常与聚集函数配合使用。
* **按课程统计**

```sql
SELECT Cno, COUNT(*)
FROM SC
GROUP BY Cno;
```

    * 过程是先按 `Cno` 分组，再在每个组内用 `COUNT(*)` 统计选课记录数，最后输出课程号和人数。
* **按学生统计**

```sql
SELECT Sno
FROM SC
GROUP BY Sno
HAVING COUNT(*) > 3;
```

    * 先按学号分组，再在每组内统计选课记录数，最后只保留记录数超过 3 的组。
* **分组与聚集的关系**
    * 学号位于各组的分组键上，统计结果来自每个组内部，因此满足条件的组可以同时输出学号和统计值。

### HAVING 与 WHERE

* **作用对象**
    * `WHERE` 筛选分组前的普通行，`HAVING` 筛选分组和聚集计算后的组。
* **平均成绩筛选**

```sql
SELECT Sno, AVG(Grade)
FROM SC
GROUP BY Sno
HAVING AVG(Grade) > 90;
```

    * 普通行尚未分组和计算时，不能先用 `WHERE AVG(Grade) > 90` 表达分组后的平均分条件；平均值由一组成绩计算出来，应在分组之后筛选。
* **处理顺序**

```text
原始数据
   ↓
WHERE：筛选参与统计的行
   ↓
GROUP BY：按指定属性分组
   ↓
聚集函数：在每个组内计算
   ↓
HAVING：筛选满足条件的组
   ↓
输出分组属性与统计结果
```

* **记忆规则**
    * 需要保留哪些记录放 `WHERE`，需要保留哪些组放 `HAVING`；二者都表示筛选，但筛选阶段不同。

## 多表查询的三条路径

### 路径选择

* **三种方法**
    * 多表查询可以走连接查询、嵌套查询和集合查询三条路径。
* **表达能力**
    * 三种方法的表达能力大体相当，同一道题常常都能完成。
* **选择依据**
    * 区别主要在哪种写法更顺手以及系统实际处理效率不同；拿到题目后选择最直接、最简单的路径，不必纠结谁对谁错。

### 连接查询

* **基本机理**
    * 连接查询用于同时涉及两个以上关系的多表查询。
    * 逻辑上先做笛卡尔积，把所有组合列出来，再按连接条件筛选，把不相等的组合删掉；SQL 写下等号连接时暗含这一过程。
* **FROM 与 WHERE 的变化**
    * `SELECT` 仍负责输出；`FROM` 写出涉及的各个表；`WHERE` 写连接条件和额外筛选条件。
* **两表连接**

```sql
SELECT *
FROM Student, SC
WHERE Student.Sno = SC.Sno;
```

* **连接条件加业务条件**

```sql
SELECT *
FROM Student, SC
WHERE Student.Sno = SC.Sno
  AND Student.Sdept = 'CS';
```

    * `WHERE` 中先写连接条件，再写筛选条件；涉及多个表时，即使没有额外业务筛选，也不能漏掉连接条件。
* **执行方式**
    * 数据库管理系统可能采用循环嵌套、排序合并或索引连接等策略完成连接，具体执行过程由系统决定，但连接条件必须由使用者写清楚。

### 自然连接与自身连接

* **自然连接**
    * 普通等值连接会输出参与连接的两个学号列；如果两个连接属性值完全一致，手动删掉其中一个重复列即可得到自然连接。
    * 关系代数可以直接使用 `NATURAL JOIN`；SQL 中没有对应简写，因此逐个写出需要的字段，避免重复输出。
* **自身连接**
    * 同一个表在查询中被引用两次时，先在思路上把它复制为两个表，再按普通多表查询处理。
    * 查询间接先修关系时，用两个角色连接同一张课程表：

```sql
SELECT First.Cno, Second.Cpno
FROM Course First, Course Second
WHERE First.Cpno = Second.Cno;
```

    * `First` 与 `Second` 是同一张 `Course` 表的两个角色，别名换成 `A`、`B` 也可以。

### 外连接与多表连接

* **外连接定义**
    * 保留一侧表的所有数据，另一侧没有对应记录时用空值占位，称为外连接。
* **三种保留规则**

| 类型 | 保留规则 |
| :--- | :--- |
| 左外连接 | 保留左表所有元组，右表无匹配时以空值占位 |
| 右外连接 | 保留右表所有元组，左表无匹配时以空值占位 |
| 整体外连接 | 两侧元组都保留 |

* **左外连接形式**

```sql
SELECT Student.Sno, Student.Sname, SC.Cno, SC.Grade
FROM Student
LEFT OUTER JOIN SC ON Student.Sno = SC.Sno;
```

    * 需要额外筛选条件时，在 `WHERE` 中继续写。
* **三表连接**

```sql
SELECT Student.Sno, Student.Sname, Course.Cname, SC.Grade
FROM Student, SC, Course
WHERE Student.Sno = SC.Sno
  AND SC.Cno = Course.Cno;
```

    * `Sno`、`Sname` 来自 `Student`，`Cno`、`Grade` 来自 `SC`，`Cname` 来自 `Course`；没有额外业务条件时仍必须写两个连接条件。

### 嵌套查询

* **定义与结构**
    * 完整的一个 `SELECT FROM WHERE` 查询块可以看成接收输入、产生输出的黑盒子；不同查询块套在一起就是嵌套查询。
    * 内层叫子查询，外层叫父查询或外层查询；子查询先执行并把结果交给父查询，父查询再利用结果继续处理。
* **层数**
    * 嵌套查询可以套两层、三层甚至更多层，没有固定层数。
* **排序位置**
    * 子查询不能使用 `ORDER BY`；它只是中间过程，最终排序应在最外层完成，中间层排序不会影响最终输出。
* **不相关子查询**
    * 子查询条件不依赖父查询时，可以独立完成，再把结果交给外层：

```sql
SELECT Sno, Sname, Sdept
FROM Student
WHERE Sdept IN (
    SELECT Sdept
    FROM Student
    WHERE Sname = '刘晨'
);
```

    * 不相关子查询不需要接收外层当前元组；它与连接查询表达同一任务时，通常不经过连接过程中的笛卡尔积和筛选。
* **由内向外安排**
    * 查询“选修了课程名为信息系统的学生学号和姓名”时，先在 `Course` 找课程号，再到 `SC` 找学生学号，最后到 `Student` 找姓名：

```sql
SELECT Sno, Sname
FROM Student
WHERE Sno IN (
    SELECT Sno
    FROM SC
    WHERE Cno IN (
        SELECT Cno
        FROM Course
        WHERE Cname = '信息系统'
    )
);
```

    * 原则是最先做的查询放在最内层，最后输出结果放在最外层；内层产生唯一值时，`=` 与 `IN` 在语义效果上可以互相替代。

### 相关子查询

* **定义**
    * 相关子查询的筛选条件依赖父查询；外层当前元组先传给内层，内层处理后返回结果，外层再继续判断。
* **按学生计算平均值**

```sql
SELECT Sno, Cno
FROM SC x
WHERE Grade > (
    SELECT AVG(Grade)
    FROM SC y
    WHERE y.Sno = x.Sno
);
```

    * `SC x` 只表示一个表和一个当前元组别名，不是把表复制成两个表；外层从当前行取 `x.Sno`，内层按它计算平均分。
* **执行过程**
    * 外层逐行取元组，把当前学号传入内层；内层按学号求 `AVG(Grade)`；外层比较当前成绩与平均分，满足条件才输出。
* **与自身连接的区别**
    * 自身连接中的 `Course First, Course Second` 是两个表角色；相关子查询中的 `SC x` 只有一个表，单独写别名表示当前元组。

### ANY、ALL 与 EXISTS

* **ANY 与 ALL**
    * `ANY` 表示“任意一个”，`ALL` 表示“全部”；应按整句集合语义理解，不要脱离上下文死记。
    * 小于 `ANY` 等价于小于集合最大值；小于 `ALL` 等价于小于集合最小值；大于 `ALL` 等价于大于集合最大值；大于等于 `ALL` 等价于大于等于集合最大值。
* **ANY 示例**

```sql
SELECT Sname, Sage
FROM Student
WHERE Sdept != 'CS'
  AND Sage < ANY (
      SELECT Sage
      FROM Student
      WHERE Sdept = 'CS'
  );
```

* **EXISTS**
    * `EXISTS` 是存在量词，表示“是否存在”；子查询不返回具体数据，只返回 `TRUE` 或 `FALSE`。
    * 内层结果非空返回真，空结果返回假；返回真时，作为筛选条件让最外层输出。
* **EXISTS 示例**

```sql
SELECT Sname
FROM Student
WHERE EXISTS (
    SELECT *
    FROM SC
    WHERE SC.Sno = Student.Sno
      AND SC.Cno = 1
);
```

    * `EXISTS` 的输出字段写什么不影响结果，直接写 `*` 更省事；`NOT EXISTS` 检查内层是否不存在符合条件的记录。

### 集合查询

* **集合运算基础**
    * 并、交、差与笛卡尔积属于传统集合运算，在 SQL 中常用 `UNION`、`INTERSECT`、`EXCEPT`。
* **UNION 与 UNION ALL**

```sql
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sdept = 'CS'
UNION
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sage <= 19;
```

    * 默认 `UNION` 自动去掉重复元组；不想去重时使用 `UNION ALL`。
* **INTERSECT**

```sql
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sdept = 'CS'
INTERSECT
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sage <= 19;
```

    * `INTERSECT` 对两个结果集取交集。
* **EXCEPT**

```sql
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sdept = 'CS'
EXCEPT
SELECT Sno, Sname, Sdept, Sage
FROM Student
WHERE Sage <= 19;
```

    * `EXCEPT` 从前一个结果集中减去后一个结果集。

### 派生表查询

* **定义**
    * 由基本表经过查询得到的中间结果表称为派生表；包含派生表的查询称为基于派生表的查询。
* **形成派生表**

```sql
SELECT Sno AS Avg_Sno, AVG(Grade) AS Avg_Grade
FROM SC
GROUP BY Sno;
```

    * 把查询放进括号并起别名 `Avg_SC` 后，派生表有两列：`Avg_Sno` 保存学号，`Avg_Grade` 保存每位学生平均成绩。
* **与派生表连接**

```sql
SELECT SC.Sno, SC.Cno
FROM SC
JOIN (
    SELECT Sno AS Avg_Sno, AVG(Grade) AS Avg_Grade
    FROM SC
    GROUP BY Sno
) AS Avg_SC
ON SC.Sno = Avg_SC.Avg_Sno
WHERE SC.Grade > Avg_SC.Avg_Grade;
```

* **与基本表的共同点**
    * 派生表和基本表在查询中都是表；涉及多个表时，同样必须建立连接。
* **适用场景**
    * 当查询中算出的中间结果稳定且可复用时，可以把它定义成派生表，再作为普通表使用。

## 数据更新

### INSERT 插入数据

* **两种方式**
    * `INSERT` 可以插入一个元组，也可以一次插入子查询结果产生的一组元组；关键区别是待插入的值是一行还是多行。
* **VALUES 插入一行**

```sql
INSERT INTO <表名> (<列名1>, <列名2>, ...)
VALUES (<值1>, <值2>, ...);
```

    * 目标列和值的个数、顺序、类型要对应；`VALUES` 后给出本次写入的常量。
    * 实际编写时建议明确写出列名，便于核对列含义，也便于定位错误。
* **缺少成绩时的插入**

```sql
INSERT INTO SC (Sno, Cno)
VALUES ('200215128', 1);
```

    * 只指定要填的列，未指定的成绩不参与本次插入。

```sql
INSERT INTO SC (Sno, Cno, Grade)
VALUES ('200215128', 1, NULL);
```

    * 明确把第三列写成 `NULL`，表示已经选课但暂时没有成绩。
* **INSERT 与 SELECT**

```sql
INSERT INTO DeptAge (Sdept, AvgAge)
SELECT Sdept, AVG(Sage)
FROM Student
GROUP BY Sdept;
```

    * `INSERT ... SELECT` 一次放入一组结果；子查询结果的列数和类型必须与目标表匹配，每列含义和类型都要对应。
    * `GROUP BY Sdept` 将学生按专业分组，`AVG` 分别作用于每个系；不写 `WHERE` 表示统计所有系。
* **完整性检查**
    * 插入前要检查实体完整性、参照完整性和用户定义完整性。
    * 实体完整性要求主属性不能取空值；参照完整性要求外码与被参照表主码保持对应；用户定义完整性是自行规定的取值范围。

### UPDATE 修改数据

* **三段式**
    * `UPDATE` 指定要改哪张表，`SET` 指定哪一列改成什么值或表达式，`WHERE` 给出筛选条件；只有满足条件的元组才会被修改。
* **修改一个元组**

```sql
UPDATE Student
SET Sage = 22
WHERE Sno = '201215121';
```

* **修改多行**
    * 没有 `WHERE` 时作用于所有行：

```sql
UPDATE Student
SET Sage = Sage + 1;
```

    * 这表示把每一行原有年龄都加 1，不是创建一个名为 `Sage+1` 的新列。
* **用子查询确定范围**

```sql
UPDATE SC
SET Grade = 0
WHERE Sno IN (
    SELECT Sno
    FROM Student
    WHERE Sdept = 'CS'
);
```

    * 子查询负责找出计算机科学系学生学号，外层更新负责修改范围内的选课成绩。
* **完整性约束**
    * 修改同样要检查完整性约束；插入时不能违反的约束，修改时也不能违反。

### DELETE 删除数据

* **基本形式**

```sql
DELETE FROM <表名>
WHERE <筛选条件>;
```

    * `DELETE FROM` 指定表，`WHERE` 指定要删除的行；没有 `WHERE` 就删除表中的全部数据。
* **删除多行**

```sql
DELETE FROM SC;
```

    * 适合清空选课表中的所有数据，但表本身仍然存在。
* **用子查询确定范围**

```sql
DELETE FROM SC
WHERE Sno IN (
    SELECT Sno
    FROM Student
    WHERE Sdept = 'CS'
);
```

    * 子查询先找出计算机科学系学生学号，再删除 `SC` 中这些学号的选课记录。
* **与 DROP TABLE 的区别**

| 操作 | 所属语言 | 影响对象 |
| :--- | :--- | :--- |
| `DELETE FROM SC` | DML | 删除表中的数据行，表本身还在 |
| `DROP TABLE SC` | DDL | 删除表本身及其基本结构 |

## 空值

### 空值的含义与来源

* **含义**
    * 空值表示不知道、不存在或者没有意义；它不是空字符串，也不是 0。
* **插入来源**
    * 插入选课记录时只提供学号和课程号、暂不提供成绩，或明确把成绩写成 `NULL`，都会产生空值。
* **更新来源**

```sql
UPDATE Student
SET Sdept = NULL
WHERE Sno = '201215210';
```

    * 更新也可以制造空值，例如学生暂时还没有专业接收时把所在系设为空。
* **本质**
    * 空值表示数据库目前不能给出该属性的值，不表示“还没有输入但数据库知道应该是多少”。

### IS NULL 与完整性约束

* **查找空值**
    * 空值判断使用 `IS NULL` 或 `IS NOT NULL`：

```sql
SELECT *
FROM Student
WHERE Sage IS NULL;

SELECT *
FROM Student
WHERE Sage IS NOT NULL;
```

* **查找整行缺失信息**

```sql
SELECT *
FROM Student
WHERE Sname IS NULL
   OR Ssex IS NULL
   OR Sage IS NULL
   OR Sdept IS NULL;
```

    * 只要一行有一个字段为空，`OR` 条件就成立，整行会被输出，便于定位漏填信息。
* **约束区别**
    * `NOT NULL` 明确表示该列不能为空；主属性受实体完整性约束，同样不能为空。
    * 唯一属性不能依靠空值证明“不重复”，因此实际数据中不要把应当唯一的属性留空。

### 三值逻辑

* **三种结果**
    * SQL 扩展为真、假、未知三值逻辑；空值与普通值比较时结果为 `UNKNOWN`。
* **算术与比较**
    * 空值参与加、减、乘、除时结果仍为空值；空值与另一个值作大小比较时结果不是普通真或假，而是未知。
* **逻辑表**

| X | Y | `X AND Y` | `X OR Y` | `NOT X` |
| :--- | :--- | :--- | :--- | :--- |
| TRUE | TRUE | TRUE | TRUE | FALSE |
| TRUE | UNKNOWN | UNKNOWN | TRUE | FALSE |
| UNKNOWN | TRUE | UNKNOWN | TRUE | UNKNOWN |

* **理解规则**
    * 真与未知做“并且”仍未知，因为不能只靠一边的真断定整体成立；真与未知做“或者”，只要一边为真，结果就是真；未知取反后仍是未知。

### 空值对查询的影响

* **不完整的“未及格”条件**

```sql
SELECT Sno
FROM SC
WHERE Cno = 1
  AND Grade < 60;
```

    * 这个条件不能包含 `NULL` 成绩，因为空值不能与 60 作普通大小比较，缺考同学不会出现在结果中。
* **同时处理未及格与缺考**

```sql
SELECT Sno
FROM SC
WHERE Cno = 1
  AND (Grade < 60 OR Grade IS NULL);
```

    * 题目中的“未及格”包括成绩小于 60 和没有成绩两种情况，必须把两个条件放在一起。
* **等价路径**
    * 还可以分别查询一号课中小于 60 分的学号和成绩为空的学号，再用集合运算合并。

## 视图

### 视图的定义与特点

* **定义**
    * 视图是建立在基本表之上的虚表；它保存的是查询定义，而不是查询结果的一份副本。
* **数据来源**
    * 视图的值通过查询从基本表映射出来，基本表数据变化后，查询视图得到的结果也随之变化。
* **语言归属**
    * 视图的定义和删除使用 DDL；对视图的查询、插入、修改和删除沿用 DML 思路。
* **对象关系**

```text
基本表（真正存放数据）
        │
        │ 查询、映射
        ▼
视图（只保存查询定义）
        │
        │ 插入 / 修改 / 删除
        ▼
基本表中的数据发生变化
```

### 建立视图

* **基本语法**

```sql
CREATE VIEW <视图名> [(<列名1>, <列名2>, ...)]
AS <子查询>
[WITH CHECK OPTION];
```

* **视图名与列名**
    * 视图名和列名可以自行指定，但应带有能说明对象含义的信息；列名可以全部省略，也可以全部指定。
* **单表视图**

```sql
CREATE VIEW CS_Student AS
SELECT *
FROM Student
WHERE Sdept = 'CS';
```

    * 该视图每次沿查询定义到 `Student` 中取值，不重新保存一份计算机系学生数据。
* **指定视图列**

```sql
CREATE VIEW IS_Student (Sno, Sname, Sage)
AS
SELECT Sno, Sname, Sage
FROM Student
WHERE Sdept = 'IS'
WITH CHECK OPTION;
```

* **WITH CHECK OPTION**
    * 对视图进行插入或修改时，数据库检查修改后的内容是否仍满足建立视图时的条件。
    * 没有写 `WITH CHECK OPTION` 时，视图不替你执行这项检查；不要通过该视图把条件之外的值带进来。
* **定义保存方式**
    * 执行 `CREATE VIEW` 时，系统只是把视图定义放进数据字典，不会当场执行其中的 `SELECT` 去复制结果；真正查询视图时才按定义取数据。

### 视图的类型与组织

* **行列子集视图**
    * 视图只来自一个基本表，在该表基础上去掉部分行、部分列，同时保留主码，称为行列子集视图。
    * 计算机系学生视图和信息系学生视图都符合这一特征；保留主码是为了让视图中的记录仍能对应确定实体。
* **多个基本表组成的视图**

```sql
CREATE VIEW IS_S1 (Sno, Sname, Grade)
AS
SELECT Student.Sno, Student.Sname, SC.Grade
FROM Student, SC
WHERE Student.Sdept = 'IS'
  AND SC.Cno = 1
  AND Student.Sno = SC.Sno;
```

    * 学号、姓名来自 `Student`，成绩来自 `SC`；连接条件 `Student.Sno = SC.Sno` 不能遗漏。
* **视图上的视图**

```sql
CREATE VIEW IS_S2 (Sno, Sname, Grade)
AS
SELECT Sno, Sname, Grade
FROM IS_S1
WHERE Grade > 90;
```

    * 视图可以像表一样使用，也可以在已有视图基础上再建立视图。
* **表达式、分组与聚集**
    * 视图定义中可以包含表达式、分组和聚集函数：

```sql
CREATE VIEW StudentAvg (Sno, AvgGrade) AS
SELECT Sno, AVG(Grade)
FROM SC
GROUP BY Sno;
```

    * `StudentAvg` 每个学生一行，分组后 `AVG(Grade)` 分别计算每个学生所选课程的平均成绩。
* **列名显式化**
    * 视图定义中尽量明确列出需要的列，不宜依赖 `SELECT *`；基本表以后增加列时，星号输出列数会变化，可能与原有视图列结构接不上。

### 删除视图

* **基本形式**

```sql
DROP VIEW <视图名>;
```

* **依赖处理**
    * 删除视图时涉及 `CASCADE` 和 `RESTRICT`：`CASCADE` 将依赖视图一起删除，`RESTRICT` 在存在依赖对象时拒绝删除并提示先处理。
* **默认边界**
    * 默认是 `RESTRICT`；若 `IS_S2` 依赖 `IS_S1`，直接删除 `IS_S1` 会被拒绝。
* **级联示例**

```sql
DROP VIEW IS_S1 CASCADE;
```

    * 使用 `CASCADE` 后，依赖 `IS_S1` 的视图也随之删除。

### 查询视图

* **使用方式**
    * 视图在外表看来就是表，因此查询视图的语法与查询基本表基本相同：

```sql
SELECT Sno, Sname, Sage
FROM IS_Student
WHERE Sage < 20;
```

* **视图消解**
    * 系统内部可以用视图消解法把视图转换为基本表上的运算，并补上定义中的条件：

```sql
SELECT Sno, Sname, Sage
FROM Student
WHERE Sage < 20
  AND Sdept = 'IS';
```

    * 转换前后输出结果不变；学习时重点是理解视图可以像表一样查询，不必深入内部实现。
* **按平均值筛选**

```sql
SELECT Sno, AvgGrade
FROM StudentAvg
WHERE AvgGrade >= 90;
```

* **直接按分组筛选**

```sql
SELECT Sno, AVG(Grade)
FROM SC
GROUP BY Sno
HAVING AVG(Grade) >= 90;
```

    * 平均成绩由一组成绩计算得到，筛选它应使用 `HAVING`，而不是把它作为分组前的普通行条件。

### 更新视图

* **基本语法**
    * 对视图执行 `INSERT ... VALUES`、`UPDATE ... SET ... WHERE`、`DELETE FROM ... WHERE` 时，写法仍与基本表相同，系统会把操作转换到视图依据的基本表上。
* **可更新条件**
    * 行列子集视图可以进行数据的增加、删除和修改。
    * 通过多个基本表组合、经过计算、分组或聚集得到的视图，通常不能直接更新。
* **不能直接更新的原因**
    * 平均分对应一组课程成绩，直接改变显示出来的平均值，不能唯一确定应修改哪些底层记录；多表组合和复杂运算结果也不一定能反向拆回原始数据。
* **结构与数据的区别**
    * 视图“不能修改结构”和视图“不能更新数据”不是一回事：结构映射改变时通常删除旧视图再重建；数据是否可更新取决于视图是否保留可更新的行列子集。
* **系统边界**
    * 不同系统对视图更新的限制可能有细节差异，应按具体系统规则判断。

### 视图的作用

* **简化操作**
    * 把平均分、最高分、最低分或某个班的方差等常用结果定义为视图，查询时不必每次重新写长查询。
* **多角度观察**
    * 同一个基本表可以按姓名与年龄、性别与年龄、姓名与所在专业等不同角度建立多个视图，数据仍只有一份。
* **逻辑独立性**
    * 基本表或原始数据发生较大变化时，原有视图仍可作为理解原来数据结构的参照，因为它保存查询定义和关系，不复制每个字节。
* **保护机密数据**
    * 只把用户需要的字段放入视图，不相关的数据不会出现在其可见范围内，比口头提醒“不要看”更可靠。
* **清楚表达查询**
    * 信息系学生、选修指定课程的学生、成绩超过某分数的学生等范围可以先定义成清晰视图，再在这些范围上继续查询。

## SQL 语句骨架

### SELECT 查询块

* **完整骨架**

```text
SELECT [DISTINCT] 目标列
FROM 基本表或派生表
WHERE 筛选条件
GROUP BY 分组列
HAVING 分组后的筛选条件
ORDER BY 排序列
```

* **各部分职责**
    * `SELECT` 处理输出，可以包含 `DISTINCT`、聚集函数和表达式。
    * `FROM` 处理输入，可以来自基本表、视图或派生表。
    * `WHERE` 处理分组前的行筛选条件。
    * `GROUP BY` 与 `HAVING` 处理分组及分组后的筛选。
    * `ORDER BY` 处理最终排序，不写方向时默认升序。
* **DML 速查**

| 操作 | 语句骨架 | 作用 |
| :--- | :--- | :--- |
| 查询 | `SELECT ... FROM ...` | 查询数据 |
| 插入 | `INSERT ... VALUES` 或 `INSERT ... SELECT` | 插入一行或一组结果 |
| 修改 | `UPDATE ... SET ... WHERE` | 修改满足条件的行 |
| 删除 | `DELETE FROM ... WHERE` | 删除满足条件的数据行 |

### 视图与 DML 的衔接

* **数据定义**
    * 用 DDL 创建模式、基本表、索引和视图。
* **数据操作**
    * 用 DML 对基本表和视图执行查询、插入、修改、删除，并处理空值。
* **空值边界**
    * 空值不能作为普通数比较；查询缺失数据使用 `IS NULL`，涉及“未及格”时要把空值条件与小于条件合并。
* **视图收束**
    * 视图把查询定义固定成表的样子，既能简化查询，也能提供不同观察角度；结构改变通常删除重建，复杂结果视图通常不能直接更新。
