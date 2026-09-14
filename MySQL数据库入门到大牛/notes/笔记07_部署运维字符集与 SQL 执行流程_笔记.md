# 部署运维、字符集与 SQL 执行流程

> 整理自《MySQL数据库入门到大牛，mysql安装到优化，百科全书级，全网天花板》P96~P112 单集精读长文
> 渲染支持：Typora / VS Code Markmap / XMind 一键脑图

```text
部署运维、字符集与 SQL 执行流程 (P96-P112)
├── 环境规划与虚拟机克隆 (P97)             ──── 双版本节点规划、克隆模式选型、克隆后标识重构流水线
├── 安装介质与卸载闭环 (P97-P99)           ──── FHS 投递规范、RPM 抽取策略、四阶段无残留卸载
├── 双版本安装与初始化 (P99)               ──── 依赖链顺序、MariaDB 冲突、initialize 与临时密码
├── 远程连接与接入链路 (P100)              ──── 四层故障排查、host 权限、认证插件降级、密码强度
├── 字符集继承与比较规则 (P101-P102)       ──── 四级继承、两级转变更、后缀命名规范
├── 请求响应编解码链路 (P102)              ──── client/connection/results 三变量、八步转码
├── 大小写规范与 sql_mode (P103)           ──── lower_case_table_names 三值、严格模式、双管齐下
├── 数据目录与物理文件映射 (P104)          ──── 四大系统库、InnoDB 与 MyISAM 文件形态演进
├── 用户生命周期管理 (P105)                ──── 'user'@'host' 二元组、USAGE 沙箱、DDL 与 DML 选型
├── 密码存储与安全策略 (P106)              ──── authentication_string、过期与重用、8.0 语法废弃
├── 权限体系与访问控制 (P107)              ──── 62 项权限、权限四表、双阶段漏斗鉴权
├── 角色抽象与激活陷阱 (P108)              ──── 角色容器、未激活态、默认角色、强制角色
├── 配置体系与逻辑架构 (P109)              ──── 选项组匹配、变量持久化、连接层/服务层/引擎层
├── SQL 执行全生命周期 (P110-P111)         ──── 四关流水线、查询缓存存废、Profiling 实测
└── 缓冲池与多实例拆分 (P112)              ──── Oracle 软硬解析、16KB 数据页、脏页刷盘、实例锁解耦
```

---

## 1. 环境规划与虚拟机克隆

> **一句话主旨**：确定双版本节点的资源规划，掌握完整克隆的选型理由与克隆后四处系统标识的重构顺序。

* **双版本隔离规划**
    * 高级篇采取「以 8.0 为主、兼顾 5.7」的双版本对照教学；8.0 是官方长期支持与未来发展的主力版本，5.7 是各企业生产环境存量占比极高的经典稳定版本。
    * 8.0 与 5.7 绝不能安装在同一台虚拟机：会面临 `3306` 默认端口冲突，并导致服务管理脚本（`mysqld`）、全局配置文件（`/etc/my.cnf`）、默认数据目录（`/var/lib/mysql`）以及动态链接库的混杂污染。
    * 节点一：IP `192.168.1.150`，主机名 `atguigu05`，安装 MySQL 8.0；节点二：IP `192.168.1.160`，主机名 `atguigu06`，安装 MySQL 5.7；两台节点的部署目录均为 `/opt`。
    * > 来源: P97

* **完整克隆（Full Clone）与链接克隆（Linked Clone）**
    * 完整克隆：将母机虚拟磁盘（VMDK）做完整独立的物理拷贝，克隆后彻底解耦，母机的删除、移动或受损均不影响子机；链接克隆：仅生成指向母机快照的差异磁盘（Delta VMDK），强依赖母机。
    * 数据库服务器这类高 I/O 负载、需要独立修改底层内核与网络配置的虚拟机必须选「创建完整克隆」，链接克隆仅适合快速临时测试。
    * 冷克隆前置条件：母机必须处于完全关机（Power Off）状态，运行中或挂起状态下克隆功能将受限或无法创建冷克隆镜像。
    * > 来源: P97

| 对比维度 | 链接克隆 | 完整克隆 |
| :--- | :--- | :--- |
| 磁盘物理文件 | 仅生成指向母机快照的差异磁盘 | 完整独立物理拷贝母机虚拟磁盘 |
| 创建耗时与容量 | 秒级完成，占用极小磁盘容量 | 取决于磁盘 I/O 速度，占用一份完整空间 |
| 与母机耦合性 | 强依赖母机，母机受损则子机立刻瘫痪 | 彻底解耦，完全独立运行 |
| 生产适用性 | 快速临时测试，严禁用于高负载与频繁改配置的系统 | 数据库服务器与集群节点搭建的标准选择 |

* **克隆后四处系统标识的重构**
    * 完整克隆是整盘物理数据的字节级复刻，新虚拟机完整继承母机的 MAC 地址、主机名、UUID 与静态 IP；未修改就同时通电会立刻引发 MAC 地址冲突、网络广播风暴与 IP 抢占断流。
    * 重构按严格流水线顺序执行：①关机状态下在 VMware 硬件面板重新生成 MAC 地址；②开机登录后修改 `/etc/hostname` 并 `reboot`；③编辑 `/etc/sysconfig/network-scripts/ifcfg-ens33` 同时改 `IPADDR` 与 `UUID`；④执行 `systemctl restart network` 使配置生效。
    * UUID 只需将字符串末尾几位字符随机修改（如末尾 `51c8` 改为 `51c9`），目的是防止网络管理器（NetworkManager）将克隆机网卡与母机网卡识别为同一硬件。
    * 修改主机名后当前 Shell 会话上下文不会自动更新主机名环境变量，必须 `reboot` 重启后提示符才变为 `[root@atguigu06 ~]#`。
    * > 易错点：若采用 DHCP 动态分配，启动克隆机前必须先将母机开机并锁定原有 IP 租约，否则 DHCP 服务器会把此前分配给母机的 IP 分配给克隆机，母机稍后开机只能拿到全新未知 IP，导致已保存的远程终端会话全线失效；采用静态绑定时 IP 硬编码在配置文件中，不存在租约争抢，可任意顺序开机。
    * > 来源: P97

* **网卡配置参数核查项**
    * `BOOTPROTO="static"` 协议必须为静态配置；`ONBOOT="yes"` 开机必须自动加载该网卡；`GATEWAY` 与 `DNS` 必须与宿主机虚拟交换机（VMnet8 NAT 模式）的网关设置保持一致。
    * 联通性验证：`ip addr show ens33` 应显示 `inet 192.168.1.160/24`，`ping -c 3 www.baidu.com` 应实现 0% 丢包率。
    * > 来源: P97

* **远程工具链与介质投递规范**
    * 生产服务器默认采用无 GUI 的最小化字符安装模式，运维交互只能通过远程加密连接完成：远程终端会话（CLI）基于 SSH 协议、端口 22，常用 Xshell；文件交互传输（SFTP）基于 SFTP 协议、端口 22，常用 Xftp。
    * 文件系统层次结构标准（FHS）目录职责：`/bin` 与 `/sbin` 存放基本用户命令与系统管理命令的二进制程序；`/etc` 存放系统级配置文件（如 `/etc/my.cnf`）；`/var` 存放动态变化数据（`/var/log`、`/var/lib/mysql`）；`/usr/local` 存放用户手动源码编译的第三方软件；`/opt` 是大型商业软件、第三方可选应用程序与离线安装包的标准存放目录。
    * > 易错点：Xftp 连接 Linux 后若中文目录或文件名出现方框或乱码问号，根因是 Windows 本地通常采用 GBK 编码而 Linux 默认使用 UTF-8 编码，传输层字符集不匹配导致解码紊乱；解法为右键会话属性 → 选项 → 勾选「使用 UTF-8 编码」后重连刷新，单纯点击刷新无法解决。
    * > 来源: P97

---

## 2. 安装介质、卸载闭环与双版本部署

> **一句话主旨**：从 RPM 抽取策略与四阶段无残留卸载讲到双版本依赖链安装、初始化与临时密码重置。

* **官方发行体系与安装包命名规范**
    * 官方四大发行版本：Community Server（社区版，开源免费，主流技术选型）、Enterprise Edition（企业版，商业收费，提供热备、高可用审计、数据脱敏及 30 天试用）、MySQL Cluster（NDB 集群版，开源免费）、MySQL Cluster CGE（电信级集群高级版，商业收费）。
    * 操作系统选型：下载页无 CentOS 专属选项，因 CentOS 完全基于 Red Hat Enterprise Linux（RHEL）源码二次构建、底层二进制 ABI 完全兼容，必须选 Red Hat Enterprise Linux / Oracle Linux；系统大版本选 7；架构选 `x86_64`。
    * 命名 `mysql-8.0.27-1.el7.x86_64.rpm-bundle.tar` 逐段含义：`8.0.27` 为软件版本号（主版本 8、次版本 0、小版本 27）；`1` 为当前版本发布修订次数；`el7` 代表 Enterprise Linux 7（CentOS 7）系统内核；`x86_64` 为 64 位架构；`rpm-bundle` 表示内含多个独立 RPM 组件的集合包；`tar` 为最终打包格式。
    * > 来源: P99

* **RPM 抽取策略与依赖安装顺序**
    * `.rpm-bundle.tar` 解压后会释放十几个 RPM 包，其中嵌入式引擎库、开发测试套件、C++ 头文件等对普通服务端运行时完全无用，生产部署绝无必要全部安装，只需抽取核心最小集合。
    * 依赖链为 `common → client-plugins → libs → client → server`，安装必须严格按此由底向上顺序执行。
    * `rpm -ivh` 参数含义：`-i` 安装（install）、`-v` 显示详细过程（verbose）、`-h` 打印进度条哈希符 `#`。
    * > 易错点：必须 `cd /opt` 进入安装包所在路径后再执行本地安装指令，在非安装包所在路径下执行，`rpm` 命令将无法定位目标文件。
    * > 来源: P99

| 软件版本 | 抽取安装包名称 | 组件功能与技术职责 |
| :--- | :--- | :--- |
| 8.0 / 5.7 共同 | `mysql-community-common` | 服务器与客户端共享的全局公共配置、字典编码与底层错误信息 |
| 8.0 独有 | `mysql-community-client-plugins` | 客户端连接所必需的认证、安全加密与通信插件包（8.0 新增必选） |
| 8.0 / 5.7 共同 | `mysql-community-libs` | MySQL 客户端共享的动态链接库（提供 libmysqlclient 运行时支持） |
| 8.0 / 5.7 共同 | `mysql-community-client` | 核心客户端命令行程序（提供 `mysql`、`mysqladmin`、`mysqldump` 等） |
| 8.0 / 5.7 共同 | `mysql-community-server` | 数据库核心服务端二进制守护进程 `mysqld` 及内置核心存储引擎 |

* **底层依赖与目录赋权**
    * `libaio`（Linux Asynchronous I/O Library）：InnoDB 重度依赖 Linux 内核的异步 I/O 接口进行脏页刷盘与日志追加，缺少该库则 `mysqld` 无法初始化 AIO 线程池。
    * `net-tools`：提供 `netstat`、`ifconfig`、`route` 等基础网络诊断命令，是排查 3306 端口监听与网络连接的必备工具。
    * 必须执行 `chmod -R 777 /tmp`：安装与初始化过程中系统会自动创建无登录权限的受限账户 `mysql`，RPM 安装脚本会以该账户身份在 `/tmp` 下读写临时文件，权限收紧会抛出静默的 `Permission denied`，导致服务初始化失败或套接字文件无法生成。
    * > 来源: P99

* **MariaDB 库冲突处理**
    * Oracle 收购 MySQL 后核心团队出走创建了完全开源的分支数据库 MariaDB；Red Hat 发布 CentOS 7 时把默认基础软件源中的底层库替换为 `mariadb-libs`，它占用了 `libmysqlclient.so` 的软链接命名空间，与官方原厂的 `mysql-community-libs` 互斥。
    * 冲突报错形态为 `error: Failed dependencies: mariadb-libs is obsoleted by mysql-community-libs-8.0.25-1.el7.x86_64`。
    * 解决命令为 `rpm -e mariadb-libs --nodeps`，`--nodeps` 用于强制解除依赖绑定以防止级联依赖阻断删除。
    * > 来源: P99

* **服务初始化与临时密码**
    * 初始化命令 `mysqld --initialize --user=mysql`：`--initialize` 会在 `/var/lib/mysql` 下创建 `mysql`、`information_schema`、`performance_schema`、`sys` 系统数据字典表空间，生成默认 SSL 密钥证书，并生成经过高强度加盐哈希的 root 用户初始临时密码。
    * `--user=mysql` 强制新生成的所有数据目录、文件及套接字的操作系统宿主用户与用户组均为 `mysql:mysql`；以 root 身份执行而未指定该参数会使底层文件属主变为 root，随后守护进程将因权限不足崩溃。
    * 提取临时密码：`cat /var/log/mysqld.log | grep "temporary password"`；密码形如 `h.p(k9wPq?wL`，共 12 位，包含字母、数字与特殊标点。
    * 服务生命周期管控：`systemctl start mysqld` 启动、`systemctl status mysqld` 查看（绿色 `active (running)` 代表已在 3306 端口监听）、`systemctl is-enabled mysqld` 检查自启状态、`systemctl enable mysqld` 启用开机自启。
    * 首次登录必须重置密码，否则任何查询都被阻断并抛出 `ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`；重置语句为 `ALTER USER 'root'@'localhost' IDENTIFIED BY 'abc123';`。
    * 重置成功后 `SHOW DATABASES;` 输出 4 个默认系统库：`information_schema`、`mysql`、`performance_schema`、`sys`。
    * > 来源: P99

* **四阶段无残留卸载闭环**
    * 卸载逻辑必须是「先看它是怎么没的，再看它是怎么来的」：初次安装常因漏装依赖、安装顺序颠倒、密码初始化错误或版本选型不当而中途夭折，若不会彻底卸载，残留的配置文件、历史数据目录、动态链接库与 RPM 包缓存会引发层出不穷的依赖冲突与服务启动失败。
    * 四阶段顺序不可颠倒：停服务 → 删程序 → 抹数据 → 清配置；完成后底层纯净度与从未装过 MySQL 的崭新机器完全一致，且**无需重启操作系统**即可直接进入全新版本安装。
    * > 易错点：严禁在数据库服务运行状态下强行卸载程序包或直接删除数据目录。服务活动时内存脏页尚未刷盘、表空间文件被内核加锁占用，强行删除会导致数据损坏、文件句柄泄漏，甚至产生不可杀死的 D 状态（不可中断睡眠）僵尸进程。
    * > 来源: P98

| 阶段步骤 | 执行命令 | 操作目的与技术机理 |
| :--- | :--- | :--- |
| 步骤 1：停服务 | `systemctl stop mysqld` | 终止后台 Daemon 进程，释放端口、内存与文件句柄 |
| 步骤 2：删程序 | `yum remove mysql-community-server` 等 | 级联卸载全部 RPM 安装包，直至 `rpm -qa` 检索为空 |
| 步骤 3：抹数据 | `rm -rf /var/lib/mysql /usr/lib64/mysql` | 递归强制抹除数据目录与依赖库，杜绝旧表空间污染 |
| 步骤 4：清配置 | `rm -rf /etc/my.cnf` | 删除全局参数配置文件，确保新安装重新生成干净配置 |

* **卸载过程中的检测与清理要点**
    * 版本核查：`mysqladmin --version` 或 `mysql --version`，输出形如 `mysqladmin Ver 8.0.25 for Linux on x86_64 (MySQL Community Server - GPL)` 说明系统内确实存在生效的 MySQL 环境。
    * 包检索：`rpm -qa | grep -i mysql`，参数含义为 `-q`（query 查询）、`-a`（all 查询所有已安装包）、`|`（管道符将前命令输出作为后命令输入）、`grep -i`（ignore-case 忽略大小写匹配）。
    * 卸载必须用 `yum remove` 而非 `rpm -e`：MySQL 组件间存在严密上下游依赖（server 依赖 client、client 依赖 libs、libs 依赖 common），`rpm -e` 一旦顺序颠倒会立刻抛出 `error: Failed dependencies` 阻断执行，而 `yum remove` 具备自动依赖解析树，能计算上下游关联包并给出级联清理方案。
    * 数据残留清理：`find / -name mysql` 通常搜出三类路径 —— `/var/lib/mysql`（默认核心数据目录）、`/var/lib/mysql/mysql`（系统权限元数据子目录）、`/usr/lib64/mysql`（64 位系统底层动态链接库与扩展支持目录）。
    * `rm -rf` 参数含义：`-r`（recursive 递归删除子目录与深层文件）、`-f`（force 强制删除只读属性或不存在文件并跳过所有确认交互）。
    * > 易错点：包管理器卸载逻辑只删除 `/usr/bin` 下的可执行程序与 `/usr/share` 下的手册，出于数据安全策略绝不会自动删除用户的数据库数据文件；不手动清理会导致重装后直接挂载历史残留的系统表空间 `ibdata1` 与事务日志，造成数据字典不一致、root 初始密码失效或服务无法启动。
    * > 易错点：Linux 下系统服务默认命名为 `mysqld`，末尾字母 `d` 代表 Daemon（守护进程），命令末尾的 `.service` 后缀可省略；而 Windows 下 MySQL 服务名通常为 `MySQL`、`MySQL80` 或 `MySQL57`。
    * > 来源: P98

* **跨平台配置文件与运维细节**
    * Windows 下 MySQL 主配置文件名为 `my.ini`，通常存放在安装目录根下或 `C:\ProgramData\MySQL\` 隐藏目录中；Linux 下统一命名为 `my.cnf`，存放在 `/etc/` 根下（完整路径 `/etc/my.cnf`）。
    * 多虚拟机环境下应维护环境备忘文档记录各节点的虚拟机标识、静态 IP、主机名、承载软件与版本、当前环境状态，避免连错节点造成误删或误操作。
    * 关闭 CentOS 7 自动锁屏：应用程序 → 系统工具 → 设置 → 隐私 → 屏幕锁定，将自动锁屏与休眠锁定切换为「关闭」。
    * > 来源: P98

---

## 3. 远程连接与接入链路排查

> **一句话主旨**：按四层链路定位远程连接失败，处理 host 授权、8.0 认证插件降级与密码强度策略。

* **远程连接四层排查模型**
    * 从宿主机客户端向虚拟机数据库发起 TCP 握手涉及操作系统、虚拟网络、防火墙规则、数据库进程及访问控制策略多层结构，必须逐层定位而非盲目重装。
    * 典型阻断报错为 `Error No. 2003: Can't connect to MySQL server on '192.168.1.150' (10060)`。
    * 四层顺序固定：物理与链路层（Ping 检测路由可达性）→ 传输与防火墙层（Telnet 3306 检测防火墙拦截）→ 数据库访问权限层（检查 `mysql.user` 表 root 是否仅绑定 localhost）→ 认证算法兼容层（解决 8.0 独有的 `caching_sha2_password` 不兼容）。
    * > 来源: P100

```text
远程数据库连接全链路排查模型：
[第一层: 物理与链路层]  Ping 检测 (网络是否通畅、路由是否可达)
        |
        v  通畅
[第二层: 传输与防火墙]  Telnet 3306 (宿主机与虚拟机防火墙是否拦截)
        |
        v  放行
[第三层: 数据库访问权限] 检查 mysql.user 表 (root 是否仅绑定 localhost)
        |
        v  放开 host=%
[第四层: 认证算法兼容性] 解决 8.0 独有的 caching_sha2_password 插件不兼容
```

* **前三层故障的现象与修复**
    * 第一层：宿主机 `ping 192.168.1.150` 收到 4 次 ICMP 应答、丢包率 0%，证明虚拟机处于开机状态且宿主机与虚拟机之间虚拟网络拓扑（VMnet8）完全通畅。
    * 第二层：`telnet 192.168.1.150 3306` 若等待数秒后报 `无法打开到主机的连接。在端口 3306: 连接失败`，证明 3306 端口无法完成三次握手、流量遭防火墙拦截；本地开发、教学与测试环境最推荐直接关闭并禁用防火墙以杜绝后续多节点集群通信的端口干扰。
    * 防火墙生命周期管控命令链：`systemctl status firewalld` 查状态（`Active: active (running)` 为强过滤）→ `systemctl stop firewalld` 立即关闭 → 再次 `status` 确认变 `dead` → `systemctl disable firewalld` 彻底禁用开机自启。
    * 第三层：防火墙关闭后 `telnet` 会瞬间黑屏并输出 MySQL 握手版本字符串，此时若 SQLyog 仍报 `Host '192.168.1.100' is not allowed to connect to this MySQL server`，说明命中的是账户主机绑定限制。
    * > 注意：Windows 宿主机的本地高级安全 Windows Defender 防火墙在系统补丁更新后有时会自动重置并开启，阻断局域网回环流量；关闭 Linux 防火墙后仍无法通信时需同步核查本地防火墙规则。
    * > 来源: P100

* **root 账户 host 拓权与权限缓存刷新**
    * MySQL 核心账户信息存放在系统库 `mysql` 的 `user` 表中，默认安全策略把初次初始化生成的 root 账户绑定主机死死限制为 `localhost`，即仅允许从服务器本地（UNIX Domain Socket 或本机 127.0.0.1）登录，严禁任何外部 IP 网络连接。
    * 查询语句 `USE mysql; SELECT host, user FROM user;` 在 8.0 初始状态下返回 4 行：`localhost/root`、`localhost/mysql.infoschema`、`localhost/mysql.session`、`localhost/mysql.sys`。
    * 拓权语句 `UPDATE user SET host = '%' WHERE user = 'root';`，随后必须立即执行 `FLUSH PRIVILEGES;`。
    * `FLUSH PRIVILEGES` 的作用机理：MySQL 启动时会把 `mysql.user` 表内容全部载入内存的授权缓存区，直接执行 `UPDATE` 修改底层表不会自动触发内存字典重载，必须显式让服务器清空并重载授权表缓存，修改才能对正在运行的服务端生效。
    * > 易错点：把 root 的 host 设为 `%` 仅适用于教学实验环境；真实生产环境与商业系统运维中绝不允许将 root 的 host 设为 `%` 并暴露在公网，生产规范要求严格指定受信任内网 IP，或绑定特定子网掩码（如限制为 `192.168.1.%`），将攻击面收敛至最小。
    * > 来源: P100

* **MySQL 8.0 认证插件降级**
    * 报错形态为 `Plugin caching_sha2_password could not be loaded: 找不到指定的模块。`，根因是 8.0 把默认认证插件从 5.7 及以前的 `mysql_native_password`（基于 SHA-1 双重哈希验证，全生态客户端与驱动完美适配）全面替换为基于 SHA-256 的 `caching_sha2_password`（支持内存缓存提升握手效率），而老版本 SQLyog、早期 Navicat 以及部分老旧 JDBC 驱动包未内置解析该插件的动态链接库。
    * 标准降级语句：`ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'abc123';` 后跟 `FLUSH PRIVILEGES;`。
    * 5.7 节点只需关闭并禁用防火墙、执行同样的 `host` 拓权与刷新即可连接成功，全程不会弹出插件加载提示，从实操上验证 5.7 默认的 `mysql_native_password` 天然与旧版客户端驱动契合。
    * > 来源: P100

* **密码强度评估组件（`validate_password`）**
    * 纯原生最小化安装中该模块默认并未强行启用；激活有两条途径：在 `/etc/my.cnf` 的 `[mysqld]` 标签下追加密码策略配置参数后重启服务，或在命令行执行 `INSTALL PLUGIN validate_password SONAME 'validate_password.so';`。
    * 激活后把密码改为 `abcd1234` 这类缺乏特殊字符或全小写的简单密码会被阻断，报错为 `ERROR 1819 (HY000): Your password does not satisfy the current policy requirements`。
    * 策略等级：`LOW`（0）仅校验最小长度；`MEDIUM`（1，默认）在长度满足前提下必须同时混用大写字母、小写字母、数字与特殊符号；`STRONG`（2）在 MEDIUM 基础上加载内置字典文件，密码包含字典常见单词或字典子串即判定违规。
    * 强度预评估函数 `SELECT VALIDATE_PASSWORD_STRENGTH('Atguigu_2026_Root');` 返回 `0 ~ 100` 之间的整数，返回 `100` 说明长度、多样性及复杂度均达最高标准。
    * 动态松绑：`SET GLOBAL validate_password.policy = LOW;`、`SET GLOBAL validate_password.length = 4;`；彻底注销组件用 `UNINSTALL PLUGIN validate_password;`。
    * > 来源: P100

| 系统参数项 | 默认取值 | 技术含义与校验规则 |
| :--- | :--- | :--- |
| `validate_password.policy` | `MEDIUM`（1） | 密码安全策略等级（支持 `LOW`/0、`MEDIUM`/1、`STRONG`/2） |
| `validate_password.length` | `8` | 允许设定的最小密码长度（字符数必须 $\ge 8$） |
| `validate_password.mixed_case_count` | `1` | 密码中必须包含的大写与小写字母的最少数量 |
| `validate_password.number_count` | `1` | 密码中必须包含的阿拉伯数字的最少数量 |
| `validate_password.special_char_count` | `1` | 密码中必须包含的非字母数字特殊字符（如 `_`、`@`、`#`）的最少数量 |

---

## 4. 字符集继承体系与比较规则

> **一句话主旨**：理清字符集在服务器、库、表、列四级的继承与覆盖，掌握两级原地变更语法与比较规则命名规范。

* **版本默认字符集差异与变量清单**
    * 5.7 默认字符集为 `latin1`，8.0 默认升级为 `utf8mb4`；这是同样建库建表语句在 5.7 下插入中文报 `ERROR 1366 (HY000): Incorrect string value` 或出现乱码、而在 8.0 下丝滑存入的根源。
    * 创建数据库和数据表时起决定性默认值作用的核心变量是 `character_set_server` 与 `character_set_database`；`character_set_system` 存储元数据（表名、字段名），两版本均为 `utf8`。
    * `latin1` 仅占用单字节（`0x00~0xFF`），内部编码表不存在汉字码点映射，底层引擎尝试把 UTF-8 中文三字节序列塞入 `latin1` 字段时因无法匹配有效字符而直接阻断写入。
    * 几乎所有现代单字节与多字节字符集都向下兼容标准 7 位 ASCII 码（0~127），因此插入英文 `'Tom'` 在两版本下都正常。
    * > 来源: P101

| 变量名 | 8.0 默认值 | 5.7 默认值 | 作用简析 |
| :--- | :--- | :--- | :--- |
| `character_set_server` | `utf8mb4` | `latin1` | 服务器默认字符集 |
| `character_set_database` | `utf8mb4` | `latin1` | 默认创建或当前选中数据库的字符集 |
| `character_set_client` | `utf8mb4` | `utf8` | 客户端来源数据的编码字符集 |
| `character_set_connection` | `utf8mb4` | `utf8` | 接收并处理请求时所用的连接字符集 |
| `character_set_results` | `utf8mb4` | `utf8` | 返回给客户端的查询结果字符集 |
| `character_set_system` | `utf8` | `utf8` | 存储元数据（如表名、字段名）的系统字符集 |

* **四级继承与覆盖链**
    * 服务器级别由配置文件中的 `character_set_server` 决定，是整个实例的顶层兜底基准。
    * 数据库级别：`CREATE DATABASE` 未显式写出 `CHARACTER SET` 时自动继承当前服务端的 `character_set_server`。
    * 表级别：`CREATE TABLE` 未显式写出 `CHARACTER SET` 时自动继承其所属数据库的字符集。
    * 列级别：定义字符类型列（如 `VARCHAR(15)`）时未单独指明 `CHARACTER SET`，该字段自动继承数据表的默认字符集。
    * > 易错点：建表时若未显式指定字符集，它不会越级去找全局的 `character_set_server`，而是严格继承其所属数据库的字符集。在历史遗留的 `latin1` 库 `dbtest1` 下新建表 `emp2`，即便全局变量已改为 `utf8`，`emp2` 依然默认是 `latin1`。
    * > 来源: P101

```text
四级继承与覆盖链：

  服务器级别  character_set_server
        |  建库未指明字符集时向下继承
        v
  数据库级别  character_set_database
        |  建表未指明字符集时向下继承
        v
  表级别      DEFAULT CHARSET
        |  建字段未指明字符集时向下继承
        v
  列级别      VARCHAR(...) CHARACTER SET

每一层级均可显式声明，覆盖上一级的继承结果。
```

* **配置生效边界与两级原地变更**
    * 修改 `my.cnf` 中的 `character_set_server` 属于静态变更，不会作用于正在运行的 `mysqld`，必须 `systemctl restart mysqld`；重启后 `character_set_server` 变更成功且 `character_set_database` 紧随其后同步变更。
    * 作用域边界：修改 `character_set_server` 仅仅影响在此之后全新创建的数据库和表，对已经存在的历史库、历史表没有任何溯及既往的追溯修改作用。
    * 变更已有库：`ALTER DATABASE dbtest1 CHARACTER SET utf8;`，此后在该库下新创建的表自动继承新字符集。
    * 变更已有表：`ALTER TABLE emp1 CONVERT TO CHARACTER SET utf8;`，该语句彻底转换表字符集及现有列中所有文本数据的编码。
    * > 易错点：仅修改数据库字符集并不会级联更新库中已经存在的表，必须对具体表执行 `ALTER TABLE` 操作。
    * > 注意：做字符集转换必须确保源字符集能被目标字符集安全兼容（例如从单字节 `latin1` 提升到多字节 `utf8`，属于小集合向大集合的扩容包含）；若从大字符集盲目强转为小字符集，或在两个不兼容的多字节编码（如从 `GBK` 强行非对称转为某种小语种编码）之间硬转，极易导致底层字节码点无法映射，造成不可恢复的乱码。
    * > 来源: P101

* **`CONVERT TO` 与普通 `CHARACTER SET` 的语义区别**
    * `ALTER TABLE t1 CONVERT TO CHARACTER SET utf8;` 为全面转换：不仅修改表级别的默认元数据定义，还会将表中现存所有字符类型字段（`CHAR`、`VARCHAR`、`TEXT` 等）的列字符集批量更新，同时把底层存储的现有二进制字节流原地重新编码转储。
    * `ALTER TABLE t1 CHARACTER SET utf8;` 为浅层修改：仅修改表元数据中的默认字符集属性，已存在的物理字段其字符集保持原样、已有数据亦不做重新编码，仅对后续 `ALTER TABLE t1 ADD COLUMN` 未指明字符集的新增列生效。
    * > 来源: P101

* **显式声明与生产统一性准则**
    * 库级显式声明：`CREATE DATABASE my_custom_db CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
    * 表级显式声明：即使所属数据库使用 `latin1`，也可在建表时单独指定，形如 `) DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`
    * 列级显式声明：粒度可细化到具体字段，同一张表中允许不同字段拥有不同字符集，例如 `en_desc VARCHAR(50) CHARACTER SET latin1` 与 `cn_desc VARCHAR(50) CHARACTER SET utf8mb4` 并存。
    * 生产环境务必保持端到端全链路字符集统一：前端展示层、应用服务端（Java Web 连接池与代码编译）、数据库服务端（server、database、table、column）必须统一采用 UTF-8，8.0 推荐 `utf8mb4`；表间或列间存在异构字符集时，跨表 `JOIN` 与跨服务传输会迫使 MySQL 在查询执行期间频繁内部转码，浪费 CPU 并可能因字符集或排序规则不匹配导致索引失效，引发慢查询与全表扫描。
    * > 来源: P101

* **比较规则（Collation）命名与行为差异**
    * 比较规则定义字符串在排序（`ORDER BY`）、比较大小（`>`、`<`）以及等值匹配（`=`、`LIKE`）时遵循的准则，命名格式为 `字符集名_语言或通用规范_比较行为后缀`。
    * 每个字符集有且仅有一个默认比较规则；`SHOW CHARACTER SET;` 在 8.0 返回 41 行记录，核心字段为 `Charset`、`Description`、`Default collation`、`Maxlen`（`gbk` 是 2、`utf8` 是 3、`utf8mb4` 是 4）。
    * `utf8` 与 `utf8mb4` 的本质区别在于最大字节数：早期 `utf8` 实为阉割版的 `utf8mb3`（Max Bytes 3），只支持最多 3 字节编码；标准 Unicode UTF-8 是变长编码，单字节兼容 ASCII、双字节覆盖欧洲拉丁语系与希腊文阿拉伯文、三字节覆盖常用汉字与日韩基础多语言平面（BMP）字符、四字节覆盖辅助平面字符（典型代表为 Emoji 表情与极少数生僻字），因此 8.0 将默认字符集全面替换为 `utf8mb4`。
    * > 来源: P102

| 后缀 | 英文全称 | 核心含义 | 技术特征 |
| :--- | :--- | :--- | :--- |
| `_ai` | Accent Insensitive | 不区分重音 | 忽略重音字符差异（如 `e` 与 `é` 视为相同） |
| `_as` | Accent Sensitive | 区分重音 | 严格区分重音字符 |
| `_ci` | Case Insensitive | 不区分大小写 | 字符串比较与匹配时 `'A'` 与 `'a'` 视为相等 |
| `_cs` | Case Sensitive | 区分大小写 | 严格区分大小写，`'A'` 与 `'a'` 视为不等 |
| `_bin` | Binary | 二进制比对 | 逐字节甚至逐比特比对 ASCII/Unicode 码点，天然区分大小写 |

* **比较规则的检索与变更**
    * 检索命令：`SHOW COLLATION LIKE 'gbk%';`、`SHOW COLLATION LIKE 'utf8%';`；示例规则包括 `gbk_chinese_ci`（GBK 的中文不区分大小写）、`gbk_bin`（GBK 二进制精确比对）、`utf8mb4_0900_ai_ci`（8.0 默认比较规则，基于 Unicode 9.0 规范，不区分重音、不区分大小写）。
    * `general_ci` 校对算法相对简单，比对与排序执行速度极快，但在某些特定西欧小语种的复杂排序上准确度稍逊；`unicode_ci` 完全遵从 Unicode 标准排序算法，能精准处理多语言混合与特殊符号排序，准确度极高但复杂字符集比对计算开销略大。
    * 查看服务端与库级规则：`SHOW VARIABLES LIKE 'collation_server';`、`SHOW VARIABLES LIKE 'collation_database';`
    * 变更库级规则：`ALTER DATABASE dbtest1 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
    * 查看表级规则：方式一 `SHOW CREATE TABLE emp1;`，方式二 `SHOW TABLE STATUS FROM dbtest1 LIKE 'emp1';` 查看 `Collation` 字段。
    * 变更表级规则：`ALTER TABLE emp1 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
    * > 注意：修改数据库的默认比较规则同样不影响已经存在的历史表，只对之后新建的表产生继承影响；需同步调整现有历史表时必须使用 `ALTER TABLE ... CONVERT TO`。
    * > 来源: P102

* **请求到响应全链路编解码**
    * 三个直接掌管网络通信链路的变量：`character_set_client` 是服务器假定客户端发送过来的 SQL 语句所采用的字符集；`character_set_connection` 是服务器接收请求后、执行 SQL 前将请求文本转换成的连接字符集，用于语法解析与处理；`character_set_results` 是服务器执行完毕后将结果集与错误信息编码后再回传客户端所用的字符集。
    * 汉字 `'我'` 在 UTF-8 下占 3 个字节（`0xE6 0x88 0x91`），在 GBK 下占 2 个字节（`0xCED2`）。
    * > 易错点：若客户端实际发出 UTF-8 的 3 字节而服务端把 `character_set_client` 错误配成 `gbk`，服务端会以双字节为单位强行拼装，把前两个字节拼成一个乱码字符、第三个字节截断溢出；请求在进入解析阶段之前就已经损坏，后续必然查不到数据或报语法错误。
    * > 来源: P102

| 步骤 | 执行主体与动作 | 本例编码结果 |
| :--- | :--- | :--- |
| 1 | 客户端用自身编码把 SQL 转化为二进制字节流发送 | UTF-8 发出 `0xE6 0x88 0x91` |
| 2 | 服务端按 `character_set_client` 解码为逻辑字符 | `client=utf8`，正确还原为 `'我'` |
| 3 | 转换为 `character_set_connection` 编码 | `connection=gbk`，内存中转为 `0xCED2` |
| 4 | 存储引擎检索比对（表字段定义为 gbk） | `0xCED2` 与物理行字节完全吻合，锁定目标行 |
| 5 | 字段值提取为逻辑字符 | 按列定义 gbk 还原为通用字符 `'我'` |
| 6 | 按 `character_set_results` 编码 | `results=utf8`，重新编码为 `0xE6 0x88 0x91` |
| 7 | 网络回传二进制结果字节流 | 字节流原样返回客户端 |
| 8 | 客户端用本地展示字符集解码呈现 | 屏幕准确渲染出汉字 `'我'` |

* **两对铁律与会话级、持久化配置**
    * 请求入站一致性：客户端发送实际采用的编码必须与服务端 `character_set_client` 严格一致。
    * 响应出站一致性：服务端 `character_set_results` 的编码必须与客户端终端或展现层所使用的解码字符集严格一致。
    * 中间连接层容忍度：`character_set_connection` 理论上可与前后两者不同，只要其编码空间能完整容纳待转换的字符集；但中途转码会消耗 CPU，且易因字符集包含范围不对称（如从 UTF-8 转为 Latin1）导致无法逆转的字符丢失。
    * 会话级快捷配置 `SET NAMES utf8mb4;` 等效于并发执行 `SET character_set_client = utf8mb4;`、`SET character_set_connection = utf8mb4;`、`SET character_set_results = utf8mb4;` 三条赋值语句。
    * 持久化配置：在 `/etc/my.cnf`（或 Windows 下 `my.ini`）的 `[client]` 段设 `default-character-set=utf8mb4` 作用于所有直连客户端工具；`[mysql]` 段设 `default-character-set=utf8mb4` 针对命令行工具；`[mysqld]` 段设 `character-set-server=utf8mb4` 与 `collation-server=utf8mb4_0900_ai_ci`。
    * `[client]` 标签下配置后，任何使用 MySQL 官方库建立连接的程序在握手阶段都会自动通知服务端将 `client`、`connection`、`results` 三变量统一步齐为 `utf8mb4`，从源头杜绝终端字符集不匹配引发的传输乱码。
    * > 来源: P102

---

## 5. 大小写规范与 sql_mode

> **一句话主旨**：划清跨平台大小写敏感的对象边界与三值语义，掌握严格模式的行为差异与生产持久化方案。

* **`lower_case_table_names` 三值语义与平台差异**
    * Windows 系统的 MySQL 8.0 实例中该变量默认返回 `1`；Linux（CentOS/Ubuntu）下默认返回 `0`。
    * 底层原因是文件系统行为差异：Windows 的 NTFS/FAT 文件系统本身对文件名大小写不敏感，Linux 的 ext4/xfs 严格区分文件名大小写；MySQL 中数据库在磁盘上对应一个目录、表对应数据文件与元数据文件，因此 Linux 默认继承文件系统的严格区分规则。
    * > 来源: P103

| 参数值 | 存储形式 | 比对查找规则 | 默认适用系统 | 核心特征 |
| :---: | :--- | :--- | :--- | :--- |
| `0` | 按 SQL 声明的原样大小写存储 | 大小写严格敏感（比对二进制字符） | Linux / Unix | `Emp` 与 `emp` 视为两张不同的表 |
| `1` | 全部自动转换为小写存储到磁盘 | 大小写不敏感（统一转小写比对） | Windows | 输入大写也会转为小写处理 |
| `2` | 按 SQL 声明的原样大小写存储 | 实际比对时转为小写比较 | macOS / 不区分大小写的文件系统 | 原样保存展示，但比对时不区分大小写 |

* **Linux 下的大小写敏感边界**
    * 严格区分大小写的对象：数据库名、数据表名、表的别名、用户自定义变量名。
    * 完全不区分大小写的对象：SQL 关键字（`SELECT`、`FROM`、`WHERE`、`GROUP BY`、`INSERT` 等）、内置函数名（`COUNT`、`MAX`、`MIN`、`NOW`、`CONCAT` 等）、列名/字段名、列的别名。
    * 实测报错形态：`USE dBtest1;` 报 `ERROR 1049 (42000): Unknown database 'dBtest1'`；`SELECT * FROM Emp1;` 报 `ERROR 1146 (42S02): Table 'dbtest1.Emp1' doesn't exist`。
    * 大小写混用可正常执行的形态：`sElEcT COUNT(*) FrOm emp1;` 与 `SELECT id, ID, name, NAME FROM emp1;` 均无报错。
    * > 来源: P103

* **两版本对参数的修改限制**
    * 5.7 中可通过修改配置文件变更：在 `/etc/my.cnf` 的 `[mysqld]` 段追加 `lower_case_table_names=1` 后 `systemctl restart mysqld`。
    * 8.0 引入全新的数据字典架构，元数据统一托管在数据字典表空间中，该参数被强制规定只能在数据库初始化阶段（`mysqld --initialize`）指定。
    * > 易错点：5.7 中修改该参数前必须先将库中所有现存的历史数据库名、表名、视图名全部手动转换为小写；若磁盘上已存在大写文件名（如 `EMP.frm`、`EMP.ibd`），改为 `1` 后 MySQL 查询时会将 SQL 统一转为小写 `emp` 去磁盘检索，导致已有历史表彻底无法被定位。
    * > 易错点：8.0 实例初始化完成后强行在 `/etc/my.cnf` 修改该参数并重启，`mysqld` 会直接启动崩溃并拒绝服务；非要修改必须备份全库数据、彻底清空 `/var/lib/mysql` 数据目录，并在初始化命令中附带该参数重新初始化整个实例。
    * > 来源: P103

* **生产级 SQL 书写规范**
    * 关键字与函数名全部大写，例如 `SELECT`、`FROM`、`WHERE`、`LEFT JOIN`、`COUNT()`、`MAX()`。
    * 库名、表名、表别名全部小写，多个单词间使用下划线分割，例如 `order_detail`、`emp_record`。
    * 字段名、字段别名全部小写，例如 `user_id`、`create_time`；SQL 语句显式以分号结尾，严格规范语法边界。
    * 大小写区分的初衷是让开发人员通过视觉扫视一眼剥离 SQL 骨架与业务结构：全大写即为语言机制与算子，全小写即为自定义业务库表实体。
    * > 来源: P103

* **`sql_mode` 的作用与两版本演进**
    * `sql_mode`（SQL 模式）是 MySQL 的核心系统变量，用于定义系统支持的 SQL 语法规范，以及执行数据插入、更新和聚合计算时的数据有效性校验严格程度。
    * MySQL 5.6 及更早版本默认值通常为空字符串，处于宽容模式；5.7 及 8.0 默认启用 `STRICT_TRANS_TABLES` 等一系列强校验参数，全面切换为严格模式。
    * 宽容模式行为：字符串超长自动静默截断保留前 N 位并给出 Warning 不报错；类型不匹配尝试强制类型转换或填充零值（`0` / `0000-00-00`）；语法不严谨时允许隐式聚合产生不可预期的数据输出。
    * 严格模式行为：字符串超长直接抛出 Error、事务中断、拒绝非法数据落盘；类型不匹配拒绝强制转换直接报错；强制 `ONLY_FULL_GROUP_BY` 阻断歧义聚合。
    * 宽容模式唯一的合理使用场景是异构数据库的历史数据迁移，可临时调为宽容模式避免个别脏数据阻断整体迁移进度。
    * > 来源: P103

* **常见约束行为与实测案例**
    * 查看方式：`SELECT @@session.sql_mode;` 查看会话级，`SELECT @@global.sql_mode;` 查看全局级。
    * 截断实测：在 `name CHAR(10)` 字段插入长度 13 的 `'1234567890ABC'`，宽容模式下仅抛警告并自动截取前 10 个字符 `'1234567890'` 存入磁盘、后缀 `'ABC'` 被静默丢弃；严格模式下直接抛出 `Data too long for column 'name'` 彻底终止执行。
    * 分组歧义实测：`SELECT name, dept, MAX(age) FROM my_emp GROUP BY dept;` 在严格模式下抛 `ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column ... this is incompatible with sql_mode=only_full_group_by`；执行 `SET SESSION sql_mode = '';` 后语句可跑通，但 `name` 字段显示的值纯属物理行上的随机首条记录，无法保证名字与最大年龄形成精确映射。
    * 类型拦截实测：在宽松模式下向 `INT` 类型 `age` 字段写入 `'aaa'` 不报错仅给警告，查询结果显示 `age=0`，系统擅自将无法解析的字符隐式转换填充为整型默认值 `0`；恢复严格模式后该语句被拒绝并抛出 `ERROR 1366 (HY000): Incorrect integer value: 'aaa' for column 'age' at row 1`。
    * > 来源: P103

| 模式名称 | 核心限制与校验规则 |
| :--- | :--- |
| `ONLY_FULL_GROUP_BY` | 强制标准 SQL 分组规范：`SELECT` 列表中除聚合函数外的所有列必须出现在 `GROUP BY` 之后 |
| `STRICT_TRANS_TABLES` | 对事务型存储引擎（如 InnoDB）开启严格数据校验，非法数据直接报错阻断事务 |
| `NO_ZERO_IN_DATE` | 禁止日期中出现月份为 0 或日为 0 的半合法日期（如 `'2026-00-01'`） |
| `NO_ZERO_DATE` | 禁止插入全零非法日期（如 `'0000-00-00'`） |
| `ERROR_FOR_DIVISION_BY_ZERO` | 除法运算遇到除数为 0 时抛出错误而非静默返回 `NULL` |
| `NO_ENGINE_SUBSTITUTION` | 建表指定的存储引擎不可用时直接报错而非自动降级使用默认引擎 |

* **生产环境双管齐下变更方案**
    * 纯临时修改（`SET GLOBAL`）立即对新建连接生效、无需重启、业务无感知，但服务器宿主机维护重启后所有配置瞬间丢失并回退到旧配置。
    * 纯静态文件修改（`my.cnf`）持久化保存在磁盘、不怕实例重启，但必须重启 `mysqld` 服务才能加载，高并发生产环境下随意重启会导致大量线上交易中断。
    * 最佳实践为两步并行：第一步执行 `SET GLOBAL sql_mode = '...'` 使线上实时生效、新接入会话即刻应用、零停机无感知（当前已建立的长连接保持原样）；第二步编辑 `/etc/my.cnf` 在 `[mysqld]` 标签下追加相同 `sql_mode` 配置落盘固化。
    * 常用严格模式组合值：`ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION`。
    * > 来源: P103

---

## 6. 数据目录结构与物理文件映射

> **一句话主旨**：把库、表、索引与日志对应到 Linux 真实文件路径，讲清两版本与两引擎的文件形态演进。

* **跨平台目录布局与核心路径**
    * Windows 下 MySQL 依托两个核心目录：程序目录（如 `D:\Program Files\MySQL\MySQL Server 8.0\`，核心是内置 `mysql`、`mysqld`、`mysqldump` 的 `bin` 子目录）与数据配置目录（如 `C:\ProgramData\MySQL\MySQL Server 8.0\`，含主配置文件 `my.ini` 与 `Data` 目录）。
    * `SHOW VARIABLES LIKE 'datadir';` 在 Linux 下返回 `/var/lib/mysql/`，印证这是数据库的官方物理落盘点。
    * 命令目录分工：`/usr/bin/` 存放客户端管理指令，`/usr/sbin/` 存放服务端守护进程 `mysqld`；`/usr/share/mysql-8.0/` 存放默认配置模板、多语言错误提示信息文件及字符集说明包。
    * > 来源: P104

| 目录路径 | 对应角色 | 承载内容与功能 |
| :--- | :--- | :--- |
| `/var/lib/mysql/` | 数据目录（Data Directory） | 存储所有系统库、用户数据库、表空间、持久化数据与重做日志 |
| `/usr/bin/` 与 `/usr/sbin/` | 命令目录（Binary Directory） | `/usr/bin/` 存放客户端管理指令；`/usr/sbin/` 存放服务端守护进程 `mysqld` |
| `/etc/my.cnf` | 主配置文件（Config File） | 服务端全局配置文件（等同于 Windows 下的 `my.ini`） |
| `/usr/share/mysql-8.0/` | 共享支持目录（Share Directory） | 存放默认配置模板、多语言错误提示信息文件及字符集说明包 |

* **客户端工具集职责划分**
    * `mysql` 是交互式客户端命令行入口；`mysqladmin` 是服务运维管理程序（检查状态、关机、刷新日志）；`mysqlbinlog` 专门用于解析与提取二进制日志（Binary Log）。
    * `mysqldump` 是逻辑全量与分库分表备份工具；`mysqlimport` 是文本数据高速导入工具；`mysqlcheck` 是表完整性检查、修复与优化工具。
    * > 来源: P104

* **四大默认系统数据库**
    * `mysql` 是核心系统库，直接记录连接实例所需的用户账号、主机授权与权限映射，其 `user` 表是整个实例访问控制体系的基石，此外还存放插件定义、时区数据、系统帮助文档等元数据。
    * `information_schema` 完整遵从 ANSI SQL 标准，维护当前实例中所有其他数据库、数据表、列字段、索引、视图、触发器及存储过程的元数据（如 `information_schema.routines`、`information_schema.triggers`）；它在物理磁盘上没有实体数据文件，是一套纯内存中的虚拟视图。
    * `performance_schema` 在底层微秒级收集 MySQL 服务的运行时内部事件，记录内存消耗、锁等待、线程执行生命周期、IO 读写频率等性能指标，设计为极低计算开销。
    * `sys` 通过一系列高级封装视图把 `information_schema` 与 `performance_schema` 结合起来，供开发人员与 DBA 一键定位「当前最消耗内存的查询」「哪个索引从未被使用」等生产级瓶颈。
    * > 来源: P104

* **InnoDB 在 5.7 下的三文件形态与表空间架构**
    * 5.7 中一个数据库对应一个子目录，`dbtest1` 目录下呈现 `db.opt`、`emp1.frm`、`emp1.ibd` 三个独立文件。
    * `db.opt` 是数据库属性元文件，用纯文本记事本即可打开，仅存储两行元数据：当前数据库默认的字符集与比较规则（如 `default-character-set=utf8` 与 `default-collation=utf8_general_ci`）。
    * `表名.frm` 是表结构定义文件（Format File），存放该表的 DDL 定义信息，包括字段名称、数据类型、主外键约束与默认值，所有引擎在 5.7 中通通都有 `.frm`。
    * 系统表空间（System Tablespace）：`/var/lib/mysql/` 根目录下的公共 `ibdata1` 文件，默认初始大小为 12MB，支持按需自动扩展；在 MySQL 5.5.7 至 5.6.6 早期版本中，全实例所有库、所有表的数据与索引默认混杂堆放在这一个全局文件中。
    * 独立表空间（File-Per-Table Tablespace）：将各表数据独立剥离为 `表名.ibd`，由参数 `innodb_file_per_table` 控制（5.6.6 及后续版本默认开启为 `ON`）；开启后 `emp1` 表的行记录数据以及聚簇索引全部封装在独立的 `emp1.ibd` 文件中。
    * > 来源: P104

* **InnoDB 在 8.0 的结构数据合一**
    * 8.0 的 `dbtest1` 目录下仅剩 `emp1.ibd` 一个文件，`db.opt` 被彻底废弃，数据库层级的字符集与排序规则统一迁移到全局数据字典表空间中维护。
    * `.frm` 文件完全不复存在，官方实现了表结构与表数据的彻底合一：表的列定义、索引元数据（SDI，序列化字典信息）被直接作为一段 JSON 序列化数据内嵌灌装在 `emp1.ibd` 独立表空间文件内部。
    * 逆向解析工具 `ibd2sdi`（IBD to Serialized Dictionary Information）可从二进制 `.ibd` 文件中解构出完整表定义 JSON：`ibd2sdi --dump-file=emp1.txt emp1.ibd`，导出文本中可见 `"name": "emp1"`、字段 `id` 的 `"type": 4`（整型，含 `is_nullable`、`is_zerofill`、`is_unsigned` 等修饰符）、字段 `name` 的 `"type": 16`（变长字符串 `VARCHAR`）。
    * > 来源: P104

* **MyISAM 的文件拆分与聚簇索引依据**
    * 5.7 下 MyISAM 呈现三文件分离架构：`student.frm` 表结构定义、`student.MYD`（MYData）存放实际数据行、`student.MYI`（MYIndex）单独存放为该表构建的 B+ 树索引结构。
    * 8.0 下 `student.MYD` 与 `student.MYI` 保持不变，`.frm` 退出历史舞台，由独立的 `student_xxx.sdi` 文本文件接替，里面以 JSON 格式完整记录该表的字段与约束定义。
    * 拆分的根本原因：MyISAM 采用非聚簇索引架构，索引与数据物理上完全解耦，索引文件 `.MYI` 的叶子节点仅记录数据的物理磁盘地址指针，数据文件 `.MYD` 按写入顺序平铺追加数据行，两者必须拆为两个独立文件。
    * InnoDB 采用聚簇索引架构，崇尚「索引即数据，数据即索引」，主键 B+ 树叶子节点本身直接包裹并容纳整行完整数据，索引与数据血肉相连，因此物理上直接整合进唯一的 `.ibd` 表空间文件。
    * > 来源: P104

* **视图与核心日志文件的物理痕迹**
    * 视图（View）是逻辑上的虚表，本身不产生、不拥有任何真实物理行数据；5.7 中创建视图 `v_emp` 只会在数据库目录下生成轻量级的 `v_emp.frm` 文件记录创建该视图的 `SELECT` 语句骨架，绝不会派生 `.ibd` 或 `.MYD`。
    * 重做日志（Redo Log）如 `ib_logfile0`、`ib_logfile1` 保障事务持久性（Crash Recovery）；回滚日志（Undo Log）在独立的 undo 表空间中维护，支撑事务回滚与 MVCC 多版本并发控制。
    * 错误日志（Error Log）记录实例启停、严重故障与崩溃追踪堆栈；二进制日志（Binary Log）如 `binlog.000001` 按顺序记录所有 DDL 与 DML 变更事件，是主从复制与灾难恢复的心脏。
    * > 来源: P104

---

## 7. 用户、密码、权限与角色

> **一句话主旨**：以 `'user'@'host'` 二元组为轴，打通用户生命周期、密码存储与安全策略、权限四表与漏斗鉴权、角色激活。

### 7.1 客户端连接参数与账户物理结构

* **完整连接参数与执行模式**
    * 标准完整命令格式：`mysql -h [主机地址/IP] -P [端口号] -u [用户名] -p[密码] [目标数据库名] -e "[SQL语句]"`。
    * `-h`（`--host`）缺省时默认为 `localhost` 或 `127.0.0.1`；`-P`（`--port`，大写字母 P）缺省时默认为 `3306`；`-u`（`--user`）缺省时可能默认匹配操作系统当前登录的用户名；`-p`（`--password`，小写字母 p）可直接紧跟明文密码或仅敲 `-p` 回车隐式输入。
    * 目标数据库名为可选项，连接成功后自动切换到该库（相当于登录后立即执行 `USE dbname;`）；`-e`（`--execute`）执行双引号内的一条或多条 SQL，查询完毕后立即打印结果并退出客户端。
    * > 易错点：命令行直接追加明文密码会触发警示 `mysql: [Warning] Using a password on the command line interface can be insecure.`，因为多用户 Linux 中其他有进程查看权限的用户可通过 `ps -ef` 从命令行参数捕获明文密码，该指令也会被原样写入 Shell 历史记录文件（如 `~/.bash_history`）；生产规范要求一律单独输入 `-p` 并在 `Enter password:` 遮罩提示中完成密文录入。
    * > 来源: P105

* **系统字典库与 `user` 表联合主键**
    * 安装初始化后数据目录自动建立名为 `mysql` 的内部核心数据库，包含存储账户配置、全局权限、库表列级权限细则、插件配置、时区数据、帮助文档等核心元数据。
    * 8.0 初始状态下 `SELECT host, user FROM mysql.user;` 呈现 4 个默认系统账户：`mysql.infoschema`、`mysql.session`、`mysql.sys` 为系统内置保留运维账户，专供数据字典锁机制、内置性能视图或内部会话提取数据使用，严禁手动删除或更改；第四项为管理员 `root`。
    * 判定一个用户的唯一标识从来不是单纯的用户名，而是主机列与用户名构成的二元组 `'user'@'host'`；`DESC mysql.user;` 显示主键定义为 `PRIMARY KEY (Host, User)`。
    * 联合主键特性：`Host` 与 `User` 联合必须全局唯一且二者皆不允许为 `NULL`；允许存在多个同名 `User`，只要绑定的 `Host` 不同，例如 `'zhangsan'@'localhost'` 与 `'zhangsan'@'%'` 在底层物理表中属于两条完全独立的记录，各自持有独立的密码哈希值、生命周期策略与权限位。
    * > 来源: P105

### 7.2 账户创建、重命名与销毁

* **创建与缺省 Host 行为**
    * 标准语法为 `CREATE USER '用户名'@'主机名' IDENTIFIED BY '认证密码';`，例如 `CREATE USER 'zhangsan'@'localhost' IDENTIFIED BY 'abc123';`。
    * 省略 `'@主机名'` 时服务端自动补全为 `'zhangsan'@'%'`，通配符 `%` 表示允许该账户从任何远程主机/任意 IP 地址发起连接认证；由于 `(Host, User)` 联合主键的互异性，`'zhangsan'@'%'` 与 `'zhangsan'@'localhost'` 可以和谐共存。
    * 重复执行完全相同的创建语句会因复合主键冲突报 `ERROR 1396 (HY000): Operation CREATE USER failed for 'zhangsan'@'%'`。
    * > 来源: P105

* **`USAGE` 权限的零权限沙箱含义**
    * 新建用户 `SHOW DATABASES;` 只能看到 `information_schema` 一张虚拟视图，连业务库、`mysql` 系统库甚至 `performance_schema` 均无法感知。
    * `SHOW GRANTS;` 仅返回 `GRANT USAGE ON *.* TO \`zhangsan\`@\`%\``。
    * `USAGE` 权限项在 MySQL 中等同于「无权限（No privileges）」，仅赋予用户连接并握手登录到数据库服务器的资格，无法对任何库、表、视图或存储过程执行读写操作。
    * > 来源: P105

* **重命名与内存权限刷新机理**
    * 直接改名的底层写法为 `UPDATE mysql.user SET user = 'wangwu' WHERE user = 'lisi' AND host = '%';`
    * 直接 DML 后的反直觉现象：磁盘表中 `lisi` 已变成 `wangwu`，但旧用户名 `lisi` 依旧能成功登录，而新用户名 `wangwu` 登录被拒绝。
    * 机理：MySQL 服务端启动初始化时会将 `mysql.user`、`mysql.db` 等权限表全量内容一次性加载解析到内存数据结构（Privilege Hash Buffers）中；客户端每次发起连接握手时鉴权模块直接在内存缓存中执行哈希比对，根本不会实时扫描磁盘上的 `mysql.user` 表；直接用 DML 修改系统表只修改了存储引擎层（InnoDB 数据页与重做日志），没有通知权限模块刷新内存结构。
    * 显式执行的 `FLUSH PRIVILEGES;` 会强制从底层字典表重新读取所有账户与权限记录，清空并重建内存哈希映射；执行后 `lisi` 登录报 `ERROR 1045 (28000): Access denied for user 'lisi'@'...'`，`wangwu` 登录验证通过。
    * > 易错点：更新 `mysql.user` 表时 `WHERE` 条件千万不能只写 `user = 'lisi'`；一旦历史环境中存在 `'lisi'@'localhost'`、`'lisi'@'192.168.%'` 等多个同名记录，缺少 `host` 条件将直接导致全量同名用户被一并更改，破坏原有主机的拓扑隔离。
    * > 来源: P105

* **`DROP USER` 与 `DELETE FROM mysql.user` 的技术选型**
    * 标准 DDL 方式 `DROP USER 'wangwu'@'%';` 支持批量删除，多个账户之间用逗号分隔，例如 `DROP USER 'user1'@'localhost', 'user2'@'%', 'user3'@'192.168.1.100';`
    * 仅声明用户名时（如 `DROP USER 'zhangsan';`）默认行为依然是追加 `@'%'`；若实际需要删除本地账号必须显式指明 `DROP USER 'zhangsan'@'localhost';`
    * `DROP USER` 具备内部原子性清理：不仅抹除 `mysql.user` 中的记录，还会级联遍历并清空 `mysql.db`、`mysql.tables_priv`、`mysql.columns_priv`、`mysql.procs_priv` 以及默认角色映射表中的所有关联权限项；执行完毕自动触发内存权限缓存的清空与重置，完全不需要也不应该执行 `FLUSH PRIVILEGES;`
    * > 易错点：`DELETE FROM mysql.user WHERE user = 'zhangsan' AND host = 'localhost';` 在生产环境被明令禁止，其只删除本表行记录，会在 `mysql.db`、`mysql.tables_priv` 等表中残留孤儿权限记录（Orphaned Privilege Records）；未来一旦运维人员重新创建同名同 Host 的新账户，这个新账户在没有任何显式授权的情况下可能自动继承当年残留的历史库表访问权限，引发严重的数据越权漏洞。
    * > 易错点：`DROP USER` 成功后，若该用户此前已建立活跃会话，服务端不会当场强行 `KILL` 中断，该用户在已建立会话中仍能执行部分会话内查询；一旦执行 `quit` 或连接超时断开便彻底无法重新通过鉴权认证；需要立即阻断其所有在途操作时必须先 `SHOW PROCESSLIST;` 定位线程 ID（`Id`），再执行 `KILL [Thread_Id];` 强行切断底层 TCP 连接。
    * > 来源: P105

### 7.3 密码存储与安全策略

* **改密语法与底层密文字段**
    * 修改当前登录用户密码的官方首选方式是 `ALTER USER USER() IDENTIFIED BY '新密码';`，其中 `USER()` 是无参会话信息函数，动态解析并返回当前连接发起者的完整身份认证上下文（即 `'当前用户名'@'当前客户端Host'`）。
    * `ALTER USER` 属于原生 DDL，修改结果不仅在底层元数据表中完成事务提交，还会自动同步刷新内存中的权限缓存缓冲区，完全不需要执行 `FLUSH PRIVILEGES;`
    * 极简写法 `SET PASSWORD = '新密码';` 直接在交互式环境中修改自身凭据。
    * 存储字段：早期 MySQL 版本存储密码的字段名为 `password`，5.7 及以后的 8.0 版本已彻底重构并更名为 `` `authentication_string` text COLLATE utf8_bin ``。
    * 服务端绝不会在元数据表中存储明文密码，无论通过 `ALTER USER` 还是 `SET PASSWORD` 传入原始字串，都会调用当前用户绑定的认证插件（8.0 默认 `caching_sha2_password`）、引入动态盐值后生成不可逆的高强度哈希串持久化到 `authentication_string` 列中。
    * > 易错点：内置加密函数 `PASSWORD()` 在 MySQL 8.0 中已被官方正式彻底移除，在 8.0 实例中调用会立即抛出 `ERROR 1305 (42000): FUNCTION mysql.PASSWORD does not exist`；`UPDATE mysql.user SET authentication_string = PASSWORD(...)` 这类 5.7 时代的刷库写法在 8.0 中不仅语法失效，还存在插件算法差异导致无法手动生成合规哈希串、以及内存权限缓存不同步的问题。
    * > 来源: P106

* **管理员修改他人密码的两种语法**
    * `ALTER USER 'zhangsan'@'%' IDENTIFIED BY 'hello';` 必须在 `ALTER USER` 后精准提供目标用户的 `'用户名'@'主机名'` 二元组。
    * `SET PASSWORD FOR 'zhangsan'@'%' = 'hello123';` 的三个语法要素：必须包含 `FOR 'user'@'host'` 明确操作受体、赋值符号必须使用等号 `=`、后接单引号包裹的全新明文密码。
    * > 提示：管理员在后台修改某个用户的密码时，该用户当前正打开的命令行连接与现存 TCP 会话不会被服务端强行阻断，仍可继续执行查询；只有当该用户退出会话或发起新的连接握手时，新密码的约束才会强制介入。
    * > 来源: P106

* **密码过期策略（Expiration）**
    * 手动即时过期：`ALTER USER 'kangshifu'@'localhost' PASSWORD EXPIRE;` 触发受限沙箱登录状态 —— 用户输入原密码仍能通过 TCP 握手进入终端，但登录后执行任何业务查询（如 `SELECT * FROM emp1;` 或 `SHOW DATABASES;`）会被当场拦截并抛出 `ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`；该连接下唯一被允许执行的操作正是 `ALTER USER USER() IDENTIFIED BY '新密码';`
    * 全局自动定期过期：`SET PERSIST default_password_lifetime = 180;`，该变量默认值为 `0` 表示全局禁用自动过期（密码永久有效），设为 180 则所有用户在创建满 180 天后登录即被自动标记为过期状态；也可写入 `my.cnf`（Windows 下 `my.ini`）的 `[mysqld]` 模块持久化。
    * 用户级覆盖：`CREATE USER 'user_test'@'%' IDENTIFIED BY 'abc123' PASSWORD EXPIRE INTERVAL 90 DAY;` 指定 90 天后过期；`ALTER USER 'app_backend'@'%' PASSWORD EXPIRE NEVER;` 设为永不过期（常用于无人值守的后台持久连接服务账户）；`ALTER USER 'user_test'@'%' PASSWORD EXPIRE DEFAULT;` 重置为跟随全局默认规则。
    * > 来源: P106

* **密码重用限制策略（Reuse Policy）**
    * 基于更改次数的重用限制：全局参数 `password_history` 默认为 `0`（不开启历史比对）；用户级语法 `ALTER USER 'zhangsan'@'%' PASSWORD HISTORY 3;` 表示改密时不能与最近 3 次使用过的密码重复，服务端会在内部安全表中记录最近 3 次改密的哈希快照，第四次改密若改回前 3 次内任意口令将被直接阻断；恢复跟随全局用 `ALTER USER 'zhangsan'@'%' PASSWORD HISTORY DEFAULT;`
    * 基于时间窗口的重用限制：全局参数 `password_reuse_interval` 单位为天、默认为 `0`（不开启时间窗比对）；用户级语法 `ALTER USER 'zhangsan'@'%' PASSWORD REUSE INTERVAL 60 DAY;` 表示 60 天内不得重新循环使用过往密码；恢复跟随全局用 `ALTER USER 'zhangsan'@'%' PASSWORD REUSE INTERVAL DEFAULT;`
    * `password_history` 与 `password_reuse_interval` 二者任意一项被触发均会拦截，叠加 `validate_password` 强度校验组件即可构建企业级数据库凭据防护网。
    * > 来源: P106

### 7.4 权限授予、回收与四表鉴权

* **权限分类与分配准则**
    * `SHOW PRIVILEGES;` 返回包含 62 项权限的矩阵，详尽列出每项权限的名称、上下文生效级别以及功能说明。
    * 库/表级别典型特权：`SELECT`、`INSERT`、`UPDATE`、`DELETE`、`CREATE`、`DROP`、`INDEX`、`ALTER`，控制用户对特定数据库或数据表的结构定义（DDL）与数据读写（DML）。
    * 列级别典型特权：`SELECT(col1)`、`INSERT(col1, col2)`、`UPDATE(col1)`，允许用户仅对某张表中的特定字段进行查询或修改以屏蔽敏感列。
    * 过程/函数级别典型特权：`EXECUTE`、`ALTER ROUTINE`、`CREATE ROUTINE`，控制存储过程、存储函数以及触发器的定义与调用执行资格。
    * 生产分配四大准则：最小权限原则（只赋予满足业务正常运转的绝对最小权限集，能给只读的绝不给写，能给 DML 的绝不给 DDL）；严密限制登录主机（严禁无脑使用通配符 `%`，必须精准绑定具体业务微服务集群 IP 或堡垒机 IP）；强制凭据复杂度策略（配置高强度复合凭据并启用历史重用限制）；定期审计与动态收缩（审计长期闲置账户，转岗降职人员及时收回富余权限，废弃账户果断销毁）。
    * > 来源: P107

* **`GRANT` 语法、权限叠加与转授权限**
    * 标准语法 `GRANT 权限列表 ON 数据库名.数据表名 TO '用户名'@'主机名';`，示例 `GRANT SELECT, UPDATE ON dbtest1.* TO 'zhangsan'@'%';`
    * 权限控制的横向范围代表用户可以接触到的物理数据资产边界（`*.*` 全局、`dbtest1.*` 单库、`dbtest1.emp1` 单表）；纵向程度代表用户在既定数据资产上能够施加的操作深度（`SELECT` 只读、`UPDATE`/`DELETE`/`INSERT` 数据写、`DROP`/`ALTER` 破坏性结构变更）。
    * 未授权操作被拦截的报错形态为 `ERROR 1142 (42000): DELETE command denied to user 'zhangsan'@'localhost' for table 'emp1'`。
    * 多次针对同一用户执行 `GRANT`，底层权限计算方式为自动合并取并集（Union），新追加的权限绝不会冲掉或覆盖已有的既有权限。
    * `ALL PRIVILEGES ON *.*` 与超级管理员 `root` 的根本差异在于转授权限的能力：`root` 天然持有 `WITH GRANT OPTION` 特权，而仅有 `ALL PRIVILEGES` 的用户虽然自身可任意读写，却无法把自己拥有的权限再转授给第三个用户，执行 `GRANT SELECT ON ... TO 'wangwu'` 会被当场拒绝。
    * 需要下级管理员具备代为授权资格时必须在授权指令末尾显式追加转授标记 `GRANT ALL PRIVILEGES ON *.* TO 'lisi'@'%' WITH GRANT OPTION;`；`SHOW GRANTS FOR 'lisi'@'%';` 末尾是否包含 `WITH GRANT OPTION` 字样，是判定其是否具备管理特权转授能力的唯一标识。
    * > 来源: P107

* **`REVOKE` 语法与全量清空**
    * 标准语法 `REVOKE 权限列表 ON 数据库名.数据表名 FROM '用户名'@'主机名';`，授权使用目标介词 `TO`，回收使用源介词 `FROM`。
    * 示例 `REVOKE SELECT ON dbtest1.* FROM 'zhangsan'@'%';` 执行后 `SHOW GRANTS` 仅剩 `UPDATE` 与 `DELETE`，`SELECT` 已被精确剥离，再次执行 `SELECT * FROM emp1;` 会抛 `SELECT command denied`。
    * 彻底清空某用户特权：`REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'lisi'@'%';`
    * > 来源: P107

* **权限四表的层级与主键拓扑**
    * `mysql.user` 为全局层级，主键 `(Host, User)`，共包含五十余个字段，划分为身份认证区（`Host`、`User`、`authentication_string`、`plugin`）、全局权限区（`Select_priv`、`Insert_priv`、`Update_priv` 等，数据类型均为 `ENUM('N','Y')`、默认值 `'N'`）、安全通道区（`ssl_type`、`ssl_cipher`）、资源配额区（`max_questions`、`max_updates`、`max_connections`、`max_user_connections`）。
    * `mysql.user` 的全局权限位一旦被置为 `'Y'`，将作为超级全局权限穿透作用于该实例下的所有数据库、所有表。
    * 资源配额语义：`max_questions` 为每小时允许执行的 SQL 查询最大次数；`max_updates` 为每小时允许执行的数据修改操作（DML/DDL）最大次数；`max_connections` 为每小时允许发起的握手连接最大频次；`max_user_connections` 为该用户同时允许建立的最大并发活跃连接数。
    * `mysql.db` 为数据库层级，主键 `PRIMARY KEY (Host, Db, User)`；若普通用户仅在 `dbtest1` 上被赋予 `SELECT`，则其 `mysql.user` 中 `Select_priv` 仍为 `'N'`，而 `mysql.db` 中对应记录的 `Select_priv` 为 `'Y'`。
    * `mysql.tables_priv` 为数据表层级，主键 `PRIMARY KEY (Host, Db, User, Table_name)`；其 `Table_priv` 与 `Column_priv` 字段不再采用枚举，而是采用 `SET` 集合类型（`set('Select','Insert','Update','Delete','Create','Drop',...)`），支持在单列中以位图方式复合记录多个表级特权。
    * `mysql.columns_priv` 为数据列层级，主键 `PRIMARY KEY (Host, Db, User, Table_name, Column_name)`，是内核支持的最细粒度权限控制机制；存储过程与函数的执行权独立记录在 `mysql.procs_priv` 表中。
    * > 来源: P107

* **双阶段访问控制与漏斗鉴权**
    * 阶段一连接核实阶段（Connection Verification）：客户端发起 TCP 握手并递交身份凭据时触发，服务端优先在内存的 `mysql.user` 结构中检索 —— 网络 Host 匹配（检查发起连接的物理客户端 IP 是否能被用户配置的 Host 命中）与账号口令核验（提取 `authentication_string` 密文，通过绑定插件与客户端传输的握手哈希安全比对）；任意一项不吻合则连接在握手期即刻被终止并返回 `ERROR 1045 (28000): Access denied`。
    * 阶段二请求核实阶段（Request Verification）：连接确立后用户发送的每一条独立 SQL 都必须实时经过该阶段，遵循「由宽到窄、逐层穿透、短路退出」的漏斗鉴权模型。
    * 第一层全局权限核查 `mysql.user`：特权列已为 `'Y'` 则判定对该用户全库全表拥有全局权限，短路放行且绝不会再多余消耗 CPU 检索后续低层级权限表。
    * 第二层数据库级核查 `mysql.db`：全局表为 `'N'` 时下潜，检查目标数据库在当前用户下是否被显式赋予该操作特权。
    * 第三层数据表级核查 `mysql.tables_priv`：库级特权未开放时下潜，检查目标表的 `Table_priv` 集合是否包含当前操作指令。
    * 第四层数据列级核查 `mysql.columns_priv`：逐一比对该 SQL 语句中所投影或涉及的每一个具体字段名，只要有任意一个被访问字段未在授权清单中即刻终止执行。
    * 穿透全部四层仍未获许可则返回 `ERROR 1142: command denied to user` 越权报错。
    * > 来源: P107

### 7.5 角色抽象与激活机制

* **角色定义与底层存储**
    * 角色本质上就是一组权限的具名集合（A named collection of privileges），自身并不是用来直接登录服务器的实体账户，而是一个权限包容器；向角色批量添加（`GRANT`）或移除（`REVOKE`）各项库表级特权后，把角色授予某个具体用户时该用户便瞬间继承角色囊括的全部权限。
    * 类比：角色如同 Java 中的接口（Interface），声明行为规范但不能直接 `new Interface()` 创建运行时会话；用户如同具体的实现类（Class）与对象。
    * 创建语法 `CREATE ROLE '角色名'[@'主机名'];`，不指定主机名时默认缺省仍为 `'%'`，支持批量创建如 `CREATE ROLE 'manager'@'%', 'boss'@'%', 'admin'@'%';`
    * 角色复用 `mysql.user` 表的物理结构存储，`SELECT host, user FROM mysql.user;` 能看到刚创建的角色；但角色记录的 `account_locked` 标志位默认为 `'Y'` 且没有登录密码，系统严禁任何人直接拿角色名发起 TCP 登录握手。
    * > 来源: P108

* **角色生命周期四步操作**
    * 赋权 `GRANT 权限列表 ON 作用域 TO '角色名';`，例如 `GRANT SELECT, UPDATE ON dbtest1.* TO 'manager';`；`GRANT ALL PRIVILEGES ON *.* TO 'boss'@'%';`
    * 查验 `SHOW GRANTS FOR 'manager';` 返回 `GRANT USAGE ON *.* TO \`manager\`@\`%\`` 与 `GRANT SELECT, UPDATE ON \`dbtest1\`.* TO \`manager\`@\`%\`` 两行，角色容器内部规则清晰可见。
    * 回收 `REVOKE UPDATE ON dbtest1.* FROM 'manager';` 再次 `SHOW GRANTS` 会发现 `UPDATE` 已被剔除，角色容器中的规则被实时更新。
    * 销毁 `DROP ROLE 'admin';` 执行后对已删除实体执行 `SHOW GRANTS FOR 'admin';` 会立即报错提示该角色/授权不存在。
    * > 来源: P108

* **角色激活陷阱与两套解决方案**
    * 将 `manager` 授予用户：`GRANT 'manager' TO 'wangwu'@'%';`，`SHOW GRANTS FOR 'wangwu'@'%';` 显示已持有 `GRANT \`manager\`@\`%\` TO \`wangwu\`@\`%\``。
    * > 易错点：MySQL 8.0 中赋予给用户的角色默认处于「未激活状态（Inactive / Disabled）」，用户虽持有角色头衔，但角色包含的权限在连接建立后并没有自动注入当前鉴权上下文；此时 `SHOW DATABASES;` 依旧只能看到 `information_schema`，退出终端重登或执行 `FLUSH PRIVILEGES;` 都毫无变化。
    * 探针函数 `SELECT CURRENT_ROLE();` 在未激活态返回 `NONE`，明确告知当前没有任何生效的角色。
    * 方案一显式配置默认角色：`SET DEFAULT ROLE manager TO 'wangwu'@'%';`，被授予多个角色时可用 `SET DEFAULT ROLE ALL TO 'wangwu'@'%';` 全量激活；配置后用户 `quit` 退出并重新登录，`CURRENT_ROLE()` 返回 `` `manager`@`%` ``，`SHOW DATABASES;` 中 `dbtest1` 赫然在列，而执行 `DELETE FROM emp1 WHERE id = 2;` 仍被阻断并抛 `ERROR 1142: DELETE command denied to user 'wangwu'@'localhost'`，完全符合角色定义。
    * 方案二全局自动激活（生产推荐）：`SHOW VARIABLES LIKE 'activate_all_roles_on_login';` 默认值为 `OFF`；`SET GLOBAL activate_all_roles_on_login = ON;` 并可持久化写入 `my.cnf` 或 `my.ini` 的 `[mysqld]` 段；开启后任何用户登录时鉴权模块都会自动把其名下被授予的所有角色无感激活。
    * > 来源: P108

* **角色撤销与强制角色**
    * 撤销用户角色 `REVOKE 'manager' FROM 'wangwu'@'%';`，该指令必须由具有足够管理权限的账户（如 `root`）执行，普通用户自行执行会因权限不足抛出 `Access denied`。
    * 回收角色同样存在存量活跃会话的延迟生效特性：管理员执行 `REVOKE` 成功后，若该用户当前已连接的会话没有断开，他在当前会话中仍能继续执行 `manager` 赋予的查询操作；只有退出并重新建立连接后角色注销才真正生效。
    * 强制角色（Mandatory Roles）：在配置文件中配置 `mandatory_roles = 'audit_role,common_reader_role'`；该实例上创建的任何现有用户与未来新用户在创建瞬间会被系统自动强制附加这些角色，无需 DBA 执行任何手动 `GRANT` 关联；被列为 `mandatory_roles` 的角色无法通过 `REVOKE` 从用户身上剥离，也无法通过 `DROP ROLE` 删除。
    * 强制角色典型应用场景：全服强制挂载安全审计收集角色、公共字典视图只读角色，构建不可绕过的安全基线底座。
    * > 来源: P108

---

## 8. 配置体系与逻辑架构

> **一句话主旨**：掌握配置文件选项组的匹配与优先级规则、系统变量持久化路径，并建立三层逻辑架构认知。

* **配置文件两种语法格式**
    * 键值对类型（Key-Value）形式为 `key = value`，通过等号连接配置项名称与设定值，例如 `datadir = /var/lib/mysql`、`socket = /var/lib/mysql/mysql.sock`、`port = 3306`，直接决定服务器启动时的环境参数和基础路径。
    * 开关状态类型（布尔/开关标记）不附带具体值，其存在本身就代表开启或关闭意图，例如 `disable-log-bin` 表示禁用二进制日志功能。
    * 凡以 `#` 开头的文本行均视作注释，在服务端引导阶段不参与参数装载。
    * > 来源: P109

* **选项组（Option Groups）与启动程序的匹配规则**
    * `[server]` 是通用服务端选项组，作用于所有服务端程序（包括 `mysqld`、`mysqld_safe` 等）；`[client]` 是通用客户端选项组，作用于所有官方客户端程序（包括 `mysql`、`mysqldump`、`mysqladmin` 等）。
    * 特定版本专用选项组如 `[mysqld-5.7]` 与 `[mysqld-8.0]` 仅在对应主版本的 `mysqld` 启动时装载生效，用于部署多版本实例或规划版本平滑升级时精确隔离配置差异。
    * > 易错点：编写配置文件时切不可将客户端配置项与服务端配置项混为一谈。若把 `socket` 指定在 `[client]` 组中，所有本地客户端均遵循该套接字通信；但若服务端参数被误写在 `[mysql]` 组内，`mysqld` 启动时根本不会予以读取。
    * > 来源: P109

| 启动程序 / 二进制命令 | 默认读取的配置文件选项组 |
| :--- | :--- |
| `mysqld`（服务端守护进程） | `[mysqld]`、`[server]` |
| `mysql`（客户端命令行） | `[mysql]`、`[client]` |
| `mysqldump`（备份工具） | `[mysqldump]`、`[client]` |
| `mysqladmin`（运维工具） | `[mysqladmin]`、`[client]` |

* **优先级仲裁与命令行覆盖**
    * 同一参数在不同选项组被赋予不同取值时，MySQL 采用后声明覆盖（Last-Wins）的仲裁规则。
    * 示例：`[server]` 段声明 `default-storage-engine = InnoDB`，`[mysqld]` 段声明 `default-storage-engine = MyISAM`；由于 `mysqld` 启动时会同时扫描两个组且 `[mysqld]` 出现在 `[server]` 之后，最终生效的是 `MyISAM`；将二者物理声明次序颠倒则生效结果反转为 `InnoDB`。
    * 命令行中显式指定的启动选项拥有最高优先级，例如 `mysqld --default-storage-engine=MEMORY` 无论配置文件中最后一个选项组如何配置都会被完全覆盖。
    * > 来源: P109

* **系统变量的作用域与两种配置路径**
    * 全局系统变量（Global Variables）作用于整个 MySQL 实例级别，修改会影响所有后续新建立的会话连接，但对已经处于打开状态的既有连接通常不产生实时影响；大部分全局变量需要具备相应管理权限（如 `SYSTEM_VARIABLES_ADMIN` 或 `SUPER`）才可调整。
    * 会话系统变量（Session Variables）仅作用于当前特定的客户端连接会话，各客户端之间互不干扰、物理隔离，连接断开后会话级变量所分配的上下文环境即刻销毁。
    * 部分变量既有全局副本又有独立的会话副本（如 `autocommit`、`sql_mode`、`sort_buffer_size`）；另一些变量天然只具备全局属性（如 `datadir`、`max_connections`）或只具备会话属性。
    * 内存动态修改（临时性）：`SET GLOBAL max_connections = 1000;`、`SET SESSION sort_buffer_size = 4194304;` 立竿见影、无需重启，但 `mysqld` 进程重启后改动全部丢失、变量值重置回初始设定。
    * 配置文件固化（持久性）：将变量名（去掉前缀）直接写入 `my.cnf` 对应选项组，如 `[mysqld]` 段下的 `max_connections = 1000` 与 `sort_buffer_size = 4M`，服务重启后参数自动加载。
    * > 结论：运维实践中若需变更全局配置，常规操作是双管齐下 —— 先用 `SET GLOBAL` 立即修改内存变量使当前生产集群免于重启，再同步编辑写入 `my.cnf` 确保未来服务重启时不发生参数漂移。
    * > 来源: P109

* **逻辑架构的层次边界与交互模型**
    * MySQL 是非常典型的 C/S（Client/Server）架构：客户端形态多样，既可以是官方自带的交互式 CLI 程序 `mysql`，也可以是基于各主流开发语言构建的应用程序；服务端是以单进程多线程模型在后台运行的 `mysqld` 守护进程。
    * 常见的五层划分认知偏差在于把「客户端连接器（Connectors）」与「底层文件系统（Filesystem）」也划归 MySQL 内部层次；客户端 Connectors 属于数据库外部的第三方应用依赖，底层物理磁盘上的存储介质与操作系统文件系统属于宿主主机的存储基础设施。
    * 真正的数据库管理系统（DBMS）本体只包含中间三层，自上而下严格划分为：连接层、服务层（SQL 层）、引擎层。
    * > 来源: P109

* **连接层核心组件**
    * 连接池（Connection Pool）与线程调度机制维护客户端网络长连接、限制最大连接数（由 `max_connections` 参数硬性约束），为每一个接入的客户端连接分派或从线程缓存区（Thread Cache）调取一个专属工作线程承接该连接存活期间发送的所有请求。
    * 客户端正常断开连接时对应的服务器线程不会被暴力杀灭，而是重新归还到线程缓存中等待服务后续接入的会话。
    * 握手阶段的认证逻辑校验客户端提交的主机名、用户名与加密凭据，密码不正确会立即阻断握手并抛出 `Access denied for user 'xxx'@'xxx' (using password: YES)`。
    * 鉴权成功后连接层立即查询 `mysql.user`、`mysql.db`、`mysql.tables_priv`、`mysql.columns_priv` 等系统权限元数据表，获取该用户所拥有的全局及库表级权限，并将其深拷贝并绑定到该连接的专属工作线程上下文中；后续所有操作的权限校验直接读取内存中的权限副本，无需每次重复扫描磁盘系统表。
    * > 来源: P109

* **服务层四大部件**
    * SQL 接口（SQL Interface）充当服务层的门面与出入口调度器，负责直接接收客户端经由连接层传递进来的各类 SQL 文本指令（DDL、DML、DQL），并在各内部组件执行完毕后将打包好的状态码或二维结果集回传。
    * 解析器（Parser）的词法解析（Lexical Analysis）从左至右逐字扫描 SQL 字符串，将连续字符切分为不可分割的标记（Tokens），识别保留关键字、数据库对象名称与字面常量值或操作符；语法解析（Syntax Analysis）依据语法规范检查各个 Token 是否拼装组合成合法的语义结构，遇到 `FOM` 误拼或丢失逗号会立即拦截并抛出 `You have an error in your SQL syntax...`，最终在内存中构建出语法分析树。
    * 优化器（Optimizer）负责裁决驱动表选择、索引选型、子查询改写与连接降级等执行路径问题；逻辑优化（Logical Optimization）利用代数等价变换规则对 SQL 表达式重写化简（谓词下推、外连接转内连接、投影剔除），物理优化（Physical Optimization）依托统计信息基于成本代价模型（Cost Model）评估各种路径的 I/O 代价与 CPU 运算代价。
    * > 结论：优化器最终输出的产物是执行计划（Execution Plan）；优化器基于成本计算选定的方案代表其「自认为最优的策略」，在统计信息过时或存在数据倾斜时，执行计划所选的不一定绝对是实际物理耗时最短的路径。
    * 查询缓存（Query Cache）采用 Key-Value 字典模型，Value 保存查询返回的原始完整结果集，Key 是客户端发送给服务端的原始 SQL 文本字节序列，在词法语法解析之前优先计算哈希值并检索；命中则直接跳过后续的解析、优化和存储引擎调用。
    * > 来源: P109

* **引擎层与存储层**
    * 插件式存储引擎（Pluggable Storage Engines）是 MySQL 与一体化商业数据库最显著的架构差异：服务层与引擎层之间定义了标准的存储引擎 API 抽象规范（包含行读取、索引遍历、事务提交等抽象接口）。
    * 存储引擎是表级别（Per-Table）的，同一个数据库实例中 A 表可使用 InnoDB、B 表可使用 MyISAM、C 表可使用 MEMORY；`SHOW ENGINES;` 会明确罗列各引擎支持的事务性、安全保存点以及 XA 分布式事务等特性矩阵。
    * 存储层通过 `SHOW VARIABLES LIKE 'datadir';` 定位物理文件所在宿主目录（Linux 下默认 `/var/lib/mysql/`），该目录存放各数据表的物理数据与索引文件（InnoDB 的独立表空间 `.ibd`、系统表空间 `ibdata1`）以及 Redo Log、Undo Log、Binlog、慢查询日志等核心日志文件。
    * > 提示：CPU 与物理磁盘之间的 I/O 读写速度存在多个数量级的巨大鸿沟，因此存储引擎不会每次按需逐字节翻找磁盘，而是必须依托内存中的缓冲池（Buffer Pool），所有查找与更新优先在内存页中完成，再通过后台线程异步刷盘。
    * > 来源: P109

---

## 9. SQL 执行流程、查询缓存与缓冲池

> **一句话主旨**：走通 SQL 从连接到引擎的四关流水线，用双版本 Profiling 实测判定查询缓存存废，并落到缓冲池的页级机制。

### 9.1 执行流水线与四个关卡

* **SQL 全生命周期四关**
    * 第一关查询缓存（Query Cache）仅 MySQL 5.7 及更早版本支持；第二关解析器/分析器（Parser）做词法分析与语法分析并生成抽象语法树；第三关优化器（Optimizer）做逻辑优化与物理优化并输出执行计划；第四关执行器（Executor）做权限终审并调度存储引擎 API 执行数据扫描。
    * 引擎层动作：调用统一标准 API（如 `rnd_next` / `index_read`）、探查 Buffer Pool 内存缓冲按需触发磁盘 I/O、逐行比对过滤组装数据集回传至服务层。
    * 结果交付阶段：若 5.7 开启缓存则将 SQL 与结果集写入 Query Cache，再经由 SQL 接口将结果集响应给客户端。
    * > 来源: P110

```text
SQL 执行流水线：

  客户端
    | 1. 发起 SQL 查询请求
    v
  连接层       2. TCP 握手认证、权限绑定、指派专属工作线程
    |
    v
  服务层
    +---> [第一关] 查询缓存    仅 MySQL 5.7 及更早版本支持
    |         命中 -> 直接返回结果集
    |         未命中 / 8.0 直通
    |
    +---> [第二关] 解析器      词法分析 + 语法分析 -> 抽象语法树
    |
    +---> [第三关] 优化器      逻辑优化 + 物理优化 -> 执行计划
    |
    +---> [第四关] 执行器      权限终审 -> 调度存储引擎 API
    |
    v
  引擎层       3. 标准 API 调用  4. 探查 Buffer Pool  5. 逐行过滤回传
    |
    v
  结果交付     6. 5.7 写入 Query Cache  7. 经 SQL 接口响应客户端
```

* **第二关：解析器的词法与语法分工**
    * 词法分析像分词机把长字符串拆分为离散的单词片段（Token）并打标：哪些是系统关键字（`SELECT`、`FROM`、`WHERE`、`GROUP BY`）、哪些是用户自定义标识符（库名、表名、字段名、别名）、哪些是比较运算符与字面常量值。
    * 语法分析在词法分析基础上利用预设的上下文无关文法，判定 Token 排列顺序是否符合 SQL 语言规范；基础语法残缺时立即阻断并抛出 `ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '...' at line 1`。
    * 语义约束示例：`SELECT department_id, job_id, AVG(salary) FROM employees GROUP BY department_id;` 从词法上看每个单词都是标准词汇，但在严格 SQL 模式（`ONLY_FULL_GROUP_BY`）下，语法/语义检查器遍历分析树时会立刻捕获 `job_id` 既非聚合函数目标、又未包含在 `GROUP BY` 分组维度中的逻辑冲突。
    * 通过语法分析后原本平铺的字符串被重构为一棵多叉层次化的语法分析树（Parse Tree），精确界定语句骨架结构，使后续程序能够以此为根据进行代数改写与物理优化。
    * > 来源: P110

* **第三关：优化器的逻辑与物理优化**
    * 子查询解嵌套（Subquery Unnesting）：将性能极差的嵌套子查询在可能的情况下平铺重写为多表内连接（`JOIN`）或半连接（Semi-Join）。
    * 外连接消除（Outer Join Elimination）：若 `LEFT JOIN` 的 `WHERE` 条件中存在对右表非空列的强过滤，优化器会自动把性能较重的值补空外连接无损降级为更轻量的内连接（`INNER JOIN`）。
    * 常量折叠与谓词下推：提前对 `1 + 1` 等常量运算进行归并，尽可能把过滤条件（`WHERE`）推向底层提前剔除无效数据行。
    * 物理优化读取数据表上由后台线程采集的统计信息（表的数据页数、行记录数、索引基数 Cardinality 等），结合成本代价模型计算各候选方案的 I/O 成本与 CPU 成本；裁决议题包括单表访问路径（全表物理扫描与索引扫描的代价对比）、多索引竞选、多表关联的驱动顺序（Driving Table）。
    * > 来源: P110

* **驱动表选择的经典案例**
    * 查询 `SELECT * FROM test1, test2 WHERE test1.id = test2.id AND test1.name = '张伟' AND test2.course_name = 'MySQL高级课程';`，其中 `test1` 存放全校学生花名册、`test2` 存放选课详情，两表通过 `id` 关联。
    * `test1` 中叫「张伟」的记录重名率极高（约 500 条），`test2` 中选修该课程的记录全校仅有 3 条。
    * 方案 A 以 `test1` 为驱动表：先过滤出 500 条记录，再循环 500 次拿每条记录的 `id` 到 `test2` 比对；方案 B 以 `test2` 为驱动表：先过滤出 3 条选课记录，仅循环 3 次回到 `test1` 核验名字。
    * 优化器在物理优化阶段计算两种顺序的综合代价后仲裁：强制指定小结果集表 `test2` 作为驱动表，大结果集表 `test1` 作为被驱动表。
    * > 来源: P110

* **第四关：执行器的权限终审与两种检索模式**
    * 权限二次判定：正式驱动存储引擎检索数据之前，执行器校验当前连接绑定的用户凭据是否对目标库、目标表及目标字段拥有对应的 `SELECT`（或写操作）权限；若在会话存活期间管理员通过 `REVOKE` 剥夺了相关权限，执行器会立即中断执行并报错拦截。
    * 存储引擎 API 是一套严谨的虚函数/抽象接口规范，执行器并不感知底层的具体存储格式是 B+ 树、哈希表还是堆表，只负责依据算法调用预设的驱动原语。
    * 全表扫描模式（以 `SELECT * FROM test_user WHERE age = 25;` 为例）：调用 `ha_rnd_init()` 初始化读指针 → 调用 `ha_rnd_next()` 读取第一行 → 执行器调用服务层过滤组件判定 `age` 是否等于 25，不等则跳过、相等则推入临时结果集缓冲区 → 循环调用 `ha_rnd_next()` 要求读取下一行 → 不断重复直至存储引擎返回读取到表尾的终止状态码 → 把内存缓冲区中攒齐的所有满足条件行统一交由 SQL 接口打包成通信协议包回传客户端。
    * 索引扫描模式（`age` 字段上建有索引）：调用 `ha_index_init()` 初始化索引上下文 → 调用 `ha_index_read_map()` 要求存储引擎在 `age` 索引的 B+ 树上定位满足条件的「第一条记录」→ 引擎层完成 B+ 树遍历后将命中主键并回表拿到的数据整行提交给执行器 → 执行器继续调用 `ha_index_next()` 向后按序扫描索引链表直至引擎返回键值不匹配。
    * > 来源: P110

### 9.2 查询缓存（Query Cache）的机理与存废

* **字符级哈希匹配与不可缓存陷阱**
    * 查询缓存底层组织形式是简单的 Key-Value 哈希结构，Value 存储执行完毕的完整结果集，Key 则是客户端提交的原始 SQL 语句文本。
    * 字符级绝对一致要求：`SELECT employee_id, last_name FROM employees WHERE employee_id = 101;` 与在 `WHERE` 后多敲一个空格的同名语句因字符串哈希值直接改变，被判定为完全不同的两条语句而穿透缓存重新走解析和优化，缓存匹配鲁棒性极差。
    * 动态函数直接失效：SQL 语句中包含任何非确定性函数或系统变量（如 `NOW()`、`CURRENT_DATE()`、`UUID()`、`RAND()`）时，MySQL 在预处理阶段便判定该查询不可缓存，因为两次发送的字符串即使一模一样，当前时间戳也是动态跳变的。
    * > 来源: P110

* **缓存失效的连锁雪崩与 8.0 移除根因**
    * MySQL 规定只要某张数据表发生了一次数据变更（`INSERT`、`UPDATE`、`DELETE`、`TRUNCATE` 或 `ALTER TABLE`），该表在查询缓存中所关联的所有历史查询结果集必须立刻全量清空淘汰。
    * 在高度并发的在线事务处理（OLTP）场景下核心业务表每秒都在经历海量写入与更新，高频查询刚写入缓存不到半毫秒就因一条细微写入被整体驱逐；极端情况下查询缓存不仅无法加速，还会因全局互斥锁（Mutex）的高频加锁与哈希槽清理导致工作线程严重排队阻塞。
    * 查询缓存唯一的适用场景是几乎没有任何写入操作的静态字典表或配置表；MySQL 8.0 官方彻底从内核代码中移除了查询缓存。
    * 运行受系统变量 `query_cache_type` 控制：`0`（OFF）彻底关闭；`1`（ON）全局开启所有合规 `SELECT` 均尝试缓存；`2`（DEMAND）按需缓存模式，只有语句显式追加 `SQL_CACHE` 关键字时才启用缓存。
    * > 来源: P110

* **缓存健康度监控指标**
    * 5.7 中可通过 `SHOW STATUS LIKE 'Qcache%';` 检视状态；在 8.0 中执行 `SHOW VARIABLES LIKE 'query_cache_type';` 返回 `Empty set (0.00 sec)`，从底层参数层面证实查询缓存模块已被物理剔除。
    * > 来源: P110

| 状态参数名 | 技术含义 | 调优与排障诊断结论 |
| :--- | :--- | :--- |
| `Qcache_free_blocks` | 缓存中空闲内存块数量 | 持续偏高说明缓存碎片化极严重，需定期 `FLUSH QUERY CACHE` 整理 |
| `Qcache_total_blocks` | 缓存分配的内存块总数 | 反映缓存系统的物理块规模 |
| `Qcache_free_memory` | 剩余未使用的空闲内存量 | 过大说明分配给查询缓存的内存存在闲置浪费 |
| `Qcache_lowmem_prunes` | 因内存耗尽被迫淘汰的查询总数 | 持续暴涨说明分配内存严重不足、频繁发生驱逐 |
| `Qcache_queries_in_cache` | 当前驻留在缓存中的查询总数 | 实时监控指标 |
| `Qcache_not_cached` | 未被缓存的查询语句计数 | 统计不符合缓存要求或被规则排斥的语句量 |
| `Qcache_hits` | 缓存成功命中的历史累计总次数 | 越大说明缓存效益越显著 |
| `Qcache_inserts` | 新写入缓存的查询历史累计总次数 | `inserts` 极高而 `hits` 极低说明写得多用得少，属典型负优化 |

* **双版本 Profiling 实测对比**
    * Profiling 工具运行机制：MySQL 默认不会耗费 CPU 和内存记录每条 SQL 的内部执行明细，`profiling` 是会话级系统变量，默认值为 `0`（OFF）；`SET profiling = 1;` 仅对当前客户端连接会话生效，一旦断开重连该参数恢复为默认值 0。
    * 开启 profiling 后当前会话执行的所有 SQL 语句及其耗时都会被缓存到性能视图中（默认保留最近 15 条）。
    * 核心命令族：`SHOW PROFILES;` 列出近期执行的所有语句的 `Query_ID`、总耗时 `Duration` 与对应 SQL 文本；`SHOW PROFILE;` 展示最近执行的一条 SQL 的各阶段；`SHOW PROFILE FOR QUERY 7;` 显式查看指定 Query_ID 的各阶段耗时；`SHOW PROFILE CPU, BLOCK IO FOR QUERY 7;` 额外输出 `CPU_user`、`CPU_system`、`Block_ops_in`、`Block_ops_out`。
    * 8.0 实测：连续两次执行完全一致的 `SELECT * FROM employees;`（Query 6 与 Query 7），两者的执行轨迹高度一致，均完整走完 17 个内部步骤（`starting`、`Executing hook on transaction`、`starting`、`checking permissions`、`Opening tables`、`init`、`System lock`、`optimizing`、`statistics`、`preparing`、`executing`、`Sending data`、`end`、`query end`、`waiting for handler commit`、`closing tables`、`freeing items`、`cleaning up`），没有出现任何跳步。
    * 8.0 内核级验证：在 `/etc/my.cnf` 强行写入 `query_cache_type = 1` 后 `systemctl restart mysqld` 会立即抛出 `Job for mysqld.service failed because the control process exited with error code.`，`mysqld` 因遭遇无法识别的未知参数 `query_cache_type` 而拒绝引导，必须彻底删除该行配置才能恢复启动 —— 证明 8.0 不是默认关闭查询缓存，而是把整个查询缓存模块的底层代码全部铲除。
    * > 来源: P111

| 执行阶段对比项 | 首次执行（未命中缓存） | 二次执行（成功命中缓存） |
| :--- | :--- | :--- |
| 执行步骤总数 | 约 25 个微状态阶段 | 仅 4 ~ 5 个阶段 |
| 关键核心阶段 | `checking permissions`、`Opening tables`、`waiting for query cache lock`、`optimizing`、`executing`、`Sending data`、`storing results in query cache` | `starting`、`waiting for query cache lock`、`checking query cache for query`、`sending cached result to client`、`cleaning up` |
| 底层核心动作 | 经历完整优化与文件检索，最后一步执行 `storing results in query cache` 把结果集序列化写入哈希表 | 在 `checking query cache for query` 阶段判定命中，立即 `sending cached result to client` 返回，跳过表打开、优化器评估与存储引擎交互 |
| 整体执行耗时 | 约 0.000350s | 约 0.000040s，出现数量级骤降 |

* **5.7 缓存脆弱性对照实验**
    * 5.7 源码中虽保留查询缓存功能，但官方在默认安装包中同样将其置为关闭状态；在 `[mysqld]` 段追加 `query_cache_type = 1` 并重启后，`SHOW VARIABLES LIKE 'query_cache_type';` 回显 `Value` 变为 `ON`。
    * 空格注入测试：`SELECT * FROM departments WHERE department_id = 10;` 走完 25 个阶段并在末端执行 `storing results in query cache`；随后在 `WHERE` 后刻意敲入三个连续空格的语义等价语句并未命中缓存，而是重新完整走了一遍 25 个阶段、又在末端重新执行一次 `storing results in query cache`。
    * 等价逻辑变换测试：改写为 `SELECT * FROM departments WHERE department_id > 9 AND department_id < 11;` 因文本哈希完全匹配不上，缓存同样彻底失效。
    * > 结论：查询缓存以文本字符串作为硬性 Key 进行精确匹配，业务代码中只要存在动态拼接 SQL、不同开发者格式化风格不一（多一个缩进、大小写不同），命中率就会急剧劣化。
    * 5.7 生产调优建议：强烈建议将 `query_cache_type` 设置为 `2`（DEMAND 模式），普通查询默认统统不进缓存，从而避免频繁更新的高频表因缓存失效引发全局锁竞争；只有在明确知晓该表是静态字典表（如城市编码表、行业分类表）且查询量极大时，才由程序员在 SQL 中显式声明 `SELECT SQL_CACHE * FROM base_district_code;`
    * > 来源: P111

### 9.3 Oracle 执行流程对照与缓冲池机制

* **Oracle 共享池与软硬解析**
    * Oracle 中 SQL 处理管线为：语法检查（校验关键字拼写、括号配对）→ 语义检查（校验表名、字段名、视图是否存在）→ 权限检查（校验当前会话账户是否对目标对象拥有访问权限）→ 共享池检查 → 执行器调度与数据读取。
    * 系统全局区（SGA，System Global Area）中的共享池（Shared Pool）由两大子模块构成：库缓存（Library Cache）保存 SQL 文本及其对应的已编译执行计划；数据字典缓冲区（Data Dictionary Cache）在实例启动时把常用系统表及元数据预热并驻留，避免每次权限校验或字段核对都去物理磁盘扫描数据字典表。
    * > 对比：MySQL 5.7 的查询缓存存的是「SQL 文本与其返回的最终数据结果集」，而 Oracle 库缓存存的是「SQL 文本与其背后的执行计划」。
    * 硬解析（Hard Parse）：SQL 传入共享池后计算哈希值并在库缓存中检索，若从未见过这条语句，优化器必须介入经历完整的语义分析、多表驱动次序评估、索引选型、代价估算，最终生成一份二进制执行计划并连同 SQL 注册进库缓存 —— 这套重型编译过程开销极大、重度消耗 CPU。
    * 软解析（Soft Parse）：若在库缓存中精准命中了完全相同的语句，直接调取此前已编译好的执行计划交付执行器调度，彻底免去优化器的编译推演开销。
    * > 来源: P112

* **绑定变量（Bind Variables）的提效与代价**
    * 硬编码拼接参数 `SELECT * FROM orders WHERE order_id = 10001;` 与 `... = 10002;` 因数值物理变动，Oracle 判定为两条完全不同的独立 SQL，导致频繁重复硬解析。
    * 改用形如占位符的绑定变量 `SELECT * FROM orders WHERE order_id = :v_order_id;` 后，无论业务端把参数赋值为多少，发送到服务端的 SQL 模板与哈希值恒定不变，仅在首次执行时执行一次硬解析，后续成千上万次调用均可享受软解析的零开销提速。
    * > 提示：绑定变量的主要弊端在于数据倾斜下的执行计划固化 —— 若某列数据分布极不均匀（某个状态值占全表 99% 数据、另一个状态值仅占 1%），强制复用基于首次参数生成的执行计划，可能导致本该走全表扫描的查询误走索引，或本该走索引的查询误走全表扫描。
    * > 来源: P112

* **内存与磁盘的物理鸿沟**
    * CPU 寄存器与各级高速缓存（L1/L2/L3）的访问耗时在纳秒（$ns$）级别；物理内存（DRAM）的访问耗时在几十到几百纳秒之间；机械磁盘的随机寻址耗时在毫秒（$ms$）级别；即便是顶级的 NVMe 固态硬盘，其随机 I/O 耗时也在数十微秒级别。
    * 算法复杂度的对比全部建立在纯内存计算的基础之上；一旦某套运算逻辑跨越物理边界触发一次磁盘 I/O，哪怕是一个 $O(1)$ 的磁盘随机读取，其物理等待耗时也足以将内存中精心优化的算法优势彻底归零。
    * > 结论：数据库的核心优化目标永远是尽可能减少物理磁盘 I/O 的交互频次，数据库缓冲池（Buffer Pool）正是为抹平 CPU 与磁盘之间的速度鸿沟而生的最核心组件。
    * > 来源: P112

* **缓冲池与查询缓存的本质区别**
    * 混淆二者是常见认知错误，必须从所属架构层级、底层缓存对象、数据动态粒度、版本存废现状四个维度划清界限。
    * MyISAM 存储引擎天然只使用 `key_buffer_size` 缓存索引块，并不在内存中缓存纯数据行，数据读取依赖操作系统自身的文件系统缓存；InnoDB 的索引与数据全部由自身自治管理，核心调优参数为 `innodb_buffer_pool_size`。
    * > 来源: P112

| 对比维度 | 查询缓存（Query Cache） | 缓冲池（Buffer Pool） |
| :--- | :--- | :--- |
| 所属架构层级 | MySQL 服务层（Server Layer） | 存储引擎层（如 InnoDB 引擎层） |
| 底层缓存对象 | 纯文本 SQL 字符串 $\rightarrow$ 结果集（Key-Value 结构） | 磁盘底层的物理数据页（Data Pages）与索引页 |
| 数据动态粒度 | 粗粒度黑盒：表有写操作则整表缓存清空 | 细粒度控制：基于页的加载、修改、LRU 淘汰与刷盘 |
| 版本存废现状 | 表现鸡肋，在 MySQL 8.0 中已被官方彻底废除 | 各大数据库的核心基石，任何版本均重度依赖并持续演进 |

* **数据页（Data Page）与成批调度**
    * InnoDB 把底层表空间划分为连续的逻辑块，这个基本物理单元被称为数据页（Data Page），默认大小为 **16KB**。
    * 执行 `SELECT * FROM employees WHERE employee_id = 101;` 时，虽然只索要一条物理上可能仅占几十个字节的记录，存储引擎也绝不可能跑到磁盘上只把这几十个字节扣出来返回。
    * 存储引擎的最小 I/O 交互单位是页：必须将 101 号员工所在的整个 16KB 数据页完整读入内存中的缓冲池，随后由服务层在内存中过滤出该单行数据。
    * > 来源: P112

* **缓冲原则、预读与脏页刷盘**
    * 缓冲池的维护策略遵循「位置决定效率，频次决定留存」：使用改进的 LRU（Least Recently Used）链表管理物理页，优先把访问频次极高的热数据页（Hot Pages）常驻在缓冲池中，空闲空间不足时自动淘汰长期未被访问的冷数据页。
    * 局部性原理与预读特性：存储系统具有显著的空间局部性 —— 如果当前某数据页被读取，那么与其物理相邻或逻辑连续的前后数据页在极短时间内大概率也会被访问；因此 InnoDB 具备预读（Read-Ahead）机制，在加载当前 16KB 数据页时若判定处于连续读取状态，会异步预先将周围一批相邻页一并提前载入缓冲池。
    * 写操作执行逻辑：先检查数据所在页是否在 Buffer Pool 中 —— 若在则直接在内存中修改该数据页；若不在则先将磁盘数据页读入 Buffer Pool，再在内存中完成修改；此时 Buffer Pool 内存页已更新但磁盘文件尚未同步，内存与磁盘数据产生不一致，该页即成为脏页（Dirty Page）。
    * 严禁改完内存立刻同步写磁盘：一个数据页足足有 16KB，而一个 `UPDATE` 往往只是把某个用户的年龄从 20 改成 21，仅仅动了其中 1 个字节；若为同步这 1 个字节强迫操作系统发起一次随机 I/O 去覆写整个 16KB 磁盘物理块，高并发下磁盘会瞬间被 I/O 队列压垮。
    * 因此修改动作优先在内存完成，系统依托后台线程按照预设的 Checkpoint（检查点）机制，平滑、周期性地批量将脏页异步刷新到物理磁盘。
    * > 来源: P112

* **Redo Log 与 Undo Log 的诞生由头**
    * 异步刷盘撕开的安全漏洞是：内存中的脏页刚被修改、还没来得及刷盘，服务器突发停电或操作系统宕机，已提交的数据就会凭空蒸发；此外若一次包含多条修改的业务事务只刷盘了一半系统就崩溃，重启后无法把这半截脏数据彻底撤销。
    * Redo Log（重做日志）解决「刷盘未完成突发宕机」的问题：在修改内存数据页的同时，将物理修改以顺序追加写的方式极速记录到 Redo Log；只要 Redo Log 成功刷盘，即便内存脏页未来得及同步，实例重启时也能依据 Redo Log 进行前滚重放，保障数据的持久性（Crash-safe）。
    * Undo Log（回滚日志）解决「事务撤销回滚与原子性」的问题：在修改数据之前先将数据被修改前的原始镜像记入 Undo Log；若事务需要回滚或崩溃后需要清理未提交的半拉子操作，系统可依据 Undo Log 逆向还原旧值。
    * > 来源: P112

* **缓冲池容量调优与多实例拆分**
    * 8.0 中 `SHOW VARIABLES LIKE 'innodb_buffer_pool_size';` 回显 `134217728` 字节即 **128MB**；在配置 64GB 内存的独立数据库宿主机上 128MB 显然过于拮据，行业经典实践通常将物理内存的 **50% ~ 75%** 全权倾斜划拨给该参数。
    * 动态扩容 `SET GLOBAL innodb_buffer_pool_size = 268435456;` 可将缓冲池提升至 256MB，生产环境应同步写入 `/etc/my.cnf` 的 `[mysqld]` 段确保持久固化。
    * 多实例拆分动因：服务器硬件核心数向几十甚至上百核演进后，若整个数据库只维护一个庞大的 Buffer Pool 单实例，数以千计的业务线程在并发访问、换入换出数据页时都必须频繁申请并竞争同一个全局保护锁（Mutex），锁争用导致线程排队自旋、CPU 使用率居高不下而系统吞吐量停滞不前。
    * InnoDB 支持通过 `innodb_buffer_pool_instances` 将缓冲池逻辑切分为多个独立实例，每个实例独立管理自己的 LRU 链表、Flush 链表和互斥锁，系统默认值为 `1`。
    * > 易错点（1GB 约束硬门槛）：只有当 `innodb_buffer_pool_size` 的总大小大于或等于 1GB 时，设置多实例才被系统真正激活生效；如果总缓冲池容量小于 1GB（例如仅为 256MB），即便在配置中强行指定 `innodb_buffer_pool_instances = 4`，MySQL 初始化时也会强行将其重置回 `1`，因为微型内存体量下多实例引发的内存碎片及实例间管理元数据损耗远大于锁解耦带来的并发收益。
    * > 来源: P112
