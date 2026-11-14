# Milvus 向量数据库：索引、读写与混合检索

## 字段划分与两类向量

### 标量字段与向量字段的分界

* **标量字段（Scalar Field）**

    * > **定义**：向量字段以外的一切字段都叫标量字段，判定标准只有一条——有没有被转成向量。

    * 一篇文章的标题、作者、图片链接、正文原文在转向量之前全是标量字段；正文一旦被模型转成向量，那个字段就脱离标量范畴，成为向量字段。

    * 一条切片入库后字段会膨胀：`product_name`、`category`、`content` 三个标量字段原样保留，`content` 再派生稠密、稀疏两个向量字段，合计五个。

    * 标量字段各有分工：`product_name`、`category` 将来做条件过滤，`content` 是检索出来后喂给模型的内容，两个向量字段只负责检索。

* **三类字段对应三套索引**

    * 稠密向量字段加速语义检索，稀疏向量字段加速关键词检索，标量字段的索引底层可用倒排、Bitmap 等实现，思路与 MySQL 索引一致。

    * 建 collection 时只给将来要搜索、要过滤的字段加索引，用不到的字段别加；主键自带默认索引，不重复加。

### 稠密向量与稀疏向量

* **稠密向量（Dense Vector）**

    * > **定义**：基于向量相似度实现语义检索的向量叫稠密向量。

    * 判定"稠"看的是有值元素的占比，不是数组长度：1024 个位置全部有值才叫稠密，大部分位置为空就是稀疏。

    * 常见嵌入模型输出 1024 维，维度在 schema 中写死。

    * 强项是换个说法也能找到：用户问"什么情况下保险不能赔"，能召回写着"遇到以下情形之一，不予理赔"的条款。

    * 弱项是一字之差的专业名词被语义带偏：药名、病名、书名这类词，语义越"聪明"错得越离谱。

* **稀疏向量（Sparse Vector）**

    * > **定义**：以词表编号为下标、只在词出现的位置记权重的向量，绝大部分位置为 0，因此得名。

    * 构造前提是一份全局词表：统计语料里出现过的全部词，量级十几万到二十万，每个词对应一个索引编号。

    * 生成时新建一个与词表等长的数组，逐词查表，在对应编号位写权重（词频），文档里没出现的词记 0；一篇 200 字的切片只有几十个非零位。

    * 真实存储会把数组压成字典，键是词表编号、值是权重，只存非零位；形态变了但仍是向量，只是换了一种省空间的存法。

    * 检索就是关键词精确匹配：查词表拿到编号，再去每篇文档的对应位看有没有值；词表一一对应，两步定位，速度快、字面精准。

    * 强项是专有名词、保单编号一字不差；弱项是换个说法就搜不到。

* **两种向量的分工**

    | 对比项 | 稠密向量 | 稀疏向量 |
    | :--- | :--- | :--- |
    | 检索方式 | 语义检索 | 关键词检索 |
    | 匹配依据 | 向量距离／相似度 | 词有没有命中、命中多少 |
    | 强项 | 换个说法也能找到 | 专有名词、编号一字不差 |
    | 弱项 | 一字之差的专业词容易串 | 换个说法就搜不到 |

### 稀疏向量的两条生成路线

* **统计式生成**

    * 遍历文档统计词表与词频再做权重，代表算法是 BM25、TF-IDF。

    * 资源消耗少、速度非常快，边界是只认字面，换个说法匹配不上。

* **学习式生成**

    * 用模型训练生成稀疏向量，代表模型是 BGE-M3、SPLADE。

    * 吃 CPU／GPU、计算速度慢，但在关键词检索之上带一点同义词与语义分析能力。

    * 选型看需求：按词精确匹配、不想烧显卡选统计式；要同义词效果且算力充足选学习式。

    * > **提示**：两种向量互补，语义检索与关键词检索都需要，所以向量库里两种都存，这也是混合检索的由来。

## 索引体系

### 索引的三大类

* **索引的作用**

    * > **定义**：索引是为提高检索速度而给字段建立的辅助结构；字段可加索引、加了查询就快，索引本身分多种类型。

    * 与 MySQL 的类比：主键索引、唯一索引、全文索引、空间索引各管一类场景，Milvus 同样按字段类型分化出不同索引。

* **Milvus 的三类索引**

    * 稠密向量索引：加在稠密向量字段上，加速语义检索。

    * 稀疏向量索引：加在稀疏向量字段上，加速关键词检索。

    * 标量字段索引：加在向量以外的一切字段上，思路几乎等同 MySQL 索引。

    * > **提示**：记住"稠密有一堆、稀疏基本只有倒排、标量一大堆"这个格局即可，具体类型用到再查。

### 稠密向量的索引算法

* **FLAT 与 KNN**

    * KNN 把库里所有向量逐一与查询向量算相似度再排序；万级勉强可用，十万、百万、千万级不可行。

    * `FLAT` 是这种蛮力打法在 Milvus 索引里的名字：最直接、精度最高、效率最差，是以时间换精度的极端。

* **IVF_FLAT**

    * > **定义**：倒排文件扁平索引，建索引时用 K 均值聚类把向量空间划成若干区域，每区算一个中心点，检索时先比中心点锁定区域，再进区域内逐条精算。

    * 区域大小由自己定，一片可装十个、二十个、上百个向量；中心点是该片所有向量的平均位置。

    * 检索只跟各中心点比距离，锁定最近的一片后，把片内几百上千个向量逐条算相似度，取 Top K。

    * 上千万篇文档按每 100 或每 1000 条划片，可切成上万个区域，中心点本身有序，所以粗筛这一步很快。

* **HNSW**

    * 基于图的索引：按数据之间的关联关系把所有向量连成分层图，上层最粗、覆盖范围最大，越往下越精细。

    * 检索从上层开始逐层下探，每下一层收窄候选范围，最终需要比较的向量数量大幅减少，性能很强。

* **AUTOINDEX**

    * 不知道选哪种就填 `AUTOINDEX`，由 Milvus 按数据量自动判断，数据规模变化时底层索引也可能跟着变。

    * 建 collection 时索引类型填 `auto`，自动模式下大概率落到 `IVF_FLAT` 那套"建索引文件—分块—按块检索"的路子。

### 稀疏与标量字段的索引

* **稀疏向量的倒排索引**

    * 稀疏向量的索引基本只有一种：倒排索引，即从词出发反查哪些文档包含它，与稀疏向量的构造天然对应。

    * 建索引写 `SPARSE_INVERTED_INDEX`，度量方式写 `BM25`，除字段名外整段是固定写法。

    * > **易错点**：倒排索引只能用 `BM25` 做相似度计算，不能填内积；只有模型生成的稀疏向量才可以用内积。

* **标量字段索引**

    * 用同一个 `add_index`，只给 `field_name` 与 `index_type` 两个参数，不写 `metric_type`。

    * 标量字段不涉及向量相似度，索引类型同样可以填 `AUTOINDEX` 交给 Milvus 选。

## 客户端连接与配置

### pymilvus 客户端与端口

* **MilvusClient 初始化**

    * > **定义**：Milvus 官方 Python 客户端，先用它把建表、读写、检索的底层机制走通，再换 LangChain 封装。

    * `MilvusClient(uri=...)` 是入口；其余参数是用户名、密码、`db_name`，本环境未开权限校验，只传 `uri` 就够。

    * `uri` 不要手写，从配置里取。

* **两个端口的分工**

    * `9091` 是 Web UI，浏览器打开可看集群运行状态与 collection 信息。

    * `19530` 供代码访问。

    * > **易错点**：9091 给浏览器看状态、19530 给客户端连，两个端口不能串。

### 配置字段与地址拼装

* **host 与 port 落配置**

    * `.env` 里写 `milvus_host` 与 `milvus_port`，`config` 的 rag 配置里加同名字段读取它们。

    * 客户端要的是完整地址，用 `@computed_field` 加 `@property` 拼出 `milvus_url`，返回 `http://{host}:{port}`，例如 `http://192.168.150.101:19530`。

    ```python
    milvus_host: str
    milvus_port: int

    @computed_field
    @property
    def milvus_url(self) -> str:
        return f"http://{self.milvus_host}:{self.milvus_port}"
    ```

* **命名与同步**

    * 字段名带业务前缀：叫 `milvus_url` 而不是 `url`，一个配置类里可能不止一个地址，只叫 `url` 说明不了是谁的地址。

    * 配置里改名后调用处必须同步，两边不一致直接报错。

    * 换环境时通常只改 host，端口一般不动。

### 文档与嵌入模型备料

* **Markdown 切片**

    * 流程：读文件 → Markdown 标题切分器 → `strip_headers=False` 保留正文标题 → 切成 `Document` 列表。

    * 每个 `Document` 两部分：`metadata` 装 H1／H2／H3 标题层级，`page_content` 装该段正文。

    * 选 Markdown 是因为结构清晰，标题层级能完整留在 `metadata` 里供后续过滤。

* **嵌入模型**

    * 可用本地 Ollama，也可用阿里云；机器性能差就用云端，本地先确认模型服务已启动。

    * 用阿里云时要把 API Key 与 Base URL 补进 `llm_settings`，初始化向量模型时从 `settings` 取值传入，否则代码读不到。

## Collection 建表

### 三步建表与 schema 开关

* **三步建表法**

    * > **定义**：Milvus 建表分定义 schema、给字段加索引、创建 collection 三步，后一步拿前一步的产物当参数。

    * 与 SQL 的差别：一条 `CREATE TABLE` 能同时写完字段与索引，Milvus 用面向对象方式必须分开走。

    * `create_collection(name, schema, index_params)` 三个参数就够，名字起好一次，后两步一直复用。

* **create_schema 的两个开关**

    * `auto_id=False`：不自动生成主键，id 自己编好、自己传入。

    * `enable_dynamic_field=False`：禁止动态字段。开着时插入带 schema 没有的字段，Milvus 会自动补字段并推断类型；类型判断不可控，表结构也不再固定，所以一律关闭。

### 字段设计与类型约束

* **id 与标题字段**

    * `id` 用 `INT64`、`is_primary=True`、不自动生成；LangChain 侧 id 是字符串，写入 entity 时转成 `int`。

    * `metadata` 是字典，Milvus 没有对应类型，不能整坨塞入；只挑一个 `h2` 存进去，类型 `VARCHAR`、`max_length=128`，将来做过滤条件。

    * 只存 `h2` 不存 `h3` 可行，演示数据不必全量搬运；换成 `h3` 也一样。

* **正文字段**

    * `content` 对应 `page_content`，类型 `VARCHAR`、`max_length=2048`。

    * 它既是给人看的内容，也是生成两路向量的原料，必须原样保留。

* **两个向量字段**

    * `dense`：`FLOAT_VECTOR`、`dim=1024`，维度与嵌入模型输出一致且写死。

    * `sparse`：`SPARSE_FLOAT_VECTOR`，不写 `dim`，维度跟着词表动态变化。

    | 字段 | Milvus 类型 | 关键约束 | 数据来源 |
    | :--- | :--- | :--- | :--- |
    | `id` | `INT64` | 主键，不自动生成 | 文档编号转整数 |
    | `h2` | `VARCHAR` | `max_length=128` | `metadata["h2"]` |
    | `content` | `VARCHAR` | `max_length=2048`，允许分词 | `page_content` |
    | `dense` | `FLOAT_VECTOR` | `dim=1024` | 嵌入模型生成 |
    | `sparse` | `SPARSE_FLOAT_VECTOR` | 维度动态 | BM25 函数自动生成 |

    * `datatype` 用 `DataType` 枚举写（`DataType.INT64`、`DataType.FLOAT_VECTOR`），不写字符串，写错编辑器会提示。

    * schema 里的字段名、写数据时的字典键、建索引时的 `field_name`，三处必须完全一致。

### 分词与 BM25 函数

* **content 的分词配置**

    * `enable_analyzer=True` 允许该字段分词；`analyzer_params={"type": "chinese"}` 指定 Milvus 内置中文分词器。

    * 不开分词，`content` 只是一段普通字符串，Milvus 不会替它切词，BM25 无从谈起。

    * 配好后插入数据时，Milvus 自动完成"分词 → BM25 统计 → 稀疏向量"；稠密向量它不管，得自己调模型转。

* **BM25 Function 接线**

    * > **定义**：把某个文本字段自动转成稀疏向量字段的内置函数，只需声明输入、输出与函数类型。

    * `Function(name=..., function_type=FunctionType.BM25, input_field_names=["content"], output_field_names=["sparse"])`，再用 `schema.add_function` 挂进 schema。

    * `name` 可自定义（如 `text_bm25_emb`）；`output_field_names` 必须与已定义的稀疏字段同名。

    * > **提示**：运行时提示 `DataType` 找不到是漏导入，补 `from pymilvus import DataType`；`Function`、`FunctionType` 同理。

### 索引参数与建表调用

* **稠密与稀疏索引**

    * 稠密：`add_index(field_name="dense", index_type="AUTOINDEX", metric_type="COSINE")`。

    * 稀疏：`add_index(field_name="sparse", index_type="SPARSE_INVERTED_INDEX", metric_type="BM25")`。

    * `metric_type` 可选 `COSINE`（余弦）、`IP`（内积）、`L2`（欧式距离），必须与检索时保持一致。

* **标量索引与建表**

    * 标量：`add_index(field_name="h2", index_type="AUTOINDEX")`，不写 `metric_type`。

    * 三样齐了交给 `client.create_collection(collection_name=..., schema=schema, index_params=index_params)`，得到完整的表。

    * 封成 `create_collection` 函数，初始化客户端之后调一次即可。

### 查看与删除

* **list 与 drop**

    * `client.list_collections()` 无参返回全部 collection；`client.drop_collection(collection_name=...)` 按名字删除。

    * 记不住方法名时在 `client.` 后敲 `collection`，补全里就有 create、drop、list 三个动作。

* **Attu 图形界面**

    * Attu 部署在虚拟机，浏览器访问，连的是 `19530` 端口；首次访问是注册流程，账号密码自建，不是预设账号。

    * 进入数据浏览器可核对字段类型与 BM25 函数；表结构、字段类型、函数与代码一致，说明建表正确。

## Entity 读写

### Document 到 entity 的加工

* **两次加工**

    * `insert` 严格要求 entity 是字典或字典列表，LangChain 的 `Document` 结构对不上，必须先转换。

    * 第一步批量转向量：`texts = [doc.page_content for doc in docs]`，`dense_vectors = embedding.embed_documents(texts)`。

    * `embed_documents` 返回的向量顺序与传入文档顺序一条对一条，`dense_vectors[0]` 就是第 0 篇的向量。

    * 第二步拼字典：`{"id": i + 1, "h2": doc.metadata.get("h2"), "content": doc.page_content, "dense": dense_vectors[i]}`，可用列表推导式批量生成。

    * 字典里没有 `sparse`：它在写入时由 Milvus 按 BM25 函数自动补齐。

* **取值细节**

    * `metadata` 是字典，用 `doc.metadata.get("h2")` 或中括号，不能点号访问。

    * 正文是 `doc.page_content`，不是 `doc.content`。

    * > **易错点**：`enumerate` 从 0 起时 id 写 `i + 1`、向量仍取 `dense_vectors[i]`；若写成 `enumerate(docs, start=1)`，`dense_vectors[1]` 就配到第二篇，整批数据错位一条。

### insert 与 upsert

* **insert**

    * `client.insert(collection_name=..., data=entity_list)`，只认两个参数：往哪张表插、插什么（单字典或字典列表）。

    * 返回结果含 `insert_count`（成功条数）与每条对应的 id。

    * 写入时 Milvus 拿每条 `content` 走分词与 BM25 补 `sparse`，`dense` 用字典里算好的直接存。

* **upsert**

    * > **定义**：`up`（update）加 `sert`（insert）的合成词，底层动作是先按 id 删掉旧记录、再插入新记录。

    * id 存在就是更新（旧记录删掉、新记录顶上），id 不存在就只剩插入，因此同时具备更新与新增两种语义。

    * 参数与 `insert` 完全一样，返回 `upsert_count`；实测一批两条可一条更新、一条新增。

    * 分不清 insert 与 upsert 时一律用 upsert，前提是 id 由自己控制得住，否则它无法判断该覆盖谁。

    * > **注意**：验证必须去 Attu 刷新看，不能盯本地 `entity_list`——那只是本地拼的列表，库里的改动不会回写。

### delete 与三种写操作

* **delete**

    * `client.delete(collection_name=..., ids=[88])`，`ids` 必须是列表，删一条也要包成列表，不能直接传数字。

    * 返回 `delete_count`；删一个已不存在的 id 无任何效果。

* **三种写操作对比**

    | 操作 | 方法 | 语义 | id 已存在 | id 不存在 |
    | :--- | :--- | :--- | :--- | :--- |
    | 新增 | `insert` | 只负责插入 | 不在处理范围 | 插入新记录 |
    | 更新或新增 | `upsert` | 先删同 id 旧记录，再插入 | 覆盖旧记录，等同更新 | 插入新记录 |
    | 删除 | `delete` | 按主键移除 | 删除该记录 | 无效果 |

## 向量检索

### 近似近邻

* **ANN 与 KNN 的分野**

    * > **定义**：ANN（Approximate Nearest Neighbor，近似近邻）先借索引快速定位"可能最相似"的一小部分向量，只在这一小部分里精算，用少量精度换大幅提速。

    * KNN 把向量空间里所有向量拎出来逐一比较再排序，一万条勉强能忍，十万、百万、千万级不可行。

    * `FLAT` 就是这种蛮力打法在索引体系里的名字：最直接、精度最高、效率最差。

* **ANN 的两条实现路线**

    * `IVF_FLAT`：先聚类分区、比中心点锁定一片、片内逐条精算。

    * `HNSW`：分层图逐层下探，越往下候选越少。

    * 真正使用时不必关心底层，指定索引方式后剩下的交给 Milvus 自动完成。

### search 参数

* **三个必填参数**

    * `collection_name`：搜哪个 collection，对应 SQL 的 `from 表名`。

    * `data`：问题向量，必须是列表；一个向量本身是列表，传多个向量就是列表的列表，支持批量查询，返回也按问题分组。

    * `anns_field`：按哪个向量字段搜，稠密传 `dense`、稀疏传 `sparse`。

* **常用与可选参数**

    | 参数 | 是否必填 | 作用 | 对应 SQL |
    | :--- | :--- | :--- | :--- |
    | `collection_name` | 必填 | 搜哪个 collection | `from 表名` |
    | `anns_field` | 必填 | 拿哪个向量字段比 | `where` 里的字段 |
    | `data` | 必填 | 问题向量（列表；多个即批量） | `like` 后面的值 |
    | `limit` | 常用 | 返回多少条相似结果 | `limit 10` |
    | `output_fields` | 常用 | 结果里带回哪些字段 | `select` 的列 |

    * `limit`：默认按相似度得分降序，`limit=3` 即 Top 3。

    * `output_fields`：必须逐个点名字段，不能写 `*`；不传默认只返回 id 与得分，正文不会回来。

    * `search_params`：检索扩展参数，最常配 `metric_type`；`offset` 与 `limit` 搭起来做分页。

    * `filter`：向量搜索前先做一次标量过滤，缩小搜索范围。

    * `order_by_fields`：排序字段，默认一定不要传；一传就按该字段排序，与相似度无关。

    * `ids`：按 id 精确匹配过滤，传了就不再走向量查询，与 `data`、`anns_field` 互斥。

* **度量方式必须一致**

    * `search_params` 里的 `metric_type` 必须与建索引时的度量方式相同：建索引用 `COSINE`，检索就得用 `COSINE`；用 `IP` 同理。

    * 两边不一致，算出来的分数没有意义。

### 稠密检索

* **三步骨架**

    * 第一步 `embedding.embed_query(query)` 把问题转成问题向量；`embed_query` 转单条，`embed_documents` 批量转文档，分工不同。

    * 第二步 `client.search(collection_name=..., data=[query_vector], anns_field="dense", limit=3, output_fields=["h2", "content"], search_params={"metric_type": "COSINE"})`。

    * 第三步两层循环打印：`for hits in result` 取某个问题的结果集，`for hit in hits` 才是一条文档。

    ```python
    def dense_search(query):
        query_vector = embedding.embed_query(query)
        result = client.search(
            collection_name=collection_name,
            data=[query_vector],
            anns_field="dense",
            limit=3,
            output_fields=["h2", "content"],
            search_params={"metric_type": "COSINE"},
        )
        for hits in result:
            for hit in hits:
                print(hit)
    ```

* **返回结构与复用**

    * `result` 是列表，每项对应一个查询向量的命中集；只传一个问题它照样是列表，所以必须两层遍历。

    * `output_fields` 里不必带 id、dense、sparse：向量字段返回只是一堆数字，正文与标题才是有用内容。

    * 复用时要改的只有四样：`anns_field`、`limit`、`metric_type`、`output_fields`，其余固定。

    * 这套代码不是稠密专用骨架：换检索字段、换传入的向量、换度量方式，它就变成另一种检索。

### 稀疏检索

* **与稠密检索的两处差异**

    * `data=[query]` 直接传原始问题字符串，不转向量——建表时挂的 BM25 函数会在库内代转。

    * `anns_field="sparse"`、`metric_type="BM25"`，其余参数与循环结构都不动。

* **实测表现**

    * 语义型问题（"教育的负面作用是什么"）稀疏检索召回不到讲"负向功能"的那篇，因为字面没有重合词。

    * 专有名词型问题（"给教师的建议是谁写的"）稀疏检索直接命中排第一且得分很高，稠密检索反而把那本书挤到第三。

    | 维度 | 稠密检索 | 稀疏检索 |
    | :--- | :--- | :--- |
    | 检索字段 | `dense` | `sparse` |
    | 度量方式 | `COSINE`（须与建索引一致） | `BM25` |
    | `data` 传什么 | 问题向量，自己转 | 原始问题字符串，库内函数代转 |
    | 擅长 | 语义相近，换个说法也能找到 | 专有名词、关键词精确命中 |
    | 短板 | 专有名词被语义带偏 | 同义改写直接漏掉 |

## 混合检索与融合重排

### 多路召回流程

* **两步流程**

    * > **定义**：多路召回指用稀疏、稠密等多种方式分别查询，得到多个结果集；融合重排指把这些结果集合并成一个列表，重新打分、重新排序。

    * 多路不止两路：稀疏加稠密是最常用的起步组合，复杂场景还有多模态检索（图搜图、音频、视频），本质都是转成向量再检索。

    * 融合要解决的核心问题是：同一篇文档在两路都出现但排名不同，以谁为准、优先级怎么定。

* **三步固定骨架**

    * ① 创建多路请求：每路封装一个 `AnnSearchRequest`，稠密路 `data=[embed_query(query)]`、`anns_field` 用稠密字段、`param={"metric_type": "COSINE"}`、`limit=3`。

    * ② 稀疏路用同一个类：`data=[query]`、`anns_field` 换稀疏字段、`param={"metric_type": "BM25"}`、`limit=3`。

    * ③ 发起请求：`client.hybrid_search(collection_name=..., reqs=[dense_req, sparse_req], ranker=ranker, limit=2, output_fields=["content"])`。

    * 请求对象不传 `collection_name` 与 `output_fields`，因为还没执行；这两项在 `hybrid_search` 外层统一指定，为所有路共用。

    * 两个 `limit` 含义不同：请求里的 `limit=3` 是每路各搜 3 条（合计 6 条候选），`hybrid_search` 的 `limit=2` 是从 6 条里最终选 2 条。

    * `ranker` 就是重排器：`rank` 是排列、加 `-er` 是排序器、前面加 `re-` 是再一次，Milvus 简写成 `ranker`。

    * > **提示**：把 `ranker` 做成函数参数而不是写死，同一段代码换 ranker 就切换融合方案；复用时只需改检索字段与检索类型。

### 加权融合与归一化

* **加权求和**

    * > **定义**：给每一路召回分配权重，按文档在各路中的得分加权求和，用求和结果重新排序。

    * 两路公式：$score(d) = w_{dense}\cdot s_{dense}(d) + w_{sparse}\cdot s_{sparse}(d)$，权重满足 $w_{dense}+w_{sparse}=1$。

    * 例：权重 0.6 比 0.4，稠密得分 0.5、稀疏得分 0.8，最终得分 $0.5\times0.6 + 0.8\times0.4 = 0.62$。

    * 支持多路，但权重总和必须为 1，路一多就得切得更细。

* **分数尺度问题**

    * BM25 的分值往往落在 0 到 10（甚至更大），余弦相似度落在 0 到 1（有时 -1 到 1），两者不是一个量级。

    * 例：稀疏 5 分乘 0.4 得 2，稀疏 6 分乘 0.4 得 2.4，稠密哪怕 0.9 乘 0.6 也只有 0.54，差一个数量级还不止，权重调得再讲究也没用。

    * > **结论**：每一路得分分值差异较大时，直接加权求和就会失真，做加权求和之前必须先归一化。

* **归一化方法**

    * `min-max`：用最小值、最大值把区间线性拉到 [0,1]，最直观，但受极值影响。

    * `z-score`：减去均值再除以标准差，对分布敏感。

    * `softmax`：取指数后归一，会放大高分者。

    * 流程固定为"先归一、后加权"：两路原始分数先统一压到 0~1（或 -1~1），再乘权重求和，最后按结果排序，顺序不能换。

    * > **提示**：这些算法都是现成的，不必自己实现；`WeightedRanker` 内部已把归一化与加权求和整套做完。

* **WeightedRanker 用法**

    * `WeightedRanker(0.5, 0.5)`：按顺序传各路权重，两路传两个、多路传多个，总和为 1。

    * 实测两个查询分别把"教育的负向功能"排到第二、"给教师的建议"排到第一，说明混合检索同时具备语义与关键词两套能力。

### RRF

* **倒数排名融合**

    * > **定义**：RRF（Reciprocal Rank Fusion，倒数排名融合）不关心文档得了多少分，只按文档在各路结果里的名次换算分数再求和。

    * 公式：$score(d) = \sum_{i=1}^{n} \frac{1}{k + rank_i(d)}$，`k` 是常量、通常取 60。

    * 例（两路、k=60）：文档 A 稠密第 1、稀疏第 6，得 $\frac{1}{61} + \frac{1}{66}$；文档 B 稠密第 4、稀疏第 3，得 $\frac{1}{64} + \frac{1}{63}$，谁大谁靠前。

    * 排名越高分母越小、值越大，名次本身成了一种分数；算出的绝对值很小但排序只看相对大小。

    * `RRFRanker()` 的 `k` 默认 60；默认不等于最佳，需按召回结果反复调，绝大多数情况 60 够用。

    * > **易错点**：RRF 代入的是名次序号，不是原始得分，也不是把名次直接相加；两路各换算成一个倒数再求和。

* **取舍**

    * 优势：不用分配权重（加多少路都行）、不用归一化、对离群值不敏感、计算简单效率高、可融合任意数量列表。

    * 缺点一：对排名靠后的文档不敏感，$1/(k+rank)$ 尾部被压平，相邻名次差异越来越小。

    * 缺点二：所有检索源同权，无法让某一路更算数；要让稠密更重只能用加权。

    * 选型：路数会涨、懒得维护权重用 RRF；明确知道哪一路该更重用加权。

### Cross-encoder

* **双塔与交叉编码的分工**

    * > **定义**：Cross-encoder（交叉编码）把用户问题与文档一起输入同一个模型做交叉运算，直接输出两者的相关性得分。

    * bi-encoder（双塔编码、并行编码）：离线把文档转成向量、在线只转问题，查询开销仅一次 embedding；模型拿文档时没看到问题，只在最后拿向量碰面，信息在压成向量那步已损失一部分。

    * cross-encoder：每条候选都要与问题配成对跑一次完整前向计算，能看到关键词是否对上、语义是否偏移，得分与排名准确得多，代价是慢、无法离线预计算。

    * 定位：bi-encoder 做初筛（保证不漏），cross-encoder 做精排（保证排得准），不能拿它扫全库。

* **工业界常规链路**

    * 稀疏召回 10 条、稠密召回 10 条，合计 20 条候选，一起交给 cross-encoder 逐对打分重排，得到最终排名。

    * 这一步是模型在融合，既不是加权也不是 RRF，方案关键不在融合算法而在那个重排模型。

* **重排模型选择**

    * 千问系列如 Qwen3-Reranker-4B：上下文长度 32K，精度 70 多分，中英文俱佳。

    * bge 系如 bge-reranker-v2-m3：主打中英文混合，参数约 0.5B（五亿），体积小、占用资源少，社区最常用。

    * 另一类只擅长外语，中文表现明显掉分，中英混合语料上选它会被拖累。

    * 部署方式决定选择：公司内网本地部署选 bge-reranker（小、省资源、计算压力小），调云端模型选千问 rerank（最强、32K）。

    * 云端入口在阿里云百炼：模型广场 → 全部模型 → 文本向量模型。

### Milvus 内置重排

* **提供方与限制**

    * Milvus 天然支持 cross-encoder 重排，入口仍是 `ranker`，混合检索函数一个字都不用改。

    * 重排提供方分两类：自部署（vLLM 面向生产、只能跑 Linux；TEI 更轻量）与云平台（DashScope、SiliconFlow、HuggingFace）；Cohere 是专门的重排模型服务。

    * 云平台中只有 DashScope 是国产且免部署，本环境用它。

    * > **注意**：DashScope 作为重排提供方从 Milvus 3.x 起才支持，老版本这一路走不通；本环境是 3.0，落在支持范围内。

* **配置密钥与重启**

    * 配置位置在虚拟机的 `deployed/milvus/user.yaml`，找到 `credential.dashscope.api_key` 填入 Key。

    ```yaml
    credential:
      dashscope:
        api_key: <你的 DashScope API Key>
    ```

    * Key 是交给 Milvus 而不是业务代码，因为调用关系发生在 Milvus 与阿里云之间，业务代码不直接参与。

    * 改完必须重启 Milvus 才生效：`docker compose restart`。

    * > **易错点**：重启命令必须在 `deployed` 目录（`docker-compose.yml` 所在目录）下执行，换目录找不到要重启的容器。

* **用 Function 造重排器**

    * 重排器创建时必须带 query，因为模型要知道拿什么去跟文档比。

    ```python
    from pymilvus import Function, FunctionType

    def build_ranker(queries: list[str]) -> Function:
        return Function(
            name="dashscope_rerank",
            input_field_names=["content"],
            function_type=FunctionType.RERANK,
            params={
                "reranker": "model",
                "provider": "dashscope",
                "model_name": "gte-rerank-v2",
                "queries": queries,
                "batch_size": 32,
            },
        )
    ```

    * `input_field_names` 填原始文档正文字段，不是向量字段；送进模型的是原文与原始问题。

    * `params` 各键：`reranker="model"` 固定走模型重排；`provider` 随服务商变（dashscope／硅基流动／vllm）；`model_name` 填具体模型；`queries` 必须是列表；`batch_size` 是一次发多少条。

    * 将来要改的只有四处：provider、模型名、queries、文档字段名。

    * 调用：`hybrid_search(query, build_ranker([query]))`；两个 query 语义位置不同——一个决定搜什么，一个决定重排时拿什么跟文档配对。

    * 实测：加权与 RRF 下都只排第二的"教育的负向功能"，在模型重排下升到第一；第二个查询 200ms 左右返回。

### 三种方案取舍

* **能力对比**

    | 方案 | 排序依据 | 要不要归一化 | 要调什么 | 多路扩展 | 精度 | 速度 |
    | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
    | 加权融合 | 各路原始得分乘权重 | 要 | 权重（和为 1） | 差，路多权重难分 | 中 | 快 |
    | RRF | 各路名次换算的倒数 | 不要 | `k`（默认 60） | 好，任意路数 | 中 | 快 |
    | Cross-encoder | 模型对（query, document）打分 | 不要 | provider、模型名 | 好 | 高 | 慢 |

* **组合方式**

    * 前两种是"算分"：一个信分数、一个信名次，都靠数学拼结果，快但精度有限。

    * 第三种是"请外援"：把候选交给专业模型重判，准但慢且依赖外部服务。

    * 常见做法是串联：先用加权或 RRF 把候选缩到十几二十条，再用 cross-encoder 精排；候选越小，重排开销越可控。

    * 只在前两种里选：需要压某一路权重、明确语义更重用加权；路数会涨、懒得维护权重用 RRF。对最终排名有要求就把最后一棒交给 cross-encoder。

## 条件过滤

### filter 的作用

* **先过滤再检索**

    * > **定义**：向量检索负责"像不像"，filter 负责"是不是这个范围"，先用标量字段过滤出子集，再在子集里做向量检索。

    * 过滤的作用是缩小搜索范围、提高检索效率，它不参与相似度计算。

    * 保险商城场景：所有产品条款放同一张表，用户问某款产品的"等待期"，不过滤会召回所有产品的等待期条款，拿别家条款回答一定出错。

    * 正确做法是先用产品名过滤到具体产品，再在该产品的条款片段里检索与问题相关的那一段。

* **过滤的适用对象**

    * 只能基于标量字段（`h2`、`content` 都可），不能基于向量字段。

    * 前提是 schema 里存了这个字段；没入库的字段无从过滤。

### 过滤语法与用法

* **语法**

    * 语法与 MySQL 的 `where` 几乎一样，写成一个字符串，学习成本极低。

    * 支持 `=`、`!=`、`>`、`<`、`>=`、`<=`、`like`、`in`，以及正则、算术运算、位运算、逻辑运算 `and`／`or`。

    * 示例：`filter='chunk like "%red%"'`，先筛出 `chunk` 含 red 的文档，再在这批文档里做向量检索。

    * 引号写法：`like` 后的字符串用双引号，外层就用单引号，避免冲突；等值过滤的值带空格必须加引号。

* **接入检索**

    * 给检索函数加一个可选参数 `filter_query: str | None = None`，拼到 `search` 的 `filter` 上；不传即不过滤。

    * 混合检索同样能加：`filter` 传给稠密、稀疏两路，各自带上条件。

    * > **注意**：过滤参数必须可选，做成必传就等于每次都必须有过滤条件。

    * 实测：`content like "%给教师的建议%"` 把多条结果收窄到一条；`h2 == "教育的功能"` 同样只剩一条。

## LangChain Milvus 集成

### 封装边界与初始化

* **能力限制**

    * 混合检索只支持稀疏与稠密两种方式，想加更多方式不行。

    * 不支持多模态向量检索，不支持对普通字段加索引。

    * 稀疏检索只支持 BM25，不支持基于模型的稀疏向量检索。

    * 大多数场景（八九成）已经够用，换来的是代码量大幅下降。

* **Milvus vector store 初始化**

    * `Milvus(embedding_function=..., collection_name=..., builtin_function=BM25BuiltInFunction(), vector_field=["dense", "sparse"], connection_args={"uri": ...}, drop_old=True)`。

    * `embedding_function` 让 vector store 自带向量化能力，`add_documents` 直接传文档，不用手动转向量。

    * `builtin_function=BM25BuiltInFunction()` 替代手写 BM25 Function，BM25 已被 LangChain 内置。

    * `vector_field` 按位置识别：第一个是稠密字段名、第二个是稀疏字段名，顺序不能反；`text` 里的原文用向量模型转稠密、用 BM25 转稀疏。

    * `index_params` 不传时自动给稠密、稀疏字段加索引；要自定义索引才需指定。

    * `drop_old=True` 在创建时自动删除已有 collection，避免重复创建冲突。

    * 指定的 `collection_name` 不用手动创建。

### schema 自动推导

* **字段映射规则**

    * schema 由 LangChain 依据 `Document` 里的数据自动生成，不需要手填。

    | Document 里的部分 | collection 字段 | 说明 |
    | :--- | :--- | :--- |
    | `id` | `pk` | 固定字段名，不叫 id；不提供则由 Milvus 生成 |
    | `page_content` | `text` | 固定字段名，存原始文本 |
    | `metadata.h1/h2/h3` | `h1`/`h2`/`h3` | metadata 摊平成普通标量字段 |
    | `text` 加向量模型 | `dense` | 名字自定义，值自动生成 |
    | `text` 加 BM25 | `sparse` | 名字自定义，值自动生成 |

* **推导结果与时机**

    * 一条记录最终七个字段：`pk`、`text`、`dense`、`sparse`、`h1`、`h2`、`h3`。

    * 不提供 id 时字段名仍是 `pk`，值由 Milvus 自动生成（实测从 90 起递增）；提供了 id 就用其值，但字段名不变。

    * 自动加索引的只有向量字段，标量字段不加。

    * > **注意**：vector store 对象创建好时 collection 还没建，因为 schema 要依据 Document 数据推导，要等 `add_documents` 之后才真正建表。

### 基本读写

* **新增与删除**

    * `vector_store.add_documents(docs)`：文档不带 id 时 Milvus 自动生成并给出警告，不用转向量、不用做任何处理。

    * `vector_store.delete(ids=[90])`：声明要求字符串，实际传数字也能删成功，属于类型标注不严谨，不必手动 `str()` 一层。

    * 方法与 `InMemoryVectorStore` 完全一致：`add_documents`、`delete`、`similarity_search`。

* **带分检索**

    * `similarity_search_with_score(query=..., k=4)` 返回元组列表，顺序是先文档后得分，unpack 必须按这个顺序。

    * 打印时写成 `for doc, score in result`；写反了就取不到文档。

### 混合检索参数

* **rank_type 与 rank_params**

    * `rank_type` 只支持 `"weighted"` 与 `"rrf"`，传的字符串就是融合方案名。

    * `rank_params` 随方案变化：weighted 传 `{"weights": [0.4, 0.6]}`，rrf 传 `{"k": 60}`。

    * 两种方案的得分尺度不同：加权得分在 0~1 附近，RRF 得分偏小（零点几），但排名仍可正常使用。

    * > **注意**：一旦加入稠密与稀疏向量，LangChain 默认开启加权混合检索，纯稠密检索与纯稀疏检索都做不了，这是封装的限制。

* **书写风险**

    * `rank_type` 没有类型提示、IDE 不补全，容易拼错（`weighted` 敲成 `waited`）。

    * > **提示**：这类参数不要手敲，直接复制现成写法，改数值即可。

### 分词器问题

* **现象与原因**

    * 现象：加权检索"给教师的建议是谁写的"，结果只排第二，不符合预期。

    * 原因：`text` 字段默认使用标准分词器（standard），中文关键词切不准，BM25 的关键词命中失真。

* **修法与验证**

    * 修法：`BM25BuiltInFunction(analyzer_params={"type": "chinese"})`——分词器在这个内置函数里指定，不在字段上。

    * 改完要重建 collection（`drop_old=True` 或手动删掉重来）才生效。

    * Attu 里字段仍显示 standard 是界面显示滞后，不能当唯一判据；用检索结果验证最可靠——同一个查询立刻升到第一。

    * > **易错点**：不指定中文分词器就跑中文混合检索，切词错误不会报错，只表现为排名错乱。

### 自定义重排器

* **为何要自己造**

    * LangChain 的 `rank_type` 不支持 cross-encoder，必须从 Milvus 官方 SDK 取 `Function` 自己定义 ranker。

    * `input_field_names` 要写 `text` 而不是 `content`，因为 LangChain 把原始文档存在 `text` 字段。

* **封装与调用**

    * 封装 `cross_encoder_search(query, expr=None)`：第一步把 query 包成列表造 ranker；第二步 `similarity_search(query=query, ranker=ranker, expr=expr)`。

    * 调检索时不再传 `rank_type`、`rank_params`，改传 `ranker`；`k` 与 `fetch_k` 可不指定。

    * > **提示**：`ranker` 这个参数名官方文档里查不到，是从源码里翻出来的，只能照抄；加了 `expr` 参数后这个函数同时具备检索与过滤。

    * 实测两个查询都排第一，是目前精度最高的一路，也是封装里最复杂的一路。

* **LangChain 侧过滤**

    * 过滤参数叫 `expr`，不叫 `filter`，语法仍是 MySQL 那种写法。

    * `similarity_search(query="教育的功能", expr='h3 like "%代表人物%"')`，加后结果从多篇收窄到两篇。
