# Java 8 新特性

## 基本使用

### [一、序言](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=一、序言)

Java8 是一个里程碑式的版本，凭借如下新特性，让人对其赞不绝口。

- Lambda 表达式给代码构建带来了全新的风格和能力；
- Steam API 丰富了集合操作，拓展了集合的能力；
- 新日期时间 API 千呼万唤始出来；

随着对 Java8 新特性理解的深入，会被 Lambda 表达式（包含方法引用）、流式运算的美所迷恋，不由惊叹框架设计的美。

### [二、方法引用](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=二、方法引用)

Lambda 表达式是匿名函数，可以理解为一段可以用参数传递的代码（代码像数据一样传递）。Lambda 表达式的使用需要有函数式接口的支持。

方法引用是对特殊 Lambda 表达式的一种简化写法，当 Lambda 体中只调用一个方法，此方法满足函数式接口规范，此时可以使用`::`方法引用语法。

从语法表现力角度来讲，`方法引用 > Lambda表达式 > 匿名内部类`，**方法引用是高阶版的 Lambda 表达式，语言表达更为简洁，强烈推荐使用。**

方法引用表达式无需显示声明被调用方法的参数，根据上下文自动注入。方法引用能够提高 Lambda 表达式语言的优雅性，代码更加简洁。下面以`Comparator`排序为例讲述如何借助方法引用构建优雅的代码。

#### [（一）方法引用与排序](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（一）方法引用与排序)

##### [1、普通数据类型](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、普通数据类型)

普通数据类型相对较容易理解。

```java
// 正向排序（方法引用）
Stream.of(11, 15, 11, 12).sorted(Integer::compareTo).forEach(System.out::println);
// 正向排序
Stream.of(11, 15, 11, 12).sorted(Comparator.naturalOrder()).forEach(System.out::println);
// 逆向排序
Stream.of(11, 15, 11, 12).sorted(Comparator.reverseOrder()).forEach(System.out::println);
```

##### [2、对象数据类型](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、对象数据类型)

**（1）数据完好**

数据完好有两重含义，一是对象本身不为空；二是待比较对象的属性值不为空，以此为前提进行排序操作。

```java
// 对集合按照年龄排序(正序排列)
Collections.sort(userList, Comparator.comparingInt(XUser::getAge));
// 对集合按照年龄排序（逆序排列）
Collections.sort(userList, Comparator.comparingInt(XUser::getAge).reversed());
```

此示例是以`Integer`类型展开的，同理`Double`类型、`Long`类型等数值类型处理方式相同。其中[Comparator](https://www.altitude.xin/code/home/#/java/util/Comparator)是排序过程中重要的类。

**（2）数据缺失**

数据缺失的含义是对象本身为空或者待比较对象属性为空，如果不进行处理，上述排序会出现空指针异常。

最常见的处理方式是通过流式运算中`filter`方法，过滤掉空指针数据，然后按照上述策略排序。

```java
userList.stream().filter(e->e.getAge()!=null).collect(Collectors.toList());
```

##### [3、字符串处理](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_3、字符串处理)

少数开发者在构建实体类时，`String`类型遍地开花，在需要运算或者排序的场景下，String 的缺陷逐渐暴露出来。下面讲述`字符串数值`类型排序问题，即不修改数据类型的前提下完成期望的操作。

实体类

```java
public class SUser {
    private Integer userId;
    private String UserName;
    // 本应该是Double类型，错误的使用为String类型
    private String score;
}
```

正序、逆序排序

```java
// 对集合按照年龄排序(正序排列)
Collections.sort(userList, Comparator.comparingDouble(e -> new Double(e.getScore())));
```

数据类型转换排序时，使用 JDK 内置的 API 并不流畅，推荐使用`commons-collection4`包中的排序工具类。了解更多，请移步查看[ComparatorUtils](https://www.altitude.xin/code/home/#/org/apache/commons/collections4/ComparatorUtils?id=comparatorutils)。

```java
// 对集合按照年龄排序(逆序排列)
Collections.sort(userList, ComparatorUtils.reversedComparator(Comparator.comparingDouble(e -> new Double(e.getScore()))));
```

> 小结：通过以排序为例，实现 Comparator 接口、Lambda 表达式、方法引用三种方式相比较，代码可读性逐步提高。

#### [（二）排序器](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（二）排序器)

内置的排序器可以完成大多数场景的排序需求，当排序需求更加精细化时，适时引入第三方框架是比较好的选择。

##### [1、单列排序](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、单列排序)

单列排序包含正序和逆序。

```java
// 正序
Comparator<Person> comparator = Comparator.comparing(XUser::getUserName);
// 逆序
Comparator<Person> comparator = Comparator.comparing(XUser::getUserName).reversed();
```

##### [2、多列排序](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、多列排序)

多列排序是指当待比较的元素有相等的值时，如何进行下一步排序。

```java
// 默认多列均是正序排序
Comparator<XUser> comparator = Comparator.comparing(XUser::getUserName)
    .thenComparing(XUser::getScore);
// 自定义正逆序
Comparator<XUser> comparator = Comparator.comparing(XUser::getUserName,Comparator.reverseOrder())
    .thenComparing(XUser::getScore,Comparator.reverseOrder());
```

### [三、Steam API](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=三、steam-api)

流的操作包含如下三个部分：创建流、中间流、关闭流，`筛选`、`去重`、`映射`、`排序`属于流的中间操作，`收集`属于终止操作。[Stream](https://www.altitude.xin/code/home/#/java/util/stream/Stream)是流操作的基础关键类。

#### [（一）创建流](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（一）创建流)

**（1）通过集合创建流**

```java
// 通过集合创建流
List<String> lists = new ArrayList<>();
lists.stream();
```

**（2）通过数组创建流**

```java
// 通过数组创建流
String[] strings = new String[5];
Stream.of(strings);
```

应用较多的是通过集合创建流，然后经过中间操作，最后终止回集合。

#### [（二）中间操作](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（二）中间操作)

##### [1、筛选（filter）](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、筛选（filter）)

筛选是指从（集合）流中筛选满足条件的子集，通过 Lambda 表达式生产型接口来实现。

```java
// 通过断言型接口实现元素的过滤
stream.filter(x->x.getSalary()>10);
```

**非空过滤**

非空过滤包含两层内容：一是当前对象是否为空或者非空；二是当前对象的某属性是否为空或者非空。

筛选非空对象，语法`stream.filter(Objects::nonNull)`做非空断言。

```java
// 非空断言
java.util.function.Predicate<Boolean> nonNull = Objects::nonNull;
```

查看[Objects](https://www.altitude.xin/code/home/#/java/util/Objects?id=objects)类了解更详细信息。

##### [2、去重（distinct）](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、去重（distinct）)

去重是指将（集合）流中重复的元素去除，通过 hashcode 和 equals 函数来判断是否是重复元素。去重操作实现了类似于 HashSet 的运算，对于对象元素流去重，需要重写 hashcode 和 equals 方法。

如果流中泛型对象使用 Lombok 插件，使用`@Data`注解默认重写了 hashcode 和 equals 方法，字段相同并且属性相同，则对象相等。更多内容可查看[Lombok 使用手册](https://www.altitude.xin/blog/home/#/chapter/99f38b299b57113aaf37016e58f3d56f)

```java
stream.distinct();
```

##### [3、映射（map）](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_3、映射（map）)

取出流中元素的某一列，然后配合收集以形成新的集合。

```java
stream.map(x->x.getEmpId());
```

`filter`和`map`操作通常结合使用，取出流中某行某列的数据，建议`先行后列`的方式定位。

```java
Optional<MainExportModel> model = data.stream().filter(e -> e.getResId().equals(resId)).findFirst();
if (model.isPresent()) {
    String itemName = model.get().getItemName();
    String itemType = model.get().getItemType();
    return new MainExportVo(itemId, itemName);
}
```

##### [4、排序（sorted）](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_4、排序（sorted）)

传统的`Collectors`类中的排序支持 List 实现类中的一部分排序，使用 stream 排序，能够覆盖所有的 List 实现类。

```java
// 按照默认字典顺序排序
stream.sorted();
// 按照工资大小排序
stream.sorted((x,y)->Integer.compare(x.getSalary(),y.getSalary()));
```

**（1）函数式接口排序**

基于 Comparator 类中函数式方法，能够更加优雅的实现对象流的排序。

```java
// 正向排序（默认）
pendingPeriod.stream().sorted(Comparator.comparingInt(ReservoirPeriodResult::getPeriod));
// 逆向排序
pendingPeriod.stream().sorted(Comparator.comparingInt(ReservoirPeriodResult::getPeriod).reversed());
```

**（2）LocalDate 和 LocalDateTime 排序**

新日期接口相比就接口，使用体验更加，因此越来越多的被应用，基于日期排序是常见的操作。

```java
// 准备测试数据
Stream<DateModel> stream = Stream.of(new DateModel(LocalDate.of(2020, 1, 1)), new DateModel(LocalDate.of(2019, 1, 1)), new DateModel(LocalDate.of(2021, 1, 1)));
```

正序、逆序排序

```java
// 正向排序（默认）
stream.sorted(Comparator.comparing(DateModel::getLocalDate)).forEach(System.out::println);
// 逆向排序
stream.sorted(Comparator.comparing(DateModel::getLocalDate).reversed()).forEach(System.out::println);
```

##### [5、规约（reduce）](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_5、规约（reduce）)

对流中的元素按照一定的策略计算。终止操作的底层逻辑都是由 reduce 实现的。

#### [（三）终止操作](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（三）终止操作)

收集（collect）将流中的中间（计算）结果存储到集合中，方便后续进一步使用。为了方便对收集操作的理解，方便读者掌握收集操作，将收集分为`普通收集`和`高级收集`。

##### [1、普通收集](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、普通收集)

**（1）收集为`List`**

默认返回的类型为`ArrayList`，可通过`Collectors.toCollection(LinkedList::new)`显示指明使用其它数据结构作为返回值容器。

```java
List<String> collect = stream.collect(Collectors.toList());
```

由集合创建流的收集需注意：仅仅修改流字段中的内容，没有返回新类型，如下操作直接修改原始集合，无需处理返回值。

```java
// 直接修改原始集合
userVos.stream().map(e -> e.setDeptName(hashMap.get(e.getDeptId()))).collect(Collectors.toList());
```

**（2）收集为`Set`**

默认返回类型为`HashSet`，可通过`Collectors.toCollection(TreeSet::new)`显示指明使用其它数据结构作为返回值容器。

```java
Set<String> collect = stream.collect(Collectors.toSet());
```

##### [2、高级收集](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、高级收集)

**（1）收集为`Map`**

默认返回类型为`HashMap`，可通过`Collectors.toCollection(LinkedHashMap::new)`显示指明使用其它数据结构作为返回值容器。

收集为`Map`的应用场景更为强大，下面对这个场景进行详细介绍。希望返回结果中能够建立`ID`与`NAME`之间的匹配关系，最常见的场景是通过`ID`批量到数据库查询`NAME`，返回后再将原数据集中的`ID`替换成`NAME`。

ID 到 NAME 映射

```java
@Data
public class ItemEntity {
    private Integer itemId;
    private String itemName;
}
```

准备集合数据，此部分通常是从数据库查询的数据

```java
// 模拟从数据库中查询批量的数据
List<ItemEntity> entityList = Stream.of(new ItemEntity(1,"A"), new ItemEntity(2,"B"), new ItemEntity(3,"C")).collect(Collectors.toList());
```

将集合数据转化成 ID 与 NAME 的 Map

```java
// 将集合数据转化成ID与NAME的Map
Map<Integer, String> hashMap = entityList.stream().collect(Collectors.toMap(ItemEntity::getItemId, ItemEntity::getItemName));
```

`ID`与`Object`类映射

```java
@Data
public class ItemEntity {
    private Integer itemId;
    private String itemName;
    private Boolean status;
}
```

将集合数据转化成 ID 与实体类的 Map

```java
// 将集合数据转化成ID与实体类的Map
Map<Integer, ItemEntity> hashMap = entityList.stream().collect(Collectors.toMap(ItemEntity::getItemId, e -> e));
```

其中`Collectors`类中的`toMap`参数是函数式接口参数，能够自定义返回值。

```java
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper) {
    return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
}
```

**（2）分组收集**

流的分组收集操作在内存层次模拟了数据库层面的`group by`操作，下面演示流的分组操作。[Collectors](https://www.altitude.xin/code/home/#/java/util/stream/Collectors?id=collectors)类提供了各种层次的分组操作支撑。

流的分组能力对应数据库中的聚合函数，目前大部分能在数据库中操作的聚合函数，都能在流中找到相应的能力。

```java
// 默认使用List作为分组后承载容器
Map<Integer, List<XUser>> hashMap = xUsers.stream().collect(Collectors.groupingBy(XUser::getDeptId));

// 显示指明使用List作为分组后承载容器
Map<Integer, List<XUser>> hashMap = xUsers.stream().collect(Collectors.groupingBy(XUser::getDeptId, Collectors.toList()));
```

映射后再分组

```java
Map<Integer, List<String>> hashMap = xUsers.stream().collect(Collectors.groupingBy(XUser::getDeptId,Collectors.mapping(XUser::getUserName,Collectors.toList())));
```

### [四、Stream 拓展](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=四、stream-拓展)

#### [（一）集合与对象互转](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（一）集合与对象互转)

将对象包装成集合的形式和将集合拆解为对象的形式是常见的操作。

##### [1、对象转集合](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、对象转集合)

返回默认类型的集合实例

```java
/**
 * 将单个对象转化为集合
 *
 * @param t   对象实例
 * @param <T> 对象类型
 * @param <C> 集合类型
 * @return 包含对象的集合实例
 */
public static <T, C extends Collection<T>> Collection<T> toCollection(T t) {
    return toCollection(t, ArrayList::new);
}
```

用户自定义返回的集合实例类型

```java
/**
 * 将单个对象转化为集合
 *
 * @param t        对象实例
 * @param supplier 集合工厂
 * @param <T>      对象类型
 * @param <C>      集合类型
 * @return 包含对象的集合实例
 */
public static <T, C extends Collection<T>> Collection<T> toCollection(T t, Supplier<C> supplier) {
    return Stream.of(t).collect(Collectors.toCollection(supplier));
}
```

##### [2、集合转对象](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、集合转对象)

使用默认的排序规则，注意此处不是指自然顺序排序。

```java
/**
 * 取出集合中第一个元素
 *
 * @param collection 集合实例
 * @param <E>        集合中元素类型
 * @return 泛型类型
 */
public static <E> E toObject(Collection<E> collection) {
    // 处理集合空指针异常
    Collection<E> coll = Optional.ofNullable(collection).orElseGet(ArrayList::new);
    // 此处可以对流进行排序，然后取出第一个元素
    return coll.stream().findFirst().orElse(null);
}
```

上述方法巧妙的解决两个方面的异常问题：一是集合实例引用空指针异常；二是集合下标越界异常。

#### [（二）其它](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（二）其它)

##### [1、并行计算](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、并行计算)

基于流式计算中的并行流，能够显著提高大数据下的计算效率，充分利用 CPU 核心数。

```java
// 通过并行流实现数据累加
LongStream.rangeClosed(1,9999999999999999L).parallel().reduce(0,Long::sum);
```

##### [2、序列数组](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、序列数组)

生成指定序列的数组或者集合。

```java
// 方式一：生成数组
int[] ints = IntStream.rangeClosed(1, 100).toArray();
// 方式二：生成集合
List<Integer> list = Arrays.stream(ints).boxed().collect(Collectors.toList());
```

### [五、其它](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=五、其它)

#### [（一）新日期时间 API](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（一）新日期时间-api)

##### [1、LocalDateTime](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、localdatetime)

```java
// 获取当前日期（包含时间）
LocalDateTime localDateTime = LocalDateTime.now();
// 获取当前日期
LocalDate localDate = localDateTime.toLocalDate();
// 获取当前时间
LocalTime localTime = localDateTime.toLocalTime();
```

![image-20220811063245421](E:\Development\Typora\images\image-20220811063245421.png)

日期格式化

```java
// 月份MM需要大写、小时字母需要大写（小写表示12进制）
DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
// 获取当前时间（字符串）
String dateTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println("dateTime = " + dateTime);
```

##### [2、Duration](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、duration)

```java
Duration duration = Duration.between(Instant.now(), Instant.now());
System.out.println("duration = " + duration);
```

##### [3、获取当前时间戳](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_3、获取当前时间戳)

如下方式获取的是 13 位时间戳，单位是毫秒。

```java
// 方式一
long now = Timestamp.valueOf(LocalDateTime.now()).getTime();
// 方式二
long now = Instant.now().toEpochMilli();
```

#### [（二）Optional](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（二）optional)

在[Optional](https://www.altitude.xin/code/home/#/java/util/Optional?id=optional)类出现之前，`null`异常几乎折磨着每一位开发者，为了构建健壮的应用程序，不得不使用繁琐的`if`逻辑判断来回避空指针异常。解锁`Optional`类，让你编写的应用健壮性更上一层楼。

##### [1、先判断后使用](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_1、先判断后使用)

`ifPresent`方法提供了先判断是否为空，后进一步使用的能力。

##### [2、链式取值](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=_2、链式取值)

链式取值是指，层层嵌套对象取值，在上层对象不为空的前提下，才能读取其属性值，然后继续调用，取出最终结果值。有时候只关心链末端的结果状态，即使中间状态为空，直接返回空值。如下提供了一种无 if 判断，代码简介紧凑的实现方式：

```java
Optional<Long> optional = Optional.ofNullable(tokenService.getLoginUser(ServletUtils.getRequest()))
                                    .map(LoginUser::getUser).map(SysUser::getUserId);
// 如果存在则返回，不存在返回空
Long userId = optional.orElse(null);
```

### [六、流的应用](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=六、流的应用)

#### [（一）列表转树](https://www.altitude.xin/blog/home/#/chapter/0c2d47d5a4a3ee8c67d5589a9c57f278?id=（一）列表转树)

传统方式下构建树形列表需要反复递归调用查询数据库，效率偏低。对于一棵结点较多的树，效率更低。这里提供一种只需调用一次数据库，通过流将列表转化为树的解决方式。

![image-20211014133305884](E:\Development\Typora\images\image-20211014133305884.png)

```java
/**
 * 列表转树
 *
 * @param rootList     列表的全部数据集
 * @param parentId 第一级目录的父ID
 * @return 树形列表
 */
public List<IndustryNode> getChildNode(List<Industry> rootList, String parentId) {
    List<IndustryNode> lists = rootList.stream()
            .filter(e -> e.getParentId().equals(parentId))
            .map(IndustryNode::new).collect(toList());
    lists.forEach(e -> e.setChilds(getChildNode(rootList, e.getId())));
    return lists;
}
```



## 详细学习

### 一.时间

#### 1.1ZoneDateTime 

##### 1.地理知识 ： 时区图

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16.png)

##### 2. ZoneDateTime 类

```
ZoneDateTime ： 是带时区的日期时间类。
区别于 LocalDateTime 类，LocalDateTime 类是 默认时区的日期时间类，中国默认的 时区是 【东八区】。
因此，当LocalDateTime 手动指定 时区后 就变成了ZoneDateTime。
时区id 可以通过ZoneId类的API 直接获取。
```

##### 3. 相关的[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020) 代码

> 下面简单演示一下相关的操作

```java
package com.northcastle.K_Date;

/**
 * author : northcastle
 * createTime:2022/3/24
 */

import java.time.*;
import java.util.Date;
import java.util.Set;

/**
 * 带时区的时间对象
 * ZoneDateTime : LocalDateTime + ZoneId
 */
public class ZoneDateTimeTest {
    public static void main(String[] args) {
        //1.获取所有的时区id
        Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
        System.out.println("availableZoneIds = " + availableZoneIds);
        System.out.println("====================================");

        // 2.1 获取当前的时间 LocalDateTime ,中国 东八区，比标准的时区要早8个小时
        LocalDateTime localDateTimeNow = LocalDateTime.now();
        System.out.println("localDateTimeNow = " + localDateTimeNow); // localDateTimeNow = 2022-03-24T21:34:58.865

        // 2.2 获取标准时区的时间 ZoneDateTime
        ZonedDateTime zoneDateTimeNow01 = ZonedDateTime.now(Clock.systemUTC());
        System.out.println("zoneDateTimeNow01 = " + zoneDateTimeNow01); // zoneDateTimeNow = 2022-03-24T13:34:58.866Z

        // 2.3 使用计算机的默认时区
        ZonedDateTime zoneDateTimeNow02 = ZonedDateTime.now();
        System.out.println("zoneDateTimeNow02 = " + zoneDateTimeNow02);

        // 2.4 指定时区id
        ZonedDateTime zoneDateTimeNow03 = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("zoneDateTimeNow03 = " + zoneDateTimeNow03);

        System.out.println();
       
        // zoneDateTime 对象 转 LocalDateTime 对象
        System.out.println("zoneDateTimeNow03.toLocalDate() = " + zoneDateTimeNow03.toLocalDate());
        System.out.println("zoneDateTimeNow03.toLocalTime() = " + zoneDateTimeNow03.toLocalTime());
        System.out.println("zoneDateTimeNow03.toLocalDateTime() = " + zoneDateTimeNow03.toLocalDateTime());
        System.out.println("zoneDateTimeNow03.toOffsetDateTime() = " + zoneDateTimeNow03.toOffsetDateTime());
        System.out.println("zoneDateTimeNow03.toInstant() = " + zoneDateTimeNow03.toInstant());

        System.out.println("====================================");
    
    }
}
```



#### 1.2LocalTime

##### 1.说明

```
LocalTime 是 Java8 中新增的 时间类，主要包含了 小时、分钟、秒、纳秒 四个属性。
LocalTime类中提供了丰富的API，帮助我们更加简便的操作时间对象。
```

##### 2.常用[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)

###### 2.1 创建LocalTime

```java
 * 1.创建时间类对象
 *    1.1 获取当前时间
 *      LocalTime.now() : 获取默认时区下的系统时钟的时间
 *      LocalTime.now(Clock clock) : 获取指定时钟的时间
 *      LocalTime.now(ZoneId zone) : 获取指定时区的、默认系统时钟的时间
 *      【补充：获取所有时区信息的方式 ： ZoneId.getAvailableZoneIds()】
 *    1.2 获取指定时间
 *      * 小时的取值范围 : [0,23]
 *      * 分钟的取值范围 : [0,59]
 *      * 秒  的取值范围 : [0,59]
 *      * 纳秒的取值范围 : [0,999,999,999]
 *
 *      LocalTime.of(int hour, int minute) : 指定小时和分钟，秒和纳秒为0
 *      LocalTime.of(int hour, int minute, int second) : 指定小时、分钟、秒，纳秒为0
 *      LocalTime.of(int hour, int minute, int second, int nanoOfSecond) : 指定小时、分钟、秒、纳秒
 *      LocalTime.ofSecondOfDay(long secondOfDay) : 指定一天当中的多少秒，纳秒将被置为0
 *      LocalTime.ofNanoOfDay(long nanoOfDay) : 指定一天当中的多少纳秒
 *
```

> 代码 

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
        //1.1 获取当前时间
        LocalTime now01 = LocalTime.now(); // 我们是中国北京，东八区
        System.out.println("now01 = " + now01);
        LocalTime now02 = LocalTime.now(Clock.systemUTC()); // UTC/GMT 标准时区时间,零时区/中时区
        System.out.println("now02 = " + now02);
        LocalTime now03 = LocalTime.now(ZoneId.of("America/Panama"));  // Asia/Shanghai 东八区 ；America/Panama 西五区
        System.out.println("now03 = " + now03);
        System.out.println("===========================");

        //1.2 获取指定时间
        LocalTime localTime01 = LocalTime.of(18, 10);
        System.out.println("localTime01 = " + localTime01);
        LocalTime localTime02 = LocalTime.of(19, 44, 56);
        System.out.println("localTime02 = " + localTime02);
        LocalTime localTime03 = LocalTime.of(19, 45, 12, 126);
        System.out.println("localTime03 = " + localTime03);
        LocalTime localTime04 = LocalTime.ofSecondOfDay(56);
        System.out.println("localTime04 = " + localTime04);
        LocalTime localTime05 = LocalTime.ofNanoOfDay(100);
        System.out.println("localTime05 = " + localTime05);
        System.out.println("===========================");

		//获取所有可用的时区
		//        List<String> zoneIds = ZoneId.getAvailableZoneIds().stream()
		//                .filter(s -> s.startsWith("Ame"))
		//                .collect(Collectors.toList());
		//        System.out.println(zoneIds);
    }
}

```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_18,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.2 获取LocalTime对象的时分秒

```java
 * 2.获取时间对象的 时、分、秒、纳秒、信息
 *      getHour() : 获取小时信息
 *      getMinute() : 获取分钟信息
 *      getSecond() : 获取秒
 *      getNano() :  获取纳秒
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
       //2.获取时间对象的 时、分、秒、纳秒
        LocalTime now = LocalTime.now();
        System.out.println("now = " + now);
        System.out.println("小时 ： " + now.getHour());
        System.out.println("分钟 ： " + now.getMinute());
        System.out.println("秒   ： " + now.getSecond());
        System.out.println("纳秒 ： " + now.getNano());

        System.out.println("===========================");
    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\a0ed52c45da145b59f7e5b4652df45f9.png)

###### 2.3 指定LocalTime对象的时分秒信息

```
 * 3.指定时间对象的 时、分、秒、纳秒
 *      withHour(int) : 指定小时
 *      withMinute(int) : 指定分钟
 *      withSecond(int) : 指定秒
 *      withNano(int) : 指定毫秒
 *
 *      with(TemporalAdjuster) : 时间矫正器
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
       //3.指定时、分、秒、纳秒
        LocalTime nowWith = LocalTime.now();
        System.out.println("nowWith = " + nowWith);
        System.out.println("指定小时6点 = " + nowWith.withHour(6));
        System.out.println("指定分钟5分钟 = " + nowWith.withMinute(50));
        System.out.println("指定秒5秒 = " + nowWith.withSecond(5));
        System.out.println("指定纳秒123000000 = " + nowWith.withNano(123000000));

        System.out.println();

        //时间矫正器
        TemporalAdjuster temporalAdjuster = (temporal)->{
            System.out.println("temporal = " + temporal);
            LocalTime localTime = (LocalTime) temporal;
            localTime = localTime.withHour(localTime.getHour()+1)
                        .withMinute(30)
                        .withSecond(30)
                        .withNano(300000000);
            System.out.println("localTime = " + localTime);
            return localTime;
        };
        LocalTime withTemporalAdjuster01 = nowWith.with(temporalAdjuster);
        System.out.println("withTemporalAdjuster01 = " + withTemporalAdjuster01);

        System.out.println("===========================");
    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_15,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.4 增加/减少 LocalTime 对象的时分秒信息

```
 * 4.加上 或者 减去 时、分、秒、纳秒
 *     plusHours(long) : 加几个小时
 *     plusMinutes(long) : 加几分钟
 *     plusSeconds(long) : 加几秒钟
 *     plusNanos(long) : 加几个纳秒
 *
 *     minusHours(long) : 减几个小时
 *     minusMinutes(long) : 减几分钟
 *     minusSeconds(long) : 减几秒
 *     minusNanos(long) : 减几个纳秒
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
       //4.加上 或者 减去 小时、分钟、秒、纳秒
        LocalTime nowPlusOrMin = LocalTime.now();
        System.out.println("nowPlusOrMin = " + nowPlusOrMin);
        System.out.println("加两小时 = " + nowPlusOrMin.plusHours(2));
        System.out.println("加20分钟 = " + nowPlusOrMin.plusMinutes(20));
        System.out.println("加20秒 = " + nowPlusOrMin.plusSeconds(20));
        System.out.println("加1纳秒 = " + nowPlusOrMin.plusNanos(1));
        
        System.out.println();
        
        System.out.println("减两小时 = " + nowPlusOrMin.minusHours(2));
        System.out.println("减20分钟 = " + nowPlusOrMin.minusMinutes(20));
        System.out.println("减20秒 = " + nowPlusOrMin.minusSeconds(20));
        System.out.println("减1纳秒 = " + nowPlusOrMin.minusNanos(1));

        System.out.println("===========================");
    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_15,color_FFFFFF,t_70,g_se,x_16-16602865188652.png)

###### 2.5 LocalTime 对象的大小比较

```
 *  5.比较时间对象的大小
 *      time01.isAfter(time02) : time01 是否比 time02 晚
 *      time01.isBefore(time02) : time01 是否比 time02 早
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
      
		//5.比较两个时间对象的大小
        LocalTime time01 = LocalTime.now();
        LocalTime time02 = LocalTime.of(21, 28, 30);
        System.out.println("time01 = " + time01);
        System.out.println("time02 = " + time02);
        System.out.println("time01是否比time02 晚 ： " + time01.isAfter(time02));
        System.out.println("time01是否比time02 早 ： " + time01.isBefore(time02));

        System.out.println("===========================");
    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\6b572c35872c47dba9d4c93a68172521.png)

2.6 LocalTiime 对象格式化为字符串

```
 *  6.时间对象 与 字符串的相互转化
 *    6.1 时间对象 转 字符串 format() + DateTimeFormatter类
 *        String format01 = localTime.format(DateTimeFormatter.ISO_TIME); // 21:56:17.05
 *        String format02 = localTime.format(DateTimeFormatter.ISO_LOCAL_TIME); // 21:56:17.05
 *        String format03 = localTime.format(DateTimeFormatter.ofPattern("HH--:--mm--:--ss")); // 21--:--56--:--17
 *
 *    6.2 字符串 转 时间对象 parse()
 *         LocalTime timeParse01 = LocalTime.parse("21:53:53", DateTimeFormatter.ofPattern("HH:mm:ss"));
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
       //6.1 时间对象 转 字符串
        LocalTime timeFormat01 = LocalTime.now();
        System.out.println("timeFormat01 = " + timeFormat01);

        String format01 = timeFormat01.format(DateTimeFormatter.ISO_TIME); // 21:56:17.05
        System.out.println("format01(ISO_TIME) = " + format01);

        String format02 = timeFormat01.format(DateTimeFormatter.ISO_LOCAL_TIME); // 21:56:17.05
        System.out.println("format02(ISO_LOCAL_TIME) = " + format02);

        String format03 = timeFormat01.format(DateTimeFormatter.ofPattern("HH--:--mm--:--ss")); // 21--:--56--:--17
        System.out.println("format03 = " + format03);

        System.out.println();
        //6.2 字符串 转 时间对象
        LocalTime timeParse01 = LocalTime.parse("21:53:53", DateTimeFormatter.ofPattern("HH:mm:ss"));
        System.out.println("timeParse01 = " + timeParse01);

        System.out.println("===========================");

    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_18,color_FFFFFF,t_70,g_se,x_16-16602865188663.png)

###### 2.7 计算两个时间对象之间的时间差

```
 *  7.获取两个时间对象之间的差 Duration
 *         Duration between = Duration.between(timeDuration01, timeDuration02);
 *         between.toDays()
 *         between.toHours() // 一共差多少个小时
 *         between.toMinutes() // 一共差多少个分钟
 *         between.toMillis() // 一共差多少毫秒
 *         between.toNanos() // 一共差多少纳秒
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.Clock;
import java.time.Duration;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;

public class LocalTimeTest {
    public static void main(String[] args) {
        //7.获取两个时间对象之间的差
        LocalTime timeDuration01 = LocalTime.of(22, 38, 50);
        System.out.println("timeDuration01 = " + timeDuration01);
        LocalTime timeDuration02 = LocalTime.of(22, 56, 32);
        System.out.println("timeDuration02 = " + timeDuration02);

        Duration between = Duration.between(timeDuration01, timeDuration02);
        System.out.println("between.toDays() = " + between.toDays());
        System.out.println("between.toHours() = " + between.toHours()); // 一共差多少个小时
        System.out.println("between.toMinutes() = " + between.toMinutes()); // 一共差多少个分钟
        System.out.println("between.toMillis() = " + between.toMillis()); // 一共差多少毫秒
        System.out.println("between.toNanos() = " + between.toNanos()); // 一共差多少纳秒

        System.out.println();

        System.out.println("ChronoUnit.HOURS.between(timeDuration01,timeDuration02) = " + ChronoUnit.HOURS.between(timeDuration01, timeDuration02));
        System.out.println("ChronoUnit.MINUTES.between(timeDuration01,timeDuration02) = " + ChronoUnit.MINUTES.between(timeDuration01, timeDuration02));
        System.out.println("ChronoUnit.MILLIS.between(timeDuration01,timeDuration02) = " + ChronoUnit.MILLIS.between(timeDuration01, timeDuration02));
        System.out.println("ChronoUnit.NANOS.between(timeDuration01,timeDuration02) = " + ChronoUnit.NANOS.between(timeDuration01, timeDuration02));

        System.out.println("===========================");

    }
}
```

> 运行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-16602865188664.png)



#### 1.3LocalDate

##### 1.说明

```
LocalDate 类 是 Java8中新增的日期类，采用了系统的默认时区。
可以方便的处理 日期对象 的 年、月、日 信息。
```

##### 2.常用[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)

###### 2.1 获取LocalDate 对象

```
 * 1.创建日期对象
 *   1.1 获取当前日期对象
 *       LocalDate.now() : 返回默认时区下的、系统始终下的 当前日期
 *       LocalDate.now(Clock clock) : 返回指定时钟的 当前日期
 *       LocalDate.now(ZoneId zone) : 返回指定时区的 当前日期
 *       【补充：获取所有时区信息的方式 ： ZoneId.getAvailableZoneIds()】
 *   1.2 获取指定日期对象
 *       LocalDate.of(int year, Month month, int dayOfMonth) : 指定年、月份名称、日期
 *       LocalDate.of(int year, int month, int dayOfMonth) : 指定 年、月份数字、日期
 *       LocalDate.ofYearDay(int year, int dayOfYear) : 指定 年分、这一年的第几天
 *       LocalDate.ofEpochDay(long epochDay) : 指定 距离 1970-01-01 00:00:00 的天数
 *

```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
        //1.1 获取当前日期
        LocalDate now01 = LocalDate.now();
        System.out.println("now01 = " + now01);
        LocalDate now02 = LocalDate.now(Clock.systemUTC());// UTC/GMT 标准时区时间,零时区/中时区
        System.out.println("now02 = " + now02);
        LocalDate now03 = LocalDate.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("now03 = " + now03);
        System.out.println("============================");

        //1.2 获取指定日期
        LocalDate localDate01 = LocalDate.of(2022, Month.MARCH, 20);
        System.out.println("localDate01 = " + localDate01);
        LocalDate localDate02 = LocalDate.of(2022, 3, 20);
        System.out.println("localDate02 = " + localDate02);
        LocalDate localDate03 = LocalDate.ofYearDay(2022, 6); // 2022-01-06
        System.out.println("localDate03 = " + localDate03);
        LocalDate localDate04 = LocalDate.ofEpochDay(3);// 距离 1970-01-01 00:00:00 三天的日期，还是 1970-01-04
        System.out.println("localDate04 = " + localDate04);

        System.out.println("============================");

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_15,color_FFFFFF,t_70,g_se,x_16-166030030058614.png)

###### 2.2 获取 LocalDate 对象的 年月日信息

```
 * 2.获取日期对象的 年、月、日 信息
 *      getYear() : 获取年分信息
 *      getMonth() : 获取月份信息（枚举类型）
 *      getMonthValue() : 获取月份的数字（数值类型）
 *      getDayOfMonth() : 获取日期信息
 *      getDayOfWeek() : 获取星期几 （枚举类型）
 *      getDayOfYear() : 获取这一年的第几天
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
         //2.获取日期对象的年月日
        LocalDate today = LocalDate.now();
        System.out.println("年分 ： " + today.getYear());
        System.out.println("月份 ： " + today.getMonth() + " ; 月份 ： "+today.getMonth().getValue()+" ; 月份 ： " + today.getMonthValue());
        System.out.println("日期 ： " + today.getDayOfMonth());
        System.out.println("星期 ： " + today.getDayOfWeek()+" ; 星期 ： "+today.getDayOfWeek().getValue());
        System.out.println("今年的第几天 ： " + today.getDayOfYear());

        System.out.println("============================");

        //  获取时区的方法 ： Asia/Shanghai
//        Set<String> asia = ZoneId.getAvailableZoneIds()
//                .stream()
//                .filter(s -> s.startsWith("Asia"))
//                .collect(Collectors.toSet());
//        System.out.println(asia);

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_15,color_FFFFFF,t_70,g_se,x_16-166030030058715.png)

###### 2.3 修改LocalDate 对象的 年月日信息

```
 *  3.指定 日期对象的属性
 *      withYear(int) : 设置年
 *      withMonth(int) : 设置月
 *      withDayOfMonth(int) : 设置日
 *      withDayOfYear(int) : 设置这一年的第几天
 *
 *      with(TemporalAdjuster) : 时间调整器，更方便的调整
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
       // 3.指定日期 年、月、日 属性
        LocalDate nowWith = LocalDate.now();
        System.out.println("now = " + nowWith);
        System.out.println("指定年分2025年 = " + nowWith.withYear(2025));
        System.out.println("指定月份6月 = " + nowWith.withMonth(6));
        System.out.println("指定日期15号 = " + nowWith.withDayOfMonth(15));
        System.out.println("指定日期这一年的第2天 = " + nowWith.withDayOfYear(2));

        System.out.println();

        // 时间矫正器 : 调整到下个月的1号
        // 1.手动调整
        TemporalAdjuster  temporalAdjuster01 = (temporal)->{
            System.out.println("temporal = "+temporal);
            LocalDate localDate = (LocalDate) temporal;
            localDate = localDate.withMonth(localDate.getMonthValue()+1)
                                    .withDayOfMonth(1); //  注意对象是线程安全的，所以必须要重新接收一边
            System.out.println("localDate = " + localDate);
            return localDate;
        };
        LocalDate withTemporalAdjuster01 = nowWith.with(temporalAdjuster01);
        System.out.println("withTemporalAdjuster01 = " + withTemporalAdjuster01);
        
        //2.使用已经封装好的
        LocalDate withTemporalAdjuster02 = nowWith.with(TemporalAdjusters.firstDayOfNextMonth()); // 现成的方法
        System.out.println("withTemporalAdjuster02 = " + withTemporalAdjuster02);

        LocalDate withTemporalAdjuster03 = nowWith.with(TemporalAdjusters.dayOfWeekInMonth(1,DayOfWeek.MONDAY)); // 调整到这个月的第一个星期一
        System.out.println("withTemporalAdjuster03 = " + withTemporalAdjuster03);


        System.out.println("============================");

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_16,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.4 增加/减少 LocalDate 对象的 年月日 信息

```
 *  4.加上 或者 减去 年、月、日、周
 *      plusYears(long) : 加几年
 *      plusMonths(long) : 加几个月
 *      plusDays(long) : 加几天
 *      plusWeeks(long) : 加几个星期
 *
 *      minusYears(long) : 减几年
 *      minusMonths(long) : 减几个月
 *      minusDays(long) : 减几天
 *      minusWeeks(long) : 减几个星期
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
    
      	// 4. 加上 或者 减去 指定数量 年份、月份、天数、周数
        LocalDate nowPlusOrMin = LocalDate.now();
        System.out.println("nowPlusOrMin = " + nowPlusOrMin);
        System.out.println("加两年 = " + nowPlusOrMin.plusYears(2));
        System.out.println("加13个月 = " + nowPlusOrMin.plusMonths(13));
        System.out.println("加12天 = " + nowPlusOrMin.plusDays(12));
        System.out.println("加一周 = " + nowPlusOrMin.plusWeeks(1));
        System.out.println();
        System.out.println("减两年 = " + nowPlusOrMin.minusYears(2));
        System.out.println("减四个月 = " + nowPlusOrMin.minusMonths(4));
        System.out.println("减22天 = " + nowPlusOrMin.minusDays(22));
        System.out.println("减1个星期 = " + nowPlusOrMin.minusWeeks(1));
        System.out.println("============================");

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_15,color_FFFFFF,t_70,g_se,x_16-166030030058816.png)

###### 2.5 LocalDate 对象的大小比较

```
 *  5.日期对象的比较
 *      date01.isEqual(date02) : 两个日期是否相等
 *      date01.isAfter(date02) : date01 是否 比 date02 晚
 *      date01.isBefore(date02) : date01 是否 比 date02 早
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
    
       // 5.日期对象比较大小
        LocalDate date01 = LocalDate.now();
        LocalDate date02 = LocalDate.of(2022, 3, 21);
        System.out.println("date01 = " + date01);
        System.out.println("date02 = " + date02);
        System.out.println("两天是否一致 ： " + date01.isEqual(date02));
        System.out.println("date01是否比date02晚 ： " + date01.isAfter(date02));
        System.out.println("date01是否比date02早 ： " + date01.isBefore(date02));

        System.out.println("============================");

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_12,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.6 LocalDate 的格式化为 字符串

```
 *  6.日期对象与字符串的转化
 *    6.1 日期对象转字符串 format() + DateTimeFormatter类
 *        String format01 = localDate.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 最常用 2022-03-22
 *        String format02 = localDate.format(DateTimeFormatter.ISO_DATE);          // 2022-03-23
 *        String format03 = localDate.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20220323
 *        String format04 = localDate.format(DateTimeFormatter.ISO_ORDINAL_DATE); // 2022-082 年分-当前日期是这一年的第几天
 *        String format05 = localDate.format(DateTimeFormatter.ISO_WEEK_DATE);    // 2022-W12-3 年分-第几周-星期几
 *        String format06 = localDate.format(DateTimeFormatter.ofPattern("yyyy--$--MM--$--dd")); // 自定义格式化
 *
 *
 *    6.2 字符串转日期对象 parse()
 *        LocalDate dateParse01 = LocalDate.parse("2022--$--03--$--23",DateTimeFormatter.ofPattern("yyyy--$--MM--$--dd"));
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
    
        // 6.1 日期对象 格式化 为字符串
        LocalDate dateFormat01 = LocalDate.now();
        System.out.println("dateFormat01 = " + dateFormat01);

        String format01 = dateFormat01.format(DateTimeFormatter.ISO_LOCAL_DATE); // 最常用 2022-03-22
        System.out.println("format01(ISO_LOCAL_DATE) = " + format01);

        String format02 = dateFormat01.format(DateTimeFormatter.ISO_DATE); // 2022-03-23
        System.out.println("format02(ISO_DATE) = " + format02);

        String format03 = dateFormat01.format(DateTimeFormatter.BASIC_ISO_DATE); // 20220323
        System.out.println("format03(BASIC_ISO_DATE) = " + format03);

        String format04 = dateFormat01.format(DateTimeFormatter.ISO_ORDINAL_DATE); // 2022-082 年分-当前日期是这一年的第几天
        System.out.println("format04(ISO_ORDINAL_DATE) = " + format04);

        String format05 = dateFormat01.format(DateTimeFormatter.ISO_WEEK_DATE); // 2022-W12-3 年分-第几周-星期几
        System.out.println("format05(ISO_WEEK_DATE) = " + format05);

        String format06 = dateFormat01.format(DateTimeFormatter.ofPattern("yyyy--$--MM--$--dd")); // 自定义格式化
        System.out.println("format06(ofPattern) = " + format06);

        System.out.println();

        //6.2 字符串 转 时间对象
        LocalDate dateParse01 = LocalDate.parse("2022--$--03--$--23",DateTimeFormatter.ofPattern("yyyy--$--MM--$--dd"));
        System.out.println("dateParse01 = " + dateParse01);

        System.out.println("============================");


    }
}

```

> 执行结果

[深度思考JDK8中日期类型该如何使用详解pdf![img](E:\Development\Typora\images\star-166030030058820.png)0星超过10%的资源416KB![img](E:\Development\Typora\images\arrowDownWhite-166030030058822.png)下载](https://download.csdn.net/download/weixin_38523728/12724454)

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_17,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.7 计算两个LocalDate 对象的日期差

```
 *  7.计算两个日期的差  Period
 *    Period between = Period.between(datePeriod01, datePeriod02); // 小一点的日期，大一点的日期，否则会出现负数
 *    7.1 获取相差的【年月日】整体的一个效果
 *        between.getYears()
 *        between.getMonths()
 *        between.getDays()
 *    7.2 分别获取相差的【年】【月】【日】的数据
 *        ChronoUnit.YEARS.between(datePeriod01, datePeriod02));
 *        ChronoUnit.MONTHS.between(datePeriod01, datePeriod02));
 *        ChronoUnit.DAYS.between(datePeriod01, datePeriod02));
 *

```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTest {
    public static void main(String[] args) {
    
        //7.计算两个日期对象之间的差
        LocalDate datePeriod01 = LocalDate.of(2022,1,27);
        System.out.println("datePeriod01 = " + datePeriod01);
        LocalDate datePeriod02 = LocalDate.of(2022,3,22);
        System.out.println("datePeriod02 = " + datePeriod02);

        Period between = Period.between(datePeriod01, datePeriod02); // 小一点的日期，大一点的日期

        System.out.println("between.getYears() = " + between.getYears());
        System.out.println("between.getMonths() = " + between.getMonths());
        System.out.println("between.getDays() = " + between.getDays());


        System.out.println("相差年数 = " + ChronoUnit.YEARS.between(datePeriod01, datePeriod02)); // 获取每个维度的数据
        System.out.println("相差月数 = " + ChronoUnit.MONTHS.between(datePeriod01, datePeriod02));
        System.out.println("相差天数 = " + ChronoUnit.DAYS.between(datePeriod01, datePeriod02));

        System.out.println("============================");

    }
}
```

> 执行结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_14,color_FFFFFF,t_70,g_se,x_16.png)

###### 2.8 其他方法

```
 *    其他方法
 *      isLeapYear() : 判断是否是闰年
 *      atStartOfDay() : 自动赋值 时分秒 为0
 *      lengthOfMonth() : 返回当前月份的长度
 *      lengthOfYear() : 返回今年有多少天
```

> 其他的这些简单API 各位Coder 可以自行 测试，不在此赘述了，



#### 1.4LocalDateTime

1.2和1.3的結合版本

##### 1.说明

```
LocalDateTime 是 Java8 中提供的 日期+时间的对象，
日期 包含 年、月、日 信息
时间包含 小时、分钟、秒、纳秒 信息
此对象默认使用系统时区，例如中国使用 【东八区】
 
```

##### 2.常用[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)

###### 2.1 创建LocalDateTime 对象

```
 * 1.创建日期时间对象
 *   1.1 获取当前日期时间对象
 *       LocalTime.now() : 获取默认时区下的系统时钟的时间
 *       LocalTime.now(Clock clock) : 获取指定时钟的时间
 *       LocalTime.now(ZoneId zone) : 获取指定时区的、默认系统时钟的时间
 *       【补充：获取所有时区信息的方式 ： ZoneId.getAvailableZoneIds()】 Asia/Shanghai
 *   1.2 获取指定日期时间对象
 *        LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute) : 指定 年、月（枚举值）、日、小时、分钟
 *        LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute, int second) : 指定 年、月（枚举值）、日、小时、分钟、秒
 *        LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond) : 指定 年、月（枚举值）、日、小时、分钟、秒、纳秒
 *        LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute) : 指定 年、月（数值）、日、小时、分钟
 *        LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second) : 指定 年、月（数值）、日、小时、分钟、秒
 *        LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond) : 指定 年、月（数值）、日、小时、分钟、秒、纳秒
 *        LocalDateTime.of(LocalDate date, LocalTime time) : 指定 日期对象 和 时间对象 两个对象
 *        LocalDateTime ofInstant(Instant instant, ZoneId zone) : 指定时间戳对象 Instant 和 时区
 *        LocalDateTime ofEpochSecond(long epochSecond, int nanoOfSecond, ZoneOffset offset) : 指定 距离 1970-01-01 00:00:00 的秒数、纳秒数、偏移数，创建LocalDateTime对象
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
        //1.1 获取当前日期
        LocalDateTime now01 = LocalDateTime.now();
        System.out.println("now01 = " + now01);
        LocalDateTime now02 = LocalDateTime.now(Clock.systemUTC());// UTC/GMT 标准时区时间,零时区/中时区
        System.out.println("now02 = " + now02);
        LocalDateTime now03 = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("now03 = " + now03);

        System.out.println("============================");

        //1.2 获取指定日期
        LocalDateTime localDateTime01 = LocalDateTime.of(2022, Month.MARCH, 20, 20, 46);
        System.out.println("localDateTime01 = " + localDateTime01);
        LocalDateTime localDateTime02 = LocalDateTime.of(2022, Month.MARCH, 20, 20, 47,20);
        System.out.println("localDateTime02 = " + localDateTime02);
        LocalDateTime localDateTime03 = LocalDateTime.of(2022, Month.MARCH, 20, 20, 47,30,120);
        System.out.println("localDateTime03 = " + localDateTime03);
        LocalDateTime localDateTime04 = LocalDateTime.of(2022, 3, 20, 20, 46);
        System.out.println("localDateTime04 = " + localDateTime04);
        LocalDateTime localDateTime05 = LocalDateTime.of(2022, 3, 20, 20, 47,20);
        System.out.println("localDateTime05 = " + localDateTime05);
        LocalDateTime localDateTime06 = LocalDateTime.of(2022, 3, 20, 20, 47,30,120);
        System.out.println("localDateTime06 = " + localDateTime06);

        LocalDate localDate = LocalDate.of(2022, 3, 20);
        LocalTime localTime = LocalTime.of(20, 48, 50);
        LocalDateTime localDateTime07 = LocalDateTime.of(localDate, localTime);
        System.out.println("localDateTime07 = " + localDateTime07);

        Instant instant08 = Instant.ofEpochSecond(100); // 1970-01-01 00:01:40
        ZoneId zoneId08 = ZoneId.systemDefault(); // 我们中国这儿的 默认时区 是 Asia/Shanghai，东八区，多8个小时
        LocalDateTime localDateTime08 = LocalDateTime.ofInstant(instant08, zoneId08); // 默认时区
        System.out.println("localDateTime08 = " + localDateTime08); // 所以这里是 1970-01-01 08:01:40

        //ZoneOffset zoneOffset09 = ZoneOffset.of("+8"); // +8小时
        //ZoneOffset zoneOffset09 = ZoneOffset.UTC; // 标准时区 零时区的时间
        //ZoneOffset zoneOffset09 = ZoneOffset.MAX; // +18 最大偏移
        //ZoneOffset zoneOffset09 = ZoneOffset.MIN; // -18 最小偏移
        //ZoneOffset zoneOffset09 = ZoneOffset.ofHours(2); // 增加两个小时
        ZoneOffset zoneOffset09 = ZoneOffset.ofHoursMinutes(0,10); // 增加0小时10分钟

        LocalDateTime localDateTime09 = LocalDateTime.ofEpochSecond(100, 0,zoneOffset09);
        System.out.println("localDateTime09 = " + localDateTime09);
        System.out.println("============================");
        //  获取时区的方法 ： Asia/Shanghai
//        Set<String> asia = ZoneId.getAvailableZoneIds()
//                .stream()
//                .filter(s -> s.startsWith("Asia"))
//                .collect(Collectors.toSet());
//        System.out.println(asia);


    }
}

```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_18,color_FFFFFF,t_70,g_se,x_16-166030051963528.png)

###### 2.2 获取LocalDateTime对象的属性信息

```
 * 2.获取日期时间对象的信息
 *      toLocalDate() : 返回一个LocalDate 对象
 *      toLocalTime() : 返回一个LocalTime 对象
 *
 *      getYear() : 获取年分信息
 *      getMonth() : 获取月份信息（枚举类型）
 *      getMonthValue() : 获取月份的数字（数值类型）
 *      getDayOfMonth() : 获取日期信息
 *      getDayOfWeek() : 获取星期几 （枚举类型）
 *      getDayOfYear() : 获取这一年的第几天
 *
 *      getHour() : 获取小时信息
 *      getMinute() : 获取分钟信息
 *      getSecond() : 获取秒
 *      getNano() :  获取纳秒
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
    
        //2.获取日期对象的年月日、时分秒信息
        LocalDateTime today = LocalDateTime.now();
        System.out.println("today = " + today);
        System.out.println("today.toLocalDate() = " + today.toLocalDate());
        System.out.println("today.toLocalTime() = " + today.toLocalTime());

        System.out.println("年分 ：  " + today.getYear());
        System.out.println("月份 ：  " + today.getMonth() + "月份数字 ： "+today.getMonthValue());
        System.out.println("日期 ：  " + today.getDayOfMonth());
        System.out.println("星期 ：  " + today.getDayOfWeek());
        System.out.println("这一年的第多少天 ：  " + today.getDayOfYear());
        System.out.println("小时 ：  " + today.getHour());
        System.out.println("分钟 ：  " + today.getMinute());
        System.out.println("秒   ：  " + today.getSecond());
        System.out.println("纳秒 ：  " + today.getNano());

        System.out.println("============================");

    }
}
```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_17,color_FFFFFF,t_70,g_se,x_16-166030051963629.png)

###### 2.3 指定LocalDateTime对象的属性信息

```
 * 3.指定日期时间对象的 年、月、日、时、分、秒、纳秒
 *      withYear(int) : 指定年分
 *      withMonth(int) : 指定月份
 *      withDayOfMonth(int) : 指定日期
 *      withDayOfYear(int) : 指定一年中的多少天
 *      withHour(int) : 指定小时
 *      withMinute(int) : 指定分钟
 *      withSecond(int) : 指定秒
 *      withNano(int) : 指定纳秒
 *
 *      with(TemporalAdjuster) : 时间矫正器
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
    
    	//3.修改指定 年月日时分秒纳秒
        LocalDateTime nowWith = LocalDateTime.now();
        System.out.println("nowWith = " + nowWith);
        System.out.println("指定年分2025 = " + nowWith.withYear(2025));
        System.out.println("指定月份6月 = " + nowWith.withMonth(6));
        System.out.println("指定日期28号 = " + nowWith.withDayOfMonth(28));
        System.out.println("指定一年的第3天 = " + nowWith.withDayOfYear(3));

        System.out.println();

        System.out.println("指定小时6点 = " + nowWith.withHour(6));
        System.out.println("指定分钟20分 = " + nowWith.withMinute(20));
        System.out.println("指定秒20秒 = " + nowWith.withSecond(20));
        System.out.println("指定纳秒 123000000 = " + nowWith.withNano(123000000));

        System.out.println();

        //时间矫正器
        TemporalAdjuster temporalAdjuster = (temporal) ->{
            System.out.println("temporal = " + temporal);
            LocalDateTime localDateTime = (LocalDateTime) temporal;
            localDateTime = localDateTime.withYear(2023)
                                        .withMonth(6)
                                        .withDayOfMonth(1);
            System.out.println("localDateTime = " + localDateTime);
            return localDateTime;
        };
        LocalDateTime withTemporalAdjuster01 = nowWith.with(temporalAdjuster);
        System.out.println("withTemporalAdjuster01 = " + withTemporalAdjuster01);

        LocalDateTime withTemporalAdjuster02 = nowWith.with(TemporalAdjusters.firstDayOfNextMonth()); // 下个月的第一天
        System.out.println("withTemporalAdjuster02 = " + withTemporalAdjuster02);

        LocalDateTime withTemporalAdjuster03 = nowWith.with(TemporalAdjusters.dayOfWeekInMonth(2, DayOfWeek.MONDAY)); // 这个月的第2个星期一
        System.out.println("withTemporalAdjuster03 = " + withTemporalAdjuster03);

        LocalDateTime withTemporalAdjuster04 = nowWith.with(TemporalAdjusters.firstInMonth(DayOfWeek.SUNDAY)); // 这个月的第一个星期天
        System.out.println("withTemporalAdjuster04 = " + withTemporalAdjuster04);

        LocalDateTime withTemporalAdjuster05 = nowWith.with(TemporalAdjusters.lastInMonth(DayOfWeek.MONDAY)); // 这个月的最后一个星期一
        System.out.println("withTemporalAdjuster05 = " + withTemporalAdjuster05);

        LocalDateTime withTemporalAdjuster06 = nowWith.with(TemporalAdjusters.next(DayOfWeek.SUNDAY)); // 下一个星期天
        System.out.println("withTemporalAdjuster06 = " + withTemporalAdjuster06);

        LocalDateTime withTemporalAdjuster07 = nowWith.with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY)); // 下一个或者当前的星期天
        System.out.println("withTemporalAdjuster07 = " + withTemporalAdjuster07);

        LocalDateTime withTemporalAdjuster08 = nowWith.with(TemporalAdjusters.previous(DayOfWeek.THURSDAY)); // 上一个星期天
        System.out.println("withTemporalAdjuster08 = " + withTemporalAdjuster08);

        LocalDateTime withTemporalAdjuster09 = nowWith.with(TemporalAdjusters.previousOrSame(DayOfWeek.THURSDAY)); // 上一个或者当前的星期天
        System.out.println("withTemporalAdjuster09 = " + withTemporalAdjuster09);

        System.out.println("============================");

    }
}
```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030051963630.png)

###### 2.4 增加/减去 年月日时分秒信息

```
 * 4.加上 或者 减去 时、分、秒、纳秒
 *         plusYears(long) : 加几年
 *         plusMonths(long) : 加几个月
 *         plusDays(long) : 加几天
 *         plusWeeks(long) : 加几个周
 *         plusHours(long) : 加几个小时
 *         plusMinutes(long) : 加几分钟
 *         plusSeconds(long) : 加几秒
 *         plusNanos(long) : 加几个纳秒
 *
 *         minusYears(long) : 减几年
 *         minusMonths(long) : 减几个月
 *         minusDays(long) : 减几天
 *         minusWeeks(long) : 减几个周
 *         minusHours(long) : 减几个小时
 *         minusMinutes(long) : 减几分钟
 *         minusSeconds(long) : 减几秒
 *         minusNanos(long) : 减几个纳秒
 *
12345678910111213141516171819
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
    
    	//4.加上 或者 减去 年、月、日、时、分、秒、纳秒
        LocalDateTime nowPlusOrMin = LocalDateTime.now();
        System.out.println("nowPlusOrMin = " + nowPlusOrMin);
        System.out.println("加两年 = " + nowPlusOrMin.plusYears(2));
        System.out.println("加2个月 = " + nowPlusOrMin.plusMonths(2));
        System.out.println("加12天 = " + nowPlusOrMin.plusDays(12));
        System.out.println("加1个周 = " + nowPlusOrMin.plusWeeks(1));
        System.out.println("加2个小时 = " + nowPlusOrMin.plusHours(2));
        System.out.println("加20分钟 = " + nowPlusOrMin.plusMinutes(20));
        System.out.println("加20秒 = " + nowPlusOrMin.plusSeconds(20));
        System.out.println("加1纳秒 = " + nowPlusOrMin.plusNanos(1));

        System.out.println();

        System.out.println("减两年 = " + nowPlusOrMin.minusYears(2));
        System.out.println("减2个月 = " + nowPlusOrMin.minusMonths(2));
        System.out.println("减12天 = " + nowPlusOrMin.minusDays(12));
        System.out.println("减1个星期 = " + nowPlusOrMin.minusWeeks(1));
        System.out.println("减2个小时 = " + nowPlusOrMin.minusHours(2));
        System.out.println("减20分钟 = " + nowPlusOrMin.minusMinutes(20));
        System.out.println("减20秒 = " + nowPlusOrMin.minusSeconds(20));
        System.out.println("减1个纳秒 = " + nowPlusOrMin.minusNanos(1));

        System.out.println("============================");

    }
}
12345678910111213141516171819202122232425262728293031323334353637
```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030051963631.png)

###### 2.5 比较两个日期时间的大小

```
 * 5.比较两个日期时间对象的大小
 *      dateTime01.isEqual(dateTime02) : 两个日期时间对象是否相等
 *      dateTime01.isAfter(dateTime02) : dateTime01 是否 比 dateTime02 晚
 *      dateTiime01.isBefore(dateTime02) : dateTime01 是否 比 dateTime02 早
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
    
		//5.比较两个对象的大小
        LocalDateTime dateTime01 = LocalDateTime.now();
        LocalDateTime dateTime02 = LocalDateTime.of(2022, 3, 22, 21, 37, 59);
        System.out.println("dateTime01 = " + dateTime01);
        System.out.println("dateTime02 = " + dateTime02);
        System.out.println("dateTime01是否与dateTime02相等 ： " + dateTime01.isEqual(dateTime02));
        System.out.println("dateTime01是否比dateTime02晚 ： " + dateTime01.isAfter(dateTime02));
        System.out.println("dateTime01是否比dateTime02早 ： " + dateTime01.isBefore(dateTime02));

        System.out.println("============================")
    }
}
```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\128603ab4fe9422ab3630736fa7eb8b8.png)

###### 2.6 LocalDateTime 对象 格式化为 字符串

```
 * 6. 日期时间对象 与 字符串 的相互转化
 *   6.1 日期时间对象 转 字符串 format() + DateTimeFormatter类
 *       String format01 = localDateTime.format(DateTimeFormatter.ISO_DATE_TIME); //2022-03-23T22:11:36.946
 *       String format02 = localDateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);//2022-03-23T22:11:36.946
 *       String format03 = localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); //2022-03-23 22:11:36
 *   6.2 字符串 转 日期时间对象 parse()
 *       LocalDateTime localDateTimeParse = LocalDateTime.parse("2022--03--23 22::07::58", DateTimeFormatter.ofPattern("yyyy--MM--dd HH::mm::ss"));
 *
```

> 代码

```java
package com.northcastle.K_Date;

import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;

public class LocalDateTimeTest {
    public static void main(String[] args) {
        //6.1 日期时间对象 转 字符串
        LocalDateTime dateTimeFormat01 = LocalDateTime.now();
        System.out.println("dateTimeFormat01 = " + dateTimeFormat01);
        String format01 = dateTimeFormat01.format(DateTimeFormatter.ISO_DATE_TIME); //2022-03-23T22:11:36.946
        System.out.println("format01(ISO_DATE_TIME) = " + format01);

        String format02 = dateTimeFormat01.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);//2022-03-23T22:11:36.946
        System.out.println("format02(ISO_LOCAL_DATE_TIME) = " + format02);

        String format03 = dateTimeFormat01.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); //2022-03-23 22:11:36
        System.out.println("format03 = " + format03);

        System.out.println();

        //6.2 字符串 转 日期时间对象
        LocalDateTime localDateTimeParse = LocalDateTime.parse("2022--03--23 22::07::58", DateTimeFormatter.ofPattern("yyyy--MM--dd HH::mm::ss"));
        System.out.println("localDateTimeParse = " + localDateTimeParse);
        System.out.println("============================");
        
    }
}
```

> 结果

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030051963632.png)



#### 1.5Instant

##### 1.说明

> `Instant` 类  是Java8 中补充的一个 时间戳类。
> 相较于 System.currentTimeMillis()获取到【毫秒】，Instant 可以更为精确的获取到【纳秒】。
> Instant 可以使用静态方法 now() 或者 of() 方法来创建一个实例对象。（案例代码中会有体现）
> Instant 类的常用API 就是获取时间戳了
>
>  * Instant 类的 getEpochSecond() : 获取的是秒
>  * Instant 类的 toEpochMilli() : 获取的是毫秒，同 System.currentTimeMillis()
>  * Instant 类的 getNano() : 获取的是纳秒，更精确了
> 同时，Instant 类还是 Java8 中 提供的新的 日期时间类LocalDateTime 与 原来的 java.util.Date 类之间转换的桥梁。

##### 2.该类的具体使用案例代码

```java
package com.northcastle.K_Date;

import java.time.Instant;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;

/**
 * Instant 类 ：存储了一个从1970-01-01 00:00:00 以来的 秒 和 纳秒 数据
 * 实际上就是一个时间戳类
 * System.currentTimeMillis() ： 获取的是毫秒
 *
 * Instant 类的 getEpochSecond() : 获取的是秒
 * Instant 类的 toEpochMilli() : 获取的是毫秒，同 System.currentTimeMillis()
 * Instant 类的 getNano() : 获取的是纳秒，更精确了
 *
 */
public class InstantTest {
    public static void main(String[] args) {
        //1.获取当前时间的Instant对象
        Instant now01 = Instant.now();
        System.out.println(now01);
        System.out.println("纪元秒 ： "+now01.getEpochSecond());
        System.out.println("时间戳 ： "+System.currentTimeMillis());
        System.out.println("毫  秒 ： "+now01.toEpochMilli());
        System.out.println("纳  秒 ： "+now01.getNano());

        System.out.println("===========================");

        // 2.获取指定时间的Instant对象
        Instant instant01 = Instant.ofEpochSecond(100);
        System.out.println(instant01);
        System.out.println("纪元秒 ： "+instant01.getEpochSecond());
        System.out.println("毫  秒 ： "+instant01.toEpochMilli());
        System.out.println("纳  秒 ： "+instant01.getNano());
        System.out.println("===========================");

        //3.指定时间戳创建 带时区的日期时间对象 ZoneDateTime
        Instant instant02 = Instant.ofEpochSecond(1647784071); // 2022-03-20 21:47:51
        ZonedDateTime zonedDateTime = instant02.atZone(ZoneId.of("Asia/Shanghai"));
        System.out.println("zonedDateTime = " + zonedDateTime);
        System.out.println("===========================");

        // 4.指定时间戳创建  默认时区的日期时间对象 LocalDateTime
        Instant instant03 = Instant.ofEpochSecond(1647784071); // 2022-03-20 21:47:51
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant03, ZoneId.systemDefault());
        System.out.println("localDateTime = " + localDateTime);
        System.out.println("===========================");

    }
}
```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030071776939.png)

##### 3.日期时间与[时间戳](https://so.csdn.net/so/search?q=时间戳&spm=1001.2101.3001.7020)之间的转换关系图

> 转换关系中涉及到了 `LocalDate`、`LocalDateTime`、`ZoneDateTime`、`Instant` 、`Date` 之间的相互转换。
> 具体的转换操作可以参考博客 ： [Java8新特性 - LocalDate、LocalDateTime、ZoneDateTime 与 Date 的相互转换](https://blog.csdn.net/qq_39505245/article/details/123768265)

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030071777040.png

#### 1.6相互转换

##### 1.Java8中的Date与各类之间的关系

```
各个类的功能说明如下 ： 
LocalDate :  Java8 日期类(默认时区，中国就是东八区)
LocalTime :  Java8 时间类(默认时区，中国就是东八区)
LocalDateTime : Java8 日期时间类(默认时区，中国就是东八区)
Instant :  Java8 时间戳类
ZoneDateTime :  Java8 带时区的日期时间类
Date : Java中的Date类，包 java.util.Date
```



> ```
> 关键的转换思路，可以根据下面的图进行理解：
> ```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_20,color_FFFFFF,t_70,g_se,x_16-166030119783143.png)

##### 3.具体的转换关系

> 下面的各个转换思路**直接看代码**，（结合上面的图更好理解）

###### 3.1 LocalDate 转 Date

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        /**
         * 1.LocalDate 转 Date
         *  思路1 ： LocalDate --> LocalDateTime --> Instant --> Date
         *  思路2 ： LocalDate --> LocalDateTime --> ZoneDateTime --> Instant --> Date
         */
        LocalDate localDate01 = LocalDate.of(2022, 3, 25);
        LocalDateTime localDateTime01 = localDate01.atStartOfDay(); // 将 时分秒 设置为0
        // 思路一 ：
        Instant instant0101 = localDateTime01.toInstant(ZoneOffset.of("+8")); //【这也是一种转换 Instant 的方法】
        Date date0101 = Date.from(instant0101);
        //思路二 ：
        ZonedDateTime zonedDateTime01 = localDateTime01.atZone(ZoneId.systemDefault());
        Instant instant0102 = zonedDateTime01.toInstant();
        Date date0102 = Date.from(instant0102);

        System.out.println("localDate01 = " + localDate01);
        System.out.println("localDateTime01 = " + localDateTime01);
        System.out.println("zonedDateTime01 = " + zonedDateTime01);
        System.out.println("instant0101 = " + instant0101);
        System.out.println("date0101 = " + sdf.format(date0101));
        System.out.println("instant0102 = " + instant0102);
        System.out.println("date0102 = " + sdf.format(date0102));
        System.out.println("=======================");
       
    }
}

1234567891011121314151617181920212223242526272829303132333435363738
```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_19,color_FFFFFF,t_70,g_se,x_16.png)

###### 3.2 LocalDateTime 转 Date

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

          /**
         * 2.LocalDateTime 转 Date
         * 思路1 ： LocalDateTime --> Instant --> Date
         * 思路2 ： LocalDateTime --> ZoneDateTime --> Instant --> Date
         */
        LocalDateTime localDateTime02 = LocalDateTime.of(2022, 3, 25, 22, 34, 30);
        // 思路一 ：
        Instant instant0201 = localDateTime02.toInstant(ZoneOffset.of("+8"));
        Date date0201 = Date.from(instant0201);
        // 思路二 ：
        ZonedDateTime zonedDateTime02 = localDateTime02.atZone(ZoneId.systemDefault());
        Instant instant0202 = zonedDateTime02.toInstant();
        Date date0202 = Date.from(instant0202);
        System.out.println("localDateTime02 = " + localDateTime02);
        System.out.println("zonedDateTime02 = " + zonedDateTime02);
        System.out.println("instant0201 = " + instant0201);
        System.out.println("date0201 = " + sdf.format(date0201));
        System.out.println("instant0202 = " + instant0202);
        System.out.println("date0202 = " + sdf.format(date0202));
        System.out.println("=======================");
    }
}

12345678910111213141516171819202122232425262728293031323334
```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATm9ydGhDYXN0bGU=,size_19,color_FFFFFF,t_70,g_se,x_16-166030119783244.png)

###### 3.3 ZoneDateTime 转 Date

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
     
        /**
         * 3.ZoneDateTime 转 Date
         * 思路 ： ZoneDateTime --> Instant --> Date
         */
        ZonedDateTime zonedDateTime03 = ZonedDateTime.of(2022, 3, 25, 22, 36, 56, 0, ZoneId.systemDefault());
        Instant instant03 = zonedDateTime03.toInstant();
        Date date03 = Date.from(instant03);
        System.out.println("zonedDateTime03 = " + zonedDateTime03);
        System.out.println("instant03 = " + instant03);
        System.out.println("date03 = " + sdf.format(date03));
        System.out.println("=======================");
    }
}

12345678910111213141516171819202122232425
```

![在这里插入图片描述](E:\Development\Typora\images\3410fcbc52144d0f8dceee3a89f42b94.png)

###### 3.4 Date 转 ZoneDateTime

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        /**
         * 4.Date 转 ZoneDateTime
         * 思路 ： Date --> Instant --> ZoneDateTime
         */
        Date date04 = new Date();
        Instant instant04 = date04.toInstant();
        ZonedDateTime zonedDateTime04 = ZonedDateTime.ofInstant(instant04, ZoneId.systemDefault());
        System.out.println("date04 = " + sdf.format(date04));
        System.out.println("instant04 = " + instant04);
        System.out.println("zonedDateTime04 = " + zonedDateTime04);
        System.out.println("=======================");
    }
}
123456789101112131415161718192021222324
```

![在这里插入图片描述](E:\Development\Typora\images\57c5e059049b436c8598984aa481907e.png)

###### 3.5 Date 转 LocalDateTime

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        /**
         * 5.Date 转 LocalDateTime
         * 思路1 ： Date --> Instant --> ZoneDateTime --> LocalDateTime
         * 思路2 ： Date --> Instant --> LocalDateTime.of(Instant,ZoneId)
         */
        Date date05 = new Date();
        Instant instant05 = date05.toInstant();
        //思路一 ： Date --> Instant --> ZoneDateTime --> LocalDateTime
        LocalDateTime localDateTime0501 = ZonedDateTime.ofInstant(instant05, ZoneId.systemDefault()).toLocalDateTime();
        //思路二 ： Date --> Instant --> LocalDateTime.of(Instant,ZoneId)
        LocalDateTime localDateTime0502 = LocalDateTime.ofInstant(instant05, ZoneId.systemDefault());

        System.out.println("date05 = " + sdf.format(date05));
        System.out.println("instant05 = " + instant05);
        System.out.println("localDateTime0501 = " + localDateTime0501);
        System.out.println("localDateTime0502 = " + localDateTime0502);
        System.out.println("=======================");
    }
}
123456789101112131415161718192021222324252627282930
```

![在这里插入图片描述](E:\Development\Typora\images\b86c81590e074f198d2539123fdbb37f.png)

###### 3.6 Date 转 LocalDate

```java
package com.northcastle.K_Date;

import java.text.SimpleDateFormat;
import java.time.*;
import java.util.Date;

public class LocalDateTimeAndDate {
    public static void main(String[] args) {
        //0.准备一个Date的格式化对象
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        /**
         * 6.Date 转 LocalDate
         * 思路1 ： Date --> Instant --> ZoneDateTime --> LocalDateTime --> LocalDate
         * 思路2 ： Date --> Instant --> LocalDateTime.of(Instant,ZoneId) --> LocalDate
         */
        Date date06 = new Date();
        Instant instant06 = date06.toInstant();
        //思路一 ： Date --> Instant --> ZoneDateTime --> LocalDateTime --> LocalDate
        LocalDate localDate0601 = ZonedDateTime.ofInstant(instant06, ZoneId.systemDefault()).toLocalDateTime().toLocalDate();
        //思路二 ： Date --> Instant --> LocalDateTime.of(Instant,ZoneId) --> LocalDate
        LocalDate localDate0602 = LocalDateTime.ofInstant(instant06, ZoneId.systemDefault()).toLocalDate();

        System.out.println("date06 = " + sdf.format(date06));
        System.out.println("instant06 = " + instant06);
        System.out.println("localDate0601 = " + localDate0601);
        System.out.println("localDate0602 = " + localDate0602);
        System.out.println("=======================");
    }
}

12345678910111213141516171819202122232425262728293031
```

![在这里插入图片描述](E:\Development\Typora\images\87a7b2bd72674206bde9a09c8cf489c3.png)



### 二.Stream

#### 简述

> Stream 是Java中对**集合**操作（过滤、查找、映射......）的一种更为简便的封装。 使用Stream可以让代码更为简洁。

1.Stream的方法分为【终结方法】和【非终结方法】 

​	【终结方法】：方法的返回值类型不再是Stream类型，不再支持链式调用，例如 forEach()方法； 

​	【非终结方法】： 方法的返回值类型仍然是Stream类型，可以继续进行链式调用，例如 filter()方法。

2.每个Stream对象，只能被使用一次，使用完成后就会被关闭，再次使用时会抛出异常。

**3.Stream对象如果不调用【终结方法】的话，中间的过程是不会执行的！（非常的关键）**

#### [（一）创建流](#)

**（1）通过集合创建流**

```java
// 通过集合创建流
List<String> lists = new ArrayList<>();
lists.stream();
```

**（2）通过数组创建流**

```java
// 通过数组创建流
String[] strings = new String[5];
Stream.of(strings);
```

应用较多的是通过集合创建流，然后经过中间操作，最后终止回集合。

**（3） Map如何获取Stream对象**

> 思路 : Map中获取 key的set集合、value的集合、entrySet的集合，然后再使用流即可
> 其实就是使用的`Collection.stream()`方法

```java
  //【Map获取Stream的方式】
    public static void getStream0101(){
        HashMap<String, Object> hashMap = new HashMap<>();
        hashMap.put("aa",100);
        hashMap.put("bb",200);
        hashMap.put("cc",300);
        //1.根据key集合获取
        Stream<String> streamKey = hashMap.keySet().stream();
        streamKey.forEach(System.out::println);
        System.out.println("==============================");

        //2.根据value集合获取
        Stream<Object> streamValues = hashMap.values().stream();
        streamValues.forEach(System.out::println);
        System.out.println("==============================");

        //3.根据 entry 集合获取
        Stream<Map.Entry<String, Object>> streamEntry = hashMap.entrySet().stream();
        streamEntry.forEach(System.out::println);
        System.out.println("==============================");

    }
```



#### [（二）中间操作](#)

##### 1.filter()

```
【方法签名】Stream<T> filter(Predicate<? super T> predicate);
【方法属性】非终结方法
【方法参数】函数式接口 Predicate , 因此可以直接传入一个Lambda表达式
【方法作用】过滤stream中的元素，返回符合条件的元素到一个新的stream中
【方法返回值】 Stream<T> 一个新的Stream对象，可以继续支持链式调用
```

##### 2.limit()

```java
【方法签名】Stream<T> limit(long maxSize);
【方法属性】非终结方法
【方法参数】long 类型的数字，表示要截取的元素个数
          必须是正整数或者0；
          如果是负数，则会抛出异常！
【方法作用】截取前maxSize个元素，返回一个新的Stream对象
【返回值】Stream<T> 截取的元素形成的Stream对象
    //2.获取Stream对象
    Stream<String> streamList = list.stream();
	//3.只取前5个元素
	streamList.limit(5)
      		  .forEach(System.out::println);
}
```

##### 3.skip()

```java
【方法签名】Stream<T> skip(long n);
【方法属性】非终结方法
【方法参数】long 类型的数字，表示要跳过的元素个数
          参数只能是正整数和0；
          如果是负数则会抛出异常！
【方法作用】跳过前几个元素，返回一个新的Stream对象
【方法返回值】 Stream<T>
```

##### 4.map()

上面有…略…



##### 5.sort()

```java
【方法签名】
          Stream<T> sorted();
          Stream<T> sorted(Comparator<? super T> comparator);
【方法属性】非终结方法
【方法参数】1.无参；2.自定义排序规则，重写Comparator接口的compara()方法
【方法作用】
		1.对stream中的元素进行自然排序（无参的时候）；
		2.对stream中的元素按自定义排序规则进行排序（重写compare方法的时候）。
【方法返回值】包含排序之后元素的新的stream流
```

具体见前面

##### 6.reduce() 规约

```java
【方法签名】
		1.Optional<T> reduce(BinaryOperator<T> accumulator);【推荐使用】
		*2.T reduce(T identity, BinaryOperator<T> accumulator);【重点理解这个】
		3.<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
		  【太复杂且涉及高并发的处理，不建议使用】
【方法属性】终结方法
【方法参数】
		重点介绍一下 T reduce(T identity, BinaryOperator<T> accumulator);
		T identity : 给定的初始值；
		BinaryOperator<T> accumulator ： 自定义的处理逻辑，可以直接是一个Lambda表达式，
【方法作用】处理Stream中的元素，将多个元素“规约”为一个元素输出
```

```java
        //0.准备一个数组
        String[] strs = {"1","5","2","3","100"};
        //1.把数组中的数据求和称为一个数据返回,不指定初始值
        Optional<Integer> reduce1 = Stream.of(strs)
                .map(Integer::parseInt)
                .reduce((x, y) -> {
                    System.out.println("x = " + x + " ; y = " + y);
                    return x + y;
                });
        System.out.println("求和结果是 ： "+reduce1.get());
        System.out.println("=================================");

        //2.把数组中的数据求和称为一个数据返回 ： 指定初始值 为 0
        Integer reduce2 = Stream.of(strs)
                .map(Integer::parseInt)
                .reduce(0, (x, y) -> {
                    System.out.println("x = "+x+" ; y = "+y);
                    return x+y;
                });
        System.out.println("求和结果是 ： "+reduce2);
        System.out.println("=================================");

        //求最大值 ： 模拟 max() 方法的实现逻辑
        Optional<Integer> max = Stream.of(strs)
                .map(Integer::parseInt)
                .reduce((x, y) -> {
                    System.out.println("x = " + x + " ; y = " + y);
                    return x > y ? x : y; // 返回一个较大的值
                });
        System.out.println("最大值是 ： "+max.get());
        System.out.println("==================================");
```



##### 7.concat()

```java
【方法签名】public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) 
【方法属性】非终结方法
【方法参数】两个类型相同的Stream对象
【方法作用】将两个Stream对象合为一个Stream
【方法返回值】 一个新的Stream对象
```

注意是将两个Stream

```java
    public static void main(String[] args) {
        //0.准备两个个数组
        Integer[] ints1 = {1,5,8,100,56};
        Integer[] ints2 = {10,50,80,1000,506};
        //1.生成两个流
        Stream<Integer> stream1 = Stream.of(ints1);
        Stream<Integer> stream2 = Stream.of(ints2);
        //2.合并两个流并进行输出
        Stream.concat(stream1,stream2).forEach(System.out::println);
    }
```















































#### （三）终止操作

##### 1.forEach()

```java
【方法签名】void forEach(Consumer<? super T> action);
【方法属性】终结方法
【方法参数】函数式接口 Consumer,所以可以直接传入一个Lambda表达式
【方法作用】遍历每一个元素
```

```java
//常规写法 ：【Lambda表达式：参数s就是集合中的元素】
streamList.forEach(s -> System.out.println(s));
// 此处直接打印集合中的元素，所以可以使用【方法调用的方式】
streamList.forEach(System.out::println);
```

##### 2.count()

```java
【方法签名】long count();
【方法属性】终结方法
【方法参数】 无参
【方法作用】返回stream中的元素个数，返回类型 long 类型
```

```java
//2.获取Stream对象
Stream<String> streamList = list.stream();
//3.使用count获取流中的元素个数
long countNum = streamList.count();
System.out.println("stream中的元素个数 : "+countNum);
```

##### 3.allMatch()、anyMatch()和noneMatch()

```java
【方法签名】
    1.boolean anyMatch(Predicate<? super T> predicate);
    2.boolean allMatch(Predicate<? super T> predicate);
    3.boolean noneMatch(Predicate<? super T> predicate);
【方法属性】终结方法
【方法参数】函数式接口 Predicate，所以可以直接传入一个Lambda表达式
【方法作用】   
	1.boolean anyMatch ： 任意一个元素满足条件，即返回true,否则返回false;
    2.boolean allMatch : 所有的元素都满足条件，才返回true,否则返回false;
    3.boolean noneMatch : 所有元素都不满足条件，才返回true,否则返回false;
【方法返回值】boolean 类型
```

```java
//0.准备一个数组
String[] strs = {"aaa","bbb","ccc","ddddd","abcdefg"};
// 1.判断所有元素是否都符合要求
boolean aa = Stream.of(strs).allMatch(s -> s.startsWith("aa"));
System.out.println("所有的元素都以 aa 开头 "+aa);
//2.判断是否存在元素符合要求
boolean aa1 = Stream.of(strs).anyMatch(s -> s.startsWith("aa"));
System.out.println("存在某个元素以 aa 开头 "+aa1);
//3.判断所有元素都不符合要求
boolean aa2 = Stream.of(strs).noneMatch(s -> s.startsWith("aa"));
System.out.println("所有的元素都不以 aa 开头 "+aa2);
```

##### 4.findFirst()、findAny()

```java
【方法签名】
		1.Optional<T> findFirst();
		2.Optional<T> findAny();
【方法属性】终结方法
【方法参数】无参
【方法作用】
		1.findFirst() : 返回流中的第一个元素；如果流是空的，则返回空；
		2.findAny() : 返回流中的任意一个元素；如果流是空的，则返回空；
		【大多数情况下，数据量不大的情况下，findAny()也会返回第一个元素，此时效果与findFirst()一致】
【方法返回值】 Optional<T> ,可以通过该类型对象的 get()方法获取内部包含的值
（Optioinal也是java8中的新特性，目的是解决空指针问题）
```

##### 5.collect()

上面有



#### Stream的并行流与安全性处理

```java
package com.northcastle.I_stream;

/**
 * author : northcastle
 * createTime:2022/3/11
 */

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.LongStream;
import java.util.stream.Stream;

/**
 * 串行流 ： 在一个线程上面执行
 * 并行流 ： 多个线程同时处理一个流
 *    parallelStream : 就是一个并行流，通过ForkJoinPool，利用多线程的方式，提高流的处理速度
 */
public class StreamParallel {

    /**
     * 1.体会串行流 : 在同一个线程内，按顺序执行
     */
    @Test
    public void test01(){
        String[] strs = {"aaa","bbb","ccc","ddd","eee","fff"};
        Stream.of(strs).map(s -> {
            System.out.println(Thread.currentThread()+" : "+s);
            return Thread.currentThread().getName()+"->"+s;
        }).count();
    }

    /**
     * 2.获取并行流的两种方式
     *   1.通过Collection接口的parallelStream() 获取一个并行流
     *   2.通过已有的串行流转换为并行流 parallel() 方法
     */
    @Test
    public void test02(){
        //方式一 ： parallelStream() 方法
        ArrayList<String> list = new ArrayList<>();
        Stream<String> parallelStream01 = list.parallelStream();
        System.out.println(parallelStream01);

        //方式二 ：串行流转化为并行流
        Stream<String> stream = list.stream();
        Stream<String> parallelStream02 = stream.parallel();
        System.out.println(parallelStream02);
    }

    /**
     * 3.并行流的使用（其实与串行流的使用方式是一样的）
     *   此时能体会到 ForkJoinPool的线程池中的线程
     */
    @Test
    public void test03(){
        String[] strs = {"aaa","bbb","ccc","ddd","eee","fff"};

        Stream.of(strs).parallel() // 串行流转换为并行流
                .map(s -> {
            System.out.println(Thread.currentThread()+" : "+s);
            return Thread.currentThread().getName()+"->"+s;
        }).count();

    }

    /*下面开始对比一下 普通的for循环、串行流、并行流 的执行效率*/

    long beginTime = 0; // 开始时间
    long endTime = 0;//结束时间
    long times = 500000000; // 运行次数
    @Before
    public void before(){
        beginTime = System.currentTimeMillis();
        System.out.println("begin time : "+beginTime);
    }

    /**
     * 普通for循环 - 374
     * begin time : 1647012437913
     * sum = 124999999750000000
     * end time : 1647012438287
     * cost : 374
     */
    @Test
    public void commonFor(){
        long sum = 0;
        for (int i = 0; i <= times ; i++) {
            sum+=i;
        }
        System.out.println("sum = "+sum);
    }

    /**
     * 串行流 - 320
     * begin time : 1647056179951
     * sum = 125000000250000000
     * end time : 1647056180271
     * cost : 320
     */
    @Test
    public void serialStream(){
        long reduce = LongStream.rangeClosed(0, times)
                .reduce(0, Long::sum);
        System.out.println("sum = "+ reduce);
    }

    /**
     * 并行流 - 239
     * begin time : 1647056314702
     * sum = 125000000250000000
     * end time : 1647056314941
     * cost : 239
     */
    @Test
    public void parallelStream(){
        long reduce = LongStream.rangeClosed(0, times)
                .parallel() // 获取一个串行的流
                .reduce(0, Long::sum);
        System.out.println("sum = "+reduce);
    }

    @After
    public void after(){
        endTime = System.currentTimeMillis();
        System.out.println("end time : "+endTime);
        System.out.println("cost : "+(endTime - beginTime));
    }

    /*上面是 普通for循环、串行流、并行流 三者的执行效率的对比，通过运行发现，在硬件强大的基础上，执行效率并不会差很多*/


    /*下面是 线程安全问题的操作*/

    /**
     * 1.体会并行流的问题
     * 使用并行流向一个集合中放入元素
     */
    @Test
    public void parallelBug(){
        ArrayList<Long> list01 = new ArrayList<>();
        ArrayList<Long> list02 = new ArrayList<>();

        int times = 1000; // 定义元素的多少
        LongStream.rangeClosed(0, times).forEach(list01::add);
        System.out.println("串行流元素数量 = "+list01.size());

        LongStream.rangeClosed(0,times).parallel().forEach(list02::add);
        System.out.println("并行流元素数量 = "+list02.size());
    }

    /**
     * 并行流的解决方案一 ： 加同步锁
     */
    @Test
    public void parallelResolve01(){

        ArrayList<Long> list01 = new ArrayList<>();
        ArrayList<Long> list02 = new ArrayList<>();

        int times = 1000; // 定义元素的多少
        LongStream.rangeClosed(0, times).forEach(list01::add);
        System.out.println("串行流元素数量 = "+list01.size());

        Object obj = new Object(); // 定义一个对象锁
        LongStream.rangeClosed(0,times)
                .parallel()
                .forEach(i ->{
                    synchronized (obj){
                        list02.add(i);
                    }
                });
        System.out.println("并行流元素数量 = "+list02.size());

    }
    /**
     * 并行流的解决方案二 ： 使用线程安全的容器
     */
    @Test
    public void parallelResolve02(){

        ArrayList<Long> list01 = new ArrayList<>();
        Vector<Long> list02 = new Vector<>();

        int times = 1000; // 定义元素的多少
        LongStream.rangeClosed(0, times).forEach(list01::add);
        System.out.println("串行流元素数量 = "+list01.size());

        LongStream.rangeClosed(0,times)
                .parallel()
                .forEach(list02::add);
        System.out.println("并行流元素数量 = "+list02.size());

    }

    /**
     * 并行流的解决方案三 ： 将线程不安全的容器 转换 为线程安全的容器
     */
    @Test
    public void parallelResolve03(){

        ArrayList<Long> list01 = new ArrayList<>();
        ArrayList<Long> list02 = new ArrayList<>();
        List<Long> synchronizedList02 = Collections.synchronizedList(list02); // 将list02 转为 线程安全的容器

        int times = 1000; // 定义元素的多少
        LongStream.rangeClosed(0, times).forEach(list01::add);
        System.out.println("串行流元素数量 = "+list01.size());

        LongStream.rangeClosed(0,times)
                .parallel()
                .forEach(synchronizedList02::add);
        System.out.println("并行流元素数量 = "+list02.size());

    }

    /**
     * 并行流的解决方案四 ：通过toArray() 或者 Collections.toList()等方法实现转换
     *
     */
    @Test
    public void parallelResolve04(){

        ArrayList<Long> list01 = new ArrayList<>();
        List<Long> list02 = new ArrayList<>();

        int times = 1000; // 定义元素的多少
        LongStream.rangeClosed(0, times).forEach(list01::add);
        System.out.println("串行流元素数量 = "+list01.size());

        list02 = LongStream.rangeClosed(0, times)
                .parallel() // 转成 并行流
                .boxed() // 把一个 long 类型转换成了 Long 封装类型的数据
                .collect(Collectors.toList());
        System.out.println("并行流元素数量 list02 = "+list02.size());

        Object[] objects = LongStream.rangeClosed(0, times)
                .parallel() // 转成 并行流
                .boxed() // 把一个 long 类型转换成了 Long 封装类型的数据
                .toArray();
        System.out.println("并行流元素数量 objects = "+objects.length);

    }

    /*上面是 线程安全问题的操作*/
}

```

