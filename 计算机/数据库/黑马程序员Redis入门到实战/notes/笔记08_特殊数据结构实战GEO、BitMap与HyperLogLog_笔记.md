# 特殊数据结构实战：GEO、BitMap 与 HyperLogLog

## 附近商户搜索的需求与接口

### 功能现状与目标

* **附近商户搜索**

    * > **定义**：围绕当前用户所在坐标，把附近的商户捞出来并按距离由近及远排序，一屏显示不下时继续翻页。

    * 首页顶部按类型分栏（美食、KTV 等），点哪个按钮就展示那一类商户的列表。

    * 排序口径默认是**按距离排序**：「附近」两个字本身就是在说距离，越近的排名越靠前。

    * 未完成的部分是排序：改造前商户能显示出来，但没有按距离排序，每个店铺旁也没有显示离我多远。

    * 后台拿到参数后要做三件事：按类型过滤、按页码分页、按经纬度排序，最后返回查询到的商户集合。

### 请求参数

* **商户类型查询请求**

    * > **定义**：请求方式为 GET、路径为 `/shop/type` 的商户列表查询，暗含「必须根据类型搜索」这一前置条件。

    * `typeId`：商户类型，用于过滤；点美食搜出来必须全是美食，不能混入其他类型。

    * `current`：页码。虽然界面上叫「滚动查询」，但它只是「滚一次多给一页」，属于**传统分页**，不是游标不断后推的那种滚动分页。

    * `x` 与 `y`：经纬度，即当前登录用户的坐标；真实项目由 APP 后台定位得到，演示时在前台写死。

    * 商户信息原本存在数据库中，现有实现只按 `typeId` 过滤再做分页，距离相关的能力完全没有。

### 数据库方案为何不够

* **关系型存储的边界**

    * > **定义**：商户表里存有 `typeId` 等完整字段，能过滤、能分页，但无法做地理位置的邻近搜索与距离排序。

    * 类型取值示例：`typeId` 为 1 的是美食相关店铺，为 2 的是 KTV 相关店铺。

    * 一旦需求扩展到「按地理坐标搜索附近商户」加「按距离排序」，数据库就做不到，必须把经纬度坐标导入 Redis 的 GEO 结构。

---

## GEO 的存储设计

### member 只放店铺 ID

* **GEO 的三元组**

    * > **定义**：GEO 存储时主要就三个参数——一个 member、一个经度、一个纬度，经纬度对应数据库表里的 `x`、`y`。

    * member 里**直接存店铺 ID**，不存整个店铺对象：Redis 是内存存储，把整个店铺信息塞进去空间占用太多。

    * 查询闭环：按经纬度筛选 → 拿到店铺 ID → 拿 ID 回数据库查店铺完整信息，效率上仍可接受。

### 按类型分组

* **以 key 承载类型过滤**

    * > **定义**：按 `typeId` 把商户分组，同类型商户作为一组，以它们的 `typeId` 作为 key 存到不同的 GEO 集合里。

    * 触发前提：写进 Redis 的只有坐标和店铺 ID，**根本没有 `typeId`**，所以存进去之后就没法再按类型过滤，只能在存的时候分开存。

    * key 的命名可以直接用 `typeId`，也可以叫「美食」「KTV」这类业务名；有几个类型就有几个 GEO key。

    * 判定效果：搜美食时直接取美食那个 key，该 key 下一定全部是美食相关店铺，**不需要再回数据库做类型过滤**。

```text
                    ┌──────────────────────────────────────┐
                    │        tb_shop（MySQL 里的商户表）      │
                    │  id │ typeId │ name │ x │ y │ ...     │
                    └───┬──────────────┬───────────────────┘
                        │ 按 typeId 分组  │
            ┌───────────┴───────────┐
            ▼                       ▼
      Map<Long, List<Shop>>    Map<Long, List<Shop>>
        key = 1（美食）           key = 2（KTV）
            │                       │
            ▼                       ▼
     ┌─────────────┐         ┌─────────────┐
     │  shop_geo_1 │         │  shop_geo_2 │
     │  member=店铺ID        │  member=店铺ID
     │  point=(x,y)          │  point=(x,y)
     └─────────────┘         └─────────────┘
          GEO 结构                GEO 结构
```

---

## 导入店铺数据到 GEO

### 导入流程

* **一次性数据搬运**

    * > **定义**：把商户坐标从数据库搬进 Redis GEO 的初始化动作，不是对外接口，因此不写 Controller，直接写单元测试方法完成。

    * 第一步查询店铺信息：数据量少时直接查所有；库里有几十万条时可以每次查 1000 条循环分批查。

    * 第二步按 `typeId` 分组：查到的店铺必须分组，同类型的放进同一集合，不同类型的放进不同集合。

    * 第三步分批写入 Redis：每个分组往 Redis 里存一个 GEO 集合。

    * 需要注入 `StringRedisTemplate` 才能往里写。

### 分组：Stream 的 groupingBy

* **分组依据**

    * > **定义**：用 Stream 的收集操作 `Collectors.groupingBy`，按 `Shop::getTypeId` 把店铺集合直接收成 `Map<Long, List<Shop>>`。

    * key 是 `typeId`（long 值），value 是同类型的店铺集合，这样多个分组天然可区分。

    * 方法引用可以进一步简写，比手写「遍历 + 判断 + 归入集合」更简洁。

```java
List<Shop> list = shopService.list();
// key 是 typeId，value 是同类型的店铺集合
Map<Long, List<Shop>> map = list.stream()
        .collect(Collectors.groupingBy(Shop::getTypeId));
```

### 逐条写入与批量写入

* **逐条 GEOADD**

    * > **定义**：遍历店铺集合，对每个店铺调用一次 `opsForGeo().add(key, new Point(x, y), id.toString())`。

    * `Point` 就是地图上的一个点，Redis 里用它承载经纬度。

    * 边界与代价：每个店铺都要发一个请求，1000 个点就是 1000 个请求，效率较低。

* **批量 GEOADD**

    * > **定义**：把店铺先封装成 `RedisGeoCommands.GeoLocation<String>` 集合，再一次 `add(key, locations)` 写入。

    * `GeoLocation` 里装的就是一个 point 加一个 member，等于把「点」和「member」打包成一个对象。

    * `GeoLocation` 集合的大小与店铺集合的大小一致。

```java
// 构造一个 GeoLocation 集合
List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>();
for (Shop shop : shops) {
    locations.add(new RedisGeoCommands.GeoLocation<>(
            shop.getId().toString(),
            new Point(shop.getX(), shop.getY())
    ));
}
// 批量写入
stringRedisTemplate.opsForGeo().add("shop_geo_" + typeId, locations);
```

    * > **易错点**：`Point` 要导 `org.springframework.data.geo.Point` 这个包，别导错。

### 导入后的验证

* **key 的数量由类型数决定**

    * > **定义**：导入完成后 Redis 里出现的 GEO key 个数，等于数据库里商户类型的个数。

    * 示例：只有美食和 KTV 两种类型时，就只会出现两个 key，值与经纬度一一对应。

    * > **提示**：测试时把 key 写成硬编码字符串并不专业，正式代码应在常量类里加一个 `SHOP_GEO_KEY`，再拼上 `typeId`。

---

## GEOSEARCH 的版本与依赖

### 版本冲突

* **旧版客户端不认识新命令**

    * > **定义**：项目使用的 Spring Boot 不是最新版本，其内置的 Spring Data Redis 是 2.3.9，不支持 Redis 6.2 提供的 `GEOSEARCH` 命令。

    * 取舍：老命令也能用，但既然已经用了新版本 Redis，就应当用 `GEOSEARCH`。

### 依赖调整

* **排除再手动引入**

    * > **定义**：在 `spring-boot-starter-data-redis` 上用 `exclusions` 排除掉内置的 spring-data-redis 与 lettuce-core，再手动引入新版本。

    * 手动引入的版本：spring-data-redis 2.6.2、lettuce-core 6.1.6。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <!-- 排除 Spring Boot 默认引入的低版本 -->
        <exclusion>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
        </exclusion>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 手动引入支持 GEOSEARCH 的新版本 -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.6.2</version>
</dependency>
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>6.1.6</version>
</dependency>
```

### 接口参数

* **经纬度参数必须可缺省**

    * > **定义**：`x`、`y` 声明为 `Double` 且标注 `required = false`，表示可有可无。

    * 触发前提：前端不一定按地理坐标查询和排序，也可能按人气、评分等维度，所以这两个参数有可能为空。

    * 分支判定：没传就按数据库查，传了就按 Redis 里 GEO 的形式查，两种处理方案并存。

```java
@GetMapping("/type")
public Result typeSearch(@RequestParam("typeId") Long typeId,
                         @RequestParam("current") Integer current,
                         @RequestParam(value = "x", required = false) Double x,
                         @RequestParam(value = "y", required = false) Double y) {
    return shopService.queryShopByType(typeId, current, x, y);
}
```

---

## 附近商户查询的实现

### 分页参数

* **from 与 end**

    * > **定义**：只有页码不够，必须由页码和每页大小算出「从哪开始查、查到哪结束」两个下标。

    * `from = (current - 1) * size`，`end = current * size`。

    * 默认每页大小取 5，即每次查 5 条。

```java
int from = (current - 1) * DEFAULT_PAGE_SIZE;
int end = current * DEFAULT_PAGE_SIZE;
```

### GEOSEARCH 的参数

* **四个入参**

    * > **定义**：`opsForGeo().search` 依次需要 key、圆心参照、半径、搜索参数四样东西。

    * key：按类型取，等于常量 `SHOP_GEO_KEY` 拼上 `typeId`；存时按类型存，取时也按类型取。

    * 圆心（reference）：用 `GeoReference.fromCoordinate(x, y)` 传入经纬度；同一族还有 `fromCircle`、`fromMember`（以 GEO 里某个成员为圆心）。

    * 半径（`Distance`）：`new Distance(5000, Metrics.METERS)`；只给数值时默认单位是米，也可以用 `metric` 指定千米等单位。

    * 搜索参数（`GeoSearchCommandArgs`）：`includeDistance()` 让结果带上距离（即 withDistance），`sort(Sort.ASC)` 指定升序，末尾 `.limit(end)` 做截断。

    * > **注意**：半径用米作单位，将来搜索结果的单位也是米，要按需求选择。

```java
GeoSearchCommandArgs args = GeoSearchCommandArgs.newGeoSearchCommandArgs()
        .includeDistance()
        .sort(GeoSearchCommandArgs.Sort.ASC);

GeoResults<RedisGeoCommands.GeoLocation<String>> results = stringRedisTemplate.opsForGeo()
        .search(
                SHOP_GEO_KEY + typeId,
                GeoReference.fromCoordinate(x, y),
                new Distance(5000, Metrics.METERS),
                args.limit(end)
        );
```

### 逻辑分页

* **limit 给不了起点**

    * > **定义**：`limit` 只能指定一个 `count`（相当于 `end`），起点永远是第一条，因此只能自己截取 from 之后的部分，属于逻辑分页。

    * 判定方式：指定 5 就返回 0 到 5，指定 15 就返回 0 到 15，`from` 无法下发。

    * 截取手法：用 Stream 的 `skip(from)`，因为它只是跳过、不需要真的拷贝集合，更节省内存；也可以用 `subList`。

    * `results` 有可能为 null，为 null 时直接返回空集合，不再往下走。

```text
  GEOSEARCH 返回的结果（下标从 0 开始）
  ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
  │  0   │  1   │  2   │  3   │  4   │  5   │  6   │  7   │ ...
  └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
    └──────────── 本次要返回 ────────────┘
       skip(from) 之后的元素

  current = 1 → from = 0  → 跳过 0 条 → 0~end
  current = 2 → from = 5  → 跳过 5 条 → 5~end
  current = 3 → from = 10 → 跳过 10 条 → 10~end（可能啥也没有）
```

### 结果与距离解析

* **GeoResult 的两件东西**

    * > **定义**：单个结果是 `GeoResult`，里面有 `content` 和 `distance` 两部分。

    * `content` 是 `GeoLocation`，它的 `getName()` 就是 member，也就是店铺 ID（字符串）。

    * `getDistance()` 拿到距离对象，再 `getValue()` 转成 double。

    * 收集两个容器：`List<Long> ids` 装店铺 ID，`Map<String, Distance> distanceMap` 以 ID 字符串为 key、Distance 为 value，用来和店铺一一对应。

    * 批量查回店铺不能用 `listByIds`（不保证顺序），要用带 `in` 与 `order by` 的 query 方式，ID 串用 `String.join(",", ids)` 拼出来。

    * 店铺实体里有一个 `distance` 成员变量，它不是数据库字段、加了 `@TableField(exist = false)`，专门用来把距离返回给前端。

```java
// 1. 先解析出 ids 和 distanceMap
List<Long> ids = new ArrayList<>(results.getContent().size());
Map<String, Distance> distanceMap = new HashMap<>(results.getContent().size());
for (GeoResult<RedisGeoCommands.GeoLocation<String>> r :
        results.getContent().stream().skip(from).collect(Collectors.toList())) {
    RedisGeoCommands.GeoLocation<String> content = r.getContent();
    ids.add(Long.valueOf(content.getName()));
    distanceMap.put(content.getName(), r.getDistance());
}

// 2. 批量查回店铺
List<Shop> shops = queryShopByIds(String.join(",", ids));

// 3. 把距离塞回店铺
for (Shop shop : shops) {
    shop.setDistance(distanceMap.get(shop.getId().toString()).getValue());
}
```

### 空页异常

* **跳过之后可能什么都不剩**

    * > **定义**：查第三页时 SQL 报语法错误，原因是 `where id in ()` 里的 ID 集合为空。

    * 触发条件：Redis 确实查到了数据，但 `skip(from)` 把这些数据全跳过去了；例如总共只有 9 条，第三页要 `skip(10)`，跳完就没有元素了。

    * > **易错点**：判断 `results != null` 只能保证「Redis 查到了东西」，不能保证「跳过 from 之后还剩东西」。

    * 兜底规则：`ids.isEmpty() || ids.size() <= from` 时说明没有下一页，直接返回空集合。

```java
if (results == null) {
    return Result.ok(Collections.emptyList());
}

// ...解析 ids 与 distanceMap...

// 跳完就没有了，说明没有下一页，直接返回空集合
if (ids.isEmpty() || ids.size() <= from) {
    return Result.ok(Collections.emptyList());
}
```

* **排序结果的读法**

    * > **定义**：展示出来的距离值带小数精度，看起来相同的两条其实因为小数点后被省略而存在先后。

    * 相邻两次翻页不会重复：第一次取到第 5 条为止，第二次从偏移 10 的位置继续，说明分页生效。

---

## BitMap 与签到

### 关系型签到表的成本

* **一行一签到**

    * > **定义**：最直觉的方案是建一张签到表，表里每一行代表某一个用户某一天的签到记录。

    * 基本字段：`id` 主键、`user_id` 签到用户、`year_month` 签到的年与月（留作统计）、`date` 真正的签到日期（判断某天有没有签到就看它）、`is_backup` 是否补签。

    * > **易错点**：`is_backup` 字段一定要有；有些应用允许补签，正常签到与补签是两码事，不标记出来后面无法区分。

    * 数据量估算：1000 万用户、保守假设每人每年签到不超过 10 次，一年就是 1 亿条，真实情况往往更多。

    * 单行字节数：两个 bigint 各 8 字节，年 1 字节，月 1 字节，日期 3 字节，补签标记 1 字节，合计 22 字节。

    * > **结论**：一个用户一天的签到就要 22 字节，一个月六百多字节，1000 万用户一年下来是天文数字，这种设计不可行。

### BitMap 的本质

* **BitMap（位图）**

    * > **定义**：BitMap 的核心思想是把每一个 bit（比特位）与某种业务状态做映射，用 0/1 表示业务状态，用 bit 的下标表示业务里的某个序号。

    * 映射约定：1 表示已签到，0 表示未签到；从一个月的第 1 天开始依次记成一串 0/1，第一位代表第 1 天，最后一位代表当月最后一天。

    * 空间账：一个月最多 31 天，31 个比特位撑死 4 个字节，与数据库方案「一天 22 字节」相差几百倍。

    * 底层实现：Redis 用 String 类型实现 BitMap，因为 String 底层存的就是字节，一个字节就是 8 个比特位。

    * 容量边界：String 最大存储上限 512 MB，换算成比特位是 $2^{32}$ 个，签到一个月只需 31 个比特位，绰绰有余。

    * 同类应用：布隆过滤器的底层同样是利用 BitMap 实现的。

    * > **结论**：把用户标识与某年某月拼在一起作为 key 存这一个月的签到情况，既省内存又天然按月聚合，按月统计特别方便。

```text
day :  1  2  3  4  5  6  7  8
bit :  0  1  2  3  4  5  6  7
val :  1  1  1  0  0  0  1  1     →  11100111
```

### BitMap 常用命令

* **命令与用途**

| 命令 | 作用 | 签到场景里拿来干什么 |
| :--- | :--- | :--- |
| `SETBIT key offset value` | 给指定位置的比特位存入 0 或 1 | 签到 |
| `GETBIT key offset` | 取出指定位置的比特位 | 判断某一天签没签 |
| `BITCOUNT key` | 统计 BitMap 里值为 1 的比特位个数 | 本月一共签了几天 |
| `BITFIELD key GET u<位数> <offset>` | 一次读多个比特位，返回十进制 | 拿整段签到记录去统计 |
| `BITFIELD_RO key ...` | 只读版本，没有写和自增功能 | 纯查询时更省心 |
| `BITOP` | 多个 BitMap 做与、或、异或等位运算 | 用不上 |
| `BITPOS key 0\|1 [start] [end]` | 查找指定范围内第一个 0 或 1 出现的位置 | 查这个月第一次缺勤在几号 |

* **命令的行为细节**

    * > **定义**：`SETBIT` 的 offset 是下标、从 0 开始，0 代表当月第 1 天，30 代表当月第 31 天。

    * `BITFIELD` 兼具查询、修改、自增三种能力，内部参数特别长、特别复杂；真要改某个位置的值直接用 `SETBIT` 指定索引改即可，所以 `BITFIELD` 基本只用来查询，且用只读版 `BITFIELD_RO`。

    * `BITFIELD_RO` 的 RO 即 read only，唯一区别是不具备修改和自增，只有查询；查询结果最终以十进制形式返回，因为二进制长了可读性太差。

    * `BITPOS` 是 position 的意思，做查找；一个月的签到记录里既有 0 也有 1，查第一个 0 出现的位置就是查这个月第一次缺勤在第几天。

    * 真正实现签到一个 `SETBIT` 就够了；`GETBIT` 拿某天有没有签到；要一次拿到完整签到结果才需要 `BITFIELD`；统计总次数用 `BITCOUNT`。

### 控制台中的一次完整演练

* **写入与查看**

    * > **定义**：key 为 `bm1`，从 offset 0 开始逐天 `SETBIT`，签到的天写 1，没签到的天不用管——默认就是 0。

    * 示例：1、2、3 号签到，4、5、6 号没签，7、8 号又签，用 binary 形式查看就是 `11100111`，总共签了 6 天。

    * > **易错点**：Redis 桌面客户端要用较新的版本（2022 年之后的）或 2020 年以前的旧版本；中间那些 2021 的版本展示二进制数据时不支持 binary 形式，选了也什么都看不见。

* **读取与统计**

    * > **定义**：`GETBIT bm1 2` 取第 3 天（下标 2）的状态，返回 1 说明签到了；`BITCOUNT bm1` 返回整月签到总次数。

    * `BITFIELD bm1 GET u2 0` 要读懂三样东西：`GET` 是查询子命令；`u2` 中 `u` 表示结果按无符号解读、`2` 是要读取的比特位个数；末尾的 `0` 是起始 offset。

    * 参数之所以叫 type 而不是「数量」，是因为除了数量还得指定返回结果有符号还是无符号：`u` 是无符号（unsigned），`i` 是有符号（int），一般用无符号。

```bash
BITFIELD bm1 GET u2 0    # 从第 0 位读 2 位，11b = 3
BITFIELD bm1 GET u3 0    # 从第 0 位读 3 位，111b = 4+2+1 = 7
BITFIELD bm1 GET u4 0    # 从第 0 位读 4 位，1111b = 8+4+2+1 = 15
```

    * `BITPOS bm1 0` 返回 3，即这个月第一次缺勤在 4 号；`BITPOS bm1 1` 返回 0。

### 签到接口

* **无参签到**

    * > **定义**：`POST /user/sign`，请求参数无、返回值无，把当前用户当天的签到信息保存到 Redis 中。

    * 为什么无参：需求写的是「当前用户」「当天」，用户信息与年月日都由代码直接获取，不需要前端传。

    * > **结论**：签到接口是无参的；将来如果要实现补签之类的功能，那时候再让前端传日期。

    * > **易错点**：BitMap 底层是 String 实现的，Spring Data Redis 把 BitMap 的操作一起封装进了字符串操作里，没有 `opsForBitMap()`，必须拿 `opsForValue()`。

* **实现的五个步骤**

    * > **定义**：获取当前登录用户 → 获取当前时间（年 + 月）→ 两部分拼成 key → 取今天是本月的第几天作为 offset → `SETBIT` 写进去。

    * key 结构：前缀 + 用户 ID + 年月，形如 `sign:5:202203`，即 5 号用户在 2022 年 3 月的签到记录。

    * 年月串由 `DateTimeFormatter.ofPattern("yyyyMM")` 从当前时间格式化得到；中间是否加横杠可自行取舍。

    * 前缀标识（如 `sign:`）应放进常量类，不要硬编码。

    * > **易错点**：`LocalDateTime.now().getDayOfMonth()` 返回的是 1 ~ 31 的**日期**，而 offset 是 0 ~ 30 的**下标**，两者差一，必须 `dayOfMonth - 1`。

    * value 取值：签到写 1、不签到写 0；Java 里为节省空间用 boolean 值，`true` 就是 1。

```java
@Override
public Result sign() {
    // 第一步：获取当前登录用户
    Long userId = UserHolder.getUser().getId();

    // 第二步：获取当前时间
    LocalDateTime now = LocalDateTime.now();

    // 第三步：拼接 key：前缀 + 用户 id + 年月
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyyMM");
    String key = "sign:" + userId + ":" + now.format(formatter);

    // 第四步：今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();

    // 第五步：写入 Redis，offset = dayOfMonth - 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);

    return Result.ok();
}
```

### 签到写入的验证

* **落位与补零**

    * > **定义**：调用接口后 Redis 里出现对应 key，转成 binary 形式才能看到比特位。

    * 落位判定：14 号签到时，1 出现在第 13 个位置上，而不是第一个位置。

    * > **提示**：Redis 底层是以**字节**为单位存储的，不是以 bit 为单位，一个字节 8 个比特位；写第 14 个比特位时它仍在两个字节的范围之内，所以占用两个字节，后面用不完的位补零。

    * 手动补签：对同一个 key 执行 `SETBIT sign:5:202203 12 1`，12 号（offset 12）就变成 1，等价于补签。

---

## 连续签到统计

### 连续签到的定义

* **连续签到次数**

    * > **定义**：从最后一次签到开始向前统计，直到遇到第一次未签到为止，这中间的总签到次数就是连续签到次数。

    * 统计口径分为两类：本月总共签到多少次，以及本月截止今天为止连续签到多少次；总次数一个 `BITCOUNT` 就能得到，连续签到则复杂得多。

    * 程序化步骤：拿到本月截止今天为止的所有签到数据 → 从最后一个比特位开始逐个向前遍历 → 每到一个位判断签到与否 → 遇到 0 就结束 → 用计数器累加。

```text
day :  1  2  3  4  5  6  7  8  9  10
val :  1  1  1  0  0  1  1  1  0   1
                                  ↑
                     从这里（最后一次）开始向前
                                ←←←←←←←←←←←
        数到第一个 0 就停 → 共连续签到 4 次
```

### 取本月截止今天的数据

* **BITFIELD 的两个关键参数**

    * > **定义**：`BITFIELD` 取数时要指定「从哪儿开始」（起始 offset）和「查多少」（比特位个数）。

    * `GETBIT` 一次只能查一个比特位，所以要一次拿多个比特位必须用 `BITFIELD`。

    * > **易错点**：`BITFIELD` 里「查多少」是**比特位的个数**，不是结束位置的下标，这个地方特别容易搞混。

    * 起始 offset 一定是 0（第一天对应下标 0）；个数等于今天是几号——一个比特位代表一天，1 号到 10 号就是 10 个比特位。

```bash
BITFIELD sign:5:202203 GET u14 0
#                  ↑  ↑   ↑
#                  |  |   └─ 从第 0 位开始读
#                  |  └───── 从第 0 位开始读 14 个比特位（无符号）
#                  └──────── GET 查询子命令
```

    * 返回的是一个十进制数字（示例为 `14339`），需要在 Java 里逐位拆解。

### 从十进制倒着取比特位

* **与 1 做与运算**

    * > **定义**：任何数字与 1 做与运算，结果就是它的最低比特位；最低位 & 1 = 该比特位本身。

    * 机理：数字 1 只在最低位上为 1、其余位全为 0，所以参与运算时只有最低位活了下来，拿到的正是最后一天的签到状态。

    * > **结论**：`number & 1` 就是取 `number` 的最低比特位，也就是最后一天的签到状态；结果为 0 说明今天没签到，直接结束。

* **右移一位取前一个比特位**

    * > **定义**：把数字右移一位（`>>>`），整体往右挪，超出的最后一位被抛弃，倒数第二位就变成了最后一位。

    * 反复「与 1 取值 → 右移一位」就实现了从后往前的逐位遍历。

```text
   1110 0000 0000 1100      ← 一段签到记录（高位在左，低位在右）
                      ↑
   最低位 & 1 = 该比特位本身   → 拿到了最后一个比特位
   最低位 & 1 = 0              → 说明今天没签到，结束

   1110 0000 0000 1100
                  ╱  右移 1 位（>>>）
   0111 0000 0000 0110
   现在再做一次 & 1，拿到的是倒数第二位
   再右移一位 → 倒数第三位
```

### 统计实现与验证

* **接口与 API**

    * > **定义**：`GET /user/sign/count`，请求参数无，返回值为连续签到天数。

    * 取用户、取日期、拼 key、取本月是几号这一段与签到接口完全相同。

    * 子命令构造：`BitFieldSubCommands.create(BitFieldType.unsigned(dayOfMonth), BitFieldOffset.from(0))`，工厂方法是静态的 `create(...)`，不用自己 new。

    * `BitFieldType.unsigned(位数)` 表示无符号类型，位数传「今天是几号」；`BitFieldOffset.from(0)` 指定从第 0 位开始读。

    * `bitField` 返回的是集合，因为 `BITFIELD` 一次可以同时做多个子命令（GET、SET、INCRBY），也可能一个都没拿到，所以要先判空。

    * 只传了一个 `GET` 时结果一定只有一个，`get(0)` 取出 number，再判一次空、判一次 0，为 0 直接返回 0。

```java
@Override
public Result signCount() {
    Long userId = UserHolder.getUser().getId();
    LocalDateTime now = LocalDateTime.now();
    String key = "sign:" + userId + ":" + now.format(DateTimeFormatter.ofPattern("yyyyMM"));
    int dayOfMonth = now.getDayOfMonth();

    // 1. 用 BITFIELD 取本月截止今天为止的所有签到记录
    List<Long> result = stringRedisTemplate.opsForValue().bitField(
            key,
            BitFieldSubCommands.create(
                    BitFieldType.unsigned(dayOfMonth),   // 今天是几号就取几位
                    BitFieldOffset.from(0)));            // 从第 0 位开始读

    // 2. 没查到任何结果，直接返回 0
    if (result == null || result.isEmpty()) {
        return Result.ok(0);
    }

    // 只传了一个 GET，所以结果一定只有一个，取出来就是我们要的十进制数
    Long number = result.get(0);
    if (number == null || number == 0) {
        return Result.ok(0);
    }

    // 3. 循环遍历比特位
    int count = 0;
    while (true) {
        // 与 1 做与运算拿到最低比特位，顺便判断是不是 0
        if ((number & 1) == 0) {
            // 为 0 说明未签到，找到断签了，结束
            break;
        }
        // 不为 0 说明已签到，计数器加一
        count++;
        // 把最后一位抛弃掉，才能判断下一个比特位
        number >>>= 1;
    }

    return Result.ok(count);
}
```

    * > **易错点**：循环里最后那句 `number >>>= 1` 必须写；先右移一位再赋值给 number 才能把最后一位覆盖抛弃掉，不做这一步就永远在判断最后一个比特位，直接死循环。

* **验证结论**

    * > **定义**：从今天往前数，连续签了几天就返回几。

    * 用 `SETBIT` 补签前面的日期后，连续签到天数随之增加（示例中由 2 天变 4 天）。

    * 把今天对应的比特位置为 0 后再查，循环一上来判到的就是 0，直接 break，返回 0，连续签到就此中断。

---

## HyperLogLog

### UV 与 PV

* **UV（unique visitor，独立访客量）**

    * > **定义**：一个人通过互联网访问、浏览网页，在一天内多次访问只记录一次，因此叫「独立」「唯一」。

    * 用途：通过 UV 能看出网站的用户访问量是多少。

* **PV（page view，页面访问量）**

    * > **定义**：用户每访问一个页面就记一次，也叫页面点击量；在同一页面上反复刷新会被记很多次。

    * 与 UV 的关系：PV 里面有很多重复值，往往比 UV 大很多。

* **PV 与 UV 的比值**

    * > **定义**：用 PV 除以 UV，衡量每个用户进来是点一下就走还是访问了很多次，即用户粘度。

    * 判定方式：靠小广告弹窗吸引点击的网站，用户点进去就走；内容好的网站用户会流连忘返，PV 与 UV 的差距越来越大。

    * 边界：这个值不一定准确，但作为估算有一定价值。

### 统计 UV 的难点

* **去重必须先存下来**

    * > **定义**：独立访客要求一个用户只记录一次，所以每次访问都要先判断该用户是否已被统计过——没统计过就计数器加一并记进「已访问集合」，已记过就什么都不做。

    * 代价：千万级用户就要往 Redis 里存上千万条数据，上亿就更不用说，把用户信息直接写入 Redis 显然不理想。

### HyperLogLog 的本质

* **HyperLogLog（HLL）**

    * > **定义**：HyperLogLog 是从 log 算法派生的一种概率算法，用于确定非常大集合的基数（也就是去重之后的数量）；可以理解为基于一些基本数据的统计去推算出一个总数量。

    * 底层实现：基于 String 接口实现。

    * 内存边界：单个 HLL 的内存**永远小于 16KB**，不管统计数百万还是数亿用户，内存占用都不会高于这个值。

    * 精度边界：测量结果不是百分之百精准，误差大概在 0.81%；一万用户的误差也就几十个人。

    * 适用前提：能容忍这个误差才用，容忍不了就不要用。

### 三个命令

* **PFADD**

    * > **定义**：插入元素，`PFADD key element [element ...]`，把用户 ID 作为 element 插进去，有一亿个用户就 add 一亿个用户 ID。

* **PFCOUNT**

    * > **定义**：统计基数，`PFCOUNT key [key ...]`；因为基于概率，塞进去的元素本质上并没有被真正保存下来，算出来的不一定是准确值。

* **PFMERGE**

    * > **定义**：合并多个 HLL，`PFMERGE destkey sourcekey [sourcekey ...]`，合并完统一再做计数。

    * 用法：把每天的 UV 做成一个 key，想统计一个月的 UV 就把这 30 多个 key 合并；想统计一年就把这一年里每一天的 key 统一合并。

    * 边界：即便把一整年的 key 都合并了，内存大小依然小于 16KB。

* **去重行为**

    * > **定义**：往同一个 key 里重复加入相同元素，`PFCOUNT` 的结果不变。

    * 示例：加入 5 个不同元素后 count 为 5；再次加入同样的 5 个元素，count 仍是 5。

    * > **结论**：HyperLogLog 天生就适合做唯一性统计，也就是 UV 统计；数量比较小的情况下它还能准确统计，数据量大了就不一定。

### 百万数据压测

* **测试前的准备**

    * > **定义**：先用 `INFO memory` 记下内存基线，里面上面那个值以字节为单位、下面那个以 MB 为单位。

    * API 对应关系：`opsForHyperLogLog()` 的 `add` 等同 `PFADD` 且可一次指定多个元素做批量插入，`size` 统计元素数量，`delete` 直接删掉，`union` 等同 merge 做多个 key 的合并。

* **分批插入的两个坑**

    * > **定义**：`add` 的 value 是可变参数（即数组），所以先准备一个大小为 1000 的 `String` 数组，循环 1000 次、1000 次地往里插。

    * 第一个坑是数组越界：循环变量 `i` 不断增加，直接 `values[i]` 必然越界，所以要另取 `j = i % 1000`，模完取值范围恒为 0 到 999，永远不会超出数组范围。

    * 第二个坑是发送时机：`j == 999` 表示数组已填满（从 0 到 999），此时才执行一次 `add`；下一轮 `i = 1000` 再模 1000 又变回 0，如此循环往复插满 100 万条。

```java
@Test
void testHyperLogLog() {
    String[] values = new String[1000];
    for (int i = 0; i < 1000000; i++) {
        // 数组只有 1000 个坑，角标必须循环使用
        int j = i % 1000;
        values[j] = "user" + i;
        // 从 0 数到 999，刚好填满，发一次
        if (j == 999) {
            stringTemplate.opsForHyperLogLog().add("hl2", values);
        }
    }
    Long count = stringTemplate.opsForHyperLogLog().size("hl2");
    System.out.println(count);
}
```

* **压测结论**

    * > **定义**：插入 100 万条后统计结果为 997593，与 100 万相差 2400 多条。

    * 误差量级：997593 / 1000000 ≈ 0.9976，误差约在 0.002 这个量级，对 UV 统计不算什么。

    * 内存占用：用压测后的 `INFO memory` 值减去基线值再除以 1024，结果大约 14KB，确实没有超过 16KB。

    * 重复插入的影响：再插入 100 万条重复数据，因为名字重复，最终统计结果不会有太大差别。

    * > **结论**：真要做 UV 统计，只需把用户信息不断往里塞即可，不用关心重不重复、有没有存在，重复由 HyperLogLog 处理。

---

## 单节点 Redis 的四个问题

### 问题与解法

* **单节点部署的四道坎**

| 单节点的问题 | 具体表现 | 解决方案 |
| :--- | :--- | :--- |
| 数据丢失 | 内存存储，宕机重启数据就没了 | RDB 持久化，把数据写入磁盘 |
| 并发能力不足 | 单节点三五万 QPS，大促数十万甚至上百万 | 主从集群：多个从节点负载均衡 + 读写分离 |
| 故障恢复 | 一个节点挂了，服务就整体不可用 | 哨兵机制：监测各节点健康状态，自动故障恢复 |
| 存储能力 | 内存容量有上限，撑不住海量数据 | 分片集群：利用插槽机制分片，可动态扩容 |

* **数据丢失**

    * > **定义**：Redis 是内存存储，性能因此更高，但一旦服务宕机、重启，数据就丢失了。

* **并发能力**

    * > **定义**：单节点的并发能力大约在三万、四万、五万这个量级，而 618、双 11 这类电商场景的并发往往达到数十万甚至上百万。

* **故障恢复**

    * > **定义**：集群中任意一个服务出现故障都不能影响其他服务，必须能在运行过程中修复故障节点，实现边运行边修复。

    * 边界：单节点挂了就是挂了，做不到这一点。

* **存储能力**

    * > **定义**：内存存储与磁盘存储不在一个数量级，磁盘能存非常多数据，内存却有上限，而需要缓存的数据越来越多。

### 主从与哨兵

* **主从集群**

    * > **定义**：Redis 内部的一种集群结构，从节点可以有很多个，多个从节点之间构成负载均衡；主从之间做读写分离。

    * 读写分离应对的是读写之间的互斥，读和写互不影响，并发能力自然更强。

    * 高可用效果：主宕机了还有从顶上去。

    * 边界：人再多，如果一直挂下去总有一天会全挂完，所以还需要故障恢复能力。

* **哨兵机制**

    * > **定义**：哨兵不断监测整个集群中每个节点的健康状态，一旦发现有人挂了就自动做故障恢复，把他扶起来。

    * 效果：整个 Redis 集群可以实现真正的高可用、高并发。

### 存储能力与分片

* **分片集群**

    * > **定义**：参考 Elasticsearch 把数据分片保存到不同节点的做法，Redis 也可以搭建分片集群，利用插槽机制把数据分散。

    * 触发前提：主从集群各节点里的数据是一样的，存储上限仍然是单个节点的内存上限。

    * 动态扩容：数据越来越多就加机器，理论上存储能力没有上限。

---

## RDB 持久化

### RDB 的定义

* **RDB（Redis Database Backup File）**

    * > **定义**：RDB 全称 Redis Database Backup File，即 Redis 的数据备份文件，也叫数据快照。

    * 原理：把内存的数据拷贝一份写到磁盘上，这份磁盘中的数据就是备份或快照；Redis 重启或发生故障时都可以从快照里读取，完成数据恢复。

    * 保存位置：快照文件（RDB 文件）默认保存在当前运行目录，也就是在哪里运行 Redis 就保存在哪里。

### save 与 bgsave

* **SAVE**

    * > **定义**：用 `redis-cli` 连上 Redis 后执行 `save`，由 Redis 主进程去执行 RDB 备份。

    * 阻塞代价：Redis 是单线程的，主进程执行 RDB 时无法执行其他动作，用户的查询、新增都做不了；RDB 要写磁盘、IO 较慢，数据量大时耗时很久，直到返回 `OK` 主进程才能处理其他请求。

    * 适用场景：不推荐日常使用，适合在 Redis 进程马上要停止、要停机的时候用。

* **BGSAVE**

    * > **定义**：执行时立即返回 `Background saving started`，保存动作由一个额外的进程在后台异步执行，不占用主进程。

    * 效果：主进程不受影响，该接受命令照样接受、照样处理。

    * 适用场景：适合在 Redis 运行过程中做。

    * > **注意**：`SAVE` 适合在服务**停机之前**——不是宕机（宕机是突然的、无法控制），主动停机时 Redis 会自动进行一次 RDB。

### 优雅停机

* **停机前的最后一次保存**

    * > **定义**：Ctrl+C 停机时日志里出现 `Saving the final RDB file before exit`，即退出之前做一次 RDB 保存，这种停机方式叫优雅停机。

    * 恢复验证：运行目录下出现 `dump.rdb` 文件，再次启动 Redis 后数据自动恢复，`get` 仍能拿到之前写入的值。

    * 边界：默认其实就有持久化，但它只在停机那一刻执行；服务运行一个月后突然宕机，没来得及持久化，数据就全丢了，所以更希望每隔一段时间备份一次。

### 触发规则与配置

* **默认触发规则**

| 配置 | 含义 |
| :--- | :--- |
| `save 900 1` | 900 秒内，至少有 1 次修改，则执行 BGSAVE |
| `save 300 10` | 300 秒内，至少有 10 次修改，则执行 BGSAVE |
| `save 60 10000` | 60 秒内，至少有 1 万次修改，则执行 BGSAVE |

* **文件与压缩配置**

    * > **定义**：`dir` 配置的值是 `.`（当前目录），所以 RDB 文件默认保存在当前运行目录；`dbfilename` 是文件名（默认 `dump.rdb`）；`rdbcompression` 表示保存时要不要压缩，默认 `yes`。

    * 压缩的取舍：32G 内存不压缩时磁盘上也是 32G，压缩后体积变小、节省磁盘，但压缩过程消耗 CPU 资源、对 CPU 压力较大。

    * > **结论**：不推荐开启压缩——磁盘不值钱，多带点磁盘即可，消耗 CPU 更严重；当然前提是 CPU 资源紧张，CPU 资源充足时完全可以采用压缩模式。

    * > **易错点**：配置 `save` 规则时行首不要加 `#`，加 `#` 代表注释。

* **改名带来的坑**

    * > **定义**：把 `dbfilename` 改成别的名字后，当前目录里没有同名文件，再次启动 Redis 时旧数据不会被读取、也就无法恢复。

    * 现象：启动日志里没有恢复数据的记录，连接后 `get` 返回 `(nil)`。

    * 触发验证：改名后再随便 `set` 一次，日志立即出现 `1 changes in 5 seconds. Saving...` 并走 `background saving`，说明自动触发了 BGSAVE。

* **间隔时间的取舍**

    * > **定义**：RDB 的时间间隔配得太长，两次持久化之间的写入一旦宕机就会丢失；配得太短（例如一秒一次）则频率过高。

    * 频率过高的代价：数据量达到 1G 甚至 10G 时，把这么多数据写入磁盘要花很久，一秒执行一次根本忙不过来。

    * 建议：一般情况下按默认即可，例如 30 秒或 60 秒；真在这期间宕机丢了数据，后续还有其他持久化方案来弥补。

### fork 与写时复制

* **页表与虚拟内存**

    * > **定义**：Linux 中所有进程都不能直接操作物理内存，操作系统给每个进程分配虚拟内存，进程只操作虚拟内存，由操作系统维护虚拟内存与物理内存之间的映射关系表，这张表称为页表。

    * fork 的本质：**不是拷贝内存数据，仅仅是把页表做拷贝**。

    * fork 后两个进程各有自己的虚拟内存与页表，但映射到同一块物理内存区域，数据只有一份，从而实现内存共享、无需拷贝数据，速度非常快，阻塞时间尽可能缩短。

    * 子进程读的其实就是主进程的数据，读出来写入一个新的 RDB 文件，写完再替换旧的 RDB 文件。

```text
fork 之前
        主进程 Redis
             │
        虚拟内存 ──── 页表 ────> 物理内存


fork 之后：两个进程，两套页表，同一份数据
        主进程 Redis                 子进程 BGSAVE
             │                           │
        虚拟内存                     虚拟内存
             │                           │
             └─────── 各自的页表 ────────┘
                        │         │
                        ▼         ▼
                  同一块物理内存
                 （数据只有一份）
```

* **Copy On Write（写时复制，COW）**

    * > **定义**：fork 会把共享内存标记为 read only，任何一个进程都只能读不能写；主进程要写时必须先拷贝一份数据，拷完才能完成写操作，拷完之后主进程读也走这份新拷贝（页表映射改过去）。

    * 触发原因：异步执行期间主进程仍可接收用户请求并修改内存数据，子进程同时在读，读写之间会产生冲突甚至脏数据。

    * 粒度：每一次只要有写，就拷贝需要写的那一页。

* **极端情况与内存预留**

    * > **定义**：拷贝耗时较久、且写的过程中不断有新请求修改共享数据，极端情况下所有数据都被修改一遍、都要拷贝一份新的，Redis 的内存占用就会翻倍。

    * 概率与边界：几乎不可能发生，但理论上存在；16G 一旦翻倍就需要 32G。

    * > **注意**：给 Redis 预留内存空间是从 COW 原理反推出来的硬性要求；32G 的服务器不能把 32G 全交给 Redis 消耗完，否则做 RDB 时可能内存不够、内存溢出。

### RDB 的优缺点

* **BGSAVE 的流程**

    * > **定义**：满足触发条件执行 bgsave → fork（只拷贝页表，物理内存仍共享，这一步主进程阻塞、不能接受请求）→ fork 一结束主进程立刻继续处理用户请求、继续改内存 → 子进程异步读内存、写出新的 rdb 文件 → 写完之后用新文件替换旧文件。

    * > **注意**：最后一步是写一个**新的** RDB 文件再替换掉旧的，而不是直接修改原来那个文件。

* **执行时机**

    * > **定义**：RDB 有两种执行方式——`SAVE` 在服务停机那一刻执行，是阻塞式的；`BGSAVE` 则要满足配置条件才执行，属于异步持久化。

    * 示例：`save 60 1000` 代表每 60 秒内如果有 1000 次修改就触发 RDB。

* **优点**

    * > **定义**：因为 fork 只是拷贝页表、子进程与主进程共享物理内存，所以异步持久化对主进程几乎零阻塞；再加上文件可以压缩，RDB 文件本身也比较紧凑。

* **缺点**

    * > **定义**：每隔一段时间才做一次持久化，两次持久化之间产生的所有写操作，一旦宕机就会全部丢失，存在数据安全上的漏洞。

    * 不能靠缩短间隔来弥补：fork 拷贝页表耗时、写 RDB 文件耗时、压缩 RDB 文件也耗时，整个过程有可能耗时十几秒甚至几十秒；一秒一次的话，第一次 RDB 还没完成第二次又来了，根本忙不过来。
