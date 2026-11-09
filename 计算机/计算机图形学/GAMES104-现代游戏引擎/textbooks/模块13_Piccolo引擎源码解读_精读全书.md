# GAMES104现代游戏引擎·Piccolo引擎源码解读（第 13 册 / 共 14 册）

> **所属课程**：GAMES104现代游戏引擎  
> **本册内容**：Piccolo引擎源码解读  
> **覆盖范围**：P35 ~ P36-P37（共 3 讲 / 2 章 / 95 分钟音频）  
> **分册说明**：全书按内容分 14 册；本册为第 13 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. Piccolo 源码解读：从目录结构到第一次动手改造（P35）
2. Piccolo 源码解读：反射系统与渲染系统（P36-P37）

---

## Piccolo 源码解读：从目录结构到第一次动手改造
> 对应块：BLK59 | 覆盖分集：P35 | 块标题（分集名）：《Piccolo源码解读第一期：代码结构》

### 开场：一只五脏俱全的麻雀

欢迎大家来到 Piccolo 的源码解读。Piccolo 是一款开源的迷你游戏引擎，麻雀虽小，五脏俱全，而我是 Piccolo 的主要开发者之一。想学习更多游戏引擎知识的同学，可以关注我们的微信公众号 GAMES104，文字版的解读也会在公众号放出。废话不多说，我们直接开始。

先把前提说清楚，这次代码讲解有三个假设：第一，你有 C++ 语言基础；第二，你学过 GAMES104，或者有游戏引擎相关的知识背景；第三，你有一台已经配置好 C++ 和 Vulkan 开发环境的电脑。如果不满足这三个假设，可能会比较难得到满意的收获。

### 先备好两样工具：git 与 CMake

Piccolo 的源码目前托管在 GitHub 上，我们也会尝试提供更多途径，让大家获取 Piccolo 的代码。它使用的版本控制工具是 git，构建工具是 CMake。所谓版本控制工具，就是用来管理不同人、不同时间、不同代码修改的；而构建工具是用来控制代码编译流程的。这两个工具非常强大和流行，可以帮助我们更好地协作和构建项目，我建议不了解的同学学习一下。

资源也顺带指两个：git 官网上能找到官方教程，有中文版本；我个人非常推荐一个叫 Learn Git Branching 的网站，一个交互式学习 git 的网站，同学们可以上去玩一玩。CMake 同样可以在官网找到官方课程，但只有英文版，如果觉得阅读有压力，可以在 GitHub 上找一找中文的 CMake 课程。当然，完全不了解这两个工具也没有关系，一步一步跟着后面的步骤操作，你照样能编译和开发 Piccolo。

### 拿到代码：锁定 v0.0.9

本次代码讲解使用的代码版本是 v0.0.9。如果你熟悉 git，可以直接把仓库拉下来，checkout 到这个 tag；如果不熟悉也没有关系，直接去 GitHub 的 release 页面找到 v0.0.9 这个代码版本，在 assets 里下载源码压缩包。下载完成、解压之后，你会得到这样一个文件夹。为了方便阅读，我们用 VS Code 打开它。

### 根目录里都有什么

我们可以把根目录下的这些文件和文件夹，简单地分一个类。

第一类，是所有点开头的文件和文件夹。它们是各个平台、各个工具的配置文件；而且这类文件在一些操作系统上默认是隐藏的，所以下载下来之后如果没看到它们，不需要担心。第二类，是 CMake 的 CMakeLists.txt 和 CMake 脚本——Piccolo 就是用 CMake 作为构建工具的。第三类，是一批批处理文件，这些是为方便大家构建 Piccolo 而编写的小脚本，后面编译时会用到。第四类，是 license、readme 和 release notes，读代码之前可以简单看一下。

除了这四类以外，根目录就只剩下一个 engine 文件夹了，我们展开来讲一讲它里面的内容：

| 文件夹 / 文件 | 里面放的是什么 |
| :--- | :--- |
| `third_party` | 小引擎用到的第三方库的源码或二进制库。为了方便大家集成，第三方库被直接嵌入到了工程和代码仓库里 |
| `asset` | 引擎启动时默认关卡所需要的一些资产 |
| `binary` | 构建过程中生成的文件夹，没有加入版本仓库，所以在文件管理器里它是灰色的。这里面放的是 Piccolo parser：它会参与到引擎的构建过程中，解析 C++ 源代码，根据模板文件生成一些功能性的代码 |
| `config` | 编辑器启动时所需要的配置文件 |
| `jolt asset` | 物理库 Jolt physics 所需要的一些 shader |
| `shader` | 引擎启动时所必需的 shader |
| `source` | 引擎源码，下一节展开讲 |
| `template` | Piccolo parser 生成代码时所需要的模板文件，全部都在这里 |

这个文件夹里还有两个文件：`.gitignore` 指明了需要忽略的文件夹，比如前面提到的 binary；`CMakeLists.txt` 指定了我们的工程是如何构建的。

### source：编辑器、解析器与运行时

打开 source 文件夹，看看里面都有什么东西。

第一个子文件夹 `generated`，存放的就是前面提到的 Piccolo parser 生成的功能性代码。第二个是 `editor`，编辑器的所有源代码都存放在这里。`meta_parser` 里是 Piccolo parser 的源代码。`precompile` 是一个 CMake 脚本和一个配置文件——正是这两个文件，使得我们在构建引擎工程之前，都会先用 Piccolo parser 去解析一遍源代码，生成我们所需要的功能性代码。

然后是 `runtime` 文件夹，这就是引擎最核心的代码所在——大家可以看到，课程当中提到过的分层结构已经出现在这里了。`test` 文件夹目前基本是空的，以后我们要编写单元测试或者其他测试的代码，都会放在这个文件夹里面。最后一个 `.gitignore`，同样指定了哪些文件不要加入到版本仓库里。

接下来我们主要关注 runtime 下面的引擎核心代码；如果你对编辑器等其他部分感兴趣，可以自行阅读。

### runtime：分层架构落到了四个文件夹上

runtime 文件夹下面一共有四个子文件夹和三个文件。这四个子文件夹，分别对应我们在第三课学到的分层架构：core layer、function layer、platform layer 和 resource layer。剩下的三个文件里，`CMakeLists.txt` 指定了 runtime 这个工程是如何构建的；`engine.cpp` 和 `engine.h` 则是整个引擎的入口。

为了让大家对整个代码有一个更直观的感受，可以把 runtime 文件夹下所有文件的代码量画成一张矩形树图，每一个矩形的大小，代表这个文件所拥有的代码量。按比例摊开是这样：

```text
runtime 一共 31,481 行有效代码
 ├─ function   25,728 行  ██████████████████████████
 ├─ core        4,674 行  █████
 ├─ resource      866 行  █
 ├─ platform       81 行  ▏
 └─ engine.cpp / engine.h（入口，其余的一百来行）
```

最大的是 function 文件夹，拥有 25,728 行代码；第二大的是 core 层，4,674 行；然后是 resource 的 866 行；最后 platform 只有 81 行；剩下的就是 engine.cpp 和 engine.h 两个入口文件。

### 编译、运行，先玩一把

现在我们来编译运行一下 Piccolo。首先用 CMake 生成工程——这行命令的意思是以当前文件夹为工程根目录，然后生成工程；生成之后可以看到多出来一个 build 文件夹，这就是我们工程所在的地方。接着再用一条命令，去构建 build 文件夹里面的工程。（这里我简单使用了一下时间压缩的魔法，大家可以看到，引擎"瞬间"就编译完成了。）

前面提到过，为了方便大家构建工程，我们编写了一些批处理脚本：`build_linux`、`build_macos` 和 `build_windows`，分别就是在三个对应平台上完成刚才那两步操作的脚本。

然后我们可以把引擎跑起来看一看：按住右键可以转换视角；点击切换 game mode，可以控制小白人的移动。相信大家已经完全搞明白 Piccolo 应该怎么玩了。接下来，我们就看一看 Piccolo 是如何跑起来的。

### 引擎入口：engine.h 里只有一个类

介绍目录结构的时候提到过，engine.h 和 engine.cpp 两个文件是整个引擎的入口。打开 engine.h 可以看到，整个文件当中只有 PiccoloEngine 这一个类，而这个类又只有这么几个 public 的函数：开头两个构造函数我们先不管；initialize 和 clear 目前是两个空的占位函数。也就是说，我们只需要关注四个函数：startEngine、shutdownEngine、run 和 tickOneFrame。

先看 startEngine：它首先注册了一些类型的元信息，然后调用 RuntimeGlobalContext 的 startSystem 函数，最后打印一个 log。与它相对应的 shutdownEngine 刚好相反：先打印一个 log，然后把所有 start 过的 system shutdown 掉，最后再把所有的类型元信息取消注册。

再看 run 函数：它首先取得"窗口要不要关闭"的信息，然后在每一轮循环里，据此决定是否去 tickOneFrame。而 tickOneFrame 正是我们要说的第四个函数，它看起来有点复杂，我们马上展开。

所以，PiccoloEngine 类做的最关键的两件事情，就是：启动和关闭 RuntimeGlobalContext 里面的 system，以及每帧的 tick。

### RuntimeGlobalContext：全局变量里的系统管家

刚才反复出现的 RuntimeGlobalContext，它的实例是一个 `g_` 开头的全局变量。这个类里面有一堆 system 和 manager 的成员变量，而 startSystem 这个函数所做的事情，就是去实例化和初始化这些 manager 和 system。

那为什么有一些是 manager，有一些是 system？

> **定义**：system 更多是底层的支持系统，比如窗口、输入、渲染等；而 manager 更偏向于引擎逻辑上的，比如物理系统、粒子系统等等。

所以如果我们以后有新的系统，就可以加到 RuntimeGlobalContext 里面去管理。

### 一帧之内：先逻辑，后渲染

如果你知道 Piccolo 引擎是逻辑和渲染分离的架构，tickOneFrame 就很好理解了：它首先去 tick 逻辑，然后计算了一下 FPS；中间有一行代码，负责交换逻辑和渲染的数据；然后再去 tick 渲染。至于后面这些，就是一些不便划分到逻辑或者渲染里面、又需要更新的东西，比如说物理的 debug renderer 和窗口的标题等等。

渲染的 tick 这一次我们不展开讲，先看看逻辑里面的 tick。

### 逻辑的骨架：World、Level、GameObject、Component

逻辑的 tick，首先去 tick world，然后是 tick input，也就是用户输入。

world 是引擎里所有逻辑上的实体的载体，它由一个一个 level 组成。代码里的处理是这样的：如果 world 还不存在，就按照指定路径的 world 资产去加载；加载完成之后，找到其中的 active level，再去 tick 这个活跃的 level。而在 level 的 tick 中，它会逐个去 tick 它所拥有的 game object，也会更新 active character 和 physics scene——这其中最主要的是 game object。

而 game object 的 tick 非常简单，就是 tick 它身上挂载的所有 component。在 Piccolo 引擎当中，game object 本身是没有任何功能的，它所有的功能，都由身上所挂载的 component 实现。

到这里，可以把这一讲的主线完整串起来了：

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

读图：入口类管启动、关闭与每帧循环；一帧之内先逻辑、后渲染，中间交换数据；逻辑这一侧层层下钻，最终落到 component 的更新上。

> **结论**：Piccolo 引擎由 engine.h 中的 PiccoloEngine 类作为入口，启动和回收由 RuntimeGlobalContext 管理的 system 和 manager，并每帧更新逻辑和渲染。逻辑 tick 的主体是 world 的 tick：一个 world 由多个 level 组成，但同时只有一个活跃的 level；level 最重要的组成部分是 game object；每个 game object 本身没有功能、不需要更新，其功能由它身上挂载的不同 component 实现，所以只需要更新它的 component。

### 动手：十分钟让小引擎跑起 Lua 脚本

光讲代码没什么意思，很多同学可能早就已经手痒了。根据今天所提到的内容，我们可以简单动手修改一下小引擎——比如挑战一下：用 10 分钟，让一个小引擎为我跑脚本。要做的事情是：下载两个第三方库，构建它们，将它们链接到 piccolo runtime，然后编写一个新的 LuaComponent 类，把它的实例添加到小白人身上，让它参与 tick，每帧执行 Lua 脚本。

#### 两个第三方库：Lua 与 sol2

首先需要下载两个会用到的第三方库。第一个是 Lua；第二个库是 sol2，它可以帮助我们非常便捷地在 C++ 里执行 Lua 脚本——像示例显示的那样，只需要几行代码，就可以跑一段 Lua 脚本。两个库都直接下载源码压缩包就好，然后解压到 Piccolo 的 engine/third_party 里面。可以看到，文件夹里多了这两个目录。

#### 接进工程：从解压到链接

接下来需要构建这两个第三方库。先观察 sol2 这个文件夹：它拥有一个 CMakeLists.txt，也就是说，我们并不需要单独为它编写 CMake 脚本，只需要在这里 add_subdirectory 就可以了——模仿文件里已有库的写法，填上 sol2 文件夹的名字就好。

再看 lua 文件夹：里面没有 CMakeLists.txt，也就是说，我们需要单独为它编写一个。可以仿照现成库的写法新建一个，简单修改一下；然后同样要在 third_party 的 CMakeLists.txt 里把 lua 添加进来。

还有最后一步，就是将这两个库链接到 piccolo runtime 上：找到 piccolo runtime 的 CMakeLists.txt，在这里链接 lua_static 和 sol2。程序编译一下——好的，编译成功了，就说明这两个库已经成功链接到了 piccolo runtime。

在编写代码之前，我们先把所做的修改提交一下吧。

#### 写一个最简的 LuaComponent

接下来编写新的 LuaComponent 代码，并把它的实例添加到小白人身上。我们把代码放在 function/framework/component 文件夹下面：模仿 animation component，新建一个 lua 文件夹，再照着 animation component 简单编写——首先把多余的代码去掉，然后添加需要的头文件。可以认为，这个 .cpp 和 .h 就是一个 component 最简单的形式。

最前面四行是固定的写法，声明 LuaComponent 继承于 Component；同时我们需要 Piccolo parser 帮助我们生成这一类的反射代码，以及序列化、反序列化相关的代码。然后是 postLoadResource——它是在反序列化生成这个类的实例的时候，会调用的一个函数；再然后是 tick。

接下来添加一个成员：我们将要执行的 Lua 脚本（m_lua_script）。并给它添加这样一个标记：`META(Enable)`。这个标记会提醒 Piccolo parser，需要帮助我们去序列化这个字段。

#### 把组件挂到小白人身上

前面提到过，我们的 world 是从一个 JSON 资产加载进来的。asset 文件夹里有这样一个默认的 world：`hello_world.json`。它里面写了一个默认的 level；这个 level 里有一些默认的 objects；这些 objects 里有一个 player，player 所在的资产路径就写在旁边——顺着它，我们可以找到 player 的资产文件。

这个 player 身上挂了一堆的 components：transform component、animation component，等等。我们仿照这种形式，添加一个我们的 LuaComponent。这个结构有两个字段：type_name 和 context。type_name 会告诉我们的反序列化工具，要把这个结构反序列化成一个什么类型——比如我们希望它反序列化成一个 LuaComponent；context 就是这个类型需要反序列化的字段：

```json
{
  "type_name": "LuaComponent",
  "context": {
    "lua_script": "print(\"hello world\")"
  }
}
```

> **易错点**：JSON 里的字段名不带 `m_` 前缀——序列化、反序列化工具会自动帮我们把这个前缀去掉。

添加完这个结构，理论上在加载 world 的时候，就会在 player 身上添加我们的 LuaComponent。

#### 先验证：一条日志就够了

我们可以在 LuaComponent 的 tick 里面添加一个 log 函数，看一看是否有效果——直接将 lua script 打印出来（为此需要添加一个日志相关的头文件）。来编译运行一下：编译成功了，引擎也已经跑起来了，但是好像没有我们期望的输出。试一下切换到 game mode——可以看到下面已经疯狂输出这些 "print hello world"。

也就是说，我们的 component 已经成功地挂载到小白人身上，并且可以参与 tick 了。

#### 真正跑起来：sol2 登场

那么接下来就是真正的最后一步：把我们的 Lua 脚本真正跑起来。首先添加前面加进来的 sol2 库的头文件；然后在 postLoadResource 里面添加 sol2 的初始化——这行代码的意思是，为这个 lua state 添加一些基础的函数，比如说我们在 Lua 里面写的 print 函数。

接着把刚才那条 log 注释掉，改用 lua state 去跑我们的脚本。再编译一次看看：切换到 game mode，可以看到命令行这里在疯狂输出 hello world——这个 hello world，就完全是由 Lua 脚本输出来的了。也就是说，我们已经成功在小引擎里面运行 Lua 脚本。

### 这一讲做了什么

这次我们了解了 Piccolo 的目录结构、基础的 tick 流程；然后通过给小引擎添加一个新的 LuaComponent，简单介绍了如何修改 Piccolo 工程、添加第三方库、JSON 资源文件如何反序列化成实例，以及引擎 component 的基本形式。

因为篇幅有限，我们只能指出引擎中关键的部分，帮助大家定位和理解；具体的细节，还是需要大家亲自去阅读源码。

### 留给你的三个讨论题

在开发引擎的时候，除了思考代码本身，我还有很多问题想和大家一起讨论：

- **全局变量式的系统管家**：小引擎使用全局变量 RuntimeGlobalContext，可以在任意地方方便地访问各个系统。这种方式有什么优缺点？有没有其他方式能实现同样的目的？
- **component 的执行顺序**：game object 上不同类型的 component，调用顺序是不确定的——比如一个 player 可以先跑脚本再跑动画，另一个先动画后脚本。这会给游戏的运行带来什么问题？如何才能让 component 有确定的顺序去执行？
- **Lua 与引擎的边界**：这次我们只实现了 Lua 脚本的简单执行——目前 Lua 脚本每帧执行一次，而且还不能调用任何引擎的接口。你会怎么设计 Lua 脚本的生命周期，以及它与引擎各个系统的交互方式？

这三道题都没有标准答案，欢迎对它们感兴趣的同学在弹幕留言，或者加入 Piccolo 开发者频道一起讨论；也欢迎任何对源码讲解内容与形式的建议。

> **承前启后**：上一节讲完「Piccolo 源码解读：从目录结构到第一次动手改造」，下一节接着讲「Piccolo 源码解读：反射系统与渲染系统」。

---

## Piccolo 源码解读：反射系统与渲染系统
> 对应块：BLK60 | 覆盖分集：P36-P37 | 块标题（分集名）：《Piccolo源码解读第二期：反射系统》

大家好，我是靓仔。上一期结束后，很多同学反馈希望有反射系统的代码讲解，应同学们的要求，这一期我们就来讲 Piccolo 引擎中的反射系统。反射系统的实现细节比较繁琐，所以同学们可要耐心，一步一步听完，才能搞明白它究竟是怎么工作的。至于反射系统在引擎中扮演的角色、Piccolo 小引擎中反射系统实现的大致思路，GAMES104 第 14 期的反射部分都有讲解，这里就不再赘述。这一讲聚焦三件事：反射系统的使用方式、具体实现，以及怎么修改和扩充它；讲完反射，我们还会带上后面一讲的渲染模块——梳理渲染流程、以 color grading 为例认识 pass，最后亲手给小引擎添加一个暗角 pass。

### 反射系统是怎么工作的

#### 编译期的数据流：从标签到 generated 代码

先看这张数据流图，它展现了 Piccolo 反射系统从编译、收集信息到生成代码的整个流程：

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

流程读下来是这样的：首先从 piccolo-parser 的源码构建出 piccolo-parser 工具；在 CMake 创建 piccolo_runtime 工程的过程中，我们会收集到 piccolo_runtime 的所有头文件路径，收集完成后通过这些路径生成 `puzzle_header.h`——它的内容就是 include 所有引擎源码的头文件。然后，CMake 在构建 piccolo_runtime 工程之前，会先构建 piccolo-precompile 工程；在构建 precompile 工程的过程中，会调用 piccolo-parser 去解析 `puzzle_header.h`，这个文件里包含的所有 piccolo runtime 的 .h 都会被解析。解析的方式是通过 clang 库拿到所有类的抽象语法树，再根据我们在类上打的标签，筛选出我们感兴趣的信息，比如一个类需要反射的字段的类型和变量名等。下一步是根据这些收集到的信息生成 C++ 代码，所有代码都会生成到 generated 文件夹里；最后这些生成的代码与 piccolo_runtime 的其他源码一起，编译出 piccolo runtime。

> **结论**：这一路看下来，真正进入 piccolo_runtime 发挥反射作用的，就是 generated 文件夹里面生成的代码。

#### 生成代码的三个组成部分

我们看看这些生成的代码是什么样的。整个文件由三个部分组成：首先是一个 type operator 类，然后是一个 typeWrapperRegister 注册函数，把 operator 中的一些函数收集了起来；再下面是一个 typeWrappersRegister 注册函数，调用前面这个注册函数。

先对照原来的 motor_component.h：这个文件里只有 MotorComponent 一个类，这个类只有 `motor_component_resource` 这一个字段标记了 `META(Enable)`，它继承于 Component。

再仔细看生成的文件。operator 类里的函数可以分成三类：

- **类与 JSON**：取类名、通过 JSON 构造、转化成 JSON 的这三个函数；
- **基类相关**：就一个函数——取得基类的反射实例；
- **字段（fields）**：五个函数，名字都以被反射字段的名字结尾，对应 MotorComponent 类中唯一被标记了 `META(Enable)` 的那个字段。这五个函数也很简单：取类型名、取变量名、set、get，以及是否是数组。我们对 `std::vector` 容器做了特殊处理——如果是 `std::vector` 容器，这里就会返回 true。

然后是 typeWrapperRegister，这个函数以 MotorComponent 的类名作为后缀：它首先把那个字段的五个函数组成一个元组，注册进 field map；然后把基类相关的函数注册进 class map。最后的 typeWrappersRegister 里面就简单套了个娃，调用前面这个注册函数。

我们可以看一下更复杂的文件生成出来是怎么样的：可以看到它的结构是一样的，无非就是各部分的数量有所不同——它有三个类，就对应三个 operator，同时也有三个对应的 typeWrapperRegister；这些 operator 里面也同样是三部分，无非就是 fields 里面有多组这样的五个函数；在 typeWrapperRegister 里也需要把这多组都注册到 field map 里面。

```text
一个生成的 reflection 文件由三部分组成：

  (1) type operator 类
      ├─ 取类名 / 通过 JSON 构造 / 转成 JSON    （三个函数）
      ├─ 取得基类的反射实例                    （基类相关，一个函数）
      └─ 被标记字段的五个函数
            取类型名 / 取变量名 / set / get / 是否是数组
            （std::vector 容器特殊处理：是则返回 true）

  (2) typeWrapperRegister（以类名作为后缀）
      ├─ 字段的五个函数组成元组 → 注册进 field map
      └─ 基类相关函数          → 注册进 class map

  (3) typeWrappersRegister —— 简单套个娃，调用 (2)
```

对照上面的文字看：operator 负责提供函数，typeWrapperRegister 负责把这些函数写进那几个 map，typeWrappersRegister 则是留给外部统一调用的入口。

#### all_reflection 与 meta_register

再看一下整个 reflection 目录，你会发现这个文件夹里所有的文件，除了 all_reflection，其他都是以 `.reflection.j.h` 结尾的，而且里面的结构都是一样的。all_reflection 很简单，就是 include 所有以 `.reflection.j.h` 结尾的文件，然后在这个唯一的 meta_register 函数里，调用那些文件里面的 typeWrappersRegister。

这个 meta_register 同学们是不是有一点点眼熟？它就是我们第一次代码解读中提到过的、会在 startEngine 里最先调用的函数。所以在这个函数被调用以后，我们终于把收集到的所有类和字段的信息都添加到了那几个 map 里面，以供我们在运行时使用。

### 动手：让 Lua 脚本读写组件字段

讲了这么久，是时候动手了。为了让同学们对反射系统的使用有更具体的理解，我们扩充一下第一次代码讲解中引入的 LuaComponent：给它添加一个 get 函数和一个 set 函数，让它能够与引擎中的 component 有简单的交互。然后我们通过这两个 set 和 get 函数，让小白人在静止和移动状态下拥有不同的跳跃高度。

#### 目标与反射范围的两种标签

观察一下 MotorComponent。要让小白人的表现按照预期，我们的 Lua 脚本要能够获取 MotorComponent 的 `is_moving` 字段；同时如果要改变小白人的跳跃高度，我们需要能够修改 `motor_component_resource` 里面的 `jump_height`。

这里就能看到反射范围的两种标签：`motor_component_resource` 标记的是 fields，也就是所有字段都会反射；而 MotorComponent 只标记了 whitelist fields，也就是只有标记了 `META(Enable)` 的字段才会参与反射。所以我们给 `is_moving` 加上 `META(Enable)`。

#### 先跑通管道：会打印日志的 set 与 get

我们在 LuaComponent 里面添加两个函数 set 和 get，这两个函数都接收一个 game object 以及一个字段名。

第一版先简单实现：就只是打印一些 log，把字段名打印出来。然后我们再把这两个函数传递给我们的 Lua 虚拟机——现在只需要简单的 `set_float` 和 `get_float`，当然其他类型也可以像这样添加。

接下来修改 player 的资源文件，把前面写的 print hello world 改成我们想要的逻辑：先判断一下 motor component 上面的 `is_moving` 字段，然后再去 set motor component 上面 `motor_component_resource` 的 `jump_height`——如果正在移动的话，将它设为十，否则设为五。get 里面我们先简单写一个 return 0。

> **易错点**：不要忘了把 game object 也传递给 Lua 虚拟机——漏了它，脚本就拿不到目标组件。

跑起来看一看：可以看到命令行已经在打印我们 get 和 set 的两个字段了。

#### 真正的 get 与 set：拆名字、查 TypeMeta、找字段

接下来真正实现 get 和 set，过程分四步：

1. 首先取得所有的 components；
2. 将 field name 拆分一下，取出点号前面的第一个字符串，也就是 component 的名字；
3. 在所有的 components 里面遍历，找出名字等于这个 component 名字的那个 component；
4. 之后通过这个 component 的名字，取得它的 TypeMeta。

我们可能希望能够更加面向对象地去查询类型信息，所以封装了两个类：TypeMeta 和 FieldAccessor。这两个类的结构很简单，就是帮我们去查询前面的那几个 map，我就不赘述了，同学们可以自己看看。

拿到 TypeMeta 之后，再迭代地去取得这个 component 上面的字段：我们可以通过 meta 上面的 fields list 取得这一个类型上面所有 field 的信息，然后再在其中找到名字等于我们要查找的 field 的那一个。

最后在 set 和 get 里面，调用我们刚刚编写的这个"迭代寻找目标 field"的函数；取得目标 field 之后，就可以调用 field accessor 的 set 去写这个字段，以及 get 去取得这个字段的内容。

再跑起来看效果：我们在原地跳的话，是跳不过墙那么高的；但是当我们移动，就可以跳过墙了。嗯——卡住了，移动不了了：小白人卡死在了墙里面。这个卡死，正好是下一节的由头。

### 给反射系统扩充函数反射

在刚才的示例中，小白人卡死在了墙里面。我们可以假装我们的 motor component 有一个"脱离卡死"的功能，那么，要怎么样才能让我们的 Lua 脚本调用到这个函数呢？在前面我们发现，其实我们的反射系统只能够反射字段。接下来我们给反射系统扩充一下，让它能反射一个最简单的函数吧。

#### 先把想要的使用方式写出来

假如我们的反射系统提供了反射函数的功能，那我们应该怎么使用呢？可能有一个标签叫做 whitelist methods；然后我们要给 `getUnstuck` 添加一个 `META(Enable)`，用于提醒我们的反射系统：我们希望反射这个函数。然后在 LuaComponent 里面会有一个 invoke 函数，跟 set 和 get 差不多；同样，我们需要把它传递给 Lua 虚拟机，再修改一下 Lua 代码，在尾部调用一下 motor component 的 getUnstuck 函数。

我们先保存一下这部分的代码——用法就定成这样，接下来就是在 piccolo-parser 里面真正解析、生成函数反射相关的代码。

#### 解析侧：仿照 Field 写一个 Method

先看一下现在的 piccolo-parser 是怎么工作的。看 main 函数：首先是处理了一下参数，然后把这些参数传递给了 parse 函数；在 parse 函数当中实例化了一个 parser，调用了它的 parse 函数；parse 函数当中就是用 clang 处理我们的源码，最后调用了一个 buildClassAST——在所有的根节点里面找到 class 的声明和 struct 的声明，然后构造许多 class，把它们传递到 schema module 里面。

再看 class 类是怎么样的。在 class 的构造函数里面，我们遍历了 class 节点下面所有的子节点，然后对 base specifier 和 field declaration 进行了特殊的处理；对于每个 field declaration，我们构造一个 Field 类。Field 的构造函数很简单，把当前 field 声明的一些信息（比如名字、类型等）存储了下来。

我们可以仿照 Field 类编写我们的 Method 类：去掉多余的代码，简单实现一下。在判断标签的 isAccessible 逻辑里面，可以看到有一些 fields 和 whitelist fields 标签，我们仿照它编写我们的 methods 和 whitelist methods 标签，然后在那里替换一下——这样 Method 类就写好了。

然后我们在 class 的构造函数里面添加对 method 的特殊处理，同时在 class 里面添加一个 m_methods 成员。这样我们就把 method 的信息收集起来了。当然，这里还需要修改一下 shouldCompile 函数——因为原来一个类只有 fields 标记了才需要 compile，但现在加入了 method。

#### 生成侧：交给 Mustache 模板

下一步，就是如何把收集到的信息生成出代码。看一下 generate 函数：首先把里面的 shouldCompileFields 替换成 shouldCompile。这个函数里关键的部分，就是把收集到的信息转换成代码生成库 Mustache 能够识别的数据结构，然后根据我们的模板渲染出代码。

我们在这里仿照 fields 的模板编写 methods 的模板——Mustache 具体的语法可以查看它的官方文档。在这里，我们的 method 只需要两段函数：getMethodName 和 invoke。当然还有下面的部分：把 methods 的两个相关函数注册到 map 里面，同样模仿 field 的写法；然后再把 methods 相关的结构传递出去。

build 一下，修复一些笔误——Mustache 是大小写敏感的——再修改一些小 bug，就可以看到已经生成了我们预期的代码。先保存一下当前的代码，代码生成这一步就完成了。

#### 运行时侧：补齐缺失的类型与访问器

但是显然，method function tuple 和 method map 还没有被实现。也就是说，虽然现在已经能够成功地解析和生成，但 piccolo runtime 还是没办法编译。所以我们现在要：

- 添加 MethodFunctionTuple 以及 MethodMap，同样把需要用到的宏实现一下；
- 仿照 FieldAccessor 复制一份，稍微修改、去除多余的代码，编写一个 MethodAccessor——要让 method 用起来跟 field 一样简单，实现一下 MethodAccessor 的各个函数；
- 在 TypeMeta 里面仿照 fields 提供一个 getMethodsList 的功能，这样我们就可以遍历一个 type 上面所有的 methods；
- 因为 class 里添加了 m_methods，还要仿照 m_field 的处理，维护 m_methods 相关的代码。

编译一下，修复一些小 bug：这里是因为没有前置声明；然后字打错了；少了一个 s；还有一个 getMethodName 忘了实现了。再次编译就通过，把当前的代码再存储一下。

到这里，我们终于给反射系统实现了一个最简单的 void 函数的反射功能。回头看，这次扩充从头到尾都是沿着 field 那条路"仿写"出来的，两条线摆在一起看会更清楚：

| 环节 | 字段反射（field） | 函数反射（method） |
| :--- | :--- | :--- |
| 类上打的标签 | fields / whitelist fields | methods / whitelist methods |
| 成员上打的标签 | META(Enable) | META(Enable) |
| 解析侧的类 | Field | Method（仿照 Field 编写） |
| 生成的函数 | 类型名、变量名、set、get、是否是数组（五个） | getMethodName、invoke（两个） |
| 注册去向 | 函数元组注册进 field map | MethodFunctionTuple 注册进 method map |
| 运行时的封装 | FieldAccessor + TypeMeta 的 fields list | MethodAccessor + TypeMeta 的 methods list |
| 编译开关 | shouldCompile 认 fields | shouldCompile 同时认 methods |

#### 验证：用反射调用 getUnstuck

但是我们还从来没有验证过它。接下来去 LuaComponent 里面真正实现 invoke 函数，看看反射能不能符合我们的预期。

实现的思路是：假设我们已经取到了目标类型，就可以在它上面找到我们想要调用的函数，然后调用 invoke，通过反射去执行我们的目标函数。那如何找到目标类型呢？其实整个过程和我们前面 set、get 里面的过程类似。不过这里需要注意的是：如果我们的目标函数是在 component 上，那么不能像 get 和 set 一样直接调用 findComponentField 函数，而是需要自己去找到对应的 component。

具体可以判断一下 target name 里面有没有点：

- 如果没有点，就说明我们的目标函数是 component 上面的一个成员函数——直接找到对应的 component，再去找它上面的函数就可以了；
- 第二种情况，我们的目标是一个 component 上面的字段，那么可以调用 findComponentField 去找到对应的字段，然后再对它 invoke。

编译运行，可以看到 getUnstuck logic 已经成功打印出来了——也就是说，我们成功使用反射调用到了 component 上面的一个函数。

回顾这一次代码解读：我们使用反射系统实现了脚本对 component 字段的读写；然后为反射系统增添了简单函数的反射功能；最后实现了脚本对 component 函数的简单调用。不过，我们的函数反射现在只能反射最简单的一种函数——void 函数。如何才能让所有类型的函数都能被反射呢？这个问题连同渲染那一讲的两个设计题，一起放到文末聊。

---

### 渲染系统：RenderSystem 与每帧流程

讲完反射，时间往后拨一段——经历了一波新冠，又过了一个年，同学们好久不见了。接下来的部分，我们主要关注小引擎的渲染模块：首先会简单介绍一下整个渲染的流程，然后以 color grading 为例介绍小引擎中 pass 的概念，最后为小引擎添加一个简单的暗角 pass。本次代码讲解需要同学们对 Vulkan API 有一定的了解，具有编写简单 GLSL shader 的能力；同时推荐同学们学习一下 RenderDoc，这个工具可以更方便地进行渲染的调试。

#### RenderSystem：初始化与每帧 tick

小引擎的渲染系统叫做 RenderSystem。在第一次代码讲解中我们提到过，RenderSystem 在 global context 的 startSystems 里面初始化，然后在 renderTick 当中每一帧进行 tick。

在 RenderSystem 的初始化中，会简单地初始化 render resource 中的一些全局贴图，render camera 和 render scene 中的一些光照，并且会初始化 render pipeline。在小引擎中，我们把渲染的各个功能抽象为一个一个 pass，所以 pipeline 的初始化中会调用各个 pass 的初始化过程。

#### 逻辑层送来的数据：RenderSwapData

在每帧执行的 RenderSystem tick 函数中，首先会调用 processSwapData，这个函数从逻辑层获得要更新的数据。我们可以看一下 RenderSwapData 这个结构：其中的 level resource description 指定新的天空盒和 color grading 的贴图；game object resources 和 game objects to delete 分别指定要添加和删除的 game object；然后 camera swap data 更新相机参数；最后三个分别用于创建新的粒子发射器、指定需要 tick 的粒子发射器、更新粒子发射器的位置。

processSwapData 剩下的代码，就是根据 RenderSwapData 结构体里面描述的需要更新的信息进行特定的更新，同学们可以自己看一看。

> **提示**：其实这里所谓的 swap 并不是十分准确，因为我们只有渲染从逻辑拿数据，谈不上双向的"交换"。

#### 一帧渲染的完整顺序

进行完数据交换之后，渲染系统就要开始每帧实际渲染的渲染过程了：

```text
renderTick（每帧）
 ├─ processSwapData        从逻辑层取要更新的数据
 ├─ prepareContext         调用图形 API 做渲染准备
 ├─ 准备每一帧 pass 共用的数据
 ├─ updateVisibleObjects   为各个 pass 筛选各自所需的具体数据
 ├─ preparePassData        调用各个 pass 的 preparePassData
 └─ 判断渲染模式
     ├─ 前向渲染：forward render → main camera pass 的 drawForward
     │                           → 依次执行各 subpass 的 draw
     └─ 延迟渲染：main camera pass 的 draw
                  → 一次几何绘制，物体几何信息写入 G-buffer
                  → 用 G-buffer 缓存的数据做光照计算
```

读图说明：先是调用一些图形 API 进行渲染的准备，也就是 prepareContext 函数；然后准备每一帧 pass 共用的数据；接下来的 updateVisibleObjects 是为各个 pass 筛选各自所需的具体数据；后面的 preparePassData 会调用各个 pass 的 preparePassData 进行数据准备；最后判断使用前向渲染还是延迟渲染——小引擎的渲染分为前向渲染和延迟渲染两种模式。

前向渲染走 forward render 函数：先拿到渲染功能的对象，再执行 main camera pass 的 drawForward 函数；在 main camera pass 的 drawForward 中，会执行各个 subpass 的 draw。如果是延迟渲染，则执行 main camera pass 的 draw：先执行一次几何绘制，将物体的几何信息存到 G-buffer 中，然后利用 G-buffer 里缓存的数据进行光照计算。

### 以 Color Grading 为例认识 Pass

#### 一张 32×32×32 的三维查找表

我们先来看这样一张 color grading look up table：它是一张长为 32×32——也就是 1024 像素——高为 32 像素的贴图。它其实是由 32 张 32×32 的小贴图组成的，我们可以把这 32 张小贴图组成如图所示的一个立方体，它就是我们颜色转换的目标色彩空间。

```text
存储形态：一张宽 1024（= 32 x 32）、高 32 的长条贴图

  [第 0 张] [第 1 张] [第 2 张] …… [第 31 张]     每一张都是 32x32

逻辑形态：把 32 层小贴图沿第三个维度堆起来，
就是一个 32 x 32 x 32 的颜色立方体
（即 color grading 的目标色彩空间）
```

读图说明：长条贴图是它在文件里的存储形态，立方体是它的逻辑形态——采样时先"挑层"，再在层内取坐标。

> **定义**：color grading 实际上做的事情，就是把原来 RGB 色彩空间的一个颜色，转换到对应的、由这张贴图表示的色彩空间的颜色——一个普通的 RGB 颜色坐标，对应立方体里一个确定的颜色。

#### 写 shader：为什么必须采样两次

接下来我们要通过编写一个 shader 来实现这个事情。首先在 shader 文件夹里面的 glsl 目录里找到 `color_grading.fragment`，这是实现 color grading 的片段着色器。

我们先把默认的 `out_color = color` 注释掉。因为我们要从一个三维的空间采样，所以至少需要从这张二维贴图上面采样两次：首先要在这 32 张小贴图里面找出两张，采样上面的颜色，最终再 mix 一下这两个贴图上面对应的颜色。

> **易错点**：只采样一次是不行的。在批改 homework 2 的作业中可以看到，有一些同学只采样了一次，那这个结果多多少少是有些问题的。

实现细节是：先通过 `color.b` 乘上 color grading look up table 的高度——这个高度其实就是这张贴图里面 32 张小贴图的层数；这样我们将这个值再分别向上和向下取整，就得到了需要采样的两张小贴图的 index。然后我们再换算一下坐标，在这两张小贴图上分别根据 `color.r` 和 `color.g` 进行采样；最终再将这两个采样的结果 mix 一下，就得到了我们想要的目标颜色。

编译运行一下，看这个 shader 能不能按照预期达到 color grading 的效果。嗯，可以看到效果其实是有一些问题的——画面中出现了一些奇怪的线条。

#### RenderDoc 抓帧：揪出多余的 mipmap

这个时候我们就需要打开 RenderDoc 进行调试一下。在 launch 里面选择我们的 piccolo editor，然后在 RenderDoc 里面启动它之后，再按一下 F2 就能抓取一帧。然后我们可以在左边的命令列表里面找到 color grading，看一下它的 input 和 output 的结果。

在这个时候我已经发现问题所在了：因为 color grading 这张贴图默认产生了一些 mipmap，导致 look up table 最终的精度不够，出现了这种线条。我们可以找到 mip levels 设置的地方，将值从默认的 0 改为 1，然后再运行一下，就可以看到能够达到我们预期的效果了。这样我们就实现了一个 color grading shader。

#### ColorGradingPass 与它的四步初始化

接下来我们看一下这个 shader 到底是怎么样真正发挥作用的。

在 Piccolo 引擎当中，color grading pass 是 RenderPass 的一个子类，定义在 color_grading_pass.h 和 color_grading_pass.cpp 文件中，它只有三个 public 的函数。我们先来看一下 RenderPass 这个类，里面有几个重要的成员：

- global render resource 指针：记录了这个 pass 可能使用到的全局资源；
- descriptor info：记录了输入、输出资源在 shader 中的布局信息；
- render pipelines：记录了执行管线的信息；
- visible nodes：记录了这个 pass 可见的物体——例如场景中可能有多个游戏对象，而该 pass 只会需要用到其中一部分。

我们的 color grading pass 被设计为 main camera pass 的一个 subpass，这个 enum 指定了 main camera 所有的 subpass。subpass 是 Vulkan API 独有的设计，作为 Vulkan render pass 对象的一个部分：一个 Vulkan 的 render pass 可以包含多个串行的 subpass，前一个 subpass 的输出作为后一个 subpass 的输入。所以我们在这里将 color grading 作为 main camera pass 的一个 subpass——可以看到，这个 subpass 就是在这里被添加到 main camera pass 里的。

我们可以仔细看一下 color grading pass 的 initialize 函数。首先它接收一个 init info，也就拿到了刚才提到的 color grading look up table 这张贴图；然后它会执行以下四步的初始化过程：

1. **setupDescriptorLayout**：设置 color grading pass 的 shader 对应的描述符布局，也就是 shader 中使用的输入输出资源的排布。这里用到了两个资源：一个是 tone mapping pass 的输出，一个是调色纹理。
2. **setupPipelines**：color grading pass 使用的 pipeline 是一条 graphics pipeline，主体部分为 vertex shader 和 fragment shader；还在此处设置视口、裁剪、朝向、剔除、混合模式、深度模板等必要的信息，细节就不展开了。上面设置好的 descriptor layout 也会用于 pipeline 对象的创建。
3. **setupDescriptorSet**：有了上面的资源描述符布局信息，还需要设置实际使用到的资源。descriptor set 可以理解为资源 handle 的集合。
4. **updateAfterFramebufferRecreate**：当我们的视口发生变化的时候，进行一些更新操作。

color grading pass 的 draw 函数会在 main camera pass 的 draw 里面被调用，这个函数就是提交了一些 drawing 命令。

### 实战：给小引擎添加一个暗角 Pass

好，现在让我们来提交一下写好的 color grading 实现，然后从头开始添加一个新的 pass，实现一个暗角的效果。

#### 复制改造与 subpass 登记

首先将 color_grading_pass.cpp 和 color_grading_pass.h 两个文件复制一份，重命名为 vignette_pass.cpp 和 vignette_pass.h，最后将这两个文件里面的 color grading 全部都替换成 vignette。

我们这个 pass 也是和 color grading pass 一样，作为 main camera pass 的一个 subpass，所以需要在 main camera subpass 的 enum 里面添加一个 vignette subpass。接下来仿照 color grading 这个 pass，添加 vignette subpass 的一些设置和调用。

在这里的这些 buffer——odd buffer、even buffer——是 subpass 之间的一些缓冲区，所以我们需要对后面的输出贴图引用进行一下替换。

还需要维护一下 subpass 之间的 dependency 关系：首先需要添加一个 dependency，所以这里要把 dependency 的数量从 8 改成 9；然后因为我们的 vignette subpass 是添加在 color grading subpass 之后的，它的序号是 6，所以需要把原来 color grading 后面的几个 pass 重新修改一下。

#### 编译踩坑与第一次运行

接下来编译一下，修复一些编译上的错误。有一个错误：函数我们是实现了的，所以应该是因为新添加的文件没有加入到构建——需要重新 cmake 一下。然后这里提示找不到 vignette fragment：我们先复制一下 color grading fragment，将它直接改名为 vignette fragment。

我们新建的这个 fragment shader，会在 CMake 构建流程当中转换成一个 .h 文件；这里要把 vignette 从小写改成大写。然后，将 vignette resource 重新改回 color grading resource——因为我们并没有为 vignette 添加一个贴图，暗角效果其实是可以不需要贴图就实现的。

好，这样修改之后就可以运行起来了。但实际上可以看到，现在的效果其实是不符合预期的：因为我们理论上应该是会进行两次 color grading——现在 vignette 的逻辑是再次进行一个 color grading——所以画面应该会变得更明显一点，但这里还是跟刚才一次 color grading 一样的效果。

#### 修正缓冲区，真正实现暗角

打开 RenderDoc，可以看到 vignette 这个 pass 的输入，还是一个没有经过 color grading 的贴图。这个应该是我们 subpass 之间传递贴图的 buffer 没有设置对。我们来重新检查一下，将它们的 odd 和 even 修改一下。

好，现在编译之后，可以看到效果应该是两次 color grading 都生效了——因为画面变得更暗了。

接下来需要真正实现暗角的逻辑，主要是修改 vignette fragment 这个 shader。实现暗角效果的时候，需要一个相比于 color grading 额外的输入，就是 uv。我们先随意地设置两个参数。还需要修改一下原来的 post process vertex shader：将它复制一下，重命名为 vignette vertex shader，将 uv 输出；然后我们把 pass 里的 shader 修改成 vignette vertex shader。同时，由于 vignette pass 是不需要贴图的，所以我们可以把 descriptor layout 修改一下。

好，编译运行，可以看到边缘有一些发黑——也就是说，我们已经成功地给小引擎添加了一个新的 pass，实现了一个暗角的效果。

### 留给你的开放问题

到这里，两段代码解读都收尾了，最后留几个开放问题给大家。

**函数反射的完全体**：这一次我们只给反射系统实现了最简单的 void 函数的反射，如何才能让所有类型的函数都能被反射呢？欢迎在评论区或者 Piccolo 开发者频道分享你的思路，也可以直接在 GitHub 发起 PR，帮助 Piccolo 引擎实现这一功能。

**更顺手的 Pass API**：通过添加暗角 pass 的过程可以看到，目前给小引擎添加 pass 还是比较繁琐的。如果是你，你会怎么设计 API，让它可以更加方便地修改和添加内容呢？

**把 shader 变成资产**：目前小引擎的 shader 是在 CMake 构建流程中编译、硬编码进各个 pass 的，你会怎么设计，让小引擎将 shader 作为一种资产、可以在运行时加载？

欢迎在评论区讨论，也可以关注微信公众号 GAMES104，在知乎上邀请 GAMES104 回答问题；想深入交流代码的同学，欢迎加入 Piccolo 开发者 QQ 频道。

---

## 本册小结

本册主题为「Piccolo引擎源码解读」，整编了 2 个模块（P35 ~ P36-P37，共 3 讲 / 95 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
