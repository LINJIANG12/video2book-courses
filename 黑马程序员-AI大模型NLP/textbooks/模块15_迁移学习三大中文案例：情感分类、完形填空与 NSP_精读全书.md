# 模块 15：迁移学习三大中文案例：情感分类、完形填空与 NSP 合辑教材

> **所属课程**：黑马程序员AI大模型NLP自然语言处理保姆级教程，PyTorch实现Transformer完整代码解析+预训练模型，一套搞定文本分类/翻译/情感分析等实战项目  
> **模块跨度**：P171 ~ P183（全模块共 13 讲系统重构）  
> **内容定位**：模块化系统学习教材，融合核心机制、架构全景、代码解析与思考自测。  
> **关联说明**：单集长文讲义同步保留于 `articles/` 目录供定向查阅。

---

## 模块导读与全景目录

1. Day11-11.迁移学习_中文分类案例_数据加载(掌握)
2. Day11-12.迁移学习_中文分类案例_数据预处理(掌握)
3. Day12-01.迁移学习_中文分类案例_模型搭建(掌握)
4. Day12-02.迁移学习_中文分类案例_模型训练(掌握)
5. Day12-03.迁移学习_中文分类案例_模型评估(掌握)
6. Day12-04.迁移学习_中文填空案例_数据预处理(掌握)
7. Day12-05.迁移学习_中文填空案例_模型搭建(掌握)
8. Day12-06.迁移学习_中文填空案例_模型训练(掌握)
9. Day12-07.迁移学习_中文填空案例_模型评估(掌握)
10. Day12-08.迁移学习_NSP案例_自定义数据集对象(掌握)
11. Day12-09.迁移学习_NSP案例_数据预处理(掌握)
12. Day12-10.迁移学习_NSP案例_模型搭建(掌握)
13. Day12-11.迁移学习_NSP案例_模型训练和评估(掌握)

---

## Day11-11.迁移学习_中文分类案例_数据加载(掌握)
> 对应分集：P171 | 原始标题：《Day11-11.迁移学习_中文分类案例_数据加载(掌握)》

迁移学习这一块要做三个案例：中文分类、中文填空、中文句子关系。这也是 NLP 阶段最后的三个案例了。

先看第一个——迁移学习的中文分类。这个任务要走的流程比之前任何一个都全乎：任务介绍、数据介绍、数据的预处理、网络模型、模型训练、模型评估。一整套动作都要求掌握。而它用的数据还是之前玩过的老数据：做情感分析时那个全国酒店的评论，好评差评，就是它。

---

### 任务介绍与数据介绍

#### 任务：拿别人的模型做二分类

任务本身不新鲜，还是中文语料的评论分类。翻开资料目录里的 `train.csv` 看一眼就认得出来：

```text
珠江花园酒店... ,1
...,0
...,1
...,0
```

`label` 那一列，1 是好评，0 是差评。所以这是一个二分类任务。

以前做这个任务是我们自己动手搭网络。现在换一种方式：**用预训练模式的 BERT 模型提取文本特征，后面接自定义的全连接层和 softmax 做二分类**。

说直白点，任务就是**把别人训练好的模型拿过来，我们接一个头上去微调**。

> **提示**：这里的"微调"只是轻微改动别人的模型，属于入门级别的操作。真正的大模型微调是单独一块内容，会有专门的库、包、函数和思路，篇幅大概两天。这里先用最小的动作把整条链路跑通。

#### 三个数据文件

数据一共三个文件：

| 文件名 | 用途 | 规模 |
| --- | --- | --- |
| `train.csv` | 训练集 | 9600 条 |
| `test.csv` | 测试集 | 与训练集同格式 |
| `validation.csv` | 验证集 | 1200 条 |

三个文件的**数据格式完全一样**，不一样的是里面的内容。格式就两列：

- `label`：标签，1 是好评，0 是差评；
- `text`：文本内容。

> **易错点**：`load_dataset(..., split=...)` 里的 `split` 不是数据集划分的动作，而是**指定你要取哪一份**。三个 CSV 各自加载时都写 `split='train'`，取的分别是各自文件里的那份数据。别被这个字面意思绕进去。

#### 从预训练模型到大模型

实际开发里，数据从哪来、模型从哪来，是两条不同的路。

有人说：是不是去网上下一份别人预训练好的模型拿来直接干活？**不一定。**有可能你手上就有公司自己的大模型——比如入职阿里、入职讯飞，直接拿公司的大模型来干活就可以。那公司没有自己的大模型怎么办？自己练一个。写算法、天天搞数据、做微调、做测试、做优化，这些活都可能是你的。

所以同一个诉求，实现路径是一层比一层强的：

```text
机器学习思想            ──> 能做
深度学习思想            ──> 能做
NLP 思想（RNN / LSTM）  ──> 能做，但效果比预训练差
预训练模型 + 迁移学习    ──> 本节的位置
大模型（通义千问 / 讯飞星火 / DeepSeek 系列） ──> 效果比预训练还要好
```

读图说明：越往下，模型带来的先验知识越多，效果越好，但需要你自己操心的底层细节越少——这也是为什么现在做同一个案例，重点从"搭网络"转移到了"接模型"。

---

### 加载数据的既有套路

在动手写之前，先把老套路对齐一下。无论你做的是什么任务，只要走的是前向传播、反向传播、搭网络那一整套，第一件事一定是把数据从 NumPy 搞成张量（Tensor），然后转成 `TensorDataset`，再转成 `DataLoader`（数据加载器），之后才能开始干活。

```text
原始数据(CSV)
     |
     v
NumPy 数组
     |
     v
Tensor 张量
     |
     v
TensorDataset 数据集对象
     |
     v
DataLoader 数据加载器  ──>  送进模型
```

读图说明：这条链的前半段（张量之前）就是本节要做的事——把 CSV 变成数据集对象，后面的封装与批次投喂交给 `DataLoader`。

`TensorDataset` 这一步在做什么？**把它转成数据集对象。** 这就是本节的核心动作：**数据加载**，先不往后说。

这套东西在深度学习阶段玩过——就是之前那个文本分类案例，用的正是这份数据。所以这一节的重点不是"学新东西"，而是"换位置"：以前你从 NumPy 那一端入手，现在从 Hugging Face 的 `datasets` 这一端入手。

---

### 环境准备与导包

先把三个数据集文件放到项目里的 `data` 目录下，然后新建 Python 文件 `demo04`，命名成迁移学习中文分类案例。

案例的注释写清楚定位就够了：NLP 任务、中文分类案例。任务是直接下载预训练模型做输入文本的特征表示，再接自定义网络进行微调输出结果。

#### 设备选择

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
```

能用 CUDA 就用 CUDA，用不了就切回 CPU。苹果的 M 系列芯片走 MPS，对应的设备字符串是 `'mps'`。

#### 导包逐个说明

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader

from datasets import load_dataset

import time
from tqdm import tqdm

from transformers import BertTokenizer, BertModel
from transformers import AdamW

from rich import print
```

每一个都别含糊：

- `torch`：PyTorch 深度学习框架，提供张量计算和自动微分功能。
- `torch.nn`：neural network，神经网络模块，线性层、卷积层、各种层都在这里。
- `DataLoader`：支持批量处理数据。
- `load_dataset`：Hugging Face 的数据集下载工具，既能下载本地文件，也能下载公开数据源——比如 GLUE、SQuAD 这类开源数据集，都可以直接拉过来。
- `time`：时间模块。
- `tqdm`：进度条库。
- `BertTokenizer` 与 `BertModel`：导入 BERT 相关组件。前者是中文分词器，后者是预训练 BERT 模型——干活用的就是它，不再是自己写机器学习或深度学习的网络了。
- `AdamW`：内置优化器，里面内置了权重衰减（weight decay）。权重衰减是干什么的？**防止过拟合**。
- `rich` 的 `print`：终端打印美化，写不写都行，加上以后输出更好看，功能上没别的区别。

> **提示**：这里用的是 `BertTokenizer` 和 `BertModel` 而不是 `Auto` 系列，也就是走到了具体模型方式那条路上——相当于直接跟模型对接，灵活性更好一些。

---

### 加载分词器与模型

#### 分词器

第一步，加载分词器，模型名用 `bert-base-chinese`：

```python
model_name = r''

my_tokenizer = BertTokenizer.from_pretrained(model_name)
```

路径用 `r''` 原样写，避免转义带来的麻烦。

#### 模型

第二步，加载模型：

```python
my_pre_model = BertModel.from_pretrained(model_name).to(device)
```

> **注意**：这个模型有一 **1.2 个 G** 那么大。不想放在原来的缓存位置，可以把模型目录复制一份放到自己项目下边，路径指过去就行——这样整理项目、迁移环境都方便。

`.to(device)` 这一步是必须的：你把模型搬到哪个设备上，后面输入的数据就得搬到同一个设备上，两边不一致会直接报错。

---

### 数据集加载函数的两种写法

第三步，定义一个函数负责加载数据集：从 CSV 文件加载训练集、测试集、验证集。

加载数据有**两种方式**。因为这是第一次在这里做这件事，两种都演示一遍；后面的案例只会挑一种用，就像讲自动模型和具体模型时一样。

用训练集演示，剩下的两个集道理完全相同。

#### 思路一：先给父目录，再给文件名

```python
# 训练数据集
# 思路一：path 设定为目标文件的父目录
train_dataset = load_dataset(
    path='csv',
    data_files='./data',
    split='train'
)

print(train_dataset)
print('训练数据的长度：', len(train_dataset))
print('训练数据前三条：', train_dataset[:3])
```

这里的 `path='csv'` 声明的是**文件类型**——你给它一个 CSV 文件，它按 CSV 去解析。因为 `data_files` 已经写到了 `./data` 这个父级节点，所以文件名不需要再写全，后面那个 `split='train'` 指明角色：这是训练用的那份。

打印这三项是标配：

- 第一条 `print(train_dataset)`：看数据集对象的整体概况；
- 第二条 `print(len(train_dataset))`：看数据量；
- 第三条 `print(train_dataset[:3])`：看前三条样本。

> **易错点**：前三条写成 `[:3]`，不是 `[:4]`。索引从 0 开始数，0、1、2 就是三条。这个细节每次都要想一下，不然打出来就是四条。
>
> **易错点**：打印带变量值的提示信息时，别忘了前面那个 `f` 前缀。写成 `print('训练数据的长度：', len(...))` 用逗号分隔是可以的；但如果想用占位符形式，写成 `print('训练数据的长度：{len(train_dataset)}')` 少了 `f`，输出就是把这段文字原样打出来，看着就是错的结果。

#### 思路二：直接给到 CSV 文件

```python
# 思路二：path 设定为文件类型，data_files 直接给到具体文件
train_dataset = load_dataset(
    path='csv',
    data_files='./data/train.csv',
    split='train'
)

print(train_dataset)
print('训练数据的长度：', len(train_dataset))
print('训练数据前三条：', train_dataset[:3])
```

两种写法的差别只在"写父目录还是写全文件"：思路一先写父类节点，再写文件名；思路二直接把 CSV 后缀写到 `data_files` 里。

跑出来结果一模一样：

```text
训练数据的长度： 9600
训练数据前三条：
{'label': [1, 1, 0],
 'text': ['珠江花园...', '...', '笔记本...']}
```

第一条数据就是珠江花园那条评论，前三条的标签是 1、1、0——两条好评，一条差评。总共 9600 条。

> **结论**：两种写法效果完全等价，一般选第二种：路径给全，`path` 里写文件后缀，一目了然，不用再去推测父级节点在哪。

#### 加载测试集与验证集

打印完训练集的基本信息，画一条华丽的分割线分节：

```python
print('-' * 10)
```

接下来加载测试集。同样有两种写法，但这里只写最后一种，注释也不带了：

```python
test_dataset = load_dataset(
    path='csv',
    data_files='./data/test.csv',
    split='train'
)

print(test_dataset)
print('测试数据的长度：', len(test_dataset))
print('测试数据前三条：', test_dataset[:3])
```

然后是验证集：

```python
valid_dataset = load_dataset(
    path='csv',
    data_files='./data/validation.csv',
    split='train'
)

print(valid_dataset)
print('验证数据集的长度：', len(valid_dataset))
print('验证数据前三条：', valid_dataset[:3])
```

验证集打出来的长度是 1200 条。

#### 返回与调用

三份数据都加载完，函数把三个数据集对象一起返回：

```python
def demo_file_to_dataset():
    # ... 训练集、测试集、验证集的加载 ...
    return train_dataset, test_dataset, valid_dataset
```

这样一次调用就能把三份数据全拿到，后面直接拿来玩就行。

```text
data/
 ├── train.csv        -> load_dataset(path='csv', data_files='./data/train.csv', split='train')
 ├── test.csv         -> load_dataset(path='csv', data_files='./data/test.csv',  split='train')
 └── validation.csv   -> load_dataset(path='csv', data_files='./data/validation.csv', split='train')
                                    |
                                    v
                    (train_dataset, test_dataset, valid_dataset)
                                    |
                                    v
                          后续：DataSet 封装 -> DataLoader -> 送模型
```

读图说明：这一节做的事情就是把三个 CSV 各自变成一个数据集对象，并整包返回；三种集加载动作完全相同，只有路径和变量名在变。

跑起来打印，输出就长这个样子，一共三套。

> **提示**：这里只是演示函数本身，所以调用之后并没有接收返回值——运行完看到输出对不对就够了，函数返回的三个对象是给后面步骤用的。

到这儿，数据加载这一关过了。数据已经到手，接下来就是往前向传播那条链上走：张量、数据集封装、数据加载器，然后才是模型。

> 💡 **承前启后**：完成对「Day11-11.迁移学习_中文分类案例_数据加载(掌握)」的理解后，下一章我们将深入探讨「Day11-12.迁移学习_中文分类案例_数据预处理(掌握)」，进一步完善知识图谱体系。

---

## Day11-12.迁移学习_中文分类案例_数据预处理(掌握)
> 对应分集：P172 | 原始标题：《Day11-12.迁移学习_中文分类案例_数据预处理(掌握)》

数据加载这一步我们做完了，接下来是数据预处理。不过这里说的“预处理”跟前面讲文本预处理时那种“先分词、再建词表、再统一长度”的流程不太一样：现在用的是预训练模型，第一件事必须是拿到它自带的分词器（tokenizer），让 tokenizer 把原始文本处理成模型能吃的格式。

这一讲要做的事只有两件，而且顺序不能颠倒：先写一个**只处理一批**的整理函数，再用 `DataLoader` 把整份数据集按批喂进来。现在把这两步拆开讲。

### 数据整理函数：一次只处理一批

数据集本身没什么新鲜的。打开 `02/data/train.tsv` 双击一看，跟之前讲文本预处理时玩过的是同一份数据，`text` 列是句子，`label` 列是 1 或 0 的标签。所以后面如果还要做词向量、做别的文本分析，很多东西可以直接复制粘贴。

`train.tsv` 的每一行就是一条样本，标签只有两种取值：

| 列名 | 含义 | 取值 |
| --- | --- | --- |
| `text` | 待分类的文本内容 | 一句中文（或中英混合）评论 |
| `label` | 情感标签 | `1` 表示好评，`0` 表示差评 |

整理函数要做的事情是：处理批次数据、统一成同一格式、转换成模型的输入。名字叫 `collate_fn`，参数只写一个 `data`。

```python
def collate_fn(data):
    ...
```

`data` 不是整个数据集，它是**传过来的那 8 条数据**。假设后面每批取 8 条，那这次调用拿到的就是从下标 2 到 9 的 8 条。数据集总共有很多批没关系——那个循环由外层的 `DataLoader` 去重复调用，而这个函数自己只负责一批。

> **提示**：`collate_fn` 是 PyTorch 的固定参数名（collate 意为“整理、校对”），它接收一个由 `Dataset.__getitem__` 取出的样本列表，返回一个 batch。这里只是把“一批样本 → 模型输入”的规则写进去，签名的第一个参数就是这批数据，不要写成 `dataset` 或 `batch` 之外的东西。

#### 拆出文本与标签

函数内部第一件事是从每条样本里取出 `x` 和 `y`。列表推导式遍历 `data`，从每个 `item` 里取 `text` 和 `label`：

```python
sentences = [item['text'] for item in data]
labels = [item['label'] for item in data]
```

`item` 是 `Dataset` 返回的一条样本，是个字典。`text` 才是句子内容，`label` 才是那个 1、0 的标签。变量名这里顺手写短了：本该叫 `sentences`，单词太长，就直接写成 `sentences` 的缩写形式，只要表意清楚即可。

#### 批量编码：batch_encode_plus 取代 encode_plus

接下来把文本转成模型输入格式，靠分词器完成。前面已经定义好 `my_tokenizer`，直接调用它的批量编码接口：

```python
inputs = my_tokenizer.batch_encode_plus(
    sentences,
    truncation=True,
    max_length=300,
    padding='max_length',
    return_tensors='pt',
    return_length=True,
)
```

为什么不用 `encode_plus`？因为那个一次只能处理一条样本。批量（batch）的单词是 `batch`，所以批量编码对应的接口就是 `batch_encode_plus`——见到 `_plus` 和 `batch_` 这两个前缀，基本能猜出它和谁是一对。

#### 编码参数逐个说明

参数逐个说清楚：

- `sentences`：输入文本列表，也就是这一批的 8 个句子，而不是一句。
- `truncation=True`：启用文本截断，超过长度的部分直接切掉。
- `max_length=300`：最大序列长度，句子短了会被补齐到 300。这里直接写 300 是图省事，规范做法是先统计一遍自己数据集里句子长度的分布，看绝大多数集中在多少，再按分布定阈值；300 已经能囊括绝大多数样本。
- `padding='max_length'`：启用填充，把不足 `max_length` 的序列补齐到 300。
- `return_tensors='pt'`：返回 PyTorch 的二维张量。不加这个参数，拿回来的还是 Python 列表，后面还得自己转。
- `return_length=True`：附带返回编码后的序列长度。原始句子长度是 10，编码后可能因为特殊符号和补齐变成别的数，这个参数就把它一并告诉你。它是可选项。

### inputs 里到底装了什么

编码结果可以打印出来看一眼。`inputs` 其实就对应以前处理过的那个 `input_ids`——文本被转成的编号序列，只不过这里是一个**字典**，里面装着三组键值对。

```text
inputs (BatchEncoding)
├── input_ids        形状 [batch_size, max_length]   文本转成的编号
├── token_type_ids   形状 [batch_size, max_length]   单句为 0，句子对为 1
├── attention_mask   形状 [batch_size, max_length]   1 为真实 token，0 为填充
└── length           每个样本编码后的真实长度（return_length=True 才有）
```

读图说明：字典的前三个键是模型真正要的输入，`length` 只是顺手带回来的统计信息，不用喂给模型。所以才需要把这三组键值对一个个抠出来。

```python
input_ids = inputs['input_ids']
token_type_ids = inputs['token_type_ids']
attention_mask = inputs['attention_mask']
```

`token_type_ids` 的形状同样也是 `[batch_size, max_length]`。它的取值规则记住一句话就够：**单句为 0，句子对为 1**。句子对指的是成对输入的句子（比如判断两句话是否同义、问答是否匹配），那种场景里第二句的 token 会被标成 1，用来告诉模型哪部分是句子 B。本案例是单句分类，所以这一项全是 0，但接口仍然要照常传。

`attention_mask` 的取值更简单：真实 token 为 1，填充出来的位置为 0，模型靠它忽略掉补齐部分的干扰。

> **易错点**：`input_ids`、`token_type_ids`、`attention_mask` 三者形状一致，都是二维张量，取错了不会报错但模型会算错。它们必须来自同一次编码结果，不要拿着上一次调用的 `attention_mask` 配这一次的 `input_ids`。

#### 标签转张量并返回

标签目前还是 Python 列表，要手动转成长整型张量（分类任务的标签必须是 `long`）：

```python
labels = torch.longTensor(labels)
return input_ids, token_type_ids, attention_mask, labels
```

返回的四个东西里，前三个是**特征**，最后一个是**标签**。到这里，一批数据的整理函数就写完了：

```python
def collate_fn(data):
    sentences = [item['text'] for item in data]
    labels = [item['label'] for item in data]

    inputs = my_tokenizer.batch_encode_plus(
        sentences,
        truncation=True,
        max_length=300,
        padding='max_length',
        return_tensors='pt',
        return_length=True,
    )

    input_ids = inputs['input_ids']
    token_type_ids = inputs['token_type_ids']
    attention_mask = inputs['attention_mask']
    labels = torch.longTensor(labels)

    return input_ids, token_type_ids, attention_mask, labels
```

> **提示**：写代码时如果某个包名报红，通常不是逻辑错，而是当前解释器环境里没装这个包。用 PyCharm 的 Jupyter 时尤其容易踩：切了内核（CPU 环境与 GPU 环境是两个不同的环境），同一个包在 GPU 环境装过、在 CPU 环境没装，代码立刻报红。把包在该环境补装一遍即可。

### 获取数据加载器

现在写 `get_data_loader`。这个函数只做两件事：加载数据集，把它封装成 `DataLoader`。

加载数据集这一步就是把前面写过的代码原样搬进来——通过 `load_dataset` 加参数直接加载：

```python
train_dataset = load_dataset('csv', data_files='./02/data/train.tsv', ...)
```

有了数据集对象，接下来交给 `DataLoader`：

```python
my_data_loader = DataLoader(
    dataset=train_dataset,
    batch_size=8,
    shuffle=True,
    drop_last=True,
    collate_fn=collate_fn,
)
return my_data_loader
```

五个参数，逐条说清：

- `dataset`：上面加载出来的数据集对象，也就是数据来源。它决定了“从哪拿”。
- `batch_size=8`：批次大小，即每批包含的样本数。这个 8 必须和整理函数里假设的“一批 8 条”对齐，否则前面那些讲解就对不上了。
- `shuffle=True`：训练集是否随机打乱。训练时要打乱，避免模型记住样本顺序。
- `drop_last=True`：是否丢弃最后一个不完整的批次。假设一批 8 条，最后只剩 7 条，那就整批不要、不处理，这样能保证每一批都严格是 8 条。
- `collate_fn=collate_fn`：指定数据整理函数，即每一批数据都要先经过它处理。参数名与函数名重名容易混，所以实现时给函数另起了一个名字（例如 `collate_fn` 加个后缀），用来说明“这个参数收的正是上面那个自定义函数”。

`drop_last` 有个必须注意的分寸：

> **易错点**：`drop_last=True` 只能在批次较小的时候用。如果 `batch_size` 设成 8000，而数据集只够几批，最后一批不够 8000 条会被整个丢掉，等于白扔一大块数据。数据量很大时丢最后三五条无所谓，但批次绝不能定得过大。

`collate_fn` 这个参数是整个流程的关键扣子，把它的执行链条画出来就清楚了：

```text
DataLoader(dataset, batch_size=8, collate_fn=collate_fn)
        │
        │  迭代取一批
        ▼
  从 dataset 取 8 条原始样本  →  [item0, item1, ..., item7]
        │
        │  这 8 条不是直接返回，而是先交给 collate_fn
        ▼
  collate_fn(data)
        ├── 取 text / label
        ├── batch_encode_plus 批量编码
        └── 拼成 (input_ids, token_type_ids, attention_mask, labels)
        │
        ▼
  for batch in train_dataloader  拿到的就是整理后的四元组
```

读图说明：外部只管从 `DataLoader` 里一批一批地取，取到的并不是原始样本，而是 `collate_fn` 处理后的结果——模型要的三样特征加一个标签。整理逻辑写在函数里，取数逻辑写在循环里，两边职责分开。

### 验证第一批数据

函数写完别忘了 `return`。为了确认整条链路通了，把 `get_data_loader` 调一次，只取第一批数据看效果：

```python
train_dataloader = get_data_loader()

for batch in train_dataloader:
    print(batch)
    break
```

`break` 的作用是“只看一批即可”：从加载器里拿一批出来就停，不然会把整个数据集全部打印一遍，既慢又刷屏。打印出来的结构就是刚才整理函数返回的四个结果：

- 第一个是 `input_ids`，一长串编号；
- 第二个是 `token_type_ids`，因为都是单句，全为 0；
- 第三个是 `attention_mask`，真实 token 位置为 1，补齐位置为 0；
- 最外层大括号里的那一组小括号就是 `labels`——值非 0 即 1，一行恰好 8 个，因为一批就是 8 条数据。

于是“1、1、1、0、0、0……”这样的序列就是这一批样本的标签：第 1、2、3 条是好评，后面几条是差评。数据加载器至此获取成功，整条预处理链路可以往下接模型搭建了。

> 💡 **承前启后**：完成对「Day11-12.迁移学习_中文分类案例_数据预处理(掌握)」的理解后，下一章我们将深入探讨「Day12-01.迁移学习_中文分类案例_模型搭建(掌握)」，进一步完善知识图谱体系。

---

## Day12-01.迁移学习_中文分类案例_模型搭建(掌握)
> 对应分集：P173 | 原始标题：《Day12-01.迁移学习_中文分类案例_模型搭建(掌握)》

每批次的数据已经处理完了，也拿到了处理后的数据加载器。接下来该搭网络模型，然后训练、预测。

先说结论：这个模型简单到只有**一个全连接层**。但“为什么敢这么简单”“为什么输出是 2”这两个问题必须想清楚，不然模型搭出来自己都不知道在干什么。

### 为什么只有一个全连接层

`label` 只有好评、差评两种取值，这是一个标准的二分类问题，所以输出维度是 2。那输入维度为什么是 768？因为我们用的是 BERT 模型，它的词向量维度是 768 维。数据在这条链路上的形状变化非常干净：

```text
原始文本
   │  tokenizer 编码 + 补齐
   ▼
input_ids / token_type_ids / attention_mask   [batch_size, max_length]
   │  BERT 编码（预训练模型，参数冻结）
   ▼
pooler_output                                 [batch_size, 768]
   │  自定义全连接层 768 -> 2
   ▼
logits                                        [batch_size, 2]
```

读图说明：BERT 负责把文本压成 768 维的语义表示，我们自己只负责在它后面接一层把 768 映射成 2。所谓“上游”和“下游”的分界就在这张图中间那道竖线上。

> **定义**：上游模型（upstream model）指的是在本模型之前完成工作的部分，比如分词器和 BERT 本体；下游模型（downstream model）指的是在它之后、拿它的输出接着做任务的模型。我们写的这个分类模型拿的是 BERT 输出的 768 维表示，所以它是下游模型。

类名定为 `AIModel`，继承 `nn.Module`。这里有个小细节：如果直接写成全大写的 `AIModel`，`AI` 两个字母连在一起容易被看成一个整体，所以把 `i` 写成小写，视觉上更清楚。

```python
class AIModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = nn.Linear(768, 2)
```

`__init__` 里第一步永远是初始化父类成员：`super().__init__()`。第二步定义全连接层 `self.fc`，它的作用就是把 BERT 的 768 维转成二阶。

全连接层的属性名写法有两种，`self.fc` 和 `self.linear` 都行，`fc` 是 Full-connected（全连接）的缩写，实际开发中这两种形式都常见。写哪个都可以，但一定要记住自己在写什么。

### forward 的输入参数

`forward` 要接收的参数比较多，三个特征都得传进来：

```python
def forward(self, input_ids, token_type_ids, attention_mask):
    ...
```

三个参数各自的含义和形状：

| 参数 | 含义 | 形状 |
| --- | --- | --- |
| `input_ids` | 文本的数字编码 | `[batch_size, max_length]`，即 `[8, 300]` |
| `token_type_ids` | 句子类型标记（句子段） | 同 `[batch_size, max_length]` |
| `attention_mask` | 注意力掩码，即是否有填充 | 同 `[batch_size, max_length]` |

`token_type_ids` 说的是句子关系：多个 0 表示同一个句子，多个 1 表示属于另一个句子。本案例是单句分类，它全是 0，但参数照传。

### 冻结 BERT 参数

第一个动作是**不计算 BERT 预训练模型的梯度**，也就是冻结参数。

为什么？BERT 是别人已经预训练好的，如果在这里还去更新它的参数，那就是在做模型微调（fine-tuning）了。我们这一步只想训练自己写的那一层，所以 BERT 的参数不参与更新。

```python
with torch.no_grad():
    bert_output = my_pre_model(
        input_ids=input_ids,
        token_type_ids=token_type_ids,
        attention_mask=attention_mask,
    )
```

`torch.no_grad()` 的作用是不计算梯度，把它包在取 BERT 输出的那段代码外面，BERT 的参数就不会被更新。整个动作的效果是：**避免更新预训练模型参数，仅训练自定义分类层（也就是那个 `Linear`）的参数**。

> **提示**：传给 BERT 的三个参数建议一律用关键字传参，不要按位置硬塞。原因是这三个张量形状完全一样，即使顺序传错了也不会报错，前向照样算得出来，但结果就是错的——这种 bug 最难查。把参数名明确写出来，虽然代码看起来长了一点，但绝对不会传错。想按位置传，得先去翻底层源码确认参数顺序，那是拿正确性赌一把。

### 取 pooler_output

拿到 BERT 的输出，就意味着拿到了那 768 维表示，接下来才能把它转成 2 维。

```python
output = self.fc(bert_output.pooler_output)
```

`bert_output` 里有很多字段，我们要的是 `pooler_output`。它是池化（pooling）处理后的结果，形状是 `[batch_size, 768]`——也就是每一条样本一个 768 维向量。分类所需的信息就在这里面。

> **易错点**：`pooler_output` 与 `last_hidden_state` 不是一回事。前者是取 `[CLS]` 位置再经过一层线性加激活得到的句向量，形状 `[batch_size, hidden]`；后者是所有 token 的隐状态，形状 `[batch_size, seq_len, hidden]`。句子级分类要用前者，直接拿后者送进 `nn.Linear(768, 2)` 会因维度不匹配而报错。

最后把结果返回：

```python
return output
```

完整的下游模型：

```python
class AIModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = nn.Linear(768, 2)

    def forward(self, input_ids, token_type_ids, attention_mask):
        with torch.no_grad():
            bert_output = my_pre_model(
                input_ids=input_ids,
                token_type_ids=token_type_ids,
                attention_mask=attention_mask,
            )
        output = self.fc(bert_output.pooler_output)
        return output
```

### 只测一批并搬数据到 GPU

模型搭好不能就这么放着，得测一下。测试函数叫 `use_AIModel`，四步：获取数据加载器、实例化模型、把模型移到 GPU、迭代数据并输出。其中实例化和搬到 GPU 可以合并成一行 `my_model = AIModel().to(device)`，拆开写就是先 `my_model = AIModel()`，再 `my_model = my_model.to(device)`。

迭代时**只测一批**，目的是避免长时间运行：

```python
for input_ids, token_type_ids, attention_mask, labels in my_dataloader:
    input_ids = input_ids.to(device)
    token_type_ids = token_type_ids.to(device)
    attention_mask = attention_mask.to(device)
    labels = labels.to(device)

    output = my_model(input_ids, token_type_ids, attention_mask)
    print(output)
    break
```

循环变量为什么能直接解包成四项？因为从 `my_dataloader` 里取出的每一批数据都是**经过那个整理函数处理过的**，而整理函数返回的正是这四个东西：`input_ids`、`token_type_ids`、`attention_mask`、`labels`。所以这里按顺序接住即可。

用 GPU 稍微麻烦的一点就在这里：数据也得搬到 GPU 上，四个张量一个都不能漏。`input_ids` 和 `token_type_ids`、`attention_mask`、`labels` 都要 `.to(device)`。

搬完数据做前向传播，把模型要的三个参数扔进去，得到 `output`。打印它的形状是 `[8, 2]`：一批 8 条数据，每条输出两个分数。这就是二分类的分数，还不是概率；要把这两个 logits 映射成概率，才能比较左右哪个大，从而判断这句话该归到 1 还是 0。

`break` 的作用就是“仅测试一批，避免长时间运行”。这一句在课堂验证里必须有，但实际开发里要是随手把 `break` 留在训练循环里，后面那批数据全都不训练了，后果很严重。

#### 关闭警告与 FlashAttention

跑起来会看到一个 `UserWarning`，内容大意是当前 PyTorch 没有编译 FlashAttention。解决方式很简单，在代码最上面过滤掉警告：

```python
import warnings
warnings.filterwarnings('ignore')
```

这句话的意思是：凡是警告就别提示我了。为什么会有这条警告？因为当前用的 PyTorch 版本没有编译 FlashAttention 这个加速功能。

> **定义**：FlashAttention 是斯坦福大学提出的一种高效注意力计算优化技术，它重新设计了内存结构，从而优化了计算量与显存占用。目前 PyTorch 等框架已经支持，但需要安装**编译过 FlashAttention 的 PyTorch 版本**。

如果你追求极致性能，可以尝试自己装一个编译过 FlashAttention 的 PyTorch，但过程复杂：GPU 型号、CUDA 版本都要兼容，还需要从源码编译 PyTorch——不是从网上下载就能直接用，而是下载编译前的源码，用自己机器的环境手动编译一遍，它才会支持 FlashAttention，性能会比现在稍微快一点。而且在 Windows 环境下配置难度更高；Mac 上的命令和 Linux 很像，相对好一些，Windows 玩这套东西更难，折腾下来要花很久。

我们把警告压制掉之后重新跑一遍，那条警告信息就不见了，输出里能看到 `cuda:0`，说明模型和数据确实跑在 GPU 上。

> 💡 **承前启后**：完成对「Day12-01.迁移学习_中文分类案例_模型搭建(掌握)」的理解后，下一章我们将深入探讨「Day12-02.迁移学习_中文分类案例_模型训练(掌握)」，进一步完善知识图谱体系。

---

## Day12-02.迁移学习_中文分类案例_模型训练(掌握)
> 对应分集：P174 | 原始标题：《Day12-02.迁移学习_中文分类案例_模型训练(掌握)》

模型搭好了，接下来是训练的活。整个流程拆开看只有五步：创建模型、创建优化器和损失函数、定义几个变量，然后分轮、分批次地做具体训练——每批里算损失、反向传播，照例三剑客（梯度清零、反向传播、梯度更新）走一遍，最后保存模型。

顺序上有个必须说清的地方：规范的 `train_model` 里，**加载数据 → 初始化模型 → 配置训练参数 → 迭代训练 → 保存模型**，这五步是骨架。

```text
train_model()
    │
    ├── 1 加载训练集（load_dataset）
    ├── 2 创建模型并移动到指定设备
    ├── 3 冻结 BERT 预训练模型的参数
    ├── 4 创建损失函数对象
    ├── 5 创建优化器对象
    ├── 6 设置模型为训练模式
    ├── 7 迭代训练（epoch 循环 → batch 循环 → 三剑客）
    │      └── 7.4 每轮训练结束，保存一次模型
    └── 返回
```

读图说明：前六步都是“准备”，第 7 步才是真正的训练，而保存模型挂在 epoch 循环内部，所以是训练一轮存一个模型文件。

### 加载数据与模型初始化

加载训练集用 `load_dataset`。这里有两种写法：一种是把类型和文件路径都扔进去；另一种是把父目录（数据集名）扔进去，后面只写子文件夹里的具体文件。两种都能加载出数据集对象。

```python
train_dataset = load_dataset('csv', data_files='./data/train.tsv', ...)
```

拿到数据集之后第二步就是创建模型并移动到指定设备，写法与前面测试时完全一致：

```python
my_model = AIModel().to(device)
```

### 冻结 BERT 参数

第三步是冻结 BERT 预训练模型的参数，因为调优只调我们自己的那一层。

```python
for param in my_pre_model.parameters():
    param.requires_grad = False
```

`my_pre_model` 就是前面加载好的 BERT 模型对象，遍历它的全部参数，把每个参数的 `requires_grad` 置为 `False`，意思是这些参数不参与梯度更新。

`requires_grad` 是 PyTorch 自动微分机制里的开关：为 `True` 时框架会替你算梯度，为 `False` 时不计算。所以冻结参数就是把它统一关掉。前面讲自动微分时已经见过这个属性，这里只是把它用在了“只训练自定义分类层”这个具体目的上。

> **易错点**：属性名 `requires_grad` 中间**有下划线**。写成 `requires_grad` 漏掉下划线，代码不一定立刻报语法错，但后面反向传播时会抛出一个信息量很低的错误——大意是某个变量既不需要梯度、也没有 `grad_fn`，无法参与反向计算。看到这类报错，第一反应就应该是回来检查这个属性名（以及把 `requires_grad` 当函数调用时有没有传对参数）。IDE 对这个属性往往不给自动补全提示，只能靠记牢。

写这个属性有两种等价方式：直接赋值 `param.requires_grad = False`，或者调用 `param.requires_grad_(False)`——加下划线之后它是一个函数，把 `False` 当参数传进去。两种写法效果完全一样，选一种顺手的即可。

### 损失函数与优化器

第四步创建损失函数对象：

```python
criterion = nn.CrossEntropyLoss(reduction='mean')
```

`reduction='mean'` 表示对一批样本的损失取平均。点进源码看会发现它的默认值本来就是 `mean`，所以这一项**不写也行**，写了只是明确表达意图。

> **提示**：这里直接用 `CrossEntropyLoss` 就够了，不必再像早先那样把 `NLLLoss` 和 `LogSoftmax` 手动串起来。`CrossEntropyLoss` 内部已经把 Softmax 与负对数似然合并，输入应当是未归一化的 logits（也就是前面打印出来的那两个分数），不要再对 `output` 额外做一次 Softmax。

第五步创建优化器对象。由于前面导包时没有导入 `optim`，要么补一句 `from torch.optim import ...`，要么给 `torch.optim` 起个别名：

```python
import torch.optim as optim

optimizer = optim.Adam(my_model.parameters(), lr=1e-4)
```

起了别名之后就不用每次都写 `torch.` 前缀，直接写 `optim.Adam(...)` 即可。优化器接收的参数是“要更新的那些参数”，也就是 `my_model` 的 `parameters()`——这里只收了我们自己的模型参数，BERT 的部分已经在上一步冻住了。`lr` 是学习率。

第六步把模型设为训练模式：

```python
my_model.train()
```

### 训练循环的层级结构

第七步是迭代训练。这里假设训练三轮，所以最外层是 epoch 循环：

```python
for epoch_index in range(3):
```

注意一个差别：以前写代码时习惯把 `epochs = 3` 定义成变量再传进来，这次直接写死在 `range(3)` 里，所以改轮数要来这里改。

每一轮内部先记下本轮的开始时间，再取数据加载器：

```python
    start_time = time.time()
    my_dataloader = get_data_loader()
```

`start_time` 用来在批次训练结束后算这一批的耗时。

#### 批次循环与进度条

接下来是本轮里每个批次的训练过程，用 `enumerate` 从 1 开始编号，并套上 `tqdm` 显示进度条：

```python
    for i, (input_ids, token_type_ids, attention_mask, labels) in enumerate(
        tqdm(my_dataloader), start=1
    ):
```

`enumerate` 的 `start=1` 让批次编号从 1 开始，语义上比从 0 开始更直观。

循环变量为什么能直接解包成四项？因为数据加载器每批返回的都是 `collate_fn` 处理后的结果，而那四个返回值就是 `input_ids`、`token_type_ids`、`attention_mask`、`labels`。数据量大的时候一批批刷屏很难看，套上 `tqdm` 就能用进度条的方式查看训练进度。

#### 单批训练的三个动作

进入批次循环后，第一个动作是**把四项参数全部移动到指定设备**：

```python
        input_ids = input_ids.to(device)
        token_type_ids = token_type_ids.to(device)
        attention_mask = attention_mask.to(device)
        labels = labels.to(device)
```

这一步容易漏：模型在 GPU 上、数据还在 CPU 上，前向传播就会因设备不一致直接报错。四个张量一个都不能少。

第二个动作是前向传播，把模型要的三个参数传进去拿输出：

```python
        output = my_model(
            input_ids=input_ids,
            token_type_ids=token_type_ids,
            attention_mask=attention_mask,
        )
```

这里仍然建议用关键字传参。原因在于这三个张量形状完全一样，位置传错也不会报错；而自己写的 `AIModel` 里 `forward` 的参数顺序是自己定死的，能按住 Ctrl 点进去看，但用别人的模型时未必知道谁在前谁在后，与其去赌，不如直接写参数名。

> **提示**：`forward` 里传什么、怎么传，取决于我们自己写的模型签名。训练循环里的调用方式必须与 `forward` 定义的参数名一致，改动其中一处时别忘了同步另一处。

第三个动作是计算损失：

```python
        loss = criterion(output, labels)
```

`output` 是模型算出来的预测值，`labels` 是真实值，两者交给损失函数算出这一批的损失。

#### 三剑客与训练日志

有了损失，接下来就是老三样，顺序不能乱：

```python
        optimizer.zero_grad()   # 梯度清零
        loss.backward()         # 反向传播，算梯度
        optimizer.step()        # 更新参数
```

清零必须在反向传播之前。PyTorch 的梯度默认是**累加**的，不清零的话这一批算出来的梯度会叠加上一批的，等于变相放大了学习率。

一批参数更新完，这一批就算结束了。接下来是打印训练信息的判断：

```python
        if i % 20 == 0:
            # 一批 8 条，20 批就是 160 条数据
            ...
```

不能每一批都打印——一批 8 条，数据量大约 9000 条，按 8 条一批就有 1000 多个批次，日志会刷到没法看。改成每 20 批打印一次，也就是每 160 条数据一条日志，整轮下来只有几十条输出，看起来舒服得多。`i` 是批次 ID，从 1 一路往上加，`i % 20 == 0` 就是在第 20、40、60……批的时候打印。

打印的内容包含四项，都是以前写惯的算式：

```python
            pred = torch.argmax(output, dim=-1)
            acc = (pred == labels).float().sum().item() / len(labels)
            use_time = time.time() - start_time
```

- `torch.argmax(output, dim=-1)`：对输出在最后一维取最大值索引，`[8, 2]` 的两个分数里谁大就取谁的下标，得到预测类别。
- 准确率：`(pred == labels)` 把两个张量逐元素比较，得到一串 `True`/`False`；`.float()` 把它转成 1.0/0.0，`.sum()` 求和得到预测正确的条数，`.item()` 取出标量，最后除以 `len(labels)`（即这一批的样本数）得到准确率。
- 耗时：当前时间减去本轮开始时间。

最后把信息打印出来，包含轮数（`epoch_index` 从 0 开始，所以要加一）、迭代步数（就是批次号 `i`）、当前损失（`loss` 保留两位小数）、准确率（保留两位小数即可，不必留四位）以及耗时。

### 保存模型与两类踩坑

第七步之后还有 7.4：走到这里说明**本轮训练完毕，保存模型**。

```python
    torch.save(
        my_model.state_dict(),
        f'./model/classification{epoch_index + 1}.pth',
    )
```

保存的路径是 `./model/` 下面按轮数命名的文件。`epoch_index` 的取值是 0、1、2，加一之后正好是 1、2、3，也就是三轮三个模型文件。`torch.save` 保存的是 `my_model` 的参数。扩展名用 `.pth`，换成别的写法也可以。

> **易错点**：`torch.save` 这一行必须写在 epoch 循环**内部**。它的缩进位置决定了作用范围：缩进在循环体内，就是“一轮训练完保存一次”；如果把它往外挪一级变成与 `for` 平级，就变成“全部轮次训练完只保存一次”——两种写法训练出的模型文件数量完全不同。

#### 目录与属性名两个坑

第一次执行训练代码会直接报错。原因不在代码逻辑，而是 `./model/` 这个文件夹根本不存在，`torch.save` 没法把文件写进去。解决办法就是先把 `model` 文件夹创建出来，之后再跑就不会有问题了。

> **提示**：凡是写“保存到某个目录”的代码，先确认该目录存在。要么手动建好，要么在代码里用 `os.makedirs(..., exist_ok=True)` 兜底，否则一定在训练跑完的那一刻才报错，白等一轮。

接下来还会遇到第二个错，报错信息大致是某个 `element 0` 既不需要梯度、也没有 `grad_fn`，也就是**没有梯度可以更新**。按经验，先顺着“更新参数”这条线往回找，锁定到冻结参数那一段：那里确实做了筛选，筛的是 `my_pre_model`（BERT），只冻结了预训练模型的参数，这部分逻辑是没问题的。

根因其实在更早的地方——写 `AIModel` 的时候，为了配合测试函数，把前向传播整段包进了 `torch.no_grad()` 里：

```python
# 问题写法：整段前向都被包进了 no_grad 里
def forward(self, input_ids, token_type_ids, attention_mask):
    with torch.no_grad():
        bert_output = my_pre_model(...)
    output = self.fc(bert_output.pooler_output)
    return output
```

`torch.no_grad()` 的意思是整段计算都不记录梯度。测试阶段只关心前向结果、不需要反向，这样写没问题；但训练时如果整个前向都不记录梯度，后面 `loss.backward()` 自然无梯度可用。

正确的做法是：`no_grad` 只包住**取 BERT 输出**那一段，不要把 `self.fc(...)` 也包进去。

```python
def forward(self, input_ids, token_type_ids, attention_mask):
    with torch.no_grad():
        bert_output = my_pre_model(
            input_ids=input_ids,
            token_type_ids=token_type_ids,
            attention_mask=attention_mask,
        )
    output = self.fc(bert_output.pooler_output)
    return output
```

这样一来，BERT 前向不产生梯度、参数不更新，而自定义分类层的前向照常记录梯度，`optimizer.step()` 更新的就恰好是它自己的参数。这也正好和训练脚本中那段 `requires_grad = False` 的筛选配合上：该冻的冻住，该学的学好。

> **结论**：`torch.no_grad()` 应该只包住“不需要梯度的那部分计算”，而不是把整个 `forward` 一裹了事。冻结参数用 `requires_grad = False` 管住参数，不记录计算图用 `no_grad()` 管住过程，两者职责不同，`no_grad` 包多了会把训练一起冻死。

#### 日志刷屏的处理

再跑一次，代码能跑起来了，进度条也出来了，但马上又出现一个新瑕疵：**信息打印得太多**。原因是在处理数据那一段里，每批都打印了一次 `inputs`。当时是为了看清编码结果长什么样，现在训练起来每批都打印，一屏接一屏往外滚，进度条被冲得没法看。

把那行打印 `inputs` 的代码注释掉，再运行，输出就变成每 20 批一行、干干净净的样子。当然，如果你就喜欢每批都看到一次输出，不注释也能跑，只是要忍受刷屏。

#### 损失为什么在震荡

日志干净之后，新的疑问来了：**损失怎么一直在变化、一直在震荡？**

先分清“震荡”这个词的含义。所谓震荡，是指整体趋势一会儿变大一会儿变小。现在打印出来的损失不是全量损失，而是**最近这 20 批的损失**。要算整轮的损失，必须在循环外面定义一个 `total_loss`，把每一批的损失累加起来再输出。

所以当前这种忽高忽低是正常的：20 批里模型在这 160 条数据上的表现本来就会波动。日志里能看到有的 20 批准确率是 25%，隔一会儿那 20 批准确率是 100%——那 20 批数据全部预测正确。

同时，训练开始时 `./model/` 目录下一个文件都没有，也是正常的：第一轮还没跑完，保存动作自然一次都没执行。等到进度条走到 100%，第一轮结束，`classification1.pth` 就出现了。

> **易错点**：没有 `total_loss` 累加时，日志里的 loss 是“这一小段批次”的 loss，不能拿它判断整轮收敛趋势。想看整轮趋势，就得在 epoch 循环里累计、在轮末输出。看到 loss 波动就断定训练不收敛，属于误判。

还有一处容易误会的现象：训练跑到第二轮时，去文件列表里看，模型文件似乎还是没更新。这时在项目上右键，执行一次 **Reload from Disk**（从磁盘重新加载），文件就会显示出来——它相当于刷新一下文件视图，并不代表文件没生成。

第一轮耗时不短，接近两分钟。这里有个务实的判断：如果你不追求极致的效率，训练出一个可用的模型就足够支撑后面的预测环节了，不必非把三轮全跑完再往下走。代码也发给大家自己跑一跑，看看各自的机器一轮要多久——等三轮跑完，`./model/` 下就会得到 `classification1.pth`、`classification2.pth`、`classification3.pth` 三个模型。

> 💡 **承前启后**：完成对「Day12-02.迁移学习_中文分类案例_模型训练(掌握)」的理解后，下一章我们将深入探讨「Day12-03.迁移学习_中文分类案例_模型评估(掌握)」，进一步完善知识图谱体系。

---

## Day12-03.迁移学习_中文分类案例_模型评估(掌握)
> 对应分集：P175 | 原始标题：《Day12-03.迁移学习_中文分类案例_模型评估(掌握)》

训练日志刷到最后，屏幕上滚出来一列东西：迭代步数 920 到 940，每个步数后面跟着一个 loss 和一个 ACC。有人会立刻起疑：这两行数字到底算不算数？它统计的是第 920 到第 940 批这 20 批的结果，既不是全集，也不是单独的一轮。损失和准确率看着都不太准，那这个模型真正能打多少分？

这就是我们现在要补上的最后一块：模型评估。套路跟前面训练那一段几乎一样，所以可以直接写。整个评估函数的流程是六件事：加载数据、创建加载器、加载模型、批量批量预测、计算准确率、打印结果。我们用一个专门的 `evaluate_model` 把它落实。

### 评估函数的六个动作

图多九，这里边讲的就是模型评估函数。先把动作清单摆出来，这就是接下来逐行写的东西：

1. 加载测试集数据；
2. 创建数据加载器对象；
3. 加载训练好的模型权重；
4. 初始化评估参数；
5. 把模型设为评估模式；
6. 迭代预测，逐批统计，再打印结果。

动作虽然多，但只有第二步有真正的坑。

#### 加载测试集数据

第一步是加载数据集，注意是测试集。上来就写 `test_dataset`，内容调用还是 `load_dataset` 那一套：

```python
# 1. 加载数据集
test_dataset = load_dataset(
    path="./data/test.csv",      # 注意：这里换成 test.csv
    split="train",               # 这里依旧写 train，无所谓
    max_length=300,
    ...
)
```

有两个地方必须说清楚。

第一个：文件名换的是 `test.csv`，这个千万不要写错。数据源在之前做切分的时候就已经把 train、test、validation 分好了，所以评估阶段直接用 `test.csv` 就行。

第二个：`split` 这个参数你依旧写 `train`，无所谓的。有人会问，那我要不要按测试集去切？没必要。因为我们在文件层面已经把数据切好了，一个文件本身就是测试集，你没必要在这个文件里再切一次。所以直接写 `train` 就对了。

> **易错点**：`split="train"` 在这里跟"训练数据"没有关系。它指的是从一个数据集对象里取哪个子集，而我们的 train/test/validation 是三个不同的 CSV 文件，切分已经发生在文件这一层，函数里再写 `"test"` 反而取不到东西。

#### 把数据加载器改成通用函数

第二步是创建加载器对象，也就是数据加载器 `my_dataloader`。正常思路是直接调用上次写好的 `get_dataloader`，但这里不能这么干。

按住 Ctrl 点进 `get_dataloader` 看一眼就明白了：这个函数里边写死的居然就是 `train.csv` 那个路径。它只服务于训练集，评估阶段拿去用会直接读错文件。要让这个函数通用，就得做两件事：

- 把文件路径从函数内部提到参数列表上来；
- 把 `shuffle` 的 `True`/`False` 也变成参数传进来。

其余部分一模一样。所以评估这里不能调用那个函数，得把加载器里那一套自己写一遍，只改两个地方：

```python
# 2. 创建数据加载器
my_dataloader = DataLoader(
    dataset=test_dataset,     # 改动一：由 train_dataset 换成 test_dataset
    batch_size=8,
    shuffle=False,            # 改动二：评估时不再打乱数据
    drop_last=True,
    collate_fn=collate_fn,
)
```

改动一：把 `dataset` 从之前的 `train_dataset` 换成 `test_dataset`。

改动二：`shuffle` 正常应该换成 `False`。评估的时候不需要再去打乱数据，顺序遍历一遍就够了。

至于 `drop_last`，它的意思就是删除最后一批。这个操作留着就行，不用动它——数据是逐批进模型的，批次结构保持一致，统计口径也简单。

> **提示**：改造 `get_dataloader` 的正确做法是把它升级为"路径 + shuffle 都可传参"的通用函数，而不是在评估函数里再复制一份。这里为了讲清评估流程先手写一遍，实际工程里把参数提上去、两边共用同一个函数更好维护。

#### 加载训练好的模型权重

第三步不用再训练了，直接加载模型。先给路径，从 `./model/` 挑一个：

```python
# 3. 加载训练好的模型
path = "./model/classification3.pth"   # classification 那一系列，选 3

my_model = MyModel().to(device)
my_model.load_state_dict(torch.load(path))
```

路径这里是 `./model/classification` 加编号，保存的时候有 1、2、3，选哪一个？肯定选 3，因为它是最后一轮训练留下的。

实例化模型以后要 `.to(device)` 搬到设备上，再用 `load_state_dict` 加载参数，权重本身由 `torch.load` 读进来。这两步的顺序别混：先有模型结构，再把参数塞进去。

#### 初始化评估参数与评估模式

第四步初始化评估参数，就两个变量：

```python
# 4. 初始化评估参数
correct = 0    # 预测正确的样本数
total = 0      # 预测的总样本数
```

它们分别记录预测正确的样本数和预测的总样本数。一会儿算准确率的时候，拿 `correct` 除以 `total` 就可以了。

第五步把模型设为评估模式，一行就够：

```python
# 5. 设置模型为评估模式
my_model.eval()
```

> **注意**：`model.eval()` 不是可有可无的礼貌动作。它会把 Dropout 关掉、把 BatchNorm 切到用滑动统计量的推理行为，评估结果才可复现。忘了它，同一份权重的评估分数会随批次抖动。

#### 逐批预测与统计

第六步是迭代预测：通过数据加载器拿到每批次数据，预测并统计。外层循环用 `enumerate` 包住数据加载器，再套一个 `tqdm` 进度条，`start=1` 让批次从第 1 批开始显示，第一批、第二批看着顺眼：

```python
# 6. 迭代预测
for i, batch in enumerate(tqdm(my_dataloader, start=1), start=1):
    # 6.1 将参数移动到 GPU
    input_ids = batch["input_ids"].to(device)
    token_type_ids = batch["token_type_ids"].to(device)
    attention_mask = batch["attention_mask"].to(device)
    labels = batch["labels"].to(device)

    # 6.2 不计算梯度
    with torch.no_grad():
        # 6.3 前向传播
        output = my_model(input_ids, token_type_ids, attention_mask)
        # 6.4 获取预测结果：取概率最大的那一类
        temp = torch.argmax(output, dim=-1)
        # 6.5 统计预测正确的样本数
        correct += (temp == labels).sum().item()
        # 6.6 统计总样本数
        total += len(labels)

    # 6.7 每隔 20 步输出一次预测进度
    if i % 20 == 0:
        # 6.7.1 打印平均准确率
        print(f"acc: {correct / total:.4f}")
```

六个子动作逐个说。

6.1 把四个参数搬到 GPU：`input_ids`、`token_type_ids`、`attention_mask`、`labels`，一个都不能落下。前面三个是模型的输入，`labels` 是拿来对答案的。

6.2 不计算梯度，写成 `with torch.no_grad():`。预测的时候不需要反向传播，也不需要保留计算图，把这两件事省掉，显存占用和耗时都明显下降。

6.3 前向传播，把前面三个参数交给模型，拿到 `output`。

6.4 获取预测结果：`torch.argmax` 在类别维度上取概率最大的那一个下标，`temp` 就是模型给出的类别。

6.5 统计预测正确的样本数：`(temp == labels).sum().item()`。先做逐元素的相等比较，得到一个布尔张量，`.sum()` 把 `True` 累加成人话数字，`.item()` 把它从张量里取成 Python 数值。

6.6 统计总样本数，直接 `total += len(labels)`。这一批有 8 条，`total` 就加 8。

6.7 输出预测进度，间隔 20 步打印一次，判断条件就是 `i % 20 == 0`。

6.7.1 打印平均准确率，`f"acc: {correct / total:.4f}"`，保留 4 位小数。

> **易错点**：`correct` 累加的是"这一批里预测对的个数"，分母 `total` 累加的是"已经处理过的样本总数"。分子分母都只在各自的位置更新一次，才不会出现"分子是全量、分母是单批"这种把准确率算到 8 倍的错误。

#### 解码样本做人工核对

准不准光看数字还不够，我们再补一个动作：解码第一个样本的文本，并把预测结果打印出来。这一步靠的是分词器的 `decode`。分词器能把词序编号映射回文本内容，`input_ids` 里放的就是文本对应词汇表中的编号，也就是词序编号。

```python
    # 6.7.2 解码第一个样本的文本并打印预测结果
    text_list = my_tokenizer.decode(
        input_ids[0],                       # 取这一批的第 1 条样本
        skip_special_tokens=True,
        clean_up_tokenization_spaces=True,  # 兼容格式
    )
    print(f"原始文本: {text_list}")
    print(f"预测标签: {temp[0]}")
    print(f"真实标签: {labels[0]}")
```

`input_ids[0]` 取出这一批里的第一条样本。`skip_special_tokens=True` 把 `[CLS]`、`[SEP]` 这些特殊标记过滤掉，`clean_up_tokenization_spaces=True` 用来兼容格式。

打印三行：`原始文本`、`预测标签`、`真实标签`。真实标签那行其实可以不写，因为每 20 批已经打印过准确率了，后面这个动作纯属可选的观察手段。它的意义是：这 20 批里的第一条样本到底被模型看成了什么？把文本还原出来，我们自己用眼睛判断一下它到底是好评还是差评。

> **提示**：如果不想只看每一批的第一条，把 `input_ids[0]` 里的 `0` 换成循环变量 `i`，再把这组打印动作整体加一层循环，就能把每一批的第一条都过一遍。想看全部样本同理。

### 跑一次评估

最后调用评估函数跑起来：

```python
# 测试
evaluate_model()
```

右键执行。模型评估的动作就启动了：先在这里边搞一个加载器拿到数据，再把模型设成评估模式，然后一条条做下去。整个过程很快，进度条从 89、90 一直涨到 100 就结束了。

这轮跑出来的准确率大概是 0.87、0.88、0.88，八十几，接近 90。对于这样一个迁移学习方案来说，还不错。

被解码出来的那条样本是这样的：

```text
原始文本: 买了一段时间了一直都在用，很不错，用盘重新做了新系统，机子运行很快，也很稳定。
预测标签: 1
真实标签: 1
```

这句话一看就是好评，"很不错""运行很快""也很稳定"，预测 1、真实 1，对上了。

真正值得记一笔的是另一件事：后面连着看下去，预测对了、预测对了、预测对了、预测对了——每一批次的第一条都是对的。这个现象本身挺有意思，它说明模型在这个数据集上已经学到了相当稳定的判别能力，同时也提醒我们：单看几条样本的直观感受，永远不能替代 `correct / total` 这个统计量。

### 迁移学习到底迁移了什么

中文文本分类到这里就告一段落，最终交付的就是"预测是多少、真实是多少、准确率是多少"这一堆东西。现在回头把整条链路说清楚。

先看数据集处理做了什么事。入口是 `load_dataset` 加载数据，加载方式有两种。数据进来以后按每批次处理：截断、补齐、设最大长度，最后返回四个结果：

| 返回项 | 含义 | 在模型里的作用 |
| --- | --- | --- |
| `input_ids` | 词序编号（词汇表索引） | 模型的输入 token 序列 |
| `token_type_ids` | 句子类型编号 | 区分句子对中的第一句/第二句 |
| `attention_mask` | 注意力掩码 | 标记哪些位置是真实 token、哪些是补齐 |
| `labels` | 真实标签 | 计算损失与评估准确率 |

再看 BERT 是怎么迁移过来的。我们从头到尾自己写的模型，其实只有一行代码——就是那一个 `Linear`，把 BERT 处理后的 768 维转成几维？转成 2 维，后面再接一个 Softmax 映射成概率。就这么点东西。

```text
                 输入文本
                     |
                     v
        +--------------------------+
        |   BertTokenizer 分词编码  |
        +--------------------------+
                     |
                     v
    +--------------------------------------+
    |  BertModel 预训练权重（冻结/微调）    |
    |  输出 last_hidden_state  (B, L, 768) |
    +--------------------------------------+
                     |
                     v      A 段：BERT 的语义特征
        +--------------------------+
        |  Linear(768 -> 2)        |   B 段：自己写的下游层
        +--------------------------+
                     |
                     v
        +--------------------------+
        |  Softmax -> 二分类概率    |
        +--------------------------+
```

这张图把程序里的 A、B 两段标了出来。A 段在 `with torch.no_grad()` 里跑的是什么动作？是 BERT 模型处理后得到的结果。B 段就是自己的模型，拿着 A 给出来的 768 维，映射成我们要的 2 维。所以 A 是预训练模型，B 是把 BERT 输出的稠密向量——可以理解成整句话的语义信息——抽取出来，再降维到 2 维。放到中文填空案例里，B 段输出的就不再是 2 维，而是词汇表大小那么多维，道理完全一样。

### 二分类的损失函数辨析

课间有同学问了一个细节，这里值得单独说清楚。

我们用的损失函数是哪个？`CrossEntropyLoss`。有人就在这里纠结：做的是一个二分类问题，二分类不是该用 BCE 吗？

> **提示**：BCE 是我建议的写法，我没说百分之百必须用它。二分类可以用二分类的 BCE 交叉熵，多分类可以用 `CrossEntropyLoss`。但二分类问题能不能用 `CrossEntropyLoss` 这种思路来做？完全可以。

所以你想用 BCE，改一行代码就行，把损失函数换成 BCE，别的都不用动。

更值得记的是后面那个问题：如果将来做的不是二分类，而是三分类、四分类或者更多类，损失函数需要跟着改吗？不需要。

> **结论**：类别数变了，动的是模型那一层，不是损失函数。三分类就把那行 `Linear` 的输出写成 3，四分类写成 4，`CrossEntropyLoss` 原地不动。

```python
# 二分类
self.linear = nn.Linear(768, 2)
# 三分类
self.linear = nn.Linear(768, 3)
# 四分类
self.linear = nn.Linear(768, 4)
```

### 一条可以复用的工程流水线

最后把整件事折叠起来看一遍。导包不用说了，上来先做分词器和预训练模型——这两步就是迁移学习本身。除了这两步，剩下的流程跟完全自己从头做是一致的。

```text
    加载数据  ->  数据预处理  ->  数据加载器
                                     |
                                     v
                              创建模型（搭建）
                                     |
                                     v
                                 训练模型
                                     |
                                     v
                                 模型评估
```

推进顺序就是这样：先拿到数据加载器，然后把模型搭好，搭好以后训练，训练完再预测。

如果是在实际开发里做这件事，而且工程排期紧，你又能确保自己的模型结构没有问题，那么"测试模型"那个环节——就是验证激活函数那一小段代码——完全可以删掉，不用测。但有两个动作你必须写：**训练**和**评估**。

> **结论**：未来写代码的流程就是这样。先拿数据加载器，再把模型搭好，然后训练，最后预测，搞定。后面两个案例纯按这个思路走。

> 💡 **承前启后**：完成对「Day12-03.迁移学习_中文分类案例_模型评估(掌握)」的理解后，下一章我们将深入探讨「Day12-04.迁移学习_中文填空案例_数据预处理(掌握)」，进一步完善知识图谱体系。

---

## Day12-04.迁移学习_中文填空案例_数据预处理(掌握)
> 对应分集：P176 | 原始标题：《Day12-04.迁移学习_中文填空案例_数据预处理(掌握)》

第二个案例来了：迁移学习的中文填空。你跟第一个案例对比一下就会明显发现，思路是不是都一样？而且我要先告诉你一句话——第二个案例和第一个案例用的数据源也是刚才那一套，一模一样的 CSV。那第三个案例呢？第三个案例用的数据源还是那一套。所以就是拿同一份数据，我们来做三个任务。

中文填空你有没有印象？我们之前做过一个完形填空，`中括号 mask` 那个，还记得吧。但之前演示的 mask，我只给你掩了一个，一个句子里就一个空。那如果一会儿有多个空、我也要填，怎么做？这就是接下来要做的具体案例。

### 完形填空为什么是一个分类问题

先看一句话：中文语料完形填空，它是一个什么问题？分类问题。

为什么说它是分类问题？因为这个空里面要填什么字，你得给出每种可能的概率。刚才做中文文本分类，好评差评，那是一个二分类问题。现在做填空，意思是什么呢？我们切到词典上看看。

我们用的模型是 `bert-base-chinese`，把它的 `vocab` 拉出来，如果是做完形填空、往这个空的位置去填充字，这里边每一个字都有可能。所以它的概率应该映射成多少个？两万多个。词汇表里有多少个字，你就有多少个概率。

> **定义**：中文完形填空（cloze / fill-in-the-blank）在建模上就是分类任务，只不过类别数不是 2，而是词汇表大小。BERT 中文预训练模型的词汇表有 21128 个 token，那么这个位置的预测就有 21128 种可能。

那接着往下：使用迁移学习来做这件事。跟刚才做中文分类的路径一样——预训练模型处理后会给咱们两万多个概率，你要做一件事，把这个概率经过 Softmax 映射成一个具体的字，填充到那个位置上，就完事了。

```text
    "这个 [MASK] 扶梯很好用"
                 |
                 v
    +-------------------------------+
    |  BertModel  (bert-base-chinese)|
    +-------------------------------+
                 |
                 v
      logits/prob  shape = (21128,)
                 |
                 v
        Softmax  ->  21128 个概率
                 |
                 v
      取概率最大的那个字 -> "电动"
```

这张图说明填空任务的输出形状：一个被掩码的位置，吐出的是覆盖整个词汇表的 21128 个概率，最后取最大的那个下标，就得到了要填的字。

### 掩码位置该掩谁

数据介绍这一段还用讲吗？不用了。`train`、`test`、`validation` 就是刚才一直在玩的那一堆，不再重复。

那上来先看句子本身。你在脑子里把它补一遍：比如"自动扶梯"——手动扶梯、电动扶梯，这些其实都可以。有人接"雷动扶梯"，雷动是啥意思？你往上一站然后上面开始劈你？行了，回来。再看另一个句子，我把超市的"市"字盖一下：有超什么呢？超兽？超人？超熊？supermarket 嘛，你可千万别。

所以这个句子如果随机 mask 之后，后面做填空就行了。但是这里有个问题：我给你的句子，它本身是完整的。那这个掩码你该掩谁？电动？扶梯？还是别的什么？

> **结论**：案例里我们设置一个随机数，把句子中的某几个词、某一个词随机掩掉，然后再做预测。因为生成样本时底层本来就是对每个字都会做预测，所以你可以随机遮掩，后面再来预测。

一会儿咱们玩的其实就是这个，跟以前玩过的完形填空是一个东西。

### 导包与预训练模型加载

先把导包这一块立起来，跟上一个案例对比着看：

```python
import torch
from torch.utils.data import DataLoader
from datasets import load_dataset
from transformers import (
    BertTokenizer,
    BertModel,
    AdamW,
    get_linear_schedule_with_warmup,
)
from tqdm import tqdm
import time
```

这里边有一处值得停一下：优化器用的是 `AdamW`。对，就是这个带权重衰减解耦的版本。

接下来加载模型：

```python
# 设备
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 分词器与预训练模型，两个都是 bert-base-chinese
my_tokenizer = BertTokenizer.from_pretrained("bert-base-chinese")
my_pretrained_model = BertModel.from_pretrained("bert-base-chinese")
```

上下两个模型都是 `bert-base-chinese`，跟上个案例完全一致。

进来以后第一件事是 `collate_fn2`。`fn2` 是啥意思？就是第二个处理函数。这个第二个处理函数跟刚才那个处理函数不一样了。但是没关系，它这儿不一样，后面的动作你看——`demo`、`test`、`dataset`，下面这个测试动作、`my_dataloader` 这一堆，是一模一样的。

> **注意**：这里的长度变了。刚才做中文文本分类的时候，我们把所有长度都干成 300；现在做填空，因为要掩码，它只要多少个字？32。你的句子再长，我也只取前 32 个内容。

这个案例要做的事先定清楚：随机遮掩 mask，然后把这个动作实现出来。

### demo02 与固定掩码位置

新建第一个脚本，叫 `demo02`。案例名称写清楚：迁移学习案例，中文的完形填空。

然后写第一句设计说明：

> 我们假设每个句子的第 16 个位置设置为 mask，基于其余字符对该位置做填充。

为什么选 16 这个位置？因为我们刚才说假设句子长度都是 32，那要预测的就是中间这个位置，16 差不多就在正中。你写第 10 个、第 9 个、第 20 多个，都无所谓，你开心就好。这个值写谁，一会儿算索引的时候把它改过来就行。甚至还能干一件事：把它改成随机的。那样打印的时候就告诉它，刚才随机生成的位置是几，这个位置填充的字应该是谁。

```text
   索引:  0     1    2     3     4    5     6     7   ...   16   ...  31
        [CLS]  这   个   电   动   扶   梯   很  ... [MASK] ... [PAD]
                                                     ^
                                                     |
                                        固定掩码位置，只预测这一个
```

这张图说明掩码位置的索引约定：序列第 16 号位置被替换成掩码，模型只对这个位置做预测。

### 从上一个案例复制导包与公共部分

导包、加载数据、预处理这一堆，你觉得跟刚才玩的中文文本分类是不是也一样？很类似。所以你听我这么讲：可以贴，但不能全贴。

导包这一长串一直到 `DataLoader` 这儿，我其实是想全贴过来的，为什么？因为这个动作高度一样。包括下面这个 `DataLoader` 加载器、`load_dataset` 加载数据、`time` 时间、`tqdm` 进度条、`BertTokenizer` 和 `BertModel` 加载 BERT 模型，还有 `AdamW` 这个优化器，往下还有 `pretrained` 那一堆。有没有发现跟昨晚那个是一样的？

GPU 这块也一样，`device` 照旧。`my_tokenizer` 和 `my_pretrained_model` 那几行代码也都一模一样，玩的还是 `bert-base-chinese`。

那哪儿不一样？

有一处要删：`file_to_dataset` 这个函数。它是干啥的？这个函数里面演示的是 `load_dataset` 的另外两种下载方式——一种是给 CSV 后缀这种方式，另一种是给一个点当路径这种方式。这个函数怎么演示过了，这里还要吗？不要了。我们这儿不用演示，那个编号 3 的代码块就不要了。

删掉之后，剩下那个处理函数就变成"土豆 4"。为了跟左边的中文分类案例以示区分，把这个位置的 `collate_fn1` 换成 `collate_fn2`。换完下面会直接报错——因为这里边是不是没有 `fn1` 了？不管它，先丢到这儿，接着往下改。

### collate_fn2 的整体结构

这个处理函数要对每批次的数据做手脚。先看数据长什么样：数据里面还是标签和文本，一次还是 8 条。

```python
def collate_fn2(batch):
    # 1. 提取文本 —— 标签不要了
    sentences = [item["text"] for item in batch]

    # 2. 批量编码
    inputs = my_tokenizer(
        sentences,
        max_length=32,          # 长度由 300 改成 32
        truncation=True,
        padding="max_length",
        return_tensors="pt",
    )

    # 3. 取出三项
    input_ids = inputs["input_ids"]
    token_type_ids = inputs["token_type_ids"]
    attention_mask = inputs["attention_mask"]

    # 4. 固定第 16 个位置设置为 mask
    labels = input_ids[:, 16].clone()                      # 4.1 先存真实 token_id
    input_ids[:, 16] = my_tokenizer.mask_token_id          # 4.2 再替换成掩码
    labels = labels.reshape(-1)                            # 4.3 转成张量（已是一维）

    return input_ids, token_type_ids, attention_mask, labels
```

这段就是整个预处理的核心，四个大动作按顺序说。

第一件事就是标签不要了。为什么呢？因为你这里边只拿句子，一会儿把句子中某个字给它掩一下。原来的分类标签在这里没有用处，所以提取文本时不再取 `label` 字段，那行直接删掉，只保留 `sentences`。

第二件事是批量编码。`sentences` 这个变量不变，但出来的长度得变——咱们不要那么长，要 32 个。所以 `max_length=32`，其余参数不动。

第三件事是把 `input_ids`、`token_type_ids`、`attention_mask` 三项取出来。这里注意听：把下面的 `raw_label` 直接删了就行。为啥？因为现在做的是掩码——把中间某个字，比如把"电"这个位置换成一个 mask，一会儿让你去填充——那么原来左边那个分类标签是不是已经没有用了？所以第四步里，分类标签的部分不要。

标签不要了，那返回值肯定就少一项，没有 `labels` 了。但是别着急，拿到这几个数据以后，我们先打印看看。

### 三处打印与掩码验证

先打印第一个句子。第一句掩码前长什么样？看 `input_ids`。这里边是几个句子？8 个句子，因为它的形状是 $(8, 32)$。

```python
    print("掩码前:", input_ids[0])
```

`input_ids[0]` 写个 0 是什么意思？就是第一个句子。掩码前它对应的编号全在这儿，每个字对应的编号都在这一行里。一会儿你就能看看有没有掩码成功。

掩码处理之后，再打印一次同一行，就有了对照：掩码前打印 `input_ids[0]`，掩码后也打印一次 `input_ids[0]`，两次一比就知道到底掩没掩成功。

最后是标签本身。真正的标签不是打印出来的 `input_ids`，而是下一步取出来的 `labels`，所以 `labels` 要单独打印一次：

```python
    print("labels:", labels)
```

这一步很关键，因为 `labels` 打印出来的结果，会和 `input_ids` 里被掩掉的那个位置严格对应上——掩掉哪个字，标签里存的就是那个字的真实编号。

### 构造填空任务的标签

第四个动作是构造标签：

> 案例中假设将第 16 个位置设置为 mask。

所以我们要做几件事。

#### 先取真实 token_id

4.1：先获取第 16 个位置的词汇的真实 `token_id`。

你知道为什么要先拿真实的 `token_id` 吗？因为我得知道你真实这个字的编号是啥。一会儿我预测出来一个编号，拿着它跟真实编号做对比，准确率是不是就知道了？就是这个意思。

那怎么取？用切片：

```python
    labels = input_ids[:, 16].clone()
```

`input_ids` 后面的冒号是什么意思？就是这 8 条里每一条我都要。这里写 16，意思就是这 8 条数据每个都拿它的第 16 个位置。

> **易错点**：写 16 拿到的其实不是第 16 个 token，而是第 17 个。序列最前面有 `[CLS]` 这个特殊 token 占掉了索引 0，所以索引 16 对应的是原文的第 16 个字。这一点在数位置的时候特别容易错，数的时候务必把索引 0 上的 `[CLS]` 一起数进去。

这句话的意思，最简化地说就是：句子 1 的第 16 个位置的真实值对应的 `token_id`，句子 2 的第 16 个位置的真实 `token_id`，点点点。8 个句子就是 8 个真实 ID。

`reshape(-1)` 是做维度转换，把结果压成一维；`.clone()` 是克隆一份——因为下一步我们要把 `input_ids` 里那个位置改掉，不克隆的话标签会跟着一起被改，那标签就不是真实值了。

现在 `labels` 里就是真实 ID、真实 ID、真实 ID……你可以认为这就是你的标签。

> **提示**：整句话操作下来，`labels` 的形状是 `(8,)`，等着跟模型的预测结果 `(8,)` 做对比。标签长度和批次里句子条数严格一致，分母才是 8 而不是 8×16。

#### 把该位置替换为掩码

4.2：将第 16 个位置的 `token_id` 设置为 mask。

```python
    input_ids[:, 16] = my_tokenizer.mask_token_id
```

`input_ids[:, 16]`，这一手对那 8 条一块全做处理。右边要给掩码的编号，写法有四种。

**写法 1**：用 `get_vocab()` 拿词汇表，再按键取。

```python
    input_ids[:, 16] = my_tokenizer.get_vocab()[my_tokenizer.mask_token]
```

`get_vocab()` 就是拿到你的词汇表。`bert-base-chinese` 那个 `mask` 的编号是多少？103。

> **提示**：为什么是 103？因为词汇表从 1 开始数，掩码标记排在第 104 个，那它的编号就是 103。这个是从 1 开始计数与从 0 开始计数之间的差，别在这里绕晕。

**写法 2**：把键直接丢进 `get_vocab()` 里取。

```python
    input_ids[:, 16] = my_tokenizer.get_vocab()["[MASK]"]
```

**写法 3**：直接调 `mask_token_id`。

```python
    input_ids[:, 16] = my_tokenizer.mask_token_id
```

**写法 4**：写死 103。

```python
    input_ids[:, 16] = 103
```

你要非跟我较真，还有写法 4。但 103 我们不写，写出来太 low 了。

为啥不写 103？万一我这个模型换了，或者模型版本升级了，它的词汇表里 mask 现在不是 103 而是 301 或者别的，有没有可能出现这种情况？有可能吧。所以这里得换一下，用代码去问分词器，别自己硬编码。

这里我们直接用写法 3，因为更短一点，这几种说法结果是一样的。

#### 标签转张量并返回

4.3：将上述的标签转成张量，然后返回。

```python
    labels = torch.tensor(labels)
    return input_ids, token_type_ids, attention_mask, labels
```

到这儿，预处理这一块就结束了。

### 数据加载器与测试代码

然后是 `get_data_loader`：

```python
def get_data_loader():
    dataset = load_dataset(path="./data/train.csv", split="train", max_length=32)
    return DataLoader(
        dataset,
        batch_size=8,
        shuffle=True,
        drop_last=True,
        collate_fn=collate_fn2,   # 换成 collate_fn2
    )
```

CSV、train 这一堆数据都不变，下面的 `train_data`、`batch_size=8` 这一堆也都不变，唯一要换的就是 `collate_fn2`。

接下来直接测试。刚才那份测试代码里有一个动作，`Ctrl+C` 记得吧？把它拉过来，取消注释：

```python
# 1. 创建数据加载器并测试
train = get_data_loader()
for batch in train:
    print(batch)
```

右键走。

有没有发现一件事？之前写这接近 100 行代码要干一个多小时，现在写这 100 行代码分分钟结束。因为这一百行代码里几乎三分之二以上都是复制粘贴，自己在其中改改处理函数的逻辑就可以了。所以千万别眨眼睛，一睁眼就"哇，100 行代码"，其实都是套的。

### 看打印结果：掩码前与掩码后

跑起来以后看输出。

先看第一个：第一个句子掩码前。你帮我数数这个位置，第几个位置被掩了？是不是第 16？16 其实就是这里边的第几个位置？第 17 个。但是注意，你得这么数：`101` 是给你加的前缀，所以从 `[CLS]` 的 101 开始数，数到第 17 个，那个位置就是被掩的位置。

> **注意**：`[CLS]` 的编码是 101，它占掉序列第一个位置；`[SEP]` 是 102；`[MASK]` 是 103。数位置的时候必须把 101 这一段算进去。

那这个位置谁给我掩的码？再往后数数，那个 `891` 在这儿，它是不是被掩成了 103？对，用 mask 替代了。后面的 `5116796814` 什么的，跟原来是一模一样的。

停一下——刚刚那段是掩码前还是掩码后？掩码前。所以我们要让掩码前的那个值当真实标签，也就是真实值。

```text
   掩码前 input_ids[0]:
   101, ... , 891, ...                      <- 891 是第 16 号位置的真实 token
                    |
                    |  input_ids[:, 16] = 103
                    v
   掩码后 input_ids[0]:
   101, ... , 103, ...                      <- 被替换成 [MASK]

   labels = input_ids[:, 16].clone()  = [1912, 122, 4960, ...]
                                        ^^^^^^^^^^^^^^^^^^^^^
                                        8 条句子第 16 号位置的真实 token_id
```

这张图画的是掩码这一步的前后关系：真实 token 被 `clone()` 存进 `labels`，原位置被改写成 103。

现在再看打印出来的 `labels`。仔细看这个操作：我把这个值存储到了 `labels` 里边，`labels` 的第一个是 1912。也就是说，第一个句子的 1912 被掩了；然后是第二个句子、第三个句子、第四个、第五、第六、第七、第八。所以这个操作拿到的就是 8 个句子中被掩码的内容的真实编号。

那模型一会儿要干嘛？你要预测这个位置的值是多少。预测出来会得到一个词的标签，拿这个预测结果跟我这 8 个真实结果去对比：第一个句子就跟第一个比，第二个句子就跟 122 比，第三个句子就跟 4960 比，依此下去。8 个句子预测 8 个值，分别和这 8 个值对比，对了几个、错了几个，准确率就出来了。

> **结论**：这就是填空任务的评估逻辑——掩掉的位置取真实 `token_id` 当标签，模型预测同一位置的类别，两者在批次维度上逐一对应比较。

再往下看，下面这堆打印的是谁？是掩码后的数据。这就是第一批次的数据，`101 ... 103 ...` 这样一排编号。一共多少条？一批 8 条。

而打印出来的 `labels` 是：`tensor([1912, 122, 4960, ...])`，跟上面单独打印 `labels` 那个值是一模一样的。

```text
   掩码前打印     ->  input_ids[0] 里第 16 号位置是真实编号
   labels 打印    ->  8 个句子的真实编号：1912, 122, 4960, ...
   掩码后打印     ->  input_ids[0] 里第 16 号位置变成了 103
```

这张图把三处打印的关系对齐：掩码前给真实值，`labels` 装的就是这些真实值，掩码后同一位置换成 103。

### 打印的清理与下一步

既然 `labels` 和掩码后的输出已经能对上，前面那些打印就都不需要了。第一个句子掩码前、最后一个句子掩码后这些全部注释掉。我告诉你，你就打一个东西就行：把 `labels` 打出来，别的都不看，就看 `labels`。

我的 `labels` 长这样，这 8 个是真实的。一会儿你在这 8 个位置做预测，预测完了跟我这 8 个做对比，对了几个、错了几个，算出准确率。

第一波数据处理到这儿就做完了。那猜猜接下来要写啥？下一节就是搭建模型，模型搭完后面是训练和预测，思路其实都差不多。保存一下。

> 💡 **承前启后**：完成对「Day12-04.迁移学习_中文填空案例_数据预处理(掌握)」的理解后，下一章我们将深入探讨「Day12-05.迁移学习_中文填空案例_模型搭建(掌握)」，进一步完善知识图谱体系。

---

## Day12-05.迁移学习_中文填空案例_模型搭建(掌握)
> 对应分集：P177 | 原始标题：《Day12-05.迁移学习_中文填空案例_模型搭建(掌握)》

接下来要做的事就是搭建下游网络模型，把刚才处理后的 768 维转成……看这儿，`vocab_size` 是什么意思？就是词汇表的大小。有多少个词，就转成多少个概率。`bias` 呢，写 `False`，不考虑偏置——因为这里边参数已经很多了，偏置可以不考虑。

### 第五个动作：BERT 下游填空模型的实现

这一段的编号顺序说一下：前面从加载数据到预处理是若干个动作，到了这里就是第五个操作，叫自定义下游模型。这个模型是基于什么的？基于 BERT 的填空任务模型。

那问你一个问题：怎么做？复制粘贴，还是自己手写？复制粘贴。

在编辑器右侧有 6 到 7 那一段操作，直接拉过来，跟刚才一样。当然我们这儿不叫 6，叫 5，而且比刚才还少一步。按住 `Ctrl+Shift+减号` 全部折叠你就看出来了：把最开始的加载数据那一段干什么了？删掉了。所以代码量比分类案例还短。

```python
class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        # 1. 全连接层：BERT 的 768 维 -> 词汇表大小
        self.linear = nn.Linear(768, my_tokenizer.vocab_size, bias=False)
        # 2. 预训练 BERT
        self.my_bert_model = my_pretrained_model

    def forward(self, input_ids, token_type_ids, attention_mask):
        # 前向传播，参数还是这几个
        with torch.no_grad():
            bert_output = self.my_bert_model(
                input_ids=input_ids,
                token_type_ids=token_type_ids,
                attention_mask=attention_mask,
            )[0]
        # 只取出第 16 个位置的预测
        output = self.linear(bert_output[:, 16])
        return output
```

#### 全连接层的输出维度

第一个动作 `__init__` 不用说了，初始化父类肯定是要的。

第二个动作是全连接层。这里跟分类案例的区别就在这一行：分类是把 BERT 的 768 维转成 2 维概率，这里不是，是转成多少维？转成 21128 维。21128 是什么？就是词汇表的大小。

```python
self.linear = nn.Linear(768, 21128)        # 这样写不合适
self.linear = nn.Linear(768, my_tokenizer.vocab_size, bias=False)   # 正确写法
```

直接写 21128 不合适吧？万一它将来词汇表变了呢？所以用 `my_tokenizer.vocab_size` 来取，这就是词汇表大小。

`bias` 等于什么？等于 `False`。你也可以按住 Ctrl 点进 `nn.Linear` 看它的签名，会发现 `bias` 默认是 `True`，也就是默认考虑偏置。我们在这里不考虑偏置，为什么？因为参数已经很多了。

> **提示**：输出维度用 `my_tokenizer.vocab_size` 而不是硬编码 21128，和上一节掩码编号用 `mask_token_id` 而不是硬编码 103 是同一个道理——换模型、换版本的时候，代码不用跟着改。

#### 前向传播只取第 16 个位置

接下来是前向传播，这里要动点手脚。

第一个动作：把前面那一段打开，前向传播的参数还是一样，它还是要传 `input_ids`、`token_type_ids`、`attention_mask` 这些值。

传完以后，不计算梯度这一套照旧。下面那一坨，`bert_output`，把 BERT 模型和那几个值扔进来：

```python
bert_output = self.my_bert_model(input_ids, token_type_ids, attention_mask)[0]
```

扔进来以后，停，别着急。这个位置原来是使用 BERT 的输出做分类，我们不是分类。我们的第三步是什么？

> 只取出第 16 个位置的预测概率值。

为什么只取第 16 个？因为我们刚才假设句子中只有第 16 个位置被掩码了，所以我们只预测这一个。那如果想要 16 和其他几个位置都被掩码呢？其实很正常，那你掩了几个就取几个就好了。

```python
output = self.linear(bert_output.last_hidden_state[:, 16])
```

`bert_output` 里面的 `last_hidden_state` 是最后一层隐藏状态，形状是 $(B, L, 768)$。这个切片怎么写？`[:, 16]`——冒号的意思是这一批有 8 个句子，8 句话都要；后面的 16 就是每一句话都取第 16 个位置。

所以这个动作就是：8 个句子，每批只取批次中这 8 个句子的第 16 个位置，然后做返回。返回的这个概率只有第 16 个位置的概率，这个概率的最后一维是多少？21128。那再拿它算成概率、取概率最高的那一个值，是不是就结束了？

> **结论**：填空模型跟分类模型的差别就两处——初始化里输出维度从 2 变成 `vocab_size`，前向传播里从"整句分类"变成"只取被掩码位置的表示再做映射"。初始化没怎么动，前向传播换成了取第 16 个位置，其他不变。

### 第六个动作：模型结构的单批前向验证

折叠掉模型这一段，该测试了。这个测试代码怎么写？把原来那段展开，直接对照着改。

```python
def use_bert_model():
    # 1. 创建数据加载器
    my_dataloader = get_data_loader()

    # 2. 创建模型并移动到设备
    my_model = MyModel().to(device)

    # 3. 遍历数据加载器
    for batch in my_dataloader:
        input_ids = batch[0].to(device)
        token_type_ids = batch[1].to(device)
        attention_mask = batch[2].to(device)
        labels = batch[3].to(device)

        # 4. 前向传播
        output = my_model(input_ids, token_type_ids, attention_mask)
        print(output.shape)
        break

use_bert_model()
```

第一个动作，`get_data_loader` 这个不变。第二个动作，创建模型并移动到设备上，不变。第三个动作，遍历 `data_loader` 拿到这堆数据，把数据切到 GPU，不变。再往下，前向传播得到数据，也是 OK 的。

#### 打印 output 的形状

接下来有一个建议：当你 `print(output.shape)` 的时候，我建议你不要只打这一行，你要打印就换一行——**强烈建议换一行**。上面一行打 `output.shape`，下面这一行我们不打 shape，我们直接把 `output` 打出来。

```python
        print(output.shape)
        print(output)          # 一行打印形状，一行打印内容
```

为何要分行？因为 `output` 里是一坨数据，打印出来很长，和形状挤在一行里根本看不清。

我想说的是：`output.shape` 你看到的一定是 `batch_size` 那个 8 和多少？不是 2。因为上次做中文分类是一个二分类，这里边是预测概率，它应该是 21128。所以形状是 $(8, 21128)$。

#### 解决 labels 打印两次

然后加上 `break`，只测试第一批。把原来那个多余的测试动作去掉之后，右键跑一遍。

等一下——`labels` 这个玩意儿怎么给我打两次？

别忘了一件事：你的 `labels` 在数据整理函数里做过操作。走到上面的 `get_data_loader`，这个 `get_data_loader` 上面调用了你的数据整理函数，而 `labels` 在那个函数里是有打印的。所以 `labels` 在预处理里打了一次，后面在看 `output.shape` 的时候又打了一次，加起来就是两次。

处理办法很简单：把两处打印注释掉一处就行，随你选哪一处。注释以后再看输出，`labels` 代表的就是真实标签——8 个句子中被掩码的部分。

#### 输出形状的解读与下一步

清理完打印再跑一遍，就没有两个 `labels` 了。

接下来看 `output.shape` 是什么？$(8, 21128)$。这是什么意思？

这步就是：8 个句子中第 16 个位置，对词汇表中每个词各有一个概率。然后你要做的事就是把这个 $(8, 21128)$ 拿去找到概率最大的那个，它就变成 $(8, 1)$。$(8, 1)$ 是什么意思？就是 8 个句子、每个句子这个位置应该是谁。然后你跟上面那份 `labels` 一对比——有几个一样、有几个不一样，预测正确的个数不就出来了？

```text
   output                        (8, 21128)   每句话第16位对全词表的分数
      |
      |  argmax(dim=-1)
      v
   pred                          (8, 1)       每句话第16位最可能的那个字
      |
      |  与 labels (8,) 逐一比较
      v
   对/错 计数  ->  准确率
```

这张图把从模型输出到准确率的路径走了一遍：先是全词表打分，再取最大，最后跟真实标签逐条比对。

其实整个案例写下来你会发现，剩下那些动作和分类案例都高度相似。未来到工作中也基本是这样——你把一套业务玩通了，剩下再玩都大差不差。到这儿，关于模型搭建我们就做完了。再往下，就该做模型的训练了。

> 💡 **承前启后**：完成对「Day12-05.迁移学习_中文填空案例_模型搭建(掌握)」的理解后，下一章我们将深入探讨「Day12-06.迁移学习_中文填空案例_模型训练(掌握)」，进一步完善知识图谱体系。

---

## Day12-06.迁移学习_中文填空案例_模型训练(掌握)
> 对应分集：P178 | 原始标题：《Day12-06.迁移学习_中文填空案例_模型训练(掌握)》

模型搭完，接下来就是把它训起来。这个动作你一看就会发现跟中文文本分类那次差别不大——还是 20 轮存一次模型，把上一份训练代码直接拉过来，改几处核心细节就能跑。

先给你挖个小坑，看看你会不会自己跳进去。代码跑起来，日志一行一行刷出来，你有没有发现又是 20、20、20？有同学已经点头了。我告诉你，这是 20 **批**，不是 20 轮——`20 批、20 批、20 批`，好好听课，别趁我挖坑你就往里跳。

### 步骤七的位置：模型训练

按流程排下去，这里就是步骤七，名字叫**模型训练（Train Model）**。那不用说，一会儿第八步自然就是模型预测。

我把上一份代码里的 `train_model` 整段复制过来粘到这儿，注释换成这一讲的说法。粘过来之后，加载什么数据？加载训练数据，一会儿评估阶段就换成测试数据。剩下那一堆循环、打印，全都一样。

所以你一会儿要真正动手改的地方其实不多，就**三个地方**。

### 三处修改的清单

#### 修改一：过滤长度大于 32 的样本

第一个要改的是**过滤长度**，标准是大于 32。上一讲我们已经把数据的最大长度定为 32，所以这里要做一次筛选。

#### 修改二：保存模型名

第二个要改的是**存模型时的文件名**。这个不卖关子，后面直接说。

#### 修改三：训练代码主体的复用

第三个要改的落在训练体最后——准确说，是把存模型那行的模型名改掉。这一条之所以单独拎出来，是因为不改就会把上一个案例训出来的模型**覆盖**掉。

### 为什么要过滤长度大于 32 的样本

这一段是关键，你得听懂我在问什么。

前面我们在分词那一步已经设了最大长度 32，也启用了文本截断和填充。那我问你一句话：如果我原来真实的句子就三个字，你把它填充成 32 个字，然后我让你去预测第十个位置的内容——那个位置本来就是填充出来的，你去预测它有什么意义？

> **结论**：填充出来的位置做掩码预测，是在预测你自己造出来的内容，结果没有真实性可言。

所以我想要的是：**真实文本长度本身就大于 32 的样本**。这样掩码盖住的那个位置，一定落在真实字符上，预测结果才站得住脚。

有人会说，我不想这么玩，我就让它填充完再预测。那模型给你乱填一通你再预测，意义同样不大。基于真实长度在 32 以上的数据做掩码、再做预测，结果才比较真实。

### 优化一：数据集过滤的写法

正式写训练动作之前，先加一段**优化一**：过滤出长度大于 32 的样本数据。

训练数据集 `train_dataset` 上面已经拿到了，怎么过滤？用 `filter` 函数配一个 `lambda`：

```python
# 优化一：过滤出长度大于 32 的样本数据（按需启用）
train_dataset = train_dataset.filter(lambda x: len(x['text']) > 32)
```

`x` 代表数据集里的一行数据；`x['text']` 就是那一行的文本内容；`len(x['text'])` 就是这条文本的长度，条件写 `> 32`。

#### 这段过滤为什么可以先注释掉

有同学一看就急：你前面在分词时做了截断和补齐，这里又按 32 过滤，是不是前后矛盾？

不矛盾。因为这个数据集里每一条句子的长度本来就都大于 32。你可以自己数一数，看着像不够长的那些，前面也已经被截断和补齐的逻辑处理过了，所以过滤条件对当前数据来说是恒真的。

实际上真正开发时更顺手的做法是在数据预处理那一步就动手：把**填充**这个动作删掉，只保留**截断**，长度不到 32 的直接拿掉。或者再往前一步——在已经拿到 `sentence` 数据集对象那里，就把长度小于等于 32 的句子过滤掉，再往下走后续流程。

> **提示**：过滤长度这件事本身只是一行代码。做不做我说了不算，你说了也不算，要看需求和公司的业务。代码我给你留着，将来业务上需要启用、或者长度阈值要调整，把这段打开改个数就行。

所以这一段我暂时给你注释掉了。

### 优化二与优化三：模型对象、优化器与保存名

继续往下看展开的那段代码。

先是 `MyModel` 创建模型对象，接着**冻结 BERT 参数**——这一讲的模型参数是冻结的，不参与更新（"冻结"在语料里对应的是冻结 BERT 参数这一动作，具体冻结范围以你实际代码里的那几行为准）。然后创建损失函数，用 `CrossEntropy`。

优化器这里换了一个：前面那个案例让我用 `AdamW`，这一讲我用 `Adam`，行不行？行，其实是一样的。损失函数管的是怎么衡量误差，优化器管的是**怎么更新参数**，这俩各司其职。

再往下是启用训练模式，还是**三轮**，还是那一套时间统计、数据加载器、批次循环：把一批数据移到设备上，`forward` 算结果，算损失，然后几步更新参数——这一整坨跟前面一模一样。

到了取预测值、算结果、处理 `label` 的地方，也一样。真正不同的就一处：**保存模型名**。

> **易错点**：模型名不改，就会把上一个案例训出来的模型文件覆盖掉。

上一个案例做的是 `classification`，也就是分类；这一讲做的是**填空**。填空在 Hugging Face 那边的任务名是 `fill_mask`（就是 `fill-mask` 那个下划线写法）。所以保存路径里的任务名要从分类改成填空。

### 执行训练：先关掉那一屏 label

训练代码就绪，直接右键跑。跑之前有个小插曲：日志里 `label`、`label`、`label` 打了一大堆，那是因为代码里把 `label` 打印出来了，本意是想看看内容，但数据一多就刷屏。算了，不看了，关掉它，不然数据太多。

继续右键，走。这就是一次完整的训练过程。

如果你在这一版里没写过过滤动作，你会发现这个案例跟左边那个案例跑起来是一模一样的。别看它俩是两个案例——**模型训练的步骤和思路完全一致**。你可以把它当成一份固定代码：将来真到公司里要写一套，你没必要从零开始，把这份代码拿走，改一改直接用就行。

### 训练为什么快这么多

这个动作的训练比上一个明显地快。为什么？

因为计算量不一样。填空任务只需要在每个被掩码的位置上生成一个概率，然后取预测结果就可以了；而做文本分类的时候，你得对整个句子做分析。再看长度：上一个案例的 `max_len` 是 300，这个案例是 32。光数据长度就差了 10 倍，算力、计算量也就差了 10 倍。

| 对比项 | 中文文本分类案例 | 中文填空案例 |
| --- | --- | --- |
| 任务类型 | 分类（`classification`） | 填空（`fill_mask`） |
| 最大长度 `max_len` | 300 | 32 |
| 输出维度 | 类别数 | 词表大小 21128 |
| 参数冻结 | —— | 冻结 BERT 参数 |
| 优化器 | `AdamW` | `Adam` |
| 训练轮数 | 三轮 | 三轮 |

结果马上就出来了：准确率刷到 1，接着 0.88，看着挺高。最后一轮甚至弄了个 100% 出来。耗时多少？**28 秒**，三轮训练完，大概一分半不到。

看下左侧文件目录，模型现在是多了第三个文件——对了，说明这一讲的模型确实存下来了，没有覆盖前两个。

### 预训练模型的性价比与后面的路

你有没有发现，这个预训练模型比我们最早自己写的那些机器学习和深度学习模型好用得多？

我提前告诉你：等后面学到**大模型**，你会发现大模型比它更好用。那大模型是什么？就是参数量比它们更多。我们现在用的 BERT 系列，参数大概在 **1.17 亿**左右；而大语言模型有个门槛，是 **10B**，也就是**100 亿**参数，比这个数量级大得多。

### 训练这一步的复用结论

回头再看一遍：如果不做那两处改动（过滤 32、改模型名），中间这一整套动作是不是完全一样？

所以还是那句话——第一遍编程只有第一次学是难的，后面越学越简单。学习要去找**相通的地方、通用的地方**，不要每次都从零学。你要是一段通用代码隔了很久再看，连 `for` 循环都要从头学、参数什么意思都看不懂，那情绪崩溃就不应该了。把通用性的东西多敲几遍、掌握透，未来再看到类似的，就跟打草稿一样往里填代码，这才是比较好的学习方式。

> 💡 **承前启后**：完成对「Day12-06.迁移学习_中文填空案例_模型训练(掌握)」的理解后，下一章我们将深入探讨「Day12-07.迁移学习_中文填空案例_模型评估(掌握)」，进一步完善知识图谱体系。

---

## Day12-07.迁移学习_中文填空案例_模型评估(掌握)
> 对应分集：P179 | 原始标题：《Day12-07.迁移学习_中文填空案例_模型评估(掌握)》

训练跑完，文件目录里多了第三个模型。接下来是评估，也就是拿上面训好的模型在这个数据集上做一次检验。

不过先看一眼要复用哪份代码。你注意那个操作里写的内容是什么——**过滤出长度大于 32 的**，对，就是它。所以这一讲的评估代码不是从训练那份拉的，而是从上一个案例的评估那份拉的。

模型评估的套路跟刚才也差不多：加载模型，做评估，最后打印结果。打印出来你会看到预测值是什么、真实值是什么——那是我们额外做的活儿，是 `EP4`（第 4 个 epoch）里截出来的几条。`EP4` 有几条数据来着？八条，而我们只把第一个样本拿出来了。

### 第八步：模型测试的代码位置

把那份评估代码整个复制粘贴拉过来，然后把注释改成第八步：**模型测试（评估）**。

流程是：加载数据（这次不是训练集，是测试数据）、创建加载器、批量迭代……这一堆都一样。修改的地方我直接告诉你。

#### 三处修改

第一处，跟上面训练时一样，**过滤出长度大于 32 的样本**。这一处你必须做，不能像训练时那样注释掉。

第二处，**模型的路径**，因为这次的模型名跟上一个案例不一样。

第三处是最后一步，**预测结果的处理方式**，要改。

### 优化一：测试集的数据过滤

先改第一个。加载数据集这一坨都一样，只是这次加载的是 `test` 那份。

加一段优化一：过滤出长度大于 32 的样本，写法跟训练时同构：

```python
# 优化一：从原始文本中过滤出长度大于 32 的样本
test_dataset = test_dataset.filter(lambda x: len(x['text']) > 32)
```

#### 为什么评估时必须过滤，而训练时可以注释

你知道我为什么在这儿一定要给你做过滤吗？因为**在这一步，还没有做截断和补齐**。截断和补齐是下一个动作里干的事。

> **结论**：评估阶段的过滤作用在**原始文本**上，所以它真的会把短样本剔掉，不是恒真条件。

这一点跟训练阶段不一样：训练那份代码在前面已经走过截断补齐，长度已经统一成 32，过滤条件恒真，所以注释掉不影响结果。评估这份代码的过滤放在分词处理之前，条件是真的在起作用。

#### 数据加载器的参数

接着写后半截：`my_dataloader` 从 `test_dataset` 建出来，`batch_size` 给 8，打乱数据，`drop_last` 把最后一批不满的删掉，然后关键一点——整理函数要换成 **2**。

为什么是 2？因为上面那个处理函数的编号叫 2。你猜下个案例我的整理函数会变成几？自然就变成 3。整个整理的动作不太一样，其他都差不多。

### 优化二：修改模型路径

优化二就是**修改模型路径**：把任务名从 `classification` 换成 `fill_mask`。

```python
# 优化二：修改模型路径
model_path = './model/fill_mask.pt'
```

下面 `model`、`device` 那一堆都一样的写法。

### 评估循环与准确率统计

往后是创建准确数、总数、`eval` 这些计数器，再到把数据移到设备上（GPU），不计算梯度那一套也都一样。

再往下是除以 `output`、做前向传播、统计正确条数，这一坨其实也差不多。但这里稍微有点区别：`total` 加 1 之后要算总的，总的算完去算准确率。这些也都一样。

只有在测试的时候，你要稍微动动手脚。改哪儿？就把预测那几行先**注释掉**，注释掉以后第三步都不用做了——所谓第三步，就是后面解码那一段。

```text
评估循环的构成                         本讲是否改动
----------------------------------    ------------
计数器：准确数 / total / eval          不改
数据搬移：移到 device                  不改
`torch.no_grad()`：不计算梯度          不改
前向传播 output -> 取预测值            不改
统计正确条数、算准确率                 不改
打印预测标签 / 真实标签                改（本讲在这里多做一步解码）
```

读图：这个评估循环里除了最后打印那一段，其余部分都是从上一个案例原样搬过来的。真正的改动只有数据入口的三处——过滤、模型路径、打印解码。

那先跑一次看看结果。右键，打印。

> **提示**：跑之前如果嫌日志太吵，把中间打印 `label` 的那几行去掉或者注释掉。数据一多，一屏全是标签。

准确率出来了：最终 `ACC` 大概在 **70%** 左右。

其实到这儿已经可以让你下课了，但别急，我们得讲完。

### 优化三：把标签解码成文本

你可能会想：老师，你之前这一步不是不做吗，到这儿怎么非做不可？

因为前面那个案例里，你压根没做打印，没有这一步也能收工；这一讲我们要看结果、要数对不对，那就得做。刚才注释掉的那段代码重新贴回来，都不用动，就是那一堆。

#### 打印出来到底长什么样

`decoder` 那段不用动：拿 `input` 的第 0 条去解，解码之后打印三样东西——原始文本、预测的标签、真实的标签。

先不解码，你看看原样是什么样：

```text
原始文本: 这本书更多的带给人思考是……（截到 32 个字符就断了）
预测的标签: 5642    真实的标签: 5642
预测的标签: 833     真实的标签: 7231
```

原始文本为什么断在 32 个字符？因为我们只要 32 个。

至于后面两行——预测的标签是 5642，真实的标签是 5642，这组对上了；下一组预测 833、真实 7231。你说 833 是谁？7231 又是谁？你还得去**词表**里一个字一个字地对。这就是问题所在。

> **结论**：标签不是给人看的，它是词表里的 id。要看得懂，必须把它映射回文本。

所以最后一个动作，就是把标签解码成真实内容。

#### 解码的写法

```python
# 优化三：把预测标签与真实标签解码成可读文本
print(f'预测的标签是: {my_tokenizer.decode(pred_label)}')
print(f'真实的标签是: {my_tokenizer.decode(true_label)}')
```

`my_tokenizer.decode` 是个函数，把你要解码的标签扔进去就行——刚才填的 `0`、也就是那一条样本的预测标签，扔进去解析一下。

真实的标签一样处理：`my_tokenizer.decode(...)`，把上面的预测标签和真实标签都转成具体的文字。

> **提示**：这一步不加也行，直接看数字对不对得上就够了；加了更好看、更好核对。

加了之后，上面那行打印数字标签的代码就该注释掉了，不然两套输出叠在一起，反而看不清。我这儿留着也无妨。

#### 解码后的结果

解码完你就看懂了：

```text
预测: 是    真实: 是
预测: 会    真实: 错
预测: 的    真实: 的
预测: 情    真实: 美
```

第一句是个"是"字，预测也是"是"；第二句真实是个"会"字，模型给弄了个"错"。

再往下看，`4368` 解出来是个"的"字，对上了；另有一条真实是"美"字，模型给弄成了"情"字。

句子本身是"什么柔的东西十分细腻的显得十分什么怎么怎么样"这种，你得在这里边去数数，那个位置到底该是哪个字。其实有些时候，这个字跟那个字意思非常非常近。

### 准确率大概 70% 的原因

> **易错点**：我们只算两个字**完全一样**的情况，才算预测正确。

意思接近但字不同的，一律记错。所以这种情况算出来的准确率就会低一些，大概在 70% 左右。这个数字不是模型不行，是评价口径严。

也就是说，准确率是拿解码出来的**字**去逐个比对的：预测的字和真实的字完全相同才计一次正确，其他都算错。既然"美"和"情"、"会"和"错"这种近义或形近的替换都不算对，70% 这个水位就是正常的。

> **提示**：你要是想让数字好看，换个评价口径就行；但换口径不等于模型变强。先把"算对了多少"这件事本身算准，比追一个漂亮小数更重要。

### 两处修改的复用结论

把这一讲回头看一遍：评估代码我改了谁？第一个，做了一次过滤；第二个，改了一下模型路径。其他东西我连动都没动，只有预测结果那部分因为要解码而多做了一步。

这个案例讲到这儿就结束了，思路都一样，很多代码是重叠的。回头把这个案例自己再做一遍——记住，重复的越多，说明你手上能直接复用的东西越多。

> 💡 **承前启后**：完成对「Day12-07.迁移学习_中文填空案例_模型评估(掌握)」的理解后，下一章我们将深入探讨「Day12-08.迁移学习_NSP案例_自定义数据集对象(掌握)」，进一步完善知识图谱体系。

---

## Day12-08.迁移学习_NSP案例_自定义数据集对象(掌握)
> 对应分集：P180 | 原始标题：《Day12-08.迁移学习_NSP案例_自定义数据集对象(掌握)》

迁移学习最后一个案例：中文句子关系。要判断第二句话是不是第一句话的下半句。

比如我把一段文本拆成句子对，然后随便给你一个第二句话，你来帮我判断：这句话是不是第一句话的下半句。这个任务叫 **NSP**。

> **提示**：NSP 是面试里会被聊到的名词。将来面试官跟你聊到 NSP，你要是听都没听过，那就尴尬了。所以这个词要格外注意一下。

### 任务定义：下一句预测

先把任务本身说清楚。

> **定义**：下一句预测（Next Sentence Prediction，NSP），输入两句话，判断第二句是否是第一句的下半句。

拿诗举例最直观。"床前明月光"，下一句是"疑是地上霜"。现在我给你一句"举头望明月"，你来判断它是不是"床前明月光"的下半句——就是这个事。

这个任务同样用迁移学习来做。

#### 数据处理：正负样本的构造

数据处理环节需要你做一个动作：**构建两句话**，让第一句和第二句形成正负样本。

怎么造？最省事的做法是：从数据集里把一句话一截两半，左边是第一句，右边是第二句。但你要是死按着一半切，每条样本的切法就是固定的，所以还得**随机打乱一下**。就这么做。

#### 为什么是二分类

用预训练 BERT 提特征，最后接全连接加 Softmax 做二分类。

为什么是二分类？因为"第二句是不是第一句的下半句"，答案要么是、要么不是，只有两种可能。

一说到二分类，你应该马上想到：这个任务跟我们做的第一个还是第二个任务高度相似？

> **结论**：跟**中文文本分类**高度相似，因为那个也是二分类。所以两者的代码可以大量复用，我们从那份代码里拷贝，比从填空案例里拷贝更省事。

顺便说一句，为什么先把完形填空那个案例关掉？因为完形填空的输出是对着词表的每个字算概率，一共 **21128** 个维度；而这个 NSP 任务是二分类，跟第一个案例同构，从分类案例拷代码显然更简单。

### 复用与新增：这个案例的代码骨架

先导包、压制警告，然后把分类案例上面那一大堆直接拉过来。分词器和模型都现成的，用的一直是 `bert-base-chinese`。加载数据那部分也不用改。

理论上来讲，拉完这些上来就该写整理函数了。但不好意思——你现在要做的是中文句子关系分析，**你得有两句话才能做**，而这个动作是你手上所没有的。所以这里要自己写一段代码。

```text
从分类案例复用                     本讲新增
--------------------------        --------------------------
压缩警告 / 导包                    class MyDataset(Dataset)
加载分词器 tokenizer                -- 数据过滤（长度 > 44）
加载预训练模型 bert-base-chinese    -- 构建句子对（22 + 22）
加载数据                            -- 随机替换 sentence2 造负样本
数据预处理函数                      -- collate_fn3（下一节继续写）
```

读图：左边是从分类案例原样搬过来的部分，右边是这一讲必须自己动手补上的数据集对象。前两个案例的数据集都是 `load_dataset` 直接加载得到的，而这里的数据集得自己造。

### 为什么需要自定义 Dataset

前两个案例里，`dataset` 是怎么来的？都是通过 `load_dataset` 这个函数直接加载来的。

那这里的 `dataset` 为什么要自己做？因为我们要自己把一句话拆成两句话。`load_dataset` 只会把文件读成数据集，它不会替你拆分句子对。

#### 拆分的长度约束：为什么是 44

有人说，拆两句话简单，除以 2 不就完了？你说得很对，就是这么做的——直接除以 2，前面是上半句，后面是下半句。

但我问你一个问题：为了让每条样本切出来的句子长度一致，你能不能对每条句子都除以 2？这样一来，切出来的句子就会**有长有短**。

所以要先做一次筛选：筛选出长度大于 **44** 的句子，然后再一分为二。为什么门槛定在 44？

因为一会儿要截取前 44 个字符。长度本来就超过 44 的话，我截取前 44 个字符时，这 44 个字符**全都来自真实内容**，没有一个是被填充出来的。

具体切法是：长度按 44 算，**前 22 个字符作为句子一，后 22 个字符作为句子二**。

```python
sentence1 = text[:22]      # 前 22 个字符，第一句
sentence2 = text[22:44]    # 第 22 到 44 个字符，第二句
```

### 自定义 Dataset 的实现

现在开始写代码。这个案例是第三个例子，叫迁移学习，任务是 NSP。

案例名字：中文句子关系任务，也即下一句任务（Next Sentence Prediction）。注释里写清楚它的含义——输入两句话，判断第二句是否是第一句的下半句。

#### 导包与类的声明

先控制好空行，然后写这个类。注意：这是**自己写的**自定义代码，上面没有现成的可以抄，所以叫自定义数据集对象。

```python
class MyDataset(Dataset):
```

继承自 `Dataset`。有同学会问，这里怎么报错了？因为上面你还没有做导包。在 `DataLoader` 那行后面补一句，把 `Dataset` 导进来，这个类就不报错了。

#### 初始化：加载与过滤

下一步不用想，第一步肯定是初始化函数。

```python
def __init__(self, data_path):
    # 初始化父类成员
    super().__init__()

    # 加载数据集
    data = load_dataset('csv', data_files=data_path, split='train')

    # 对数据做过滤：只保留长度大于 44 的句子
    dataset_item = data.filter(lambda x: len(x['text']) > 44)

    # 记录过滤后的数据集
    self.dataset_item = dataset_item
```

几步逐条说：

- `super().__init__()`：初始化父类成员，这个动作每个继承来的类都要做。
- 参数只要一个 `data_path`，也就是你要用的数据路径。
- `load_dataset` 这里写 `'csv'`，第二个参数是数据文件路径。因为我们把路径留成了 `data_path` 这个参数，所以这里要把 `data_path` 扔进去；目录就用 `./data/` 下面那份文件。`split` 写 `'train'`。
- 过滤条件写 `len(x['text']) > 44`。为什么是 `x['text']`？因为文本内容那一列就叫 `text`。

过滤完之后，这一行我为什么敢拍胸脯？因为过滤后数据集里**每一条的长度都一定大于 44**，一定是够的，不需要做任何填充。

#### 长度方法

第二个操作是定义函数，获取数据集长度：

```python
def __len__(self):
    return len(self.dataset_item)
```

用 `__len__` 返回过滤后数据集的大小就行。

#### 取元素：先假设有关系

第三个动作是获取数据集的元素：

```python
def __getitem__(self, index):
    # 假设 label = 1，表示两个句子有关系
    label = 1

    # 根据索引获取句子
    text = self.dataset_item[index]['text']
```

这里 `index` 就是索引。

先看这一句"假设 `label` 等于 1，表示这两个句子有关系"是什么意思。我们要做的事是判断第二句是不是第一句的下半句，那"是"还是"不是"，总得返回一个结果给我。所以：

| label | 含义 |
| --- | --- |
| 1 | 两个句子有关系，`sentence1` 是上半句、`sentence2` 是下半句 |
| 0 | 两个句子没有关系 |

> **易错点**：这里的 0 和 1 表示的是**句子之间有没有承接关系**，不是好评差评那一套情感极性。别把上一个案例的标签含义带过来。

所以默认先按"有关系"处理，把 `label` 设为 1；如果一会儿代码处理完发现它俩其实没关系，就把 `label` 改成 0。

根据索引取句子时，`self.dataset_item` 传入 `index`（传数值，别加括号），后面跟上 `['text']` 取文本字段。

#### 构建句子对：默认正样本

第三步是构建句子对。默认是有答案的——`sentence1` 和 `sentence2` 默认有关系。

```python
    # 构建句子对，默认两条句子是有关系的
    sentence1 = text[:22]
    sentence2 = text[22:44]
```

`[22:44]` 你也可以写成 `[22:]`，从 22 开始到 44 结束，切出来也是 22 条字符。

到这里，所有样本都是正样本——因为 `sentence2` 永远是 `sentence1` 的下半句。这样训练出来的模型只会说"是"，那不行。

#### 构建负样本：随机替换 sentence2

第四个操作：构建负样本。

```python
    # 随机替换 sentence2，构造负样本
    if random.randint(0, 1) == 0:
        j = random.randint(0, len(self.dataset_item) - 1)
        sentence2 = self.dataset_item[j]['text'][22:44]
        label = 0
```

这里有几个点要交代清楚。

第一，`random` 导包了没有？导一下。用 `import random` 还是 `from ... import ...`？用 `import random` 的话，后面就能直接写 `random.randint`，用起来更顺。

第二，`random.randint` 生成的随机数**包左包右**，所以 0 和 1 都有可能被生成出来。当生成的是 0，我们就进这个分支。

第三，进分支以后干什么？随机拿一个句子来替换 `sentence2`。怎么随机拿？

```python
j = random.randint(0, len(self.dataset_item) - 1)
```

这里必须**减 1**。为什么要减 1？因为 `randint` 两端都包，不减 1 就有可能取到 `len`，索引越界就取不到句子了——减 1 才能确保这个索引指向的句子确实存在。

拿到这个随机数 `j` 以后：

```python
sentence2 = self.dataset_item[j]['text'][22:44]
```

这行的意思是：`self.dataset_item[j]` 取出第 `j` 条样本，再取它的 `text`，然后截 `22:44`，把它当成新的 `sentence2`。也就是从这里取出随机的某条子样本，把它的下半句拿来替换位置。

> **结论**：只要走进了这个分支，就说明这里构造的是负样本，`label` 应该改成 0。

#### 返回值

最后把样本数据返回：

```python
    return sentence1, sentence2, label
```

也就是句子一、句子二，再加上标签。整个元素获取的写法就是：先假设它俩有关系，再随机判断要不要撤掉这个假设；要撤，就把 `sentence2` 换掉、把 `label` 置 0。

### 测试数据集对象

数据集这一坨写完，是不是得调用一下才知道对不对？第四步：测试。

```python
my_dataset = MyDataset(data_path)
print(f'数据集长度: {len(my_dataset)}')

sentence1, sentence2, label = my_dataset[2]
print(f'sentence1: {sentence1}')
print(f'sentence2: {sentence2}')
print(f'label: {label}')
```

- 第一件事：把 `MyDataset` 实例化，把数据路径扔进去。
- 第二件事：`len(dataset)` 打印数据集长度。
- 第三件事：取**第三个样本**——也就是 `my_dataset[2]`，因为索引是从 0 开始的——把 `sentence1`、`sentence2`、`label` 三个值一起接出来打印。

#### 跑出来的结果

长度打印出来是 **7472**。

```text
sentence1: 接电员没几分钟，店员送配器，热得不行
sentence2: 拉底尔本来是想去最好的酒店，结果一肚子火
label: 0
```

`label` 是几？是 **0**。这个 0 是"无关"，不是好评差评——说明这两句话没有关系。

再随便抓一条看看。注意：因为批次在这里是随机生成的，所以抓到 1 的概率会比 0 小一些，因为负样本的替换是随机打乱的。

```text
sentence1: 接电员没几分钟，送配器热得不行，淋浴房缺少一个放……
sentence2: 服务……
label: 0
```

又来一个 0，还是无关。为什么它俩无关？因为 `sentence2` 在这里是被随机替换、打乱过的。等我一会儿专门去找出和它对应的那条数据，相关的样本自然就会多起来。

### 数据集对象之上的加载器：整理函数

测试数据集对象这一步做完了。接下来你猜该干嘛？数据集对象只是把数据封装起来了，上面用的还是 `user_dataset` 这套包装。那现在要做的事就是获取它的**数据加载器（DataLoader）**。

加载器上面还需要一个东西：**自定义整理函数（collate_fn）**。这里有个明显的空缺——数据加载器前面少一个 `collate_fn3`，你得先把那个整理函数写出来才能往下走。

所以下一段要写的就是它俩：`collate_fn3` 和对应的加载器构造。这一节的第五步是数据整理函数，负责处理批次数据；到这儿，NSP 案例这边自定义 `Dataset` 的构建与测试就已经完成了。

> 💡 **承前启后**：完成对「Day12-08.迁移学习_NSP案例_自定义数据集对象(掌握)」的理解后，下一章我们将深入探讨「Day12-09.迁移学习_NSP案例_数据预处理(掌握)」，进一步完善知识图谱体系。

---

## Day12-09.迁移学习_NSP案例_数据预处理(掌握)
> 对应分集：P181 | 原始标题：《Day12-09.迁移学习_NSP案例_数据预处理(掌握)》

句子关系已经拆出来了，下一步手上缺的东西就一个：**数据加载器**（DataLoader）。没有它，数据没法按批送到模型面前。而要把加载器装起来，前面还欠一个零件——第四步的**自定义数据集对象**，第五步的**数据整理函数**，第六步才是加载器本身。

所以这一节干的是四、五、六这三件事：把自定义的 `MyDataset` 接进来，把整理函数 `collate_fn3` 写出来，最后把 `DataLoader` 装好跑一遍。

### 从分类案例搬来的四、五、六

这一步最省事的做法，是把中文文本分类案例里的四、五、六三块代码整段拉过来。拉过来之后要改的只有一处：整理函数原先叫 `fn1`，这次换成本案例自己的 `fn3`，所以加载器里 `collate_fn` 这个参数要跟着改，下面调用加载器的地方也要跟着改。

改完以后，框架里就多了一个原来没有的东西：

```text
   1 导包 / 压缩警告          ← 三个案例通用，直接复用
   2 加载分词器 tokenizer      ← 三个案例通用
   3 加载预训练模型            ← 三个案例通用
   4 加载数据                 ← 本案例换成自定义 MyDataset
   5 写整理函数 collate_fn3    ← 本案例自己写（本节重点）
   6 创建 DataLoader          ← 复用分类案例，只改 collate_fn 指向
```

读图说明：第 4 步在分类案例里是 `load_dataset` 直接加载，本案例换成了自定义的数据集对象；第 5 步是全新内容，必须自己动手；第 6 步结构照搬，唯一要动的是整理函数的引用。

### 整理函数分六步：先看清单

展开 `collate_fn3`，里面要做的事情一共六步。先把清单摆出来，后面逐个落实：

```text
collate_fn3(data)  ── data 是这一批的 8 条样本
   │
   ├── 1 提取文本      sentences = [item[:2] for item in data]   → [(s1, s2), ...]
   ├── 2 获取标签      labels    = [item[2]  for item in data]   → [0/1, ...]
   ├── 3 批量编码      my_tokenizer.batch_encode_plus(...)
   ├── 4 提取编码结果  input_ids / token_type_ids / attention_mask
   ├── 5 标签转张量    torch.longTensor(labels)
   └── 6 返回结果      (input_ids, token_type_ids, attention_mask, labels)
```

读图说明：前三步是“把这一批原始样本拆成文本和标签”，中间两步是“把文本变成模型能吃的张量”，最后一步把四个结果交出去。加载器一批一批调用它，所以函数内部只处理一批。

> **定义**：整理函数（`collate_fn`）是 `DataLoader` 的一个参数，作用是把 `Dataset.__getitem__` 取出来的一批样本，整理成真正送进模型的那个 batch。它拿到的 `data` 就是这一批的 8 条，不是整个数据集。

### 提取文本：为什么是 `item[:2]`

第一步是提取文本。先别急着写，先问一句：这一批数据传进来到底长什么样？

它长这样——外层是列表，里面每一个元素是一个三元组：

```text
data = [
        (sentence1, sentence2, label),   ← 第 1 条
        (sentence1, sentence2, label),   ← 第 2 条
        ...
        (sentence1, sentence2, label),   ← 第 8 条
       ]
```

读图说明：这是数据集对象 `__getitem__` 返回值的堆叠结果。上一节里返回值写的就是 `return sentence1, sentence2, label`，一批 8 条摞起来就是上面这个形状。

那么“只要前两个、丢掉第三个”怎么写？列表推导式遍历 `data`，每一项切片取前两位：

```python
sentences = [item[:2] for item in data]
```

`item[:2]` 的意思就是：从这一条样本里，只要前两个——`sentence1` 和 `sentence2`。第三个位置上的 `label` 被切掉了。为什么先切掉它？因为标签是单独一回事，第二步专门处理，混在一起后面没法直接喂给分词器。

> **易错点**：`item[:2]` 切出来的仍然是元组 `(sentence1, sentence2)`，不是两句拼成的一句话。后面 `batch_encode_plus` 收到的是一个元组列表，每一条样本自己是一对句子——这正是句子对任务需要的输入形态，不要手工拼成字符串再传进去。

### 获取标签：为什么是 `item[2]`

第二步是取标签，格式上和上一步对称：要的是一个纯标签的列表 `[label1, label2, ...]`。同样遍历 `data`，这次只要下标 2 那一位：

```python
labels = [item[2] for item in data]
```

到这里，一批数据的“原料”就分完了：一句一对句子，一份一串标签。

> **提示**：写成 `item[:2]` 和 `item[2]` 之所以成立，是因为我们在 `__getitem__` 里把返回顺序定死成了 `sentence1, sentence2, label`。如果哪天改了返回顺序，整理函数里这两个下标必须同步改——这种靠位置约定的代码，改一处忘一处是最典型的翻车方式。

### 批量编码与最大长度 50

第三步把文本交给分词器批量编码。写法与分类案例基本一致，就是把要编码的对象换成上面抽出来的 `sentences`：

```python
inputs = my_tokenizer.batch_encode_plus(
    sentences,
    truncation=True,
    max_length=50,
    padding='max_length',
    return_tensors='pt',
    return_length=True,
)
```

参数里真正要动脑筋的只有 `max_length`。剩下的：

- `truncation=True`：超过长度的部分截掉；
- `padding='max_length'`：不够长度的补齐；
- `return_tensors='pt'`：直接返回 PyTorch 张量，省得自己再转；
- `return_length=True`：附带返回编码后的长度。它是可选项，这里用不上，删掉也不影响运行——所以这里就把它删了。

现在说 `max_length` 到底写多少。分类案例里写的是 300，那是给长短不一的整句评论留的余量。本案例不一样，我手上的文本是被**截过**的，长度分布很集中，所以这个数要重算，不能顺手抄 300。

先算下限。上一节做数据过滤时，条件写的是长度大于 44，然后前 22 个字当上半句、后 22 个字放下半句。所以进到这里的每一对句子，内容长度是 22 + 22 = 44 个字符，一个字都不缺。

那是不是写 44 就够？不够。句子对送进 BERT 时是要拼接的，拼接格式是这样的：

```text
[CLS]  sentence1  [SEP]  sentence2  [SEP]
  ↑       22         ↑       22        ↑
 1 个     个         1 个     个        1 个
```

读图说明：格式固定为 `[CLS] + 句子一 + [SEP] + 句子二 + [SEP]`，句首一个分类标记、句尾一个分隔标记，两句之间再用一个分隔标记隔开，一共多出三个特殊符号。

44 加上这 3 个特殊符号，就是 **47**。所以 47 是这条链路上真正不能低于的下限。

> **结论**：`max_length` 必须大于 47。写 48、49、50 都行，我习惯写 50——凑个整，看着顺眼，而且留了一点余量保险起见。

为什么说写大一点无所谓？因为 `padding='max_length'` 会把不够的部分补齐。写 50 的时候，假如某条实际只有 47 个 token，多出来的 3 个位置用填充补齐，模型靠 `attention_mask` 把那 3 个位置忽略掉就行，不影响结果。真正要避免的只有一件事：**写得比 47 小**，那样一个字都还没拼完就被 `truncation=True` 截断了，模型看到的是残句。

> **易错点**：这里 50 和分类案例那个 300 不可互换。300 是按“整句评论的长度分布”定的，本案例是按“44 个内容字符 + 3 个特殊符号”推出来的。长度上限定得过大，代价是每个批次的张量都要按这个宽度补齐，计算量白涨——这一节末尾你会发现训练速度的差别相当明显。

### 提取编码结果与标签转张量

第四步是把编码结果里模型要的三样东西抠出来，写法和分类案例完全一样：

```python
input_ids = inputs['input_ids']
token_type_ids = inputs['token_type_ids']
attention_mask = inputs['attention_mask']
```

三者形状相同，都是 `[batch_size, max_length]`。`token_type_ids` 在本案例里终于不再全是 0 了：

| 值 | 含义 |
| --- | --- |
| `0` | 属于句子一（含句首的 `[CLS]`） |
| `1` | 属于句子二（含句中与句尾的 `[SEP]`） |

这个“哪一段属于哪一句”的信息，就是句子对任务比单句任务多出来的一路输入。

> **定义**：`token_type_ids` 又叫句子段标记（segment ids），用来告诉模型序列里的 token 分属第一句还是第二句。单句分类任务里它整片为 0，句子对任务里才真正发挥作用。

第五步把标签从 Python 列表转成长整型张量——分类任务的标签必须是 `long`：

```python
labels = torch.longTensor(labels)
```

第六步返回，把四样东西按固定顺序交出去：

```python
return input_ids, token_type_ids, attention_mask, labels
```

整理函数的完整样子：

```python
def collate_fn3(data):
    # 1. 提取文本：只要每条样本的前两位（sentence1, sentence2）
    sentences = [item[:2] for item in data]

    # 2. 获取标签
    labels = [item[2] for item in data]

    # 3. 批量编码：44 个内容字符 + 3 个特殊符号 = 47，上限取 50
    inputs = my_tokenizer.batch_encode_plus(
        sentences,
        truncation=True,
        max_length=50,
        padding='max_length',
        return_tensors='pt',
    )

    # 4. 提取编码结果
    input_ids = inputs['input_ids']
    token_type_ids = inputs['token_type_ids']
    attention_mask = inputs['attention_mask']

    # 5. 标签转张量
    labels = torch.longTensor(labels)

    # 6. 返回结果
    return input_ids, token_type_ids, attention_mask, labels
```

### 加载器装配：不要用 `load_dataset`

整理函数写完了，接下来是第六步，把加载器装起来。这一步是本节最容易写废的地方，因为上面那段代码一拉过来，里面写的还是 `load_dataset`。

```python
# 错误写法：这里用的是人家自带的 dataset，你的 MyDataset 就白写了
train_dataset = load_dataset(path='./data/train.csv', split='train', max_length=50)
```

**这里坚决不能用 `load_dataset`。** 理由很直白：`load_dataset` 拿回来的是 Hugging Face 自带的数据集对象，它给不出“一句话拆成两句话”这种结构。你前两节辛辛苦苦写的 `MyDataset`，如果这里还挂 `load_dataset`，那就等于写完就扔，一点作用都没起。

正确写法是把数据集换成自己的那个类，路径还是 `./data/train.csv`：

```python
my_dataset = MyDataset(data_path='./data/train.csv')

my_dataloader = DataLoader(
    dataset=my_dataset,        # 换成自定义数据集对象
    batch_size=8,
    shuffle=True,
    drop_last=True,
    collate_fn=collate_fn3,    # 换成这个案例自己的整理函数
)
return my_dataloader
```

两次替换各自的意义：`dataset` 从“自带数据集”换成“自定义数据集对象”，这是让句子对被真正造出来的关键；`collate_fn` 从 `fn1` 换成 `fn3`，这是让这一批数据按句子对格式编码的关键。其余参数原样不动——`batch_size=8` 是每批 8 条，`shuffle=True` 训练时打乱，`drop_last=True` 丢掉最后一个不齐的批次。

> **提示**：`collate_fn` 的引用和整理函数的签名是一体两面。函数改成了 `collate_fn3`，加载器这里就必须同步，否则要么报名字错误，要么悄悄用了上一案例的整理函数——那种情况下数据也能出来，只是格式全错，最难查。

### 测试加载器：为什么第一次跑没有打印

加载器创建完，照例测一遍。先调用函数拿到对象，再从里面取一批看看：

```python
my_dataloader = get_data_loader()

for batch in my_dataloader:
    print(batch)
    break
```

跑起来的现象有点扫兴：**程序不报错，但什么也没打印**。

为什么？倒着捋一遍就清楚了。外层循环从加载器里取一批，加载器把这一批交给 `collate_fn3`，而 `collate_fn3` 里从头到尾没有任何输出语句——它只负责算，不负责说。所以数据是处理了，只是没吭声。

想让结果可见，就在整理函数里自己加打印。比如打印 `inputs` 看编码后的张量，打印 `sentences` 看处理前的两句话，再打印取出来的三个字段看处理后长什么样。要是不想看，这一段完全可以跳过——前面两个案例已经把这个格式看过一遍了，这里是同一个东西。

到这里数据加载器就算装配完成。下一个动作不用猜：数据齐了，该轮到模型了——自定义一个基于 BERT 的句子关系模型。再往后就是老规矩，第八步训练、第九步评估。

> 💡 **承前启后**：完成对「Day12-09.迁移学习_NSP案例_数据预处理(掌握)」的理解后，下一章我们将深入探讨「Day12-10.迁移学习_NSP案例_模型搭建(掌握)」，进一步完善知识图谱体系。

---

## Day12-10.迁移学习_NSP案例_模型搭建(掌握)
> 对应分集：P182 | 原始标题：《Day12-10.迁移学习_NSP案例_模型搭建(掌握)》

模型搭建这一节的结论可以一句话说完：**跟第一个案例一模一样，代码拉过来一个字都不用改。**

这话说出来你可能不信——第三个案例，句子关系，怎么可能跟第一个文本分类的模型完全一样？还真就是一样。原因在上一节定任务的时候就埋下了：文本分类做的是二分类，NSP 判断“第二句是不是第一句的下半句”也是二分类，答案非是即否，模型要出的就是两个分数。既然输出维度一样、输入维度一样、中间用同一个 BERT，那这个模型没有任何需要动的地方。

所以这一节真正值得花时间的，其实是后面那个问题：负样本到底是怎么随机构造出来的。

### 从案例一直接搬过来的八号

上一节的第六步停在了数据加载器上，接着往下走就是模型。做法很干脆：把案例一里的模型搭建那两段（六和七）整段复制过来，把序号从七改成八，**别的一个字都不动**。

搬过来之后立刻做一件事：跑一遍看模型结构。测试函数叫 `use_linear_model`，调用起来，输出正常，说明模型通了。

整个模型的结构是这样的：

```text
                   输入：input_ids / token_type_ids / attention_mask
                                      │  [8, 50]
                                      ▼
                    +----------------------------------+
                    |   BERT 预训练模型（参数冻结）      |
                    |   bert-base-chinese               |
                    +----------------------------------+
                                      │
                                      ▼  pooler_output  [8, 768]
                    +----------------------------------+
                    |   nn.Linear(768, 2)              |
                    +----------------------------------+
                                      │
                                      ▼  logits  [8, 2]
                                      │
                                      ▼  Softmax
                          有关系 / 没关系 的概率
```

读图说明：上游的 BERT 把一句话对压成一个 768 维向量，下游只有一层 `Linear`，把这 768 维映射成 2 维，最后经 Softmax 得到两类概率。上游负责理解，下游负责判决，分界线就是中间那一层线性变换。

> **结论**：自己做迁移学习时，**自己写的往往只有这一层全连接**。BERT 那么大一坨参数是预训练好搬过来的，我们要练的就是把它那 768 维表示翻译成任务答案的这一步。

上一节把 `max_length` 从 300 压到 50，这里就能看出回报了：同样一句话对，分类案例要按 300 的宽度补齐，本案例只按 50 补齐，BERT 内部注意力的计算量随序列长度是平方级增长的，两头一对比，差距相当可观。

#### 只有一层全连接的两行代码

模型类的完整样子就这么短：

```python
class MyModel(nn.Module):
    def __init__(self):
        super().__init__()
        # 768 维 → 2 维：NSP 是二分类
        self.linear = nn.Linear(768, 2)

    def forward(self, input_ids, token_type_ids, attention_mask):
        # 取 BERT 输出：只包 no_grad，不更新预训练参数
        with torch.no_grad():
            bert_output = my_pre_model(
                input_ids=input_ids,
                token_type_ids=token_type_ids,
                attention_mask=attention_mask,
            )
        # 768 → 2，得到两个类别的分数
        output = self.linear(bert_output.pooler_output)
        return output
```

三处关键点：

- `nn.Linear(768, 2)` 里的 **768** 是 BERT 的输出词向量维度，`bert-base-chinese` 固定是这个数；**2** 是因为本案例是二分类。换成三分类写 3、四分类写 4，这一层之外不用动。
- `with torch.no_grad():` 只包住**取 BERT 输出**这一段。`no_grad` 的作用是不记录梯度，这样预训练模型的参数就不会被更新；把它包到外面去连 `self.linear` 一起裹住，训练时反向传播就找不到梯度了。
- `bert_output.pooler_output` 的取出是必须的。BERT 的输出对象里不止一个字段，句子级任务要的是这个池化后的结果，形状 `[batch_size, 768]`，一个样本一行；另一个常见的 `last_hidden_state` 是 `[batch_size, seq_len, 768]`，三个维度，直接丢给 `nn.Linear(768, 2)` 会因为维度对不上报错。

> **易错点**：`no_grad` 包多了不是“稍微影响效果”，而是直接把训练冻死——参数没有梯度，`optimizer.step()` 无从更新，损失会一条直线不下降。这个坑前面踩过一次，这里不再踩。

### 负样本为什么必须随机构造

模型这一节的后半段，同学问了一个很实在的问题：做数据处理的时候，那个 `i` 到底在干什么？为什么要在取样本的时候判一个随机数？先把它讲透。

先看数据本来的样子。上一节的过滤保证了每条文本长度都大于 44，切片之后每个句子对都是这个形状：

```text
样本一   [ 上半句 22 字 ][ 下半句 22 字 ]    长度 44
样本二   [ 上半句 22 字 ][ 下半句 22 字 ]    长度 44
样本三   [ 上半句 22 字 ][ 下半句 22 字 ]    长度 44
```

读图说明：每条样本前半段是上半句、后半段是下半句，两者本来就是同一段文本里前后相接的两截。

如果老老实实把每条样本切成两半当成一对句子送进去，会得到什么？每一对的 `sentence2` 都是 `sentence1` 名副其实的下半句。也就是说，**标签永远是有关系**。拿这样的数据去训练，模型学到的唯一规律就是“输出有关系”，别的什么都没学会——因为数据里根本不存在反例。

更要命的是，连评估都失去意义：拿这种样本去测，模型答“有关系”永远对，准确率漂亮得毫无价值。

所以必须人为往数据里掺进“没关系”的句子对。这就是**负样本**的来路。

> **定义**：正样本指“第二句确实是第一句的下半句”的句子对，标签为 1；负样本指“第二句与第一句没有承接关系”的句子对，标签为 0。NSP 的二分类能力完全建立在正负样本混合出现的基础上。

构造手法很直接：随机从数据集里抓另**一条**样本，把它的下半句拿过来，替换掉当前样本的 `sentence2`。位置不变、长度不变，只是内容换成了一句跟上半句毫不相干的话。

```text
样本一   上半句A(22字) │ 下半句A(22字)   ← 原来的正样本，label = 1
样本七   上半句G(22字) │ 下半句G(22字)
                              ↑
                              └── 把这一段拽过来

构造结果 上半句A(22字) │ 下半句G(22字)   ← 负样本，label = 0
```

读图说明：从别的样本上取走它的下半句，覆盖到当前样本的第二句位置。上半句没变、长度没变，但“下半句”这个身份没了，这句话对就成了反例。

具体到代码上就是判一个随机数：

```python
# 假设有关系
label = 1

# 随机判断是否构造负样本
if random.randint(0, 1) == 0:
    j = random.randint(0, len(self.dataset_item) - 1)
    sentence2 = self.dataset_item[j]['text'][22:44]
    label = 0
```

`random.randint(0, 1)` 两端都包，生成 0 或 1 各一半概率。走到 0 这一支，就去随机抓另一条样本、取它的 `22:44` 那一段当新的 `sentence2`，同时把 `label` 从 1 改成 0。抓取索引那里必须减 1：`randint` 包右端，不减 1 有可能取到 `len`，索引越界就抓不到数据。

> **提示**：严格说，随机构造出的“负样本”偶尔也可能碰巧语义相关——从同一个数据集里随机抓的第二句，跟第一句未必真的毫无关联。这是这种造法的固有噪声，属于 NSP 任务的常规做法；数据量越大、句子来源越杂，这种巧合的概率越低。

模型和数据到这里就齐了。再往下，第八步训练、第九步评估，全是把案例一的代码搬过来改个名字的事。

> 💡 **承前启后**：完成对「Day12-10.迁移学习_NSP案例_模型搭建(掌握)」的理解后，下一章我们将深入探讨「Day12-11.迁移学习_NSP案例_模型训练和评估(掌握)」，进一步完善知识图谱体系。

---

## Day12-11.迁移学习_NSP案例_模型训练和评估(掌握)
> 对应分集：P183 | 原始标题：《Day12-11.迁移学习_NSP案例_模型训练和评估(掌握)》

模型搭好了，接下来是训练。

训练这一段有个省事的结论要先说：把案例一里的训练代码整段拉过来，**整个 `train` 里只改一个地方**——模型保存那行的任务名。数据一样、模型一样、参数一样、损失函数一样、优化器一样、三剑客一样，全都不用动。案例一保存的是 `classification`，本案例改成 `nsp` 就完事。

评估也几乎一模一样，只不过评估环节要多改两个地方，而且其中有一个坑，改漏了程序不报错、结果却是错的。这一节把训练跑起来、把评估跑对、再把这一个 batch 解码出来用眼睛核一遍，最后把三个案例的复用关系收个尾。

### 训练：只改模型保存名

训练函数原样搬过来，序号改成 9。逐个交代改动：

- **数据、模型、参数**：不动。文本分类和 NSP 都是二分类，损失函数、优化器、学习率、轮数全都通用。
- **保存路径**：从 `./model/classification{轮数}.pth` 改成 `./model/nsp{轮数}.pth`。

```python
torch.save(
    my_model.state_dict(),
    f'./model/nsp{epoch_index + 1}.pth',
)
```

这行改动的意义不只是换个文件名，而是**让三个案例的产物不互相覆盖**。整个 NLP 阶段我们已经存了两批模型：第一批 `classification1/2/3.pth` 是中文文本分类的，第二批 `fill_mask1/2/3.pth` 是中文填空的。现在这一批用 `nsp` 前缀，跑完就是 `nsp1.pth`、`nsp2.pth`、`nsp3.pth`，三套模型各自独立，评估的时候按任务名去取，不会串。

> **注意**：`torch.save` 这一行的缩进位置决定保存频率。写在 epoch 循环内部是“一轮存一个”，挪到循环外面就变成“全部轮次跑完只存一个”。本案例要的是三个模型文件，所以它必须待在循环里。

#### 跑出来的训练日志

右键执行，训练启动。这一轮的观感跟第一次做文本分类完全是两码事：**快得多**。

原因上一节已经埋好了。分类案例的 `max_length` 是 300，本案例是 50。BERT 内部注意力的计算量随序列长度是平方级增长的，序列长度缩到六分之一，计算量掉的不是一点半点。一轮训练下来大概 **26 秒**左右，三轮转眼就完了。

日志里每 20 批打一次汇总，最后一段的准确率大约在 **88.1%** 上下——记住这个口径：它是**这 20 批数据的汇总**，不是整个测试集的准确率，也不是单独一轮的平均。真正有意义的分数要看后边的评估。

训练跑完去看 `./model/` 目录，如果文件列表里没刷新出来，在项目上右键执行一次 **Reload from Disk**（从磁盘重新加载），`nsp1.pth`、`nsp2.pth`、`nsp3.pth` 就都在了。这不是文件没生成，只是文件视图没刷新。

### 评估：两个必改加一个隐藏的坑

评估代码从案例一拉过来，序号改成 10。改哪里？

第一个必改是**数据集**：这里不能再用训练集，要换成测试集，也就是 `./data/test.csv`。

第二个必改是**模型路径**：从 `./model/classification3.pth` 换成 `./model/nsp3.pth`。选 3，因为那是最后一轮训练留下的权重。

改完这两处还不够。往评估函数上面看，创建数据加载器的那段代码里，加载数据用的是 `load_dataset`，它还带着 `path`、`split`、`max_length` 那一套参数。这就是那个坑。

回到本案例的数据集对象上看一眼签名就明白了：`MyDataset` 的 `__init__` 只收一个参数，就是 `data_path`。构造函数不一样，参数列表自然不一样——你没法拿“路径 + split + max_length”去喂它。

```python
# 改之前：用的是自带数据集，句子对根本没被构造出来
test_dataset = load_dataset(path='./data/test.csv', split='train', max_length=50)

# 改之后：换成自定义数据集对象，只传一个数据路径
test_dataset = MyDataset(data_path='./data/test.csv')
```

> **易错点**：这一处漏改，程序**不会报错**。`load_dataset` 照样能加载出数据，加载器照样能取批次，循环照样跑完，准确率照样打印。但走的是自带数据集，句子对没被拆过、`token_type_ids` 全是 0，模型测的根本不是 NSP。这种“能跑但算错”的错误最难发现，因为它不给你任何提示。

换完数据集，下面那坨参数全部删掉，只留一个 `data_path`。再往下就和案例一一样了：加载器、加载权重、初始化 `correct` 与 `total`、`my_model.eval()`、逐批预测统计。最后调用评估函数跑起来。

### 看结果：把标签和预测摆到一起

评估跑完，屏幕上出来的是准确率、解码后的原始文本、预测标签、真实标签。**不要只看那个准确率**，把文本和两个标签并排读一遍，你才知道模型到底学到了什么。

```text
原始文本: [CLS] 每每阅读此书，我都怀着一颗…… [SEP] 以至于我长时间沉浸…… [SEP]
预测标签: 1     真实标签: 1      ← 对：确实是下半句
预测标签: 1     真实标签: 0      ← 错：把无关的两句话判成了承接
预测标签: 0     真实标签: 0      ← 对
预测标签: 1     真实标签: 1      ← 对
```

一条条读过去：

- “每每阅读此书，我都怀着一颗……的心情”接“以至于我长时间沉浸……”。注意这里的文本是被截到 44 个字符的，第二句结尾其实是断在半个词上（本意大概是“久久不能平静”），但语感上它接得上，真实标签 1，预测 1，对了。
- “酒店设备不错，也很干净，但离地铁站还有一段距……”接的是后半句“打扫呼叫了两个小时也没人来，房间下水道臭气……”。前一句在说酒店的位置，后一句在投诉服务和气味，两截不是一回事，真实标签 0；模型却判成了 1，错了。
- 再往下两条，一条是“书里面的文字都是现世最流行最有……”接“画都干净，除了面积小”，这显然接不上，真实 0、预测 0，对了；另一条是“给孩子朗读的好处大家都知道，关键是如何读，读……”接“读什么，书中取到的例子是国外的”，前后咬得上，真实 1、预测 1，对了。

> **结论**：准确率是个统计量，它告诉你平均水平；单条样本的文本加标签对照，告诉你模型在哪儿对、在哪儿错。两件事都要做，缺一样你都不算真正评估过这个模型。

### `decode` 两个参数的实验

评估里打印原始文本用的是分词器的解码方法，当时那两个参数写的都是 `True`：

```python
text_list = my_tokenizer.decode(
    input_ids[0],
    skip_special_tokens=True,
    clean_up_tokenization_spaces=True,
)
```

`input_ids[0]` 取的是这一批的第一条样本。两个参数分别管一件事，而且都用实验试过效果。

#### `skip_special_tokens`：是否忽略特殊符号

先按字面拆这个词：`skip` 跳过，`special` 特殊，`token` 记号——跳过特殊记号。特殊记号指哪些？就是 `[CLS]` 和 `[SEP]` 这类，它们是编码时补进去的、不是原文里的字。

把它改成 `False` 再跑一遍，答案立刻浮出来：

```text
[CLS] 每每阅读此书，我都怀着一颗……的心情 [SEP] 以至于我长时间沉浸在思考回顾中 [SEP] [PAD] [PAD] [PAD] ... [PAD]
```

`[CLS]` 出现在开头，两个 `[SEP]` 分别出现在两句之间和句尾，后面挂着一长串 `[PAD]`——那就是 `max_length=50` 补齐出来的填充位。

> **提示**：`[PAD]` 之所以显示这么长一串，是因为这条样本编码后的真实长度没到 50，`padding='max_length'` 把它补到了 50。填充位的内容对模型没有意义，模型是靠 `attention_mask` 里的 0 把它们忽略掉的，纯粹为了张量宽度整齐才存在。

所以这个参数写 `True` 的作用是：解码时把特殊符号过滤掉，保证你看到的“原始文本”里只有原文的字，干干净净。写 `False` 就全都露出来，做调试、想确认拼接格式对不对的时候很好用。

#### `clean_up_tokenization_spaces`：是否清理分词空格

这个参数直译是“清理分词化空格”，它处理的是分词过程中被塞进去的那些空格。把它改成 `False` 再跑，效果**不太明显**——字与字之间看起来还是有空格。

为什么效果不明显？因为那些空格本来就是分词器在解码拼字时自己加的分隔动作，改成 `False` 只是不去做额外的清理，原本的分隔还在。所以这个参数在这个场景下差异很小。

> **结论**：两个参数都写 `True` 就行。第一个 `True` 负责把 `[CLS]`、`[SEP]`、`[PAD]` 这些噪声挡在输出之外，效果立竿见影；第二个 `True` 保持默认的清理行为，不动它最稳。

### 三个案例的流程复盘

NSP 到这里就结束了。把三个案例并排摆一次，你会看到一个很清楚的规律：

| 环节 | 中文文本分类 | 中文填空 | 中文句子关系 |
| --- | --- | --- | --- |
| 导包 / 压缩警告 | 相同 | 相同 | 相同 |
| 加载分词器 | 相同 | 相同 | 相同 |
| 加载预训练模型 | 相同 | 相同 | 相同 |
| 数据处理 | 相同 | 相同 | **自定义 `MyDataset`** |
| 获取数据加载器 | 相同 | 相同 | 换 `collate_fn3` |
| 创建模型 | `Linear(768, 2)` | 输出为词表大小 | `Linear(768, 2)` |
| 训练 | 相同 | 相同 | 只改保存名 |
| 评估 | 相同 | 换模型路径 | 换数据集与模型路径 |

读表说明：真正有差异的只有数据处理和少数的名字替换。导包、分词器、预训练模型这三块，三个案例一字不差；训练和评估的骨架，三个案例也几乎一字不差。

#### 数据处理的差异在哪

数据动作一样的时候，代码整段复制就行；**数据有额外要求的时候，才需要自己写一个数据集对象**。本案例的额外要求就是“把一句话拆成两句”，所以必须自定义。

中文句子关系的处理链条从头到尾是这样：

```text
整句话（长度 44 字以上）
   │  过滤：只保留 len(text) > 44 的句子
   ▼
一条长文本
   │  前 22 字 → sentence1      后 22 字 → sentence2
   ▼
正样本句子对（label = 1）
   │  以 1/2 的概率随机抓另一条样本的下半句，覆盖 sentence2
   ▼
负样本句子对（label = 0）
   │  batch_encode_plus：拼接 [CLS] s1 [SEP] s2 [SEP]，补齐到 50
   ▼
input_ids / token_type_ids / attention_mask / labels
   │  BERT（参数冻结）→ pooler_output [8, 768]
   ▼
Linear(768, 2) → Softmax
   ▼
有关系 / 没关系
```

读图说明：从原始长文本到最终的两类概率，中间只有两处是“我们自己写的东西”——负样本的随机替换，以及最后那一层 `Linear`；其余全是搬过来的成熟部件。

把这张图分成两段看，就是整条链路的骨架：

```text
                        输入：文本 + 标签
                                │
                                ▼
        +------------------------------------------+
        |  A 段：my_model.pretrained（BERT）        |
        |  预训练模型，参数冻结，负责理解语句        |
        +------------------------------------------+
                                │  [8, 768]
                                ▼
        +------------------------------------------+
        |  B 段：nn.Linear(768, 2)                 |
        |  自己写的下游层，把 768 维压成 2 维        |
        +------------------------------------------+
                                │  [8, 2]
                                ▼
                          Softmax → 有关系 / 没关系
```

读图说明：A 段就是**加载过来的 BERT 模型**，也就是 `my_model.pretrained` 那部分；B 段就是我们自己写的那一层全连接，把 BERT 处理后的 768 维表示转换成 2 维——因为句子关系做的是二分类。

A 段和 B 段的分工，就是“迁移学习”这四个字落到代码上的全部含义：**A 段是别人训练好的、搬过来直接用；B 段是自己按任务写的、需要从头训练的。**

#### 标签的两个约定

最后把标签讲清楚，这块有两个可以自己定的地方。

**第一，1 和 0 谁代表有关系。** 现在的约定是 1 表示两句有关系、0 表示没关系。这个约定不是死的，把 1 当“没关系”来用完全可以，只要整套逻辑里保持一致就行。

**第二，用不用布尔值。** 实际开发里做二分类，有人习惯用 `True` 和 `False` 当标签，这跟用 1 和 0 是一回事，模型算起来没有区别。

#### 一段值得听的工程建议

这三个案例放在大纲里，其实**正常只讲一个**——把第一个中文文本分类讲透就够了，剩下两个案例给点时间，本来是可以自己独立做出来的。

但有一个不该被跳过的动作，是**第一个案例一定要自己独立敲一遍**。敲完之后，第二个、第三个直接在第一个的基础上复制代码改，别从头再来。原因很实在：改代码本身就是一种锻炼——你要判断改哪一行、改完出错要能顺着报错信息锁定位置。等你把一个半成品扔给自己、能在一分钟内指出该动哪一行，这门手艺就真到手了。反过来，你要是每次都删了重写，就永远学不会“读别人的代码、定位要改的地方”这件事。

整个 NLP 阶段的代码到这里就写完了：先拿数据加载器，再把模型搭好，然后训练，最后评估。四个动作，三个案例，同一套骨架。

---

## 模块 15 全景总结与技术沉淀

本全书系统整合了 迁移学习三大中文案例：情感分类、完形填空与 NSP 模块的 13 个核心专题（P171 ~ P183）。
建议读者在学完本章后，对照 `notes/` 目录下的思维导图树状笔记进行复盘与知识自测，巩固底层机理与工程实践能力。
