# LangGraph 核心：State、控制流与 Agent 构建

## 图的构成：节点、状态、边

* **状态（State）**

    * > **定义**：状态是整张图从头到尾唯一流转的数据载体，节点读它、改它，边只决定它按什么顺序被处理。

    * 状态在代码里是一个类，类里的每个字段就是图中一份数据。

    * 字段来源分两类：调用时传入的原材料，以及执行过程中产生的中间结果与最终结果。

    * 判定标准：凡是要跨节点传递的东西，都必须先在状态里开一个字段；否则上游算出的值无处存放，下游也取不到。

* **节点（Node）**

    * > **定义**：节点是图里真正干活的函数，入参是一个完整状态对象，返回值是本次要更新的字段字典。

    * 节点内部没有固定职责，调模型、执行工具、查数据、只拼字符串都算合法实现。

    * 节点之间不直接传参，唯一的传参通道是状态。

* **边（Edge）**

    * 边就是连接两个节点的线，连的时候就把执行顺序定下来了。

    * 节点的执行顺序由边决定，与函数在代码里出现的先后无关。

* **入口与出口（START / END）**

    * > **定义**：`START` 与 `END` 是 LangGraph 预置的两个特殊节点常量，分别充当整张图的入口与出口，不需要自己定义。

    * 两者与其他构件一起从 `langgraph.graph` 导入即可。

    * 最小拓扑就是一条直链，两端由这两个常量撑住。

```text
START ──> node1 ──> node2 ──> END
```

### TypedDict 与状态约束

* **TypedDict**

    * > **定义**：`TypedDict` 是"用 class 语法声明字段、运行时仍然是 dict"的类型。

    * 状态类名随意，但必须继承 `TypedDict`，这一点是死的。

    * 普通字典的 key 没有任何约束力，今天叫 `name`、明天叫 `a`；节点里取 `state["name"]` 就会取不到。`TypedDict` 把这类错误提前到传入时暴露，不必拖到运行时才炸。

    * 字段声明之后 key 固定、值的类型受注解约束，key 对不上的字典不能冒充该状态。

    * 取值仍按字典来：`state["name"]` 或 `state.get("name")`；`state.name` 点不出来，因为运行时它真的不是普通对象。

| 对比项 | 普通 `dict` | `TypedDict` 类 |
| :--- | :--- | :--- |
| key 是否固定 | 随便写，无约束 | 由字段声明钉死 |
| 值的类型 | 不做检查 | 按类型注解约束 |
| 运行时本质 | 字典 | 仍是字典 |
| 取值方式 | `d["k"]` 或 `d.get("k")` | 一样，中括号或 `get` |

## 搭图的四步骨架

| 步骤 | 做什么 | 要点 |
| :--- | :--- | :--- |
| 定义状态 | 继承 `TypedDict`，列出字段 | 输入参数与中间结果都算流转数据，都得有字段 |
| 定义节点 | 普通函数，入参是状态 | 只返回要更新的字段，不必原样抄回整个状态 |
| 装配成图 | `StateGraph` → `add_node` → `add_edge` → `compile` | 先加节点再连边，边里传节点名 |
| 调用 | `graph.invoke(字典)` | 传状态、收状态，两头都是字典 |

### 定义节点与装配

* **节点的形状**

    * 入参是一个完整状态对象，返回值用字典给，只放要更新的字段。

    * 只返回部分字段是合法的：没返回的字段保持原样，不会被清空。

    * 写节点的三步：先从状态读旧数据，再写自己的计算逻辑，最后返回要更新的那部分。

* **状态更新入口**

    * 更新不是靠在节点里直接改状态对象，而是靠返回值：LangGraph 拿到返回字典后把新值合并回去。

    * > **易错点**：在节点里写 `state["age"] = 31` 再 `return state` 不是这套用法；要改什么用返回值说，返回值是唯一的合并入口。

* **builder**

    * `StateGraph` 是"带状态的图"，第一个参数叫 `state_schema`，用来说明图内流转的数据长什么样。

    * 它返回的还不是图，而是一个图的构建器，后续所有添加动作都往这个 builder 上做。

    * `add_node(name, func)` 两个参数：字符串节点名 + 节点函数本身；名字不是装饰，后面连边全靠它。

    * `add_edge(start, end)` 一条边：从哪开始、到哪结束，两头都写名字，`START`/`END` 是常量、中间节点用 `add_node` 时起的字符串。

    * `compile()` 之后才得到最终可执行对象。

    * > **易错点**：第二个参数只写函数名，写成 `node1()` 是把函数当场执行，传进去的是返回值而不是节点。

    * > **易错点**：必须先 `add_node` 再 `add_edge`；节点还没进图就去连边，等于指着一个不存在的名字连线。

### 调用与执行轨迹

* 整张图都在加工状态，所以调用时传状态、拿回来的也是状态；状态本来就是字典，那就传字典、收字典。

* 只传输入字段即可，中间结果字段留空，由执行过程长出来。

```text
invoke 传入        {"name": "杰克"}
node1 执行后       {"name": "杰克", "greeting": "Hello, 杰克"}
node2 执行后       {"name": "杰克", "greeting": "HELLO, 杰克"}
```

* 三行轨迹说明两件事：`greeting` 是执行中新增的字段；node2 能接着改，是因为它拿到的状态里已经有 node1 写好的值。

* **图的渲染**

    * `graph.get_graph()` 相当于穿透进图内部拍一张照，把节点和边都显出来。

    * 在 notebook 里可用 `IPython.display` 的 `display` 与 `Image` 包住 `draw_mermaid_png()` 的返回值直接渲染。

    * > **提示**：不同版本里这个拍照方法的名字可能不同（`draw_mermaid_png`、`draw_png` 之类），按本机补全提示写；图出不来先确认绘图依赖是否安装。

    * 设计对得上时，渲染结果就是一条直链，没有分叉。

## Reducer 与字段更新语义

* **Reducer**

    * > **定义**：reducer 是每个状态字段各自的合并规则，接收该字段的旧值与本次更新带来的新值，返回值作为该字段的最终值。

    * 状态里的每一个字段都有自己的 reducer，规则按字段生效，互不串用。

    * 节点返回字典后，LangGraph 逐字段合并：只处理返回值里出现的字段，没出现的字段根本不进这道流程。

### 默认 Reducer

* 默认 reducer 等价于一句 `return new`：新值进来，旧值出局。

```python
def reducer(old, new):
    return new
```

* 这段函数不会出现在你的代码里，但默认行为就是它。

* 覆盖效果的可验证证据：同一个字段被两个节点先后改写，第一个节点写进去的值被第二个节点写进去的值盖掉，最终只剩最后一版。

| 观察点 | 拿到的值 | 说明 |
| :--- | :--- | :--- |
| node1 收到 | `initial value` | `invoke` 传进去的初始值 |
| node2 收到 | `node1 value` | node1 的返回值已顶掉初始值 |
| 最终 result | `node2 value` | node2 又顶了一次 |

* 这条证据链的结论是：初始值活不过第一个节点，第一个节点的值活不过第二个节点，后写的赢。

### 自定义 Reducer 与 Annotated

* **Annotated**

    * 用法是给字段套一层 `Annotated[类型, reducer]`：第一截写数据类型，第二截写要挂的 reducer。

    * 只改类型注解不会改变合并语义，必须用 `Annotated` 显式挂上 reducer，该字段才脱离默认覆盖。

```python
from typing import Annotated, TypedDict
import operator


class CustomReducerState(TypedDict):
    count: int                              # 无 Annotated，走默认覆盖
    notes: Annotated[list[str], operator.add]   # 挂加法 reducer，走拼接
```

* **operator.add**

    * 它是加法运算符的原始写法：`add(a, b)` 干的就是 `a + b`，因为 `+` 是特殊符号，不能当对象传递，标准库才把它包成函数。

    * 列表加列表等于合并两个列表，`["node1"] + ["node2"]` 得到 `["node1", "node2"]`，语义正好是追加。

* **自定义 reducer 函数**

    * 形状固定为两个参数、返回合并结果：`def add(old: list[str], new: list[str]) -> list[str]: return old + new`。

    * 换成 `operator.add` 效果完全一致，平时不必自己造这个函数。

* **调用时机**

    * node1 执行时 LangGraph 发现 `notes` 被更新，把该字段旧值（`invoke` 时传入的空列表）当第一个参数、把节点返回的 `["node1"]` 当第二个参数传进去，拼出 `["node1"]`。

    * node2 执行时旧值已是 `["node1"]`，新值是 `["node2"]`，拼成 `["node1", "node2"]`。

    * 开 debug 单步看会多出一次调用：初始灌入状态那一步 reducer 也被调一次，此时旧值和新值都是空列表。

* 反证：`notes` 只写 `list[str]` 不挂 reducer 时，最终结果只剩 `['node2']`，第一次写入的 `"node1"` 被整个列表覆盖掉。

## 边与执行流向

### 普通边与并行

* **普通边**

    * 状态管的是"数据怎么更新"，边管的是"节点按什么顺序跑"；一条普通边把一个节点连向另一个节点，流向固定，A 执行完执行 B，一条路走到底。

    * 串行图整个工作流只有一条路径，模型或函数只是路径上的一个点。

* **并行的真实含义**

    * > **定义**：并行执行指同一份上游状态被同时分发给多个下游节点，各节点互不干扰地处理；它不表示多个节点在同一时刻一起运行。

    * CPU 同一时刻只能执行一个任务，所以"同时"指的是分发同时、输入相同，不是同时刻计算。

    * A 更新完的状态会同时给到多条出边，B 和 C 拿到的状态一模一样，然后各处理各的那一份。

```text
              ┌──> B ──┐
START ──> A ──┤        ├──> D ──> END
              └──> C ──┘
```

* 这张并行图的边一共六条：`START→A`、`A→B`、`A→C`、`B→D`、`C→D`、`D→END`。

* 调用时可以直接 `graph.invoke({})`，不传初始值也成立，列表字段一开始就是空列表。

* 执行结果印证并行语义：A 收到空列表，B 和 C 都收到 `['A']`（完全相同），D 收到 `['A', 'B', 'C']`。

* **并行写入的排序**

    * 决定因素不是边的添加顺序：把 `A→B`、`A→C` 的书写顺序对调，结果不变。

    * 决定因素是节点名：多个并行节点往同一字段填内容时，按节点名称的字典序排列；把 `C` 改名成 `AC`，它就排到了 `B` 前面。

    * > **提示**：这个顺序本身不重要，不必记；只要记住 B 和 C 是并行关系，拿到同一份状态、各算各的，谁先谁后不影响结果。

### 条件边与路由函数

* **条件边（Conditional Edge）**

    * > **定义**：条件边不写死终点，而是给一个路由函数，用它的返回值决定下一个节点。

    * 第一个参数是起点节点；第二个参数是路由函数（route），其返回值就是下一个节点的名字；第三个参数 mapping 可选。

    * mapping 用于兜底映射：key 是路由函数的返回值，value 是节点名；路由函数直接返回节点名时，mapping 可以省掉。

    * 图上的条件边画成虚线，用来区分写死的普通边。

* **路由函数**

    * 写法和节点很像，参数同样是状态，内部按条件返回不同的节点名。

    * 返回值必须是节点名，这是条件边的命门；返回的不是节点名，就得再补 mapping 去映射，很麻烦。

```python
from typing import Literal


def route(state: ScoreState) -> Literal["success", "fail"]:
    score = state["score"]
    if score >= 60:
        return "success"
    return "fail"
```

* **Literal 返回类型标注**

    * 作用是把路由函数可能返回的节点名罗列出来，让 LangGraph 能把图补全。

    * 不加也能运行，但图会断：起点节点后面什么都没有，因为 LangGraph 只认得节点和边，无法钻进函数体看 `return` 了什么。

    * 传给 LangGraph 时它只能看到函数名、参数和返回值类型，看不到函数体；标上 `Literal["success", "fail"]` 它才知道这个路由只可能去这两个节点。

    * > **注意**：凡是用条件边、自己写判断函数，返回值类型一定要用 `Literal` 把可能的下一个节点名罗列出来；少了这步程序照样跑，但图画不出来。

* 条件边案例的输入处理：`input()` 返回的永远是字符串，而状态字段声明的是 `int`，必须 `int()` 转一道再写回去，否则后续大小比较直接报错。

### Command 与跳转

* **Command**

    * > **定义**：`Command` 把"状态更新"和"下一个节点的名字"合进一个返回值，让节点同时承担普通节点与路由函数的职责。

    * 两个字段：`update` 放原本节点该返回的字典，负责更新状态；`goto` 放下一个节点的名字。

    * 从 `langgraph.types` 导入。

```python
from langgraph.types import Command


def score_node(state: ScoreState) -> Command[Literal["success", "fail"]]:
    score = int(input("请输入用户分数："))
    goto = "fail"
    if score >= 60:
        goto = "success"
    return Command(update={"score": score}, goto=goto)
```

* 采用 `Command` 后，路由函数整个删掉，条件边也不再需要，只留 `START → score` 以及各分支到 `END` 的普通边。

* `goto` 初值给 `"fail"` 相当于默认按失败处理，只有达标才翻盘，`else` 可以省掉。

* **类型声明**

    * 返回值标注成 `Command[Literal["success", "fail"]]`，LangGraph 才知道这个节点只可能去这两个地方。

    * 不加声明也能调用，程序照跑，只是图画不出来；加它的唯一目的是让图能看见。

    * > **提示**：删边之后图看上去是残缺的——`score` 后面没有出边，`success` 与 `fail` 也没有入边；这不是边写漏了，走向藏在 `goto` 里。

### 三种控制流对比

| 写法 | 谁决定下一站 | 状态更新写在哪 |
| :--- | :--- | :--- |
| 普通边 | 建图时 `add_edge` 写死 | 节点返回的字典 |
| 条件边 | `route` 函数的返回值 | 节点返回的字典 |
| `Command` | 节点返回的 `goto` | 节点返回的 `update` |

* 从普通边到条件边，是把"往哪走"从建图时挪到了运行时。

* 从条件边到 `Command`，是把"往哪走"和"改什么"从两个地方合回一个返回值，控制权一步步交给节点自己。

## 用 LangGraph 构建 Agent

* > **定义**：最小的 agent 不需要任何额外框架概念，一张只有三个节点的图，其中一个节点的函数体里调一次模型，它就已经是 agent。

* 构建 agent 拆开看很朴素：加入模型调用，模型调用是一段代码，代码封装进节点，节点连成图。

* 完整顺序是：初始化模型 → 创建状态 → 创建节点 → 添加边 → 编译 → 调用。

### 最小模型图

* 模型初始化用 `init_chat_model`，模型名是必填项；要控制行为可加 `extra_body`，把 `thinking` 的 `type` 设成 `disabled` 关掉思考。

* `load_dotenv()` 不能省：不显式传 API Key 时模型会去读环境变量，先把 `.env` 读进环境，就不用每次手动传 key。

* 状态两个字段：`input` 存用户提问，`output` 存模型结果，字段名可替换。

```python
def llm_node(state: LLMState):
    response = model.invoke(state["input"])
    return {"output": response.text}
```

* 取 `response.text` 而不是 `response.content`：`content` 是原始内容块，`text` 才是拼好的纯文本。

* 图就是 `START → llm → END` 一条直链，与前面任何一张图没有结构差异，唯一变化是节点里多了一次模型调用。

* > **提示**：直接调模型与调 agent 的入参和返回完全不同。`model.invoke(...)` 收字符串或 `BaseMessage` 列表、返回 `AIMessage` 对象；`agent.invoke(...)` 收字典（内含 `messages`）、返回也是字典（内含 `messages`）。

### 消息对象改造

* 状态字段类型换成 `HumanMessage` 与 `AIMessage`，输入输出都用 LangChain 的消息类型，不再用裸字符串。

* 节点里不能把单个消息对象直接传给模型，必须用列表包起来，哪怕只有一条。

```python
def llm_node(state: LLMState):
    response = model.invoke([state["input"]])
    return {"output": response}
```

* 返回侧反而更省事：`response` 本身就是 `AIMessage`，不用取 `.text`，直接放进 `output`。

* 调用时 `input` 也要传 `HumanMessage` 实例，不能传字符串。

* > **易错点**：只要走消息，就必须是消息的列表，哪怕只有一条；`model.invoke(HumanMessage(...))` 必错，`model.invoke([HumanMessage(...)])` 才对。

### 消息列表与追加语义

* 分成 `input`、`output` 两个字段有硬伤：多轮交互时第二次调用会覆盖第一次的输入和输出，模型永远只看得到最后一句话，这不叫对话。

* 改法是把输入输出合成一个 `messages` 列表字段，输入放这里、输出也从这里取并放回这里。

* 列表元素类型标 `BaseMessage` 即可，`HumanMessage` 与 `AIMessage` 都继承它。

```python
class LLMState(TypedDict):
    messages: Annotated[list[BaseMessage], add]
```

* 必须挂 reducer：状态默认语义是覆盖，要不断往里拼就得用 `Annotated` 挂 `add`（即 `operator.add`），新旧值走列表拼接而非替换。

* 节点两处跟着改：取值直接 `state["messages"]` 丢给模型，不用再包一层；返回要把单条 `AIMessage` 装进列表，让 reducer 的"列表 + 列表"成立。

```python
def llm_node(state: LLMState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}
```

* 图一行都不用动，只把调用时的字段名换成 `messages`。

* 一次调用里 reducer 会被用两次，`old` 与 `new` 的含义如下：

| 时刻 | old（已有值） | new（本次返回） | 状态落库结果 |
| :--- | :--- | :--- | :--- |
| invoke 进入图 | 空 | `[HumanMessage]` | `[HumanMessage]` |
| 进入 `llm_node` | — | — | `[HumanMessage]`，整包传给模型 |
| 节点返回后 | `[HumanMessage]` | `[AIMessage]` | `[HumanMessage, AIMessage]` |

* > **结论**：`AIMessage` 不是节点自己 append 进去的，是 reducer 拼进去的；节点只负责返回一个装了新消息的列表。

* **与 agent 调用约定同形**

    * 传入字典、字典里 `messages` 是消息列表，返回字典、字典里也是消息列表，这正是调 agent 的形状，阻塞与流式都一样。

    * `create_agent` 的返回值类型是 `CompiledStateGraph`，而 `StateGraph(...)` 加 `compile()` 产出的正是同一个东西，所以 agent 底层就是 LangGraph。

    * > **结论**：agent 所谓的"记忆"不是模型记住了什么，而是状态里的消息列表越来越长，每次调用都把整包消息重新喂给模型。

### MessagesState

* **MessagesState**

    * > **定义**：`MessagesState` 是 LangGraph 预置的状态类型，内部只有一个 `messages` 字段，类型是消息列表（名为 `AnyMessage`，作用等同 `BaseMessage`），并自带 `add_messages` 这个追加型 reducer。

    * 它与手写的"消息列表 + reducer"完全等价，没有理由再手写一遍。

    * 三种用法：需要额外字段就继承后再加；没有额外字段就继承后 `pass`；连继承都省掉，直接把它当状态类型用。

```python
from langgraph.graph import StateGraph, START, END, MessagesState

def llm_node(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}
```

* 流式调用同样成立，因为编译出来的就是同一个对象；从流里逐块取 `.text` 打印，把 `end` 设为空字符串、`flush` 设为 `True`，就能像打字机一样连续输出。

* > **提示**：真正的难点不在 API，而在时刻记住 `messages` 是一个列表：从状态里取出来是列表，往状态里返回时也要先装进列表。

### Checkpointer 与会话记忆

* 历史消息只是内存里的一个列表，程序一停就没了；要形成真正的会话记忆必须挂 checkpointer。

* 挂载位置与以往用 agent 时不同：不是在建 agent 时指定，而是在 `builder.compile()` 编译时传进去。

```python
from langgraph.checkpoint.memory import InMemorySaver

graph = builder.compile(checkpointer=InMemorySaver())
```

* 光有 checkpointer 还不够，调用时必须告诉它"这是哪个会话"，所以要准备 config，里面放 `thread_id`。

```python
config = {"configurable": {"thread_id": "T1"}}

result = graph.invoke(
    {"messages": [HumanMessage("12 个苹果平均分给 3 个人，每人分几个？")]},
    config,
)
```

* 第二轮只发新问题、不重复前提，仍能接上：返回的 `messages` 里依次是第一轮提问、第一轮回答、第二轮提问、第二轮回答。

* 接上的机理还是 reducer：checkpointer 把上一轮状态取回来，新的 `HumanMessage` 通过 `add` 拼进旧列表，模型看到的是三条消息（两问一答），自然知道"4 个人"在接着上一题问。

* > **结论**：LangGraph 里做会话记忆用的还是 checkpointer，只是指定位置从建 agent 时挪到了编译时；调用时把 `thread_id` 放进 config 传进去，流式调用也照传。

## 工具型 Graph

* 模型自己执行不了函数，它只能"说"要调哪个工具、参数是什么；真正把函数跑起来、把返回值拿回来的是我们自己的 Python 代码。

* 跑通一次工具调用的完整链路：先把用户问题与本地工具清单一起发给模型，模型返回要调的工具名与参数，我们手动调工具，再把结果发回模型，模型生成最终答案。

* 一条用户提问、两次模型调用，中间夹一次解析和一次转发，这就是全部复杂度的来源。

```text
请求
  │
  ▼
模型节点 ──思考──▶ 要不要调工具？调哪个？
  │
  │ 返回 AIMessage，里面带 tool_calls（工具名 + 参数 + 调用 id）
  ▼
工具节点 ──按名字查字典──▶ 手动 invoke 工具，拿到结果
  │
  │ 把结果封装成 ToolMessage，带上对应的 id
  ▼
模型节点 ──再思考──▶ 还要不要继续调？
  │
  │ 不需要了
  ▼
最终答案
```

* 中间这一段可重复：模型拿到结果后可能还要再调一次，也可能直接回答，所以真实形状是带出口的环，不是直线。

### 工具索引与绑定

* **按名索引**

    * 模型返回的是工具名的字符串，要据此找到函数，就得有一张 key 为工具名、value 为工具本身的表。

    * `tools_by_name = {tool.name: tool for tool in tools}`，一个推导式即可；之后 `tools_by_name[name]` 就能取到工具。

* **StructuredTool**

    * 拿到手的不是普通函数，而是 LangChain 包装过的工具对象，它有 `name`、`description`，底层才是原函数，并带一个 `invoke` 方法。

    * `invoke` 的入参必须是字典，key 是参数名，不能传位置参数；这与模型返回的 `args` 是字典正好严丝合缝。

    * > **易错点**：编辑器补全里可能看不到 `invoke`，因为此时变量还没声明类型；声明类型后它就在了，不是工具被包坏了。

* **bind_tools**

    * > **定义**：绑定不是把工具塞进模型里，只是给模型配一份"可用工具清单"；真正执行工具的仍然是 Python 代码，模型只负责挑。

    * `model.bind_tools(tools)` 接收工具列表、返回绑好工具的模型，之后调用它会自动把工具的参数信息封装进请求。

    * 若自己发请求，就得在 `tools` 字段里把每个工具的名字、描述、参数 schema 拼成 JSON，绑定把这件苦力收了。

* **tool_calls 结构**

    * 模型返回的 `AIMessage` 上多一个 `tool_calls` 列表，每个元素三个关键字段：

```text
tool_calls = [
    {
        "name": "get_weather",        # 要调的工具名，就是函数名
        "args": {"city": "杭州"},      # 参数，已经是字典
        "id":   "call_xxxxxxxx",      # 这一次调用的唯一标识
    },
]
```

    * `name` 拿去查 `tools_by_name`，`args` 直接喂给 `invoke`，`id` 留到封装工具消息时用。

### 模型节点与工具节点

* 模型节点与之前的写法完全一样，唯一区别是用绑好工具的模型，状态直接用 `MessagesState`。

* 节点确实没变，但它返回的东西变了：`response` 里现在可能带着 `tool_calls`，后面的分支、循环、消息封装全冲这个字段来。

* 工具节点要做四件事：取最后一条消息、拿 `tool_calls`、挨个调工具、把结果封成消息还回去。

* 三个坑：

    * 不能写 `tool_calls[0]`：一次提问可以有多次调用，比如同时问杭州和北京天气会返回两个 `tool_call`，必须循环。

    * 也可能一个都没有：问一句"你好"返回里没有 `tool_calls`，此时什么都不做，直接返回 `{"messages": []}`。

    * 工具结果不能直接扔回消息列表，必须包成 `ToolMessage`，至少带 `content`（执行结果）与 `tool_call_id`（对应调用的 id）。

```python
from langchain_core.messages import ToolMessage


def tool_node(state: MessagesState):
    tool_calls = state["messages"][-1].tool_calls
    if len(tool_calls) <= 0:
        return {"messages": []}

    tool_messages = []
    for tool_call in tool_calls:
        tool = tools_by_name[tool_call["name"]]
        result = tool.invoke(tool_call["args"])
        tool_messages.append(
            ToolMessage(content=result, tool_call_id=tool_call["id"])
        )
    return {"messages": tool_messages}
```

* 必须带 id 的理由：一次可能回去好几条工具消息，模型得知道哪条结果对应它哪一次调用，否则两个城市的天气就会对不上号；id 是模型发起调用时生成的，原样带回去即可。

* 返回值仍是字典，列表里的消息会被状态的 reducer 自动拼到已有消息后面，不用手动合并。

### 条件出口与循环

* 模型节点之后走向不确定：要调工具就去工具节点，能直接答就结束，所以这里是条件边而不是普通边。

* 判断依据就一条：看最后一条消息里有没有 `tool_calls`。

```python
def should_continue(state: MessagesState):
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"
    return END
```

```python
builder = StateGraph(MessagesState)
builder.add_node("llm", llm_node)
builder.add_node("tools", tool_node)

builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", should_continue, ["tools", END])
builder.add_edge("tools", "llm")

graph = builder.compile()
```

* `tools → llm` 这条边最容易漏：工具执行完必须回到模型，让它基于结果再想一次；漏了它就是死胡同，流程断在那里。

* 这条图的形状：`llm` 是唯一判断点，要么直接结束，要么去 `tools`；`tools` 没有选择权，必须回 `llm`。出口只有一个，就是模型不再要求调工具的那一刻。

* 消息列表按轮次生长：

```text
第 1 轮 llm 之后  :  Human, AI(tool_calls×2)
工具节点执行之后   :  Human, AI(tool_calls×2), Tool, Tool
第 2 轮 llm 之后  :  Human, AI(tool_calls×2), Tool, Tool, AI(最终答案)
```

* 状态在循环里不断变长，所以这个字段的 reducer 必须是 `add_messages` 这种追加语义，不能是覆盖。

### 预定义节点 ToolNode 与 tools_condition

* 整段里真正麻烦的只有两块：工具调用节点（取 name/args/id、循环、invoke、封 `ToolMessage`）和 `should_continue` 路由判断。

* 这两块完全通用，任何带工具的 agent 都要写一遍，因此 LangGraph 预置了 `ToolNode` 与 `tools_condition`。

* `ToolNode` 的第一个参数要传工具列表本身，不是工具名字符串；节点内部的活它替你干了，但它得知道有哪些工具、每个怎么调。

* `tools_condition` 内部就是"取最后一条消息、用 `hasattr` 看有没有 `tool_calls`、有就返回 `"tools"` 否则 `END`"；返回值映射它已约好，所以不用再传候选节点列表。

```python
from langgraph.prebuilt import ToolNode, tools_condition

builder.add_node("tools", ToolNode(tools))
builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", tools_condition)
builder.add_edge("tools", "llm")
```

* 换上之后整份代码缩成四步：定义工具 → 定义模型并绑定工具 → 定义模型节点 → 拼图，图的结构一点没变。

* **create_agent 的适用边界**

    * `create_agent` 一行就能得到等价的带工具 agent。

    * 会真正自己搭图的只有两种情况：公司业务自定义多、不想用这个黑盒；或要把细节攥在自己手里，比如工具节点加日志、权限校验、人工确认，模型节点前后插自己的处理。

    * 其余多数场景没必要手搭；但看懂这一层，别人给一张 agent 的图时至少知道那个环怎么转、哪段在解析、哪段在兜底。

## 工作流模式

* LangGraph 的核心价值是提高复杂智能体的稳定性：在完全由 AI 自主的 agent 之上再加一层编排，让它既有 AI 的能力，又按要求的路径做事。

* 官方给出的模式是 workflows 五种加 agents 一种，共六种，分两档：

    * 前一档模型只是工作流里的一个节点，走哪个节点、走几条分支全由你画的边决定。

    * 后一档不只让模型当节点，还让模型主导执行路径，走哪条路由模型说了算。

| 模式 | 谁决定走哪条路 | 形态 | 典型场景 |
| :--- | :--- | :--- | :--- |
| 串行 prompt chaining | 边写死 | 模型 A → 模型 B → 结束 | 先起草，再润色 |
| 并行 parallelization | 边写死 | 一份输入分多支，最后合并 | 同一份材料多角度同时处理 |
| 编排者 orchestrator-worker | 编排模型 | 拆任务 → 多个 worker → 汇总 | deep research 写研究报告 |
| 评估优化 evaluator-optimizer | 评估模型 | 生成 ↔ 评估，循环迭代 | 写小说、反复打磨 |
| 路由 routing | 路由节点 | 认意图 → 派到不同分支 | 一个入口处理多种任务 |
| agent | 模型自己 | 模型循环决定调什么工具 | 开放式任务 |

### 串行与并行

* 串行是最基本的 prompt chaining：调模型，把输出喂给下一个模型，最后结束，整条路径由边写死，模型没有选择权。

* 并行是 parallelization：同一份输入分给几个分支同时处理，再把结果合并。

* 两者的共同点是模型只是工作流里的普通节点，跟一个函数、一次数据库查询没有本质区别：路径由你画，模型负责干活。

### 编排者模式

* 编排者（orchestrator-worker）多一个专门负责编排的角色：把大任务拆成若干子任务，分给不同 worker 执行，最后汇总成一份答案。

* 典型场景是 deep research 这类研究报告 agent：一句"帮我写一份游戏开发研究报告"，它不会一口气写完，而是拆成市场调研、竞品分析、技术方案各一路，分给不同 worker，再合并成报告。

* 适用条件是任务复杂、能拆成几块互不依赖的子任务、最后需要统一答案。

### 评估优化模式

* 两个角色唱双簧：`evaluator`（评估员）与 `optimizer`（优化器）。先生成一版交给评估员，评估报告回给生成方照着改，改完再评，来回迭代到结果能看。

* 写小说是最贴切的场景：先写一章，评估员挑出节奏、人物、逻辑上的问题，模型带着问题重写。

* 它与 agent 循环的区别在判据归谁：agent 是自己判断"还要不要再调一次工具"，评估优化的循环判据在评估者手里，是另一个角色在说"还不行，再改"。

## 意图路由

* **路由（routing）**

    * > **定义**：接收用户输入，先识别这句话属于哪一类任务，再把请求引导到与该任务相关的节点或子流程上。

    * 本质是分工：活儿拆开分给不同节点，中间站一个路由节点专门判断类型，判完交给具体节点干活。

    * 电商客服是标准例子：用户可能问"还有货吗""我要退货""耳机坏了怎么赔"，背后要干的活完全不同，必须先分类再派发到售前咨询、退款、退货等节点。

* **需求设定**

    * 三个干活的节点加一个判意图的节点：天气、闲聊、翻译，外加 `route`。

    * 意图识别结果存进状态字段 `intent`，条件边据此决定进哪一个分支，各节点干完活把结果发给用户。

```text
   START ──▶ route ──┬──▶ chat（闲聊）─────────┐
   （用户请求进来）    │                        │
                     ├──▶ weather（查天气）────┤
                     │                        │
                     └──▶ translate（翻译）────┤
                                              ▼
                                             END
```

### 状态与意图识别节点

* 状态直接继承 `MessagesState`，再补一个 `intent` 字段：`messages` 的能力与 `add_messages` 的追加语义自动具备，自己从零写反而要重复 `Annotated` 那一堆。

```python
from langgraph.graph.message import MessagesState


class IntentState(MessagesState):
    intent: str
```

* 意图判断只能交给模型：用户只说一句话，代码判断不出这句话要干什么。

* 节点做法：写一条系统提示词，明确列出可选意图 `weather`、`translate`、`chat`；从 `state["messages"]` 取最后一条，点 `content` 拿到用户原话。

* 只发"最后一条用户消息 + 系统提示词"，不要把整个消息列表塞进去：意图识别只需要最后一条，前面的历史带过来纯属浪费。

* 提示词里限定三个值是有回报的：三个节点的名字刚好就是 `chat`、`weather`、`translate`，模型输出的意图值直接等于节点名，后面省掉一堆映射。

* 模型返回的 `AIMessage` 不放进消息历史：它只是意图判断，不是给用户的答案；只把 `response.content` 取出来存进 `intent`。

### 路由函数与分支节点

* **路由函数**

    * 模型已经判完并存进 `intent`，而 `intent` 的取值刚好就是三个节点名，所以路由函数不需要 `if/elif`，直接把 `intent` 返回即可。

```python
from typing import Literal


def intent_router(state: IntentState) -> Literal["chat", "weather", "translate"]:
    return state["intent"]
```

    * 返回值上的 `Literal` 只标一下有三种情况，起说明作用。

    * 条件边挂法是 `add_conditional_edges("route", intent_router)`，三个分支节点各自 `add_edge(..., END)`，最后编译。

* **分支节点**

    * 闲聊节点把 `state["messages"]` 整包丢给模型，返回的 `AIMessage` 塞回消息列表；虽然节点里没看到 `messages` 字段，但继承 `MessagesState` 后它本来就在。

    * 天气节点在拿不到真实接口时直接写死一条 `AIMessage` 占位，等有了真工具再换掉，不丢人。

    * 翻译节点的系统提示词要交代两件事：按用户要求翻译、直接返回译文不要任何解释。

* > **提示**：写 LangGraph 就是一行一个节点、一个节点一个函数，写完一起拼；别一开始就把所有逻辑塞进一个函数里。

## 子图嵌套

* **父图与子图**

    * > **定义**：把复杂工作流拆开，最外层那张图叫父图，内部被复用的独立 graph 叫子图（subgraph）。

    * 拆分动机：真实业务的工作流往往非常庞大，全堆在一张图里不利于维护，图会越画越不成样子。

    * 拆分后有两个直接好处：每张图都短，单独看得懂、改得动；任何节点都可插拔，想替换哪个就替换哪个，整个系统组件化。

    * LangGraph 支持把已编译好的子图直接当作父图的一个节点使用。

### 两种嵌法

* **直接嵌入**

    * `add_node` 第二个参数除了填自定义函数，也可以填一个别人写好的 graph，相当于把另一张图拿来当自己的一个节点。

    * 父与子的 checkpoint 自动复用、两者共享：父图 checkpointer 里与状态里存的东西子图能用。

    * 子图里发生的事件也能被父图捕捉，自定义事件与 `interrupt` 事件都可以自动向上冒泡。

    * 代码分开写、维护独立，运行时又像一张图，既好维护又保持一体感。

* **包装调用**

    * 不把子图当节点，而是在父图节点函数里用 `subgraph.invoke(...)` 手动调用，把子图当工具使。

    * 代价明显：复杂，而且父子之间什么都不共享；数据传递要自己从状态里取、自己拼成子图要的格式、再把结果取出来拼回父状态。

    * 两套状态、两套 checkpoint 各维护各的。

```python
def weather_node(state: IntentState):
    result = weather_agent.invoke({"messages": state["messages"]})
    return {"messages": [result["messages"][-1]]}
```

* > **结论**：两种模式里更推荐直接嵌入；做父子拆分只是为了把任务拆开，它们本质上还是同一个 agent，没必要父一套子一套各维护自己的 checkpoint，除非确有这种业务需求。

### 直接嵌入的数据共享规则

* 第一条，父图自动把自己的 key 共享给子图：父里有的字段，子图也能读到、能用。

* 第二条，子向父写回：子图更新状态时，若父图也有同名字段，就按父图的 reducer 写回——子图可以写父图的数据，但写回方式听父图的。

* 第二条的实际效果：父图与子图都有 `messages` 时，子图更新不会覆盖父图消息，而是走父图的 `add_messages` 追加，于是子图的消息自然拼进父图的消息流，父子看起来共享同一份会话。

* 反向规则：子图修改的 key 若是它自己特有、父图没有的，对父图完全不可见，不影响父图；这给了子图自由度，可以有自己的任务与字段。

* 所以共享策略很灵活：想共享，字段就起一样的名字，甚至直接继承父图的状态；不想共享，就起自己的名字，互不干扰。

* 子图状态至少要能承接消息，因此让它继承 `MessagesState`；子图特有字段想加就加，父图看不见。

* **用 agent 当子图**

    * `create_agent` 创建出来的 agent 返回值是编译好的图，天然就能当子图用，不必手写工具节点那一套。

    * 定义只需两样：模型加工具；创建完先单独 `invoke` 验证它能跑，注意必须传字典、字典里 `messages` 是装了 `HumanMessage` 的列表，不能直接传字符串。

    * 换进父图只是把原来写死的节点函数替换成这个 agent：`builder.add_node("weather", weather_agent)`。

* 功能复测三条分支都要过：闲聊正常回、翻译正常回、天气走子图查真实结果，父子才算真正融合。

## 多 Agent 编排

* **Think in LangGraph**

    * 开发 LangGraph 的 agent 或图，必须用 LangGraph 的思维看业务：最基本的概念只有节点、边、状态，其中节点与边决定整张图的结构。

    * 结构受业务流程影响：业务流程里"先干嘛、再干嘛"就是一个个节点，它们之间的先后顺序就是边。

    * 分析路径是三步：按业务边界划分职责，一个节点负责一个职责，节点就出来了；按业务流程顺序定义节点执行顺序，边就出来了；节点和边都有，图就有了。

* **落地五步**

    * 先用 LangGraph 的方式把业务看一遍并做拆分，想清将来有哪些节点、整体结构长什么样。

    * 再按分析结果创建状态，然后一个个实现节点，最后用边连成图。

    * 收尾把整个 graph 交给项目里的 FastAPI 做生命周期管理。

* **职责即子图**

    * 保险智能顾问的职责远不止聊天与咨询，还包括产品咨询、保险组合方案推荐、产品投保、理赔咨询、复杂问题转人工，每一条背后都有相当复杂的流程。

    * 例如方案推荐要先获取用户画像、再推荐、再确认是否满意，满意就保存、不满意重新生成；投保要经过保费试算、健康告知、保单信息填写、订单确认支付、生成保单。

    * 所以每个职责都应由独立智能体完成，六个职责就是六条独立任务线、六个独立 agent；它们组合成一个大 agent 时，各自是子图，组合层是父图。

    * 父 agent 只做一件事：意图识别，然后把信息分给不同子 agent 去完成。

* **不塞进一个 agent 的理由**

    * 一是 agent 会变得非常臃肿，可控制性变差，同一个 agent 一会儿做投保、一会儿做理赔咨询。

    * 二是所有功能揉在一起数据会很多、容易产生冲突；内容越来越长之后，AI 分析时越来越容易出幻觉。

    * 职责划清、由多个 agent 去做、再由统一编排者分配工作，才是要的结构。

### 子图划分与节点形态

* 为控制复杂度只做五个子图：闲聊、保险方案咨询与推荐、理赔、转人工、fallback；加一个 `route` 共六个节点。

* `fallback` 的职责是意图识别失败时再追问一句、补充信息；闲聊也要有，人工客服都常碰到闲得没事来找人聊天的用户，AI 客服同样需要这个出口。

| 节点 | 英文名 | 职责 |
| :--- | :--- | :--- |
| 意图识别 | `route` | 识别用户意图，决定往哪个子图走 |
| 保险方案咨询与推荐 | `insurance_advisor` | 推荐方案、咨询条款、生成方案 |
| 理赔 | `claim` | 保单查询、理赔查询、订单进度查询、理赔指引 |
| 闲聊 | `chat` | 承接闲聊 |
| 转人工 | `human_handoff` | 复杂问题转人工 |
| 兜底追问 | `fallback` | 意图识别失败时追问补信息 |

```text
                        ┌──────────────┐
          START ──────▶ │  route node  │  意图识别
                        └──────┬───────┘
                               │ 条件边
      ┌───────────────┬────────┼─────────┬───────────────┐
      ▼               ▼        ▼         ▼               ▼
 insurance_        claim     chat   human_handoff    fallback
 advisor          （理赔）  （闲聊）   （转人工）      （追问补信息）
（咨询与推荐）
      │               │        │         │               │
      └───────────────┴────────┴─────────┴───────────────┘
                               ▼
                              END
```

* 业务节点有三种实现形态，都成立：普通函数、LangChain 的 agent、LangGraph 的图；节点的本质通常是函数，但塞一张图或一个 agent 进去照样能跑。

* 这五个子图之间还会共享一些工具。

### 意图识别的状态字段

* 父图做意图识别，首先要有历史消息，会话记忆靠 `messages`；其次要存意图识别所需的信息，但不能只存"AI 识别出的意图是什么"。

* 原因：复杂多轮交互里用户输入可能极其简单。正在做保险推荐时问"你是否确定这套方案"，用户只回"确定"三个字，单独把这句话发给模型，它判断不出属于哪个子工作流。

* 要识别得更准，除了本轮这句话，还得告诉模型上一轮工作流是哪一个，并把上一轮 AI 返回的内容也带上。

* `route` 节点需要的三个字段：

    * `messages`：跨工作流共享的会话消息，负责会话记忆，也是意图识别的原材料。

    * `previous_workflow`：上一轮对话的工作流，用来消解"确定""好"这种无信息量输入。

    * `active_workflow`：本轮对话的工作流。

* `active_workflow` 有两个作用：本轮意图识别的结果要存下来，后面才知道往哪儿路由；它同时是下一轮推算 `previous_workflow` 的唯一依据。

* 时序捋法：

```text
第 N 轮
  请求进来
    └─ 意图识别之前：active_workflow 里存的还是第 N-1 轮的工作流
       └─ 把它抄进 previous_workflow，连同 messages 一起交给模型
          └─ 识别完成后：active_workflow 改写为第 N 轮的工作流
             └─ 按 active_workflow 路由到对应子图

第 N+1 轮
  请求进来
    └─ 意图识别之前：active_workflow 天然就是第 N 轮的工作流
       └─ 于是又能直接抄进 previous_workflow ……
```

* 关键点：本轮意图识别完，`active_workflow` 记录的是本轮工作流；等下一轮请求进来、在意图识别之前，它装的就成了上一轮的工作流。

* 因此记录本轮工作流的深层目的，是让下一轮能知道上一轮是谁，从而写进 `previous_workflow`。

* 反过来说，如果没有 `active_workflow` 这个字段，"记住上一轮工作流是谁"根本做不到，上一轮的值没有地方落，所以这个字段必须有。
