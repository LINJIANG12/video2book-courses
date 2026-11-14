# 黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目·Agent 应用开发：LangChain、工具调用、记忆与流式对话（第 3 册 / 共 6 册）

> **所属课程**：黑马程序员Python+FastAPI+LangGraph多Agent搭建企业级AI应用，手把手用最新LangChain+Milvus搭建真实AI项目  
> **本册内容**：Agent 应用开发：LangChain、工具调用、记忆与流式对话  
> **覆盖范围**：P64-P66 ~ P98-P102（共 39 讲 / 10 章 / 459 分钟音频）  
> **分册说明**：全书按内容分 6 册；本册为第 3 册  
> **整编说明**：正文逐字保留各块模块长文，仅补导读、目录与章间过渡。

---

## 导读与全景目录

1. 让模型长出双手：Agent 与 Tool Calling（P64-P66）
2. 手写一遍 Tool Calling，再交给 LangChain（P67-P69）
3. LangChain 模型：三种初始化与直接调用（P70-P74）
4. 消息、结构化输出与 Agent 的调用方式（P75-P79）
5. 工具描述的三种方式与 MCP 的 stdio 接入（P80-P82）
6. 从 HTTP 接 MCP，到给 Agent 装上记忆（P83-P86）
7. 会话记忆搬进 PostgreSQL（P87-P90）
8. 基础对话接口与 SSE 数据格式（P91-P94）
9. 让回答一个字一个字地长出来：SSE 流式对话与会话历史接口（P95-P97）
10. 会话历史：查得出来、删得干净（P98-P102）

---

## 让模型长出双手：Agent 与 Tool Calling
> 对应块：BLK17 | 覆盖分集：P64-P66 | 块标题（分集名）：《Agent 与 Tool Calling 原理》

我们这个智能客服，将来是要给用户推荐保险产品的。那问题就来了：AI 到底怎么推荐？

真做起来流程会很复杂，这里先把它简化成一条能跑通的链路：用户跟 AI 聊天，我们根据用户的需要去产品库里查一批保险产品出来——这一批东西，我们叫它**候选产品**；然后把候选产品发给 AI，让 AI 根据用户需求从里面筛出一部分，再推荐给用户。

```text
用户 ──说出需求──▶ AI 客服
                      │
                      │ ① 我们按需求去产品库里捞一批
                      ▼
             ┌─────────────────┐
             │   保险产品库    │  数百上千甚至上万条
             └────────┬────────┘
                      │ ② 每个险种各取几条
                      ▼
             候选产品（小批量）
                      │ ③ 连同用户需求一起交给 AI
                      ▼
             ┌─────────────────┐
             │  AI 按需求筛选  │
             └────────┬────────┘
                      │ ④ 挑出合适的
                      ▼
                 推荐给用户
```

读图：真正拍板的是 AI，但 AI 并不是从全库里挑，而是先由我们查出一小批候选产品喂到它面前，它在这个小池子里选。所以这个接口的职责就一句话——查出一部分保险产品，给 AI 用。

### 从"给首页查"到"给 AI 查"

#### 需求：先筛一小批候选产品

这个查询跟前面那个查询不是一回事。前面那个查询是给项目首页展示用的，目的是把所有的保险都查出来，所以它带分页；而现在给用户推荐的时候，你不能把所有保险产品都查出来发给 AI。

真实场景里，一个商城的保险产品可能达到数百、上千甚至上万条，如此之多，一下发给 AI 也不合适。所以这里要做的是：在这么众多的产品里先做一次初步筛选，筛出一小部分，把这部分发给 AI。**查询候选产品，本质就是一次筛选。**

筛的依据首先是分类。保险产品有四大分类：重疾险、医疗险、寿险、意外险。用户跟 AI 咨询的过程中，最后要讨论的是买什么保险——他可能只想买医疗险，也可能只想买寿险，还可能同时想买好几种。假如他只想买医疗险和寿险，那查候选产品时，就应该是医疗险查一部分、寿险查一部分。

#### 三个参数的含义

接口是 `GET /products/candidates`，`candidates` 就是候选的意思。关键在参数，一共三个。

| 参数 | 讲的是什么 | 落到查询里的条件 |
| :--- | :--- | :--- |
| `category` | 产品分类，可以传一个，也可以传多个 | `category = 当前这一轮的分类`，逐轮查 |
| `premium` | 用户心里的保费预算，也就是上限 | `min_premium <= premium`，不传就不加这个条件 |
| `limit_per_category` | 每个险种最多返回几条（接口上写的是 5） | `LIMIT 5`，每一轮各限一次 |

第一个参数 `category`：传一个，就只推荐这一种；传两个，就两种都推荐。

第二个参数 `premium`：用户心里是有预算的——"我买这个保险最多花 2000 块，多了我不掏"。那我们去查的时候，能不能把超过这个价格的产品也查出来？不能。产品表里除了分类，还有一个价格（最低保费），有的便宜，有的很贵，所以正好拿它做筛选：产品的保费不高于这个预算。

第三个参数 `limit_per_category`：每个险种最多返回的数量。数据库里有成千上万条产品，你不可能全查出来，"到底查几个"就由这个数字说了算，接口上写的是 5。

> **易错点**：`limit_per_category` 的"每个险种"是字面意思。查寿险和医疗险两个分类，是寿险给你 5 条、医疗险也给你 5 条，然后汇总到一起返回；不是所有分类加起来一共 5 条。

返回值也很简单：一个 `list`，里面装的是 `ProductResponse`——就是前面定义过的那个响应模型。没有分页，直接查一个列表出来就行。

> **提示**：分析一个接口，顺序是先理清需求，再结合数据库表看这几个字段在表里分别是干什么的，将来对应到查询逻辑该怎么写。脑子里先有这副图景，再动手写代码。

#### 不能一条 IN 语句查完

这个列表不是一次性查出来的，它是**按分类分别查**出来的。`category` 传了 `life` 和 `medical` 两个值，就得按这两个分类分别去查，查出两个列表，再把两个列表的结果合而为一。

为什么不能一把查完？因为 `LIMIT` 限的是结果集的总条数。你要是这么写：

```sql
SELECT * FROM products
WHERE status = 'active'
  AND category IN ('life', 'medical')
  AND min_premium <= 5000
LIMIT 5;
```

这条语句出来的是**总共 5 条**——完全可能 5 条全是寿险，医疗险一条都没有。而需求要的是每类各 5 条，所以只能一次查一类：

```sql
SELECT * FROM products
WHERE status = 'active'
  AND category = 'life'
  AND min_premium <= 5000
ORDER BY min_premium ASC, id ASC
LIMIT 5;
```

查完这一轮，把 `category` 换成 `medical` 再查一遍，最后手工把两个结果合并。这个"先分类、再合并"的思路，就是整个接口里最复杂的那部分；它想清楚了，剩下的实现都很简单。

#### 把每轮结果攒进同一个列表

仓库层里这个函数，接收的就是刚才那三个参数——分类、价格、每个分类要查的数量。因为将来要把所有查询结果汇总，所以一上来先定义一个空列表，等着装查到的数据。

```python
from decimal import Decimal

from sqlalchemy.orm import Session

from models import Product


def get_candidate_products(
    db: Session,
    category: list[str],
    premium: Decimal | None,
    limit_per_category: int,
) -> list[Product]:
    candidates: list[Product] = []

    for item in dict.fromkeys(category):
        conditions = [
            Product.status == "active",
            Product.category == item,
        ]
        if premium is not None:
            conditions.append(Product.min_premium <= premium)

        products = (
            db.query(Product)
            .filter(*conditions)
            .order_by(Product.min_premium.asc().nulls_last(), Product.id.asc())
            .limit(limit_per_category)
            .all()
        )
        candidates.extend(products)

    return candidates
```

逐段看这几十行代码：

`dict.fromkeys(category)` 是对分类做了个去重，避免同一个分类传两遍、白白查两遍。实际业务里前端不会传重复值，这一层属于顺手加的保护，直接 `for` 循环遍历 `category` 也完全可以。

循环体里第一件事是组装查询条件。第一个是状态得是激活态，也就是上架中的商品；第二个是分类得等于当前这一轮的这个分类，这是基本条件。价格条件是**可选**的——`premium` 可能是 `None`，不是 `None` 时候才往条件里 `append` 一条"价格不高于预算"，不传就不做价格筛选。

条件齐了就开始查：`select` 这张表，`where` 这一堆条件，再按价格升序排——价格一样就按 `id` 排；后面还跟了个 `nulls_last()`，意思是遇到价格为空的行，把它排到最后，不要顶在最前面。最后 `limit` 一下，限的正是"每个分类最多查几条"。

查出来的 `products` 只是当前这一轮、当前这个分类的结果，所以要把它追加到上面那个空列表里去。这里用的是 `extend`：

```python
a = [1, 2, 3]
b = [4, 5, 6]

print(a + b)     # [1, 2, 3, 4, 5, 6]
print(a)         # [1, 2, 3]，原来的列表没变

a.extend(b)
print(a)         # [1, 2, 3, 4, 5, 6]，a 自己变长了
```

用 `+` 拼接是**产生一个新列表**，原来那两个列表一个都没变；而 `extend` 是把第二个列表里的元素一个一个添加到第一个列表里，第一个列表原地就变了。这里要的正是后者：一开始定一个空的，循环里每查到一堆产品就 `extend` 进去，循环走完，这个列表里装的货就越来越全——寿险的、医疗险的，全在同一个篮子里。

> **提示**：保费为什么用 `Decimal` 而不是普通浮点数。计算机里存浮点数是有精度损失的，而 `Decimal` 是十进制的，没有精度损失，属于高精度浮点数。凡是钱、费率这类值，都用它。

到这里，逻辑其实就通了。Service 层什么都不用做：定义一个函数，进来直接调 repository，传参，直接返回，没有任何额外处理。Router 层更简单：调 service，直接返回就行，唯一要多看一眼的是依赖注入——把数据库会话注进来。

实现上还有一条省事的路：需求理清楚之后，把这段描述直接丢给 AI，让它把代码生成出来也可以。

接口这块到此为止。但要注意，我们接下来要开发的**不是一个聊天机器人，而是一个 agent**。这条链路里，AI 从头到尾都是被动的：我们查好了数据喂过去，它挑完就结束。那到底什么是 agent？

### 聊天机器人与 Agent 差在哪

在人工智能领域，agent 通常翻译成"智能体"。

> **定义**：Agent 是一种能够感知环境、进行推理、自主决策并采取行动，以实现特定目标的智能系统。

它跟聊天机器人最大的差别就在这几个动作上：它能**感知环境**——聊天机器人能感知环境吗？不能；它还要能**推理**——推理这个事聊天机器人好像有一点；但它还得能**自主决策并采取行动**。所以 agent 相比聊天机器人要强大很多。

#### 缸中之脑与贾维斯

传统聊天机器人的交互模式非常单一：你问一句，它答一句。它的主要能力就是文本生成——你问一个问题，它一个字一个字往外生成，它只能干这件事，别的干不了。而且它没有自主性，你问什么它答什么，没有更复杂的自主能力。

Agent 就完全不一样了：你只需要向它下达一个命令，它就自主执行；它能操作软件、能发邮件、能联网搜索，什么都可以。你只设定一个最终目标，剩下的它自己去干。演示里用过的 Claude 就是这样的 agent——你说让它写代码，它就开始写代码了。

| 维度 | 聊天机器人 | Agent |
| :--- | :--- | :--- |
| 交互方式 | 你问一句，它答一句 | 给一个目标，自己拆步骤执行 |
| 核心能力 | 文本生成 | 文本生成，外加调用工具真把事情办了 |
| 自主性 | 没有，问什么答什么 | 自主规划、自主决策 |
| 与外界 | 感知不到，也动不了 | 通过工具去看、去查、去发、去订 |

> **结论**：聊天机器人如同一个"缸中之脑"——它只有一个大脑，没有手、没有脚、也没有眼睛鼻子，不能感知外界，不能与外界交互，唯一能做的事就是思考：你问一个问题，它思考出一个答案。

而我们要实现的 agent，像的是钢铁侠头盔里那个智能管家贾维斯。贾维斯能看东西、能联网搜索、能发邮件、能订机票，功能太多了。我们要做的事，就是把传统聊天机器人进化成这样的 agent。

那为什么贾维斯这样的 agent 能做到这些高级功能，传统聊天机器人就不行？拿一个具体需求走一遍就清楚了。

#### 同一个问题，两种回答

用户来问："帮我计划一个五天的北京之旅，预算 8000 元，我喜欢人文历史。"

如果这句话发给的是聊天机器人——聊天机器人就是个大模型——模型会干什么？它只有推理能力，没有别的手段，所以只能根据自己所训练的数据来回答。模型是训练出来的，训练时把人类历史上的语料都塞进去，训了什么数据，它就知道什么知识；没训过的，它不知道。

所以问它五天北京之旅，它只能根据训练时学到的"北京有哪些地点、有哪些玩的地方"，给你编出一套建议。但这套建议一定靠谱吗？不一定。五天里的天气怎么样？预算 8000 元，当下的机票多少钱、火车票多少钱、景点门票现在是多少钱？这些东西全是变化的。而模型训练时这些数据都是死的，训练时是一个值，现在早就变了。它感知不到环境，所以不知道，只能照训练时的样子瞎编。

同一个问题抛给 agent，第一步是**自主规划和决策**。

它会先想：我要规划一个北京之旅，预算 8000，一共五天，用户喜欢人文历史；那要给出这套建议，我需要知道些什么？先干什么、再干什么？由此形成一条规划：先查最近一段时间的机票，再查对应的天气（是下雨还是晴天），再看酒店价格是什么样，然后行程怎么排，最后算下来预算还剩多少。

任务规划靠谁来做？靠模型。模型相当于大脑，大脑负责思考，而 agent 底层也装着模型，模型就在这个大脑里负责思考。

但模型虽然能规划，它有没有查机票、查天气的能力？没有。它没办法与外界交互——这就是下一步要补的东西。

#### 模型加工具，闭环才转起来

为了让 agent 具备这些能力，我们要提前给它准备好一批**工具**：查机票的工具、查天气的工具、搜索景点的工具，等等，还可以更多。工具准备好之后，只要把它们提前告诉 agent 就行了。

于是它先用模型完成规划，再按步骤走：该查机票查机票，该查天气查天气，该查景点查景点，一步一步照规划执行。

执行完拿到工具返回的结果之后，再**感知反馈**：查到了机票、查到了天气，那就根据这些结果来思考，看是否满足了用户的需求。比如预算 8000，查到的机票价格加上酒店价格是不是超了？超了，就带着更严的价格限制重新去查。

```text
   用户目标：五天北京、预算 8000、喜欢人文历史
                         │
                         ▼
                ┌──────────────────┐
                │模型（大脑）做规划│
                └────────┬─────────┘
                         │ 这一步该干什么
                         ▼
                ┌──────────────────┐
                │     调用工具     │
                │  查机票/查天气   │
                └────────┬─────────┘
                         │ 拿到工具结果
                         ▼
                ┌──────────────────┐
                │     感知反馈     │
                │    预算够不够    │
                └────────┬─────────┘
                    否   │    是
              ┌──────────┘    └──────────▶ 输出旅行方案
              ▼
        修正条件，回到规划
```

读图：这是一个不断重复的闭环——调用工具、拿结果、判断是否符合、不符合就带着新条件再调工具、再拿结果再思考，一直转到查到的数据和规划出来的结果符合用户要求为止。

反馈和感知这个动作谁来做？还是模型。因为思考完还要再去调工具，这个思考过程是大脑干的，而大脑就是模型。

> **结论**：所谓 agent，就是把模型结合了工具，给它安上手和脚，从而实现了自主规划、任务执行的能力。工具就相当于模型的眼睛、模型的手、模型的脚。

这个比喻再往深一步：你的手摸到一个很烫的东西，这个信号会反馈到大脑，大脑感知到了，立刻下令"手赶紧缩回来"——这就是对外感知和思考反馈。Agent 是同一个道理：工具让它看到了什么、摸到了什么，这些结果都要回到模型那里再思考一轮，决定下一步。

这么一看，贾维斯还有那么神奇吗？没有了，不就是个模型加工具嘛。

### 工具就是函数

既然 agent 是"聪明的大脑加工具"，那工具到底怎么定义？查天气、查机票这些工具，写成什么样子？

答案出乎意料地简单：**工具的本质就是函数**。

函数大家会写。那我写一个查天气的函数，它就变成了一个查天气的工具；写一个查机票的函数，它就是一个查机票的工具。没有想的那么神奇。

```python
def get_weather(location: str) -> str:
    """Get the weather in a given location.

    Args:
        location: The city name or coordinates.
    """
    return f"Current weather in {location} is sunny."
```

拆开看这个函数，它恰好对应一个工具需要的四样东西：

| 函数里的成分 | 对应工具里的什么 |
| :--- | :--- |
| `get_weather` 这个函数名 | 工具的名字 |
| `location: str` 参数列表 | 工具要接收的参数 |
| 文档注释 | 工具的说明书，描述它是干什么的、参数是什么意思 |
| `return` 出去的值 | 工具执行完的结果 |

那段文档注释写的是 "Get the weather in a given location"，就是"获取指定地点的天气"；`Args` 里写 `location` 是城市名字或者坐标；`return` 的字符串是"该地点当前天气是晴天"。

不过这个工具是**假的**：它压根没去查天气，不管传哪个地方，永远返回晴天。这里重点不是怎么真的查天气，而是怎么定义一个工具——一个能用的工具，长成这样就算定义好了。

工具的问题解决了，真正的麻烦才浮出来：工具是我们自己写的，写在本地代码里；而模型跑在哪儿？跑在云端的服务器上——比如 DeepSeek，就跑在人家官方的服务器上，离我们十万八千里。

模型在远端，工具在本地，那我想让模型调工具，它怎么调？

- 模型怎么知道我们有哪些工具？
- 就算它知道了，它又该怎么调我们的工具？

这两个问题，就是 Tool Calling 要解决的全部。

### Tool Calling：隔着云端怎么调本地

#### 模型怎么知道我们有哪些工具

第一个问题，答案就两个字：**告诉它**。

我们平时访问模型本来就在发请求，请求里本来就带着消息。既然能发消息，那能不能把"我本地有哪些工具"也一起发给它？可以。

具体发什么？把我们本地函数的名字、函数的作用、函数的参数信息，全都告诉它。这几样东西，其实就是刚才那段文档注释里写的——文档注释本来就是在描述这个函数干什么、参数是什么。把这些信息给模型，模型就知道我们本地有哪些工具了。

那这个参数该往哪儿放？别只看那些"复制一段 demo 就能跑"的示例，去看 API 文档里 chat completion 的完整请求参数。请求的 body 里，除了熟悉的 `messages`（一个列表，每项是带 `role` 和 `content` 的字典）、`model`（模型名字），还有 `max_tokens`（限制模型输出的 token 数量）、`temperature`（采样温度，调整模型输出的随机性）等等。其中有一个参数叫 `tools`——它就是"模型可以调用的工具列表"。

> **提示**：学一个 API，直接复制 demo 当然能用；但想弄清楚它到底支持什么，还是得翻 API 文档里的请求参数说明。很多能力就藏在那些你没注意过的字段里。

`tools` 里装的是什么？是一个列表，列表里每一项是一个对象（在 Python 里就是字典），形如：

```json
[
  {
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Get the weather in a given location.",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city name or coordinates."
          }
        },
        "required": ["location"]
      }
    }
  }
]
```

`type` 是类型，`function` 里面三件套：`name` 是要调用的工具名，`description` 是这个工具干什么用的描述，`parameters` 是这个工具的参数说明。一一对回去，就是函数名、文档注释、参数列表。

#### 模型怎么"调用"本地的函数

第二个问题更有意思：模型知道了你有哪些工具，可它在云端、你在本地，它怎么调你？

先反问一句：它**需要**调你吗？不需要。它想要的只是两件事——"我想调某个工具"这个意图传达出去，以及工具执行完的那个结果。至于这个函数到底是谁执行的，它不关心。

那就可以这样：让模型在返回结果里告诉我们，它想调哪个工具、想传什么参数；我们在本地替它调；调完再把结果返回给它。

我们向模型发一个请求，模型一定会返回一个结果。在这个结果里，它完全可以带上"我要调 `get_weather`，参数是杭州"。我们收到之后不要去执行什么远程调用，就在本地把这个函数跑一遍，拿到 "Current weather in 杭州 is sunny"，然后再**重新向模型发起一次请求**，把结果带上——模型就拿到工具的执行结果了。

> **易错点**：模型返回的这段"我要调某某工具"不是最终答案，不能直接转给用户。它只是中间步骤。

> **结论**：整套机制的核心只有两点——第一，让模型知道我们有哪些工具，办法是把工具信息随请求发过去；第二，帮模型调用工具，办法是模型只报"调谁、传什么"，真正执行在我们本地，结果再回传。这套方案有个专门的名字，叫 **Tool Calling**，也就是工具调用。

#### 一次杭州天气的完整往返

把上面两半拼起来，走一遍真实流程。用户问："杭州天气如何？"

```text
① 用户提问"杭州天气如何"，我们的程序把 messages + tools 一起发给模型
        ↓
② 模型推理：天气我自己不知道，但工具列表里有 get_weather，就调它
        ↓
③ 模型返回：tool_calls = get_weather("杭州")
        ↓
④ 我们在本地执行：get_weather("杭州")
        → "Current weather in 杭州 is sunny"
        ↓
⑤ 结果作为新消息，再向模型发起一次请求
        ↓
⑥ 模型拿到结果，结合原来的问题分析
        ↓
⑦ 用中文生成答案："杭州天气晴朗，适合户外游玩"
        ↓
⑧ 我们的程序把最终答案返回给用户
```

把这八步连起来看：

① 用户这句话先扔给模型，让它分析；扔过去的同时，把本地的工具信息也一起传过去——`messages` 里装用户的问题，`tools` 里装本地工具的信息，两者组装成一次请求。

② 模型现在既知道用户的问题，也知道你本地有哪些工具，于是开始推理：要回答这个问题，我需不需要调工具？该调哪个？它一想，天气这事自己不知道，再看工具列表，正好有个 `get_weather`。

③ 于是它返回一个结果，里面带上要调的工具名 `get_weather` 和想传的参数"杭州"。业务复杂的时候，发过去的工具不止一个也没关系——模型会根据自己的描述和用户的提问，自己判断该调哪个。

④ 我们收到这段信息，先判断：它要调的是 `get_weather`，那就在本地把这个函数执行一遍，把"杭州"作为参数传进去，于是查到了杭州的天气。

⑤ 把这个结果再次发给模型——所谓"返回给模型"，其实就是重新发起一次请求，只是这次把工具的执行结果也带上。

⑥ 模型拿到结果，就能分析这个结果，再结合用户原来的问题来回答：杭州的天气是 sunny，也就是晴天。

⑦ 用户是用中文问的，所以模型生成答案时也要用中文："杭州天气晴朗，适合户外游玩。"

⑧ 我们把这句最终答案发给用户，整个流程结束。

这就是一个 agent 底层的工作流程。绕了一圈，核心还是那两点：让模型知道我们的工具，帮模型调用工具。前者把工具信息发过去，后者让模型把"要调谁"告诉我们、由我们代劳。

到这儿，Tool Calling 的基本原理就说完了。

### 本讲提到的题目

课上没有出题，只交代下去参照笔记把候选产品查询接口的 service 与 router 补完，没有给出具体题面，这里不代拟题目。

> **承前启后**：上一节讲完「让模型长出双手：Agent 与 Tool Calling」，下一节接着讲「手写一遍 Tool Calling，再交给 LangChain」。

---

## 手写一遍 Tool Calling，再交给 LangChain
> 对应块：BLK18 | 覆盖分集：P67-P69 | 块标题（分集名）：《手写 Tool Calling 与 LangChain 入门》

Tool Calling 的原理听完了，但真让你动手做，你会做吗？

反正我不会。光在脑子里过一遍就够烦了：得把工具的说明组织成一大坨参数，跟着消息一起发给模型；模型回来一句"我要调谁"，我还得解析出来，按名字去找对应的函数，把参数塞进去，执行完了再把结果拼成一条格式对得上的消息发回去，还得判断这次结果够不够用。这一圈下来，谁写谁崩溃。

所以我先准备了一份示例代码，我们看一眼这东西到底长什么样。不要求你们会写——等会儿学了框架，这些活全都不用你干。但这一遍必须看懂，因为框架帮你省掉的，正是这一圈最脏最累的活。

### 两个函数就是两个工具

新建一个 `Day05` 的目录，里面开一个 notebook，叫 `01_tool_calling`，专门拿来做测试。OpenAI 这个包先确认一下——项目里已经装好了，不用再装。

代码一上来是 `import json` 和 `import os`。`json` 是后面解析模型返回的参数用的；`os` 是读环境变量的，读的是 DeepSeek 的 API Key。

这里有个坑得先说清楚：我们的 Key 写在 `.env` 里，而 `.env` 是配置文件，它不叫环境变量。环境变量是你电脑系统里的那一套东西。所以拿 `os.getenv()` 直接去取，是取不到的。

为此我整了一个小东西：`dotenv` 里的 `load_dotenv()`。它干的事就一句话——把 `.env` 文件里的键值对全部读出来，塞进系统环境变量里。跑完这一行，`.env` 里的配置就都变成环境变量了，你继续用 `os` 去取就行。这么做纯粹是为了跟你们以前的代码习惯对上：以前大家都是 `os.getenv` 取，那就接着这么取。

然后定义两个函数：

```python
import json
import os

from dotenv import load_dotenv
from openai import OpenAI

# .env 是配置文件，不是环境变量，先把它灌进环境变量
load_dotenv()


def get_weather(location: str) -> str:
    """查询指定城市的天气"""
    return f"current weather in {location} is sunny"


def square_root(x: float) -> float:
    """求 x 的平方根"""
    return x ** 0.5
```

不用管这俩函数内部写得多糊弄：一个是查天气，一个是算平方根，数据都是假的。反正是两个函数——有俩函数，就有俩工具。

### 工具得先被描述成模型看得懂的样子

工具有了，接下来要向 AI 发请求，就得先把工具的信息组织出来。这段你自己看，变态不变态？

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "查询指定城市的天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市名称",
                    }
                },
                "required": ["location"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "square_root",
            "description": "计算一个数字的平方根",
            "parameters": {
                "type": "object",
                "properties": {
                    "x": {
                        "type": "number",
                        "description": "要开方的数字",
                    }
                },
                "required": ["x"],
            },
        },
    },
]
```

每一个工具是一个对象，外层 `type` 写死是 `function`；里层 `function` 底下三样东西：`name` 是函数名，`description` 是描述，`parameters` 是参数的说明——类型是 `object`，`properties` 里逐个列参数名和参数类型，`required` 说清哪些必填。

两个工具，就是两段这样的对象堆在列表里。让你写，你肯定写不出来；写出来也大概率少个括号、错个层级。这段先放着不管，等会儿你就知道它为什么可以完全不写。

### 第一次请求，模型只给你一个调用意图

工具描述清楚了，接下来就该向模型发请求了。先准备一个 OpenAI 的客户端，传 API Key 和 `base_url`——这个你们都认识。

```python
client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL"),
)
```

然后调 `client.chat.completions.create()`。参数四个：第一个是模型名，第二个是消息，第三个是工具，第四个是给厂商侧的额外参数——这次要不要思考？把思考模式给它关了，演示工具调用用不着它。

我把这一段调用封装成了一个函数 `send_message(messages)`。也就是说以后你要发请求，只管把消息给我，我帮你发。注意它里面用的是上面定义好的那份工具列表——等于我调模型的时候，把工具和消息一起发过去了。

```python
def send_message(messages):
    return client.chat.completions.create(
        model="deepseek-v4-flash",
        messages=messages,
        tools=tools,
        # 模型侧的额外开关：这次把思考模式关掉
        extra_body={"thinking": {"type": "disabled"}},
    )
```

到这里准备工作就做完了，但注意：还没向模型发请求。先执行一下，确认依赖和 Key 都没问题。接下来只要调一次 `send_message`，请求就真的发出去了。

```python
messages = [{"role": "user", "content": "杭州的天气怎么样"}]
response = send_message(messages)
print(response.choices[0].message)
```

现在问一句：我发起这个请求，AI 是不是一定会给我一个结果？会给。但这个结果会是"杭州今天晴"吗？不是。

为什么？因为这次发过去的请求里带了工具。按前面分析的，模型拿到问题以后，第一件事不是回答，而是判断：这个问题要不要调工具、要调哪一个。判断完了，它把"我要调谁、参数是什么"告诉你。

所以返回值里，`content` 只是一句"我要检查一下天气 for 杭州"，真正关键的是 `tool_calls`：

```text
content='我要检查一下天气 for 杭州'
tool_calls=[
  ChatCompletionMessageToolCall(
    id='call_0f3a...',
    function=Function(
      name='get_weather',
      arguments='{"location": "杭州, China"}'
    ),
    type='function'
  )
]
```

`tool_calls` 里每一个调用有自己的一个唯一 `id`，这是模型自己生成的，将来调用很多的时候，它得靠这个 id 区分不同的调用。`function` 里两样东西：`name` 是要调的工具名字，`arguments` 是要传的参数。注意 `arguments` 是一段 JSON 字符串，不是字典，要拿 `json.loads` 解一下才能用。

问的是杭州天气，两个工具一个算平方根、一个查天气，模型选的是 `get_weather`，参数给的是 `location: 杭州, China`。这一步它只告诉你"我要调谁"，并没有真的调——它调不了，工具在你手里。

### 工具是我们替它跑的

拿到这个结果，接下来干什么？

从里面解析出要调谁、参数是什么；拿着名字去工具列表里找对应的那个函数；把参数塞进去执行。这个逻辑你会写吗？你肯定不会——解析、查找、调用，三步全得自己来。所以这段还是不让你写，我封装了一个 `execute_tool`：你只要把响应给它，再把消息列表给它，它替你把这一圈跑完。

```python
TOOL_MAP = {"get_weather": get_weather, "square_root": square_root}


def execute_tool(response, messages):
    message = response.choices[0].message
    # 模型这条带 tool_calls 的消息也要进历史，后面靠 id 对账
    messages.append(message.model_dump())

    for tool_call in message.tool_calls:
        name = tool_call.function.name
        args = json.loads(tool_call.function.arguments)
        result = TOOL_MAP[name](**args)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": str(result),
        })
    return messages
```

它干三件事：解析出工具名和参数，按名字从工具表里找出那个函数并调用，然后把执行结果追加进消息列表。

追加进去的这条消息，角色是 `tool`——不是 `user`，也不是 `assistant`。这是工具调用独有的一种新角色。它还得带一个 `tool_call_id`，跟刚才那一次调用对上号，模型才知道这条结果是给哪个调用的。

拿到工具的执行结果以后，再向模型发一次消息。这一次消息列表里已经有：用户最初的问题、模型那条带 `tool_calls` 的消息、工具执行结果。三样一起发过去，模型就能给出最终答案了。

```python
execute_tool(response, messages)
response = send_message(messages)
print(response.choices[0].message.content)
```

```text
杭州现在天气是晴天，天气不错，很适合出门活动
```

整条流程，跟前面分析的完全一致。三种角色各就各位：

| 角色 | 谁产生的 | 装的是什么 |
| :--- | :--- | :--- |
| `user` | 用户 | 最初的问题 |
| `assistant` | 模型 | 回答，或者带 `tool_calls` 的调用意图 |
| `tool` | 我们的代码 | 工具执行结果，靠 `tool_call_id` 对账 |

### 这个循环叫 ReAct，它才叫 Agent

再往上看一层：这种模式已经不是"用户问一句、AI 直接生成结果"那么简单了。

有了 Tool Calling 技术，我们可以提供大量的工具给模型，它的工作流程就变成这样：用户提问发给模型，模型首先不是想着生成答案，而是分析——我要回答用户这个问题，是否需要调用工具、需要调用哪一个。判断完了，它把要调的工具信息发给我们，其实就是让我们帮它去调用工具。

所以分工很清楚：用户提问，模型负责思考（要不要调工具、调哪个），我们负责 action（执行工具），执行完再把工具的结果发回给模型。模型继续思考：这个结果是否足以回答用户的问题？不足，就再调一次、再把结果拿回来；直到工具的结果足够回答，才生成答案给用户。

```text
用户提问
   │
   ▼
模型思考（reasoning）──── 要不要调工具？调哪一个？参数是什么？
   │
   │ 返回 tool_calls：工具名 + 参数
   ▼
我们执行（action）────── 按名字找出函数 → 传参 → 拿到结果
   │
   │ 结果包成 role="tool" 的消息，追加进消息列表
   ▼
模型观察（observation）── 这些结果够回答了吗？
   │
   ├─ 不够 ──► 回到「模型思考」，再来一轮
   │
   └─ 够了 ──► 生成最终回答给用户
```

整个过程中包含了用户的提问、模型的分析和思考（reasoning）、我们替它执行的动作（action），以及执行结果反馈回去后的观察（observation）——判断是否足以回答用户问题，能回答就直接生成结果，不能回答就重复前面的步骤。

这就是一个固定流程：请求来了先思考，思考完 action 执行，执行完再思考。模型在思考和执行之间不断重复，直到能够回答用户问题。

> **定义**：像这样"思考 + 执行"不断循环的模式，叫 ReAct。Re 是 reasoning，思考和推理；Act 就是 action，执行。它就是一个 Agent——Agent 不是什么神秘的东西，本质是模型加工具，再加这一圈循环。

原理到这里就说完了。但你们也看出来了，手写 Agent 麻烦不麻烦？非常麻烦，而且随着工具增多，代码会成倍增长——每多一个工具，就多一大段 JSON Schema，多一个分发分支。所以这种手写的方案已经被淘汰了。要提高 Agent 的开发效率，就必须得学工业级的框架。

### 框架登场：LangChain 的来历

LangChain 是由 Harrison Chase 在 2022 年 10 月发布的。还记得那个时候发生了什么事吗？不记得也正常——那正是 ChatGPT 3.5 第一次发布、普通人第一次接触到 AI 的时候，人类第一次认识到 AI 这么强。ChatGPT 刚出来，他就已经把框架整好了，专门用来开发 Agent，这个眼光确实独到。

当然，最早的 LangChain 不是用来开发 Agent 的，它只是用来调模型的，后来才慢慢演变成智能体开发框架。现在 LangChain 里有一套非常完善的 Agent 开发组件。

> **提示**：学任何东西都要经常看官网。官网上才有最完整、最权威的文档，别的地方都是二手的。

打开官网的 Docs，第一屏就写着 LangChain 能干什么：build（构建 Agent）、test and evaluate（测试和评估）、deploy（部署）、monitor（监控，帮你去跟踪、debug Agent 的运行状况）。也就是说，它支持 Agent 开发的完整生命周期——从开发到测试、到部署、再到监控，每一个部分都有对应的工具。

我们最关心的当然是 build 这一部分。build 底下主要有三个框架：

| 框架 | 定位 | 适合做什么 |
| :--- | :--- | :--- |
| LangChain | 开发 Agent 的基础工具 | 写单个 Agent，把模型和工具绑起来 |
| LangGraph | 工作流编排组件 | 把多个 Agent 组合成更复杂的流程 |
| Deep Agents | 建在前两者之上的高级封装 | 图省事，开箱即用，但灵活性不如前两者 |

LangChain 用来开发一些基本的 Agent；LangGraph 用来做多 Agent 的组合，实现更高级的工作流、更复杂的 Agent；Deep Agents 是基于 LangChain 和 LangGraph 做的一种高级 Agent 开发组件，也可以说是半自动化组件，里面已经自带了一些复杂的 Agent 开发组件，用起来更方便，但灵活性就不如前面两种。所以目前企业里用得最多的，还是 LangChain 和 LangGraph。

除了这三个，还有个 LangSmith，负责测试、观察、评估和部署这一套——很可惜是收费的，我们就不去研究了，主要就学 LangChain 和 LangGraph。

### 四步做出一个 Agent

接下来通过一个快速入门，看看用 LangChain 开发 Agent 到底有多方便。

第一步是装依赖：LangChain 本身的依赖，加上 LangChain 对模型支持的依赖。LangChain 目前支持全球大部分的模型，包括 DeepSeek，所以除了 `langchain`，还要装 `langchain-deepseek`。这两个项目里都已经装好了，API Key 也都配好了，这两步直接过。

然后是正式开发。在 LangChain 里开发一个 Agent 就四步，在脑子里记住就行：加载环境变量 → 定义工具 → 定义 Agent（绑定模型和工具）→ 调用 Agent。

有人问，模型不用定义吗？不用。模型在 LangChain 里非常简单，只要写个名字就够了。所以定义 Agent，本质上就是在绑定模型和工具。有了工具、有了模型、Agent 也有了，最后一步就是调用它。

在 LangChain 里创建 Agent 要用到一个函数叫 `create_agent`，我们提供模型和工具就行。新建一个 notebook，叫 `02_hello_langchain`。

#### 把 .env 灌进环境变量

第一步加载环境变量。我们的配置是配在 `.env` 里的，而 `.env` 不属于环境变量，需要把它加载到环境变量里去。

这里肯定有人要问：我们之前不是有 pydantic 吗？`app/core/config` 里不是有 Settings，里面不是已经有 LLM 的 API Key 了吗，为什么还要整一个加载环境变量？

> **易错点**：pydantic 的 Settings 只干了一件事——读取 `.env`，把值存进 Settings 对象里。它并没有把这些值放进环境变量。而 LangChain、OpenAI SDK 这些库取 Key，走的是 `os.environ`。所以你有 Settings，不等于环境变量里有 Key；要让它进环境变量，还是得用 `load_dotenv()`。

所以第一步就是执行 `load_dotenv()`，一行，完事。这个函数你不会写也没关系，直接复制，因为它是死的，不用改。

```python
from dotenv import load_dotenv

load_dotenv()
```

#### 一个装饰器替掉整段 JSON

第二步定义工具。定义工具就是定义函数，函数大家都会写。

```python
from langchain.tools import tool


@tool
def get_weather(location: str) -> str:
    """查询指定城市的天气"""
    return f"current weather in {location} is sunny"
```

但是别忘了，我们定义工具，最终是要把工具发给模型的。要发给模型，就得组织前面那一大堆参数——这个要不要我们写？不用。只需要给函数加一个装饰器：`@tool`。

这个 `@tool` 从 `langchain.tools` 里导，或者直接从 `langchain` 导也可以。加上它，就等于标记这个函数不是个普通函数，而是一个工具。

加上装饰器以后，工具信息由 LangChain 自动组织。怎么组织？刚才写的文档注释就是对工具的描述，将来它会成为 `description`；名字呢，就是函数名；参数呢，就是你传的那个参数。函数名有了、参数有了、描述也有了，这些信息就不用你管了，`@tool` 这个装饰器、或者说 LangChain 已经全帮你整好了。

所以定义工具非常方便：加一个装饰器，加一段文档注释，就够了。想加更多的工具呢？继续写 `@tool` 就完了。刚才那个求平方根的也搬过来：

```python
@tool
def square_root(x: float) -> float:
    """求 x 的平方根"""
    return x ** 0.5
```

恭喜，又写好一个工具了。跟你们以前定义函数是不是没什么区别？区别只有一个——加装饰器、加文档注释。但这个习惯一定要养成：注释不是写给同事看的客套话，它就是模型判断这个工具是干什么的唯一依据，写得含糊，模型就选错工具。

#### create_agent 把模型和工具绑在一起

第三步定义 Agent，用 `create_agent` 这个函数。

```python
from langchain.agents import create_agent

agent = create_agent(
    model="deepseek-v4-flash",
    tools=[get_weather, square_root],
)
```

名字一看就懂：create 是创建，agent 是智能体——创建智能体，很直白。

参数就两个。第一个 `model` 传模型，怎么填？填模型名就行了，非常简单，`deepseek-v4-flash`。顺一句选型：别一看 pro 就往上冲，flash 现在升级过，比 pro 还强。第二个 `tools` 传工具，它的类型是个 Sequence，中括号就是列表，列表里装工具——一个 `get_weather`，一个 `square_root`，传进去就行。

就这么两行，Agent 创建好了。前面手写的那一大堆——JSON Schema、按名字分发、消息拼装——一行都没写。

#### invoke 传字典，收回一个字典

最后一步，调用 Agent，用 `invoke` 函数，也就是 `agent.invoke()`。

```python
response = agent.invoke({
    "messages": [
        {"role": "user", "content": "杭州天气如何"}
    ]
})
```

`invoke` 里传的是一个字典，这个字典必须有一个属性叫 `messages`。为什么？因为你最终调用就是在发消息，工具不需要你发了——工具已经绑进 Agent 里了，我们只管发消息。所以传字典，字典里是 `messages`；消息得是个列表，所以加中括号；中括号里是具体的消息，一条消息就是一个带 `role` 和 `content` 的字典，`role` 写 `user`，`content` 写"杭州天气如何"。

要传多条消息呢？列表里写多个就完了。

返回值用 `response` 接住，打印出来东西很多，但整体是个字典结构。有意思的地方在这儿：我传进去的时候，`messages` 里只有一条消息；它返回的 `messages` 里有四条。

| 消息类型 | 对应角色 | 内容 |
| :--- | :--- | :--- |
| `HumanMessage` | `user` | 我发的"杭州天气如何" |
| `AIMessage` | `assistant` | 判断出该调 `get_weather`，带 `tool_calls` |
| `ToolMessage` | `tool` | 工具执行结果 `current weather in 杭州 is sunny` |
| `AIMessage` | `assistant` | 杭州当前天气是晴朗的 |

第一条 `HumanMessage`，就是人类的消息，也就是我发的用户消息。第二条 `AIMessage`，是 AI 消息，它说"用户询问了杭州的天气，我应该去调 get_weather for 杭州"，还带着 `tool_calls`——模型决定调工具了。再往下是 `ToolMessage`，就是工具执行的结果，因为 AI 说了要调 `get_weather`，我们替它调完，得把结果再发回给 AI。最后一条又是 `AIMessage`，这才是 AI 的回复：杭州当前天气是晴朗的。

如果不想看这么多，只想看最后那句回答：`response` 是个字典，取 `messages`；`messages` 是个列表，最后一条用 `[-1]`；再取它的 `content`。

```python
print(response["messages"][-1].content)
```

```text
杭州当前天气是晴朗的
```

这就是 AI 的结果。notebook 的好处是上面的执行结果会被记住，下面这几行直接跑就行，不用从头再执行一遍。

到这里，LangChain 的快速入门就说完了。回头把四步捋一遍：加载环境变量——`load_dotenv()`，死代码，复制就行；定义工具——写函数，加 `@tool`，加文档注释；定义 Agent——`create_agent` 传模型名和工具列表；调用 Agent——`invoke` 传一个带 `messages` 的字典，里面装消息列表，有一条传一条，有多条传多条。

就这么四步，手写那一圈最麻烦的活，全被框架吃掉了。

> **承前启后**：上一节讲完「手写一遍 Tool Calling，再交给 LangChain」，下一节接着讲「LangChain 模型：三种初始化与直接调用」。

---

## LangChain 模型：三种初始化与直接调用
> 对应块：BLK19 | 覆盖分集：P70-P74 | 块标题（分集名）：《LangChain 模型初始化与调用》

上一轮跑 `create_agent` 的时候，我们往第一个参数里塞了一个字符串——`"deepseek-v4-flash"`——然后就完事了。

你回头看一眼这行代码，会发现它跳过了挺多东西。按道理，要用一个模型，怎么也得知道它的 API Key 是什么、base_url 在哪吧？我一个都没传。模型还有一堆自己的参数——`temperature` 温度、要不要思考（thinking）——这些也没法指定，因为传进去的只是一个名字。

它照样跑通了。这一篇要做的，就是把这个"照样跑通"拆开：LangChain 到底替我们推断了什么，推开之后又该怎么自己接管。

### 只写一个模型名，为什么也能跑通

关键在一个概念：**模型提供商**（provider）。

我们今天用的模型，每一个都有出处。GPT 系列是 OpenAI 的；DeepSeek 是 DeepSeek 家的；Claude 是 Anthropic 的（`Claude Code` 就是这家做的）；Gemini 是 Google 的；再往下还有阿里云。

出处不一样，最直接的后果是：接口地址不一样。OpenAI 有自己的地址，DeepSeek 有自己的地址，Anthropic 有自己的地址，Google 有自己的地址，阿里云也有自己的地址。所以 `init_chat_model` 里才会有 `model_provider` 这个参数——你用的提供商不同，这个值就得填不同的。

那为什么刚才什么都没填也行？

因为 LangChain 会根据模型名字来推断提供商。你传的模型名以 `deepseek` 开头，它就认定你是 DeepSeek 平台；名字以 `gpt` 开头，它就认定你是 OpenAI。这套推断在它支持的范围内相当准。

> **易错点**：推断的依据是名字，不是你实际连的地址。名字一样、平台不一样的时候，它就会推错。

最典型的例子是阿里云百炼。百炼上挂的模型非常多，不只是千问，还有 DeepSeek、Gemini 这些别家的模型。你在百炼上调用一个 DeepSeek 模型，LangChain 按名字一推，认为你连的是 DeepSeek 官方平台——可你实际是阿里云。这就推错了。

反过来说，如果你用的就是 DeepSeek 官方平台，那推断不会出问题。

那推断出提供商，到底换来什么？两件事。

第一件，base_url 定下来了。每个提供商有自己固定的接口地址，LangChain 一查表就知道该往哪发请求，所以你不用填 `base_url`。

第二件，api_key 有了来源。LangChain 里有一套默认的环境变量约定：发现你的提供商是 OpenAI，就去环境变量里读 `OPENAI_API_KEY`；发现是 DeepSeek，就去读 `DEEPSEEK_API_KEY`。所以 `api_key` 你也不用填。

> **提示**：这套约定认的是标准名字。环境变量里必须叫 `DEEPSEEK_API_KEY`，你图省事写成 `DSPIK`，它就找不到，接着就是报错。

两条链合起来，就是下面这个过程：

```text
init_chat_model("deepseek-v4-flash")
              |
              v
   [1] 按名字前缀推断提供商  ->  DeepSeek
              |
              v
   [2] 查表得到接口地址  ->  不用填 base_url
              |
              v
   [3] 查表得到变量名  ->  不用填 api_key
              |
              v
   读环境变量 DEEPSEEK_API_KEY
              |
              v
        客户端组装完成
```

名字进，客户端出。中间那三步——推断提供商、查地址、读密钥——平时都被这一行调用遮住了。

当然，这一整套你可以完全不理。手动传 `api_key` 和 `base_url`，一样能跑，LangChain 不会拦你。只是大多数时候没必要。

### init_chat_model：名字就把用途说完了

要把模型的参数改掉，光在 `create_agent` 里传个名字是不够的——那种写法用的全是默认参数，你改不了。要改，就得先把模型单独初始化出来。

LangChain 提供的这个函数叫 `init_chat_model`。拆开看：`init` 是初始化，`chat` 是聊天，`model` 是模型。初始化聊天模型，函数名就是它的说明书。

它接收的参数里，头两个最要紧。

`model` 是模型名称，**必填**。初始化一个模型，总得知道是哪个模型。

`model_provider` 是模型提供商。前面说过，每个模型都有自己的出处，出处不同，值就不同。

除了这两个，模型自己的参数也都支持：`temperature` 温度、`max_tokens` 输出长度上限，还有控制思考的 `extra_body`。这些不是 LangChain 发明的，是模型平台本身的参数，`init_chat_model` 只是把它们原样透传下去。

把常填的几个摆在一起：

| 参数 | 作用 | 不传的时候 |
| :--- | :--- | :--- |
| `model` | 模型名称 | 不给不行，必填 |
| `model_provider` | 模型提供商 | 按模型名前缀推断 |
| `api_key` | 访问密钥 | 按提供商去环境变量里读约定的那个 |
| `base_url` | 接口地址 | 按提供商查表取默认地址 |
| `temperature` | 采样温度，越大越发散 | 用平台默认值 |
| `max_tokens` | 输出长度上限 | 用平台默认值 |
| `extra_body` | 平台私有参数，比如 thinking | 不生效 |

看清楚这张表，两种用法就自然分出来了。

**默认初始化**：只填一个名字。提供商由 LangChain 按名字推断，地址查表得到，密钥去环境变量里读。三样你都不用管。

**自定义初始化**：名字之外，把提供商、`api_key`、`base_url` 三样亲手填进去。

> **定义**：默认初始化是"只给名字，其余交给约定"；自定义初始化是"名字照给，但提供商、密钥、地址都得自己递上去"。

那什么时候用哪个？

分界线只有一条：**默认初始化只对 LangChain 支持的模型有效**。

它支持的那一批，基本就是前面点过名的那几家——OpenAI、DeepSeek、Anthropic、Google。国内的模型目前被支持的很少，DeepSeek 是其中一个。

所以小米的模型就不行。你填一个小米模型的名字进去，它推不出提供商——不是"可能推错"，是根本推不出来，因为它不在那份名单里。

> **提示**：我说的是**默认初始化**不支持，不是说 LangChain 用不了这个模型。用不了默认路线，走自定义路线就行，模型照样能调。

逻辑就变成了一个很实际的选择：想省事、不想写 `api_key` 和 `base_url`，那就用 DeepSeek，一个名字搞定；用的是名单外的模型，就老老实实走自定义初始化，把三样东西填全。

### 默认初始化：五步跑通一个 agent

先把流程对齐。

之前创建 agent 是四步：加载环境变量、定义工具、定义 agent（把模型和工具绑上去）、调用 agent。现在模型初始化被单独拎出来了，所以多一步，变成五步：

```text
[1] 加载环境变量   load_dotenv()
        |
[2] 定义工具       @tool
        |
[3] 初始化模型     init_chat_model(...)   <-- 新增的一步
        |
[4] 定义 agent     create_agent(model, tools)
        |
[5] 调用 agent     agent.invoke({...})
```

这五步后面会反复写，写到不用想。

先看加载环境变量。我们要把 `.env` 文件的内容读回环境里，用的是 `python-dotenv` 里的 `load_dotenv`：

```python
from dotenv import load_dotenv

load_dotenv()
```

调一次，`.env` 里的键值对就进了环境变量。后面 LangChain 去环境变量里找 `DEEPSEEK_API_KEY`，找的就是这里加载进来的。

第二步定义工具，还是快速入门里那个天气工具，原样复制过来。工具本身这一节不展开，只需要知道它得先于 agent 定义：

```python
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """查询指定城市的天气。"""
    ...  # 与快速入门那份天气工具一致，直接复制
```

第三步是这次新加的重点——初始化模型：

```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "deepseek-v4-flash",
    temperature=1.2,
    extra_body={"thinking": {"type": "disabled"}},
)
```

这里有几个细节值得停一下。

导入路径是 `langchain.chat_models`，不是 `langchain_classic`。写错包，函数就找不到。

模型名是**位置参数**，可以直接放在第一个位置，也可以写成 `model="deepseek-v4-flash"`。两种写法等价，看个人习惯。

名字后面跟的 `temperature` 和 `extra_body`，就是"模型自身参数"。这些参数不是 LangChain 规定你必须写的，是模型平台自己能接的。DeepSeek 的官方文档里列了 `messages`、`thinking`、`max_tokens`、`temperature` 这些，凡是文档里有的，这里都可以往下传。

控制思考的写法是 `extra_body` 里嵌一层字典：

```python
extra_body={"thinking": {"type": "enabled"}}    # 开启思考
extra_body={"thinking": {"type": "disabled"}}   # 关闭思考
```

`enabled` 是开，`disabled` 是关。

> **提示**：这种嵌套写法不用背。它是 DeepSeek 平台的私有约定，去官方文档里抄现成的就行——不同模型这一层长得完全不一样，记也记不住。

第四步，把模型和工具交给 `create_agent`：

```python
from langchain.agents import create_agent

agent = create_agent(model, [get_weather])
```

模型是位置参数，直接传；工具要用列表包起来，哪怕只有一个。

第五步，调用：

```python
response = agent.invoke(
    {"messages": [{"role": "user", "content": "杭州北京天气如何"}]}
)
print(response)
```

agent 的入参是一个字典，字典里 `messages` 对应一个列表，列表里再放消息字典，每条消息有 `role` 和 `content`。

跑起来，第一感觉是慢。这时候 `thinking` 还开着，模型得先把思路走一遍再动手，返回结果里除了答案还多出一段 `reasoning content`。慢得有道理——它确实多干了一层活。

把 `enabled` 改成 `disabled`，再跑一次——三秒，时间差不多缩了一半，`reasoning content` 也没了。

这就是模型自身参数的实际影响：一行配置，换来的是一倍的时间差。

### 自定义初始化：把千问伪装成 OpenAI

现在换个模型。这次用阿里云百炼上的千问。

按前面的分界线，千问在名单外。LangChain 不认识它，推不出提供商，也不知道你的 `api_key` 和 `base_url` 是什么。三样都得自己递。

第一样就卡住了：`model_provider` 填什么？

直觉是填"阿里"。不行——LangChain 不认识这个值，填了等于没填。

能填的是 `openai`。把千问**伪装**成 OpenAI。

为什么这么干能成？因为 OpenAI 的接口规范事实上已经成了行业通用标准。千问兼容它，DeepSeek 也兼容它——你之前拿 OpenAI 的客户端调 DeepSeek，调得通，那就是兼容。既然大家都长一个样，把提供商写成 `openai`，请求就能发出去。

但伪装是有代价的。你一旦写了 `model_provider="openai"`，LangChain 就默认你要去 OpenAI：地址会指向 OpenAI 的接口，密钥会去读 `OPENAI_API_KEY`。而你根本不是 OpenAI 的用户。所以 `api_key` 和 `base_url` 这两样，必须手动填——你不填，它就真去找 OpenAI 了。

> **定义**：所谓自定义初始化，就是把 LangChain 不支持的模型伪装成 OpenAI 提供商，同时自己把 `api_key` 和 `base_url` 递上去。

小米的模型是同一个套路：模型名填小米的，提供商填 `openai`，`api_key` 和 `base_url` 填小米的。千问就填千问的。

千问这两样东西有个特点——**每个人都不一样**。你得自己在阿里云百炼上开通、申请，拿到属于自己账号的那一份。

拿到了往哪放？还是放 `.env`。前面说过，这类东西一律进配置文件，不要写死在代码里：

```dotenv
DASHSCOPE_API_KEY=<你在百炼上申请到的 key>
DASHSCOPE_BASE_URL=<百炼兼容 OpenAI 模式的接口地址>
```

变量名可以随便起——反正 LangChain 不认识这套，不会自动去读。阿里这边习惯叫 `DASHSCOPE_` 开头，跟着这个习惯走就行。

变量进了 `.env`，不等于进了代码。`load_dotenv()` 会把它们加载进环境变量，但你还得从环境变量里**读出来**。

读的工具是 `os`——Python 内置的操作系统模块，代表当前这套操作系统；`os.environ` 就是它的环境变量表。两种读法都行：

```python
import os

api_key = os.getenv("DASHSCOPE_API_KEY")
base_url = os.getenv("DASHSCOPE_BASE_URL")

# 完全等价的另一种写法
# api_key = os.environ.get("DASHSCOPE_API_KEY")
# base_url = os.environ.get("DASHSCOPE_BASE_URL")
```

> **易错点**：不要图省事把 key 直接粘成字符串字面量。key 一进代码，就会跟着代码进版本库；`os.getenv` 读的是环境，代码里只有变量名，泄露不了。

把三样凑齐：

```python
import os
from dotenv import load_dotenv
from langchain.chat_models import init_chat_model

load_dotenv()

model = init_chat_model(
    model="<百炼上开通的千问模型名>",
    model_provider="openai",                 # 伪装
    api_key=os.getenv("DASHSCOPE_API_KEY"),
    base_url=os.getenv("DASHSCOPE_BASE_URL"),
)
```

模型拿到了，后面的 agent 代码一个字都不用改——照样 `create_agent(model, [get_weather])`，照样 `invoke`。

验证的办法很简单：问它"你是谁"。

它回答"我是通义千问，阿里巴巴……"。答对了，说明请求真的打到了千问，而不是打到 OpenAI 那边报个 401 回来。

### Model 类初始化：当平台不吃 OpenAI 那一套

伪装成 OpenAI 这条路，成立的前提是"对方兼容 OpenAI 规范"。

但 OpenAI 这套规范并不是什么硬性标准。它之所以成了事实标准，纯粹是因为 OpenAI 起步早、先发优势大，后来者不得不兼容它。可总有不愿低头的：我就不兼容你。

碰上这种平台，前面两招同时失效——既没法走默认初始化（LangChain 不认识），也没法伪装成 OpenAI（人家压根不认这套规范）。

路只剩一条：用**专用的模型类**。

先把三条路摆在一起看：

| 你的模型 | 提供商怎么写 | api_key | base_url | 用哪个入口 |
| :--- | :--- | :--- | :--- | :--- |
| LangChain 支持的（DeepSeek、OpenAI 等） | 不写，按名字推断 | 不写，去环境变量读 | 不写，查表得到 | `init_chat_model(名字)` |
| 兼容 OpenAI 但 LangChain 不认识（千问） | 写 `"openai"` 伪装 | 手填 | 手填 | `init_chat_model(...)` |
| 不兼容 OpenAI（Ollama） | 不需要 | 本机不需要 | 本机 localhost | `ChatOllama(...)` |

第三条路的代表是 Ollama。

Ollama 是个本地模型部署平台。它的好处是上面什么模型都能挂——GPT 也行、千问也行、DeepSeek 也行，只要你在本地把它跑起来。坏处是它的接口不兼容 OpenAI。

好在 LangChain 给它配了专门的包：`langchain_ollama`。这个包在项目依赖里已经有了，不用另外装。

用的时候不再导 `init_chat_model`，而是从 `langchain_ollama` 里导 `ChatOllama`——这个类就是"模型类"，只管 Ollama 一家：

```python
from langchain_ollama import ChatOllama

model = ChatOllama(
    model="qwen3.5:0.8b",
    temperature=1.1,
)
```

为什么这里不用传 `api_key` 和 `base_url`？

因为 Ollama 启动在你本机。你打开它就能直接聊，什么时候问过你密钥？没有。地址也不用写，它就在本机，默认就是 `localhost`。

> **易错点**：`ChatOllama` 的模型名参数叫 `model`，不叫 `name`。写成 `name=` 会直接报 `Field required`——它缺的是 `model` 这个字段，不是没给值。报错信息看着像是在说"少传了个必填项"，实际是键名对不上，改成 `model=` 就通了。

模型建好，往 agent 里一传，流程和前面完全一样：

```python
agent = create_agent(model, [get_weather])
response = agent.invoke(
    {"messages": [{"role": "user", "content": "杭州天气如何"}]}
)
```

问它"你是谁"，它答"我是千问 3.5"——本地部署的那份 0.8B。再问天气，工具正常触发，答"杭州当前晴朗"。

速度是另一个惊喜：800 毫秒，连一秒都不到。原因很朴素——模型就跑在你这台机器上，没有网络往返。

### 不建 agent，也能调模型

前面三种初始化，最后都把模型塞进了 agent：模型 + 工具绑在一起，形成一个能自己决定"先干什么、再干什么"的东西，然后再调。

这条路线是为 agent 开发准备的。但公司里的需求不一定都长这样——很多场景里根本用不上工具，你只是想问模型一句话、拿一句回答。为了这个去建一个 agent，纯属多余：白搭一层工具绑定的开销，还多绕一道。

模型本身就能直接调。

流程也从五步缩成三步：加载环境变量、初始化模型、调用模型。

```python
from dotenv import load_dotenv
from langchain.chat_models import init_chat_model

load_dotenv()

model = init_chat_model(
    "deepseek-v4-flash",
    extra_body={"thinking": {"type": "disabled"}},
)

response = model.invoke("你好，你是谁")
print(response)
```

`init_chat_model` 前面都讲过，这里只多一个经验：调模型这种场景，思考模式关掉更划算。开着思考，每次都要先生成一段推理再作答，很耗时；关掉，直接出答案。

> **提示**：`extra_body={"thinking": ...}` 这一整段是 DeepSeek 的私有写法，换个模型就失效。它的可移植性只限于 DeepSeek 系列。

调用用的是 `invoke`，和 agent 一样。但**入参完全不一样**：这里直接给一个字符串。`model.invoke` 的签名接受字符串，你不必再套一层 `{"messages": [...]}`。

跑出来一句"你好呀，我是 DeepSeek"。调用成功。

真正需要留心的是**返回值**。

agent 的 `invoke` 返回的是一个字典，字典里 `messages` 是一个消息列表——human message、AI message 全在里面，是完整的一段对话。而模型的 `invoke` 只还你一条 `AIMessage`，消息内容在它的 `.content` 属性上。

没有了那层字典，也没有了历史消息列表。这是最容易出岔子的地方，下一节单独摆开对比。

### 模型调用与 agent 调用：同一个 invoke，两套进出

两个调用用的是同一个方法名 `invoke`，形状看着也一样，但进出的东西完全是两套。

| 对比项 | agent 调用 | 模型调用 |
| :--- | :--- | :--- |
| 方法 | `agent.invoke(...)` | `model.invoke(...)` |
| 入参 | `dict`，键 `messages` 指向消息列表 | 字符串，或消息列表 |
| 返回 | `dict`，键 `messages` 指向消息列表 | 一条 `AIMessage` |
| 取文本 | 从 `messages` 里取最后一条 | 直接读 `.content` |

一句话概括：agent 的入参是"一个带消息列表的字典"，出参也是"一个带消息列表的字典"；模型的入参是"一句字符串"，出参是"一条消息"。

模型调用相当于把 agent 返回的那一堆里，最后那条 AI message 单独摘出来给你。中间那些 human message、中间步骤，它不返回。

> **易错点**：写过 agent 之后再来调模型，手会自己往 `invoke` 里塞 `{"messages": [...]}`，也会顺手写 `response["messages"]` 去取值。这两下都会失败——前者不是它要的形状，后者它根本不是字典。

这个区别务必记牢。后面不管叠多少工具、多少记忆，`invoke` 这个入口不会变，但它是"agent 的 invoke"还是"模型的 invoke"，进出永远按上面这张表走。

### 多条消息：把对话历史塞进一个列表

只传一个字符串，一次只能说一句话。想带上历史、想加系统提示，怎么办？

`invoke` 的入参本来就允许两种形态：`str`，或者一个 sequence——也就是列表。多消息走第二种。

但列表里不能直接放字符串。放几条字符串进去，模型分不清哪句是用户说的、哪句是它自己前面说的。所以列表元素要换成字典，带上 `role` 和 `content`：

```python
response = model.invoke([
    {"role": "system", "content": "你扮演小团团，用小团团的语气回答问题"},
    {"role": "user", "content": "你好，你是谁"},
])
print(response.content)
```

`system` 那条是角色设定，`user` 那条是真正的提问。顺序就是对话顺序。

小团团是个网红，做游戏直播的，说话有一股很重的个人腔调。这正好用来验证系统消息有没有生效——模型平时一开口都是正经的自我介绍，被这么一设，它张口就是"哎呀呀，我是小团团，就是那个可爱又迷人的小团团"。

它照着演了，说明 `system` 这条被读进去了。

> **提示**：模型自己不记上一轮。所以每一轮对话，`system` 那条都要重新放回列表最前面，它才记得自己该演谁。

于是模型调用的入参就收束成两种：一句字符串，或者一个装消息字典的列表。前者够用就前者，要历史、要角色就用后者。

这两条路都亲手跑一遍——尤其是 `{"messages": [...]}` 和 `"你好"` 这两者的区别，跑一次比看十遍表都记得住。

> **承前启后**：上一节讲完「LangChain 模型：三种初始化与直接调用」，下一节接着讲「消息、结构化输出与 Agent 的调用方式」。

---

## 消息、结构化输出与 Agent 的调用方式
> 对应块：BLK20 | 覆盖分集：P75-P79 | 块标题（分集名）：《消息类型、结构化输出与流式调用》

我们一直在用 LangChain 调模型。不知道你们注意到没有：不管问什么，返回来的东西长得都一个样——一个带着角色、带着内容的结构。这不是巧合，是 LangChain 把我们和模型之间来回的消息全都封装起来了。

顺着这条线往下走，这一讲讲四件事：消息为什么要封装成类、怎么让模型吐出固定结构的数据、agent 的两种调用方式各是什么脾气、系统提示词到底该放在哪。

### 消息不是字典，是类

仔细想一想：我们发给模型的消息，模型返回给我们的消息，是不是都包含两样东西——`role` 和 `content`？

`role` 是消息的角色，`content` 是消息的内容。就这两个，是所有消息必备的东西。

角色有哪几种？`system`、`user`、`assistant`，还有 `tool`。前面做会话记忆的时候已经用过前三种：`system` 定人设，`user` 放我们说的话，`assistant` 存 AI 返回的话，几条拼起来就是一轮带记忆的对话。

既然格式这么固定，以前我们是怎么传的？拼字典：

```python
messages = [
    {"role": "system", "content": "你是一个专业的 AI 助手"},
    {"role": "user", "content": "你好，你是谁"},
    # 要带会话记忆，再补上 AI 说过的话
    {"role": "assistant", "content": "……"},
]
```

写两条还能忍，写到三条、五条就开始烦了：要加引号，要加冒号，键名还得一个字母都不错，写错一个整段就跑不通。消息越长，这段代码越没法看。

LangChain 的态度很直接：既然消息就这么几种固定格式，你们别写了，我封装。

于是所有消息统一成一种类型，叫 `BaseMessage`——一切消息的顶级父类，角色和内容这些最基本的信息都在它里面。光有父类还不够，光有父类你还是得自己说明"这条是系统消息、那条是用户消息"，所以它干脆按角色派生出一批子类，把角色定死：

| 消息类 | 角色 | 用在哪 |
| :--- | :--- | :--- |
| `SystemMessage` | `system` | 定人设、定规则 |
| `HumanMessage` | `user` | 我们说给模型的话 |
| `AIMessage` | `assistant` | 模型返回的话，存进记忆 |
| `ToolMessage` | `tool` | 工具执行结果回填给模型 |

以后传消息，不传字典，直接传这些类的对象：

```python
from langchain_core.messages import SystemMessage, HumanMessage

messages = [
    SystemMessage("你是一个专业的 AI 助手"),
    HumanMessage("你好，你是谁"),
]

response = model.invoke(messages)
print(response)
```

看括号里：第一个位置参数就是 `content`，角色一个字都不用写。`SystemMessage` 天生就是 `system`，`HumanMessage` 天生就是 `user`。上面这两行对象，和最开始那两条字典完全等价，但短、清楚、不会写错。

跑一下，正常返回。

> **结论**：以后传多条消息，别拼字典，直接造消息对象。要记住的名字就四个：`SystemMessage`、`HumanMessage`、`AIMessage`、`ToolMessage`。

### 让模型吐出固定结构的数据

模型给我们的回答，本质上永远是一个字符串。你问"你是谁"，回来的是一串文字；聊得再花，回来的还是一串文字。

字符串适合聊天，不适合干高级的活。比如我想让模型输出一段 JSON，我拿到 JSON 再转成对象；或者干脆让它直接给我一个对象——默认做不到。

结构化输出就是来解决这个的：让模型输出的东西不再是普通字符串，而是固定结构的数据。

顺便说一句，这类需求根本用不着 agent。前面说过很多场景不需要 agent、直接调模型就够，结构化输出就是最典型的一种——它要的只是一次干净的回答，不需要模型自己规划、自己调工具。

举个具体需求。我要写一本小说，让模型帮我设定一座城市——月球的首都。这座城市得有自己的背景信息：有名字、有位置、有氛围、有经济来源。我要的不是一段散文，是四个字段各就各位。

整件事在原来的流程上只多两步：先把结构定义下来，再把结构跟模型绑上。

#### 先把结构定下来

结构用什么定？用 Pydantic 的 `BaseModel`。它在项目里的本职就是约束，这里正好拿它约束模型的输出。

```python
from pydantic import BaseModel, Field


class CapitalInfo(BaseModel):
    name: str = Field(description="首都的名字")
    location: str = Field(description="首都的位置")
    vibe: str = Field(description="首都的氛围")
    economy: str = Field(description="首都的经济来源")
```

类名 `CapitalInfo`，意思是首都信息。四个字段：`name`、`location`、`vibe`、`economy`。

每个字段都用 `Field` 包一层，配上 `description`。`description` 是写给模型看的说明书——这个字段是干什么的、该填什么内容，一句话说清楚。写英文写中文都行，模型都认。

这一步做完，约束就有了：将来模型必须按这个类吐数据，字段名、字段含义都由这个类说了算。

#### 再把结构绑到模型上

绑定用 `with_structured_output`。`structured` 是结构化，`output` 是输出，两个词合起来就是"按固定结构输出"。括号里传的参数叫 `schema`，意思就是约束，值填刚才那个类。

```python
model_with_structured_output = model.with_structured_output(schema=CapitalInfo)

response = model_with_structured_output.invoke(
    "你是一个科幻小说作家，帮我设定月球的首都城市的背景信息"
)

print(response)
```

这里有个容易看漏的点：`with_structured_output` 不是给原模型"设置"一下就完了，它返回的是一个新的模型——一个带结构化输出能力的模型。所以下面要用这个新模型去 `invoke`，不能还拿原来的 `model` 调。

变量名随便起，叫 `model_with_structured_output` 只是让名字自己说明问题。

绑完之后，调用的写法反而轻了：一条消息就够，不用像前面那样一次塞好几条。返回的也不再是字符串，而是 `CapitalInfo` 类型的对象，`response.name`、`response.location`、`response.vibe`、`response.economy` 都能直接点出来。

跑出来的内容是这样：城市名字挺有小说味；位置在月球正面澄海中央，依托一处巨大的天然熔岩洞穴建起来；氛围是银色赛博朋克与古典未来主义的混合，街道上铺着会发光的苔藓；经济来源里有农业，有月影交易所，还有量子加密的金融数据——编得还挺全。

这些东西如果不加结构化输出，就是一大段字符串，你得自己切、自己解析，字段还可能缺一两个。加了之后，四个字段一个不少，严格按类来。

这套做法不限于城市。想让人物设定也结构化，就定义一个类：姓名、小传、性格，定义什么，模型就填什么。

> **结论**：结构化输出两步走。第一步，定义一个 `BaseModel` 子类，把要输出的结构定下来；第二步，用 `with_structured_output(schema=那个类)` 把约束绑到模型上，拿到新模型再调用。这样一来，输出一定是你指定的那个类型的对象。

### Agent 的阻塞调用，参数是个字典

模型讲完，轮到 agent。

模型调用和 agent 调用不一样。模型初始化完就能直接调；agent 除了模型，还得有工具，把模型和工具一起交给它，才能调。步骤多几步，调用方式也更复杂一点。

流程还是那五步：加载环境变量、初始化模型、定义工具、创建 agent、调用 agent。

```python
from langchain.agents import create_agent
from langchain_core.messages import HumanMessage

agent = create_agent(model=model, tools=[])
```

`model` 和 `tools` 位置传、关键字传都行，关键字传看着更清楚。工具这一格先给个空的也能跑——这套架子先搭通，工具后面再往里加。

agent 准备好之后，第一种调用方式就是 `invoke`：

```python
response = agent.invoke({"messages": [HumanMessage("你好，你是谁")]})
print(response)
```

这里的差别必须记住，它是模型调用和 agent 调用的分水岭。

> **易错点**：模型调用传的是字符串，或者消息列表；agent 调用必须传字典。字典里必须有一个 `messages` 键，值是一个列表，列表里装消息——`HumanMessage`、`AIMessage`、`SystemMessage` 都能装。这个参数传错了，代码直接废。

返回的也是字典，字典里也有 `messages`。

| 对比项 | 模型调用 | Agent 调用 |
| :--- | :--- | :--- |
| 传入参数 | 字符串，或消息列表 | 字典，键为 `messages` |
| 返回结果 | `AIMessage` 对象 | 字典，内含 `messages` |
| 前置准备 | 初始化模型即可 | 还要备工具、建 agent |

打印出来，`response` 是一大坨字典，各种字段都在，不好看。只想看 AI 说的那句话，就这么取：

```python
print(response["messages"][-1].content)
```

`messages` 是消息列表，`[-1]` 取最后一条，也就是这次 AI 的回答，再取它的 `content`。

在 notebook 里，`response` 一旦生成就一直留在内存里，取内容不用把模型再跑一遍——变量还在，接着用就行，这点很舒服。

这种"发出去、等到底、一次拿全"的调用叫阻塞调用：调用发起后，线程被占住，模型把结果全部生成完才返回，等的这段时间什么也干不了。

> **提示**：严格说它叫阻塞调用，强调的正是"占住线程、等全部结果"这个特征。口语里常混着叫同步调用，但跟流式调用对比时，用"阻塞"这个词才准确。

### 流式调用：从事件流里捞 token

阻塞调用的毛病是等。模型把结果全部生成完才返回，句子越长等得越久。

流式调用不等全部：AI 吐出第一个 token，就立刻输出；后面每吐一个，就接着输出一个。你访问 DeepSeek、访问豆包的时候，消息都不是一次性砸出来的，是一个字一个字往外蹦——那就是 token。

> **提示**：token 不一定是一个完整的单词。实测里 `deepseek` 就被拆成两段吐出来。所以流式拿到的是片段，别把它当词看。

先说版本这件事。LangChain 升级很快，文档天天在变，调用方式也跟着天天变。现在最新的是 V3 模式，往前还有 V1、V2。V1、V2 的写法很变态，我们直接学 V3。

V3 里流式调用用的方法是 `stream_events`。`stream` 是"流"，小溪小河那种流动；`events` 是因为它把 AI 返回数据这件事看成一个一个的"事件"。合起来：用流动的方式处理消息返回的事件。

要注意，`stream_events` 不会直接给你结果——它是流式的，结果要持续地来，所以它返回的是一个 `GraphRunStream` 对象，也就是一条流。东西得从流里自己取。

流里不止一种事件：`values` 是 agent 的快照，`subgraphs` 是子 agent，`extensions` 是自定义事件，`output` 是最终结果。这些我们都不关心，只关心 `messages`——消息事件流。

取值是一层套一层的：

```text
agent.stream_events({"messages": [...]}, version="v3")
        │
        ▼
   GraphRunStream            此时还没有答案，只有一条流
        │  只取 .messages
        ▼
   message                   一个消息事件
        │  它的 .text 是一个 ChatModelStream
        ▼
   遍历 .text
        │
        ▼
   chunk                     一个片段，也就是一个 token
        │
        ▼
   print(chunk, end="", flush=True)    终端上逐字出现
```

落到代码上就三行主干：

```python
stream = agent.stream_events(
    {"messages": [HumanMessage("你好，你是谁")]},
    version="v3",
)

for message in stream.messages:
    for chunk in message.text:
        print(chunk, end="", flush=True)
```

第一个参数跟阻塞调用一模一样：字典，`messages` 列表，列表里装消息。不管阻塞还是流式，agent 的调用一律传字典，这一点没变。

第二个参数 `version="v3"` 是流式调用多出来的。必须给——它还有 V1、V2，你不指定 V3，它就按 V1 处理你。

输出这块有个小坑：`print` 默认是带换行的，直接 `print(chunk)` 会变成一行一个片段，终端上竖着往下掉。所以要 `end=""` 把换行去掉，让片段连成一句。

`flush=True` 是另一件事：让它立即输出。不加的话，输出可能先进缓冲区、攒一会儿才刷出来，看起来会一阵一阵地跳，甚至乱序。写上这一句，才是真正的"边生成边显示"。

跑起来看：第一个 token 大概几百毫秒就出现了，然后持续不断地往外吐，直到整段说完。同样的内容走阻塞调用，那次花了五秒。这就是差距。

运行时会冒一条警告，说 V3 的流式协议还处于实验阶段。实验期归实验期，不影响我们用。

把两种调用摆在一起，差别就这几处：

| 对比项 | 阻塞调用 | 流式调用 |
| :--- | :--- | :--- |
| 方法 | `invoke` | `stream_events` |
| 参数 | 一个字典 | 字典 + `version="v3"` |
| 返回 | 字典，内含 `messages` | `GraphRunStream` 事件流 |
| 取结果 | `response["messages"][-1].content` | 遍历 `.messages`，再遍历 `.text` 拿 chunk |
| 体感 | 五秒后一次性出现 | 几百毫秒开始逐字出现 |

### 系统提示词该放在哪

系统提示词这件事，大家以前就知道：要给模型一个系统提示词，就是在消息列表里加一条 `SystemMessage`。

```python
stream = agent.stream_events(
    {
        "messages": [
            SystemMessage("你现在扮演火箭队的武藏，用武藏的口吻回答问题"),
            HumanMessage("你好，你是谁"),
        ]
    },
    version="v3",
)
```

试一下就知道它生效不生效。第一遍问"你好，你是谁"，回来的话沾了点边，但武藏那句招牌台词没出来；再补一句"用武藏的口吻回答"，语气就对了。

这个办法是有效的，但它有个缺点：每次调用都得把系统提示词塞一遍。系统提示词本来就是个固定的东西，跟这一次问什么没关系，每调一次传一次，确实烦。

agent 给了个简化的办法：创建的时候就把系统提示词交出去，参数叫 `system_prompt`。

```python
agent = create_agent(
    model=model,
    tools=[],
    system_prompt="你现在扮演火箭队的武藏，用武藏的口吻回答问题",
)
```

系统提示词是固定的，那就提前绑在 agent 身上。绑完之后，调用时 `messages` 里只放 `HumanMessage` 就行，系统提示词自动生效——而且不管跟它对话多少次，每次都在。

重跑一遍，武藏那套台词整段出来了：既然你诚心诚意地发问了，我就大发慈悲地告诉你；为了维护世界和平，为了防止世界被破坏……语气到位，说明绑定生效了。

> **结论**：系统提示词是固定的，就别每次往消息列表里塞。定义 agent 的时候用 `system_prompt` 传一次，绑在 agent 身上；之后 `messages` 里只放对话内容，它自动生效，对话多少轮都算数。

> **承前启后**：上一节讲完「消息、结构化输出与 Agent 的调用方式」，下一节接着讲「工具描述的三种方式与 MCP 的 stdio 接入」。

---

## 工具描述的三种方式与 MCP 的 stdio 接入
> 对应块：BLK21 | 覆盖分集：P80-P82 | 块标题（分集名）：《工具描述方式与 MCP stdio 接入》

agent 初始化、接模型、添工具、流式调用，这一路我们已经跑通了。但那个"工具"的定义，一直是简化版——LangChain 里工具的定义还有不少东西没讲。接下来就把这块补齐。

agent 里最核心的两个部分：一个是模型，一个是工具。模型这一路已经全部讲完，现在就轮到工具。

### 工具是给模型用的函数

工具简单说就是一个函数。但这个函数不是给我们用的，是给模型用的。所以每一次调模型，我们都必须把工具的信息一并告诉它——哪些信息？三样：

- 工具的名字；
- 工具的作用描述；
- 工具需要的参数。

对着 DeepSeek 的官方文档看就很清楚：请求里的 `tools` 要带上 `name`、`description`、`parameters`。

> **提示**：模型看不到你的函数体，它只能看到你递给它的这段"说明书"。说明书里没写的事，在模型眼里就等于不存在。

问题是，我们之前有没有自己动手传过这三样？没有。我们只是加一个 `@tool` 装饰器，再写一段文档注释，就完事了。因为 LangChain 会自己把这些信息取出来：函数名、参数、文档注释。这三样凑在一起，已经足以描述一个工具。

不过 `@tool` 取信息的途径不止这一条。它一共有三种。

### 三种描述方式，先摆在一起看

| 描述方式 | 名字从哪来 | 作用描述从哪来 | 参数描述从哪来 | 适合的场景 |
| :--- | :--- | :--- | :--- | :--- |
| 装饰器属性 | `@tool("square_root")` 里传的字符串 | `description="..."` | `args_schema=...` | 参数少、一句话就能说清 |
| 文档注释 | 函数名 | docstring 的第一段 | docstring 的 `Args:` 段 | 绝大多数情况 |
| BaseModel | 函数名 | docstring 的第一段 | `args_schema=` 指向一个 BaseModel 类 | 参数多，而且取值有约束 |

我们之前一直在用的都是第二种。第一种和第三种没讲过，下面挨个演示。

演示代码直接复制之前那段流式调用的骨架，只改工具那一块。整个流程就那么五步：加载环境变量、初始化模型、定义工具、把工具绑给模型创建 agent、调用 agent。工具以外的部分一个字都不用动，这样才能看清"描述方式"到底改了什么。

### 用装饰器属性描述工具

先把快速入门里那个 `square_root` 拿过来——就是算平方根的那个函数。定义工具就是定义一个函数，关键是让它带上 `@tool`。

现在做个对照实验：把文档注释删掉。

不写 docstring，函数名再起得随便一点，比如叫 `tool1`。那么这个工具虽然加了装饰器，可这些信息还够描述它本身吗？不够。这时候把它交给 agent 去调，是要出问题的——这玩意儿明明是算平方根的，你问"帮我计算 429 的平方根"，它算不出来，因为它根本不知道你这个工具是干什么的。

跑一下就知道，它甚至会直接报错：说你这个 function 里没有 doc string，没有文档描述。

所以结论很硬：定义一个工具，你必须把信息告诉它；不告诉，就没法用。

那怎么告诉？第一种方案，把信息写进装饰器的属性里：

```python
from langchain_core.tools import tool

@tool("square_root", description="计算数字 x 的平方根")
def square_root(x: float):
    return x ** 0.5
```

这里只有两个位置在描述工具。一个是名字——`@tool` 后面直接传的那个字符串就是工具名，不用写成 `name=...`，直接传字符串即可。另一个是 `description`，写工具的作用。参数只有一个，`args_schema` 可以先不加。

起名的讲究在这儿：你叫 `tool1`，模型看到这个名字能知道你是干啥的吗？不知道。可要是叫 `square_root` 就不一样了——square root 这个单词什么意思？平方根。名字本身就很专业，模型一看就明白这个工具是算平方根的了。再配上 `description="计算数字 x 的平方根"`，名字和作用就都讲清楚了。

跑一下：问"帮我计算 429 的平方根"，AI 说可以，我来帮你算，然后返回一个工具调用——要调的工具名是 `square_root`，参数是 `429`，最后告诉我"429 的平方根约为 20.7123"。这不是算出来了吗？说明第一种描述方式成立。

### 用文档注释描述工具

第二种就是一直在用的：`@tool` 里什么属性都不加，函数写完之后直接跟一段文档注释，在注释里描述。

docstring 里能写工具的作用，也能写每个参数的作用——注释可以换行，可以写得很详细。`Args:` 段落里一个参数占一行，这是以前写 Python 注释的老习惯，只是很多人只写过不带参数的那种。

拿天气这个函数来说：

```python
@tool
def get_weather(location: str, units: str, include_forecast: bool) -> str:
    """获取当前天气和可选的天气预报。

    Args:
        location: 城市名称或者坐标
        units: 温度单位
        include_forecast: 是否包含天气预报
    """
    # 函数体是写死的示意实现
```

这个函数的参数有点多：`location`、`units`、`include_forecast`，三个。要描述，就得把每个参数都说清楚，而这正好是文档注释擅长的。

翻译一下这三行：`location` 是城市名称或者坐标；`units` 是温度的单位——中国用摄氏度，国外用华氏度，所以这里必须写清楚单位；`include_forecast` 就是是否要做天气预报。

实现逻辑很简单，全是写死的：单位是摄氏度就返回 22，是华氏度就返回 72（22℃ 大约对应的就是 72℉，真算的话有公式，这里就写死）；天气也是写死的，打印一句"当前天气阴，某地址多少度"；要带天气预报，就补一句"接下来五天都是晴天"。

为什么要用这么个写死的函数？因为它足够复杂，有三个参数要描述。函数一旦复杂，你想用第一种方式在装饰器里塞属性，就不好塞了——你不如像文档注释这样，直接写个名字、写参数、每个参数什么作用，很自然。

这种方式比第一种好在哪？好在自然。往装饰器里加属性本来就是件麻烦事；写文档注释是顺手的事，既给函数加了注释，又实现了描述函数的效果。函数再复杂一点、参数有四五个，装饰器属性里就更没法描述了。

实测一下：问"杭州天气，接下来几天的天气如何"。我特意强调"接下来几天"，就是想让它不光查天气，还要查预测。AI 返回消息时做了工具调用，调的是 `get_weather`，参数可就不止一个了——`location` 传了，`include_forecast` 也传了 `true`。这是 AI 自己决定的，不是我们让它传的。工具返回"杭州 22 度，接下来五天都是晴天"，AI 最后回复"当天 22 度，接下来五天都是晴天"。成功了。

所以参数一多，文档注释明显更方便。这也是我推荐的方式。

### 用 BaseModel 约束参数

第三种方式是用 BaseModel 来描述参数。什么时候轮到它？参数不光多，而且还有约束的时候。

BaseModel 本来就能做校验，所以它既能接收参数，又能校验参数。缺点是很繁琐：得先定义一个类，再往里传。所以这一种属于了解即可，演示一遍就够。

定义一个类 `WeatherInput`，继承 `BaseModel`，三个属性就是刚才那三个参数，每个属性后面跟一个 `Field` 写描述：

```python
from typing import Literal
from pydantic import BaseModel, Field

class WeatherInput(BaseModel):
    location: str = Field(description="城市的名字或者坐标")
    units: Literal["celsius", "fahrenheit"] = Field(description="温度的单位")
    include_forecast: bool = Field(description="是否包含天气预报")
```

跟第二种比，差别在 `units` 的类型上：它不是 `str`，而是 `Literal["celsius", "fahrenheit"]`。

`Literal` 译作"字面量"，你也可以把它理解成"常量"。意思是这个值只能在这几个里二选一，不能是第三种。为什么？因为温度单位总共就两种情况，要么摄氏度，要么华氏度。

不加以限制会出什么事？我们的代码里是 `if unit == "celsius"` 这样判断的，用的是英文单词。AI 完全有可能传一个中文的"摄氏度"进来——有没有这个可能性？有。那这个判断还成立吗？不成立。人家传的是摄氏度，你回了 72，结果就是错的。

所以像单位这种取值固定的参数，必须把可选值钉死。`Literal` 写进类型里，AI 看到这段描述之后就知道了：我只能传这两个中的一个。这样它将来传的时候就不会传错，百分之百传的是这两个里的一个。

> **提示**：约束不只是"让模型更听话"。它在代码侧还多了一层校验——真传了别的值，Pydantic 会直接报错，而不是让你的 `if` 悄悄走进错误分支。

用法是把类挂到装饰器上：`@tool(args_schema=WeatherInput)`，这个类就是"参数约束"。此时文档注释不用再写那么一堆了，只写一句简单介绍就行，参数的约束交给 `WeatherInput` 去做。

有人会说：这也太繁琐了，我根本写不出来。写不出来就别用这种，用上面文档注释那种就可以了。

那要是想用文档注释、又想约束温度单位呢？把解释写长一点就行——"温度单位，可选值只能是摄氏度或华氏度"。这么写行不行？怎么不行，用文字把可选值告诉它，一样有效。

所以理论上 BaseModel 能做的事，文档注释也能做，区别只是文档注释是"说给它听"，BaseModel 是"说给它听 + 传错就报错"。第三种方式了解一下，以后在别人代码里看到能看懂就行。

实测：把 `get_weather` 注掉，换成带 `args_schema` 的版本，问同样的问题——也能查天气，回答"当前天气 22 度，未来五天都是晴天"。

### 三种方式怎么选，以及参数名的两个禁区

收一下：定义工具，需要把工具的名字、作用的描述、需要的参数都告诉模型。告诉它的方式有三种——

- 基于 `@tool` 装饰器的属性；
- 基于函数的文档注释；
- 基于 BaseModel 作为 schema。

推荐第二种：跟以前一样写个 `@tool`，接一堆参数，再配一段文档注释。剩下两种知道一下就行，万一以后看到了，你能看懂它在干什么。

还有一条必须记住的约定：自定义工具的**参数名**并不是真的随便起。有两个名字不能用，一个叫 `config`，一个叫 `runtime`。

因为这两个参数名 LangChain 内部要用，是框架预留的注入位。除了这两个，其他参数名你随便叫。

> **注意**：函数参数一旦叫 `config` 或 `runtime`，LangChain 会把它当成框架自己的注入位置，传进来的是运行时对象，而不是模型给的值，工具的行为就不受你控制了。

### 工具开发不完，全世界还在重复开发

工具会定义了，那是不是将来就能开发各种各样的 agent 了？给 agent 添的工具越多，agent 的能力就越强——那就尽情开发工具吧。

道理上讲是行得通的。可你想想：工具你自己开发，你开发得完吗？开发不完。要用的工具太多了，查天气、查时间、查地图、查酒店、查航班，五花八门。

再问一句：这些需求只有你需要吗？今天我做 A 业务需要，明天做 B 业务也需要；全世界各地的人都在开发 agent，那大家的 agent 必然会有一些重复的能力需求。这些工具是不是每个人都得重复定义一遍？这是不是一种浪费？太浪费了，浪费人力物力。

于是就有了 MCP。

### MCP 是暴露工具、获取工具的一套标准

MCP 全称 Model Context Protocol，模型上下文协议，是 Anthropic 推出的一种开放标准。这标准是人家提出来的，心里有点别扭也没办法——毕竟标准已经在那儿了，全球范围内一直在用，该用还得用。

> **定义**：MCP 是一套让 AI 应用便捷连接外部系统的协议。所谓"连接外部系统"，就是调工具。

协议是它提的，那这个协议到底干什么用？我强调一遍：**MCP 的作用是把工具暴露出去**，暴露给全世界，让全世界的人都能拿到你的工具。

这样一来，那些通用的操作——文件操作、Web 搜索查询、航班、天气——就可以由专业的一方提前定义好：

- 查航班？航空公司最该提供这个 MCP 服务。任何人需要查航班，不用自己再写工具，去拿航司提供的就行；
- 查天气？天气预报这件事往往是国家在做，那气象局对外提供一个这样的工具，需要天气的人直接调气象局这个工具就可以了。

思路能明白吧？但紧接着就出现一个问题：所有公司都要对外暴露自己的工具，往哪儿暴露？别人想拿他的工具，又怎么拿？

这就是 MCP 的第二层意义：它同时是"暴露工具"和"获取工具"的一套标准。你想对外提供工具，就遵循 MCP 协议；全世界的工具都遵循同一个协议，那我们调用这些工具的方式也就统一了。

拿电脑打比方。你的电脑想接一个鼠标、一个键盘、一个摄像头，如果每个厂家生产的连接线都不一样，那你这台电脑就没法开口了——开 A 厂家的口，B 厂家的设备就插不进来。所以全世界定了一个统一的协议叫 USB，要求所有外设厂家都按 USB 做接口，电脑厂家也只做 USB 口。这样你买到的任何外设，一插就能用。

MCP 在 AI 世界里扮演的就是 USB 这个角色：工具提供者遵循 MCP 协议对外暴露工具，相当于生产键盘鼠标的人按 USB 做产品；我们开发 AI 应用时基于 MCP 协议去对接，相当于电脑上只留 USB 口。

所以要注意，MCP 仅仅是一种协议——它规定怎么交互、怎么通信，具体工具不是它提供的。协议它提的，我们用就完了。

### host、client、server 三个角色

MCP 里有三个概念必须分清：MCP server、MCP client、MCP host。

**MCP server** 是提供 MCP 服务的应用。它可以是远程服务，也可以是本地服务。

**MCP client** 是用来连 MCP server 的。server 对外提供工具，你想拿到这些工具，就用 client 去连它，连上就能读到里面的工具信息。

**MCP host** 可以理解成一个 AI 应用。AI 应用要用工具，所以它内部必须持有一个 MCP client——有了 client 才能连 server，连上 server 才能拿到 server 提供的工具。

三者一句话概括：一个提供工具，一个获取工具，一个使用工具。对照 USB 那套：host 是你的电脑，client 是那根连接线，server 是外部设备。电脑用连接线连外设，就是这个逻辑。

```text
MCP Host（我们的 AI 应用）
├── client-1 ──▶ MCP Server（天气）──▶ 工具：查当前天气、查预报
├── client-2 ──▶ MCP Server（航班）──▶ 工具：查航班、订航班
├── client-3 ──▶ MCP Server（地图）──▶ 工具：地点搜索、路径规划
└── client-4 ──▶ MCP Server（文件）──▶ 工具：读文件、写文件
```

读图说明：一个 host 里可以挂多个 client，每个 client 各连一个 server；工具从 server 拉到本地之后，就交给 host 里的 agent 调用。图里画几个 client 不重要，重要的是这条"host 持有 client、client 连 server、server 提供工具"的链路。

所以简单说：我们将来开发的 AI 应用就叫 MCP host，host 持有 client 去调 server，从而拿到 server 提供的工具。

### 两种通信方式：stdio 与 Streamable HTTP

client 要调 server 拿工具，中间必然有通信。MCP 的 client 与 server 之间有两种通信方式：

| 方式 | 全称 | 底层 | 工具在哪 | 每次调工具 |
| :--- | :--- | :--- | :--- | :--- |
| stdio | 标准输入输出 | 本地进程的标准输入输出 | 先下载到本地，作为一个进程跑起来 | 不需要网络 |
| Streamable HTTP | 流式 HTTP | HTTP 请求 / 响应 | 留在远端，不落地 | 每次都要发一次 HTTP 请求 |

Streamable HTTP 的本质就是 HTTP：你发一个请求，对面给你一个响应。拿工具也是发请求——比如高德地图暴露了一个地址搜索的工具，我要调它，就得向它发 HTTP 请求。

具体用哪种，不用你操心，取决于 server 提供了哪种。对方只提供 stdio，你就只能用 stdio；对方提供 Streamable HTTP，你就走 HTTP。

两者的差别值得说清楚：

- **stdio 是本地通信方式**。client 从 server 端把工具"下载"到本地——工具不就是函数吗，相当于把对面写的那个函数下载下来，在本地作为一个进程跑起来，然后调它。所以 stdio 模式一开始要做一次下载，会有一次网络交互；下载完，工具就在本地了，AI 应用再调工具还需要网络吗？不需要了，直接调本地的这个工具就行。一句话：先下载，再使用。
- **Streamable HTTP 是远程调用方式**。它不下载工具，而是每一次调工具都发一个 HTTP 请求过去。

一个本地，一个远端。这个差别了解一下就行——真用起来，server 那边是本地还是远端，人家已经写好了，你只管用。

### 面试官问起 MCP，怎么答

这套概念平时不用你管，但面试官要是问"你知不知道 MCP？MCP 是干什么的？"，你得答得出来。

第一，我知道 MCP；第二，MCP 是一个协议，用来对外暴露常见的工具，这样我们开发 AI 应用时可以直接通过 MCP 协议获取这些工具来用，不用再自己重复开发。

再收一句更完整的：MCP 是 AI 应用与工具之间对接的一套标准，遵循这套协议，全世界的服务就可以共享工具。

> **结论**：MCP 就是一套用来获取工具、调用工具的协议和标准。它不是工具本身，也不是某个具体服务。

### 去魔搭挑一个现成的 MCP 服务

动真格之前得先有服务。企业里常见的 MCP 服务，很多公司都已经提供了：

- 高德地图：酒店搜索、地图搜索、路径规划，跟地图有关的它全都能做；
- file system：专门操作文件的文件系统 MCP；
- 时间 MCP：专门查时间；
- 航班 MCP：查航班、订航班。

这些去哪找？魔搭社区 ModelScope。这个平台上有模型、有数据、有文档，还有 skills，也有 MCP。MCP 分类里的数量很吓人：浏览器相关的有五六百个，搜索相关的 1000 多个，交流协作 500 多个，开发工具 3000 多个，高德地图的也在里面。

我们演示一两个就够，不演示太多。选哪个？

模型是基于训练数据来回答的，而训练数据往往比较落后——DeepSeek 的训练数据截止到 2025 年的某个月份。所以你问它最新的东西，它不知道；你问它现在几点，它也不知道。所以第一个就演示时间 MCP：让模型查时间，这是非常常见的需求。

在魔搭里搜"时间"，出来一堆，一个两个三个四个五个。挑哪个？看访问量和调用量：

- 第一个：访问量只有 1500，调用量为 0；
- 第三个：访问量 52K；
- 还有一个访问量上百万、调用量 1 亿次的，很强——可惜用不了。前两天试过，不是充值的问题，是这个服务由外网提供，国内访问不了，所以大家别试了；
- 最后用的是那个国产的：调用量四百多万，也算常用。

它提供的工具挺全：查询当前时间、获取相对时间、获取时间戳、获取某个月的天数、时间转换、获取一年中的第几周。

### 看懂那段 JSON 配置

点开 stdio 模式，给出来的是一段 JSON 配置：

```json
{
  "mcpServers": {
    "time-mcp": {
      "command": "npx",
      "args": ["-y", "time-mcp"]
    }
  }
}
```

这段配置在告诉你什么？

最外层是一个字典，里面有一个键 `mcpServers`——注意带 s，说明这里可以配置很多很多的服务。里面现在只放了一个服务，服务的名字叫 `time-mcp`。这个服务的内容里又有两个键：`command` 是指令，`args` 是参数。

所以这个 MCP 服务将来运行时，执行的就是一条命令：`npx -y time-mcp`。

`npx` 是 Node.js 提供的一个下载管理器，作用就是把这个包拉到本地再跑起来。stdio 模式"把工具下载到本地"这一步，就是 npx 干的。

我们要做的事，无非是按这个结构配一份出来。

### 在 LangChain 里用 MultiServerMCPClient 拉工具

在 LangChain 里用一个 MCP 服务，先要装 MCP 的适配器 `langchain-mcp-adapters`。这个东西你就当成 MCP 的客户端：MCP 要有 client 和 server，装了适配器就等于装了客户端，然后我们用它去连 server——魔搭那个平台，可以理解成 server 端。

> **提示**：项目依赖里已经带了 `langchain-mcp-adapters`，跟着项目走就不用再装。

但 LangChain 里的写法跟魔搭给的 JSON 不完全一样。魔搭给的是服务端的配置，LangChain 里要创建的是客户端 `MultiServerMCPClient`，它接收一个字典——这个字典就是魔搭配置里 `mcpServers` 后面那一坨，一一对应。

| 魔搭给的 JSON | 传给 `MultiServerMCPClient` | 说明 |
| :--- | :--- | :--- |
| `"mcpServers": {...}` | 花括号里的内容直接当参数 | 外面那层壳去掉 |
| `"time-mcp"` | `"time-mcp"` | 服务名照抄 |
| `"command": "npx"` | `"command": "npx"` | 照抄 |
| `"args": ["-y", "time-mcp"]` | `"args": ["-y", "time-mcp"]` | 照抄 |
| 没有这一项 | `"transport": "stdio"` | 多出来的一个键，指定交互方式 |

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient({
    "time-mcp": {
        "command": "npx",
        "args": ["-y", "time-mcp"],
        "transport": "stdio",
    }
})

tools = await client.get_tools()
```

逐段看：创建客户端时把字典传进去，服务名 `time-mcp` 跟魔搭上的一致；复制过来之后多补一个 `transport`，填 `stdio`，把交互方式钉死；`get_tools()` 就是"获取工具"，返回的直接就是工具，不用再自己定义。

也就是说，以前是手动写工具，现在只要准备一个 MCP 客户端，让客户端去拉工具，拉回来的就是工具了。

第一次跑必然报错：

```text
TypeError: 'coroutine' object is not iterable
```

协程对象不可迭代。这个错该认识——它说明 `get_tools()` 是个异步函数。MCP 是异步的，调异步函数就得加 `await`。

> **提示**：以后再看到报错里出现 `coroutine` 字样，第一反应就应该是"这是个协程，要 `await`"。

加了 `await` 再跑，还是报错。这次不是代码写错了，是 notebook 环境的问题：在单个 notebook 里运行，异步事件循环已经在跑着了，会撞车。笔记里给了解决方案——在文件最上面先加一段处理，把已有的循环让出来，然后重启 kernel 再跑。

加完之后还有一条警告，说某个函数过期了。那是警告不是错误，不管它。

跑通以后把工具打印出来，拿到六个，跟网页上列的一一对得上：

```text
current_time     获取当前时间
relative_time    获取相对时间
days_in_month    获取某个月的天数
get_timestamp    获取时间戳
convert_time     时间转换
get_week_year    获取一年中的第几周
```

每个工具都带着名字、描述和参数。这些信息不是给我们看的，是给模型看的，所以不用管它长什么样。

### 同步的 invoke 调不了异步工具

工具拿到了，直接扔给 agent，就能用了吗？问一句"现在是什么时间"，报错：

```text
StructuredTool does not support sync invocation
```

当前这个工具不支持同步环境——因为它是个异步工具。MCP 是异步的，MCP 拿到的工具自然也是异步的；而我们现在的调用方式是同步调用，两者对不上。

那怎么改成异步调用？加 `await` 不行。`await` 是等待一个异步函数执行完，它没办法把一个同步函数变成异步函数。

正解是换方法名：`invoke` 是同步调用，`ainvoke` 就是异步调用。点进源码能看到 `ainvoke` 带着 `async` 标识，`invoke` 没有。

```python
result = await agent.ainvoke(输入)   # invoke → ainvoke，并且前面必须有 await
```

`ainvoke` 本身是异步函数，所以必须加 `await`，不然又要报那个 coroutine 的错。

改完再跑，AI 回答："现在是 2026 年 8 月 11 号 05:54:22。"对了。中间它肯定调了 `current_time` 那个工具。

### 整段代码唯一变的地方

回头看这一整段代码，跟前面的代码相比，唯一的差别在哪？工具。

以前工具是手写的，现在不手写了，从 MCP 拉取。怎么拉？三步：先定义 MCP 客户端，再用 `get_tools()` 拉取工具，拉到工具之后交给 agent 使用。

把这些临时调试的代码删掉，合二为一，跟以前相比还是那五步：

```text
① 加载环境变量
② 初始化模型
③ 定义工具        ← 唯一变化：以前手写，现在 MultiServerMCPClient + get_tools() 拉取
④ 创建 agent，绑定模型和工具
⑤ 调用 agent      ← 工具是异步的，要用 ainvoke + await，不能再用 invoke
```

读图说明：骨架一步没变，换掉的只是第③步的工具来源；第⑤步的调用方式是被工具的性质连带改的——异步工具只能用异步调用。

这就是 MCP 的接入方式：不用自己定义工具，准备一个客户端，把工具拉下来，交给 agent。工具可以来自本地进程（stdio），也可以来自远端服务（Streamable HTTP），对我们写业务代码的人来说，这两者的差别只剩配置里多一个 `transport`。

> **承前启后**：上一节讲完「工具描述的三种方式与 MCP 的 stdio 接入」，下一节接着讲「从 HTTP 接 MCP，到给 Agent 装上记忆」。

---

## 从 HTTP 接 MCP，到给 Agent 装上记忆
> 对应块：BLK22 | 覆盖分集：P83-P86 | 块标题（分集名）：《MCP HTTP 模式与短期记忆》

### 从本地进程到网络请求：MCP 的两种接法

前面我们接 MCP，走的是 STDIO 这条路。它的做法是：通过一个命令，把工具对应的脚本下载到本地，再作为一个进程启动起来。也就是说，这时候你的这些工具其实是在本地跑着的，agent 和工具之间的交互，本质上是一次进程间通信。

至于这个工具内部有没有再去发网络请求，我们不管——它完全可以发，也可以不发，这不影响 MCP 这一层怎么约定。

现在讲第二种：HTTP。这时候我们去和 MCP 的 server 端交互，靠的是 HTTP 请求：发请求把工具拉过来，用工具也还是走这套请求。所以整条链路全是网络交互，没有本地进程这回事。

| 维度 | STDIO | HTTP |
| :--- | :--- | :--- |
| 工具在哪 | 下载到本地，作为进程启动 | 在远端的 MCP Server 上 |
| agent 与工具之间 | 进程间交互 | HTTP 请求 |
| 配置里靠什么定位 | 启动命令 | 服务地址 |
| `transport` 取值 | `stdio` | `http` |

```text
STDIO ：  agent ──进程间交互──► 本地脚本（工具进程）

HTTP  ：  agent ──HTTP 请求──► MCP Server（远端）──► 工具
```

读图：STDIO 的箭头落在本地那个进程上；HTTP 的箭头落在远端服务上，工具挂在服务后面，agent 根本不知道远端有几个进程。

#### 魔搭上白嫖一个 12306 的 MCP

这次演示用一个 12306 的 MCP 服务，是在魔搭社区上找到的。魔搭上的 MCP 服务很多，像这种能白嫖的也很多，大家可以自己去找。

连到 12306 那个页面以后，左侧就有一个免费版，24 小时有效。临时用一用完全没问题。我点连接的时候它让我登录，我就不登录了——不影响看它的配置长什么样。连接之后，旁边会出现一段信息，那就是这个 MCP 的信息，也就是我们等下要往代码里配的东西。

#### 配置只加一对键值，transport 要入乡随俗

这里有个地方一定要弄明白：MCP 的那个大字典，是从最外层的大括号到里面那一层大括号之间的部分，中间装的是「MCP 的名字：MCP 的配置」。

之前接时间 MCP 的时候，我们其实已经把外面那个大字典传进去了。所以现在要加一个新的 MCP，只需要加一个逗号，再来一对「名字 + 配置」，就完事了。换成 HTTP 的 MCP，配法跟时间 MCP 是一回事，只是配置里给的是地址而不是命令。

从魔搭拷过来的时候，配置里那个字段叫 `type`；到了 LangChain 里，它叫 `transport`。这个就入乡随俗——你用的是 LangChain，就按 LangChain 的规则来，把 `type` 改成 `transport`，其余不用动。

```python
mcp_config = {
    # ... 前面已经配好的时间 MCP ...
    "12306": {                              # 新加的：先补一个逗号，再写名字 + 配置
        "url": "<魔搭页面上复制的那串地址>",
        "transport": "http",                # 魔搭上叫 type，到 LangChain 改成 transport
    },
}
```

`transport` 就两种取值：`http` 走 URL 路径的方式，`stdio` 走命令的方式。把这个配好，这个 MCP 就能用了。

#### 一句问话串起两个 MCP

配完先问它一句：「帮我查一下下周四从北京到杭州的高铁。」

它会先想：下周四是什么时间？于是调时间 MCP 里的 `get_current_date`，查到当前是 2026 年 8 月 12 号；那下周四就是 8 月 20 号。接着再去查高铁，调 12306 这个 MCP 的工具。第一次会慢一点，等一下就好了。

查回来的东西很全：出发站、到达站、时间，商务座、一等座、二等座的价格都有。消息一条条往下翻，肯定能找到最终结果。

将来你要做一个旅行规划的 agent，这套东西接上去就实现了。

到这儿 MCP 的用法其实就一句话：去网上找到 MCP 的配置，复制过来，按 LangChain 的模式改造一下——以「MCP 名字 + 配置」的键值对形式贴进字典，再补上 `transport`（`http` 或 `stdio`）。就这两种方式。

#### pretty_print：把一长串消息打印得能看

返回的信息太多，不好看。LangChain 给了一个很顺手的方式。

`response` 是一个字典，字典里有个 `messages`。我们一直强调：agent 调用时传的是字典，返回的也是字典，里面都是消息。用 `response["messages"]` 就把这次交互的所有消息拿到了；`for` 循环一遍历，拿到的每条就是一个 message。

LangChain 的 message 自带一个非常好用的方法：`pretty_print()`。pretty 是漂亮，print 是打印，合起来就是以非常漂亮美观的形式打印这条消息。

```python
for message in response["messages"]:
    message.pretty_print()
```

打出来就能看清整个交互过程：

- 第一句是我问的「帮我查一下下周四从北京到杭州的高铁」；
- AI 说我要帮你查高铁票，北京到杭州、下周四，但首先让我看一看当前时间；
- 于是它调了 `get_current_date`——这显然是我们接进来的那个 MCP 里的工具；
- 查询结果是 2026 年 8 月 12 号；
- AI 说行，我知道了，当前时间是它，那下周四就是 8 月 20 号；
- 接下来查高铁，从北京到杭州，时间是 8 月 20 号；
- 拿到所有车票信息以后，它筛选了一遍返回给我们：车站、出发站、到达站、时间、价格。

### 我们的 Agent 现在还没有记忆

那么到这里，我们的 agent 有记忆吗？没有。

道理大家都清楚：记忆靠的是消息拼接。我们现在还没拼过消息，所以现在的 agent 是没有记忆的。

接下来就讲 agent 的记忆。agent 也就是所谓的智能体，它的记忆分成两类：一类叫短期记忆（short-term memory），一类叫长期记忆（long-term memory）。

### 短期和长期，差的不是存多久

大家不要被字面意思误导。我相信大多数人第一次看到这两个词，都会以为短期记忆就是临时记忆、断电就没了，长期记忆就是永久记忆、持久保存。第一次看到会有这种想法很正常，直观上确实像。但对 agent 来讲不是这样。

对 agent 来说：

- 短期记忆是当前任务或会话的上下文；
- 长期记忆是跨任务、跨会话的经验或知识。

换言之，短期记忆针对某一个任务或某一个会话，长期记忆是跨越任务和会话的。所以它们的区分不在于你是临时存储还是永久存储，而在于生命周期——短期记忆的生命周期，就是某一次任务或某一个会话。

> **结论**：短期与长期的分界是生命周期，不是「临时存还是永久存」。

会话是什么？我们之前做保险项目的时候就有会话管理：用户登录成功以后，左侧有一个会话列表，每一个会话里面可以有很多很多条消息——这就叫一个会话。所以短期记忆指的就是这一个会话中的记忆，也就是这个会话里所有的历史消息。我们平常讲的「会话记忆」，就是短期记忆。

反过来，跨会话的记忆就是长期记忆。经常用 GPT 的同学会发现：你不管创建多少次新会话，它都能记住你的身份。我以前跟它聊过，说我是做 IT 培训的讲师，它以后就一直记得我是 IT 培训的讲师。这就是跨任务、跨会话的经验和知识。它会在跟我聊天的过程中不断收集我的个人信息，最后汇总成一个人物画像，慢慢地这个 agent 就变成了我的专属 agent：他很了解我、很懂我，我跟他讲任何东西，他能从我的角度去思考。比如我让他写一个项目，他会从教学的角度去想这个项目该怎么写。这就是长期记忆。

不过国内目前的这些模型，比如说 DeepSeek，目前还没有实现这种长期记忆的能力。

这里要再强调一遍：记忆是模型本身的能力吗？不是。记忆一直以来都是靠外部功能实现的。短期记忆靠的是拼接历史消息；长期记忆要靠某种编码的方式——在与用户对话的过程中不断收集用户信息，把这些信息长期保存起来，形成所谓的用户偏好，这就形成长期记忆了。这些都要靠代码实现。前面我们说 `Agent = model + harness`，那个 harness 里面就包含了长期记忆、短期记忆这些东西。

但咱们现在做 agent，其实不用考虑长期记忆，只考虑短期就够了。

### 会话记忆在 LangChain 里叫 agent state

短期记忆就是当前会话的记忆。我们之前理解的会话记忆主要是会话历史，其实它还包括一些别的东西，比如任务的中间状态、执行的一些结果等等，这些都可以算短期记忆。但目前为止先不管这些，先做会话历史的记忆；等学了 LangGraph，再讲任务状态那种短期记忆。

会话历史，大家做 AI 伴侣的时候是不是全靠人工维护？每次和 AI 对话，要手动把消息存到数据库；每次发请求，还得把它查回来手动拼接。麻烦不麻烦？

在 LangChain 里就不用你做了，它实现了全自动化的记忆管理。怎么实现的？

首先要理解一个概念：在 LangChain 中，会话的短期记忆都被称为 agent 的 state——状态。也就是说，所谓的会话记忆、短期记忆，在 LangChain 里被认为是 agent 的一种状态，所以它会有一个叫 agent state 的东西。你的记忆其实就存在这个 agent state 里边。

这个逻辑能懂吧：以前你是手动存，现在 LangChain 自动存，存哪儿了？存到 agent state 里。这里边默认存的就是消息，也就是一个 messages 的列表：用户发的消息、AI 返回的消息，全部都在这个 state 里边。

### 每聊一轮，就拍一张快照

历史消息都存在 agent state 里，但 agent state 仅仅是一个对象。这个对象里存了消息，那它将来要不要持久化？要吧。不持久化，一停机是不是就没了。

所以它要持久化。怎么持久化？

我们每与 AI 交互一次，消息会存进 agent state，同时给当前的 state 生成一个快照。第一次发一条消息、AI 返回一条，当前 state 发生变更，生成一个快照；我又发一条、AI 又回一条，状态又变更，再生成一个快照；再发，再生一个。也就是说，每一次与 AI 交互，都会产生一个新的快照。所有的快照合在一起，就是完整的会话历史。

```text
第 1 轮：  state 变更 ──► 快照①
第 2 轮：  state 变更 ──► 快照②
第 3 轮：  state 变更 ──► 快照③
                            │
                            └──► 三张快照连起来 = 完整的会话历史
```

读图：每一轮交互都在 state 上留下一次变更，变更就落一张快照；历史不是单独存的一份，而是这些快照攒出来的。

如果只是为了查历史消息，其实只要最后一个快照就够了——最后一个快照理论上已经包含了所有历史消息。那为什么还要保存每一次？

因为在 DeepSeek 这类聊天软件里，我们可以和 AI 多次交互。比如我觉得有一次写得不对，可以从某个位置「重新写」、重新发送，那之前的这些对话就没了，相当于从历史对话的某一个位置重新开始。这叫什么？时光回溯——能回退到之前任意一次对话。

靠什么实现？就是靠快照。每一次对话都有快照，我想回到第一次，只要找到第一个快照，从那儿重新发消息就行了。所以，只做会话历史，存最后一次就够；要做时光回溯，所有快照都得留着。LangChain 实现得比较完整，把每一次快照都记下来了。

### 快照交给 Checkpointer 管

快照有了，往哪儿存？所有快照会存到一个东西里，叫 Checkpointer。换句话说，所有这些快照都由 Checkpointer 来管理。

Checkpointer 就是一个用来管理快照的对象，它有很多种不同的实现方案：

| 实现 | 快照存在哪 | 性质 |
| :--- | :--- | :--- |
| `InMemorySaver` | 进程内存 | 停机就没了 |
| `SqliteSaver` | SQLite 数据库 | 落盘保存 |
| `PostgresSaver` | PostgreSQL 数据库 | 落盘保存 |

注意 SQLite 是一种数据库；PostgreSQL 我们前几天一直在用，增删改查用的就是它。

第一种是基于内存，后两种都是基于数据库，相当于一种永久存储。这种永久存储需不需要你自己写 SQL？不需要，人家已经自动帮你实现了。

到这里两个概念就清楚了：agent state 用来存、管理会话消息，所有会话历史都在 state 里；state 会形成一个一个的快照；快照交给 Checkpointer 存到数据库里。将来要取某一个快照，找谁？找 Checkpointer——它管快照，取快照当然得找它。

### thread_id：一次会话一个身份

还有一件事没完。同一个人跟 agent 聊：一上来先建一个会话，聊一段时间；又新建一个会话，再聊。这就有很多很多会话了，而这些会话是不同的会话，应该有不同的记忆。

我们前面画的那种多轮交互——用户发 A、AI 回 B，用户发 C、AI 回 D——其实是一个会话里的多轮交互，它们算一套记忆。新建一个会话，就得从头开始，也要记一套新的。

不同会话之间怎么区分？不然记忆不就乱套了。为了区分不同会话的记忆，我们有一个概念：thread_id。thread 就是会话的意思，所以 thread_id 就是会话的唯一标识。

每发起一个会话，都会有一个新的 thread_id；当前会话下的所有聊天，都由这一个 id 来标识，它们同属于同一个 thread_id。将来要根据某一个会话来查记忆，就拿这个 id 去查。

LangChain 在帮我们存一个会话的所有消息时，就是以 thread_id 作为 key，来存它的所有快照。每个会话都有自己的 id、自己的快照。点击某一个会话，它就拿着这个会话对应的 id 去查对应的快照；点第二个会话也是如此。

```text
thread_id = "A"  ──►  快照① → 快照② → 快照③     （会话 A 的历史）
thread_id = "B"  ──►  快照① → 快照②              （会话 B 的历史）
                          │
                          ▼
                  Checkpointer 统一管理
```

读图：同一个 thread_id 下面挂的是这个会话自己的快照串，不同 thread_id 各走各的；底层由 Checkpointer 统一存取，所以查记忆靠的是 id。

这一点刚好跟我们之前的业务相符。我们有一张会话表，叫 `chat_thread`，它刚好有一个 id，这个 id 将来就作为 thread_id 来用。所以我们在页面里添加一个会话、创建一个 id，把这个 id 交给 LangChain，它就基于这个 id 来管理里面的快照。

### 动手：三步把记忆接上

概念讲完了，得动手。下面以内存版的会话记忆为例——现在没必要上数据库版，先用内存版演示。

#### 先导入 InMemorySaver：包在 langgraph 里

要用记忆，就得让 Checkpointer 来管快照，所以第一步是导入一个 checkpointer。

```python
from langgraph.checkpoint.memory import InMemorySaver
```

这就是刚才说的三种实现之一：基于内存版本的 Checkpointer。

需要注意的是它导的包：不是 LangChain 的包，而是 LangGraph 的 `langgraph.checkpoint.memory`。为什么不是从 LangChain 导？因为 LangChain 底层是基于 LangGraph 实现的，而这个记忆其实是 LangGraph 做的。

#### create_agent 多传一个 checkpointer

接下来创建 agent。老步骤还记得吧：加载环境变量、初始化模型、定义工具、创建 agent 绑定模型和工具，最后调用。今天在绑定这一步多绑一个东西：checkpointer。

```python
from dotenv import load_dotenv

load_dotenv()

model = init_chat_model(
    "deepseek-v4-flash",
    extra_body={"thinking": {"type": "disabled"}},
)

agent = create_agent(
    model=model,
    tools=[],                       # 这次没有工具，给空
    checkpointer=InMemorySaver(),   # 今天新加的第三个东西
)
```

环境变量不记得没关系，完全可以复制粘贴。模型这里传两个参数：第一个是模型名字，第二个是自定义参数，我们一般自定义 `extra_body`，里面把 `thinking` 的 `type` 设成 `disabled`。工具这次没有，给空就行。

`create_agent` 以前只接收两个东西：模型和工具；今天接入第三个——`checkpointer`，用它来实现会话记忆。加完这一行，这个 agent 就自带记忆功能了，你不需要做任何数据库的增删改查，它就记住了。

#### 调用时带上 config

有记忆之后怎么发请求？要注意：会话记忆需要一个 thread_id 来标识，所以每次发起请求都必须指定对应的 thread_id。这个 id 是你自己写的，叫什么都行，但一定要确保它是唯一的——每一个会话一个。

写法是固定格式：

```python
config = {"configurable": {"thread_id": "1"}}
```

`configurable` 这个属性名、以及里面 `thread_id` 这个键，都不能变；可变的只有 id 的值。每创建一个新会话就换一个新 id，同一个会话始终共享同一个 id。

然后就可以调用了。以前调 `invoke`，传进去的是字典，返回的也是字典；今天除了消息，还得再传一个 `config`。用 `invoke` 行，用 `stream` 也行，只是 `stream` 写起来稍微繁琐一点：

```python
stream = agent.stream_events(
    {"messages": [HumanMessage(content="12个苹果分给4个人，每人分几个？")]},
    config=config,
    version="v3",
)

for message in stream.messages:
    for chunk in message.text:
        print(chunk, end="", flush=True)
```

三个参数：第一个是消息字典（字典里 `messages` 装用户消息），第二个是带 thread_id 的 `config`，第三个是版本号 `version="v3"`。拿到的 `stream` 用两层 `for` 循环遍历：外层拿 message，内层拿这条消息里的文字片段，`print(..., end="", flush=True)` 一边生成一边打出来。把 config 这样传进 agent，从此以后你向它发起的一切请求都是有记忆的。

> **易错点**：`stream` 里拿到的 `chunk` 是消息片段，不是一条完整的 message，不能对它调 `pretty_print()`，正常 `print` 就行。

所以要把记忆接上，其实就三件事：导入一个 checkpointer；创建 agent 时绑定 checkpointer；发请求时传一个带 thread_id 的 config。

### 跑起来：12 个苹果分给 4 个人

第一次先问一个简单的：「12 个苹果分给 4 个人，每人分几个？」它答出来了，能聊。

再问一次，同一个 `config`、同一个 thread_id，问：「如果是 3 个人呢？」它直接接着上一轮那 12 个苹果往下算——这一问里我们并没有再提 12 个苹果。说明记忆生效了。

到这儿，会话记忆就实现了。

回顾一下这套东西怎么落地的：短期记忆就是会话历史，存在 agent state 里的 messages 中；每交互一次生成一个快照，同一个会话的所有快照共享同一个 thread_id，方便按 id 找回这段历史；所有快照交给 Checkpointer 管理，内存、SQLite、PostgreSQL 都行。落到代码上，就是刚才那三步。

代码不用重写一遍：把之前那个 agent 复制过来，给它添上记忆就可以了。

> **承前启后**：上一节讲完「从 HTTP 接 MCP，到给 Agent 装上记忆」，下一节接着讲「会话记忆搬进 PostgreSQL」。

---

## 会话记忆搬进 PostgreSQL
> 对应块：BLK23 | 覆盖分集：P87-P90 | 块标题（分集名）：《PostgreSQL 持久化记忆与对话模块》

保险顾问的 agent 还没动手写，我打算先给它配一个 checkpointer。顺序听着有点倒，理由却很实在：记忆是地基，地基先打好，agent 建起来直接插上去就行。

之前那套 `InMemorySaver` 在 demo 里够用，进程一重启，聊过的内容全清零。放到真实项目里，这等于没有记忆。所以要换成一个能长期存下去的实现——基于 PostgreSQL 的 checkpointer。除了这个实现方式，其他地方一个都不动。

### 换掉 InMemorySaver，只动两个地方

先别急着敲代码，先问自己一句：拿一份已经跑通的 in-memory 版本，把 checkpointer 换成 PostgreSQL，将来哪个地方会变？

导的包会变，checkpointer 的构造会变。其他的变不变？不变。

这个判断很值钱。它意味着后面那一整套「创建 agent → 绑定 checkpointer → 调用时传 config」的流程，一行都不用改。底座换了，接口没换。

换实现就得有依赖，名字叫 `langgraph-checkpoint-postgres`。这个不用自己装，项目依赖里已经带了。同时还会牵出另一位：`psycopg[binary]`。后面这个名字得单独说清楚，因为它是整段里最容易含混的地方。

#### 驱动不同，连接池不能复用

我们之前用 PostgreSQL，主要做增删改查，走的是 SQLAlchemy。SQLAlchemy 底层连库用的是 `asyncpg` 这个驱动。

当时那个连接串是按「协议 + 驱动」的格式写的：`://` 前面既有数据库类型，也有驱动名。

```text
postgresql+asyncpg://user:pass@host:5432/db
^^^^^^^^^^^^^^^^^^ 协议 + 驱动
```

今天用的不是 SQLAlchemy，是 LangGraph 的 checkpointer。底层同样是 PostgreSQL，但它走的是另一条路，驱动换成了 `psycopg` 的异步版本。而且它的 URL 里不让你写驱动名：

```text
postgresql://user:pass@host:5432/db
^^^^^^^^^^ 只有协议，驱动藏在库内部
```

同样的数据库名、用户名、密码、地址，前半段的写法却不一样。两个东西连的是同一个 PostgreSQL，但底层驱动不同、连接方式不同、URL 路径也不同。

结论可以直接推出来：SQLAlchemy 那一套连接池，在 checkpointer 这里用不了。得单独再配一套。

> **提示**：`://` 前面那一段有个正式名字叫 scheme（方案、协议），不是 schema。schema 是数据库里的另一个概念——库内的命名空间，默认叫 public。这两个词长得像，含义毫不相干，配置里差一个字母就连不上。

#### 用一套配置拼出两条 URL

数据库地址原来配在 `DatabaseSettings` 里：主机名、端口、用户名、密码，再加一堆连接池参数，最后拼出一个 URL。当时那个 URL 里的 scheme 是手写死的，因为只有一个消费者。

现在有两个消费者，除了 scheme 之外其余完全一样。那就把拼 URL 的动作抽出来，scheme 当参数传进去。

```python
class DatabaseSettings(BaseSettings):
    host: str
    port: int
    user: str
    password: str
    database: str

    def build_url(self, scheme: str) -> str:
        return (
            f"{scheme}://{self.user}:{self.password}"
            f"@{self.host}:{self.port}/{self.database}"
        )

    @property
    def url(self) -> str:
        return self.build_url("postgresql+asyncpg")

    @property
    def checkpointer_url(self) -> str:
        return self.build_url("postgresql")
```

一个 `url` 给 SQLAlchemy 用，一个 `checkpointer_url` 给 checkpointer 用。两者前半段不同，后半段完全一致，一套配置产出两条路径，不用维护两份密码和端口。

> **结论**：以后凡是遇到「同一个数据库、两套客户端」的情况，先比较它们的 URL 差在哪里，通常只差 scheme 这一段，抽一个参数化函数就够了，别复制两份配置。

### 连接池这段代码是死的，参数得读懂

in-memory 版本有多简单？`InMemorySaver()` 一个对象就完了。PostgreSQL 版本麻烦，因为它依赖连接池，构造要多几步。

这段代码将来跟 `database.py` 那份一样，属于「死代码」——照抄就行。但照抄和读懂是两回事，参数还是得知道在干什么。

```python
from psycopg.rows import dict_row
from psycopg_pool import AsyncConnectionPool
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

settings = get_settings()

checkpointer_pool = AsyncConnectionPool(
    conninfo=settings.DB.checkpointer_url,
    min_size=1,
    max_size=5,
    kwargs={
        "autocommit": True,
        "prepare_threshold": 0,
        "row_factory": dict_row,
    },
    open=False,
)

checkpointer = AsyncPostgresSaver(checkpointer_pool)
await checkpointer.setup()
```

逐条过一遍。

`conninfo` 就是 URL 路径，读的是刚配好的 `checkpointer_url`——专供 checkpointer 的那一条，不是业务库那条。

`min_size=1` 和 `max_size=5`：池子一创建默认先放 1 个连接，用完了允许再建，最多到 5 个。这跟 SQLAlchemy 里那两个参数是同一个道理——那边配的是池大小和池满之后允许的额外连接数，这边直接给下限和上限。

会不会太小？这个池子只服务 checkpointer 一个消费者，存记忆的频率并不高，1 和 5 足够。不放心可以调成 3 和 10，也就差不多了。连接池一般都很小，因为建立连接的速度非常快，没必要开一个大池子常年占着。

`autocommit=True`：自动提交事务，以后不用自己管事务。

`prepare_threshold=0`：不做预备，也就是不缓存预处理语句。

`row_factory=dict_row`：查询结果以字典形式返回。

这三项不是随手写的，是 LangChain 要求的固定写法——要用连接池就必须这么写，官方给的示例就是这个形状，别自己改。

`open=False`：创建池对象的那一刻不去建连接，池子是空的。什么时候建？后面显式 `open()` 的时候。所以有两步配套动作：

```python
await checkpointer_pool.open()    # 初始化连接
await checkpointer_pool.wait()    # 等池子里的连接就绪
```

> **易错点**：`open=False` 和这两步是一整套。只 `open()` 不 `wait()`，第一次查询很可能撞上一个还没准备好的空池；报错的位置离真正的原因很远，很难查。

### AsyncPostgresSaver 与自动建表

对象从哪来，看 import 就明白了。之前是从 `langgraph.checkpoint.memory` 导 `InMemorySaver`，这一把从 `langgraph.checkpoint.postgres.aio` 导 `AsyncPostgresSaver`。

名字里那个 `aio` 就是异步。它只认异步调用，同步的接口全部用不了，这一点后面还得再改一次代码。

但两者都是 checkpointer：一个内存实现、一个 PostgreSQL 实现，接口一致。所以创建对象这一步依然是「把东西塞进去」——把连接池交给它就行。跟内存版相比多出来的唯一动作，就是前面那个连接池的初始化。

还有一步不能漏。内存版不需要建表，反正全在内存里；连到真数据库就必须有表。于是：

```python
await checkpointer.setup()
```

这行跑完，表就自动建好了。打开数据库刷新一下，会多出四张表：`checkpoints`、`checkpoint_blobs`、`checkpoint_writes`、`checkpoint_migrations`。会话历史将来全存在这里，建表语句一句都不用我们写。

> **定义**：`setup()` 是 checkpointer 的建表入口，负责把 LangGraph 需要的那套表结构补齐。它只补缺、不覆盖已有数据，所以放在初始化流程里重复执行是安全的。

### 跑通一次多轮对话，看数据落在哪

先写个 demo 验证。复制一份到 `day06` 目录，新建 notebook，名字就叫「测试 PostgreSQL 的 checkpointer」。

初始化那一段换成刚写的连接池加 saver，后面接回原来的流程：初始化模型、创建 agent、把 checkpointer 绑上去。

第一次跑会撞上一个错，错在异步：`AsyncPostgresSaver` 只允许异步调用，而原来的代码是同步的。改法是把 `stream` 换成 `astream_events`，`for` 换成 `async for`，该 `await` 的地方补上 `await`，整套调用全部改成异步。

说实话，早知道要全改异步，我就不在 demo 里用流式调用了，省得来回折腾。

> **提示**：这不是 PostgreSQL 版特有的坑，凡是 `aio` 这个包都这样。换成异步以后，所有拿返回值的地方都要 `await`，漏一个拿到的是协程对象而不是结果。

然后测记忆。先把对话内容分给四个人、每人说三句，再分一次，接着换成三个人再来一轮，问一句「有没有记忆」——有。打开数据库看：`checkpoints` 表里躺着好几条记录，这些就是快照。

真正的消息正文在 `checkpoint_blobs` 里，`message` 字段存的是加密过的内容。

> **注意**：看到那一列乱码不用慌，它是加密后的消息，本来就不是给人读的，你肯定看不懂，设计如此。

顺便纠正一个直觉上的误解：四轮对话留下来的不是「四条消息」，而是四个状态快照。每一步的状态存一条，消息挂在对应状态下面。

到这儿，基于 PostgreSQL 的会话记忆就跑通了。整体流程跟在内存里那套没有任何区别：创建 agent、绑一个 checkpointer、调用时传 config。唯一的变化是 checkpointer 的初始化变复杂了——它依赖一个连接池，所以要先建池，再把池交给 `AsyncPostgresSaver`。

### 让 FastAPI 管 Checkpointer 的生死

新问题来了：这段初始化逻辑写在哪？

它是个连接池，跟业务库那个连接池性质一样，而连接池应该在项目启动时创建、项目关闭时停止。所以照 `database.py` 的规矩办：放进基础设施层，定义初始化和关闭两个方法，交给 FastAPI 的生命周期管理。

#### 基础设施层的新文件

在 `app/infrastructure` 下新建 `checkpointer.py`，把刚才验证过的代码搬进去，加上日志。

```python
import logging

from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver
from psycopg.rows import dict_row
from psycopg_pool import AsyncConnectionPool

from app.core.config import get_settings

logger = logging.getLogger(__name__)
settings = get_settings()

checkpointer_pool = AsyncConnectionPool(
    conninfo=settings.DB.checkpointer_url,
    min_size=1,
    max_size=5,
    kwargs={"autocommit": True, "prepare_threshold": 0, "row_factory": dict_row},
    open=False,
)

checkpointer = AsyncPostgresSaver(checkpointer_pool)


async def init_checkpointer() -> AsyncPostgresSaver:
    await checkpointer_pool.open()
    await checkpointer_pool.wait()
    await checkpointer.setup()
    logger.info("Checkpointer 初始化成功")
    return checkpointer


async def close_checkpointer() -> None:
    await checkpointer_pool.close()
    logger.info("Checkpointer 连接池已关闭")
```

模块导入时池子对象就建好了，但 `open=False` 意味着它是空的、没有连任何东西，所以 import 这个模块不会去打数据库。真正的连接要等 `init_checkpointer` 执行，这也是把 open/wait 单独抽成一个函数的原因之一。

关闭只需要一步：关掉连接池，`close_checkpointer` 就是这一件事。

这跟 `database.py` 的关系是并列的：写法一样、套路一样，但服务对象不同——那份给业务 ORM 用，这份给等会儿的 agent 用。

#### 接进 lifespan

两个函数什么时候执行？项目启动时执行初始化，项目关闭前执行关闭。落到代码上，就是 `main.py` 里那个 lifespan：`yield` 之前的在启动时跑，`yield` 之后的在关闭时跑。

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    logger.info("安心宝应用启动中")
    await init_db()
    checkpointer = await init_checkpointer()
    agent = await init_insurance_agent(checkpointer)

    yield

    logger.info("应用关闭中")
    await close_db()
    await close_checkpointer()
```

顺序不是随手排的：agent 必须拿到 checkpointer 才能建，所以 checkpointer 先就绪；关闭时反过来，先关业务库，再关 checkpointer 的池子。

```text
启动
  ├─ init_db()                业务库连接池（SQLAlchemy / asyncpg）
  ├─ init_checkpointer()      checkpointer 连接池（psycopg）
  │    ├─ open()   建连接
  │    ├─ wait()   等就绪
  │    └─ setup()  补表
  └─ init_insurance_agent()   保险顾问 agent（绑定上面的 checkpointer）

运行
  └─ 请求 → agent → 记忆自动读写

关闭
  ├─ close_db()
  └─ close_checkpointer()
```

这张图就是整个应用三件事的先后：数据库连接池、checkpointer 连接池、agent。谁依赖谁，启动和关闭正好反着走一遍。

### Windows 上的事件循环报错

改完再启动，日志先报业务库初始化成功，紧接着初始化 checkpointer 的时候炸了。大意是：这个 event loop 没法在异步模式下使用，请换一个兼容的事件循环，具体报的是 `RuntimeError` 加上一句 `Please use a compatible event loop`。

一堆英文看着乱七八糟，原因其实不复杂：连接池底层靠异步事件驱动，而 Windows 上默认的事件循环类型不对。`psycopg` 的异步连接需要事件循环支持 IO 事件的注册与监听，Windows 默认那种循环恰好不提供这套接口，于是连接一建就报「请换一个兼容的事件循环」。代码没错，是循环选错了。

解法就一行，写在启动入口里。

```python
import sys

loop = "asyncio" if sys.platform == "win32" else "auto"
```

这一行把平台判断和取值合在了一起：`sys.platform` 是 Python 内置的判断操作系统的属性，等于 `win32` 就说明在 Windows 上，此时给 `"asyncio"`；其他平台用默认的 `"auto"`。

它等价于这么写：

```python
if sys.platform == "win32":
    loop = "asyncio"
else:
    loop = "auto"
```

> **提示**：三元表达式拆成多行时，要用一对圆括号把整个表达式包起来，告诉 Python 这是一行；不包会被当成三条独立语句，直接语法报错。

然后把这个值塞进 uvicorn 的启动参数。原来的启动调用配了 `app` 路径、`host`、`port`、`reload`，现在多一个 `loop`，其余不动。

再启动一次：项目启动、业务库初始化、checkpointer 初始化，一路没有报错。停项目的时候，日志依次是「应用关闭中」「连接池关闭」「checkpointer 关闭」，顺序也对。

### 建一个 agents 包

checkpointer 备好了，接下来初始化 agent。

agent 放哪儿？不能塞进以前的业务包。它属于一个特殊功能，跟订单、商品那些业务不是一类东西，所以单开一个包，叫 `agents`。

用复数是故意的——将来这里不会只有一个智能体。先建第一个：`insurance_advisor.py`，保险顾问。以后跟 agent 相关的代码都往这个包里放，而且这次建的是 Python 文件，不是 notebook。

创建过程还是那几步：加载环境变量、初始化模型、初始化工具、创建 agent，把模型和工具绑上去。

以前绑两样，现在要记住绑三样：模型、工具、checkpointer。

#### 环境变量没加载，key 只好手动传

第一步在这个项目里要打个折。所有环境配置早就交给 Config 管理了，Config 在启动那一刻就把 `.env` 里的东西读进了 `settings`，API key、chat model 都能读到，所以没必要再加载一遍环境变量。

但不加载会带来一个连锁反应。`init_chat_model` 有两种初始化方式：默认初始化和自定义初始化。默认初始化时，只要指定了模型名，它就能推断出用的是哪家模型，然后自动去环境变量里找对应的 API key 和 base url。

问题就在这儿：`.env` 只是被读进了 `settings`，并没有写进进程的环境变量。于是默认初始化去环境变量里翻，什么也翻不到。

只能手动给了：

```python
model = init_chat_model(
    model=settings.LLM.chat_model,
    model_provider="deepseek",
    api_key=settings.LLM.api_key,
)
```

`settings.LLM` 里本来就有两项：`chat_model` 和 `api_key`，直接取。`model_provider` 写不写都行，不写它也能从模型名推断出来，写上更明确。

另外还要把之前配置里的 `extra_body` 一起搬过来，作用是关掉思考——思考太浪费时间，这个场景不需要。

#### Agent 全项目只初始化一次

工具这一步先跳过。现在还不知道保险顾问要挂什么工具，等想清楚了再加。

直接创建 agent：

```python
async def init_insurance_agent(checkpointer: AsyncPostgresSaver):
    model = init_chat_model(
        model=settings.LLM.chat_model,
        model_provider="deepseek",
        api_key=settings.LLM.api_key,
    )

    agent = create_agent(
        model=model,
        tools=[],
        checkpointer=checkpointer,
        system_prompt=SYSTEM_PROMPT,
    )

    logger.info("保险顾问 agent 初始化完毕")
    return agent
```

这里的 checkpointer 是参数传进来的，不是函数里自己 new 的。这一点必须讲清楚：checkpointer 已经在 main 的 lifespan 里初始化过了，那个才是全项目唯一的一份。如果在这个函数里再导一次包、再 new 一个，项目里就有两个 checkpointer 了，不合理。

整个项目有一个 agent 就配一个 checkpointer，够了。

`system_prompt` 决定这个 agent 的角色。没有它，`create_agent` 建出来的只是个通用的智能体，不知道自己是谁。提示词是这样的：

```text
你是安心宝的智能保险顾问。
你需要使用专业、准确、容易理解的语言回答用户的保险问题。
当信息不足时，先向用户追问，不要编造保险产品或保障的内容。
```

提示词写在 `create_agent` 的参数里，好处是它跟着 agent 走：以后不管对话多少轮，这个角色设定都在，不用每次请求重复传。

最后 `return agent`，函数就完整了。调用它的地方只有一个——lifespan：

```python
agent = await init_insurance_agent(checkpointer)
```

为什么放在这儿？因为要用 checkpointer，而 checkpointer 只在 lifespan 里拿得到，那就只能在这儿建。

> **结论**：同类型的 agent 全项目只初始化一次。谁来聊都是跟同一个 agent 聊，没必要每个请求建一个；只有确实需要不同类型的 agent 时，才建多个实例。

### 对话接口只有一条 POST

agent 有了，接下来通前后端。

打开保险顾问的页面，按 F12 看 Network，发一条消息，会看到一个请求：方法是 POST，路径是 `/api/v1/chat`，请求体里两个字段——`message` 和 `thread_id`。

- `message`：发给 AI 的消息内容；
- `thread_id`：会话 id，将来会直接作为 checkpointer 里的那个 `thread_id`。

这个对应关系要记住：前端说的「会话」，在 checkpointer 里就是 `thread_id`。同一个 `thread_id` 的多轮消息，会被自动串成一段连续的记忆。

响应返回什么？接口文档里对 AI 对话的响应有些要求，先不管，第一步就返回一个普通字符串——AI 回复的那段话。

后端要做的事很直白：接住请求，从请求体里取出 `message` 和 `thread_id`，拿这两个东西去调刚建好的 agent。

```text
浏览器
  │  POST /api/v1/chat  { thread_id, message }
  ▼
chat 模块（modules/chat）
  │  取出 message 与 thread_id
  ▼
保险顾问 agent（全局唯一）
  │  按 thread_id 读历史 → 调模型 → 追加本轮消息
  ▼
PostgreSQL（checkpoints / checkpoint_blobs）
```

记忆的读写全在 agent 那一侧完成，chat 模块不碰数据库，它只负责把请求翻译成一次 agent 调用。

### chat 模块：四件套砍成三件

对话属于业务，不能放进 `agents` 包——那个包是专门放智能体的，聊天本身是业务。所以要在 `modules` 下新建一个 `chat` 模块。

按以前的规矩，一个业务模块是五件套：`schemas`、`models`、`repository`、`service`、`router`。先定义请求和响应的约束，再写数据库模型和增删改查，然后是业务和接口。

但跟 AI 对话，有数据库操作吗？没有。所以 `models` 和 `repository` 直接去掉。

有人会追问：会话记忆不是数据库操作吗？是，但那个需要我们操作吗？不需要。它是 LangChain 内置的，人家自己去管会话——消息存储、消息拼接，全自动完成，我们一行 SQL 都不用写。

于是这个模块只剩三样：

| 文件 | 职责 | 是否保留 |
| :--- | :--- | :--- |
| `schemas.py` | 约束请求参数与响应结果 | 保留 |
| `models.py` | 数据库表映射 | 去掉 |
| `repository.py` | 增删改查 | 去掉 |
| `service.py` | 取 `message` 和 `thread_id`，调用 agent | 保留 |
| `router.py` | 暴露 `POST /api/v1/chat` | 保留 |

模块建好，骨架就有了。接下来往这三个文件里填内容，顺序还是老规矩：先 schema，再 service，最后 router。

> **承前启后**：上一节讲完「会话记忆搬进 PostgreSQL」，下一节接着讲「基础对话接口与 SSE 数据格式」。

---

## 基础对话接口与 SSE 数据格式
> 对应块：BLK24 | 覆盖分集：P91-P94

Agent 建好了，模块也有了，接下来就该动手写代码。

按顺序来，第一件事是 schema。这个接口的响应结果就是一个字符串，所以响应 schema 不用写，咱们只写请求 schema。请求里的东西也很简单，两个字段：一个是会话 id，一个是 message。

### 请求 schema 就两个字段

写在哪？就写在 `schema` 这个文件里，写个类，类名叫 `ChatRequest`——对话的请求嘛。我们之前定过规矩，所有的 schema 都继承 `BaseSchema`，所以从 `app.common.schema` 里把 `BaseSchema` 导进来，让它继承。

两个字段。第一个 `thread_id`，UUID 类型。用 `str` 行不行？也行，它本质就是个 UUID，只不过发到前端以后变成了字符串，前端提交过来再转回 UUID，就这一串。为什么要用 UUID——你去翻一下 `ChatThread` 那个 model，它的 `id` 就是 UUID 类型。将来数据库表里的会话是它，发到 LangChain 里的 id 也是它，只是前端把它叫成了 `thread_id`，一个东西两个名字。

第二个 `message`，字符串，就是用户发的那句话。

回看笔记，它还加了个约束：最小长度是 1。也就是说这个字段不能为空。那咱们也加上，从 `pydantic` 导 `Field`，用 `Field(min_length=1)` 约束。

```python
from uuid import UUID

from pydantic import Field

from app.common.schema import BaseSchema


class ChatRequest(BaseSchema):
    """基础对话的请求参数"""

    thread_id: UUID
    message: str = Field(..., min_length=1)
```

| 字段 | 类型 | 约束 | 说明 |
| :--- | :--- | :--- | :--- |
| `thread_id` | `UUID` | 无 | 会话 id，对应 `ChatThread.id`；本质是 UUID，到前端变成字符串 |
| `message` | `str` | `min_length=1` | 用户这一轮发的话，不能为空 |

到这里 schema 就搞定了。

### 对话逻辑写进 Service

下一步写对话逻辑，写到 service 里。沿用之前的套路：所有的 service 都先定义一个类，再把函数写到类当中。类名叫 `ChatService`。

#### 构造函数要 agent，不要 session

这个类里要写构造函数，接收它需要的东西。那我先问一句：这个 service 是来写对话的、是实现聊天的，那聊天需要什么？

当然需要 id，但那是写真正的对话的时候才用，现在只是在定义类本身。这个类当中所有的函数，将来一定会用到一样东西——不是 session，是 agent。

你对话不得用 agent 吗？刚才咱们已经把它定义出来了，前端发请求过来，你就是要调它，所以 service 里一定要用 agent。那就在构造函数里让别人把 agent 传给我：`self.agent = agent`。

那 agent 是什么类型？咱们好像从来没讲过。看它怎么来的——`create_agent` 创建的。那 `create_agent` 的返回值是什么类型？不知道怎么办？点进去看。按住 Ctrl 键，鼠标左键一点，就跳到源码里了。这源码看着头皮发麻，不要紧，咱只看这个函数的返回值。不管它是什么，反正是它，Ctrl+C 复制回来，类型咱不就知道了。

给它定义上，导一下包。同理，`main` 当中拿到的是这个类型，将来传给 service 时也是这个类型——同一条线上下来的，不会有问题。

#### chat 方法：先组织参数，再调用

方法名可以叫 `chat`，其实也可以叫 `chat_stream`，因为最终咱们希望用流式。但现在先不写流式，先写个阻塞式。

这个聊天函数需要知道什么？这次就需要刚才前端传过来的那两样了——一个 id，一个 message。干脆让前端直接把 `ChatRequest` 传进来，从当前包的 `schema` 里 `from .schema import ChatRequest`。

函数注释写清楚：实现 Agent 对话的一个函数。

要对话就调 `self.agent`，调什么？以前一直用 `invoke`，那是同步阻塞的。但现在咱们在 FastAPI 里，FastAPI 是异步环境，异步环境要用 `ainvoke`。用了 `ainvoke` 就必须 `await`，而函数就必须是 `async def`。

`ainvoke` 里要传两样：第一是消息 `input`，第二是配置 `config`。先传消息，再传 config，有了 config 才有会话记忆。

消息不能在这儿手写死，它在 `request` 里。所以要先把 `request.message` 取出来，封装成 `{"messages": [...]}` 这个形状，列表里是发给 AI 的消息，一个 `HumanMessage`，它的 `content` 就是 `request.message`。`HumanMessage` 导一下包。

config 不用另写，就用之前那套，只是 id 不能写死——id 从 `request.thread_id` 里取。

> **提示**：`input` 和 `config` 都是位置参数，按位置传就行，不必写成键值对；你想写成键值对也没问题，只是没必要。

调用完得到一个 response。agent 传进去是字典，返回的还是字典，我们要的是里边的 `messages`，再取最后一条，然后取这条消息的 `content`。把上面这些落下来，代码长这样：

```python
from uuid import UUID

from langchain_core.messages import HumanMessage
from sqlalchemy.ext.asyncio import AsyncSession

from app.module4.chat_thread.repository import ChatThreadRepository

from .schema import ChatRequest


class ChatService:
    def __init__(self, session: AsyncSession, agent) -> None:
        # agent 的类型就是 create_agent 的返回值，Ctrl 点进源码复制即可
        self.agent = agent
        self.repository = ChatThreadRepository(session)

    async def chat_stream(self, user_id: UUID, request: ChatRequest):
        """实现 Agent 对话的一个函数"""
        input = {"messages": [HumanMessage(content=request.message)]}
        config = {"configurable": {"thread_id": request.thread_id}}
        response = await self.agent.ainvoke(input, config)
        return response["messages"][-1].content
```

跟之前那套调用没有太大差别，只是挪到了 service 里边来写。

#### 少了最关键的一步：会话归属校验

但是，我要说但是了。

现在咱们这个系统是有用户概念的。你不能随便一个阿猫阿狗，带上一个 thread_id 就能往这儿发消息。因为每一个 thread_id 都属于一个特定的用户——刚才那两条会话，这个 thread_id 属于谁？属于「黑马虎哥」这个用户。

那如果有人看到了我的 id，然后往我这儿发消息，是不是就把我的会话污染了？

大家要知道，这个 id 是明文的。就像 DeepSeek 一样，你抓一下它的请求，`chat_session_id` 是明文的，后边跟着 `prompt`，就是你的 message。假如我们不做任何限制，有人看到了我的 id，模拟这个请求向我发过来，我的会话就被污染了。这合理吗？不合理。

所以要验证一下当前用户是谁：只有验证过这个 id 属于当前用户，才允许他发这条消息；如果不属于，不能让他往下走。

这个校验咱们做过没有？早就做过了，会话管理里一直在做。它不在 service 里，在 repository 里，有一个接口叫 `required_owner`：你把用户 id 给我、把会话 id 给我，我去查；查到了就属于你，查不到就不属于你，就是这个逻辑。

那我们要在 chat service 里调它，就需要拿到 repository。构造函数里创建 `ChatThreadRepository`，从 `app.module4.chat_thread.repository` 导进来。创建它需要 session，我没有，怎么办？让别人传给我——谁用我，谁就传给我，这就是依赖。

所以这个类需要两样东西：一个 `AsyncSession`，用来调 repository；一个 agent，用来对话。我自己不写增删改查，但我用别人的增删改查。

有了 repository，校验就简单了：调 `required_owner`，把 `user_id` 和 `request.thread_id` 传进去，拿到的结果就是 `ChatThread` 对象。`user_id` 我这没有，那将来谁调我谁就传一个给我，方法签名上加上它；会话 id 就在 `request.thread_id` 里。

如果结果为空，说明这个会话不属于当前用户，得报错、抛异常。咱们之前定义过一个自定义异常，语义正好是「当前会话不存在或不属于当前用户」，直接 `raise` 它，导一下包。

```python
from app.common.exception import ChatThreadNotFoundException

# 加在 chat_stream 方法的最前面
thread = await self.repository.required_owner(user_id, request.thread_id)
if not thread:
    raise ChatThreadNotFoundException()
```

这两行加在方法的最前面，后边才是组织消息、组织 config、调 agent。

顺序就定死了：先校验属不属于当前用户，属于了，再去组织消息、组织 config、调用 agent，最后返回结果。

service 写完了，但这时候还测不了——咱们只写了 service，缺对应的 router。

### Router 把接口挂出来

router 的写法咱们都知道，一直在说：参考接口文档。请求方式、请求路径、请求体，只要知道了这些，接口就能写。

#### 起手与接口签名

起手先来一个 router：

```python
router = APIRouter(prefix="/api/v1/chat", tags=["与AI的对话接口"])
```

路径文档里写的是 `/api/v1/chat`，那就照它写；`tags` 就是这个接口的归类——与 AI 的对话接口。

然后就是用它，`@router.post`，因为这是个 POST 请求。路径还传吗？不传了，上边的 `prefix` 已经写过了。响应类型也别写，咱们响应返回的就是个字符串。

函数就叫 `chat`，FastAPI 里当然用异步函数。参数就是刚才那两样，而它们已经定义成 schema 了，所以第一个参数是 `request: ChatRequest`，从 `.schema` 导进来。

到这里，请求方式、请求路径、请求参数全都有了，接口理论上就定义好了，接下来该写业务逻辑。而写业务逻辑就要调刚才那个 service。

#### session 靠依赖注入，agent 靠 app.state

创建 `ChatService` 需要两样东西：session 和 agent。

session 好办，依赖注入：参数声明成 `AsyncSession`，值用 `Depends(get_session)`，而 `get_session` 来自咱们之前写的 `app.infra.database`。

agent 是个难点。agent 是在 `main` 里定义的，我现在在 router 里要用，怎么用？用不了。这时候就要讲 FastAPI 的一个支持了。

在 FastAPI 当中，或者说在咱们的 `lifespan` 函数当中，我们是不是已经拿到了 `app`？`app` 代表的是整个 FastAPI 应用。这个时候 `app` 上有一个隐含的东西，叫 `state`。我们只需要给 `state` 添加一个属性就行：

```python
app.state.agent = agent
```

`state` 是个什么东西？你可以把它理解成一个字典，但它比字典更强大——它可以动态地往里边放东西。我们放进去的，就是刚刚创建的那个 agent。

> **注意**：`app.state` 是全局的 state，一切进入 FastAPI 的请求都共享它，所以它们共享的是同一个 agent。

那怎么从 state 里把 agent 取出来？

当我们拿到一个 FastAPI 的请求时，接口里可以接收一个 `Request` 类型的参数——变量名随便起，叫 `app_request` 也行。这个 `Request` 类型是 `fastapi` 里来的。它身上有个 `state`，也就是 `request.state`，就是我们刚说的 `app.state`——因为 state 是全局的，任何请求取出来的都是同一份。

一般不这么直接取，正规写法是先拿 app、再拿 state、再拿 agent：

```python
agent = request.app.state.agent
```

用一个变量接住，再往下传。

> **提示**：以后再写 FastAPI，想在请求之间共享一个全局变量，就把它放进 `app.state`；任何请求来了都能从 `request` 拿到 `app`，从而拿到 `state`，再拿到里边的变量。

把这条链路串起来看：

```text
main.py 的 lifespan
    agent = create_agent(...)
    app.state.agent = agent          ← 全局 state，所有请求共享
                 │
                 │ 任意请求进入 FastAPI
                 ▼
    router.chat(...)                 ← 需要 ChatService
                 │
                 ▼
    get_chat_service(request, session)
        request.app.state.agent ──┐
        session（Depends 注入）───┤
                 │                │
                 ▼                │
        ChatService(session, agent)
                 │
                 ▼
        service.chat_stream(user_id, request)
```

`main` 里存一次，之后每个请求都沿着 `request.app.state.agent` 这一条路径把它取回来，塞进 service。

写接口的时候，先把取 service 的逻辑直接摊在接口里，是这样的：

```python
@router.post("")
async def chat(
    request: ChatRequest,
    app_request: Request,
    user_id: UUID = Header(...),
    session: AsyncSession = Depends(get_session),
):
    agent = app_request.app.state.agent
    service = ChatService(session=session, agent=agent)
    return await service.chat_stream(user_id, request)
```

#### 把取 service 抽成依赖

以后如果我们写更多接口，它们是不是都依赖这个 service？那获取 service 的动作就该抽成一个函数：

```python
def get_chat_service(
    request: Request,
    session: AsyncSession = Depends(get_session),
) -> ChatService:
    agent = request.app.state.agent
    return ChatService(session=session, agent=agent)
```

代码原样搬过来，最后直接 `return` service 就行。它依赖 `request` 和 `session`：session 靠 `Depends(get_session)`，request 直接声明。

以后要 service，还用自己写这一堆吗？不用了，接口参数里直接注入：

```python
service: ChatService = Depends(get_chat_service)
```

将来 FastAPI 看到你需要 service，就来帮你调 `get_chat_service`；调它的时候发现它需要 request 和 session，就帮你把这两个注入进去，然后把 service 交给你。

依赖注入写一次不会，你就写两次；写两次不会，就写三次。写得多了，service 你就会写了，依赖注入也就会写了——咱们现在已经写过三四次了。

#### 挂载与实测

`user_id` 上哪整？以前讲过，从请求头里取，把之前的代码 Ctrl+C 过来就行。`UUID` 和 `Header` 导一下包。

启动之前，又忘了件事——挂载。每次写完都忘。在 `main` 里 `include_router(chat_router)`，现在一共三个：`/api/v1/chat`、`/api/v1/chat_threads`、`/api/v1/products`。

重启，回到文档刷新，接口出来了。测一下。

第一次故意传个假的 `thread_id`，走你——400，`当前会话不存在或者不属于当前用户`。校验是成功的，不该放行的就是不放行。

第二次换成正确的 `user_id` 和正确的会话 id，message 传个 `hi`——调上了，返回「你好，我是智能保险顾问安心宝，很高兴为你服务，请问今天你想了解什么保险问题」。

跑通了。

> **注意**：这时候不能在前端测，因为前端要的是 stream 模式，而咱们写的是阻塞式，返回的是一个完整字符串。

自己写这一遍的时候，就注意两点：第一，先找到上节课那个很长的 `main`，agent 已经拿到了但还没往 `state` 里存，现在存进去；第二，再写 router——先定 `get_chat_service`，需要 session 就传 session，需要 agent 就从 request 里取，然后在接口里注入 service，调完直接返回。

### 阻塞式的痛点与 SSE

对话虽然成功了，但你应该能发现一个问题：这个对话是阻塞式的，要等很久才能拿到结果。而我们最终期待的肯定是流式。

流式该怎么做？可能比大家想的会复杂一点。

以前咱们做流式对话，是在 notebook 里做测试的。在 notebook 里测试，你只需要打印就完了——AI 返回一个片段，你 print 一个片段，流式调用以前写过，就是打印输出。

但现在不是打印输出，是要跟前端交互：AI 返回一个片段，你往前端扔一个片段；而且是持续地扔，不是发一次。怎么才能持续地往前端发？这个咱们以前没学过。

要用到一个技术，叫 SSE。

SSE 的全称是 Server-Sent Events，服务端发送的事件。它是一种服务器主动向浏览器推送数据的协议：前端跟服务端一旦建立连接，这个连接就挂在这儿，服务端不是返回一次就结束，而是不断地向前端推送数据，这样就能实现流式输出的效果。

但要注意，SSE 底层依然是 HTTP，只是在 HTTP 的基础上做了一些特殊处理。

以前用 HTTP，响应的格式基本都是 `application/json`；用 SSE 就不能用这种格式了，要用 `text/event-stream`，也就是事件流的意思。

一旦你用了这种格式，前端就知道了，前端会用 `EventSource` 这一套 API 来接收。这个跟我们没关系——我们不写前端，只写后端。后端只要按这个格式响应，前端就能用这种特殊的方式接收，从而实现流式。

> **结论**：前端发起一次请求，服务端以流式不断向前推送，前端边收边渲染。

### SSE 的数据格式

那服务端该怎样返回 `text/event-stream` 格式的数据？它是有特殊要求的。

标准的 SSE 里，服务端返回的数据是纯文本，普通文本，但这个文本在格式上有要求：应该包含 `event`、`data`、`id`、`retry` 四个字段。不是要求必须全包含，至少要包含一个字段——`data`。

每个字段后边跟一个冒号和空格，并以换行结束；而最终消息要以两个换行结束：

```text
字段名: 字段值\n     ← 每个字段一行，行尾一个换行
字段名: 字段值\n
\n                  ← 空行，代表这一条消息结束
```

`
` 就是换行符。跟一个 `
` 仅仅代表我换了一行，跟两个 `
` 代表我这一条消息结束了。也就是说，一条消息里可以包含好几个「字段名: 字段值」，按这个形式去返回。

> **定义**：SSE 的一条消息 = 若干行 `字段名: 字段值
`，再加一个额外的 `
` 收尾。

最简单的形式，只有一个字段 `data`，值是「你好」，后边直接两个 `
`，这条纯数据消息就结束了：

```text
data: 你好

```

稍微复杂一点，带上事件类型：

```text
event: message
data: {"content": "你"}

```

这里的 event 类型是 `message`，`data` 是一个 JSON，两个 `
` 代表这条消息彻底结束了。

最后还有一种 event 类型叫 `done`。done 就是完成的意思：前边是一条一条的消息，到最后我告诉前端「消息完事了、结束了」，这个时候前端跟服务器的连接就会断开。所以 `done` 是最后一条：

```text
event: done
data: end

```

想更复杂，带上 id 之类的，也 OK。但通常来讲，最简单的就是只有 data，稍微复杂一点的就是带事件类型。

> **注意**：在咱们的项目里，建议大家都用带 event 的那种，不要发纯数据的。你得告诉前端什么时候结束，不然它不知道。

普通消息的 event 就叫 `message`，最后一条的 event 叫 `done`。前端拿到 `done`，就知道该断开连接了。

那这种格式怎么发出去？最笨的办法是手动拼接字符串：拼 `event: message`、拼换行、拼 `data:`、再拼内容，一点一点拼出 `

` 的结尾。麻烦不麻烦？太麻烦了，每次返回都要拼一遍。

SSE 的格式到这儿就清楚了，但要真按这个格式发，就非常繁琐了。别着急，这件事有现成的办法。

> **承前启后**：上一节讲完「基础对话接口与 SSE 数据格式」，下一节接着讲「让回答一个字一个字地长出来：SSE 流式对话与会话历史接口」。

---

## 让回答一个字一个字地长出来：SSE 流式对话与会话历史接口
> 对应块：BLK25 | 覆盖分集：P95-P97 | 块标题（分集名）：《SSE 流式对话与会话历史接口》

SSE 的数据格式我们已经弄明白了：`event:` 一行、`data:` 一行，末尾再来一个空行。格式是懂了，可要是每返回一句话都得自己动手把字符串拼成这个鬼样子，那也太麻烦了。列一列都嫌累，更别提后面还有 `id`、`retry` 这些根本用不上的字段。

那能不能不拼？

不用拼。FastAPI 里已经把这件事办好了，我们只要照着它的要求做两件事就可以。

### 就两件事：改响应类型，换返回对象

第一件事，把 router 的响应类型改掉。注意，这里改的**不是** `response_model`，而是 `response_class`——很特殊，就改这一个字母。改成一个叫 `EventSourceResponse` 的东西，顾名思义，事件来源式的响应，也就是基于事件流的响应。

第二件事，返回值不能再用普通字符串，要用 `ServerSentEvent` 这个类来封装。这不就是 SSE 的全称吗？Server-Sent Events，服务器发送事件。FastAPI 里专门为它准备了一个类。

为什么要多这一层？因为 `ServerSentEvent` 这个类内部自带了 `id`、`event`、`data`、`retry` 四个字段。它的意思很清楚：**拼结构不是我们的事，我们只负责造对象**。需要哪几个字段就传哪几个——比如只关心事件名和数据，那就传 `event` 和 `data`，剩下两个空着；将来把这个对象返回出去，FastAPI 会按照 SSE 的规范把它转成

```text
event: message
data: 你好

```

这种格式。转换这一步由框架完成，我们连换行符都不用操心。

> **提示**：`response_model` 和 `response_class` 只差一个词，性质完全不同。前者约束的是**响应体里的数据结构**，后者换掉的是**整个响应类型**。要做 SSE，认准 `response_class`。

### 先用假数据把一条流跑起来

测 FastAPI 当然得改 router，我们打开 chat 的 router，先做一次不改业务逻辑的验证。

```python
from fastapi.sse import EventSourceResponse, ServerSentEvent

@router.post("/chat/sse", response_class=EventSourceResponse)
async def chat_sse():
    for i in range(10):
        yield ServerSentEvent(event="message", data=f"message_{i}")

    yield ServerSentEvent(event="done", data="")
```

装饰器上写 `response_class=EventSourceResponse`，这是第一件事。函数体里不再 `return`，而是 `yield`，这是第二件事。

`return` 和 `yield` 的区别，正是流式与一次性返回的分水岭。`return` 交出去一次，函数就结束了，只能给一个结果；流式要的是不断产生、不断往前送，所以得用 `yield`。`yield` 的脾气我们之前学过：把一个东西临时交出去，当前函数就地暂停，等外面处理完了，再回到暂停的地方接着往下跑。用它一路产出，就得到一个"动态列表"——也就是生成器（generator）。

这里写了十次循环，就是想向前端返回十条消息。每循环一次交出一条，前端处理完，我们再回来转下一圈，这就模拟出了流式的效果。

数据先随便造：`data` 写 `message_0` 到 `message_9`，`event` 写 `message`。为什么是 `message`？前面讲格式的时候说过，普通消息的事件类型就是 `message`；整个对话结束的那一条，事件类型是 `done`。所以循环里全是 `message`，循环结束之后再补一条 `done`——注意这一条必须写在循环**外面**，它代表"没有了"。

至于 `done` 那条的 `data` 里写什么，已经无所谓了，写 `end` 写 `done` 都行，因为后面真的没有内容了，前端只认 `event` 这个类型。

重启服务，到接口测试工具里发一个请求，问它"讲个笑话吧"。第一次跑，状态码 200，但返回的东西不对，后台还报了错：`async for` 不能用在非异步的函数里。

原因很朴素：这个接口当时写的是普通 `def`，里面却写了 `async for`。把函数改成 `async def`，问题就没了。

> **提示**：`async for` 只对**异步可迭代对象**有意义。像 `range(10)` 这种同步序列，用普通 `for` 就够了；真正需要 `async for` 的，是 `astream_events` 的返回值、以及我们自己写的异步生成器这类"边产边取"的东西。判断依据不是"函数是不是 async"，而是"被遍历的对象能不能异步产出"。

改完再发一次，返回的内容就是我们要的样子：

```text
event: message
data: message_0

event: message
data: message_1

...

event: done
data:
```

一行事件名、一行数据、一个空行，一条接一条，和最初要求的格式完全对得上。只是它跑得太快，十条一瞬间就结束了，肉眼看不出"一条一条"的过程。这没关系——等接上真正的对话，就能看见字一个个往外蹦了。

### 把假数据换成真回答

router 里现在吐的是 `message_0` 这种假数据。真数据得从 AI 那里来，而调 AI 的代码在 service 层，所以要改的是 service。

原来的写法是阻塞式的：调一次，等模型把所有话都想完，再一次性把结果拿回来。现在要改成异步的流式调用。方法名换成 `astream_events`，前两个参数（输入和 config）不动，加一个 `version="v3"` 就够了。

它返回的是一个流对象。拿到流之后要遍历，而且是**两层遍历**：

```python
async def chat_stream(user_id: str, request: ChatRequest):
    # ……组装 agent、准备 config，略……

    stream = agent.astream_events(
        {"messages": [{"role": "user", "content": request.message}]},
        config=config,
        version="v3",
    )

    async for message in stream:
        async for text in message.text:
            yield ServerSentEvent(event="message", data=text)

    yield ServerSentEvent(event="done", data="")
```

外层一条一条拿消息，内层再从这条消息里把文本片段一段一段抠出来。每抠出一段，就把它塞进 `ServerSentEvent`：`event` 还是 `message`，`data` 换成这个片段，然后 `yield` 出去。

等所有循环都跑完，再补一条 `event="done"`。

写法和刚才那个假数据演示是同一套，只换了两处：数据从 `message_i` 变成了真的文本片段；`yield` 的位置从 router 挪到了 service。区别只在于，这一次吐出来的是真东西。

### router 只做一件事：把 yield 原样转出去

service 写完了，router 反而更简单——它没有真正的业务逻辑，只负责调 service，然后把 service 吐出来的东西原样吐给前端。

但这里不能只 `yield` 一次。service 那边 `yield` 了多少次，router 这边就得 `yield` 多少次，所以还得再套一层 `async for`，接到一个就转一个。

```python
@router.post("/chat/stream", response_class=EventSourceResponse)
async def chat_stream(user_id: str = Depends(get_current_user), request: ChatRequest = ...):
    async for event in chat_service.chat_stream(user_id, request):
        yield event
```

整条通路长这样：

```text
浏览器
  │  POST /chat/stream
  ▼
router.chat_stream
  │  async for event in chat_service.chat_stream(...)
  ▼
service.chat_stream
  │  agent.astream_events(..., version="v3")
  │  每抠出一个文本片段
  │  yield ServerSentEvent(event="message", data=片段)
  ▼
router 原样 yield 出去
  │
  ▼
EventSourceResponse 按 SSE 规范写给浏览器
  │  循环结束后 yield ServerSentEvent(event="done", data="")
  ▼
浏览器逐条收到 event: message
```

读图就一句话：这是一条 `yield` 链，service 从模型那里接片段，router 从 service 那里接事件，`EventSourceResponse` 再把事件按 SSE 格式写给浏览器。每一层都不改内容，只做传递。

这条链上埋着两个坑，刚才都踩过：接口函数必须是 `async def`；循环必须用 `async for`——因为 `chat_service.chat_stream(...)` 是异步生成器，不是普通列表。第一次是把接口写成普通 `def`、里面却用了 `async for`，语法直接过不去；第二次是外层改成异步了，里面那层循环却忘了加。同一个知识点，前后绊了两跤。

重启，刷新页面，新建一个会话来问，打字机的效果就出来了。以前是问一句，界面上咔咔咔转一两秒，然后"啪"地出现一整段；现在是第一个字几乎立刻就到，后面跟着一串。

我顺手让它讲个笑话。它说：身为保险顾问，讲笑话可能不太专业，但我可以试一试——为什么保险公司不喜欢教数学？因为客户听完复利计算后，都会问能不能给我一个保本版本的数学。一点都不好笑。但字确实是一个一个蹦出来的。

回头把这半段压缩成两句：把阻塞调用换成 `astream_events` 并带上 `version="v3"`；遍历消息片段，每拿到一个就用 `ServerSentEvent` 封一下 `yield` 出去，全部结束再 `yield` 一个 `done`。就这些。

### 刷新一下，对话就没了

对话能用，也不报错，但一刷新页面，历史全空。

不是没存。是我们只做了会话**列表**的查询，点开某一个列表项去看它里面的**内容**，这个接口没做。

所谓"会话历史"，其实两条线：一条查出有哪些会话，一条查出某个会话里聊过什么。前一条早就写完了，后一条空着，所以页面上点进去什么都没有。

先说清楚这些消息到底是谁存的。从头到尾，我们没有手动往数据库写过一条消息。我们做的是三件事：创建 checkpointer，把它交给 agent，请求时在 config 里带上 `thread_id`。之后每一轮对话，用户发一条、AI 回一条，LangChain 都自动把这两条写进 checkpointer。下次带着同一个 `thread_id` 再来，它就自己把历史捞出来，拼上最新这句，一起发给模型——所以这个 agent 天然就有记忆，我们一行存取代码都没写。

省事是省事了，麻烦也在这儿：消息不是我们存的，我们也就不知道怎么查。

有人会说，这还不简单，去数据库里找不就完了？checkpointer 对应的表就在那儿，消息不都在里面吗？问题是那些内容都是加密、序列化过的，你看不懂，我也看不懂。想查历史，只能回头找 LangChain 要方法。

### 读历史：只要最后一个快照

每一次和 AI 交互，都会形成一个 state 快照。你向 AI 发一条消息、AI 回一条，存一份快照；再来一轮，又存一份。一个会话就是这样一堆快照摞在一起，而 `thread_id` 是它们的共同标识——不同的 `thread_id` 代表不同的会话。

那查历史是不是要挨个快照翻？不用。**最后一份快照里已经包含了完整的历史消息**，取它一个就够。

顺序是这样的：先有 `thread_id`，凭它找到这个会话的所有快照，再取最后一个。但这一步不用我们自己动手，LangChain 提供了 `get_state`，传一个 config 进去，直接就把最后一个快照给你。

- `agent.get_state(config)`：拿最后一个快照；
- `agent.get_state_history(config)`：拿所有快照。

我们要的是前者。而且要用异步版本 `aget_state`，因为我们的 checkpointer 一直是异步的 Postgres。这个 `config` 必须和对话时用的是同一个——`thread_id` 对上了，才查得到对应的消息。

```python
snapshot = await agent.aget_state(config)
print(snapshot)
```

第一次跑忘了写 `await`，报错说协程没有被等待。看到 `coroutine` 这个词就该反应过来：异步函数忘了 `await`。补上就有了。

拿到的是一个 `StateSnapshot` 对象，snapshot 就是"快照"这个词。它里面有个属性叫 `values`，是个字典；字典里有个键叫 `messages`。所以取历史消息的路径是：

```python
messages = snapshot.values["messages"]

for message in messages:
    message.pretty_print()
```

打印出来，正好是之前那两轮：

```text
Human: 12 个苹果分给 4 个人，每人分几个？
AI: 这是个简单的平均分问题，每人分 3 个。
Human: 如果是 3 个人呢？
AI: 12 ÷ 3 = 4，每人分 4 个。
```

两条用户消息、两条 AI 消息，一共四条，全都回来了。

`pretty_print` 是 LangChain 给所有 Message 自带的方法，专门用来调试——之前讲消息类型的时候说过。但真实开发里这个 API 其实不用掌握，你在控制台打印给谁看呢？查询会话历史，最终是要把消息**返回给前端**。

### 前端要的是 role 和 content

能不能直接把 `Message` 对象 `return` 出去？不行。

那个对象的结构很复杂，元数据、工具调用、各种标记全裹在里面，直接扔给前端，前端也没法用。前端要的历史消息格式很朴素，一个 JSON，里面两个字段：

```json
{ "role": "user", "content": "12 个苹果分给 4 个人，每人分几个？" }
```

`content` 好办，每条消息里都有。`role` 没有——因为 LangChain 根本不用字段标角色，它用**类型**标：

| 消息类型 | 对应角色 |
| :--- | :--- |
| `HumanMessage` | `user` |
| `AIMessage` | `assistant` |
| `SystemMessage` | `system` |
| `ToolMessage` | `tool` |

所以要做的，是把类型翻译成 role。翻译之前得先判断类型。`type()` 只能把类型打印出来看看，判断分支用不了；判断用 `isinstance`——问一句"这个对象是不是某种类型"，是就返回 `True`，不是就返回 `False`。

拿刚才那四条消息试一下 `isinstance(message, HumanMessage)`：第一条是，`True`；第二条不是，`False`；第三条是，`True`；第四条不是，`False`。结果就是 `True False True False`，和消息的实际身份完全对得上。

那就简单了：

```python
from langchain_core.messages import HumanMessage, AIMessage

history = []
for message in messages:
    if isinstance(message, HumanMessage):
        role = "user"
    elif isinstance(message, AIMessage):
        role = "assistant"
    else:
        continue

    history.append({"role": role, "content": message.content})
```

判断出角色以后，每条消息组装一个字典：`role` 用刚算出来的，`content` 直接取 `message.content`。列表初始为空，每处理一条 `append` 一次，循环结束，`history` 里就是前端要的全部结果。

那个 `else: continue` 是有意留的——`system` 和 `tool` 这两类消息不返回给前端，前端只要 `user` 和 `assistant`。判不出来的一律跳过。

> **易错点**：`import` 的位置也会坑人。包如果写在使用它的代码下面，前面已经跑过的部分不会凭空拿到这个名字，得从头再执行一遍才生效。

### 删会话：删的是整个 thread

查询说完了，删除呢？

删一个会话，光把会话列表里的那条记录删掉不够，消息历史也得一起清——否则下次有人拿同一个 `thread_id` 来查，残留的快照还会被当成历史。

这里有一个关键的不对称，值得单独记一句：**查历史只查最后一个快照，删历史必须删掉这个 `thread_id` 下的所有快照**。

而删除不走 agent，走 checkpointer。因为所有快照本来就存在 checkpointer 里，它自己最清楚怎么清。方法叫 `delete_thread`，异步版本加个 `a`：

```python
await checkpointer.adelete_thread("s_01")
```

参数就是 `thread_id`。要删谁就指定谁，`s_01` 对应的所有快照会一并消失。

删之前先去数据库看一眼：`thread_id` 那一列有 `s_01`、`s_02` 等等，其中 `s_01` 底下挂着一大堆快照。执行完这句再刷新，`s_01` 那一批全没了；再把其他几个 `thread_id` 照同样方式删掉，几张表一起刷，干干净净。

把它俩放在一起看，分工就清楚了：

| 动作 | 用谁的方法 | 方法 | 作用范围 |
| :--- | :--- | :--- | :--- |
| 读取会话历史 | agent | `aget_state(config)` | 最后一个快照（含完整消息） |
| 读取全部快照 | agent | `aget_state_history(config)` | 该 thread 的所有快照 |
| 删除会话 | checkpointer | `adelete_thread(thread_id)` | 该 thread 的所有快照 |

一句话记住：**查用 agent 的方法，删用 checkpointer 的方法**。

到这儿，两件事就闭环了。前半段让回答一个字一个字地长出来，后端只管把模型吐出的片段封成 `ServerSentEvent` 一路 `yield`；后半段把 LangChain 替我们存下的历史取回来，翻译成前端认识的 `role` / `content`，删的时候连根一起拔掉。

> **承前启后**：上一节讲完「让回答一个字一个字地长出来：SSE 流式对话与会话历史接口」，下一节接着讲「会话历史：查得出来、删得干净」。

---

## 会话历史：查得出来、删得干净
> 对应块：BLK26 | 覆盖分集：P98-P102 | 块标题（分集名）：《会话历史查询、修复与删除》

会话管理这一块，创建会话、查询会话列表、重命名会话、删除会话，前面都已经写完了。功能清单看着挺齐，可真到页面上用，还差最要紧的一块：点开左边某一条会话，右边的聊天记录得能显示出来。这个接口一直空着，所以今天第一件事就是把它补上。

第二件事是回头修删除。之前那个删除只删了会话本身的记录，历史消息还留在库里，属于半拉子工程。查和删凑在一起，这条链路才算完整。

### 接口文档先说清楚：查历史必须带 thread_id

写接口的老规矩，先分析接口文档，再动代码。打开智能客服的接口文档，会话管理那一组里挂着：创建会话、查询会话列表、重命名会话、删除会话，再加上今天要做的查询会话历史。

文档里对这个接口的描述是「查询当前用户指定会话的历史消息，以及 interrupt 的状态」。interrupt 这一块今天先不管，只盯历史消息。文档那一段挺长，就不整段搬过来了，把关键的三样拎出来看。

- 请求方式是 `GET`；
- 请求路径是 `/api/v1/chat/threads/{thread_id}/messages`；
- 路径里带着 `thread_id`，后面还要再拼一段 `messages`。

| 项目 | 内容 |
| :--- | :--- |
| 请求方式 | `GET` |
| 请求路径 | `/api/v1/chat/threads/{thread_id}/messages` |
| 路径参数 | `thread_id`：要查的是哪一条会话 |
| 响应体 | 一个对象，含 `thread_id` 与 `messages` 列表 |
| 错误码 | 401 未登录；404 会话不存在或不属于当前用户 |

为什么路径里非得带这个 `thread_id`？想想上一节读快照的时候，`config` 里必须塞一个 `thread_id`，不然根本不知道要读哪一条会话的状态。接口这边是同一个道理：页面上会话列表可能有一大堆，用户点哪一个，前端就把哪一个的 id 带上来。点一下第一条会话，看请求路径里那一段 id；再点另一条，id 就跟着变了——路径里的这一段，就是当前会话的身份证。没有它，后端不知道你要查谁的历史。

> **提示**：读快照要 `config`，`config` 里要 `thread_id`；查历史要路径参数 `thread_id`。两处要的是同一个东西，只是入口不同。

响应体长这个样子：

```json
{
  "thread_id": "当前会话的 id",
  "messages": [
    { "role": "user", "content": "用户说了什么" },
    { "role": "assistant", "content": "AI 回了什么" }
  ]
}
```

一个对象、两个字段。`thread_id` 标记这是哪条会话，`messages` 是这条会话的全部历史消息。而 `messages` 是个列表，列表里每一条又是一个小对象，只有 `role` 和 `content`——消息嘛，总得有角色证明是谁说的，再有内容说明说了什么。

这里有个便宜可以捡。上一节演示读快照的时候，我们其实已经把「把消息列表封装成这个形状」的活儿干过一遍了。所以今天真正新写的，只是外面这一层对象，里面那串消息的拼装逻辑是现成的。

文档里还标了错误情况：未登录是 401；会话不存在、或者这条会话不属于当前用户，是 404。

第二个 404 值得多说两句。别人传一个 `thread_id` 过来，我们总不能闭着眼就查吧？`thread_id` 在页面上是明文的——你跟 DeepSeek 聊天，地址栏里那个 thread 也是明文的。如果任何人拿到一个 id 就能查历史，那不等于把别人的聊天记录拱手送出去？这不安全。所以做用户身份认证的时候，一定要判断传过来的 `thread_id` 是否属于当前用户，不属于就报错。这个判断今天就要在 service 里落地，不能等。

> **结论**：接口三要素定完了——`GET`、`/api/v1/chat/threads/{thread_id}/messages`、返回值是「一个 id 加一个消息列表」的对象；归属校验必须做。

### Schema 先落地：两个类拼出响应体

请求参数只有一个路径参数，不用额外定义 schema；响应得定义。

放哪个包？项目里现在两个包：`chat` 管 AI 对话，`chat_thread` 管会话管理。会话历史属于会话管理，不属于对话，所以 schema 落在 `chat_thread` 的 schema 文件里。

继承谁？不用 `BaseModel`，用项目里统一的 `BaseSchema`；`role` 的取值类型用 `typing.Literal`。

```python
class ChatMessage(BaseSchema):
    role: Literal["user", "assistant"]
    content: str


class ChatHistoryResponse(BaseSchema):
    thread_id: str
    messages: list[ChatMessage]
```

`ChatMessage` 里，`role` 只有两种取值：`user` 和 `assistant`，也就是用户消息和 AI 消息；`content` 是字符串。`ChatHistoryResponse` 里，`thread_id` 是字符串，`messages` 是列表，列表里装的是上面那个 `ChatMessage`。

跟文档对一遍：对象里有没有 `thread_id`？有。有没有 `messages`？有。`messages` 是不是列表？是。列表里的对象有没有 `role` 和 `content`？有。严丝合缝。

> **提示**：`role` 用 `Literal` 而不是普通的 `str`，两种之外的取值会在校验阶段就被拦下来，不用等前端发现角色不对。

两个类一拼，文档里那棵响应树就落地成了代码。

### 把 Agent 注进 ChatThreadService

以前写业务都从 repository 起步，这次不一样。为什么？因为会话历史不由我们手动查数据库——上一节读快照调的是谁？`agent.aget_state`，一行就把快照拿到了。所以查历史这件事不需要写 SQL，需要的是 agent。

问题来了：`ChatThreadService` 里有没有 agent？没有。那就在构造函数里注进去。之前 agent 是注在 `ChatService`（`chat` 包，AI 对话那个）里的，这次换个地方，注到 `ChatThreadService`。

第一个改动，让构造函数把 agent 收下来：

```python
class ChatThreadService:
    def __init__(self, session: AsyncSession, agent):
        self.session = session
        self.agent = agent
```

第二个改动，谁创建 service，谁负责把 agent 递进来。创建 `ChatThreadService` 的是 `ChatThreadRouter` 里的 `get_service`，所以改它。

agent 从哪来？启动的时候我们已经把它挂到 `app.state` 上了，取法和 `chat` 那边完全一样：拿 FastAPI 注入的 `Request`，走 `request.app.state.agent`。

```python
def get_service(
    request: Request,
    session: AsyncSession = Depends(get_session),
) -> ChatThreadService:
    # agent 挂在 app.state 上，取出来直接传给 service
    return ChatThreadService(session, request.app.state.agent)
```

`Request` 是 FastAPI 自带的，不用自己造，参数里声明一下它就给你注进来。拿到它以后 `request.app.state.agent` 一步到位。原来先取 agent 再传是两行，写成一行也一样。

```text
启动阶段
    app.state.agent = 已编译好的 Agent
              │
              ▼
ChatThreadRouter.get_service(request)     ← FastAPI 注入 Request
              │  request.app.state.agent
              ▼
ChatThreadService(session, agent)
              │
              ▼
       service 内部随时 self.agent.xxx
```

图里这条线是单向的：agent 在应用启动时挂上 `app.state`，router 在被调用时把同一个对象掏出来，塞进本次请求新建的 service 实例。所以 service 不用自己去连、去建，拿到的是现成的。

> **结论**：agent 的注入分两步——构造函数收下它，router 里从 `request.app.state` 取出来递给它。少任何一步，service 里都拿不到 agent。

### Service：校验、取快照、封装三步走

agent 到位了，就能安心写查询逻辑。整件事分成三步：

1. 校验这条会话是否属于当前用户；
2. 查会话历史；
3. 把历史封装成前端要的结构返回。

三步决定了参数：`user_id` 和 `thread_id`，都得给，而且都是字符串。少了 `user_id` 没法校验归属，少了 `thread_id` 不知道查哪条。

第一步的写法跟会话重命名、删除时用的是同一套：调 repository 里的归属查询方法（笔记里叫 `find_owned`，有的地方写成别的名字，道理一样），查不到就抛异常。查到了，才证明这条会话确实属于当前用户，可以继续往下走。

第二步就是上一节练过的东西：组一个 `config`，把 `thread_id` 塞进 `configurable`，然后 `await self.agent.aget_state(config)`。返回的是一个 `StateSnapshot`。

第三步要根据 schema 拼对象。先把快照里的消息列表取出来，逐条判断类型：是 `HumanMessage` 就记 `role = "user"`，是 `AIMessage` 就记 `role = "assistant"`，两类都不是的直接 `continue` 跳过——这个接口只展示这两种消息，别的不要。然后每一条有内容的消息，构造成 `ChatMessage` 塞进列表。循环结束，把列表和 `thread_id` 一起塞进 `ChatHistoryResponse` 返回。

```python
async def get_chat_history(self, user_id: str, thread_id: str) -> ChatHistoryResponse:
    # ① 先确认这条会话属于当前用户
    thread = await self.repository.find_owned(user_id, thread_id)
    if not thread:
        raise HTTPException(status_code=404, detail="会话不存在或不属于当前用户")

    # ② 用 thread_id 组 config，读快照
    config = {"configurable": {"thread_id": thread_id}}
    snapshot = await self.agent.aget_state(config)

    # ③ 逐条转成 schema 里的对象
    messages: list[ChatMessage] = []
    for message in snapshot.values["messages"]:
        if isinstance(message, HumanMessage):
            role = "user"
        elif isinstance(message, AIMessage):
            role = "assistant"
        else:
            continue
        if message.content:
            messages.append(ChatMessage(role=role, content=message.content))

    return ChatHistoryResponse(thread_id=thread_id, messages=messages)
```

几个点值得停一下。

`aget_state` 是异步方法，前面必须 `await`。`snapshot.values` 是个字典，里面那一项 `messages` 就是 LangGraph 状态里存着的整串消息。取法就是「先点出 `values`，再按 key 取 `messages`」，两段缺一不可。

往里塞的时候，不能像上一节演示那样塞字典了。那会儿没有 schema，随手一个字典就能打印；现在有 `ChatMessage` 了，就直接构造对象，`role` 和 `content` 两个属性一一对应。字典换成对象，道理是一回事。

> **易错点**：`continue` 不是「跳过这一条消息的内容」，而是直接结束本轮循环、进入下一条。放在这里正合适：不是用户消息也不是 AI 消息的，连角色都没法给，留着没用。

最后那层壳别忘。`messages` 只是里面，外面还得有 `ChatHistoryResponse`，`thread_id` 用前端传进来的那一个，`messages` 用刚拼好的列表。两层齐了，格式才算对上。

### Router：只做转调的薄壳

service 写完还不能直接测，还差 router。

照着接口文档来：请求方式是 `GET`，路径是在原有会话路径后面拼 `messages`，响应模型是 `ChatHistoryResponse`。最省事的做法是复制一条现成的路由（比如重命名那条），改四个地方：

1. `@router.post` 改成 `@router.get`；
2. 路径末尾补上 `/messages`；
3. `response_model` 换成 `ChatHistoryResponse`；
4. 函数体里不再收请求体，只有路径参数 `thread_id`，然后调 `service.get_chat_history`，把 `user_id` 和 `thread_id` 一起传进去。

```python
@router.get("/{thread_id}/messages", response_model=ChatHistoryResponse)
async def get_chat_history(
    thread_id: str,
    user_id: str,  # 用户 id 的取法同前面的重命名、删除接口
    service: ChatThreadService = Depends(get_service),
):
    return await service.get_chat_history(user_id, thread_id)
```

这个 router 里没有任何业务，取到参数转手交给 service 就结束。用户 id 怎么来的，跟前面几个接口完全一致，不用在这里再讲一遍。

改完刷新页面——500。

### 空会话踩坑：values 里未必有 messages

报错说的是取不到 `messages` 这个键。可 `values` 明明拿得到，为什么里面会没有 `messages`？

回到上一节的操作现场就明白了。当时我把某条会话的消息全删了，删完再 `get_state` 一次：`values` 还在，但 `values` 里已经没有 `messages` 了。一条刚建出来、还没说过话的会话也是一样——状态存在，消息键不存在。

也就是说，`snapshot.values["messages"]` 这种下标写法，默认了「这个键一定有」。只要遇到一条空会话，它就当场炸掉。调接口的第一次点击，往往正是空会话。

第一版修法是老实做判断：先把 `values` 取出来，看看里面有没有 `messages`，没有就直接返回一个空的响应，说明这是第一次来、没有任何历史消息，别再往下遍历。这个思路是对的，就是代码啰嗦。

再想一层：字典除了 `d[key]`，还有 `d.get(key, default)`。取到了用取到的值，取不到就用默认值。默认给一个空列表，后面的 `for` 循环遍历空列表，一条都进不去，最后照样返回一个 `messages` 为空的响应。这样一来，判断和遍历合成了一步：

```python
# 改前：默认这个键一定存在
for message in snapshot.values["messages"]:

# 改后：没有就按空列表处理
for message in snapshot.values.get("messages", []):
```

这就是所谓的健壮性代码。数据结构的完整与否不该由调用方赌，尤其「第一次进页面」这种必然出现的情况，不能靠运气。

> **易错点**：`values` 存在不等于 `messages` 存在。快照对象永远有 `values`，但里面装什么，取决于这条会话走到哪一步。

### content 有时候是列表

空会话能返回 200 了。接着在页面上真聊两句，再刷新——又 500。

这回的报错是「输入字段无效」：要求 `string`，实际传进来的是 `list`。问题定位到 `content=message.content` 这一行——从消息里取 `content`，取出来的是个列表，而 schema 里 `content` 声明的是 `str`，Pydantic 校验直接不通过。

奇怪的地方在于：之前测试打印出来，`content` 明明是字符串。把这条真实会话的快照打出来对着看，`HumanMessage` 的 `content` 没问题，问题出在 `AIMessage` 上——它的 `content` 是个列表。

```text
message.content 的两种形态：

字符串   "我是保险顾问，可以帮你解答各种保险相关的问题"

列表     [ { ... "text": "我是保险顾问，可以帮你解答……" ... } ]
```

LangChain 的消息内容允许「内容块」的形式，一段文本被包在块里，块里那个字段叫 `text`。所以不是取不到文字，是文字多包了一层。

最直觉的修法是从列表里挖：`message.content[0]["text"]`。不行。列表里可能有多个块，写死下标，块一多就丢内容；块的结构再变一变，这行代码又得改。

其实 LangChain 早想到了这件事，给消息留了一个 `message.text`。它表面上是个属性，点进去看是个函数，只是被包装成了属性的样子。内部逻辑很简单：`content` 是字符串，原样返回；`content` 是列表，就自己遍历每个块，把文本取出来拼成一个字符串返回。

于是所有取内容的地方统一换成 `text`，判空也一起换：

```python
# 改前
if message.content:
    messages.append(ChatMessage(role=role, content=message.content))

# 改后
if message.text:
    messages.append(ChatMessage(role=role, content=message.text))
```

| `content` 的实际形态 | 直接取 `content` | 取 `text` |
| :--- | :--- | :--- |
| 纯字符串 | 得到字符串，能用 | 得到同一段字符串 |
| 内容块列表 | 得到列表，校验失败报 500 | 自动遍历、拼接成字符串 |

换完再刷新，历史出来了。继续问一句，流式回复一条条往外冒，速度很快；点一条从没聊过的会话，是空的；再点回原来那条，消息还在。查询这一半算是通了。

> **提示**：只要是把 LangChain 的消息往外送——接口返回、写日志、前端展示——一律用 `message.text`，别直接碰 `content`。它在两种形态下都能给你一个干净的字符串。

### 删除：会话删了，快照还在

查询做完，删除那个老功能就得回头看一眼了。页面上的删除按钮不是不能用，点一下会话确实从列表里消失了，但那个删除是不完整的。

进 `ChatThreadService` 的 `remove`：开一个事务，先查当前会话是否属于当前用户，属于就把 `chat_thread` 表里对应的记录删掉。逻辑本身没毛病，问题在于——**它删的只是这张表**。

历史消息不在 `chat_thread` 表里，在 LangGraph 的 `checkpoints` 里。去数据库翻一下就清楚了：`chat_thread` 表空了，`checkpoints` 里那条 thread 的快照一点没动。

```text
一次删除涉及两处数据：

    chat_thread 表   →  会话记录（原有逻辑已经删了）
    checkpoints      →  这条 thread 的全部状态快照（原来没删）

    只删前者：会话列表里没了，历史消息还躺在库里
```

这显然不合理。会话都删了，历史还留着，既占空间，也让「删除」这个动作名不副实。

理论上这里该补一步：不仅删会话，还要把这条会话的所有状态快照一起删掉。快照怎么删？上午讲过的那个 API——checkpointer 上的 `delete_thread`，传一个 `thread_id` 进去就清干净了。

checkpointer 从哪拿？前面不是把 agent 注进 service 了吗，agent 上就挂着 checkpointer，往下翻就能看到，上面正好有 `delete_thread`。所以改动只有一行，加在删完业务表记录之后：

```python
# 删完会话表里的记录，再把这条 thread 的快照一起清掉
await self.agent.checkpointer.delete_thread(thread_id)
```

传的参数就是 `thread_id`，方法签名里它可能就叫 `id`，反正是同一个东西。这是个异步方法，前面的 `await` 不能省。

> **提示**：这一步之所以写起来这么顺，是因为前面把 agent 注进了 `ChatThreadService`。要是还像以前那样 service 里没有 agent，就得绕一圈去别处取 checkpointer。

router 不用动，只改 service 里的删除逻辑，原有的删除流程原样保留，就在后面补这一行。

验证一下。页面上那条会话历史挺长，点删除，返回 200。去数据库刷新 `chat_thread` 表，没了；再看 `checkpoints`，只剩另一条会话的一条记录。两边都干净，说明会话和历史是一起删掉的。

> **结论**：删除会话要动两处——业务表里的会话记录，和 LangGraph 里这条 thread 的全部快照。少删一处，就是脏数据。

把这一整块串起来看：从接口文档读出路径与响应结构，用两个 schema 类把响应体钉死，把 agent 注进原来不碰它的 `ChatThreadService`，service 里按「校验归属、读快照、封装消息」三步走，router 只做转调；然后踩了两个坑——`values` 里可能没有 `messages`，`content` 可能是列表——一个用 `.get` 的默认值填掉，一个换成 `message.text`；最后回到删除，补上 `delete_thread` 那一行。

会话历史这条链路，到这儿才算真正闭环。

---

## 本册小结

本册主题为「Agent 应用开发：LangChain、工具调用、记忆与流式对话」，整编了 10 个模块（P64-P66 ~ P98-P102，共 39 讲 / 459 分钟音频）。
建议配合 `notes/` 目录下的复习笔记复盘，需要查证细节时可直接回到本册正文。
