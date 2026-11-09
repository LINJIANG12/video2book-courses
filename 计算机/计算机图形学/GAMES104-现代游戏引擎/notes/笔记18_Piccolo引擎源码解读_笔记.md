# Piccolo引擎源码解读

## 学习前提与工具链

* **Piccolo**

    * > **定义**：一款开源的迷你游戏引擎，麻雀虽小、五脏俱全。

* **三项前提假设**

    * 具备 C++ 语言基础；学过 GAMES104 或有游戏引擎知识背景；本机已配置好 C++ 与 Vulkan 开发环境——不满足则较难得到满意收获。

* **git（版本控制工具）**

    * > **定义**：管理不同人、不同时间、不同代码修改的工具。

    * 学习资源：官网官方教程（有中文版）；交互式学习网站 Learn Git Branching。

* **CMake（构建工具）**

    * > **定义**：控制代码编译流程的工具。

    * 官网有官方课程（仅英文版），GitHub 上可找中文课程；完全不了解也可跟着构建步骤照做，照样能编译和开发。

* **代码版本锁定**

    * 源码版本锁定 v0.0.9：熟悉 git 就把仓库拉下来 checkout 到该 tag；不熟悉就去 GitHub release 页面找到该版本，在 assets 里下载源码压缩包。

---

## 工程目录布局

* **根目录内容四分类**

    * 点开头的文件与文件夹：各平台、各工具的配置文件，在部分操作系统上默认隐藏，下载后看不到属正常。
    * CMakeLists.txt 与 CMake 脚本：指定工程如何构建。
    * 一批批处理文件：为方便构建引擎而写的小脚本，编译时用到。
    * license、readme、release notes：读代码前可简单看一下。
    * 除这四类，根目录只剩一个 engine 文件夹。

* **engine 文件夹**

    | 子目录 / 文件 | 内容 |
    | :--- | :--- |
    | third_party | 第三方库的源码或二进制库，直接嵌入工程与代码仓库以方便集成 |
    | asset | 引擎启动时默认关卡所需的一些资产 |
    | binary | 构建过程中生成、未加入版本仓库；内含 Piccolo parser，参与引擎构建：解析 C++ 源代码、按模板文件生成功能性代码 |
    | config | 编辑器启动所需的配置文件 |
    | jolt asset | 物理库 Jolt physics 所需的一些 shader |
    | shader | 引擎启动时所必需的 shader |
    | source | 引擎源码 |
    | template | Piccolo parser 生成代码所需的全部模板文件 |

    * 另有两个文件：.gitignore 指明需要忽略的文件夹（如 binary）；CMakeLists.txt 指定工程如何构建。

* **source 文件夹**

    * generated：Piccolo parser 生成的功能性代码。
    * editor：编辑器的所有源代码。
    * meta_parser：Piccolo parser 本身的源代码。
    * precompile：一个 CMake 脚本加一个配置文件——正是它们使构建引擎工程之前，都会先用 parser 解析一遍源代码、生成所需的功能性代码。
    * runtime：引擎最核心的代码，分层结构就落在这里。
    * test：目前基本为空，以后编写单元测试等代码放这里。

---

## runtime 分层与代码量分布

* **四个子文件夹对应四层架构**

    * core layer、function layer、platform layer、resource layer 四个子文件夹与分层架构一一对应。
    * 另有三个文件：CMakeLists.txt 指定 runtime 这个工程如何构建；engine.cpp 与 engine.h 是整个引擎的入口。

* **代码量分布**

    * runtime 共 31,481 行有效代码，分布极不均衡：function 一层独大（25,728 行），platform 仅 81 行。

    ```text
    runtime 代码量分布（条长 ∝ 代码量）
     ├─ function   25,728 行  ██████████████████████████
     ├─ core        4,674 行  █████
     ├─ resource      866 行  █
     ├─ platform       81 行  ▏
     └─ engine.cpp / engine.h（入口文件，共一百来行）
    ```

---

## 引擎入口与一帧流程

* **PiccoloEngine（引擎入口类）**

    * > **定义**：engine.h 里唯一的类，整个引擎的入口。

    * public 函数中：开头两个构造函数不用管；initialize 与 clear 目前是两个空的占位函数；真正要关注四个函数——startEngine、shutdownEngine、run、tickOneFrame。
    * 这个类做的最关键的两件事：启动与关闭 RuntimeGlobalContext 里的 system；每帧的 tick。

* **startEngine 与 shutdownEngine（互为镜像）**

    * startEngine：注册一些类型的元信息 → 调用 RuntimeGlobalContext 的 startSystem 函数 → 打印一条 log。
    * shutdownEngine：打印一条 log → 把所有 start 过的 system shutdown 掉 → 把所有类型元信息取消注册。

* **run 与 tickOneFrame**

    * run：取得「窗口要不要关闭」的信息，每轮循环据此决定是否 tickOneFrame。
    * tickOneFrame 顺序：先 tick 逻辑 → 计算 FPS → 用一行代码交换逻辑与渲染的数据 → tick 渲染 → 更新那些不便划分到逻辑或渲染、又需要更新的东西（物理的 debug renderer、窗口标题等）。

* **RuntimeGlobalContext（全局系统管家）**

    * > **定义**：装着一堆 system 与 manager 成员变量的类，其实例是一个 `g_` 开头的全局变量；startSystem 所做的事就是实例化并初始化这些 manager 与 system。

    * 以后有新的系统，就加到 RuntimeGlobalContext 里面去管理。

* **system 与 manager 的分工**

    * > **定义**：system 更多是底层的支持系统（窗口、输入、渲染等）；manager 更偏向引擎逻辑上的（物理系统、粒子系统等）。

* **逻辑实体的层级下钻**

    * 逻辑 tick 的次序：首先 tick world，然后 tick input（用户输入）。
    * world：引擎里所有逻辑上实体的载体，由一个一个 level 组成；world 还不存在时按指定路径的 world 资产去加载，加载完成后找到其中的 active level 去 tick。
    * level：一个 world 由多个 level 组成，但同时只有一个活跃 level；level 的 tick 会逐个更新它拥有的 game object，也会更新 active character 与 physics scene——其中最主要的是 game object。
    * game object：tick 就是逐个 tick 它身上挂载的所有 component；game object 本身没有任何功能，所有功能都由挂载的 component 实现，所以只需要更新它的 component。
    * 同一 game object 上不同类型 component 的调用顺序是不确定的（可先跑脚本再跑动画，也可相反）。

* **一帧主线**

    ```text
    PiccoloEngine（engine.h，引擎入口）
     ├─ startEngine / shutdownEngine
     │     └─ 启动 / 关闭 RuntimeGlobalContext 里的 system 与 manager
     └─ run（循环，直到窗口要关闭）
           └─ tickOneFrame（每帧一次）
                 ├─ 逻辑 tick ─► world ─► active level ─► game object ─► component
                 ├─ （交换逻辑与渲染的数据）
                 └─ 渲染 tick
    ```

---

## 编译运行与 Lua 组件改造

* **两步构建与运行**

    * 先用 CMake 以当前文件夹为工程根目录生成工程（生成后多出 build 文件夹，即工程所在），再构建 build 文件夹里的工程。
    * 批处理脚本 build_linux、build_macos、build_windows 分别在三个平台完成这两步。
    * 运行操作：按住右键转换视角；点击切换 game mode 可控制小白人移动。

* **Lua 与 sol2**

    * Lua：脚本语言库；sol2：帮助我们非常便捷地在 C++ 里执行 Lua 脚本的库，几行代码即可跑一段脚本。
    * 两个库都直接下载源码压缩包，解压到 engine/third_party 里面。

* **第三方库接入构建的两种形态**

    * sol2 文件夹自带 CMakeLists.txt：无需单独写脚本，在 third_party 的 CMakeLists.txt 里模仿已有库的写法 add_subdirectory 填上文件夹名即可。
    * lua 文件夹没有 CMakeLists.txt：需仿照现成库新建一个、简单修改，再在 third_party 的 CMakeLists.txt 里把 lua 添加进来。
    * 最后一步：在 piccolo runtime 的 CMakeLists.txt 里链接 lua_static 与 sol2；编译成功即说明两个库已成功链接。

* **LuaComponent（component 的最简形态）**

    * 放在 function/framework/component 文件夹下，模仿 animation component 新建 lua 文件夹编写；这对 .cpp 与 .h 可认为就是一个 component 最简单的形式。
    * 最前面固定写法：声明 LuaComponent 继承于 Component；同时需要 Piccolo parser 帮助生成这一类的反射代码以及序列化、反序列化相关的代码。
    * postLoadResource：反序列化生成这个类的实例的时候会调用的函数。
    * tick：每帧执行；验证阶段先在 tick 里打一条 log 看是否被调用。
    * 成员 m_lua_script（要执行的 Lua 脚本）加 `META(Enable)` 标记：提醒 Piccolo parser 帮忙序列化这个字段。
    * 真正跑脚本：在 postLoadResource 里做 sol2 初始化——为 lua state 添加一些基础函数（比如 Lua 里写的 print 函数），再用 lua state 执行脚本。

* **JSON 资产的反序列化链**

    * world 从 JSON 资产加载：默认 world 资产（hello_world.json）里写了默认 level，level 里有一些默认 objects，player 所在的资产路径写在旁边，顺着可找到 player 的资产文件。
    * player 身上挂了一堆 components（transform component、animation component 等），仿照这种形式添加 LuaComponent。
    * component 结构的两个字段：type_name 告诉反序列化工具要把这个结构反序列化成什么类型；context 是这个类型需要反序列化的字段。

    ```json
    {
      "type_name": "LuaComponent",
      "context": {
        "lua_script": "print(\"hello world\")"
      }
    }
    ```

    * > **易错点**：JSON 里的字段名不带 `m_` 前缀——序列化、反序列化工具会自动帮我们把这个前缀去掉。

    * 验证结果：直接运行看不到期望输出，切入 game mode 后日志持续刷屏——component 已成功挂到小白人身上并可以参与 tick。

---

## 反射系统编译期数据流

* **端到端流水线**

    ```text
          piccolo-parser 源码
                │  构建出 piccolo-parser 工具
                ▼
     CMake 创建 piccolo_runtime 工程
     （过程中收集 runtime 全部头文件的路径）
                │
                ▼
          生成 puzzle_header.h
     （内容就是 include 所有引擎源码的头文件）
                │
                ▼
     CMake 构建 runtime 之前，先构建 piccolo-precompile 工程
     （precompile 调用 piccolo-parser 解析 puzzle_header.h）
                │
                ▼
     用 clang 库拿到所有类的抽象语法树（AST），
     按类上打的标签筛选感兴趣的信息
     （比如需要反射的字段的类型、变量名等）
                │
                ▼
       根据收集到的信息生成 C++ 代码，
          全部写进 generated 文件夹
                │
                ▼
    生成的代码与 runtime 其他源码一起编译
              ──▶ piccolo_runtime
    ```

* **解析手段与产物**

    * 解析靠 clang 库：拿到所有类的抽象语法树（Abstract Syntax Tree，AST），再根据类上打的标签筛选感兴趣的信息——需要反射的字段的类型、变量名等。
    * puzzle_header.h 里包含的所有 piccolo runtime 的 .h 都会被解析。
    * > **注意**：真正进入 piccolo_runtime 发挥反射作用的，就是 generated 文件夹里面生成的代码。

---

## 生成代码的结构与注册入口

* **type operator 类（第一部分）**

    * 类与 JSON：取类名、通过 JSON 构造、转化成 JSON，共三个函数。
    * 基类相关：只有一个函数——取得基类的反射实例。
    * 字段（fields）：每个被 `META(Enable)` 标记的字段对应五个函数，名字都以被反射字段的名字结尾——取类型名、取变量名、set、get、是否是数组。
    * 是否是数组对 `std::vector` 容器做了特殊处理：如果是 `std::vector` 容器，这里就会返回 true。

* **typeWrapperRegister（第二部分）**

    * 函数名以类名作为后缀；先把字段的五个函数组成一个元组，注册进 field map；再把基类相关的函数注册进 class map。

* **typeWrappersRegister（第三部分）**

    * 简单套个娃，调用前面的 typeWrapperRegister；是留给外部统一调用的入口。

* **多类文件的推广**

    * 结构完全一样，只是各部分数量不同：有几个类就对应几个 operator 与几个 typeWrapperRegister；fields 里出现多组「五函数」时，typeWrapperRegister 里也要把这多组都注册进 field map。

* **reflection 目录与 meta_register**

    * 目录里除 all_reflection 外，其余文件都以 `.reflection.j.h` 结尾，且内部结构一致。
    * all_reflection 做两件事：include 所有以 `.reflection.j.h` 结尾的文件；在它唯一的 meta_register 函数里，调用那些文件里面的 typeWrappersRegister。
    * meta_register 就是 startEngine 里最先调用的函数：它被调用以后，收集到的所有类和字段的信息才真正进入那几个 map，供运行时使用。

---

## 反射标签与组件读写

### 反射范围标签

* **fields 与 whitelist fields（两种类级标签）**

    * `fields`：打此标签则所有字段都会反射——`motor_component_resource` 标记的是它。
    * `whitelist fields`：打此标签则只有标记了 `META(Enable)` 的字段才会参与反射——MotorComponent 类只标记了它（该类继承 Component，初始只有 motor_component_resource 一个字段被标 `META(Enable)`）。

* **META(Enable)（成员级标签）**

    * 打在字段上，提醒 Piccolo parser 反射并序列化该字段。
    * 要让 Lua 脚本能获取 MotorComponent 的 `is_moving` 字段，就给它补上 `META(Enable)`。

### set 与 get 的实现

* **接口形态与管道验证**

    * LuaComponent 新增 set 与 get 两个函数，都接收一个 game object 以及一个字段名。
    * 第一版只打印 log、把字段名打出来跑通管道；以 `set_float`、`get_float` 传给 Lua 虚拟机，其他类型也可以像这样添加。
    * 脚本逻辑：先判断 motor component 上的 `is_moving` 字段，再 set `motor_component_resource` 的 `jump_height`——正在移动设为 10，否则设为 5（让小白人在静止和移动状态下拥有不同跳跃高度）。
    * > **易错点**：不要忘了把 game object 也传递给 Lua 虚拟机——漏了它，脚本就拿不到目标组件。

* **真正实现的四步**

    * 取得所有的 components；
    * 将 field name 拆分，取出点号前面的第一个字符串，也就是 component 的名字；
    * 在所有 components 里遍历，找出名字等于这个 component 名字的那个 component；
    * 通过 component 的名字取得它的 TypeMeta，再经 meta 上的 fields list 取得该类型所有 field 的信息，从中找到名字等于要查找的 field 的那一个。

* **TypeMeta 与 FieldAccessor**

    * 两个封装类，为的是更面向对象地查询类型信息；结构很简单，就是帮忙查询前面那几个 map。
    * 取得目标 field 后，调用 FieldAccessor 的 set 写入字段、get 取得字段内容。
    * 效果：原地跳跳不过墙那么高，移动状态就可以跳过墙；随后小白人卡死在墙里——由此引出函数反射需求。

---

## 函数反射扩充

* **先定用法，再补实现**

    * 设想的用法：有一个标签叫 whitelist methods；给 `getUnstuck` 添加 `META(Enable)` 提醒反射系统反射这个函数；LuaComponent 里有一个 invoke 函数（跟 set、get 差不多）；同样把它传递给 Lua 虚拟机，再修改 Lua 代码，在尾部调用 motor component 的 getUnstuck 函数。

* **解析侧改动（piccolo-parser）**

    * 现有链路：main 函数处理参数并传给 parse 函数 → parse 里实例化一个 parser 并调用它的 parse → parse 用 clang 处理源码 → 最后调用 buildClassAST，在所有根节点里找到 class 与 struct 的声明，构造许多 class 传递到 schema module。
    * class 构造函数遍历 class 节点下所有子节点，对 base specifier 和 field declaration 特殊处理；每个 field declaration 构造一个 Field 类，把当前 field 声明的一些信息（名字、类型等）存储下来。
    * 仿照 Field 编写 Method 类：在判断标签的 isAccessible 逻辑里，仿照已有的 fields 与 whitelist fields 标签编写 methods 与 whitelist methods 标签。
    * 在 class 构造函数里添加对 method 的特殊处理，同时在 class 里添加一个 m_methods 成员。
    * 修改 shouldCompile 函数：原来一个类只有 fields 标记了才需要 compile，现在加入了 method，要同时认 methods。

* **生成侧改动（Mustache 模板）**

    * generate 函数里把 shouldCompileFields 替换成 shouldCompile；关键部分是把收集到的信息转换成代码生成库 Mustache 能够识别的数据结构，再根据模板渲染出代码。
    * 仿照 fields 模板编写 methods 模板；method 只需要两个函数——getMethodName 与 invoke。
    * 下面的注册部分同样模仿 field 的写法：把 methods 的两个相关函数注册到 map 里，再把 methods 相关的结构传递出去。
    * > **易错点**：Mustache 是大小写敏感的。

* **运行时侧补齐**

    * 添加 MethodFunctionTuple 与 MethodMap，同样把需要用到的宏实现一下。
    * 仿照 FieldAccessor 复制一份、修改出 MethodAccessor，让 method 用起来跟 field 一样简单。
    * 在 TypeMeta 里仿照 fields 提供一个 getMethodsList 功能，可以遍历一个 type 上面所有的 methods。
    * class 里添加了 m_methods 之后，还要仿照 m_field 的处理维护 m_methods 相关的代码。

* **field 与 method 全线对照**

    | 环节 | 字段反射（field） | 函数反射（method） |
    | :--- | :--- | :--- |
    | 类上打的标签 | fields / whitelist fields | methods / whitelist methods |
    | 成员上打的标签 | META(Enable) | META(Enable) |
    | 解析侧的类 | Field | Method（仿照 Field 编写） |
    | 生成的函数 | 类型名、变量名、set、get、是否是数组（五个） | getMethodName、invoke（两个） |
    | 注册去向 | 函数元组注册进 field map | MethodFunctionTuple 注册进 method map |
    | 运行时的封装 | FieldAccessor + TypeMeta 的 fields list | MethodAccessor + TypeMeta 的 methods list |
    | 编译开关 | shouldCompile 认 fields | shouldCompile 同时认 methods |

* **invoke 的目标分派**

    * 找目标类型的过程与 set、get 里的类似；但目标函数在 component 上时，不能像 get 和 set 那样直接调用 findComponentField，需要自己去找到对应的 component。
    * 判据是 target name 里有没有点：没有点——目标是 component 上面的一个成员函数，直接找到对应 component 再找它上面的函数；有点——目标是 component 上面的字段，调用 findComponentField 找到对应的字段后再对它 invoke。
    * 验证：getUnstuck logic 成功打印——成功使用反射调用到了 component 上面的函数。
    * 局限：目前函数反射只能反射最简单的一种函数——void 函数。

---

## 渲染系统与每帧流程

* **学习前提**

    * 需要对 Vulkan API 有一定的了解、具有编写简单 GLSL shader 的能力；推荐学习 RenderDoc，可更方便地进行渲染调试。

* **RenderSystem 的挂载与初始化**

    * 渲染系统叫 RenderSystem：在 global context 的 startSystems 里初始化，然后在 renderTick 当中每一帧 tick。
    * 初始化内容：render resource 中的一些全局贴图，render camera 与 render scene 中的一些光照，并初始化 render pipeline。
    * 小引擎把渲染的各个功能抽象为一个一个 pass，所以 pipeline 的初始化中会调用各个 pass 的初始化过程。

* **RenderSwapData（逻辑层送来的更新数据）**

    * 每帧 RenderSystem tick 首先调用 processSwapData，从逻辑层获得要更新的数据。
    * level resource description：指定新的天空盒与 color grading 的贴图。
    * game object resources 与 game objects to delete：分别指定要添加与删除的 game object。
    * camera swap data：更新相机参数。
    * 最后三项分别用于：创建新的粒子发射器、指定需要 tick 的粒子发射器、更新粒子发射器的位置。
    * processSwapData 剩下的代码，就是根据结构体里描述的需要更新的信息进行特定的更新。
    * > **提示**：这里所谓的 swap 并不是十分准确——只有渲染从逻辑拿数据，谈不上双向的「交换」。

* **一帧渲染的完整顺序**

    ```text
    renderTick（每帧）
     ├─ processSwapData        从逻辑层取要更新的数据
     ├─ prepareContext         调用图形 API 做渲染准备
     ├─ 准备每一帧 pass 共用的数据
     ├─ updateVisibleObjects   为各个 pass 筛选各自所需的具体数据
     ├─ preparePassData        调用各个 pass 的 preparePassData
     └─ 判断渲染模式
         ├─ 前向渲染：forward render → main camera pass 的 drawForward
         │                          → 依次执行各 subpass 的 draw
         └─ 延迟渲染：main camera pass 的 draw
                      → 一次几何绘制，物体几何信息写入 G-buffer
                      → 用 G-buffer 缓存的数据做光照计算
    ```

* **两种渲染模式**

    * 前向渲染：走 forward render 函数——先拿到渲染功能的对象，再执行 main camera pass 的 drawForward 函数；在 drawForward 中会执行各个 subpass 的 draw。
    * 延迟渲染：执行 main camera pass 的 draw——先执行一次几何绘制，将物体几何信息存到 G-buffer 中，然后利用 G-buffer 里缓存的数据进行光照计算。

---

## Pass 机制与 Color Grading

### 查找表与调色原理

* **color grading 查找表（look up table，LUT）**

    * > **定义**：color grading 实际上做的事情，就是把原来 RGB 色彩空间的一个颜色，转换到对应的、由这张贴图表示的色彩空间的颜色——一个普通的 RGB 颜色坐标，对应立方体里一个确定的颜色。

* **存储形态与逻辑形态**

    * 存储形态：一张长为 32×32（即 1024 像素）、高为 32 像素的贴图，由 32 张 32×32 的小贴图排成。
    * 逻辑形态：把这 32 张小贴图沿第三个维度堆成一个 32×32×32 的颜色立方体，即颜色转换的目标色彩空间。
    * 采样口诀：先「挑层」，再在层内取坐标。

    ```text
    存储形态：宽 1024（= 32 x 32）、高 32 的长条贴图
      [第 0 张] [第 1 张] [第 2 张] …… [第 31 张]    每一张都是 32x32
    逻辑形态：32 层小贴图沿第三个维度堆成 32 x 32 x 32 的颜色立方体
      （即 color grading 的目标色彩空间）
    ```

### 片段着色器与调试

* **两次采样实现**

    * 实现位置：shader 文件夹 glsl 目录里的 color_grading.fragment（片段着色器）；先把默认的 `out_color = color` 注释掉。
    * 要从一个三维的空间采样，至少需要从这张二维贴图上采样两次：先在这 32 张小贴图里面找出两张。
    * 找层方法：用 `color.b` 乘上查找表的高度——这个高度就是贴图里 32 张小贴图的层数；将这个值分别向上和向下取整，得到需要采样的两张小贴图的 index。
    * 再换算坐标，在这两张小贴图上分别根据 `color.r` 和 `color.g` 进行采样；最终把两个采样的结果 mix 一下，得到想要的目标颜色。
    * > **易错点**：只采样一次是不行的——只采样一次的结果多多少少是有些问题的。

* **RenderDoc 抓帧与 mipmap 修复**

    * RenderDoc 用法：在 launch 里选择 piccolo editor 启动，按 F2 抓取一帧，在左边命令列表里找到 color grading，查看它的 input 和 output。
    * 画面出现奇怪线条的根因：color grading 这张贴图默认产生了一些 mipmap，导致查找表最终的精度不够。
    * 修复：找到 mip levels 设置的地方，将值从默认的 0 改为 1，再运行即可达到预期效果。

### RenderPass 基类与 subpass

* **RenderPass 的重要成员**

    * global render resource 指针：记录了这个 pass 可能使用到的全局资源。
    * descriptor info：记录了输入、输出资源在 shader 中的布局信息。
    * render pipelines：记录了执行管线的信息。
    * visible nodes：记录了这个 pass 可见的物体——例如场景中可能有多个游戏对象，而该 pass 只会需要用到其中一部分。

* **subpass（Vulkan API 独有设计）**

    * > **定义**：Vulkan render pass 对象的一个部分；一个 Vulkan 的 render pass 可以包含多个串行的 subpass，前一个 subpass 的输出作为后一个 subpass 的输入。

    * color grading 被设计为 main camera pass 的一个 subpass；一个 enum 指定了 main camera 所有的 subpass，subpass 就是在这里被添加进去的。

* **ColorGradingPass 的类位置**

    * color grading pass 是 RenderPass 的一个子类，定义在 color_grading_pass.h 与 color_grading_pass.cpp 文件中，只有三个 public 的函数。

### 初始化四步与绘制

* **initialize 的四步**

    * initialize 接收一个 init info，由此拿到 color grading 查找表这张贴图，然后执行四步初始化。
    * setupDescriptorLayout：设置 pass 的 shader 对应的描述符布局，即 shader 中使用的输入输出资源的排布；用到两个资源——tone mapping pass 的输出与调色纹理。
    * setupPipelines：使用的 pipeline 是一条 graphics pipeline，主体部分为 vertex shader 和 fragment shader；还在此处设置视口、裁剪、朝向、剔除、混合模式、深度模板等必要信息；上面设置好的 descriptor layout 也会用于 pipeline 对象的创建。
    * setupDescriptorSet：有了资源描述符布局信息，还需设置实际使用到的资源；descriptor set 可以理解为资源 handle 的集合。
    * updateAfterFramebufferRecreate：当视口发生变化的时候，进行一些更新操作。

* **draw**

    * 在 main camera pass 的 draw 里面被调用；这个函数就是提交了一些 drawing 命令。

---

## 暗角 Pass 添加实战

* **复制改造与 subpass 登记**

    * 将 color grading pass 的两个文件复制一份，重命名为 vignette_pass，文件里的 color grading 全部替换成 vignette。
    * 在 main camera subpass 的 enum 里添加一个 vignette subpass；仿照 color grading 添加 vignette subpass 的一些设置和调用。
    * odd buffer、even buffer 是 subpass 之间的一些缓冲区，需要对后面的输出贴图引用进行替换。

* **dependency 维护**

    * 需要添加一个 dependency：数量从 8 改成 9。
    * vignette subpass 添加在 color grading subpass 之后，序号是 6，所以原来 color grading 后面的几个 pass 序号要重新修改。

* **编译与首跑踩坑**

    * 函数明明实现了仍报错：新添加的文件没有加入到构建——需要重新 cmake 一下。
    * 提示找不到 vignette fragment：复制 color grading fragment 直接改名；新建的 fragment shader 会在 CMake 构建流程中转换成一个 .h 文件，引用名要把小写改成大写。
    * 暗角效果其实可以不需要贴图实现：把 vignette resource 重新改回 color grading resource。
    * 首跑效果不符：理论上会进行两次 color grading（当时 vignette 的逻辑仍是再次调色）、画面应更明显，实际跟一次一样——RenderDoc 显示 vignette 的输入还是一个没经过 color grading 的贴图，是 subpass 之间传递贴图的 buffer 没有设置对；把 odd、even 修改后，两次调色都生效（画面变更暗）。

* **真正的暗角逻辑**

    * 修改 vignette fragment shader：相比 color grading 额外需要一个输入 uv；先随意设置两个参数。
    * 复制原来的 post process vertex shader，重命名为 vignette vertex shader，让它输出 uv；并把 pass 里的 shader 修改成 vignette vertex shader。
    * vignette pass 不需要贴图，所以把 descriptor layout 修改掉。
    * 结果：画面边缘有一些发黑——成功给小引擎添加了一个新 pass，实现了暗角效果。

---

## 已知局限与开放问题

* **全局变量式的系统管家**

    * RuntimeGlobalContext 让各系统可以在任意地方被方便地访问；这种方式有什么优缺点、有没有其他方式实现同样目的，是开放问题。

* **component 的执行顺序**

    * 不同类型 component 的调用顺序不确定会给游戏运行带来什么问题、如何让 component 有确定的执行顺序，是开放问题。

* **Lua 与引擎的边界**

    * 初版只实现了脚本的简单执行（每帧执行一次、不能调用引擎接口）；Lua 脚本的生命周期、与引擎各系统的交互方式如何设计，是开放问题。

* **函数反射的完全体**

    * 目前只能反射最简单的 void 函数；如何让所有类型的函数都能被反射，是开放问题。

* **更顺手的 Pass API**

    * 目前给小引擎添加 pass 还是比较繁琐；如何设计 API 让修改和添加内容更方便，是开放问题。

* **shader 资产化**

    * 目前 shader 在 CMake 构建流程中编译、硬编码进各个 pass；如何设计让 shader 作为一种资产、可以在运行时加载，是开放问题。
