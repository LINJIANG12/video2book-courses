# 字符串与 StringBuilder

## API 与帮助文档

### API 的本质

* **API（Application Programming Interface，应用程序编程接口）**

    * > **定义**：就是 JDK 提供的各种功能的 Java 类。

    * 这些类不需要我们自己再编写一遍，直接使用即可。

    * 它们已经把底层实现封装起来，`Random` 就把随机算法封装了起来，不需要关心实现细节，只要学会如何调用。

    * 常见的 JDK 提供类：`Scanner` 表示键盘录入，`Random` 表示随机数，每个类表示一类事物，类里都有很多好用的方法。

     ```java
      Random r = new Random();
      int num = r.nextInt(100);
      ```

    * > **易错点**：`Student` 这种类是我们自己写的；Java 本身在底层早已提供好各种各样的 Java 类，只是数量太多记不住。

### 帮助文档的查法与页面结构

* **API 帮助文档（.chm）**

    * Java 把所有类汇总在一个后缀名为 `.chm` 的帮助文档里，双击即可打开。

    * 文档原版是英文的，看到的中文由翻译软件翻译，阅读时能大概读懂即可，不必深究每个词。

    * 文档结构是「包 → 类」两层：`java.io` 包里的类用于读取本地文件内容或把数据保存到文件；`java.lang` 包提供利用 Java 编程语言进行程序设计的基础类。

    * `String` 类就在 `java.lang` 包下，在包里下拉列表中找到它，点进去右侧就是该类的全部信息。

    * 走索引搜类名的五步：打开文档 → 点左上角「显示」→ 点「索引」→ 在输入框输入类名并回车 → 再点「显示」，类就出现在右侧。

* **类页面从上往下的五块**

    ```text
    +--------------------------------------------------------+
    | Random                              <- 类声明：定义在哪个包
    +--------------------------------------------------------+
    | java.lang.Object                    <- 继承结构：父类是谁
    | implements Serializable            <- 额外实现的接口
    | 已知直接子类：XXX                    <- 有没有子类
    +--------------------------------------------------------+
    | 类描述：适用于生成伪随机数的流        <- 这个类能干什么
    | Since: 1.0                          <- 从哪个 JDK 版本开始有
    +--------------------------------------------------------+
    | 构造方法 Random()                   <- 对象怎么被创建出来
    +--------------------------------------------------------+
    | 成员方法 nextInt() / nextInt(int bound) / nextDouble() |
    +--------------------------------------------------------+
    ```

    * 类声明那一行决定导包：`Random` 定义在 `java.util` 包，所以使用它要写 `import`。

    * 文字描述一般只读第一行即可；`Random` 的实例即对象，适用于生成伪随机数的流。`Serializable` 是可序列化接口，学习 IO 时再回头看。

    * `Since` 版本决定可用范围：写 `1.0` 表示所有 JDK 版本都能用；写 `8` 表示 JDK 8 才引入，版本大于等于 8 才能用。

    * 构造方法决定对象如何被创建，`Random` 的空参构造正是此前创建它对象的方式。

    * 查「获取随机小数的方法」的思路：返回值必须是小数类型，锁定返回值类型为 `double` 或 `float` 的成员方法，落在 `nextDouble()` 与 `nextFloat()` 上。

    * | 方法 | 返回值 | 取值范围 |
      | :--- | :--- | :--- |
      | `r.nextDouble()` | `double` | `0.0`（含）～`1.0`（不含） |
      | `r.nextFloat()` | `float` | `0.0`（含）～`1.0`（不含） |

    * 两个方法作用完全一样，区别只在返回值类型；小数默认类型是 `double`，所以代码里用 `nextDouble`。

    * IDEA 中写完 `Random r = new Random();` 会自动多出一行 `import`，这叫导包，相当于定位类所在位置；也可在左侧 `Libraries` → `java.base` → `java` → `util` 里展开找到源文件。

### 导包规则

* **import 导包（导包）**

    * > **结论**：不需要导包的情况只有两种——用本包中的类、用 `java.lang` 包下的类；除此之外全部都要导包。

    * 与本类同包的类直接使用，IDEA 里不会出现 `import`。

    ```java
    package com.itheima.a01api;

    public class Test {
        public static void main(String[] args) {
            Student student = new Student();   // 同包，直接用
        }
    }
    ```

    * 不在同一包的类要写 `import`，例如 `import com.itheima.aa.Teacher;`。

    * `java.lang` 是 Java 提供的最核心、最基础的包，用它的类不用导包，`String s = "aaa";` 上方不会出现 `import java.lang.String;`。

## String 类与常量池

### String 的位置与不可变性

* **String 类**

    * > **定义**：`String` 是 Java 已定义好的表示字符串的类，定义在 `java.lang` 包下。

    * Java 程序中所有字符串文字（双引号引起来的内容）都是 `String` 类的对象。

    * 字符串与任意数据类型相加都做拼接操作，并产生一个新的字符串。

* **字符串不可变**

    * > **定义**：字符串的内容在创建完成之后不能发生改变。

    * 两个字符串拼接时，不改变原本两个字符串的内容，而是产生一个新字符串，整个过程一共出现三个字符串。

    ```text
    正确：拼接不动老对象，另造一个新的

      "AB"           "CD"            "ABCD"
      0x001          0x002           0x003（新对象）
        +------- 拼接 -------+

    错误：以为第一个字符串被"撑大"了

      "AB" --> "ABCD"
      ^ 这样理解不对，"AB" 并未变成 "ABCD"
    ```

    * 写 `name = name + "真帅";` 并不是改变字符串内容，而是创建新字符串再赋值给变量，旧对象纹丝不动，此时总共出现两个字符串。

    * > **易错点**：任何情况下都不要认为拼接会修改已有字符串的内容，字符串永远不可变。

### 创建 String 对象的两种方式

* **直接赋值**

     ```java
      String s = "ABC";
      System.out.println(s);   // ABC
      ```

    * 代码最简单，也是最常用的方式；除了简单，它还会复用串池里的数据，因此更节约内存。

* **new 搭配四种构造方法**

    * `new String()`：创建没有任何内容的字符串对象，`System.out.println("--" + s1 + "@");` 打印出来两个分隔符中间是空的。

    * `new String(String 原串)`：根据传入字符串的内容再创建一个新的字符串对象，这种方式用得不多，但常被拿来做面试题。

    * `new String(char[] chs)`：把字符数组里所有内容放进字符串，`{'a','b','c','d','e'}` 得到 `ABCDE`。

    * `new String(byte[] bytes)`：按 ASCII 码表把数字转成字符，`{97,98,99,100,101}` 得到 `ABCDE`。

    ```java
    char[] chs = {'a', 'b', 'c', 'd', 'e'};
    String s3 = new String(chs);      // ABCDE

    byte[] bytes = {97, 98, 99, 100, 101};
    String s4 = new String(bytes);   // ABCDE
    ```

### 常量池与两种方式的内存差别

* **StringTable 串池（字符串常量池）**

    * > **定义**：用来存储字符串的区域，配合栈、堆、方法区一起构成字符串的内存模型。

    * 位置随版本变化：JDK 7 以前（1 到 6）在方法区，JDK 7 时从方法区挪到堆内存；不论在哪，运行机制不变。

* **直接赋值的复用机制**

    * 直接赋值产生的字符串对象直接放在串池里；串池不会上来就创建对象，先观察池里有没有这个字符串。

    * 池里没有才创建一个，并把它的内存地址赋给变量；池里已经有了就复用，把同一地址再次赋给新变量。

    ```text
      String s1 = "ABC";        <- 串池里没有，才创建
    栈  s1  ----------------> 0x0011
    堆  StringTable:  0x0011 : "ABC"

      String s2 = "ABC";        <- 串池里已有，直接复用
    栈  s2  ----------------> 0x0011
    堆  StringTable:  0x0011 : "ABC"

    两个变量指向同一块空间：创建一次，用无数次
    ```

* **new 出来的一个都不复用**

    * 只要是 `new` 出来的就是在堆里开辟新的空间，`new` 一次开一个，从不复用。

    ```text
    栈  chs ---> 0x0011     堆  0x0011 : a b c d e   <- char[] 数组

    栈  s1  ---> 0x0022     堆  0x0022 : "ABCDE"     <- 第一次 new
    栈  s2  ---> 0x0033     堆  0x0033 : "ABCDE"     <- 第二次 new

    0x0022 与 0x0033 内容一样、地址不同
    ```

    * `new String(已有字符串)` 同理：原串在串池，`new` 出来的对象在堆里另开空间，两个对象内容相同地址不同。

    * 变量 `String s` 声明在栈上，变量里记录的是字符串对象的内存地址，不是内容本身。

## 字符串比较

### == 比较的是什么

* **== 比较运算符**

    * > **定义**：`==` 比较的是变量里记录的内容，变量记录什么就比什么。

    * 基本数据类型变量里记录真实数据，比的是数据本身；引用数据类型变量里记录内存地址，比的是地址。

    ```text
           基本数据类型                     引用数据类型
      ------------------------------  ------------------------------
      int a = 10;                      String s1 = new String("张三");
      int b = 20;                      String s2 = "张三";
          |                                   |          |
        真实数据                            内存地址
          |                                   |          |
         10 vs 20                           地址A vs 地址B
          |                                   |          |
          +------------ 比较 ---------------+--- 比较 -+
          ↓                                   ↓
        false                              false
    ```

    * 内容完全一样但一个 `new` 出来、一个直接赋值的两个字符串，用 `==` 比较结果是 `false`。

    * 没有任何玄学，就是变量里放什么就比什么这一条规则。

### equals 与 equalsIgnoreCase

* **equals()**

    * > **定义**：比较两个字符串的内容是否完全相等，完全一样返回 `true`，否则返回 `false`，返回值是布尔类型。

    * 用当前字符串调用，把要比的字符串传进小括号：`username.equals(rightUsername)`。

    * 比较用户名、密码时用它，要求完全一致，多一个空格、少一个字母都不行。

* **equalsIgnoreCase()**

    * > **定义**：比较字符串内容时忽略大小写，小写 `a` 与大写 `A` 视为相同。

    * 比较验证码时用它，验证码不区分大小写。

    ```java
    String username = "张三";
    String rightUsername = "张三";

    boolean result1 = username.equals(rightUsername);              // true
    boolean result2 = username.equalsIgnoreCase(rightUsername);   // true
    ```

    * IDEA 中写完调用表达式按 `Ctrl + V` 可自动生成左边的 `boolean result =`，右键即可运行。

### 模拟登录的三次机会

* **模拟登录（三次机会）**

    * 题目形态：已知正确用户名和密码，模拟用户登录，总共给出三次机会，登录后给出相应提示。

    * 正确用户名取 `张三`，正确密码取 `123456`，键盘录入用 `Scanner sc = new Scanner(System.in);`。

    * 判断要两个条件同时成立，必须用 `&&` 连接：用户名和密码全都对才算登录成功。

    * 键盘录入和判断都要放进循环——只把判断放进循环、录入留在外面，等于拿同一份数据比了三遍，违背「三次机会」要求每次失败后能重新输入。

    * `Scanner sc = new Scanner(System.in);` 这一句不要塞进循环，循环外创建一次即可。

    ```java
    String rightUsername = "张三";
    String rightPassword = "123456";
    Scanner sc = new Scanner(System.in);

    for (int i = 1; i <= 3; i++) {
        System.out.println("请输入用户名");
        String username = sc.next();

        System.out.println("请输入密码");
        String password = sc.next();

        boolean result = username.equals(rightUsername) && password.equals(rightPassword);
        if (result) {
            System.out.println("登录成功");
            break;                                  // 一旦登录成功，循环结束
        } else {
            if (i <= 2) {
                System.out.println("登录失败，还剩下" + (3 - i) + "次机会");
            } else {
                System.out.println("登录失败，账号" + username + "被锁定，请联系客服");
            }
        }
    }
    ```

    * 不写循环也能先跑通一次：先比一次、再加提示，等逻辑对了才用循环补成三次。

    * 剩余机会用 `3 - i` 算：第一次失败 `i = 1` 剩 2 次，第二次 `i = 2` 剩 1 次，第三次 `i = 3` 走锁定分支。

    * `break` 必须加，否则第一次就输对时程序还会继续要求输入，登录成功之后不会跳出这一环。

    * > **注意**：这段代码有一个已知缺陷，靠现有知识改不了——若前两次输的是 `张三`/`123456`，第三次故意输成 `李四`，最后被锁定的会是 `李四`，因为打印时拿到的是最后一次录入的 `username`；等学到本地文件 `File` 或 MySQL 数据库才能解决。

* **布尔变量的判断写法**

    * > **易错点**：对布尔变量判断时不要写 `==`，因为容易漏写一个等号，`if (result = true)` 就变成了赋值而不是判断，程序铁定走 `if` 分支，输错的账号也会显示登录成功。

    * 正确写法是把变量直接写在小括号里：`if (result) { ... } else { ... }`，表示判断变量里记录的值是真是假。

    * 结果为 `boolean` 的基础类型变量本来也用不上 `equals`，直接用 `==` 判断即可。

## 遍历与字符统计

### charAt 与 length

* **charAt(int index)**

    * > **定义**：根据索引返回字符串里对应的字符，返回值类型是 `char`。

    * 字符串也有索引，规则与数组一模一样——从 0 开始逐个往后。

    ```text
          索引：0    1    2    3    4    5    6
          字符：你   好   1    2    3
          String str = "你好123"
          str.length()  ->  7        合法索引：0 ~ 6
    ```

* **length()**

    * > **定义**：返回字符串的长度，也就是里面字符的个数。

    * > **易错点**：数组的 `length` 是属性，调用时后面不加小括号；字符串的 `length()` 是方法，用字符串调用时必须加小括号，漏了括号拿到的是方法本身的引用而不是长度。

    * 索引超范围会抛 `StringIndexOutOfBoundsException`，例如长度为 7 的字符串取索引 7，报错信息是 `Index 7 out of bounds for length 7`，与数组越界规则完全一致。

* **遍历写法**

    ```java
    String str = "你好123";

    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);      // c 依次表示每一个被遍历到的字符
        System.out.println(c);
    }
    ```

    * 字符串没有 `str.fori` 的循环提示，要写成 `str.length().fori` 才会出提示，即先点出 `length()` 再点 `for i`。

### 大写小写数字计数

* **字符区间判断**

    * 判断当前字符是大写、小写还是数字，用三段区间：`'a'` 到 `'z'`、`'A'` 到 `'Z'`、`'0'` 到 `'9'`。

    * > **易错点**：区间右边写的都是字符不是阿拉伯数字，遍历拿到的是 `char`，必须用单引号写成 `'0'`、`'1'` 这种形式。

    * `System.out.println('0');` 打印出 `0` 本身；`System.out.println('0' + 0);` 打印出 `48`，因为字符 `'0'` 在 ASCII 码表里对应 48。

* **计数器思想**

    * > **定义**：需要统计次数时，定义一个变量初始赋值为 `0`，在符合要求的位置让它自增。

    * 既不是大写、也不是小写、也不是数字的字符走 `else`，可以提示「当前字符不参与统计」。

    ```java
    Scanner sc = new Scanner(System.in);
    System.out.println("请输入一个字符串");
    String str = sc.next();

    int upperCount = 0;      // 统计大写字符的
    int lowerCount = 0;      // 统计小写字符的
    int numberCount = 0;     // 统计数字字符的

    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        if (c >= 'a' && c <= 'z') {
            lowerCount++;
        } else if (c >= 'A' && c <= 'Z') {
            upperCount++;
        } else if (c >= '0' && c <= '9') {
            numberCount++;
        } else {
            System.out.println(c + " 当前字符不参与统计");
        }
    }

    System.out.println("大写字符出现了" + upperCount + "次");
    System.out.println("小写字符出现了" + lowerCount + "次");
    System.out.println("数字字符出现了" + numberCount + "次");
    ```

## 截取与替换

### substring

* **substring(beginIndex, endIndex)**

    * > **定义**：从 `beginIndex` 这个索引开始，截到 `endIndex` 这个索引结束，规则是包头不包尾、包左不包右。

    * 写 `0, 3` 表示包含索引 0、不包含索引 3，真正拿走的只是 0、1、2 三个位置的数据。

    ```text
    sStr = "ABCDEFG"

    索引:   0    1    2    3    4    5    6
    字符:   A    B    C    D    E    F    G

    substring(1, 5)
            ^               ^
            包含起点        这个位置不算在内

    真正拿走的:  索引 1、2、3、4  ->  "BCDE"

    substring(1)
            ^
            从 1 一直拿到末尾                        ->  "BCDEFG"
    ```

    * 只有一个 `beginIndex` 的重载：只给起点，默认一路截到字符串末尾。

    * > **结论**：只有方法的返回值才是截取之后的小串，它不会对调用者的那个字符串产生任何影响，因为字符串本身不可变；必须用一个变量去接收返回值，否则打印出来的还是原来那个字符串。

    ```java
    String sStr = "ABCDEFG";

    String res  = sStr.substring(1, 5);   // BCDE
    String res2 = sStr.substring(1);      // BCDEFG
    ```

    * IDEA 中敲完调用表达式按 `Ctrl + Alt + V` 可以自动在左边生成接收的变量名。

* **只保留第一个字符**

    * 两种写法结果一致：用 `charAt` 取索引 0 的一个字符，或用 `substring(0, 1)` 因为包头不包尾只拿一个字符。

    ```java
    String username = "张三";

    char firstName = username.charAt(0);
    String encryption = firstName + "***";       // 张***

    String firstName2 = username.substring(0, 1);
    String encryption2 = firstName2 + "***";    // 张***
    ```

    * 截取结果不对时，直接改括号里的参数，绝大多数情况是索引写错了。

### replace 与敏感词过滤

* **replace(被替换数据, 替换数据)**

    * > **定义**：把字符串中指定的内容替换成新的内容，第一个参数是被替换的数据，第二个参数是用来替换的数据。

    * 同样只有返回值才是替换之后的结果，直接打印原字符串仍是原样。

    ```java
    String sStr = "你玩的好菜呀 TMD";
    String res = sStr.replace("TMD", "***");
    System.out.println(res);
    ```

* **substring 与 replace 的区分**

    | 方法 | 依据什么 | 匹配方式 | 典型场景 |
    | :--- | :--- | :--- | :--- |
    | `substring` | 位置 | 从 `beginIndex` 到 `endIndex` 固定区间取数据 | 只保留用户名的第一个字 |
    | `replace` | 内容 | 全串替换掉指定的那些内容 | 把脏话过滤成 `***` |

    * `substring` 不管里面是什么内容，按固定位置取数据；`replace` 替换的目标位置不确定，可能在前、在中间、也在最后，所以它是替换指定内容。

* **敏感词过滤的三步**

    * 敏感词库用 `String[]` 数组提前存好；键盘录入用户说的话用 `Scanner`；循环遍历词库逐个替换。

    ```java
    String[] arr = {"TMD", "SB", "NMD", "LGO"};

    Scanner sc = new Scanner(System.in);
    System.out.println("请输入您想说的话：");
    String msg = sc.next();

    for (int i = 0; i < arr.length; i++) {
        msg = msg.replace(arr[i], "***");   // 必须把结果重新赋值回 msg
    }

    System.out.println(msg);
    ```

    * > **易错点**：循环里一定要把替换结果再次赋值给 `msg`，否则循环转一圈什么都没干。

    ```text
      初始 msg:   你玩的好菜呀 TMD  SB  LGO

      第 1 轮   i = 0   arr[0] = "TMD"  ->  msg: 你玩的好菜呀 ***  SB  LGO
      第 2 轮   i = 1   arr[1] = "SB"   ->  msg: 你玩的好菜呀 ***  ***  LGO
      第 3 轮   i = 2   arr[2] = "NMD"  ->  串里根本没有它，不替换，msg 保持不变
      第 4 轮   i = 3   arr[3] = "LGO"  ->  msg: 你玩的好菜呀 ***  ***  ***

      循环结束，打印 msg
    ```

    * 词库里的词在原串中不存在时 `replace` 不会报错，直接不动就是了。

    * 敏感词过滤里可以先用 `contains` 判断当前串有没有脏话，有再替换，没有就不替换。

## String 常用方法

### contains、startsWith 与 endsWith

* **contains(CharSequence)**

    * > **定义**：判断某个小串在大串中是不是包含，返回布尔类型。

    * 形参类型是 `CharSequence`，它是接口，`String` 实现了这个接口，所以传字符串对象过去属于接口的多态。

    * 小串必须在大串中连续，`"ABCDEFG".contains("ABD")` 是 `false`，中间隔着别的字符就不算包含。

* **startsWith 与 endsWith**

    * 是一组：`startsWith` 判断是否以某个小串开头，`endsWith` 判断是否以某个小串结尾。

    * `startsWith` 常用一个参数的版本；两个参数的版本里第二个参数表示规定起始查找的位置，即从指定索引开始拿后面这一段判断是否以该小串开头。

    ```java
    String sStr = "ABCDEFG";

    boolean b2 = sStr.startsWith("BC");       // true
    boolean b3 = sStr.startsWith("ABC", 1);   // false，1 索引开始是 BCDEFG
    boolean b4 = sStr.endsWith("ABC");        // false
    boolean b5 = sStr.endsWith("FG");         // true
    ```

    * `endsWith` 以后最常见的用法是判断文件后缀名：写 `".jpg"` 判断是不是图片，写 `".txt"` 判断是不是文本文件。

### indexOf 与 lastIndexOf

* **indexOf(…)**

    * > **定义**：在大串中查找某个小串或某个字符的索引，查的是第一次出现的位置。

    * 重载较多：可以传字符、可以传字符串、还可以指定 `fromIndex`；不指定时默认从索引 0 开始查找。

    * > **易错点**：用字符形式调用时要看清重载的形参类型，形参是 `int` 的那个重载要写字符对应的整数值（IDEA 会自动填成 `97`），形参是 `char` 的才写 `'a'`。

    * 大串里有多个相同小串时，返回的仍是第一次出现的索引。

* **lastIndexOf(…)**

    * > **定义**：查最后一次出现的位置，`indexOf` 从前往后查，`lastIndexOf` 从后往前查。

    ```java
    String sStr = "ABCDEFG";

    int i1 = sStr.indexOf('a');         // 0
    int i2 = sStr.lastIndexOf('a');    // 4
    int i3 = sStr.indexOf("A");        // -1
    ```

    * 没找到时返回 `-1`，因为不存在负数索引，一看到 `-1` 就知道要查找的内容不存在。

### isEmpty、toCharArray 与 trim

* **isEmpty()**

    * > **定义**：判断字符串里有没有内容，也就是长度是不是为零。

    * `""` 长度为零返回 `true`；注册登录场景可用它判断用户到底有没有输入数据。

* **toCharArray()**

    * > **定义**：把字符串变成一个字符数组，数组长度跟字符串长度一模一样，里面装的就是这个字符串里的每一个字符，不需要传参数。

    ```java
    String sStr = "ABCDEFG";

    char[] array = sStr.toCharArray();
    for (int i = 0; i < array.length; i++) {
        System.out.print(array[i]);
    }
    // A B C D E F G
    ```

    * 用途是绕开字符串不可变：想把 `ABC` 的零索引改成大写 `A`，就先转成字符数组、改变数组里对应位置、再用 `new String(数组)` 转回字符串。

* **toUpperCase 与 toLowerCase**

    * > **定义**：把字符串里的英文字母整体转成大写或小写，返回新字符串。

    * 只能转换英文字母，中文的「一」转不成大写。

* **trim()**

    * > **定义**：去除字符串头尾的空格，中间空格保留。

    ```java
    String str3 = "  A B C  ";
    String train = str3.trim();
    System.out.println(train);      // A B C
    ```

    * 用户在输入框不小心按出前后空格很难被发现，校验用户名和密码前先 `trim` 一下能改善体感。

* **方法的记忆策略**

    * > **结论**：`String` 的常见方法不需要背，数量太多背也背不上；学会查看 API 文档，或在 AI 帮助下生成并看懂就够了。

    * 在文档输入框里输入 `String` 回车，`All Classes` 里往下拉就是全部方法列表。

## StringBuilder

### 拼接效率的差距

* **百万次拼接的实测**

    * 用原始字符串拼接 100 万次，统计运行时间得到 `161338` 毫秒，换算成秒是 161 秒，不到 3 分钟。

    * 秒与毫秒的换算关系是 `1 秒 = 1000ms`；运行时间用 `System.currentTimeMillis()` 记录起止时刻相减得到，单位毫秒。

    ```java
    String s = "";
    long start = System.currentTimeMillis();

    for (int i = 0; i < 1000000; i++) {
        s += "ABCDEFG";
    }

    long end = System.currentTimeMillis();
    System.out.println("程序运行的总时间为：" + (end - start) + "ms");
    ```

    * 换成 `StringBuilder` 拼接同样的数据，右键运行「刷」的一下就结束了。

* **冗余数据与单一容器**

    * 原始拼接从左往右依次进行，每一步都新建一个字符串，中间产物全是垃圾。

    * `StringBuilder` 全程只认同一个容器，用 `toString()` 一次性变回字符串。

    ```text
    原始拼接：每一步都造一个新字符串，中间产物全是垃圾

      "AAABBB"        + "CCC"   ->  新建 "AAABBBCCC"
      "AAABBBCCC"     + "DDD"   ->  新建 "AAABBBCCCDDD"
      "AAABBBCCCDDD"  + "EEE"   ->  新建 "AAABBBCCCDDDEEE"   <- 最终结果

      100 万个字符串，就是 100 万个这样的垃圾等着被丢掉

    StringBuilder：从头到尾只认这一个容器

      同一个容器 sb
        |-- append("AAA")
        |-- append("BBB")
        |-- append("CCC")
        |-- append("DDD")
        `-- append("EEE")
                 |
                 `-- toString() 一次性变成 "AAABBBCCCDDDEEE"
    ```

    * > **结论**：拼接过程中操作的都是同一个容器，不产生过多冗余数据，所以效率非常高。

### 构造方法与四个常用方法

* **两种构造方法**

    * 空参构造 `new StringBuilder()`：容器里不包含任何内容，`length()` 返回 `0`。

    * 带参构造 `new StringBuilder("ABC")`：容器里包含创建时传入的内容。

* **四个成员方法**

    | 方法 | 作用 |
    | :--- | :--- |
    | `append` | 添加数据，默认添加在末尾 |
    | `reverse` | 把容器里的内容进行反转 |
    | `length` | 获取长度，也就是里面字符的个数 |
    | `toString` | 把容器里的内容再变回字符串 |

    ```java
    StringBuilder sb2 = new StringBuilder("ABC");

    sb2.append("AAA");       // ABCAAA
    sb2.reverse();           // CBA
    System.out.println(sb2.length());

    String res = sb2.toString();   // 变回字符串后接收
    ```

    * 反复 `append` 会一直往后加，`"ABC"` 依次加 `AAA`、`BBB`、`CCC` 后变成 `ABCAAABBBCCC`。

    * `StringBuilder` 本身只是容器，并不是字符串，所以添加或反转之后还要 `toString` 变回字符串再用变量接收。

    * 变量名常取类型首字母：`int i` 取 `i`，`String s` 取 `s`，`StringBuilder` 是两个单词，取每个单词首字母 `SB`；光标放在变量上按 `shift + F6` 可一次改掉所有用到的地方。

### insert 与指定位置插入

* **insert(int offset, …)**

    * > **定义**：在指定的下标位置插入对应的元素，不关心这个位置原本是什么，只管往里塞。

    * 典型场景是给字符串中所有整数的前后都加上星星：直接在 `String` 本身上操作非常麻烦，用 `StringBuilder` 的 `insert` 走到哪儿算到哪儿，遇到整数就在它前后的下标处各插一次星号即可。

    * 配合「边遍历边记录连续数字段的起止下标」的思路，用 `Character.isDigit()` 判断当前字符是不是数字，就能扫出连续的一段数字。

## 字符串算法练习

### 数组拼接成指定格式

* **原始 String 方式**

    * 把 `int[]` 数组拼成 `(1, 2, 3)`：开头左括号、结尾右括号，数字之间逗号加空格。

    * 不知道先赋什么值时的默认规则：赋一个长度为 `0`、没有内容的字符串；这里可以直接把左括号作为初始值，少写一次拼接。

    ```java
    String str = "(";
    for (int i = 0; i < arr.length; i++) {
        if (i == arr.length - 1) {
            str = str + arr[i] + ")";     // 最后一个元素，直接收尾
        } else {
            str = str + arr[i] + ", ";    // 其余元素，后面跟逗号空格
        }
    }
    ```

    ```text
      初始        str = "("
      第 1 轮 i=0  "("   + arr[0]=1  + ", "   ->  str = "(1, "
      第 2 轮 i=1  "(1, " + arr[1]=2  + ", "   ->  str = "(1, 2, "
      第 3 轮 i=2  "(1, 2, " + arr[2]=3 + ", " ->  str = "(1, 2, 3, "
    ```

    * > **易错点**：不判断最后一个元素就直接拼逗号空格，最后一个数字后面会多出一个逗号空格。

    * 功能性代码写在工具类里，构造方法私有，对外提供静态方法，方法最后用 `return str;` 而不是打印。

    ```java
    public class ArrayUtils {

        private ArrayUtils() {
        }

        public static String arrayToString(int[] arr) {
            String str = "(";
            for (int i = 0; i < arr.length; i++) {
                if (i == arr.length - 1) {
                    str = str + arr[i] + ")";
                } else {
                    str = str + arr[i] + ", ";
                }
            }
            return str;
        }
    }
    ```

* **StringBuilder 方式**

    ```java
    public static String arrayToString2(int[] arr) {
        StringBuilder sb = new StringBuilder("(");

        for (int i = 0; i < arr.length; i++) {
            if (i == arr.length - 1) {
                sb.append(arr[i]).append(")");
            } else {
                sb.append(arr[i]).append(", ");
            }
        }

        return sb.toString();
    }
    ```

    * 效果与原始方式完全一致，数组 `{1,2,3,4,5}` 得到 `(1, 2, 3, 4, 5)`。

    * > **易错点**：方法的返回值类型是 `String`，所以返回时必须用 `sb.toString()` 把 `StringBuilder` 变回字符串，不能把 `StringBuilder` 对象本身直接扔出去。

### 循环反转

* **题目形态**

    * 键盘录入一个字符串，将该字符串反转；当输入「拜拜」时程序才停止；录入的是一个词，所以用 `next()`。

* **反转的两种写法**

    * 原始 String 方式也能转：倒着遍历取字符再拼接，循环写成 `for (int i = str.length() - 1; i >= 0; i--)`，但很麻烦。

    * `StringBuilder` 提供了 `reverse` 方法，一步到位。

    ```java
    StringBuilder sb = new StringBuilder(str);
    String res = sb.reverse().toString();
    ```

* **循环的选择**

    * > **结论**：知道循环次数用 `for`，不知道次数但知道循环结束条件用 `while`；本题不知道用户要反转多少个字符串，所以用 `while`。

    ```java
    Scanner sc = new Scanner(System.in);

    while (true) {
        System.out.print("请输入一个字符串：");
        String str = sc.next();

        if (str.equals("拜拜")) {
            System.out.println("欢迎使用本系统");
            System.out.println("期待与您的下次见面");
            break;
        }

        StringBuilder sb = new StringBuilder(str);
        System.out.println(sb.reverse().toString());
    }
    ```

    * `Scanner sc = new Scanner(System.in);` 放在循环外面，创建一次即可。

    * > **易错点**：不能只把 `if` 判断放进循环，录入留在循环外面会把 `123` 反复无限反转再打印；要把所有代码都放进循环。

    * 习惯上先把主体逻辑写完运行验证，确认一次录入能正常反转、输入「拜拜」能正常退出，最后再补循环。

### 按八位拆行并补零

* **题目形态**

    * 键盘录入任意字符串，按长度为 8 拆分每一段并输出；长度不是 8 整数倍的，在后面补零；空格字符串不处理。

    * 例如 `ABCDABCDA` 长度为 9，第一行输出前八个 `ABCDABCD`，第二行输出 `A` 并补零到 8 位。

    * 不管用哪种解法，「最后一行的长度」和「要补的零的个数」这两个数都绕不开，区别只在于什么时候算。

* **解法一：遍历打印，打满八个换行**

    ```java
    for (int i = 0; i < str.length(); i++) {
        System.out.print(str.charAt(i));    // 只打印不换行，用 print 不用 println

        if ((i + 1) % 8 == 0) {
            System.out.println();           // 只换行，不打印任何数据
        }
    }
    ```

    * > **易错点**：判断时不能直接写 `i % 8 == 0`，因为 `i` 从 0 开始；不加一的话索引 0 取余就为 0，第一个字符 `A` 打出来马上换行，结果变成第一行 `A`、第二行 `BCDABCDA`。

    * 补零个数这样算：`lastLineCount = str.length() % 8` 是最后一行字符数，`count = 8 - lastLineCount` 是要补的零个数，再循环打印 `count` 个 `0`。

* **解法二：先把零补齐，再整段切下来**

    * 和解法一的差别相当于把「补零」与「换行」两步对调：先在字符串后面拼上零凑够 8 的整数倍，再每 8 个切一段输出。

    ```java
    int lastLineCount = str.length() % 8;
    int count = 8 - lastLineCount;

    String line = "";
    if (count != 0) {
        line = "12345678".substring(0, count);
    }
    str = str + line;

    for (int i = 0; i < str.length(); i += 8) {
        String res = str.substring(i, i + 8);
        System.out.println(res);
    }
    ```

    * 补零的这段是关键：靠 `substring` 包头不包尾，`substring(0, 7)` 正好取到 0 到 6 这七个字符。

    * 循环步长 `i += 8` 的含义是每截取八个索引就往前跳八步，第一行从索引 0 开始，第二行就从索引 8 开始。

    ```text
      输入 ABCDABCDA，长度 9
      9 % 8 = 1        ->  最后一行只有 1 个字符
      要补的零数 = 8 - 1 = 7

      补零之后： ABCDABCDA0000000     （共 16 个字符，正好 2 行）

      i = 0  ->  substring(0,  8)  ->  ABCDABCD
      i = 8  ->  substring(8, 16)  ->  A0000000
      i = 16 ->  循环条件不成立，跳出
    ```

    * 补零处加 `if (count != 0)` 判断是必要的，`count` 为 0 时不需要补；数据量少时用原始拼接即可，用 `StringBuilder` 拼接同样没问题。

### 打乱字符串内容

* **间接改变内容的思路**

    * 字符串创建完后内容不能改变，所以直接改字符串与已学知识相违背；办法是先转成字符数组，在数组中变化，再转回字符串，属于间接操作，得到的也是一个新字符串。

    * 三个核心环节：想到用 `toCharArray` 绕开不可变；在数组中做随机交换；交换完再用 `new String(arr)` 变回字符串。

    ```java
    String str = "ABCD";
    char[] arr = str.toCharArray();

    Random r = new Random();
    for (int i = 0; i < arr.length; i++) {
        int index = r.nextInt(arr.length);
        char temp = arr[i];
        arr[i] = arr[index];
        arr[index] = temp;
    }

    String result = new String(arr);
    System.out.println(result);
    ```

    * 打乱数组的固定套路：定义数组 → 遍历取到每个元素 → 在循环中获取随机索引 → 当前元素与随机索引上的元素交换 → 循环结束后再遍历查看结果。

    * > **注意**：`nextInt(arr.length)` 的括号里只能写 `arr.length`，不能写 20、30、40 之类的固定数字，因为上面的字符串长度不固定；长度是 10 时 `nextInt(10)` 的随机范围正好是 0 到 9。

### 超大数相加

* **题目形态**

    * 定义两个字符串记录非负整数（可能为零，也可能是正数），求它们的和，例如 `12395 + 133 = 12528`。

* **为什么不能走捷径**

    * 字符串直接相加做的是拼接操作，不是加法；转成 `int` 再相加也不行，因为 `int` 最大值是 `2147483647`，字符串里的数比它大就接不住。

    * 所以要把字符串每一位都放进 `int` 数组里算，数组能记录很多内容。

* **三个必须先想清楚的细节**

    * 两个数组长度要一样长，这样计算时索引才对得上，比如 `12395 + 123` 要让两个数组的个位对个位、十位对十位。

    * 结果数组的长度要加一，因为最高位相加可能进位，五个九相加结果是六位数。

    * 计算从个位数也就是数组的最大索引开始；计算结果不能整段存入结果数组，个位存进结果数组，十位往前进一位，所以必须定义一个表示进位的变量，每一位相加都要再加上进位。

    ```text
      index       4    3    2    1    0
      str1        1    2    3    9    5     len = 5
      str2        0    0    1    3    3     len = 3，左边补 0

      从最大索引 4（个位）开始算，每次相加还要再加上进位 num：

        5 + 3 + 0  =  8     个位 8 存进 sum，不进位
        9 + 3 + 0  = 12     个位 2 存进 sum，十位 1 存进进位 num
        3 + 1 + 1  =  5     个位 5 存进 sum，不进位     <- 百位别忘了再加 1
        2 + 0 + 0  =  2     个位 2 存进 sum，不进位
        1 + 0 + 0  =  1     个位 1 存进 sum，不进位

      结果：  1  2  5  2  8   ->  12528
    ```

* **字符转数字统一减 48**

    * 字符 `'1'` 在 ASCII 码表里对应 49，减 48 就得到整数 1。

    | 字符 | `'1'` | `'2'` | `'3'` | `'4'` | `'5'` | `'6'` | `'7'` | `'8'` | `'9'` |
    | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
    | 码值 | 49 | 50 | 51 | 52 | 53 | 54 | 55 | 56 | 57 |
    | 减 48 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |

    * 数字字符在 ASCII 码表里是连续的，所以减 48 这种方式对 `0` 到 `9` 一律通用。

* **copyData 方法**

    * > **定义**：`copyData(String str, int len)` 接收字符串和数组长度两个参数，返回 `int[]`，作用是把字符串中的数据放入 `int` 数组。

    * 形参这样定是因为完成这件事必须先拿到字符串，还要知道数组长度——数组长度跟字符串长度不一定相等。

    ```java
    public static int[] copyData(String str, int len) {
        int[] arr = new int[len];
        int index = arr.length - 1;          // 表示数组中应存入的位置，初始为最大索引

        for (int i = str.length() - 1; i >= 0; i--) {
            char c = str.charAt(i);
            int num = c - 48;
            arr[index] = num;
            index--;
        }

        return arr;
    }
    ```

    * 数组长度以长的那个为准：`int len = str1.length() >= str2.length() ? str1.length() : str2.length();`，这一行也可以用 `if` 大括号判断，三元运算符只是更紧凑的写法。

    * > **易错点**：必须倒着遍历字符串并且用单独的 `index` 变量从最大索引开始存，正着遍历、用 `i` 直接下标会把 `133` 存成 `[1, 3, 3, 0, 0]`，位置对不上；倒着遍历才能得到 `[0, 0, 1, 3, 3]`，即 `00133`，个位才能对个位。

    ```text
      字符串 "133"，数组长度 5

      正着遍历（错的）：
        arr[0] = '1' - 48 = 1
        arr[1] = '3' - 48 = 3
        arr[2] = '3' - 48 = 3
        -> arr = [1, 3, 3, 0, 0]     得到 13300，位置对不上

      倒着遍历（对的）：
        i = 2  ->  index = 4  ->  arr[4] = 3
        i = 1  ->  index = 3  ->  arr[3] = 3
        i = 0  ->  index = 2  ->  arr[2] = 1
        -> arr = [0, 0, 1, 3, 3]     得到 00133，个位跟个位才对得上
    ```

* **核心计算**

    ```java
    int[] sum = new int[len + 1];      // 结果数组，长度加一
    int num = 0;                       // 进位，初始值 0

    for (int i = arr1.length - 1; i >= 0; i--) {
        int temp = arr1[i] + arr2[i] + num;

        sum[i + 1] = temp % 10;        // 结果个位存入结果数组
        num = temp / 10;               // 结果十位存入进位
    }

    sum[0] = num;                      // 最高位可能的进位补到首位

    StringBuilder sb = new StringBuilder();
    if (sum[0] != 0) {
        sb.append(sum[0]);
    }
    for (int i = 1; i < sum.length; i++) {
        sb.append(sum[i]);
    }
    System.out.println(sb);
    ```

    * > **易错点**：存结果时必须写 `sum[i + 1]` 而不是 `sum[i]`，因为结果数组比两个加数数组长一位，索引 4 上的 `5` 与 `3` 相加得到的 `8` 要存在索引 5 上。

    * 循环必须倒着写 `for (int i = arr1.length - 1; i >= 0; i--)`，写成从 0 开始正着遍历就错了。

    * `sum[0] = num;` 这一步处理最高位进位，`99999 + 99999` 才能得到 `199998`；没有进位时 `num` 为 0，赋过去不影响结果。

    * 拼接前判断 `sum[0]` 是否为 0 是必要的，没有进位时首位是 0，拼接出来会多一个前导零；把判断反过来写成 `if (sum[0] != 0)` 就不会出现空分支占位，从索引 1 开始往后拼即可。

    * 打印三个数组时不必各写一个循环，可以直接调用工具类里的 `arrayToString`。

    * 核心代码里有三个循环，效率略低；全部写完、理解之后可以再思考能否把三个循环合成一个，效率能提升三倍。

## 课后作业

### 手机号与邮箱脱敏

* **手机号脱敏**

    * 把任意手机号中间四位变成星星，`13112345678` 变成 `131****5678`：前 3 位 + 四个星号 + 后 4 位拼接。

    ```java
    String phone = "13112345678";
    String masked = phone.substring(0, 3) + "****" + phone.substring(7);
    ```

    * 能这么写是因为手机号长度固定，中间四位的位置固定，索引可以写死。

* **邮箱脱敏**

    * 保留邮箱名的第一个字母，`@` 后面原样保留，`ZW1234@163.com` 变成 `Z***@163.com`。

    * 难点在于邮箱名长度不确定，`@` 的索引不确定，不知道从哪开始截、截到哪里，所以必须先定位再动手。

    ```java
    String email = "ZW1234@163.com";

    int at = email.indexOf("@");              // 关键：先定位 @ 的索引

    String masked = email.substring(0, 1)    // 邮箱名只保留第一个字母
            + "***"                           // 中间用星星盖住
            + email.substring(at);            // 从 @ 开始原样保留
    ```

    ```text
    ZW1234@163.com
          ^
          indexOf("@") = 6   —— 邮箱名多长都无所谓，索引一变，substring 自然跟着变
    ```

    * > **易错点**：`indexOf` 返回的是从 0 开始的下标，`ZW1234@163.com` 里 `@` 的下标是 6，保留后半段必须写 `substring(6)` 而不是 `substring(7)`，写成 7 会把 `@` 本身也切掉，结果成了 `Z***163.com`。

    * 两道小题的分野很清楚：手机号是位置固定、索引可写死，邮箱是位置不固定、必须先定位。

### 字符次数统计与最后一个单词

* **不区分大小写的字符计数**

    * 题目形态是计算某个字符出现的次数，判断时不区分大小写，输入里既有大写 `A` 又有小写 `a` 时找 `A` 的结果是两次。

    * 思路是利用 ASCII 码表：`'A'` 到 `'Z'` 与 `'a'` 到 `'z'` 是两段连续区间，同一个字母的大小写之间差的是一个固定偏移量。

    * 把两个字符都归一化到同一种大小写再比较，要么都 `toUpperCase()`，要么都 `toLowerCase()`，两边一致才不会漏掉另一种形态。

* **最后一个单词的长度**

    * 题目形态是计算最后一个单词的长度，`hello world` 的最后一个单词是 `world` 长度 5，另一个字符串的最后一个单词是 `move` 长度 4。

### 身份证信息提取

* **身份证规则**

    * 身份证号码有固定分段：第 1 到 6 位是地址码，第 7 到 14 位是出生日期，第 15 到 18 位是其余信息。

    ```text
    321104200801121234
    |     |      |   |
    1~6位  7~14位  15~18位
           20080112
                     ^ 第 17 位 = '3'  奇数 -> 男
    ```

* **出生年月日**

    * 出生日期占第 7 到 14 位，对应下标 6 到 13，分别截取年、月、日再拼接成 `2008年01月12日`。

    ```java
    String id = "321104200801121234";

    String year  = id.substring(6, 10);   // 2008
    String month = id.substring(10, 12);  // 01
    String day   = id.substring(12, 14);  // 12
    System.out.println(year + "年" + month + "月" + day + "日");
    ```

* **性别判断**

    * 倒数第二位也就是第 17 位表示性别，奇数是男性，偶数是女性。

    ```java
    char gender = id.charAt(16);       // 第 17 位，拿到的是字符 '3'
    int num = gender - '0';            // 先转成数字，再谈奇偶
    System.out.println(num % 2 == 1 ? "男" : "女");
    ```

    * > **易错点**：`charAt(16)` 拿回来的是字符 `'3'` 不是数字 `3`，字符不能直接参与计算；虽然 `char` 能隐式转成 `int`，但 `'3'` 本身的整数值是 51 不是 3，算出来照样是奇数纯属巧合蒙对；老老实实先做 `gender - '0'`，或者用 `Integer.parseInt(String.valueOf(gender))`。

### 整数前后加星

* **题目形态**

    * 把字符串中所有整数的前后都加上星星，其他字符不变，其中连续的数字视为一个整数。

    * 例如 `234`、`90`、`3` 这几个整数各自前后都要加星。

* **实现思路**

    * 直接在 `String` 本身上操作非常麻烦，改用 `StringBuilder` 的 `insert` 方法，它可以在指定下标处插入元素且不关心该位置原本是什么。

    * 配合「边遍历边记录连续数字段的起止下标」的思路，遇到整数就在它前后的下标处各插一次星号；用 `Character.isDigit()` 判断当前字符是不是数字即可扫出连续的一段数字。

### 随机验证码

* **题目形态**

    * 长度固定为 5，内容是四个字母加一个数字，字母大小写均可，数字可以出现在任意位置。

    * 正例有两种形态：四个字母后面直接跟数字，以及数字夹在字母中间；从头到尾没有数字、或者塞了两个数字都是错的。

* **拆解思路**

    * 复杂问题要学会拆成一步一步；可以先拆粗再拆细，每写完一步都要清楚这一步达成什么效果。

    * 总体拆两步：先生成一个初步的验证码（四个字母后面跟一个数字），再把末位数字与前面的某个随机位置做交换。

    * 「生成四个随机字母」再往下拆成四步：把随机内容放入数组、生成随机索引、通过索引获取随机元素、把过程重复四次并拼接。

    ```text
    char[52] 字母池（小a~小z + 大A~大Z）
            |
            |  第1~4步  随机索引抽 4 次，拼成 4 个字母
            v
       ABCD                          第5步：末尾拼一个随机数字
            |
            |  第5步  str += num
            v
       ABCDE                          末位（索引 4）是数字，位置还固定着
            |
            |  第6步  str.toCharArray()
            v
       [A][B][C][D][E]
            |
            |  第7~8步  随机索引 index 与末位元素交换
            v
       [A][B][E][C][D]                数字被甩到了随机位置
            |
            |  第9步  new String(array)
            v
       ABECD                          4 个字母 + 1 个数字，且数字位置随机
    ```

* **字母池的构造**

    * 不要手写罗列 52 个字母，要去发现规律：小写 26 个、大写 26 个，都可以用循环加偏移量填出来。

    ```java
    char[] arr = new char[52];

    for (int i = 0; i < 26; i++) {
        arr[i] = (char) ('a' + i);
    }
    for (int i = 0; i < 26; i++) {
        arr[i + 26] = (char) ('A' + i);
    }
    ```

    * 能这样写是因为小写 `a` 在 ASCII 码表里是 97，`i` 为 0 时就是 97，强转成字符放进索引 0；`i` 为 1 时是 98，转成字符就是 `b`，循环完小写 `a` 到 `z` 就都在数组里了。

    * 复制粘贴大写那段时要改两处：强转的字符从 `'a'` 改成 `'A'`，目标下标从 `arr[i]` 改成 `arr[i + 26]`。

    * > **易错点**：只改一个地方不够，大写会覆盖掉已经填好的小写，因为两段循环的索引范围重叠；添加大写时必须做位置偏移，偏移 26 个单位。

* **随机抽取与数字落位**

    ```java
    Random r = new Random();
    int index = r.nextInt(arr.length);   // 参数传数组长度，得到 [0, length) 内的随机索引
    char c = arr[index];                 // 通过随机的索引获取随机的元素

    int num = r.nextInt(10);             // 括号里写 10，在 0 ~ 9 之间获取随机数
    str += num;                          // 把数字拼接在 str 的后面

    char[] array = str.toCharArray();    // 字符串不可变，要改内容先转成字符数组
    // 用随机索引与最大索引上的数据做位置交换
    String result = new String(array);   // 再把字符数组转回字符串
    ```

    * 交换完把字符数组再转回字符串，转回来的就是符合要求的内容。

    * > **提示**：第 7 步生成随机索引时建议写成 `r.nextInt(array.length - 1)`，只在前面四个位置里挑——前面四个字母的位置在哪无所谓，只要数字的位置是随机的就行；如果索引抽到最后一位，就变成自己跟自己交换，等于啥也没干。

    * 遇到复杂问题不慌，先拆分、一步一步拆，拆到最后再整理，然后写代码；每写完一步都要清楚它达成什么效果，否则后面的步骤没法往下写。

### 乘积版超大数

* **题目形态**

    * 与超大数相加几乎相同，只是求的是乘积而不是和。

    * 相加时结果数组只要在原有基础上加一就可以，相乘则加一肯定不够。

    * > **结论**：这道题约 90% 的代码可以照搬，唯一不一样的就是如何计算第三个数组的长度。
