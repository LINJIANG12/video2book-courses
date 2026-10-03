# ArrayList 集合

## 集合与数组的差别

* **数组（Array）**

    * 数组是一个容器，可以存储同种数据类型的多个值。
    * 数组一旦定义完毕，长度不可变；再想添加数据只能新建一个更长的数组，把原来的数据搬过去再追加，代码非常麻烦。
    * `int[] arr = new int[3];` 从声明上就能看出元素类型是 int。

* **集合（Collection）**

    * > **定义**：集合就是一个长度可变的容器。
    * 添加元素时自动扩容，删除元素时自动缩减，可长可短、伸缩灵活。
    * 刚创建时长度是 0，每添加一个元素长度加一，依次变成 1、2、3、4、5。
    * 删除一个元素后长度自动缩减，比如从 5 变成 4。
    * 以后要定义容器存储多个数据时直接用集合，不必再考虑容器长度问题。

* Java 中集合种类很多，ArrayList、LinkedList、HashSet 等差别后续单独展开，其中 ArrayList 用得最多。

## 集合的两个特点

* 长度可变。
* **只能存储引用数据类型（对象），不能存储基本数据类型。**
    * > **易错点**：往集合里添加整数时不能写 `int`，必须写对应的包装类 `Integer`。

## 泛型（Generics）

* **不写泛型的 ArrayList**

    * `ArrayList list = new ArrayList();` 从这行代码判断不出集合能存什么类型的数据。
    * 因为没有做限定，集合里可以添加任意类型：字符串 `"abc"`、Student、Teacher、Cat、Dog 全都收得住。
    * 自己写的类没有重写打印方法，直接打印时输出的是内存地址。
    * 取数据时 `list.get(0)` 不知道该用什么类型接收，只能用 `Object`——Object 是 Java 里所有类的祖类，任意对象都能赋给 Object 变量，这就是多态。
    * 多态的弊端是无法使用子类的特有行为，想用就必须向下转型。
    * > **易错点**：数据是别人添加的、类型未知时，每取一个数据都要写一长串 `o instanceof String` → 强转、`else if` → 强转，代码量比不用集合还大。

* **泛型的作用**

    * > **定义**：泛型的作用就是限定集合当中存储的数据类型。
    * 书写格式是一个尖括号，把数据类型直接写在尖括号中。
    * 类名后面带尖括号 E（如 `ArrayList<E>`），就是在提醒创建对象时必须写泛型。
    * 变量名左边和 `new ArrayList` 后面的尖括号都要写上同一个类型。

* **JDK7 起的菱形语法**

    * JDK7 之后后面的泛型可以省略不写，但尖括号必须保留，里面的内容默认与前面保持一致。
    * 最终写法：`ArrayList<String> list = new ArrayList<>();`

* **泛型带来的两层好处**

    * 添加时做类型校验：泛型写 `String` 后再 `add` 一个 Student，代码直接编译报错，提示需要 String 实际给了 Student。
    * 获取时不用强转：`get` 左边自动生成 String 类型，可直接调用 String 的特有方法。

```java
// 泛型写 String 之后
ArrayList<String> list = new ArrayList<>();
list.add("abc");
list.add(new Student());   // 编译报错：需要 String，实际给了 Student
```

## 创建 ArrayList 对象

* 用空参构造先创建一个长度为 0、没有任何元素的集合，再通过里面的方法操作数据。
* 泛型决定能存什么类型：`ArrayList<Student> list = new ArrayList<>();` 中要存学生对象，泛型就写 `Student`。
* > **注意**：写泛型时注意包名，用的是哪个包下的 `Student` 就写哪个。
* 往集合里添加的是学生对象时，泛型就不能写 `String` 或 `Integer`。

## 增删改查的方法总览

| 类别 | 方法 | 说明 |
| :--- | :--- | :--- |
| 增 | `add(E e)` | 把元素添加到末尾 |
| 增 | `add(int index, E e)` | 把元素添加到指定索引 |
| 删 | `remove(Object o)` | 按元素内容删除，返回是否成功 |
| 删 | `remove(int index)` | 按索引删除，返回被删除的元素 |
| 改 | `set(int index, E e)` | 修改指定索引，返回被替换的元素 |
| 查 | `get(int index)` | 获取指定索引上的元素 |
| 查 | `size()` | 获取集合的长度 |

## 单参数 add

* > **定义**：`list.add(E e)` 把当前元素添加到末尾，返回 boolean 表示是否添加成功。
* 源码中它是**一直返回 `true`** 的，添加什么数据都必定成功，永远不会返回 `false`。
* 因此实际使用时直接忽略返回值即可。

```java
ArrayList<String> list = new ArrayList<>();

boolean res1 = list.add("AAA");
boolean res2 = list.add("BBB");
boolean res3 = list.add("CCC");

System.out.println(res1);   // true
System.out.println(res2);   // true
System.out.println(res3);   // true
```

* **为什么已经有 void 了还要返回 boolean**

    * Java 里集合不止 ArrayList 一个。HashSet 的元素必须唯一：第一次 `add("AAA")` 返回 `true`，第二次再添加同一个 AAA 就返回 `false`。
    * 所以 ArrayList 的 `add` 保留返回值不是为了服务自己，而是**跟其他集合保持统一**。
    * 统一的做法来自抽象方法：把 `eat` 这类共性方法抽到父类里声明为抽象方法，规则里包含返回值、方法名、形参，强制子类按同一格式重写。调用时直接看父类即可，不必一个子类一个子类地翻。
    * 集合体系里的 `add` 就是被定义在**接口**中作为抽象方法的，子类 ArrayList 再按这个规则重写。

## 双参数 add

* > **定义**：`list.add(int index, E e)` 把元素插入指定索引位置，第一个形参是索引，第二个是要添加的数据。
* 它是 ArrayList 独有的方法，返回值是 `void`，没有返回值。
* 执行后原来该索引及其之后的元素整体向右挪一格，长度加一。

```java
ArrayList<String> list = new ArrayList<>();

list.add("AAA");
list.add("BBB");
list.add("CCC");

list.add(0, "QQQ");
```

```text
插入之前（集合里已有 AAA、BBB、CCC）

┌─────┬─────┬─────┐
│ AAA │ BBB │ CCC │
└─────┴─────┴─────┘
   0     1     2


执行 list.add(0, "QQQ") 之后

┌─────┬─────┬─────┬─────┐
│ QQQ │ AAA │ BBB │ CCC │
└─────┴─────┴─────┴─────┘
   0     1     2     3
```

* **索引范围只能是 0 到长度**

    * 集合长度为 3 时，可添加的索引范围是 0~3。
    * 0~2 是已经存在的索引位置。
    * 3 是最大索引之外的一位，等于把元素追加到末尾，此时效果等同于单参数 `add`。
    * 写 4 就相当于跳过了一个位置，程序直接报错 `IndexOutOfBoundsException`——当前集合长度为 3，不存在索引 4。
    * > **易错点**：0~3 只是这个案例的值，范围随长度变化。集合里有 DDD、EE、长度为 5 时，可添加的索引范围就是 0~5，要灵活对待。

## remove 的两个重载

* **按元素删 `remove(Object o)`**

    * > **定义**：根据元素内容删除，元素存在删除成功返回 `true`，元素不存在删除失败返回 `false`。

    ```java
    ArrayList<String> list = new ArrayList<>();

    list.add("AAA");
    list.add("BBB");
    list.add("CCC");
    list.add("QQQ");

    boolean res = list.remove("QQQ");

    System.out.println(res);   // true
    ```

* **按索引删 `remove(int index)`**

    * > **定义**：根据索引删除，返回值是被删除的那个元素。
    * 索引越界时代码报错。
    * > **易错点**：删除时索引千万不能写错，不存在的索引会直接让代码报错。

    ```java
    ArrayList<String> list = new ArrayList<>();

    list.add("AAA");
    list.add("BBB");
    list.add("CCC");
    list.add("QQQ");

    System.out.println(list);   // 删除之前
    String res = list.remove(0);
    System.out.println(list);   // 删除之后
    System.out.println(res);    // AAA，被删除的元素还给你
    ```

* 两个重载的差别在于参数类型（`Object` 还是 `int`）和返回值类型（`boolean` 还是元素本身）。

## set

* > **定义**：`set(int index, E e)` 把指定索引上的数据改成新数据，返回值是被替换掉的旧元素。
* 第一个参数是要修改的索引，第二个参数是新元素。
* 要修改的索引必须存在，否则代码报错。

```java
ArrayList<String> list = new ArrayList<>();

list.add("AAA");
list.add("BBB");
list.add("CCC");
list.add("QQQ");

System.out.println(list);           // 修改之前：AAA BBB CCC QQQ

String res = list.set(0, "ZZZ");    // 把 0 索引改成 ZZZ

System.out.println(res);            // AAA，被替换的元素
System.out.println(list);           // 修改之后：ZZZ BBB CCC QQQ
```

## get 与 size

* `get(int index)` 是最纯正的查找方法，按索引查找并返回对应元素，用于获取单个数据。
* `size()` 获取集合的长度。
* 遍历集合要靠 `get` 加 `size` 配合使用：循环变量 `i` 依次表示集合中的每一个索引，`list.get(i)` 依次获取每一个元素。
* > **易错点**：集合的长度不叫 `length`，它叫 `size`。

```java
for (int i = 0; i < list.size(); i++) {
    String s = list.get(i);
    System.out.println(s);
}
```

## 遍历与直接打印集合的区别

* **遍历**

    * > **定义**：遍历是把容器里面的数据一个一个拿出来，拿到之后是打印、计算、截取还是替换都随当前需求而定。
    * 当前例子只是把元素打印出来，实际还可以对每个元素做计算、截取、替换等各种处理。

* **直接打印集合**

    * 只能看一看集合当中有什么，无法操作里面的每一个元素。

* 两者都把数据打到控制台，但本质区别在于能否对元素逐个操作。

## 包装类（Wrapper）

* **集合放不下基本数据类型的解决办法**

    * 集合里无法直接添加 byte、short、int、long、float、double、char、boolean 这八种基本数据类型。
    * 一定要添加时，转成其对应的包装类；包装类本身就是引用数据类型，可以放进集合。
    * 包装类概念本身会在后续单独展开，这里只需记住结论。

* **八种基本数据类型与包装类的对应**

    * 命名规律是把基本数据类型的首字母变成大写。
    * 只有两个特例：`int` 的包装类叫 `Integer`，`char` 的包装类叫 `Character`。
    * 要往集合里添加整数，不写 `int`，写 `Integer`。

    | 基本类型 | 包装类 |
    | :--- | :--- |
    | `byte` | `Byte` |
    | `short` | `Short` |
    | `int` | `Integer` |
    | `long` | `Long` |
    | `float` | `Float` |
    | `double` | `Double` |
    | `char` | `Character` |
    | `boolean` | `Boolean` |

* **从内存角度看 Integer**

    * `Integer` 就是 Java 已经写好的一个类，和 Student、Teacher、Dog、Cat 没有任何区别，只不过它描述的是整数。
    * 它同样有属性、构造方法和成员方法，属性 `value` 被 `final` 修饰，一旦对象创建，属性值就不能改变。
    * `Integer b = 20;` 是一种简化写法，等价于 `Integer b = new Integer(20);`。
    * `int a = 10;` 在栈里声明变量 `a`，整数 10 真实存在于变量中；`Integer b = 20;` 则在堆里创建对象、给属性 `value` 赋 20，再把内存地址通过等号赋给栈里的 `b`。
    * 把基本数据类型用对象包起来之后，就可以在 Integer 类里定义大量操作整数的方法，以后直接调用即可，不必自己写内部逻辑。

```text
int a = 10;

   栈
┌────────────┐
│     a      │
├────────────┤
│     10     │ ← 值直接躺在变量里
└────────────┘


Integer b = 20;   等价于 Integer b = new Integer(20);

   栈                    堆
┌────────────┐      ┌──────────────────┐
│     b      │      │   Integer 对象   │
├────────────┤      ├──────────────────┤
│   0X0011   │─────▶│ value(final)= 20 │
└────────────┘      └──────────────────┘
   ↑ 栈里的 b 只拿到对象的内存地址
```

## 输出方括号格式

* 需求是定义集合添加数字并遍历，输出要求前后有方括号，中间是元素、逗号空格再加元素。
* 泛型不能写 `int`，必须写 `Integer`；添加时可以先定义 `Integer` 变量再 `add`，也可以直接 `list.add(1)`。
* 开头用 `print` 打印左方括号且**不换行**，去掉 `ln`。
* 循环内判断当前是不是最后一个元素：`i == list.size() - 1` 是最大索引，打印元素后接右方括号并换行；否则打印元素后接逗号空格。
* > **易错点**：这里用的是 `print` 而不是 `println`，那个不带 `ln` 的开头输出不要漏掉。

```java
ArrayList<Integer> list = new ArrayList<>();

list.add(1);
list.add(2);
list.add(3);

System.out.print("[");              // print，不换行
for (int i = 0; i < list.size(); i++) {
    Integer num = list.get(i);
    if (i == list.size() - 1) {     // 最后一个元素
        System.out.print(num + "]");
        System.out.println();
    } else {
        System.out.print(num + ", ");
    }
}
```

输出结果：

```text
[1, 2, 3]
```

## 按 id 查找学生索引

* **需求形式**

    * 定义集合添加学生对象（属性有 id、姓名、年龄），遍历集合把每个学生的所有属性打印到控制台，每个学生独占一行。
    * 另定一个方法根据 id 查找学生信息：id 存在返回该索引，不存在返回 -1。
    * 返回 -1 是因为程序中不存在负数索引，返回 -1 别人就知道要找的东西不存在。

* **方法签名推导**

    * 按 id 查找需要知道「查哪个 id」和「在哪个集合里查」，所以两个形参。
    * 题目要求返回索引，所以返回值类型是 `int`。

* **比较 id 用 equals 而不是 ==**

    * id 是 `String` 引用数据类型，不能用 `==` 判断，要用 `equals`。
    * 循环里 `i` 表示索引，`list.get(i)` 取出的才是每一个学生对象，用变量 `stu` 接收，再判断 `stu.getId().equals(id)`。
    * 一致就 `return i`，方法结束；循环全部走完还没找到，才 `return -1`。
    * > **易错点**：`return -1` 必须写在循环外面，不能写在 `else` 里。写在 `else` 里意思是"刚找完 0 索引没有，就认为你不存在"，这不合理；只有循环结束了才能说明集合里所有元素都找完了。

```java
public class Student {
    private String id;
    private String name;
    private int age;

    public Student() {
    }

    public Student(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

```java
ArrayList<Student> list = new ArrayList<>();

Student s1 = new Student("001", "张三", 18);
Student s2 = new Student("002", "李四", 19);
Student s3 = new Student("003", "王五", 20);

list.add(s1);
list.add(s2);
list.add(s3);

for (int i = 0; i < list.size(); i++) {
    Student stu = list.get(i);
    System.out.println(stu.getId() + " " + stu.getName() + " " + stu.getAge());
}

public static int findStudent(String id, ArrayList<Student> list) {
    for (int i = 0; i < list.size(); i++) {
        Student stu = list.get(i);
        if (stu.getId().equals(id)) {
            return i;      // id 一致，把当前索引返回，方法结束
        }
    }
    return -1;             // 只有循环结束仍未找到，才说明 id 不存在
}
```

* 调用结果：`findStudent("002", list)` 返回 `1`；`findStudent("004", list)` 返回 `-1`。

## 链式编程

* > **定义**：链式编程就是把多行代码写在一行当中，用前一个方法的结果继续调用后面的方法。
* `stu.getId().equals(id)` 要分步看：先执行 `stu.getId()` 得到字符串类型的 id，再拿这个结果去调用 `equals`。
* 好处是可以少定义几个变量——不用先用一个变量记录 id、再在判断里点 `equals` 去比较，起名字麻烦时直接链式调用即可。
* 核心逻辑是利用前一个方法的结果继续调用后面的方法。

## 本讲必须掌握的四件事

* **集合是什么**：一句话说得清——长度可变的容器。
* **集合有什么特点**：两个，一是长度可变，二是只能存储引用数据类型、不能存基本数据类型，存整数要写 `Integer`。
* **怎么创建集合对象**：空参构造加泛型，想存什么类型就把这个类型写在尖括号里。
* **集合里常见的方法**：分增删改查四类——`add`、`remove`、`set`、`get`，额外还有 `size` 拿长度。