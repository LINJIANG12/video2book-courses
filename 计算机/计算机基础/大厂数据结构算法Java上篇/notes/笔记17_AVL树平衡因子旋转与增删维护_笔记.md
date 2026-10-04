# AVL 树：平衡因子、旋转与增删维护

## 高度与平衡因子

### 高度的递推式

* **节点高度 h(node)**

    * > **定义**：一个节点的高度等于它左右孩子高度中较大的那个再加一。

    * 公式：

$$
h(node)=\max(h(node.left),\,h(node.right))+1
$$

    * 刚创建出来的新节点，高度就是 1。

* **平衡因子（balance factor，bf）**

    * > **定义**：平衡因子等于当前节点左子树高度减去右子树高度。

    * 公式：

$$
bf= h(node.left)-h(node.right)
$$

### 平衡与失衡的判定

* **平衡的取值区间**

    * > **定义**：平衡因子落在 $-1,0,1$ 这三个值上就是平衡的。

    * 大于 1 或者小于 $-1$ 就是失衡。

    * $bf>1$ 表示左边更高，$bf<-1$ 表示右边更高。

* **查询操作**

    * 查询不会改变树的平衡，与普通二叉搜索树的查询完全一样，不必重复实现。

---

## 右旋与左旋

### 旋转方法的接口约定

* **右旋（rightRotate）与左旋（leftRotate）**

    * > **定义**：`rightRotate(Node node)` 的参数 `node` 是要旋转的那个节点，实际调用时传进来的基本就是失衡节点。

    * 返回值是旋转之后产生的新根节点。

    * 返回新根而不是 void，是因为旋转的本质就是在以这个节点为根的子树里把根旋转下去，原根往下走了，谁成了新根谁就得返回给调用方。

    * `leftRotate` 的参数和返回值完全是同一套说法。

* **红黄绿三色标记**

    * 红色是传进来要旋转的节点，黄色是将来作为新根的节点，绿色是要换爹的节点。

    * 不带颜色标记的普通节点，以及换爹的绿色节点，旋转前后高度都不会变。

    * 红黄绿只是讲课时好指着说，命名怎么好理解怎么来，不必与示例一致。

### 右旋 rightRotate

* **代码骨架**

    ```java
    public Node rightRotate(Node node) {
        Node yellow = node.left;   // 先拿黄色
        Node green = yellow.right;  // 再从黄色身上拿绿色

        yellow.right = node;       // 上位
        node.left = green;         // 换爹

        return yellow;
    }
    ```

* **绿色换爹的依据**

    * 绿色的 G 在黄色的右侧，所以 G 比黄色大。

    * 绿色连同黄色整个都在红色的左侧，所以 G 一定比红色小。

    * 直接让 G 当红色的左孩子：原来的爹是黄色，现在新爹是红色，身份同时从右孩子变成左孩子。

* **语句顺序**

    * 取节点的顺序不能换：必须先 `node.left` 拿到黄色，再 `yellow.right` 拿到绿色。

    * 上位 `yellow.right = node` 必须在换爹 `node.left = green` 之后，绝对不能前移。

    * > **易错点**：先执行上位的话，黄色的右孩子当场被红色顶掉，下一行再取 `yellow.right` 拿到的已经是红色，绿色早就没了。

    * 换爹这一行可以前移到最前面，此时 green 变量可以省掉，三行就搞定：

    ```java
    public Node rightRotate(Node node) {
        Node yellow = node.left;
        node.left = yellow.right;   // 换爹
        yellow.right = node;        // 上位
        return yellow;
    }
    ```

    * > **提示**：一开始还是老老实实多写几行，先把逻辑写得最清晰，后面再做代码的调整和优化。

* **空指针判断**

    * 不需要判断 yellow 是不是 null：既然要右旋就说明左边高，左边都能高出这个高度，它的左孩子不可能为空。

    * green 为 null 也没关系，只是拿到了一个 null，再把 null 赋值给将来红色的左孩子。

    * > **易错点**：不要画蛇添足加 `if (yellow != null)` 和 `if (green != null)`。

### 左旋 leftRotate

* **代码骨架**

    ```java
    public Node leftRotate(Node node) {
        Node yellow = node.right;  // 这回是通过右孩子拿到黄色
        Node green = yellow.left;   // 这回绿色是黄色的左孩子

        yellow.left = node;         // 上位：黄色的左孩子变成红色
        node.right = green;         // 换爹：绿色改挂到红色的右边

        return yellow;
    }
    ```

* **与右旋的差异**

    * 左旋是通过右孩子拿到黄色，黄色这边是左孩子，绿色取 `yellow.left`。

    * 绿色原来是黄色的左孩子，旋转之后爹变成红色，身份变成红色的右孩子。

    * 最后返回的新根依然是黄色。

---

## 组合旋转

### 左右旋 leftRightRotate

* **适用与步骤**

    * 解决 LR 这种折线型失衡。

    * 先对失衡节点的左子树左旋，再对根节点右旋。

* **代码**

    ```java
    public Node leftRightRotate(Node node) {
        node.left = leftRotate(node.left);   // 左子树左旋,旋转上来的黄色要成为根的左孩子
        return rightRotate(node);            // 根节点右旋
    }
    ```

* **必须接返回值的原因**

    * 第一步的 `node.left = ...` 不能省：原来根节点的左孩子是红色的，旋转之后黄色上来要重新赋给 `node.left`，父子关系得改对。

    * 第二步右旋完直接 return，返回的就是旋转上来的黄色。

    * 左边怎么上位、怎么换爹，右旋代码里全都有，一个字的重复代码都不用再写。

### 右左旋 rightLeftRotate

* **适用与步骤**

    * 解决 RL 这种折线型失衡，与左右旋完全镜像。

    * 先旋转失衡节点的右子树，再左旋失衡节点。

* **代码**

    ```java
    public Node rightLeftRotate(Node node) {
        node.right = rightRotate(node.right);  // 右子树右旋
        return leftRotate(node);               // 失衡节点左旋
    }
    ```

* **返回值的处理**

    * 第 2 步的 `node.right = ...` 同样必须接返回值：失衡节点的右孩子原来是红色的，右旋之后要变成黄色的。

    * 第 3 步左旋之后，新的根节点直接 return。

---

## 旋转后的高度维护

* **只有红色和黄色需要更新高度**

    * > **结论**：只有红色和黄色这两种角色的节点需要更新高度，其余节点旋转前后高度都不会变。

    * 绿色节点是实证：旋转之前高度是 1，旋转之后还是 1。

    * 黄色节点的高度是会变的：左右旋例子中黄色旋转之前左右孩子最大值加一等于 2，旋转上去之后变成 3。

    * 红色节点高度实打实地变了：旋转前 $h(8)=\max(3,1)+1=4$，旋转后 $h(8)=\max(2,1)+1=2$。

* **更新顺序必须先红后黄**

    * 旋转之后红色在下面、黄色在上面，必须先把下面节点的高度算对，再拿它去算上面的高度。

    * > **易错点**：先更新黄色再更新红色不行，下面还没算对就取最大值，得不到正确结果。

    * 左旋代码里同样是先红后黄。

    * 左右旋和右左旋不需要额外更新高度，这个操作已经被调用的 `leftRotate` 和 `rightRotate` 包含了。

---

## balance

### 分支骨架

* **职责**

    * > **定义**：传进来一个节点，检查它是不是失衡了；失衡就做相应的旋转让它重新平衡，返回平衡后的新根节点；没失衡就原封不动地返回。

    * 这是相当重要的方法，后面 `put` 会直接调它。

* **两个前置判断**

    * 节点是 null 就没必要往下走任何失衡判断，直接返回 null，特殊处理掉。

    * 判断失衡的工具就是平衡因子，直接调 `balanceFactor` 求当前节点的平衡因子。

* **三个分流**

    * `$bf>1$` 表示左边更高，对应 LL 或 LR，走左旋再右旋。

    * $-1\le bf\le 1$ 就是平衡的，不需要任何旋转，原样返回当前节点。

    * `$bf<-1$` 表示右边更高，对应 RL 或 RR，走右旋再左旋。

    ```text
                      balance(node)
                           │
                    node == ○ ?
                    ／         ＼
                 是                否
                  │                  │
            return ○        bf = balanceFactor(node)
                                     │
                ┌────────────────────┼────────────────────┐
             bf > 1           -1 ≤ bf ≤ 1           bf < -1
                │                   │                   │
           左子树更高                平衡              右子树更高
                │                   │                   │
       LL 或 LR,左旋再右旋    原样 return node   RL 或 RR,右旋再左旋
    ```

### 四种失衡与对应旋转

| 失衡类型 | 形状描述 | 判定条件 | 对应旋转 |
| :--- | :--- | :--- | :--- |
| LL 左左 | 左边更高，左子树也是左边更高 | `bf > 1 && balanceFactor(node.left) > 0` | `rightRotate(node)` |
| LR 左右 | 左边更高，左子树却是右边更高 | `bf > 1 && balanceFactor(node.left) < 0` | `leftRightRotate(node)` |
| RL 右左 | 右边更高，右子树却是左边更高 | `bf < -1 && balanceFactor(node.right) > 0` | `rightLeftRotate(node)` |
| RR 右右 | 右边更高，右子树也是右边更高 | `bf < -1 && balanceFactor(node.right) < 0` | `leftRotate(node)` |

* **细分的依据**

    * `bf > 1` 或 `bf < -1` 只定出哪边更高，要分出 LL/LR 还是 RL/RR 必须再看对应子树的平衡因子。

    * 左子树的平衡因子大于 0 是 LL，小于 0 是 LR。

    * 右子树的平衡因子大于 0 是 RL，小于 0 是 RR。

* **写法建议**

    * > **提示**：这张表就是 `balance` 的全部业务逻辑。

    * 把四个条件按「左高两条、右高两条」分成两两一组，每个条件都写成两个 `balanceFactor` 判断的与运算。

    * 连 `else if` 都不用写，靠 `return` 直接短路，谁先命中算谁的。

### 删除特有的零平衡因子

* **补两个「等于」的情况**

    * 删掉节点 8 后，失衡节点 6 的左子树高度 2、右子树高度 0，高度差超过 1；但左子树的左右高度差等于零，原判定只判了大于零和小于零，没有一个能匹配上。

    * > **易错点**：左子树的平衡因子等于 0，也应该按 LL 处理做一次右旋，因为左子树这边是平衡的，要把它视为左左。

    * > **易错点**：对称地，右子树的平衡因子等于 0，也应该按 RR 处理做一次左旋。

    * 这两个「等于」的条件必须加进去，新增的时候不会出现这个问题，只有删除才会。

* **完整实现**

    ```java
    public Node balance(Node node) {
        if (node == null) {
            return null;
        }
        int bf = balanceFactor(node);

        // bf > 1:左边更高。左子树的 bf >= 0 归 LL(删除时 bf == 0 也算 LL),< 0 归 LR
        if (bf > 1 && balanceFactor(node.left) >= 0) {
            return rightRotate(node);
        }
        if (bf > 1 && balanceFactor(node.left) < 0) {
            return leftRightRotate(node);
        }

        // bf < -1:右边更高。右子树的 bf > 0 归 RL,<= 0 归 RR(删除时 bf == 0 也算 RR)
        if (bf < -1 && balanceFactor(node.right) > 0) {
            return rightLeftRotate(node);
        }
        if (bf < -1 && balanceFactor(node.right) <= 0) {
            return leftRotate(node);
        }

        return node;
    }
    ```

    * `>= 0` 和 `<= 0` 里的那个等号，就是专门为删除时那两种特殊情形留的口子。

---

## put：新增与更新

### 方法分工

* **put 与 doPut**

    * `put` 接受 `key` 和 `value` 两个参数，查找起点是根节点。

    * `doPut` 是专门做递归的方法，返回值类型就是节点类型，除 `key` 和 `value` 外还要多传一个「从哪个节点开始去找空位」的节点参数。

* **根节点的赋值**

    ```java
    private Node root;   // 前面的代码里还没定义根节点,这里补上

    public void put(int key, Object value) {
        this.root = doPut(this.root, key, value);
    }
    ```

    * 第一次调 `put` 时根节点是空的，`doPut` 会在第一个 `if` 上直接成立，返回一个新节点。

    * > **易错点**：这个创建好的新节点必须赋值给根节点，否则这棵树永远长不出来。

### 递归的三种情况

* **情况一：找到空位了**

    * 判断有没有空位看传进来的节点已经是 `null` 了，说明这儿有空位，那就创建一个新的节点对象返回。

    * 刚创建出来的节点，高度就是 1。

* **情况二：key 已经存在**

    * 判断依据是当前的 `key` 等于当前节点的 `key`，直接走更新流程，把 `value` 更新成传进来的新值然后 return，后面的代码根本不用执行。

    * > **易错点**：更新完全不需要考虑改高度、也不需要重新平衡，因为更新根本不会影响树的高度，树不会失衡。

* **情况三：继续查找**

    * `key` 小于当前节点的 `key` 就沿左子树递归向左找，`key` 大于当前节点的 `key` 就沿右子树递归向右找。

    * 小于和等于都写了，剩下的直接 `else` 肯定就是往右走。

* **方法体**

    ```java
    private Node doPut(Node node, int key, Object value) {
        // 情况一:找到空位了,创建新节点返回(新节点高度为 1)
        if (node == null) {
            return new Node(key, value);
        }

        // 情况二:key 已经存在,走更新的逻辑。不用改高度,也不用重新平衡
        if (key == node.key) {
            node.value = value;
            return node;
        }

        // 情况三:继续查找
        if (key < node.key) {
            node.left = doPut(node.left, key, value);   // 向左找,并建立父子关系
        } else {
            node.right = doPut(node.right, key, value);  // 向右找,并建立父子关系
        }

        // 下面这两行是 AVL 相对于二叉搜索树新加的
        updateHeight(node);
        return balance(node);
    }
    ```

### 回溯路上逐层更新

* **两行新代码的位置**

    * 只要下一层递归找到空位并返回新节点，就把它接到当前节点上，把父子关系建起来。

    * 以前的二叉搜索树代码写到这里就结束了，现在加了节点以后当前节点的高度需要变，所以调 `updateHeight`。

    * 当前节点高度一更新，这个节点就有可能失衡，失衡就调 `balance`；没失衡的话 `balance` 最终返回的就是节点本身。

    * > **结论**：高度是在递归回去的过程中一点一点更新的，这也是为什么 `updateHeight` 必须放在递归调用的后面，而不是前面。

* **插入 9、5、3 的过程**

    * 插 9：树是空的，第一个 `if` 成立，创建出节点 9 返回，然后赋值给根节点。

    * 插 5：5 小于 9 所以向左找，9 的左孩子是 null 创建出 5 建立父子关系；`updateHeight(9)` 把 9 更新成高度 2，`balance(9)` 因左右高度差 $1-0=1$ 未失衡原样返回。

    * 插 3：找到 5，发现 5 左边有空位创建 3 并建立父子关系；`updateHeight(5)` 把 5 更新成高度 2，`balance(5)` 未失衡。

    * 5 返回到节点 9 那一层，`updateHeight(9)` 把 9 更新成高度 3，`balance(9)` 检出左右孩子高度差 $2-0=2$ 已超过 1。

    * 该失衡左边更高且左孩子也是左边更高，属于 LL，对 9 做一次右旋，`balance` 返回 5，根节点从 9 替换成 5。

    ```text
       put(9)             put(5)            put(3)
          9                 9                  5
                          /                 / \
                         5                 3   9

    插入 3 后的最终形态

            5
           / \
          3   9
    ```

---

## remove：删除

### 方法架子

* **接口与命名**

    * 删除方法返回 `void`，删完就删完，不把被删除的值再抛回来。

    * 方法名从 `delete` 改成 `remove`，是为了跟 Java 中 `Map` 的删除保持一致。

    * > **定义**：`doRemove` 的返回值代表删剩下的那个节点，返回值类型就是节点类型。

    ```java
    public void remove(T k) {
        root = doRemove(root, k);
    }
    ```

* **为什么要更新根节点**

    * 删除的过程中树会重新平衡，平衡之后根节点可能就变了。

    * > **易错点**：删完之后必须拿返回值更新一下根节点，这个动作省不掉。

* **递归内部五步**

    ```text
    private Node<T> doRemove(Node<T> node, T k)

      ① node == null                     → 没必要删了，直接返回
      ② 没找到 k                         → 看是向左走还是向右走，继续递归
      ③ 找到了 k                         → 四种子情况，分派处理
      ④ 更新"删剩下的节点"的高度
      ⑤ 检查它失衡 → 失衡就在 balance 内部旋转，返回平衡后的新根
    ```

    * > **易错点**：第 ③ 步只是「找到」，找到之后还不能急着往上返回，因为第 ④、⑤ 两步是对删剩下的那个节点做的；找到了就直接 `return`，高度没人更新、失衡也没人修。

### 空节点与没找到

* **节点为 null**

    * 没必要执行后续的任何操作了，直接返回空。

* **没找到 k 的两条递归分支**

    * k 比当前节点小就往左，左孩子那一路更新 `node.left`；k 比当前节点大就往右，右孩子那一路更新 `node.right`。

    ```java
    if (node.k.compareTo(k) < 0) {
        // k 比当前节点小，往左
        node.left = doRemove(node.left, k);
    } else if (node.k.compareTo(k) > 0) {
        // k 比当前节点大，往右，逻辑跟上面完全类似
        node.right = doRemove(node.right, k);
    }
    ```

    * > **提示**：这里的写法是「传参进去 + 接返回值出来」成对的；只传进去不接返回值等于白递归，子树的根换了上面还指着旧节点，树就断了。

### 四种子情况

| 子情况 | 判断条件 | 删剩下的是谁 | 能不能立刻 return |
| :--- | :--- | :--- | :--- |
| 一 | 左孩子为空 **并且** 右孩子为空 | 空 | 能，直接 `return null` |
| 二 | 左孩子为空（只有右孩子） | 右孩子 | 不能 |
| 三 | 右孩子为空（只有左孩子） | 左孩子 | 不能 |
| 四 | 走到 `else`（左右孩子都有） | 后继节点 | 不能，后面还要接子树 |

* **情况一**

    * 没有孩子，当前节点删了它就什么都不剩，返回空就可以了。

* **情况二与情况三**

    * 删剩下的是一棵还活着的子树，它的高度可能已经变了，也可能自己就失衡了，所以不要立刻返回。

    * 把删剩下的这个孩子先暂存进 `node` 这个变量（反正 node 删了没用了），让程序继续向下执行第 ④、⑤ 步。

    ```java
    } else if (node.left == null) {
        node = node.right;      // 只有右孩子：先暂存进 node，不立刻 return
    } else if (node.right == null) {
        node = node.left;       // 只有左孩子：同理，也不立刻 return
    }
    ```

    * > **注意**：唯一可以立刻 `return` 的是情况一，因为删剩下的是空，高度压根不会变。

### 两个孩子与后继节点

* **找后继节点**

    * 从待删除节点的右子树开始向左找，向左找到头，那就是它的后继节点。

    ```java
    Node<T> s = node.right;        // s 代表后继节点，初始值是待删除节点的右子树
    while (s.left != null) {        // 只要 left 不为空，就表示向左还没走到头
        s = s.left;
    }
    ```

* **处理后事**

    * 找到后继节点后不能立刻用它代替待删除节点，因为它可能还有自己的孩子。

    * 处理后事的方式是再递归地调 `doRemove`，从 `node.right` 里把 `s.k` 删除，这次递归的返回值就是后继节点删完之后剩下来的内容。

    * 剩下来的内容作为后继节点的右子树，因为后继节点最终要取代掉原来的那个节点。

    ```java
    node.right = doRemove(node.right, s.k);
    s.right = node.right;
    ```

    * 例：递归删除 10 之后，9 的右子树变成 12 带 11、13，11 顶到原来 10 的位置成为 12 的左孩子，12 的右孩子 13 不变。

* **左右子树的赋值顺序**

    * 后继节点的右子树先赋，它的后继节点的左子树应该是被删除节点的左子树。

    * > **易错点**：顺序必须是先给右子树赋值，再给左子树赋值，不能颠倒。

    ```java
    s.left = node.left;             // 顺序不能颠倒：先右后左
    node = s;                       // 用 s 代替掉之前被删除的 node，删剩下的是后继节点
    ```

    * 所有操作完成后，返回前还要调整后继节点的高度，以及对后继节点做失衡检查和旋转。

* **删 9 的完整回放**

    * 进入第一个 `else` 找到 9，因为有两个孩子进入第二个 `else` 走两个孩子分支。

    * 从右子树沿左孩子走到头找到后继节点 `s`，对应到 10。

    * 先从 9 的右子树里把 10 递归删掉，11 顶上来到 12 的左边，后继节点的后事处理好了。

    * 后继节点调整高度后，左子树比右子树高、高度差已超过 1，属于左左型，做一次右旋即可。

    * 这个例子中 x = 10 是红色下来，y = 5 是黄色上去，u = 6 是绿色换爹，z = 12。

    ```text
    左左型右旋（L-L Rotate）

          x  ← 失衡点，左高右低        y  ← 新根
         / \                          / \
        y   z          右旋          t   x
       / \                                    \
      t   u                                  u   z
    ```

---

## 红黑树起步

### 与 AVL 树的区别

* **判断平衡的依据**

    * > **定义**：红黑树也是一种自平衡的二叉搜索树，与 AVL 树主要的区别就是判断平衡的依据不一样。

    * AVL 树看一个节点左右子树的高度差是不是超过了一，超过了一表示不平衡。

    * 红黑树判断一棵子树是不是平衡，有自己的一套规则。

* **性能取向**

    * 优势主要体现在插入和删除时：红黑树的旋转次数会少一些，性能上稍微比 AVL 要高一些。

* **节点对象新增的两个属性**

    * `parent` 代表父节点，因为新增和删除时经常用到，干脆做成节点属性；代价是编码更复杂，新增和修改时得维护它指向正确的对象。

    * `color` 表示颜色，初始值赋为 `RED`，即刚创建出来的节点都认为是红色的。

    * 颜色只有两种，用枚举类型 `Color` 表示，取值为 `RED` 和 `BLACK`。

### 五条特性

| 特性 | 内容 |
| :--- | :--- |
| 一 | 节点颜色只有两种：黑色、红色，所以节点对象里要加一个 `color` 属性表示颜色 |
| 二 | 如果出现了 `null`，一律把它看成是黑色 |
| 三 | 红色的节点不能相邻，这条非常重要 |
| 四 | 根节点必须是黑色 |
| 五 | 从根到任意一个叶子节点，所经过路径中的黑色节点数目是一样的 |

* **逐条要点**

    * 特性三意味着红色节点都被黑色隔开了，层与层之间没有相邻的红节点；一旦相邻就意味着不平衡。

    * 特性四意味着根节点如果在某个过程中变成红色了，也得重新调整成黑色才算平衡。

    * > **结论**：满足这五条就表示是一个平衡的红黑树，实际判断时最关键的就是第三和第五条。

* **null 参与的判定**

    * 当发现一个叶子节点没有自己的兄弟时，就要把这个 `null` 当成黑色一起数进来。

    * > **提示**：红色叶子可以单独一个，它有没有孩子都可以；黑色的、又没有兄弟的叶子肯定是不平衡的——黑色叶子必须成对出现。

    * 例：某棵以 6 为根、1 和 8 为叶的树，补上 null 后到每个叶子路径中都是三个黑色，是平衡的；另一棵补上 null 后到 2 的右孩子只有两个黑色，不平衡。

    * 判断诀窍是「盯着黑色看」，看黑色在两边是不是平衡的。

### 节点类的三个找人的方法

* **isLeftChild 判断是否左孩子**

    * 先得判断节点不为空且父节点不为空，再判断父节点的 `left` 属性与当前节点是不是指向同一个对象。

    ```java
    private boolean isLeftChild(Node<T> node) {
        return node != null
            && node.parent != null
            && node.parent.left == node;   // 父子指向的是同一个对象
    }
    ```

* **uncle 找叔叔**

    * > **定义**：叔叔就是跟父亲平辈儿的节点。

    * 父亲是爷爷的左孩子，叔叔就是爷爷的右孩子；反过来则是爷爷的左孩子。

    * 根节点以及有父亲但没有爷爷的节点都没有叔叔，直接返回 null。

    ```java
    private Node<T> uncle(Node<T> node) {
        if (node == null || node.parent == null || node.parent.parent == null) {
            return null;                    // 根节点、以及没有爷爷的节点，都没有叔叔
        }
        return isLeftChild(node.parent)
            ? node.parent.parent.right      // 父亲是爷爷的左孩子 → 叔叔是爷爷的右孩子
            : node.parent.parent.left;      // 父亲是爷爷的右孩子 → 叔叔是爷爷的左孩子
    }
    ```

* **sibling 找兄弟**

    * 节点为 null 或没有父节点时返回 null，否则是父亲的左孩子就取父亲的右孩子，反之取父亲的左孩子。

    ```java
    private Node<T> sibling(Node<T> node) {
        if (node == null || node.parent == null) {
            return null;                    // 连父节点都没有，肯定没有兄弟
        }
        return isLeftChild(node) ? node.parent.right : node.parent.left;
    }
    ```

* **isRed 与 isBlack**

    * 这两个方法要定义在树内部而不是 `Node` 内部，因为有可能处理到 null，不能直接去点属性。

    * `isBlack` 写成 `!isRed(node)` 是一种写法，也可以展开成「`node` 为 null」或「`color` 等于 BLACK」两种情况。

    ```java
    private boolean isRed(Node<T> node) {
        return node != null && node.color == Color.RED;
    }

    private boolean isBlack(Node<T> node) {
        return !isRed(node);    // 不是红色就肯定是黑色，null 也算黑
    }
    ```

### 旋转代码的两处不同

* **与 AVL 右旋左旋的差别**

    * 旋转的核心逻辑大部分是一样的，不一样的主要有两点。

    * 节点对象多了一个 `parent` 属性，旋转之后要让它的 `parent` 指向正确的值，处理起来更麻烦。

    * 旋转代码没有返回值了，旋转后新根的父子关系直接在旋转方法内部建立好。

    * > **结论**：写红黑树的旋转，rotate 里面自己把 parent 链路修完整，外面就不用再操心重新挂接。

* **三个节点的 parent 维护**

| 节点 | 旋转前的 parent | 旋转后应该指向 |
| :--- | :--- | :--- |
| green | yellow | pink |
| yellow | pink | parent（pink 原来的爹） |
| pink | parent | yellow |

* 绿色旋转前的爹是黄色，旋转后爹指向粉色；黄色旋转前是粉色的孩子，旋转后要变成原来粉色的爹；粉色旋转后爹指向黄色。

* > **易错点**：绿色节点一定要加非空判断，它有可能就是 null，不判断就去改 `parent` 属性就会空指针。

* **新根与上层的父子关系**

    * 先通过粉色节点的 `parent` 把更上层的节点抓在手里，因为等下 `pink.parent` 就被改写成 yellow 了。

    * `parent` 为 null 说明旋转节点本身就是根，此时给它的 `left`、`right` 赋值会抛空指针，要单独处理 `root = yellow`。

    * > **提示**：只有 `parent` 不为 null 时才需要建立它与黄色的父子关系，所以这两个分支要写在 `if` 和 `else if` 里。

    * 判断黄色该当左孩子还是右孩子，看 `parent` 的左孩子是不是原来的粉色；成立就挂左边，否则挂右边。

    ```java
    private void rightRotate(Node<T> pink) {
        Node<T> yellow = pink.left;
        Node<T> green = (yellow != null) ? yellow.right : null;
        // 先把更上层的节点抓在手里，等下 pink.parent 就被改写成 yellow 了
        Node<T> parent = pink.parent;

        yellow.right = pink;
        pink.left = green;

        if (green != null) {
            green.parent = pink;
        }
        yellow.parent = parent;
        pink.parent = yellow;

        if (parent == null) {
            root = yellow;                 // 粉色本身就是根，黄色顶上成为新根
        } else if (parent.left == pink) {
            parent.left = yellow;
        } else {
            parent.right = yellow;
        }
    }
    ```

    * 粉黄绿三色标记法：pink 是要右旋的节点，`yellow = pink.left` 是将来的新根，`green = yellow.right` 是要换爹的节点。

    * 基本旋转部分和 AVL 完全一样：黄色的右孩子变成粉色，粉色的左孩子变成绿色。