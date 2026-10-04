# 基础算法-160-B树-contains

## B 树容器的属性

### 最小度数与根节点

* **最小度数（minimum degree，t）**

    * > **定义**：整棵 B 树统一使用的最小度数，新建节点一律跟着树的 `t` 走。

    * B 树里所有节点的 `t` 值必须一致，新节点的最小度数直接由树的 `t` 决定。

    * 无参构造时 `t` 取默认值 2，因为 B 树节点至少得有两个孩子，不能再少了，`2` 正好是"至少两个孩子"的最小取值。

    * > **提示**：默认 `t = 2` 不是随手写死的。

* **根节点（root）**

    * > **定义**：整棵 B 树的入口节点，在构造方法里就被创建出来。

    * 构造时执行 `root = new Node(t)`，根节点一出生就非空。

    * 因此后续访问 `root` 时不需要做非空判断。

### 键数目的上下界

* **最小键数（MIN_KEY_NUMBER）**

    * k 的数目比孩子数目少一，而孩子的最小数是 `t`，所以 `MIN_KEY_NUMBER = t - 1`。

* **最大键数（MAX_KEY_NUMBER）**

    * 孩子的最大数是 `2t`，k 比孩子少一个，所以 `MAX_KEY_NUMBER = 2 * t - 1`。

    * 这两个值在树被创建时就固定不变，用 `final` 声明。

* **整体区间**

    * 一个节点的孩子数落在 $[t, 2t]$ 里，k 的数目比孩子数少一，于是 k 的数目范围就是 $[t-1, 2t-1]$。

    * 这两个端点一次定死、全树通用，`split` 和合并时都要用到。

    * > **易错点**：`MIN_KEY_NUMBER` 与 `MAX_KEY_NUMBER` 是实例常量，不能加 `static`；它们的值依赖具体的 `t`，只能在构造方法里赋值。

```java
public class BTree {

    // 根节点，构造的时候就已经创建出来了
    private Node root;

    // 最小度数，整棵树的节点共用同一个 t
    private int t;

    // 节点中最小的 k 的数目
    private final int MIN_KEY_NUMBER;

    // 节点中最大的 k 的数目
    private final int MAX_KEY_NUMBER;

    public BTree(int t) {
        this.t = t;
        this.MIN_KEY_NUMBER = t - 1;
        this.MAX_KEY_NUMBER = 2 * t - 1;
        // 根节点提前创建好，它的最小度数由 t 决定
        this.root = new Node(t);
    }

    public BTree() {
        // 默认最小度数为 2：一个节点至少要有两个孩子，不能再少了
        this(2);
    }
}
```

---

## contains

* **存在性判断（contains）**

    * > **定义**：判断一个 k 在 B 树中是否存在，返回布尔值。

    * 它对应的就是一次查找，直接复用节点类上已写好的 `get` 方法。

    * 查找到了，节点返回的结果不为 `null`，就返回 `true`；没查找到，`get` 返回 `null`，就返回 `false`。

    * 对 `root` 不用做非空判断，因为构造方法里已经创建了一个不为空的节点对象。

```java
public boolean contains(int key) {
    return this.root.get(key) != null;
}
```

---

## put 新增

### 插入规则

* **新增（put）**

    * > **定义**：把一个新的 k 插入 B 树；节点类没有设计 `values`，所以这一版做了简化，只插 k 不带 value。

    * 当前节点是叶子节点时，按它的大小顺序直接插入即可。

    * B 树的一个节点可以放多个 k，所以先后插入的 1 和 2 会落在同一个节点的 `keys` 数组里。

    * k 的数目不是越多越好，k 多了会增加比较次数、拉低查询效率。

    * 插入后 k 的数目到达 `MAX_KEY_NUMBER`（即 $2t-1$）就必须对该节点做分裂，分裂后每个节点里 k 的数目相应减少。

    * B 树里不允许出现重复的 k，遇到相同的 k 走更新逻辑，不做插入。

* **插入位置（i）**

    * 当前节点的 k 是升序的，所以必须先在当前节点中为新 k 找到一个插入位置 `i`。

    * 查找方式与 `get` 里的比较过程类似：临时变量 `i` 初值为 0，循环条件是 `i` 仍在有效范围内，每次取当前节点第 `i` 个 k 与待插 k 比较，然后 `i++`。

    * 二者相等就是找到了重复的 k，走更新逻辑直接返回。

    * 当前节点的 k 比待插 k 小，则继续 `i++` 向后找。

    * 当前节点的 k 比待插 k 大，则 `break` 退出循环，退出时的 `i` 就是插入位置。

    * 例：当前节点已有 1 和 3，要插入 2，插入位置是索引 1，3 原来在索引 0、要向后移动，2 落到原来 3 的位置。

### 递归线索 i

* **孩子下标的对齐**

    * > **结论**：非叶子节点不能直接插入，必须递归；插入位置是 `i` 就进第 `i` 个孩子，两个 `i` 永远对齐。

    * 例：从根开始插入 4，根里找到插入位置是索引 1（4 比 2 大），根是非叶子，所以递归到第 1 个孩子，也就是装着 3 的那个节点，在它里面再找一次合适的位置完成插入。

* **`i` 的双重身份**

    * > **提示**：`i` 是整个递归过程里唯一的线索——它在当前节点里表示"插入位置"，到了下一层就变成"第几个孩子"，本质上是同一件事。

    * 叶子节点调 `left.insertKey(key, i)` 完成插入；非叶子节点调 `this.doPut(left.children[i], key)` 递归下去。

* **方法的拆分**

    * `put` 只有一个 k 参数，第一版实现不带 value。

    * `put(int key)` 本身只做一件事：调 `this.doPut(this.root, key)`，即从 `root` 开始。

    * 递归方法需要比 `put` 多一个"从哪个节点开始"的参数，所以单独写成 `doPut(Node left, int key)`。

### 插入过程演示

* **叶子阶段**

    * 刚建树时只有一个根节点，根节点里没有任何 k。

    * 插入 1 和 2，两个 k 都落在同一个节点的 `keys` 里，此时还没到上限。

    * 插入 3 时 k 的数目已经到达 `MAX_KEY_NUMBER = 2t - 1 = 3`，触发分裂。

* **下探阶段**

    * 分裂的结果是：小的留在左边，中间的作为父节点，大的分到右边的新节点。

    * 插入 4 时已经不是叶子节点，要向下找到叶子；4 比根的 2 大，于是找到右边装着 3 的叶子节点，它还没满，直接把 4 加进去。

    * 插入 5 之后这个节点又满了，再分裂一次，结果是根节点有 2 和 4 两个 k，比 2 小的 1 在最左边，中间的 3 在中间，比 4 大的 5 在右边。

```text
插入 1、2：两个 k 都落在同一个节点的 keys 里
    [1, 2]

插入 3：到达 MAX_KEY_NUMBER = 2t - 1 = 3，触发分裂
    [1, 2, 3]  ──►    [2]
                   /   \
                 [1]   [3]

插入 4：不是叶子，得往下找 → 4 > 2，走到右边的叶子 [3]，它没满，直接放进去
    [2]
   /   \
 [1]   [3, 4]

插入 5：右叶子又满了，再分裂一次
    [2]
   /   \
 [1]   [3, 4, 5]   ──►    [2, 4]
                        /  |  \
                      [1] [3] [5]
```

### doPut 实现

* **三步骨架**

    * 第一步：在当前节点中找到 k 的插入位置 `i`。

    * 第二步：叶子节点直接插入，非叶子节点递归到第 `i` 个孩子。

    * 第三步：插入后如果 k 的数目到达了 `MAX_KEY_NUMBER`，就做节点分裂。

```java
public void put(int key) {
    this.doPut(this.root, key);
}

private void doPut(Node left, int key) {
    // 第一步：在当前节点中找到 k 的插入位置 i
    int i = 0;
    while (i < left.keyNumber) {
        if (left.keys[i] == key) {
            // B 树里不允许重复的 k，遇到重复走更新逻辑；
            // 当前实现还没有 value，所以这里直接返回
            return;
        } else if (left.keys[i] > key) {
            // 当前节点的 k 已经比待插的 k 大，插入位置就是 i
            break;
        }
        i++;
    }

    // 第二步：叶子节点直接插入，非叶子节点递归到第 i 个孩子
    if (left.leaf) {
        left.insertKey(key, i);
    } else {
        this.doPut(left.children[i], key);
    }

    // 第三步：插入后如果 k 的数目到达了 MAX_KEY_NUMBER，就做节点分裂
}
```

---

## split 分裂

### 触发时机与节点命名

* **触发条件**

    * 每个节点 k 的最大值等于 $2t-1$，插入后 k 的数目一旦到达这个上限就必须分裂。

    * 例如 `t = 2` 时 $2t-1 = 3$，节点里已经有 3 个 k 就到了上限，接下来要分裂。

* **节点称呼**

    * 待分裂的旧节点叫 `left`，因为分裂之后它位于左侧。

    * 新创建出来的节点叫 `right`，因为分裂之后它位于右侧。

    * 它们共同的父节点叫 `parent`。

* **参数 `index`**

    * > **定义**：`index` 表示被分裂节点 `left` 是父节点 `parent` 的第几个孩子。

    * 它是第 0 个孩子就传 0，是第 1 个孩子就传 1。

    * 方法签名是 `split(Node left, Node parent, int index)`：待分裂节点、父节点、孩子序号三者缺一不可。

* **图示约定**

    * 黑色大字是 k，可能一个也可能多个；小字是 k 的下标；红色表示它作为孩子时的孩子下标。

### 分裂口诀

* **三条规律**

    * 第一条：从索引 `t` 开始的后半段，共 `t - 1` 个 k 搬进新节点 `right`。

    * 第二条：索引 `t - 1` 处的中间 k 上浮，插入 `parent` 的第 `index` 位。

    * 第三条：新节点 `right` 挂到 `parent` 的第 `index + 1` 个孩子位，因为它是 `left` 的右邻居。

* **一分为三的核心思想**

    * 大的交给新节点 `right`，中间的上浮给父节点，小的留给自己 `left`。

    * 中位数自己不上阵，这正是搬 `t - 1` 个而不是 `t` 个的原因。

    * > **提示**：第二条为什么成立：被分裂节点是父节点的第 `index` 个孩子，它这一整棵子树里所有的 k 都大于父节点第 `index - 1` 个 k、又小于父节点第 `index` 个 k，所以中间那个 k 插进父节点时不偏不倚正好落在第 `index` 个位置，上浮之后索引不会变。

    * `t = 2` 时被分裂节点若是父节点的第 1 个孩子，中间 k 上浮后的索引就是 1；若是第 2 个孩子，上浮后的索引就是 2。

### t 等于二与 t 等于三的对照

* **t 等于二**

    * `t = 2` 时满节点的 k 是 `[3, 4, 5]`，它是根的第 1 号孩子，所以 `index = 1`。

    * `k[2] = 5` 搬进新节点 `right`，插到 `parent` 的 `index + 1 = 2` 处；`k[1] = 4` 上浮到 `parent` 的 `index = 1` 处；`k[0] = 3` 留在 `left` 自己身上，`keyNumber` 变成 $t-1 = 1$。

    * 新节点必须作为父节点的孩子，且是第 2 号孩子，因为它比 `left` 大、位于 `left` 右侧；分裂后整体仍然满足升序。

* **t 等于三**

    * `t = 3` 时 $2t-1 = 5`，最大 k 数目是 5；这个叶子节点的 k 还没到上限，再加一个 8 就满了。

    * 从索引 `t = 3` 开始的后半段（7 和 8）搬进 `right`，一共 $t-1 = 2$ 个。

    * 索引 $t-1 = 2$ 处的 6 上浮到父节点；剩下的 4 和 5 留给 `left`，`keyNumber` 变成 $t-1 = 2$。

    * 规律与 `t = 2` 完全一致，只是索引和个数跟着 `t` 变。

| 分裂结果 | t = 2 | t = 3 |
| :--- | :--- | :--- |
| 分裂上限 | $2t-1 = 3$ | $2t-1 = 5$ |
| 搬进 right 的 k | 从索引 $t=2$ 起，共 $t-1=1$ 个 | 从索引 $t=3$ 起，共 $t-1=2$ 个 |
| 上浮到 parent 的 k | 索引 $t-1=1$ 处的 4 | 索引 $t-1=2$ 处的 6 |
| 留在 left 的 k | `[3]`，keyNumber 变 $t-1=1$ | `[4,5]`，keyNumber 变 $t-1=2$ |

```text
t = 2，被分裂节点 [3, 4, 5]，index = 1（它是根的第 1 号孩子）

分裂前                              分裂后
        [2, 4]                          [2, 4]
       /   |   \                        /   |   \
     [1]  [3]  [5]     ────►        [1]  [3]  [5]
          被分裂的 [3, 4, 5]
```

```text
t = 3，MAX_KEY_NUMBER = 5，叶子节点已经满了

分裂前                                分裂后
        [1, 2, 3]                           [1, 2, 3, 6]
          /  |  |   \                         /  |  |   |   \
       ...  ... ...  [4,5,6,7,8]  ──►      ... ... ... [4,5] [7,8]
                                             留下        搬去
```

### 基础实现的三步

* **right 的创建**

    * `right` 用带参构造创建，参数直接传整棵树的 `t`，因为全树节点的 `t` 值一致。

    * `right` 的 `leaf` 属性直接照抄 `left`：分裂前节点和新节点有同一个父亲、必然在同一层，所以叶子就是叶子、非叶子就是非叶子。

* **k 的搬迁与计数更新**

    * `System.arraycopy(left.keys, t, right.keys, 0, t - 1)`：原始数组是 `left.keys`，从索引 `t` 开始拷贝，目的数组是 `right.keys`，拷到它的 0 号位置，元素个数是 $t-1$。

    * 搬过去以后 `right.keyNumber` 要变成 $t-1$，因为 `keyNumber` 表示有效 k 的数目。

    * `left` 自己也要减，`left.keyNumber` 同样变成 $t-1$：它把中位数和右半边都分出去了。

    * > **易错点**：`System.arraycopy(src, srcPos, dest, destPos, length)` 的第五个参数是"个数"而不是"结束位置"。

* **上浮与挂载**

    * 中间的 k 就是 `left.keys[t - 1]`，取出来存进 `middle`。

    * 调 `parent.insertKey(middle, index)` 把中间 k 插到父节点的第 `index` 位。

    * 调 `parent.insertChild(right, index + 1)` 把新节点挂到父节点 `index + 1` 的孩子位。

```java
private void split(Node left, Node parent, int index) {
    // 第一条：创建 right，把 left 中较大的一半 k 搬进去
    Node right = new Node(this.t);
    // 新节点和 left 是同一层，leaf 属性照抄
    right.leaf = left.leaf;

    System.arraycopy(left.keys, t, right.keys, 0, t - 1);
    right.keyNumber = t - 1;
    left.keyNumber = t - 1;

    // 第二条：中间那个 k 上浮到父节点
    int middle = left.keys[t - 1];
    parent.insertKey(middle, index);

    // 第三条：right 作为 parent 的第 index + 1 个孩子
    parent.insertChild(right, index + 1);
}
```

### 非叶子节点分裂

* **多出来的那一步**

    * > **结论**：非叶子分裂比叶子分裂只多一件事——把右半边的孩子也搬到 `right`，别的步骤一模一样。

    * 非叶子节点分裂时，除了较大的 k 要搬过去，较大的几个孩子也要搬过去，成为新节点的孩子。

    * 例：`t = 2`，被分裂节点 `keys = [6, 8, 10]`、`children = [5, 7, 9, 11]`，`k[2] = 10` 搬进 `right`（$t-1 = 1$ 个），`k[1] = 8` 上浮到 `parent`。

    * 孩子从索引 $t = 2$ 起搬 $t = 2$ 个：9 和 11 成为 `right` 的 0 号、1 号孩子；剩下 `k = [6]`、`children = [5, 7]` 留在 `left`。

    * 分裂后 `right` 仍挂在 `parent` 的第 2 个孩子位，6/5/7 一组、9/10/11 一组，升序依然成立。

* **拷贝参数的差别**

    * 数组从 `left.keys` 换成 `left.children`，目的数组换成 `right.children`。

    * 起始索引与拷贝 k 时一样，都是从 `t` 开始；目的数组同样从索引 0 开始接。

    * 拷贝个数不同：拷 k 拷 $t-1$ 个，拷孩子要多一个，拷 $t$ 个，因为一个 k 对应两个孩子。

    * > **易错点**：孩子比 k 多一个。拷 k 拷 $t - 1$ 个，拷孩子要拷 $t$ 个——起始索引都是 `t`，只是个数差一。

| 拷贝对象 | 原始数组 | 起始索引 | 目的数组起点 | 拷贝个数 |
| :--- | :--- | :--- | :--- | :--- |
| k | `left.keys` | $t$ | 0 | $t - 1$ |
| 孩子（仅非叶子） | `left.children` | $t$ | 0 | $t$ |

```text
t = 2，被分裂节点 keys = [6, 8, 10]，children = [5, 7, 9, 11]

分裂前                                分裂后
        [4, 8]                               [4, 8]
       /   |    \                            /   |    \
     [5]  [6]   [10]       ────►         [5]  [6]   [10]
        /  |   \                              |   /  |   \
       5  7   9  11                           7   9   11
```

### 根节点分裂

* **为什么更复杂**

    * > **结论**：根节点分裂要多创建一个节点。普通分裂只产生 1 个新节点（`right`），根分裂产生 2 个（`newRoot` + `right`），旧根降级为 `newRoot` 的 0 号孩子。

    * 普通节点的分裂只需要一个 `right`，而根分裂要先把新根造出来，造好之后 `right` 以下的步骤就跟以前完全一样。

* **数据的分配**

    * 例：`t = 3`，根节点 `[1, 2, 3, 4, 5]` 已经满了（$5 = 2t - 1$）。

    * 大的 4 和 5 移到下面的新节点 `right`，作为 `newRoot` 的第 1 号孩子。

    * 中间的 3 成为 `newRoot` 自己的 k。

    * 剩下的 1 和 2 留在旧根节点，作为 `newRoot` 的第 0 号孩子。

* **判断与赋值**

    * 判断待分裂节点是不是根有两种办法：检查它是否等于 `this.root`；或者检查 `parent` 是否为 `null`，没有父亲就说明是根。

    * 实现上提前判断 `parent == null`，为真就创建 `newRoot`，用 `newRoot.insertChild(left, 0)` 把旧根挂成它的 0 号孩子。

    * `newRoot` 的 `t` 与其他节点一样传当前树的 `t`；它有孩子，所以 `newRoot.leaf = false`。

    * 之后把 `this.root = newRoot`，同时 `parent = newRoot`，因为后续要用到 `parent`。

```text
t = 3，根节点已经满了（5 = 2t - 1）

分裂前                                分裂后
     [1, 2, 3, 4, 5]     ────►          [3]
                                          /     \
                                     [1, 2]   [4, 5]
```

### 完整实现

* **三个关键数字**

    * > **结论**：`t` 决定"右半边从哪儿开始搬"，`t - 1` 决定"搬几个 k"和"中间 k 在哪儿"，`index` 贯穿"中间 k 上浮到哪儿"和"新节点挂到哪儿"。

    * 根节点的特殊处理放在最前面：先造新根并更新 `parent`，然后对原来的根继续走普通分裂。

    * 普通分裂的工作是创建 `right`、搬较大的 k（非叶子还要搬较大的孩子）、更新新旧两个节点的 `keyNumber`、上浮中间 k、把 `right` 挂到 `parent`。

```java
private void split(Node left, Node parent, int index) {
    // ---------- 特殊情况：分裂的是根节点 ----------
    // parent 为 null，说明没有父亲，我们分裂的就是根节点
    if (parent == null) {
        Node newRoot = new Node(this.t);
        newRoot.leaf = false;
        // 旧的根节点 left，成为新根节点的 0 号孩子
        newRoot.insertChild(left, 0);
        this.root = newRoot;
        // 接下来要用到的 parent 就是这个新根
        parent = newRoot;
    }

    // ---------- 第一条：右半边较大的 k 搬进新节点 right ----------
    Node right = new Node(this.t);
    // 新节点和 left 同一层，leaf 属性照抄
    right.leaf = left.leaf;

    System.arraycopy(left.keys, t, right.keys, 0, t - 1);
    right.keyNumber = t - 1;
    left.keyNumber = t - 1;

    // 非叶子节点还要把右半边的孩子搬过去，比 k 多一个
    if (!left.leaf) {
        System.arraycopy(left.children, t, right.children, 0, t);
    }

    // ---------- 第二条：中间那个 k 上浮到父节点 ----------
    int middle = left.keys[t - 1];
    parent.insertKey(middle, index);

    // ---------- 第三条：right 作为 parent 的第 index + 1 个孩子 ----------
    parent.insertChild(right, index + 1);
}
```