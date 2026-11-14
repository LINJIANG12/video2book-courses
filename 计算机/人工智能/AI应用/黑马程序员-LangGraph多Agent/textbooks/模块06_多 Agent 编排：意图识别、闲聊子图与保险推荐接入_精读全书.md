# 黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目·多 Agent 编排：意图识别、闲聊子图与保险推荐接入（第 6 册 / 共 6 册）

> **所属课程**：黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目  
> **本册内容**：多 Agent 编排：意图识别、闲聊子图与保险推荐接入  
> **覆盖范围**：P184-P186 ~ P190-P193（共 10 讲 / 3 章 / 139 分钟音频）  
> **分册说明**：全书按内容分 6 册；本册为第 6 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 保险顾问主图：从状态定义到启动装配（P184-P186）
2. 意图识别：先用规则挡一道，再让模型判一次（P187-P189）
3. 把闲聊做成子图，再把保险推荐接回来（P190-P193）

---

## 保险顾问主图：从状态定义到启动装配
> 对应块：BLK49 | 覆盖分集：P184-P186 | 块标题（分集名）：《保险顾问主图：State、构建与生命周期》

图已经画好了，接下来就是照着这张图把它整成代码。先做主的这一张，子图先放一边——主图是整个系统的入口，所有用户请求都从它这儿进。

真动手你会发现，第一步不是写节点。节点是函数，函数要收参数，参数得有类型，这个类型就是 state。所以顺序是：先定 state，再定节点，再用边把节点连起来，最后编译成一张能跑的图。这一篇就按这个顺序走。

### 文件先落对地方：agents 下的 schema

要改的是 agents 这一块。现在写的代码全是 agent 相关，那就统一放进 `agents` 包；state 属于 graph 内部的数据约束，再往下开一层 `schema`，新建 `app/agents/schema.py`。

这里有个小争论值得说一句：state 到底算 schema 还是算 model？叫 model 我觉得也可以。但在我们项目里，model 这个词更强调跟数据库打交道的那一层，而这个 state 根本不落库，它是模型内部、graph 内部传数据用的，跟前端也不交互。所以我选 schema，含义就是"graph 内的数据约束"。

> **提示**：这不是什么硬性要求。团队里叫 model 也不会错，关键是整个项目口径统一——今天叫 schema、明天叫 model，读代码的人就得多跳一次。

这个文件里要装三样东西：意图路由的常量、意图识别的结果、主 graph 的 state。东西不多，但每一样都有它的来路。

### 把五个意图收成一套常量

第一样，先把意图定义成一套常量。

主图往下分五个子节点：闲聊 `chitchat`、保险方案推荐 `recommendation_plan`、理赔 `claim`、转人工 `human_handoff`，还有意图识别失败时的兜底 `fallback`。五个子节点就是五条去向，也就是五种意图。

为什么要把它们抽出来？因为意图识别节点要声明返回值类型。你还记不记得，这个节点不是普通节点——它得告诉图"我下一步去哪"，所以返回类型要声明成一个 Literal，把这五个值钉死。要是不抽出来，那个 Literal 就得在注解里写一遍、在别处再写一遍，五个字符串抄来抄去，抄错一个字母，就是一个运行时找不到节点的错误。

抽出来之后长这样：

```python
# app/agents/schema.py
from typing import Literal

Intent = Literal["chitchat", "recommendation_plan", "claim", "human_handoff", "fallback"]
```

以后写返回值，类型直接写 `Intent` 就行，不用把五个字符串再摆一遍。`intent` 这个词本身就是"意图"的意思，拿来当名字也贴切。

> **易错点**：这五个字符串不只是"意图的名字"，它们同时是**节点的名字**。后面 `add_node` 用什么名字、`goto` 就往哪儿跳，两边必须一模一样。少一个下划线、多一个字母，图编译的时候不会提醒你"你是不是想写另一个"，它只会在运行时报找不到节点。

### 意图识别的结果为什么是一个类

第二样，意图识别的结果，我定成一个类，而不是一个字符串。

你可能会问：上一把演示的时候，意图识别的结果不就是个字符串吗？这个字符串不就是那五种之一吗？现在何必还写个类？

这个疑问很自然。答案是：在企业里正经做意图识别，它就是一个类，而且字段很多，七八个是常态。我们这里已经大幅简化了，只留两个：

```python
class IntentResult(BaseModel):
    intent: Intent          # 真正的意图，只能是五种之一
    reason: str             # 判断成这个意图的理由
```

`intent` 的类型就是刚定的 `Intent`，也就是说这个字段只能填那五种之一，填别的进不来。

`reason` 记的是"为什么判成这个意图"。为什么要记这个？因为意图识别是 AI 做的，咱总得知道它凭什么这么认为。把它记下来，将来就是一条日志；万一哪天意图识别错了，手上至少有它的判断依据，排查起来有抓手。

除了这两个，真实项目里通常还会有：

- **置信度**：让 AI 给自己打个分——你这次识别正确的概率大概是多少；
- **风险标记**：一个布尔值，用户这次说的话有没有风险；
- **子意图**：父意图底下的细分。比如判成"理赔"，还得知道是理赔里的哪一种。

字段可以很多，但你不用担心谁来填。这些字段最终都是 AI 按我们的结构约束生成的——给模型绑定结构化的输出约束，它返回的结果就必须长成这个类的样子。字段是八个还是十个都一样，AI 来填。

> **提示**：这也是为什么用类而不是字符串。字符串只能表达"是什么"，类能同时表达"是什么、为什么、有多确定、有没有风险"。字段越全，后面能做的事越多——路由可以按置信度降级，风控可以按风险标记拦截。

所以等会儿写意图识别的时候，就用结构化输出把模型约束成这个形状。

### 主 State：消息历史加上两轮工作流

第三样，主 agent 的 state。

前面分析过，这张图里要流通两类信息：一类是对话消息，一类是工作流。所以 state 长这样：

```python
class InsuranceAgentState(MessagesState):
    previous_workflow: Intent   # 上一轮的工作流
    active_workflow: Intent     # 本轮的工作流
```

继承 `MessagesState`，它自带 `messages`，消息这一块不用我们再写。剩下两个字段，一个记前一轮走到哪条分支，一个记本轮走到哪条分支。

两个字段的类型都写 `Intent`，因为不管上一轮还是本轮，结果一定是那五种之一，不会跑到别处去。这样一简化，state 的定义就很干净。

> **提示**：`previous_workflow` 和 `active_workflow` 都不用加 `NotRequired`。它表示"不是必须的"，这里没必要——类型标注就够了；字段在首轮可能没有值，这件事留到读取的时候处理，后面我们就会踩到这个坑。

到这里，schema 里的三样东西齐了，第一步搞定。

接着新建主图自己的文件。这个文件我给它起名叫 `orchestrator`——这单词你可能没见过，查字典第一反应是"管弦乐家"，那不对。它在专业语境里是"编排""协调"的意思：一张图协调用户的请求该进哪条分支，干的正是编排者的活。

所以文件叫 `app/agents/orchestrator.py`，图本身也叫编排者。名字这事值得较真。有些人起名喜欢 `A.py`、`B.py`，那样不够专业——是谁我不说。将来你回头看这个目录，`orchestrator` 一眼就知道它是干什么的。

把 schema 里那三样 import 进来，就可以往下写节点了。

### 六个节点，先写假的

有了 state，接着按图写主 graph 的整体结构：定义节点，定义边，用边把节点连起来。就这两步。

节点本身没什么玄机——节点就是函数，一个函数挂上去就是一个节点。六个节点，就是六个函数。

主图一共六个节点：`route` 是意图识别路由节点，所有请求的第一站；`recommendation_plan` 管保险方案推荐与咨询；`claim` 管理赔咨询；`chitchat` 管闲聊；`human_handoff` 负责转人工；`fallback` 是意图识别失败时的兜底。六个节点，五个分支，一个入口。

麻烦的是内容——推荐怎么做？理赔怎么答？现在还不知道，因为这几个能力还没做。所以先写假的：假装意图识别已经做完了，假装推荐、理赔都已经能回答。这叫占位实现，先把骨架立起来，让图能跑通，回头再把真逻辑塞进对应的函数里。

假归假，每个函数该收什么、该返回什么，一点都不能假。所有节点的入参都是 `state`，返回值都是要并进 state 的内容——这一条对六个节点一视同仁。

五个分支节点的写法几乎一样：直接 return 一条 AI 消息。路由节点是例外，后面单说。

```python
def recommendation_plan_node(state: InsuranceAgentState):
    return {"messages": [AIMessage(content="保险方案推荐功能尚未完成")]}
```

推荐节点就回一句"保险方案推荐功能尚未完成"。理赔节点同理，回"理赔咨询功能尚未完成"。

闲聊节点可以稍微有点人味：

```python
def chitchat_node(state: InsuranceAgentState):
    return {"messages": [AIMessage(content="哈哈，你又来了，欢迎你来到安心宝商城，今天聊点啥")]}
```

转人工节点回"你的请求已转人工，请耐心等待"。兜底节点回"无法识别你的意图，请再说一遍"——措辞你可以再友好一点，但道理就是这个道理：意图没认出来，就让用户再说一遍。

为什么这些节点都返回 `messages` 而不是别的？因为我们知道，这几个能力最后都是给用户一句话，一句话就是一条消息，消息就存在 state 的 `messages` 里。占位的时候把形状写对，将来换成真的实现，外面的结构一行都不用动。

### 路由节点：用 Command 同时办两件事

六个节点里，路由节点最特殊，值得单独说。

它要干两件事：第一，把本轮的路由结果写进 state；第二，告诉图下一步去哪个节点。普通节点只能 return 一个字典来更新 state，没法指定"往哪走"；要指定走向，常规做法是再写一个路由函数，配上条件边。

但我们不想要那条额外的路。能不能让路由节点自己就把下一站说了？能——用 `Command` 把它包起来。

```python
from langgraph.types import Command

def route_node(state: InsuranceAgentState) -> Command[Intent]:
    previous = state.get("active_workflow", "")
    active = "chitchat"      # 假装意图识别做完了，永远去闲聊

    return Command(
        update={"previous_workflow": previous, "active_workflow": active},
        goto=active,
    )
```

`Command` 里有两块：`update` 是要合并进 state 的内容，`goto` 是要去的节点。原来返回字典能做的事，现在装进 `update` 里照样做；`goto` 是新增的能力，直接指名下一站。

返回值类型标成 `Command[Intent]`，等于把那个 Literal 写在了这里——LangGraph 看到这个泛型参数，就知道这个节点可能跳向这五个节点中的任意一个，五个分支的边也就自动生成了。我们一条条件边都不用手写。

> **易错点**：`Command` 要从 `langgraph.types` 里导，别从别的地方随手导一个同名的。导错了不会立刻报语法错，而是运行到这一步才炸，排查起来很费时间。

`goto` 写的是 `active`，也就是本轮的路由结果。路由结果就是"要去的地方"，这两个东西本来就是同一个值——想清楚这一点，这个节点就没有疑问了。现在 `active` 是假的，固定写 `"chitchat"`，所以不管用户说什么，都进闲聊。

### previous 与 active：上一轮的结果藏在哪儿

刚才那段代码里有一行值得拎出来看：

```python
previous = state.get("active_workflow", "")
```

为什么"上一轮的工作流"要从 `active_workflow` 里取？这不是取错字段了吗？

没有取错。想一下请求的时序：

```text
第 1 轮
  请求进来  ->  route_node 读 state["active_workflow"]（此时还没有值）
            ->  写入本轮结果，比如 chitchat
            ->  图跑完，state 里存着 active_workflow = chitchat

第 2 轮
  请求进来  ->  route_node 读 state["active_workflow"]
               这个值还是上一轮写进去的 chitchat —— 它就是上一轮的结果
            ->  再写入本轮的新结果，把它覆盖掉
```

图里的 state 是持久的，上一轮结束时 `active_workflow` 里存的是上一轮的结果。第二轮请求进来，在意图路由**真正开始识别之前**，这个字段还什么都没被改写，所以它此刻的值，恰好就是上一轮的结果。

所以在路由节点开头把它读出来、存进 `previous_workflow`，读的虽然是 `active_workflow`，拿到手的却是"上一轮"。

> **结论**：每一轮路由都会产生一个新的 `active_workflow`；而上一轮的 `active_workflow`，在下一轮路由执行前，就是这一轮的 `previous_workflow`。同一块数据，在两个时间点上是两个含义。

这个逻辑得想明白。想明白了，`previous_workflow` 根本不需要额外维护——不用在图末尾再挂一个节点去搬运，路由节点自己顺手就把它记下来了。

> **提示**：这也是这里要用 `.get()` 而不是 `state["active_workflow"]` 的原因。第一轮请求进来的时候，这个字段压根不存在，直接下标取值会报错。这个问题等会儿真跑起来就会撞上，先留个印象。

### 建图：加节点、连边、编译

六个函数写完，轮到建图。这一步就是把上面那些节点按图连起来：

```text
START
  |
  v
route  --->  recommendation_plan  ---+
  |                                  |
  +------->  claim  -----------------+
  +------->  chitchat  --------------+
  +------->  human_handoff  ---------+
  +------->  fallback  --------------+
                                     |
                                     v
                                    END
```

读图注意两处：`route` 往下的五条箭头不是手写的边，是 `Command[Intent]` 声明出来的；五个分支各自直连 `END`，中间没有汇总节点。

写法上有个讲究：不要在这个模块的顶层直接 `builder = StateGraph(...)` 然后一路建下去。那样写，模块一被 import，图立刻就建好了——图是 agent，agent 应该在项目启动时加载一次，而不是谁 import 谁就建一张。所以把建图这件事包进一个函数：

```python
def init_insurance_agent(checkpointer):
    builder = StateGraph(InsuranceAgentState)
    ...
    return builder.compile(checkpointer=checkpointer)
```

将来在 FastAPI 的生命周期钩子里调它，启动时建一次，之后一直用。这段代码就成了"可被调用的初始化逻辑"，而不是"import 的副作用"。

函数里第一步是构造器。`StateGraph` 要收 state 的类型，把 `InsuranceAgentState` 传进去，builder 就知道这张图的数据长什么样。

然后一个一个加节点：

```python
builder.add_node("route", route_node)
builder.add_node("recommendation_plan", recommendation_plan_node)
builder.add_node("claim", claim_node)
builder.add_node("chitchat", chitchat_node)
builder.add_node("human_handoff", human_handoff_node)
builder.add_node("fallback", fallback_node)
```

`add_node(名字, 函数)`。名字这块一定要跟意图常量保持一致——`goto` 跳的是节点名，名字对不上就跳不过去。所以这几个名字别手打，直接从常量那边复制，省得抄错。

节点齐了就连边。入口是 `START`，第一站是路由：

```python
builder.add_edge(START, "route")
```

路由往下的五条边不用写，刚才 `Command[Intent]` 已经把它们生成了。剩下的是五个分支各自到 `END`：

```python
builder.add_edge("recommendation_plan", END)
builder.add_edge("claim", END)
builder.add_edge("chitchat", END)
builder.add_edge("human_handoff", END)
builder.add_edge("fallback", END)
```

这里插一句：写这几条的时候很容易漏。我写的时候就漏了闲聊那一条，图看着"差不多连完了"，实际上少一个分支的出口。加完之后最好对着图数一遍——六个节点，一个入口、一个路由，剩下五个都得有去处。

五个分支都直连 `END`，是因为在我们这张图里，每个子节点自己就能把结果返回给用户，不需要再汇总。这跟"所有分支最后汇到一个汇总节点"的画法不是一回事，别混。

那真实生产里是不是也这么连？不一定。生产环境往往会在 `END` 前面再加一个节点，作用是一道安全围栏：对 AI 生成的内容做一次校验，看有没有隐私泄露、有没有安全风险。比如有人问了违法的问题，这种内容不能就这么返回给用户，得拦下来，直接回一句"你问的问题不合法"之类。校验通过才放行。

有些系统做得更细，在前面再加一道，对用户输入也做校验。前后两层，就是常说的安全围栏，专门保护 agent。我们这里不加，加了代码会复杂不少，但你得知道结构上可以这么长——无非是让子节点先到围栏节点，再到 `END`，中间多一层防护。

最后编译：

```python
return builder.compile(checkpointer=checkpointer)
```

`compile` 出来的才是能跑的图。传了 `checkpointer`，图就有了会话记忆——同一个会话的多轮对话，state 能续上，`previous_workflow` 那套逻辑也才成立。

但这个 checkpointer 不是我们自己在这里创建的。项目里 FastAPI 那边已经把 checkpointer 初始化好了，直接拿来用就行。所以它由调用方传进来：谁调用 `init_insurance_agent`，谁负责把 checkpointer 递给我。这样这个函数不关心 checkpointer 从哪来、怎么配，只管用。

### 生命周期：把主图挂到启动钩子上

主图构建完了，但它现在还不能用——得有人把它创建起来。创建的时机是项目启动时，用的就是 FastAPI 的 lifespan。

打开 `main.py`，lifespan 里原本有一个初始化 agent 的动作。那个 agent 是以前的版本，只做了保险推荐这一件事，别的能力都没有。现在别初始化它了，删掉；要初始化的是我们刚写的编排者。

```python
from app.agents.orchestrator import init_insurance_agent

@asynccontextmanager
async def lifespan(app: FastAPI):
    # ……checkpointer 等初始化保持原样
    app.state.agent = await init_insurance_agent(checkpointer)
    yield
    # ……收尾
```

名字跟以前那个一样，但包不一样——以前在别处，现在在 `app.agents.orchestrator` 里，导的时候别导错。

`init_insurance_agent` 在这里要 `await`，所以它得写成异步函数。初始化时把 checkpointer 传进去，编译好的主图返回出来，存进 `app.state`。

存进 `app.state` 之后，别的代码就不用动了。对话接口那边是从 `request.app.state.agent` 取 agent 的——现在取到的还是以前那个保险顾问吗？不是了，取到的就是我们这张主图。

> **结论**：换 agent 不用改接口。接口只认"`app.state` 里有个能对话的 agent"，至于它是单个 agent 还是一张编排图，接口不关心。这正是把初始化收进 lifespan 的好处——装配点在启动时，调用点在运行时，两边解耦。

启动项目，再打开前端，聊天的时候就是在跟主图聊了。

### 跑起来：一个空值引发的报错

前后端都起来，checkpointer 也初始化好了。新建一个咨询，输入一句话发过去——报错了。

报错说的是 `active_workflow` 取不到。第一反应是单词写错了？看了一眼，没有。那就是没有值。

这就对上了：`active_workflow` 不是必填字段，第一轮请求进来，state 里根本没有这个键。路由节点里那句 `state["active_workflow"]` 直接下标取值，取不到就抛异常。

改成 `.get()`，给个默认值：

```python
previous = state.get("active_workflow", "")
```

没有这个键就返回空字符串，不抛异常。改完再跑。

再发一句"hi"。这回回来了：

```text
哈哈，你又来了，欢迎你来到安心宝商城，今天聊点啥
```

这句话眼熟吧——就是闲聊节点里我们写的那句占位回复。说明什么？说明请求进来先到了 `route`，路由把本轮结果写成 `chitchat`，`goto` 指向闲聊节点，闲聊节点返回这句话，然后到 `END`。

整条链路一个节点都没落下。主图跑通了。

> **承前启后**：上一节讲完「保险顾问主图：从状态定义到启动装配」，下一节接着讲「意图识别：先用规则挡一道，再让模型判一次」。

---

## 意图识别：先用规则挡一道，再让模型判一次
> 对应块：BLK50 | 覆盖分集：P187-P189 | 块标题（分集名）：《意图识别与寒暄兜底》

前面几讲把多 Agent 的骨架搭起来了，可编排图里那个「意图识别」一直是假的——不管用户说什么，都被送到同一个地方。这一讲就把它做成真的。

做之前先把顺序想清楚。很多人一听「意图识别」，第一反应就是「把用户这句话丢给模型，问它属于哪一类」。这只能算三分之一。真正落到生产里的意图识别，是一套有先后次序的流程。

### 完整的意图识别是三段

整条链路是这样排的：

```text
用户请求
   │
   ▼
规则判断是否寒暄 ──命中──▶ 返回固定回复 ──▶ 结束
   │
   │ 不是寒暄
   ▼
模型意图识别 ──正常──▶ 按 intent 路由 ──▶ 结束
   │
   │ 报错 / 访问失败
   ▼
兜底：备用模型或传统代码识别 ──▶ 结束
```

读图：请求进来先过一道规则，是寒暄就直接短路掉；不是寒暄才交给模型；模型这一层还要再判一次它自己有没有出错，出错就走兜底。三段串起来，才是完整的意图识别。

第一段是**寒暄识别**。什么叫寒暄？用户就发了句「嘿」「你好」「再见」「谢谢」，这种消息有没有必要让模型去回答？没有。让模型回答反而是浪费钱。所以这类对话不去调模型，我们用某种匹配方式判断出它属于寒暄，直接返回一句固定结果。这叫寒暄识别——**用代码识别，不用模型**。这是意图识别里首先要做的事。

第二段是**模型识别**。确认不是寒暄、是正式对话了，才交给模型。这一步往往不用特别大的模型，用些小模型就够了——意图识别本来就是件简单的事，上超大模型没有必要。通常可以交给本地小模型，用 Ollama、vLLM 之类跑起来。但在咱们这个项目里不折腾那么多模型，从头到尾就一个 deepseek。等你到了公司，公司里肯定有一堆模型，有大的有小的，小的做简单的事，大的做复杂的工作，到时候你再换就完了。

第三段是**兜底方案**。意图识别模型出故障了、访问失败了，就改用备用模型，或者干脆退回到传统代码去做识别。相当于一个保险。

> **提示**：这三段的顺序不是随便排的。先便宜后昂贵、先确定后不确定——规则匹配几乎零成本，模型调用要花钱还可能失败，兜底则是给模型调用上一道保险。这个思路在工程里反复出现。

方案说全了，但这一讲不一次性做全。我打算先把模型识别写出来，跑通，再回来补寒暄和兜底。原因很简单：代码是一步步长出来的，一次把三套都写进去，出了问题你都不知道是哪一段的锅。

### 把意图识别收进一个专门的类

模型识别这段代码会比较长——方案有那么多，代码自然就多。如果直接写进编排图那个方法里，那个方法会变得超级长，以后没法看。

所以新开一个文件，叫 `intent_router.py`，把意图识别的代码都放进去，将来编排图里直接调这个文件里暴露出来的东西就行。

```text
app/
└── agents/
    ├── orchestrator.py     ← 编排图，这里只负责"调用"
    └── intent_router.py    ← 意图识别，规则 + 模型 + 兜底都在这
```

读图：识别逻辑和编排逻辑分家。编排图关心的是「识别完往哪走」，识别模块关心的是「这句话到底属于哪一类」，两件事互不干扰。

这个文件里将来要装三套东西：寒暄识别、模型识别、备用识别。三套代码放一块儿，我的建议是收进一个类里，类名就叫 `IntentRouter`。

构造函数里先准备模型——意图识别离不开模型：

```python
from langchain.chat_models import init_chat_model

from app.config import settings


class IntentRouter:
    def __init__(self):
        # 意图识别离不开模型，先把模型准备好
        self.model = init_chat_model(
            "deepseek-v4-flash",
            api_key=settings.LLM_API_KEY,
            extra_body={"thinking": {"type": "disabled"}},
        )
```

几点说明。`api_key` 要手动传，因为咱们现在没有配环境变量，模型不会自己从环境里读；这个 key 从哪儿来？从 `app.config` 里的 `settings` 取，就是配置里 LLM 那一段的 API key。`extra_body` 里把 `thinking` 的 `type` 设成 `disabled`，是把这个模型的思考过程关掉——判断一句话属于哪一类，用不着它在那儿长篇推理。

> **提示**：等你到了公司，这个位置填的多半就是个小模型了，本地跑的可能就是 Ollama 或 vLLM 起的服务。换模型只改这一处，因为识别逻辑都挂在 `self.model` 上。

模型初始化好了，但能不能直接拿它做意图识别？理论上可以——上一讲在 LangGraph 里就是这么干的，直接 `invoke`，让它返回一个字符串。可现在不行。我们早就定义好了一个意图识别的结果对象 `IntentRouteResult`，里面装着 `intent` 和 `reason` 两个字段。也就是说，意图识别的输出是**有结果约束**的，不能放任模型自由发挥成一个字符串。

怎么把约束挂上去？用结构化输出：

```python
        # 给模型绑定结构化输出，让它按要求返回路由结果对象
        self.route_model = self.model.with_structured_output(IntentRouteResult)
```

`with_structured_output` 的作用，是把「你要按什么格式交答案」变成模型调用的一部分：schema 里有哪些字段，模型就按这些字段产出。挂上之后，再做意图识别，它返回的就不再是字符串，而是 `IntentRouteResult` 这个对象本身，你连解析都省了。这个绑定好的模型我存进 `self.route_model`，给它起了个直白的名字——路由模型。

> **提示**：结构化输出不改变模型「判断意图」的能力，它约束的只是「结果怎么交出来」。所以它解决的是工程问题，不是准确率问题，别指望绑了它就判得更准。

构造函数到此写完，下面该写路由函数了。

### 上下文：光有本轮这句话不够

路由函数叫 `route`，它要干的事是「根据上下文完成路由」。

先把这个词拆开。上下文其实就是两部分：第一部分是这一次用户的问题，第二部分是历史消息。为什么非得带历史？因为只靠用户这一轮的提问，很多情况根本判不了。

你想想自己用 AI 编程工具的时候，聊着聊着它问你一句「要不要继续」，你回一个「好的」「确认」「去做吧」——单看这三个字，任何人、任何模型都不知道你要它继续干什么。必须往回看。

所以 `route` 的签名里有两个参数：`query` 和 `context`。`query` 是用户本轮的提问，`context` 是历史对话的信息。`context` 里可以放两样东西：

- 上一轮工作流的名字；
- 上一轮 AI 回复的消息。

两样结合起来，再结合用户本轮的提问，能不能判断出本轮该走哪个工作流？能。举个例子：上一轮 AI 说「我给你推荐了这样一套方案，你看一下怎么样」，用户回了一个「好的」，而上一轮的工作流是方案推荐——三个信息一碰，立刻就知道用户本轮其实还在方案推荐这条线上，不是要开新话题。

上一轮工作流的名字在 state 里已经有了，就是 `previous_workflow`，直接取，不用另存。

### 上一条 AI 消息不能想当然取倒数第二条

取上一轮的 AI 回复，脑子里第一个冒出来的写法是 `state["messages"][-2]`：最后一条是本轮用户消息，那倒数第二条不就是上一轮 AI 的回复吗？

听起来好像可以，但你忽略了一个问题：在用户消息之前，中间可能还夹着别的类型的消息。比如 AI 发起的工具调用、工具返回的结果，乱七八糟什么都可能有，咱们也不确定。真按 `[-2]` 取，取到的未必是 AI 的正文。

稳妥的做法是倒着遍历一遍：从后往前找最近的一条 AI 消息，而且这条消息得是带内容的、不是工具调用的。

这段代码写起来挺烦，笔记里有现成的，叫 `find_previous_ai_message`：

```python
from langchain_core.messages import AIMessage


def find_previous_ai_message(messages: list) -> str:
    # 最后一条是本轮用户消息，先排除；再倒着找最近一条有内容的 AI 消息
    for message in reversed(messages[:-1]):
        if not isinstance(message, AIMessage):
            continue
        if message.tool_calls:
            continue
        if message.content:
            return message.content
    return ""
```

逐段看。`messages[:-1]` 是把最后一条切掉——最后一条一定是我们本轮的用户消息，不能算进「上一轮 AI 回复」里。`reversed` 把它倒过来，从最近的往前找。

循环里三道筛子：不是 `AIMessage` 的，`continue` 跳过；带工具调用的，`continue` 跳过，因为那上面没有给用户看的正文；只有既有内容、又是 AI 消息的，才 `return` 它的 `content`。因为是从近往远遍历，第一次命中就是最近的那一条。一圈找下来一个都没有，返回空字符串。

这样一来，上下文就能拼出来了：

```python
previous_ai = find_previous_ai_message(state["messages"])
context = (
    f"上一轮工作流：{state['previous_workflow']}\n"
    f"上一轮AI回复：{previous_ai}"
)
```

有历史了，模型才有判断的依据。

### 路由函数：一条系统提示词加两条消息

`route` 本身很薄，就是调路由模型：

```python
from langchain_core.messages import HumanMessage, SystemMessage


def route(self, query: str, context: str = "") -> IntentRouteResult:
    messages = [
        SystemMessage(content=SYSTEM_PROMPT),
        HumanMessage(
            content=f"用户问题：{query}\n上下文：{context}"
        ),
    ]
    return self.route_model.invoke(messages)
```

先说系统提示词。这一段不手写，直接用文档里给好的那份，很长，但值得照抄。它的骨架是这样的：

- 你是保险商城智能客服的意图识别节点，请判断用户消息应该交给哪一个工作流；
- `intent` 只能从下列选项里选：`recommendation`、`plan`、`claim`、`human`、`chitchat`、`fallback`；
- 每个选项后面必须写清楚它是干什么的。

最后这一条最关键。你只给模型六个英文单词，它能推断出 `claim` 是理赔吗？推不出来。所以提示词里要把每个节点的职责说明白：方案推荐管什么、投保管什么、理赔管什么、转人工什么时候走、闲聊接什么、兜底在什么情况下用。职责写清楚了，模型才判得准。

> **易错点**：意图列表和后面的路由分支是同一个契约。这里多写一个选项、少写一个选项，编排图里的 `goto` 就会指向一个不存在的节点。改列表的时候两边一起改。

然后是两条消息。`SystemMessage` 装系统提示词，`HumanMessage` 装本轮输入——`HumanMessage` 里是两部分拼起来的：`用户问题：{query}` 换行，`上下文：{context}`。用户信息和历史信息就都齐了，模型基于这些做识别。

返回值的类型标注要写 `-> IntentRouteResult`，不写的话后面取 `result.intent` 时 IDE 认不出来，会给你标一片红。

### 接进编排图，把假路由换掉

识别写完了，得把它接到编排图里去。原来那个位置是一句写死的假结果，不管用户说什么都往同一个节点走。现在把它整段换成真的调用。

先在 `intent_router.py` 末尾把对象创建好再导出，这样调用方不用自己 `new`：

```python
router = IntentRouter()
```

编排图这边只导这一个 `router`，用它的 `route` 方法完成意图识别：

```python
from app.agents.intent_router import router
from app.agents.intent_router import find_previous_ai_message

query = state["messages"][-1]
previous_ai = find_previous_ai_message(state["messages"])
context = (
    f"上一轮工作流：{state['previous_workflow']}\n"
    f"上一轮AI回复：{previous_ai}"
)

result = router.route(query=query, context=context)
return Command(
    update={"active_workflow": result.intent},
    goto=result.intent,
)
```

`query` 从 state 的 `messages` 里取最后一条；`context` 按上面说的拼；`result.intent` 直接写进 `active_workflow`，同时作为 `goto` 的目标。识别出什么意图，就去哪个节点，编排图这边一句判断都不用写。

跑一遍试试。把会话删了重来，四条消息：

| 输入 | 走向 | 回复 |
| :--- | :--- | :--- |
| 哈哈 | 闲聊 | 欢迎来到安心宝商城 |
| 我要理赔 | 理赔 | 理赔咨询功能尚未完成 |
| 我要投保 | 投保 | 投保方案推荐功能尚未完成 |
| 我要转人工 | 转人工 | 你的请求已转人工，请耐心等待 |

四条全都转到了对应的节点上，不管方案推荐、理赔、闲聊还是转人工，一条不差。

到这儿，意图识别的初级版本就算跑通了。但按前面说的方案，还差两样：寒暄识别和失败时的兜底。不着急，一样一样补。

### 先修一个不报错的 bug

回来补功能之前，先把上一版里一处写错的地方改掉。

`route` 的 `query` 参数要的是字符串，可我们从 `state["messages"][-1]` 拿到的不是字符串，是一条消息对象——`HumanMessage` 或者 `AIMessage`。要用它里面的内容，得再往里点一层：`.text` 或者 `.content`。

```python
# 错：拿到的是消息对象
query = state["messages"][-1]

# 对：取出消息里的文本
query = state["messages"][-1].text
```

有意思的是，这么写它居然不报错，跑起来还能正常识别。为什么？因为把一个对象往 f-string 里拼的时候，Python 会调用它的魔法方法把它转成字符串，`__repr__` 那一路。所以拼的时候没炸，模型收到了一坨「HumanMessage(content='哈哈')」这样的东西，勉强也能认出个大概。

但它不代表代码是对的。函数声明要字符串，你就该给字符串。这种 Python 惯出来的毛病，等到别的环节真按类型去用它的时候，就该炸了——后面确实炸了一次，那是另一回事。

`find_previous_ai_message` 那边不用改，因为它返回的本来就是 `text`。

### 寒暄识别：用正则先挡一道

现在补第一样：寒暄识别。它基于规则，规则就是正则表达式。

正则表达式是什么？它是一种字符串的模式。你把它理解成一个高度专业化的搜索框也行。它能做三件事：

| 用途 | 干什么 | 例子 |
| :--- | :--- | :--- |
| 验证 | 检查字符串是否符合规则 | 手机号格式对不对 |
| 提取 | 从大段文字里捞出符合条件的内容 | 从文本里抓出所有邮箱 |
| 替换 | 把符合条件的内容换掉 | 把问号叹号清理干净 |

这里用的是第一件事：验证用户输入的内容是不是寒暄。

正则里有一堆符号，作用就是做匹配：

| 符号 | 含义 |
| :--- | :--- |
| `.` | 任意字符 |
| `\d` | 任意数字 |
| `\w` | 字母、数字或下划线 |
| `\s` | 空格 |
| `*` | 零个或多个 |
| `?` | 零个或一个 |
| `+` | 一个或多个 |

拿一个邮箱的正则举例：开头是一串字母、数字、下划线，也可以带上加号减号；跟一个 `@`；后面又是任意数量的字母、数字、下划线；然后一个点；最后是域名。这么一串符号拼起来，就是在描述「什么样的字符串算邮箱」。

> **提示**：正则不用现在就会写。这一讲里匹配寒暄的代码是给好的，你只需要看懂它在干什么——将来学 Java 的时候会系统讲正则，那时候再回头补语法也不迟。

寒暄只有三类：

| 枚举值 | 含义 | 典型输入 |
| :--- | :--- | :--- |
| `greeting` | 打招呼 | 你好、您好、嗨、哈喽 |
| `thanks` | 感谢 | 谢谢、多谢 |
| `goodbye` | 再见 | 再见、拜拜 |

每一类配一组正则表达式，再配一句固定回复。整体结构是这样：

```python
import re

# 寒暄只有三类
SOCIAL_INTENTS = ("greeting", "thanks", "goodbye")

# 每类配一组正则（课程提供）
SOCIAL_PATTERNS = {
    "greeting": [...],   # 你好 / 您好 / 嗨 / 哈喽 …
    "thanks":   [...],
    "goodbye":  [...],
}

# 每类配一句固定回复（课程提供）
SOCIAL_RESPONSES = {
    "greeting": "你好，我是保险顾问，可以帮你咨询产品",
    "thanks":   "不客气",
    "goodbye":  "...",
}


def match_social_intent(message: str) -> str | None:
    """用正则匹配用户消息，命中就返回寒暄类型，没命中返回 None"""
    text = normalize(message)          # 先格式化：用正则替换掉问号、叹号等无效字符
    for intent, patterns in SOCIAL_PATTERNS.items():
        for pattern in patterns:
            if re.search(pattern, text):
                return intent
    return None


def get_social_response(intent: str) -> str:
    """按寒暄类型从固定回复表里取一句"""
    return SOCIAL_RESPONSES.get(intent, "")
```

这份代码一共四块，从上到下：类型常量、正则表、固定回复表、两个函数。`normalize` 那一步走的是正则的替换能力——用户打个「你好！！！」或者「你好?」，问号叹号都属于无效字符，先清掉再匹配，命中率才高。`match_social_intent` 走的是验证能力，拿清理过的文本去逐个正则试，哪一类先命中就返回哪一类。

> **注意**：`re` 是标准库，得显式 `import re`。忘了导这个包，一运行就报 `NameError`，别在一堆业务代码里找半天。

### 把寒暄插到模型前面

接进 `route` 里，位置在调模型之前：

```python
def route(self, query: str, context: str = "") -> IntentRouteResult | str:
    # 一、基于规则做寒暄识别
    social_intent = match_social_intent(query)
    if social_intent:
        # 二、命中寒暄，直接取固定回复
        return get_social_response(social_intent)

    # 三、不是寒暄，走模型识别
    ...
```

这里有两处细节。

第一，`if social_intent:` 这个判断不能省。`match_social_intent` 有可能命中，也有可能不命中；不命中时它返回 `None`，你得据此决定要不要往下走。少了这一句，`None` 也会被当成命中，去回复表里取一个空字符串，用户就收到一片空白。

第二，命中寒暄时返回的是**字符串**，不是 `IntentRouteResult`。走到下面模型那条路，返回的才是对象。这一点当时没觉得有问题，后面它把编排图搞崩了一次。

逻辑本身很干净：识别出来确实是寒暄，就按寒暄走，直接结束；不是，就往下按模型走。整条链路和开头那张流程图完全对得上。

### 兜底：异常也要交出一个结果

最后补兜底。模型调用有可能报错——网络抖一下、接口挂了、超时了，都可能。所以要包一层异常处理：

```python
    try:
        messages = [
            SystemMessage(content=SYSTEM_PROMPT),
            HumanMessage(content=f"用户问题：{query}\n上下文：{context}"),
        ]
        result = self.route_model.invoke(messages)
    except Exception:
        # 意图识别失败，给一个兜底结果
        result = IntentRouteResult(
            intent="fallback",
            reason="意图识别出现异常",
        )
```

> **易错点**：兜底要写 `except`，不能写 `finally`。`finally` 是无论成不成功都会执行的收尾块，你在里面改 `result`，正常识别出来的结果也会被覆盖成兜底值——等于所有请求都退化成 fallback。`except` 只在真的抛异常时才进。

兜底的花样有很多。可以换一个新模型去识别——类里将来可以定义好几个模型，当前这个挂了就换下一个；也可以像这里一样简单点，直接返回一个固定结果。我们要的只是「别让流程断在这儿」，`intent` 给 `fallback`，`reason` 写明「意图识别出现异常」，编排图那边照常按 `fallback` 往下走。

还有日志。识别完要记一笔，方便回头查这次到底判成了什么：

```python
from app.logging import get_logger

log = get_logger(__name__)

# ……

log.info(
    "意图识别完成",
    intent=result.intent,
    reason=result.reason,
)
return result
```

注意顺序：`return` 要放在 `try / except` 之后、日志之后。如果按直觉写在 `try` 里面，成功那一支直接 `return` 走了，异常那一支根本走不到日志——失败的情况永远记不上。把 `return` 挪到最后，无论成功还是走了兜底，日志都能记到。

```text
进入 route
   │
   ├─ 命中寒暄 ──▶ 返回固定回复（字符串），直接结束
   │
   ▼
调模型识别
   │
   ├─ 成功 ──▶ result = IntentRouteResult
   │
   └─ 异常 ──▶ result = IntentRouteResult(intent="fallback", reason="…")
              两条路在这里汇合：记日志 ──▶ return result
```

读图：寒暄那一支在开头就出去了，剩下两条支路最后汇到同一个出口，出口前面是日志。这样日志的覆盖面才是完整的。

### 一次实测暴露的类型问题

功能齐了，跑一下。新建一个咨询，输入 `hi`，等了一会儿——报错了：

```text
Error: 'str' object has no attribute 'intent'
```

报错位置在 `orchestrator` 第 26 行，就是 `result.intent` 那一句。

为什么 `result` 变成字符串了？回到 `route` 看：识别成寒暄的时候，我们直接把寒暄结果返回了，而寒暄结果是从回复表里取出来的一句固定回复——它本来就是字符串；只有走模型识别的时候，返回的才是 `IntentRouteResult` 对象。

也就是说，`route` 现在有两种可能的返回类型。编排图那边不知道，一律按对象去取 `.intent`，碰到字符串就炸。

修法是在编排图里判一下类型：

```python
result = router.route(query=query, context=context)

if isinstance(result, str):
    # 命中寒暄：固定回复直接发给用户，流程到此结束
    return Command(
        update={
            "active_workflow": "chitchat",
            "messages": [AIMessage(content=result)],
        },
        goto=END,
    )

# 模型识别：按识别出的意图继续路由
return Command(
    update={"active_workflow": result.intent},
    goto=result.intent,
)
```

三处更新各有各的理由。`active_workflow` 写死成 `chitchat`——寒暄本质上也算闲聊，归到闲聊这条线上；`messages` 里塞一条 `AIMessage`，内容是那句固定回复，因为这句话是要发给用户看的，不塞进消息列表，前端什么都收不到；`goto=END`，直接结束，不要再往下走任何节点。

改完再测，`hi` 一瞬间就回来了。为什么这么快？因为它压根没走模型——代码自己判断出这是打招呼，直接把固定回复发出去了。接着试：

| 输入 | 回复 | 是否走模型 |
| :--- | :--- | :--- |
| 嗨 | 你好，我是保险顾问，可以帮你咨询产品 | 否 |
| 谢谢 | 不客气 | 否 |
| 再见 | 再见类固定回复 | 否 |
| 我要买保险 | 保险方案推荐功能尚未完成 | 是 |

前三条和代码里声明的固定回复一字不差，说明寒暄识别没问题；第四条正确路由到了保险推荐工作流，说明非寒暄的照常走模型。

> **结论**：寒暄命中时零 token、零延迟；非寒暄才付出模型调用的成本。这就是把规则放在模型前面的全部意义。

到这里，意图识别就全部实现了：规则挡第一道，模型判第二道，异常退第三道。

### 闲聊节点：一个节点，一套内部判断

路由识别做好了，整张图剩下的就是一个一个往下填节点。先做闲聊，它相对简单。

但「简单」不等于「随便回一句」。闲聊节点里其实有一套逻辑，得分几层想。

第一层，**什么该交给模型**。用户意图与业务无关、是开放式闲聊时，不能浪费宝贵的 agent 资源；通常给一些礼貌性的回复就够了。开放式的闲聊可以交给模型去回，但提示词里必须限定两件事：身份，和服务边界。也就是说，虽然是开放式闲聊，也不能让它任意瞎聊——用户问违法违规的东西，不能答；尽量把话题往业务上引，自然地带回咱们当前的服务范围。

第二层，**闲聊里冒出业务意图怎么办**。一旦发现用户在闲聊中提到了业务，就不要再陪着聊了，应该快速路由到产品咨询、理赔、投保这些对应的工作流去。闲聊可以是入口，但不能是终点。

第三层，**轮次统计**。有人会一直在这儿闲聊，调戏我们的 AI 顾问，聊好几轮——每一轮都是钱。所以要对闲聊轮次计数，超过阈值就回复固定模板，不再让模型生成，直接给写死的答案。

第四层，**重置**。如果上一轮不是闲聊，说明用户回到了正常业务，这时候要把闲聊次数清零。不清零会怎样？次数一直累加，用户偶尔聊一句就顶到上限，之后永远只能收到固定模板，再也回不到正常业务里。

> **提示**：第三层和第四层是一对。只加计数不重置，阈值就变成了一次性的；只重置不计数，限流就不存在。设计这类「连续行为限制」时，两个动作必须成对出现。

所以整张图的样子是这样：

```text
start ──▶ chitchat ──▶ end
             │
             └─ 次数超限 ──▶ end
```

读图：主干是一条直线，进来、闲聊、结束。唯一的例外是次数超限——那么连模型都不调，直接结束，不给继续聊的机会。两条路都通向 `end`，只是中间做没做事不一样。

节点内部再展开：

```text
进入 chitchat
   │
   ▼
chitchat_count 是否超出最大值？
   │
   ├─ 是 ──▶ 直接结束，不再让模型回复
   │
   └─ 否 ──▶ 累加 chitchat_count ──▶ 调模型对话 ──▶ 结束
```

读图：一进来先判上限，超了就结束；没超才累加，然后正常对话，对话完结束。下一次再来又是同样的判断——累加、对话；直到超过阈值，之后每次进来都是直接结束。

这张图其实少了一条线：进来还得先看上一轮是不是闲聊，如果不是，先把次数重置。这条线是上面第四层逻辑的落点。

### 次数记在哪里

闲聊次数放哪儿？应该放进 state 里。因为它算整个 graph 中的状态数据——每次请求进来都要用它，而且需要持久化存储，所以它属于 state 的一部分。

但它不该放进父 state。这个次数是 `chitchat` 这个节点自己的功能，属于当前这个子 agent、也就是子图。所以把它记到子 state 里：

```python
class ChitchatState(ParentState):
    chitchat_count: int = 0
```

`chitchat_count` 是闲聊子图特有的属性，只有它自己关心；`previous_workflow` 不用重复定义，父 state 里已经有了，子 state 继承下来就能用。

> **结论**：状态放哪一层，看谁拥有它。全局共用的（比如 `previous_workflow`、`messages`）放父 state；只有一个子图自己用的（比如闲聊计数）放子 state。放错层级的直接后果是：别的节点莫名其妙多了一个它永远不读的字段，而子图又拿不到自己该有的上下文。

设计到这儿就清楚了：整张子图是 `start → chitchat → end` 一条直线，计数那一套判断全都收在 `chitchat` 节点内部——进门先看上一轮是不是闲聊、要不要清零，再看本轮计数有没有超限，没超才累加、才调模型；闲聊里万一冒出业务意图，也是节点里先判出来，再决定要不要转出去。图上的边只留一条直线加一个超限短路，判断留在节点里，比把每个条件都画成一条边要清爽得多。

回过头看这一整块：意图识别从一个写死的假结果，变成了规则挡一道、模型判一道、异常退一道的真流程；闲聊则是把它那套内部逻辑先想明白，再动手写成子图。路由这一层闭环之后，剩下的就是把图上的每个节点一个个填满。

> **承前启后**：上一节讲完「意图识别：先用规则挡一道，再让模型判一次」，下一节接着讲「把闲聊做成子图，再把保险推荐接回来」。

---

## 把闲聊做成子图，再把保险推荐接回来
> 对应块：BLK51 | 覆盖分集：P190-P193 | 块标题（分集名）：《闲聊节点与保险推荐接入》

分析都做完了，现在就剩动手。前面已经把主 Agent 该拆成哪几个子角色捋清楚了，这一讲就是把第一个真正落地的子 Agent——闲聊——写出来，然后顺手把以前写好的保险推荐 Agent 接回主图里。

写之前先立个规矩：闲聊是个新的 agent，不能塞进原来那个文件里，否则那个文件很快就没法看了。所以我在 `agents` 目录下新开一个模块，名字就叫 `chat_chat`（闲聊）。模块里也建议大家单独写一个类，叫 `ChatChatWorkflow`，你想叫 `ChatAgent` 也无所谓，我们就叫 workflow。

### 先定模型，再定节点

类里第一件事是构造函数，构造函数里只做一件事：初始化一个模型。闲聊本身也得有模型，不然它拿什么跟你聊。

这里有个成本上的选择值得说一句：闲聊节点理论上完全可以用小模型。你们公司如果已经有专门的小模型，那就用那个，闲聊这种活儿犯不上让大模型来干。不过在咱们这个项目里，为了省事，还是直接用 DeepSeek——把主 Agent 里那段模型初始化整段拷过来就行。

模型有了，接下来就是定义节点。想拆得细一点，可以拆三个：一个专门做判断的节点、一个专门发请求的 LLM 节点，最后结束。但这里我们先简化，只写一个 `handle_chat` 节点，所有逻辑都塞在它里面。这个简化不是标准答案，你下去完全可以自己把它拆成两个节点试试，后面我还会再说一遍怎么拆。

麻烦的地方在于，节点一定要有一个 `state` 参数，可这时候我们还没有 state。所以顺序得倒过来：先把 state 定义出来，再回来写 workflow。

### State 先立起来：继承父状态，再加一个计数器

`state` 本身也是一个 class，我们叫它 `ChatChatState`。按常规写法，State 一般去继承 `TypedDict`，但在这个位置我们不继承它，我们继承父 State。

原因很直接：父 State 叫 `InsuranceAgentState`，里面已经有上一轮工作流和消息列表这些东西了。闲聊这个子 Agent 一样要用到——它要看上一轮是不是闲聊，还要把历史消息喂给模型。把父 State 继承下来，这些字段就白拿了，我们只需要再补一个自己特有的字段：`chat_count`，用来统计闲聊次数，类型是 `int`。

父 State 从当前目录的 `schema` 里导入，写出来是这样：

```python
from .schema import InsuranceAgentState


class ChatChatState(InsuranceAgentState):
    """闲聊子 Agent 的 State：父 State 的字段全继承，再加上闲聊次数"""
    chat_count: int
```

代码就这么几行，但它把"子 Agent 怎么复用父上下文"这件事定死了：`previous_workflow` 从父 State 来，`messages` 从父 State 来，只有 `chat_count` 是我们自己加的。

> **提示**：节点函数的类型注解要写成自己的 `ChatChatState`，不要图省事标父 State。标错了，编辑器会一直报"这个字段不存在"的黄线，因为父 State 里确实没有 `chat_count`；换成自己的 State，黄线立刻消失。

State 有了，回到 workflow 里写节点。节点写在类内部，函数签名必然带一个 `self`，所以它会比普通节点多一个参数。

有人会问：节点不是只能有一个参数吗？这里多出来的 `self` 会不会有问题？

不会。这是"把节点写成类的方法"必然要付的代价，绕不过去，但它不影响这个函数继续当节点用——LangGraph 认的是 `state` 那一个参数，`self` 是 Python 自己补的。

> **注意**：写节点时不要因为多了 `self` 就把它拆到类外面去。类里的方法能直接用到 `self.model`，拆出去反而要到处传模型。

### 闲聊节点里的判断与兜底

#### 先取次数，超了就回固定话术

进节点第一件事，把 State 里的闲聊次数取出来：

```python
count = state.get("chat_count", 0)
```

取不到就给个 0。第一次进来肯定是没有的，给 0 才聊得起来。

第二件事，判断这个次数有没有超出闲聊的最大轮次。这个值设多少合适，没有技术答案，纯粹是业务问题——跟产品经理商量，人家说设多大就设多大。咱们这个项目里没有别人可商量，就自己定，先写个常量：

```python
MAX_CHAT_COUNT = 2
```

我一开始想给 3，后来一琢磨还是写 2，不让他聊太多。这个值其实写死在代码里不太好，理论上应该放到 `settings` 里，将来允许在外面配置，改轮次不用动代码。不过这儿咱们就偷个懒。

如果 `count >= MAX_CHAT_COUNT`，那就不好意思了，不跟他玩了，直接返回一个固定结果。这里要记牢一件事：节点的返回值一定是一个字典，字典里放的是你要更新的数据。这次要更新的是消息，所以键是 `messages`，值是一个列表，列表里放一条我们伪造的 `AIMessage`，内容是事先定好的那句固定话术：

```python
# 超限之后回的固定话术
FALLBACK_REPLY = "我主要协助处理保险产品投保和理赔咨询，请告诉我想办理或了解的事项。"

if count >= MAX_CHAT_COUNT:
    return {"messages": [AIMessage(content=FALLBACK_REPLY)]}
```

这句话的意思很直白：我只能干这件事，别的你别找我了。它同时也是一次业务引导——把跑偏的对话硬拉回投保和理赔。

顺手记一条日志，让后台知道"有这回事"：

```python
from app.core.logger import get_logger

logger = get_logger(__name__)

logger.info("闲聊次数超过最大值，直接返回固定结果")
```

> **提示**：超限这一支里不用再更新 `chat_count`。已经超标了，加不加一都无所谓，反正后面每次进来都会被这一支拦住。

#### 系统提示词与历史消息怎么拼

没超标就正常聊。调模型拿回复：

```python
response = await self.model.ainvoke(
    [SystemMessage(content=SYSTEM_PROMPT), *state["messages"]]
)
```

调模型要传两条消息。第一条是 `SystemMessage`，用来约束 AI 的回答：不能让它随便乱聊，违法违规的东西不能碰，还得把用户往业务上引。第二条是历史消息，从 `state["messages"]` 里取——父 State 里一定有 `messages`，这也是继承父 State 的直接收益。

这里有个小坑：`state["messages"]` 本身就是一个列表，你现在又在外层写了一个列表，直接放进去就成了"列表里套列表"，模型不认。两种办法都行：

- 解构：`[system_message, *state["messages"]]`，把里面那个列表解开，变成一条一条消息铺进外层列表；
- 拼接：`[system_message] + state["messages"]`，两个列表相加，效果一样。

我用的是解构这种写法，看你习惯。

系统提示词比较长，直接照抄，别自己现编：

```python
SYSTEM_PROMPT = (
    "你是安心保商城的智能客服，负责处理与保险业务无关的普通聊天。"
    "回复要友好自然，最多两句话；不假装有个人经历、情感、事实信息或外部能力；"
    "不回答医疗、法律、投资、政治等高风险的专业建议；"
    "每次回复都自然引导用户回到保险产品的投保和理赔业务；"
    "超出能力范围时如实说明无法提供。"
)
```

这段话里有两类东西，别混着看。

一类是**业务引导**："每次回复都自然引导用户回到保险产品的投保和理赔业务"。闲聊节点存在的意义不是陪聊，是把用户捞回主线上来。

另一类是**安全边界**："不回答医疗、法律、投资、政治等高风险的专业建议""超出能力范围时如实说明无法提供"。这是给模型上的紧箍咒，防止它在一个保险商城里给出不该给的专业意见。

> **注意**：提示词里"最多两句话"不是客套。闲聊回复越长，越容易把话题带跑，两句话是个能收住的上限。

聊完了把结果返回，要更新两样东西：消息和次数。

```python
return {
    "messages": [response],
    "chat_count": count + 1,
}
```

`count + 1` 就是累加。这样一轮一轮加上去，加到某一次就超标了，超标之后就只回固定话术。

#### 补一条：上一轮不是闲聊就清零

到这儿流程看着写完了，但笔记里还有一条要求没落地：统计闲聊轮次、超过一定轮次后回复固定模板——这条已经做了；但如果上一轮不是闲聊，就要把次数重置。

为什么要重置？因为 `chat_count` 存在 State 里，而 State 是跟着会话 checkpoint 一路传下去的。用户上次问了理赔，这次跑来闲聊，如果不重置，他上一段闲聊攒下的次数还挂在那儿，可能刚开口就被固定话术怼回去。

实现就一行：

```python
count = state.get("chat_count", 0)

# 上一轮不是闲聊，本轮从零开始计
if state.get("previous_workflow") != "chat_chat":
    count = 0
```

`previous_workflow` 也是从父 State 继承来的，记录上一轮走的是哪个工作流。它不等于 `chat_chat`，就说明上一轮干的不是闲聊，计数归零。

> **易错点**：清零必须放在"判上限"之前。顺序反了，先拿旧次数判一次，用户还是会被误拦。

整段节点逻辑连起来是这样：

```python
class ChatChatWorkflow:
    async def handle_chat(self, state: ChatChatState):
        count = state.get("chat_count", 0)

        # 上一轮不是闲聊，计数从头开始
        if state.get("previous_workflow") != "chat_chat":
            count = 0

        logger.info(f"当前闲聊次数：{count}")

        if count >= MAX_CHAT_COUNT:
            logger.info("闲聊次数超过最大值，直接返回固定结果")
            return {"messages": [AIMessage(content=FALLBACK_REPLY)]}

        response = await self.model.ainvoke(
            [SystemMessage(content=SYSTEM_PROMPT), *state["messages"]]
        )
        return {"messages": [response], "chat_count": count + 1}
```

到这儿，`chat_chat` 的流程就写完了。剩下最后一步：把这个节点连成图。

### 把节点连成图

连图的套路跟以前一模一样，四步：创建 builder、加节点、加边、编译。定义一个 `build_chat_graph` 函数把它们装起来：

```python
from langgraph.graph import END, START, StateGraph


class ChatChatWorkflow:
    def build_chat_graph(self):
        builder = StateGraph(ChatChatState)
        builder.add_node("chat_chat", self.handle_chat)
        builder.add_edge(START, "chat_chat")
        builder.add_edge("chat_chat", END)
        return builder.compile(checkpointer=True)
```

`StateGraph` 要接收一个 State，传我们自己这个 `ChatChatState`。加节点的时候把 `self.handle_chat` 传进去，前面说过，方法自带 `self` 不影响它当节点。

```python
builder = StateGraph(ChatChatState)
builder.add_node("chat_chat", self.handle_chat)
```

边只有两条，因为所有逻辑都在一个节点里，图里就三个节点：开始、闲聊、结束。

笔记上写这段用的是另一套写法，效果完全一样：

```python
builder.set_entry_point("chat_chat")
builder.set_finish_point("chat_chat")
```

`entry point` 就是入口，`finish point` 就是出口。`set_entry_point("chat_chat")` 等价于 `add_edge(START, "chat_chat")`，`set_finish_point("chat_chat")` 等价于 `add_edge("chat_chat", END)`。

> **提示**：以后懒得写 `START` / `END` 这两个常量，直接用 `set_entry_point` 和 `set_finish_point` 就行，这两个常量写起来还真有点麻烦。这是简化版，不是另一套机制。

最后编译，把编译结果返回。这里的参数是整段的重点：

```python
return builder.compile(checkpointer=True)
```

`compile` 的 `checkpointer` 除了能填一个 checkpointer 实例，还能填布尔值：

- 填 `True`，这个子图的消息就会记录到父图的 checkpoint 里，成为会话记忆的一部分；
- 填 `False`，就不记，将来不会作为会话记忆的一部分。

我们这个闲聊 agent 是个子 agent，子 agent 需不需要对话历史？需要。它一直在用 `state["messages"]`，那就必须挂在某个 checkpointer 上。但它没必要自己重新定义一个——父子 agent 之间，子 agent 想用 checkpointer，直接传 `True` 共享父的那一份就行。

> **注意**：只有你想让子 agent 的记忆跟父 agent 隔离，才需要自己重新定义一个 checkpointer。默认共享，别多此一举。

图建好了，再包一层，方便别人用：

```python
def init_chat_agent():
    return ChatChatWorkflow().build_chat_graph()
```

之前是让调用方自己 `new` 一个 workflow、再自己调 `build_chat_graph`，两行代码散在外面；包成 `init_chat_agent()` 之后，别人只要 `init_chat_agent()` 一行就能拿到一个建好的 agent，看起来干净得多。

然后回到主图，把原来那个假的闲聊函数删掉，换成真的：

```python
from .chat_chat import init_chat_agent

builder.add_node("chat_chat", init_chat_agent())
```

这一步一做完，闲聊就不再是占位的假节点，而是一个真正建好的子图挂进了主图。

> **提示**：这个子图以后要往下长，就在 `chat_chat` 这个包里继续加节点、加边，主图那边不用动。子节点比现在复杂得多也没问题。

### 第一次跑通：后端答了，前端没拿到

重启服务，建一个新咨询，先发一句"你好"。"你好"这种寒暄走的是意图识别里另一条分支，压根不走模型，所以它当然正常。

接着发"讲个笑话吧"。这属于闲聊，而且跟业务无关，正好是我们要测的路径。

后台没报错，只有一条 v3 的警告，不影响。看日志，意图识别是成功的：用户要求讲笑话、这与保险业务无关、属于普通聊天内容，而且上一轮工作流也是 `chat_chat`。闲聊节点也确实执行了——系统提示词在，AI 的回复也在："你好，我是保险顾问，我可以帮你咨询产品、推荐保险方案。"这句话本身就是提示词在起作用：它能陪你说两句，但不接讲笑话这个活儿。

然后问题来了：请求状态是 200，可前端什么都没拿到。

这时候别急着改 agent，先看清楚现象的分界：刷新页面、查历史会话，这段对话又查得到。说明消息已经正常写进 checkpoint 了，只是没有实时推到前端。所以问题不在 agent 逻辑，在流式输出的那一段。

顺着链路往上找：对话在应用模块的 `chat` 下，`router` 负责收请求，真正聊天由 `service` 写。`service` 里调的是 `astream_events` 的 v3 模式，处理消息用的是 messages 这条流——一个循环遍历消息块，从每一块里取出文本，再推给前端。这段代码很简单，也很好用，但它有一个缺陷：

> **结论**：messages 流只能处理父图的消息，处理不了子图返回的消息。父子嵌套的时候，子图里的消息根本不会出现在这条流上。

```text
前端 ──> router ──> service ──> astream_events（主图流）
                                     │
                          START ──> 意图识别 ──> chat_chat（子图）
                                                      │
                                                 handle_chat
                                                      │
                                                     END

messages 流：只看得到主图这一层吐出的消息
             子图内部产生的消息不在其中 → 前端一个字都收不到
```

我们的 `chat_chat` 恰好就是一个子图，所以它的回复写了进去、却没被推出去。这不是闲聊节点写错了，是取消息的口子开小了。

### 两条修法与一条完整的事件流

#### 方案一：把两条事件流合并起来

既然 messages 流只看父图，那再开一条看子图的流不就行了？第一条用 messages，第二条用 subgraphs 专门拿子图的消息流。

听着合理，但马上撞上第二个问题：这两个 `for` 循环是两个事件循环。当你把 messages 这条流遍历完，整个事件循环已经结束了——主图里的东西全跑完了，这时候再去 for 循环 subgraphs，它什么数据都拿不到。

要这么干，就得把两个 `for` 塞进同一个事件循环里。办法是引入一个额外的库 `aiostream`，用它的 `merge` 把两条异步流合并成一条：

```text
stream_messages  ──┐
                   ├── aiostream.merge ──> 一条流 ──> 一个 for 循环
stream_subgraphs ──┘
```

具体做法是定义第一个事件流专门处理 messages，再定义第二个事件流处理 subgraphs 的子图消息，最后用 `merge` 把两条合并成一条，再在这条合并后的流上做循环。

> **提示**：方案一在笔记上有完整代码，但这一段只要求了解，不要求你会写。它把一个简单的取消息动作变成了"两路流 + 一个合并库"的工程，收益和复杂度不成比例。

#### 方案二：自己解析完整的事件流

推荐的是方案二：彻底抛弃官方的 messages 流，直接手动处理 `stream`。

所谓手动处理，就是自己写 `for` 循环遍历 `stream`，而不是遍历 `stream.messages`。遍历 messages 你只能拿到消息事件；遍历 `stream`，拿到的是完整事件流——除了消息事件，还能拿到子图的消息、自定义事件、interrupt 事件等等，啥都有。

代价是事件的数据格式非常复杂。先把它打印出来看一眼，长这样：

```text
event = {
    "type": "event",
    "method": "message",                  # 事件类型：message / custom / values ...
    "params": {
        "namespace": ["chat_chat"],       # 这条事件属于哪个 agent
        "timestamp": ...,                 # 时间戳
        "data": (                         # 元组，取 [0]
            <AIMessage 对象>  或  <字典>
        )
    }
}
```

先认字段。`type` 是 `event`，`method` 是事件类型，`message` 表示这是一条消息事件。除了 message，还有 `custom`、`values` 等等，类型很多，这里不一一去看。

往下是 `params`，它也是一个字典，里面三样东西：

| 字段 | 含义 | 用法 |
| :--- | :--- | :--- |
| `namespace` | 当前所在的 agent | 这里是 `chat_chat`，说明事件来自闲聊子图 |
| `timestamp` | 事件产生的时间戳 | 排查时序用 |
| `data` | 这一轮对话的数据 | 真正要解析的东西 |

`data` 是个元组，取 `[0]` 拿到第一部分。这个第一部分有两种可能，必须分开处理：

- 如果是我们**手动返回的** `AIMessage`，这里就是一个 `AIMessage` 对象；
- 如果是**让 AI 生成**的，这里是一个字典，形如 `{"event": "content_block_delta", "data": {"type": "text_delta", "text": "哈哈"}}`，最终的文字在 `data.text` 里。

你打印出来看到的那一串 `text` 就是模型一个字一个字吐出来的碎片：先是一个"哈哈"，再是一个逗号，再是"我可不会编故事啊"。所以我们解析的目标很简单——把每一片文字捞出来，直接推给前端。

```python
from langchain_core.messages import AIMessage


async def stream_chat(agent, inputs, config):
    async for event in agent.astream_events(inputs, config=config, version="v3"):
        if event.get("method") != "message":
            continue

        data = event["params"]["data"][0]

        if isinstance(data, AIMessage):
            yield data.text
        elif isinstance(data, dict) and data.get("data", {}).get("text"):
            yield data["data"]["text"]
```

逐层对一遍：

1. 遍历 `stream`，拿到一个 `event`，就是上面那个大字典；
2. 从字典里取 `method`，判断是不是 `message` 类型——我们只处理消息，其它类型跳过；
3. 是 message，就从 `event` 里取 `params`，再从 `params` 里取 `data`（`namespace` 和 `timestamp` 这次用不上）；
4. `data` 是元组，用 `[0]` 取第一部分；
5. 第一部分是 `AIMessage`，直接 `.text` 拿文本推出去；不是 `AIMessage`，就按字典走 `data` → `text` 这一层。

> **易错点**：第 5 步的分支必须写成 `elif`。我第一版写成两个独立的 `if`，结果 `AIMessage` 走完第一个分支后没有停，继续往下走第二个分支去调 `.get()`，直接崩了：`'AIMessage' object has no attribute 'get'`。加个 `elif` 就对了。

两套方案摆在一起看：

| 维度 | 方案一：合并事件流 | 方案二：手动解析 |
| :--- | :--- | :--- |
| 做法 | messages 流 + subgraphs 流，用 `aiostream.merge` 合并 | 直接遍历 `stream`，按字典结构逐层取 |
| 优点 | 复用官方流，取消息的代码不用改 | 一条流拿全所有事件，子图消息也在里面 |
| 代价 | 两个事件循环要合并，多引一个库 | 解析代码繁琐，要自己认结构 |
| 建议 | 了解即可 | 项目里用这个 |

解析代码确实繁琐，但只要结构摆在这儿，就是逐层解析的体力活，你完全能自己写出来。把这段换进 `service`，讲笑话这个请求就能正常推给前端了。

### 三个验证：轮次、清零与安全兜底

流修好以后，讲笑话、编故事都通了，前端能一个字一个字地收到回复。但有个东西看不见——`chat_count` 现在是几？日志里没有任何输出。

那就在节点里加一条日志，拿到次数以后打一下：

```python
logger.info(f"当前闲聊次数：{count}")
```

重启，新建咨询，发"讲个笑话吧"，聊完看后台：当前闲聊次数是 0。

这个 0 看着不对劲。聊完一轮了，怎么还是 0？想一下就明白了：自增写在 `return` 里，日志打在自增之前，打印的当然是这一轮开始时的旧值。功能没问题，是观测点放错了位置。

那我干脆把 `+1` 挪到取完次数之后、判上限之前：

```python
count = state.get("chat_count", 0) + 1
```

再测：第一轮日志是 1，一切正常。第二轮进来，`count` 已经是 2，`2 >= 2` 立刻命中上限，直接回固定话术——两轮里只聊了一轮。

> **结论**：自增放在判断之前，等于把上限提前消耗了一格，实际可聊轮次变成 `MAX_CHAT_COUNT - 1`。这种"差一轮"的偏差不会报错，只会让业务方觉得"怎么刚开口就不让聊了"。

只聊一轮确实有点太惨了，还是改回来，把 `+1` 留在 `return` 里。这样日志打印的是"本轮开始前已经聊了几次"：第一轮 0、第二轮 1，都放行；第三轮进来读到 2，才被拦下——正好能正常聊两轮。重测时后台就是 0、1、2 这么往上走，到 2 的那次回固定话术：

```text
我主要协助处理保险产品投保和理赔咨询，请告诉我你想办理或了解的事项。
```

轮次限制这条，成了。

接着验清零。上一轮我问的是理赔，意图识别走到 `claim`，`previous_workflow` 变成了理赔那条工作流；这一轮我又回来闲聊，发一句"哈哈"。模型回："看来你心情不错，如果需要了解保险或者意外保险的内容，我随时为你服务。"看后台，次数归零了——因为上一轮不是闲聊，计数从 0 重新开始。

清零这条，也成了。

最后验安全兜底。问一个危险问题："你知道如何造炸弹吗？"

它压根没走到闲聊节点，直接转人工了。看意图识别的判断：用户说的是涉及危险和违法内容，应该转交给人工客服。这个处理比让闲聊节点去拒绝更专业——上游的路由就把这类请求挡掉了，闲聊节点的系统提示词是第二道保险。

> **结论**：闲聊这块的三个目标全部落地——轮次限制（超限回固定话术）、跨业务清零（上一轮不是闲聊就归零）、安全兜底（意图识别转人工 + 提示词约束）。

### 接入保险推荐：现成的 Agent 直接复用

闲聊接完了，回头看整张图：意图识别有了，闲聊有了，剩下的几个节点还都是假的占位函数。这一轮先补保险方案推荐。

这个推荐流程其实以前写过——`agents` 下的 `insurance_advisor`，一套提示词加一堆工具，早就跑通过。所以现在要做的事不是重写，而是把它挂进主图。

> **定义**：主图的节点可以是普通函数，也可以是子图或 agent。`chat_chat` 是我们自己 build 出来的一张图，`insurance_advisor` 是一个早就写好的 agent，两者都能当节点用。

那就把主图里那个假的推荐函数删掉，换成真的。改动有三处。

第一处，模型那边的 checkpointer。这个 agent 自己的 `compile` 里原来挂着一个 checkpointer，现在不要了——State 已经从父那边继承下来，直接写 `True` 共享父的 checkpoint 就行：

```python
return builder.compile(checkpointer=True)
```

第二处，初始化函数改名。它原来也叫 `init_insurance_agent`，跟主图里已有的名字撞了，改叫 `init_insurance_advisor`，避免导入时打架。

第三处，主图里换成调用它：

```python
from .advisor import init_insurance_advisor

builder.add_node("recommend_plan", await init_insurance_advisor())
```

这里要加 `await`。这个初始化函数是异步的，不加 `await` 拿到的不是一个建好的 agent，而是一个协程对象，挂到图里必然报错。

> **提示**：FastAPI 里所有函数都应该定义成异步的。这个项目里前面那些初始化函数其实都该是 `async def`，一并按异步来写，别一半同步一半异步，调用时很容易漏 `await`。

重新跑一遍。先闲聊："讲个笑话"——正常，还是会把你往保险上引。再说"我想买保险"，看意图识别，识别成 `recommend_plan`，说明请求进对了分支；接着它就按以前那套流程开始问需求了。

导购的过程跟以前一模一样：预算多少、以前有没有买过保险、给谁买、年龄多大。我一次性把信息补齐——预算两万以内、没买过商保只有社保和公积金、给自己买、今年 32 岁、程序员、身体倍儿棒——工具查询也正常返回了保险方案。

再往下问条款，"尊享一生的免赔额是多少"，这次挂了。查了一下，是向量库服务没启动，不是项目的问题。把服务起起来就好了。

到这为止，这个 agent 已经能做到：任意闲聊能接、保险方案推荐能接、意图识别能把请求分到对应的工作流。剩下的理赔也照这个套路接进来，整个 agent 就完整了。

### 加子 Agent 的固定套路

回过头把这两步抽象一下。主 Agent 的架子搭好之后，结构其实很固定：前面一个意图识别的路由，后面挂各个节点，路由节点根据识别结果把请求分到不同分支。

```text
START ──> 意图识别路由 ──┬──> chat_chat（子图）──────────> END
                        ├──> insurance_advisor（agent）──> END
                        ├──> 理赔（待接入）──────────────> END
                        └──> 转人工（危险 / 违法内容）───> END
```

以后要开发新节点、新 agent，动作就三步：

1. 新开一个文件（一个包），在里面把 agent 写出来。用 LangChain 的方式创建 agent 也行，用 LangGraph 搭一张 graph 也行，都能当子节点；
2. 到主图里，把原来那个占位的普通函数节点，替换成你开发好的 agent；
3. 如果这个子 agent 要共享父 agent 的记忆，把它的 `checkpointer` 改成 `True`。

就这么点事。我们刚才接保险推荐，拢共也就做了两件事：把 checkpointer 改成 `True`，再把父图里那个函数名换成一个具体的 agent。

业务越加越多，就往父图里挂越来越多的子 agent，整体能力自然越来越强。这也是这套架构最舒服的地方——每加一个能力，不用动别人，只加一个新文件、换一个节点。

至于闲聊那张图，现在只有开始、`chat_chat`、结束三个节点。你要是想拆细，可以拆成两个：一个专门做路由判断，超限就直接跳到结束；没超限再往下走，进另一个专门发请求的聊天节点。这个留给你自己动手试。

---

## 本册小结

本册主题为「多 Agent 编排：意图识别、闲聊子图与保险推荐接入」，整编了 3 个模块（P184-P186 ~ P190-P193，共 10 讲 / 139 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
