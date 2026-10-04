# 进阶数据结构和算法-261-Huffman-Tr

## 频次与路径长度的对应关系

* **哈夫曼树（Huffman Tree）**

    * > **定义**：一棵二叉树，它的字符全部存放在叶子节点上，构造原则是让频次高的字符路径短、频次低的字符路径长。

    * 频次高的，希望它的路径短；频次低的，希望它的路径长。这句话要靠算法真的干出来，而不是只停留在口头上。

    * 让频次低的路径变长的具体手段：给它们"找个爹"，凭空多出一层父节点，两个低频字符的路径各增长一层。

    * 示例中 `c` 一路向右只走了 1 层，频次最低的 `a` 走了 2 层，频次最低者路径最长这一效果就落地了。

    * 示例中最终树形为：根 `(10)` 的左孩子是 `(3)`、右孩子是 `c(7)`；`(3)` 的两个孩子是 `a(1)` 和 `b(2)`。

* **示例字符集**

    * 沿用的字符串里，`a` 出现 1 次，`b` 出现 2 次，`c` 出现 7 次。

    * 对应的路径是 `a` 走左→左、`b` 走左→右、`c` 只走右，对应编码分别为 `00`、`01`、`1`。

---

## 四步构建循环

* **第一步：字符与频次入队**

    * 统计每个字符的出现频率，把字符及其频率放入优先队列（PriorityQueue）。

    * > **注意**：队列里认为**频次低的优先级更高**，所以本例中优先级最高的元素是 `a:1`。

* **第二到四步：合并与回填**

    * 每次从队里取出**两个频次最低的元素**出队，本例第一次取出的是 `a:1` 和 `b:2`。

    * 给这两个元素"找个爹"：`a:1` 当左孩子，`b:2` 当右孩子，爹的频次记成两个孩子频次之和，即 $1 + 2 = 3$。

    * 把新建的父节点重新放回队列，重复上述步骤；第二次出队的是 `(3)` 和 `c:7`，新父节点频次为 $3 + 7 = 10$。

    * 循环退出条件是**队列里只剩一个元素**——没有两个可取的了，把这唯一剩下的元素出队，它就是最终的根节点 `root`。

* **本例的队列轨迹**

    * 每一轮都是"出队两个 → 新建父节点 → 放回队列"的固定动作，队列长度每轮减一。

    ```text
    第 1 轮：出队 a:1, b:2   →  新建父节点 (3)   →  放回队列
             队列：[c:7]  [(3)]

    第 2 轮：出队 (3), c:7   →  新建父节点 (10)  →  放回队列
             队列：[(10)]

    剩 1 个元素：出队 → root = (10)
    ```

* **为什么必须"找个爹"**

    * 给 `a:1` 和 `b:2` 找了爹之后，它们的路径就变长了，而频次低的字符本来就希望路径长，找爹正好加了一层路径。

    * 每合并一次，低频字符就被多挂一层，这是把设计原则转成可执行动作的关键环节，而不是单纯的数据结构操作。

---

## 待交付的四件事

* **树的构建与编码产出**

    * **构建哈夫曼树**：根据一个原始字符串把这棵树创建出来。

    * **找到每个字符对应的数字编码**：本例中 `a` 全程向左走所以编码是两个 `0`，`b` 先左后右所以编码是 `01`，`c` 编码是 `1`。

* **比特统计与编解码**

    * **统计比特数**：原始字符串编码之后一共占用多少个比特。

    * **编解码**：把原始字符串编码出去，再能正确还原回来。效果是 `a` 编成 `00`、两个 `b` 编成两段 `01` 即 `0101`、七个 `c` 编成七个 `1`。

---

## 节点类的设计

* **存储树有两种做法**

    * 一种是定一个节点对象，里面有左孩子、右孩子，还可以存数据值，比如存它是哪个字符、出现频次是多少。

    * 另一种是用数组存树中每个节点，因为哈夫曼树是**满二叉树**，满二叉树的性质跟之前学过的完全二叉树类似，用数组也能构建。

    * 这里选择第一种，建立一个节点对象。

* **节点类的属性**

    * `Character ch`：存对应的字符，父节点不存字符、存 `null` 也完全可以，因为只有叶子节点的字符才有编码。

    * `Integer freq`：存频次；为了让比较器的方法引用写法顺眼，把 `getFrequency` 重构成首字母小写的 `freq()`，去掉 `get` 前缀。

    * `left` 与 `right`：两个孩子，是必须的属性。

    * `String code`：记录它对应的实际编码，比如 `a` 是 `00`、`b` 是 `01`、`c` 是 `1`。

    * 构造方法加两个：一个根据字符构造供叶子节点用，另一个根据频次和左右孩子构造供父节点用。

* **叶子判定只看左孩子**

    * 判断是不是叶子的方式是**看它有没有左孩子**，左孩子是 `null` 就表示是叶子。

    * `a`、`b`、`c` 都没有孩子，所以是叶子；3 和 10 的左孩子不为空，所以是非叶子。

    * > **易错点**：只判断左孩子够不够？**够的**。因为哈夫曼树是满二叉树，有左孩子肯定也同时会有右孩子，所以判断一个就够了。

* **节点类代码**

    * `toString` 打印的就是字符和它的出现频次，其余 `get` 方法属于样板代码。

    ```java
    public class Node {
        // 字符
        private Character ch;
        // 频次
        private Integer freq;
        // 左孩子
        private Node left;
        // 右孩子
        private Node right;
        // 编码
        private String code;

        // 叶子节点：根据字符构造
        public Node(Character ch) {
            this.ch = ch;
        }

        // 父节点：根据频次 + 左右孩子构造
        public Node(Integer freq, Node left, Node right) {
            this.freq = freq;
            this.left = left;
            this.right = right;
        }

        public Integer freq() {
            return freq;
        }

        public void setFreq(Integer freq) {
            this.freq = freq;
        }

        public String getCode() {
            return code;
        }

        public void setCode(String code) {
            this.code = code;
        }

        // 满二叉树，有左孩子必然也有右孩子，所以只看左边就够了
        public boolean isLeaf() {
            return left == null;
        }

        @Override
        public String toString() {
            return "Node{ch=" + ch + ", freq=" + freq + "}";
        }
    }
    ```

    * > **提示**：这里用 `String` 作为编码字段是为了拼接方便；实际编码应该是一串 0 和 1 的二进制位，实现起来会复杂一些，这里简化成用字符串存储 `0`、`1` 来表示。

---

## 频次统计

* **统计入口与数据结构**

    * 给外层的哈夫曼树类加一个构造方法，把原始字符串作为参数传进来，同时存成成员变量——**编码时还要用到它**。

    * 拿到字符串后调用 `toCharArray()` 得到 `char` 数组再遍历，字符串里的每个字符就遍历出来了。

    * 借助一个 `Map` 集合统计最快，准备 `HashMap<Character, Node>`，`key` 是字符、`value` 是对应的节点对象；这个 `map` 编码时也要遍历，同样做成成员变量。

    * 逻辑是先检查 `map` 中有没有该字符，没有就把字符作 `k`、新建的初始 `Node` 作值放进去；有了就拿到它的 `Node` 对象让频次加一。

* **基础写法：containsKey 与 get**

    * 先判存在再取值，命中已有节点时频次自增；跑完 `a` 出现 1 次、`b` 出现 2 次、`c` 出现 7 次，全部正确。

    * 此时刚生成的这些 `Node` 对象就是叶子节点的状态，打印出来可以核对字符和频次统计得对不对。

    ```java
    public class HuffmanTree {
        // 原始字符串，编码时还要用到
        private String original;
        // key 是字符，value 是对应的节点，编码时还要遍历它
        private HashMap<Character, Node> nodes;

        public HuffmanTree(String original) {
            this.original = original;
            this.nodes = new HashMap<>();

            for (Character c : original.toCharArray()) {
                if (!nodes.containsKey(c)) {
                    nodes.put(c, new Node(c));
                }
                Node node = nodes.get(c);
                node.setFreq(node.freq() + 1);
            }
        }
    }
    ```

* **等价写法：computeIfAbsent**

    * `computeIfAbsent` 接受两个参数：第一个是要处理的 `k`，也就是这个字符；第二个是一个 `Function`，当 `map` 中还没有这个 `k` 时返回它要用的值，可以写成方法引用 `Node::new`。

    * 整行含义是：如果 `map` 中缺失了这个 `key`，就创建一个 `Node` 对象放进 `map`；没有缺失就不执行创建操作，直接返回已有的值。

    * 拿到返回的 `Node` 后直接增加它的频次，**两行代码就搞定了**，与基础写法作用完全一致。

    ```java
    for (Character c : original.toCharArray()) {
        Node node = nodes.computeIfAbsent(c, Node::new);
        node.setFreq(node.freq() + 1);
    }
    ```

---

## 排队与合并

* **优先队列的初始化**

    * 准备一个 `PriorityQueue`，队列里直接放 `Node`节点对象就行，因为节点里字符、频次都已经包括了。

    * 不自己写循环，直接拿 `map` 的 `values` 整个集合传给队列的 `addAll`，让它内部循环着放好。

    * > **注意**：别忘了给优先级队列一个**比较器**来规定比较规则；现在要按节点的**频次属性**比，所以方法引用就是 `freq`。

    ```java
    PriorityQueue<Node> queue =
            new PriorityQueue<>(Comparator.comparingInt(Node::freq));
    queue.addAll(nodes.values());
    ```

* **构建循环的写法**

    * 循环条件是队列的 `size` **大于等于 2**，因为每次都要出队两个元素，剩一个时就该退出。

    * 调用两次 `poll`，第一个出队的叫 `x`、第二个叫 `y`；父节点频次就是两个孩子频次之和，创建时调用三参数构造，`x` 是左孩子、`y` 是右孩子。

    * 父节点放回队列用 `offer`；循环结束后队列只剩一个节点，最后再 `poll` 一次拿到根节点 `root`。

    * > **提示**：父节点对象存什么字符不重要，字符为 `null` 也没问题，因为只有叶子节点存的字符才有编码。

    ```java
    PriorityQueue<Node> queue =
            new PriorityQueue<>(Comparator.comparingInt(Node::freq));
    queue.addAll(nodes.values());

    while (queue.size() >= 2) {
        Node x = queue.poll();
        Node y = queue.poll();

        // 爹的频次 = 两个孩子频次之和，x 是左孩子，y 是右孩子
        Node parent = new Node(x.freq() + y.freq(), x, y);

        queue.offer(parent);
    }

    Node root = queue.poll();
    ```

* **构造结果的验证点**

    * 打个断点跑一遍：最终生成的 `root` 频次是不是 10。

    * 展开 `root` 后左孩子是不是频次 3 的那个父节点、右孩子是不是 `c:7`。

    * 再展开左边，两个孩子是不是 `a:1` 和 `b:2`；三点都对上说明这棵树已经正确生成。

---

## 深度优先遍历求编码

* **递归骨架**

    * 求编码的做法是对这棵二叉树做一次遍历，等遍历到叶子节点时，根据它一路走来的路径就知道编码了。

    * 用一次**深度优先遍历**（Depth-First Search，DFS），最简单的方式就是递归，写一个方法 `dfs`，刚开始把根节点传给它。

    * 递归终点是**只要递归到叶子节点就可以返回了**，到达叶子时做最终处理——找到它的编码。

    * 如果不是叶子节点，比如这里的 3，就分别递归它的左孩子和右孩子。

* **编码由向左向右拼出**

    * 编码就是从根到该叶子的路径上，每一步"向左记 0、向右记 1"拼出来的字符串。

    * 利用一个初始为空的 `StringBuilder`：**向前走**时向左就在尾部 `append` 一个 `0`、向右就 `append` 一个 `1`；**向回走**时就把刚才加的那个字符减掉。

    * 一句话总结：向回走就减掉一个字符，向前走就根据向左还是向右加上 `0` 或者 `1`。

    * 回到 3 再回到 10 的过程中把加过的字符依次减完，右子树还有一个 `c`，向右加一个 `1` 就得到 `c` 的编码 `1`，之后同样减掉。

* **本例的遍历过程**

    * 进入非叶子节点时 `code` 不变，每下探一层就追加一位，回溯时删掉末位。

    ```text
              10
            /    \
           3      7
          / \     |
         a   b    c

    动作              进入时 code    到达叶子时的 code
    根 → 左(3)        ""              ""
      3 → 左(a)       "0"             "00"     ← a 落袋
      回溯(减末位)     "00"            "0"
      3 → 右(b)       "0"             "01"     ← b 落袋
      回溯(减末位)     "01"            "0"
    回根(再减末位)     "0"             ""
    根 → 右(c)         ""             "1"      ← c 落袋
      回溯(减末位)     "1"             ""
    ```

* **求编码的 dfs 代码**

    * 给 `dfs` 加一个参数，`new` 一个 `StringBuilder` 传进去，参数名就叫 `code`，表示当前累积的路径编码。

    * 每次递归方法调用结束之后（即向回走时）就删掉最后一个字符，调用 `deleteCharAt`，索引是 `code.length() - 1`，因为长度减一就是最后一个字符的位置。

    ```java
    public void dfs(Node node, StringBuilder code) {
        // 递归终点：叶子节点
        if (node.isLeaf()) {
            node.setCode(code.toString());
            return;
        }

        code.append("0");
        dfs(node.left, code);
        if (code.length() > 0) {
            code.deleteCharAt(code.length() - 1);
        }

        code.append("1");
        dfs(node.right, code);
        if (code.length() > 0) {
            code.deleteCharAt(code.length() - 1);
        }
    }
    ```

    * > **易错点**：`deleteCharAt` 的索引要加前置条件——只有 `code.length() > 0` 时才删。整趟遍历走到最外层回溯收尾时 `StringBuilder` 已经被减空，再执行 `deleteCharAt(code.length() - 1)` 等于去删下标 $-1$，会直接越界。

* **把编码存回节点而不是打印**

    * 知道叶子节点的编码后**不用把它打印出来**，把它记录到节点对象的 `code` 属性里即可。

    * 遍历结束后再统一打印：遍历 `map` 的 `values`，打印节点以及它对应的编码。

---

## 比特数统计

* **统计口径与公式**

    * 每个字符的编码是多少、每个字符的出现频次是多少都已经知道了，把两者的乘积逐字符累加就是总比特数。

    * 总比特数等于所有字符的"频次 × 编码长度"之和：$\text{bits} = \sum_{\text{字符 } x} \text{freq}(x) \times \text{len}(\text{code}(x))$。

    * 实现上不用另写函数，在求编码的那次深度优先遍历里顺手做就行。

* **本例逐字符账目**

    * `a` 编码 `00` 出现 1 次，占 $1 \times 2 = 2$ 个比特；`b` 编码 `01` 出现 2 次，占 $2 \times 2 = 4$ 个比特；`c` 编码 `1` 出现 7 次，占 $7 \times 1 = 7$ 个比特。

    * 三个字符的频次合计 10，占用比特合计 13。

    | 字符 | 频次 | 路径 | 编码 | 编码长度 | 占用比特 |
    | --- | :--- | :--- | :--- | :--- | :--- |
    | a | 1 | 左 → 左 | `00` | 2 | $1 \times 2 = 2$ |
    | b | 2 | 左 → 右 | `01` | 2 | $2 \times 2 = 4$ |
    | c | 7 | 右 | `1` | 1 | $7 \times 1 = 7$ |
    | **合计** | **10** | | | | **13** |

* **累加式 dfs 代码**

    * 先定义变量 `bits` 初始为零；对叶子节点来说，它的值更新成**节点的出现频次乘以编码的长度**，这就是一个字符占用的比特数。

    * 对非叶子节点，它应该累加由两个子节点返回的数，例如 3 这个节点自己频次不参与计算，但要负责把 `a` 返回的 2 和 `b` 返回的 4 加起来。

    ```java
    public int dfs(Node node, StringBuilder code) {
        int bits = 0;

        if (node.isLeaf()) {
            // 叶子：频次 × 编码长度
            bits = node.freq() * code.length();
            node.setCode(code.toString());
        } else {
            code.append("0");
            bits += dfs(node.left, code);
            if (code.length() > 0) {
                code.deleteCharAt(code.length() - 1);
            }

            code.append("1");
            bits += dfs(node.right, code);
            if (code.length() > 0) {
                code.deleteCharAt(code.length() - 1);
            }
        }

        // 只能在方法最后 return，否则上面的回溯删除就执行不到了
        return bits;
    }
    ```

    * > **易错点**：在叶子分支里直接 `return` 比特数**不行**。一旦在这里返回，下面那个删除字符的操作就不执行了；所以先不着急返回，得在方法的最后再返回，同时方法返回值类型也要从 `void` 改成 `int`。

    * 跑完取返回值打印：`a` 占 2 个、`b` 占 4 个、$c$ 占 7 个，加起来正好 13 个比特。