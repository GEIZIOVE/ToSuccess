# 美团分布式ID生成框架Leaf分析

# 摘要

目前主流的分布式ID生成方式，大致都是基于`数据库号段模式`和`雪花算法（snowflake）`，而美团（Leaf）刚好同时兼具了这两种方式，可以根据不同业务场景灵活切换。



# Leaf原理分析

## Snowflake生成ID的模式

7849276-4d1955394baa3c6d.png ![img](E:\Development\Typora\images\171dec925bdac001tplv-t2oaga2asx-zoom-in-crop-mark3024000.webp) `snowflake`算法对于ID的位数是上图这样分配的：

​															**1位的符号位+41位时间戳+10位workID+12位序列号**

加起来一共是64个二进制位，正好与Java中的long类型的位数一样。美团的Leaf框架对于snowflake算法进行了一些位数调整，位数分配是这样：

​															**最大41位时间差+10位的workID+12位序列化**

虽然看美团对Leaf的介绍文章里面说

```
Leaf-snowflake方案完全沿用snowflake方案的bit位设计，即是“1+41+10+12”的方式组装ID号。
```



`Leaf-snowflake`不同于原始snowflake算法地方，主要是在workId的生成上，`Leaf-snowflake`依靠`Zookeeper`生成`workId`，也就是上边的`机器ID`（占5比特）+ `机房ID`（占5比特）。`Leaf`中workId是基于ZooKeeper的`顺序Id`来生成的，每个应用在使用Leaf-snowflake时，启动时都会都在Zookeeper中生成一个顺序Id，相当于一台机器对应一个顺序节点，也就是一个workId。



其实看代码里面是没有专门设置符号位的，如果`timestamp`过大，导致时间差占用42个二进制位，时间差的第一位为1时，可能生成的id转换为十进制后会是负数：

```java
//timestampLeftShift是22，workerIdShift是12
long id = ((timestamp - twepoch) << timestampLeftShift) | (workerId << workerIdShift) | sequence;
```

#### 时间差是什么？

因为时间戳是以1970年01月01日00时00分00秒作为起始点，其实我们一般取的时间戳其实是起始点到现在的时间差，如果我们能确定我们取的时间都是某个时间点以后的时间，那么可以将时间戳的起始点改成这个时间点，L**eaf项目中，如果不设置起始时间，默认是2010年11月4日09:42:54**，这样可以使得支持的最大时间增长，Leaf框架的支持最大时间是起始点之后的69年。

#### workID怎么分配？

Leaf使用`Zookeeper`作为注册中心，每次机器启动时去Zookeeper特定路径/forever/下读取子节点列表，每个子节点存储了IP:Port及对应的workId，遍历子节点列表，如果存在当前IP:Port对应的workId，就使用节点信息中存储的workId，不存在就创建一个永久有序节点，将序号作为workId，并且将workId信息写入本地缓存文件workerID.properties，供启动时连接Zookeeper失败，读取使用。因为workId只分配了10个二进制位，所以取值范围是0-1023。

#### 序列号怎么生成？

序列号是12个二进制位，取值范围是0到4095，主要保证**同一个leaf服务**在同一毫秒内，生成的ID的唯一性。 序列号是生成流程如下：

​	 1.当前时间戳与上一个ID的时间戳在同一毫秒内，那么对sequence+1，如果sequence+1超过了4095，那么进行等待，等到下一毫秒到了之后再生成ID。

​	 2.当前时间戳与上一个ID的时间戳不在同一毫秒内，取一个100以内的随机数作为序列号。

```java
if (lastTimestamp == timestamp) {
        sequence = (sequence + 1) & sequenceMask;
        if (sequence == 0) {
            //seq 为0的时候表示是下一毫秒时间开始对seq做随机
            sequence = RANDOM.nextInt(100);
            timestamp = tilNextMillis(lastTimestamp);
        }
} else {
    //如果是新的ms开始
       sequence = RANDOM.nextInt(100);
}
lastTimestamp = timestamp;
```



`Leaf-snowflake`启动服务的过程大致如下：



- 启动Leaf-snowflake服务，连接Zookeeper，在leaf_forever父节点下检查自己是否已经注册过（是否有该顺序子节点）。
- 如果有注册过直接取回自己的workerID（zk顺序节点生成的int类型ID号），启动服务。
- 如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的workerID号，启动服务。

但`Leaf-snowflake`对Zookeeper是一种弱依赖关系，除了每次会去ZK拿数据以外，也会在本机文件系统上缓存一个`workerID`文件。一旦ZooKeeper出现问题，恰好机器出现故障需重启时，依然能够保证服务正常启动。



## segment生成ID的模式

`Leaf-segment`号段模式是对直接用`数据库自增ID`充当`分布式ID`的一种优化，减少对数据库的频率操作。相当于从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，业务服务将号段在本地生成1~1000的自增ID并加载到内存.。

![img](E:\Development\Typora\images\171dec9886f1191dtplv-t2oaga2asx-zoom-in-crop-mark3024000.webp) 

这种模式需要依赖MySQL

- `biz_tag`：针对不同业务需求，用biz_tag字段来隔离，如果以后需要扩容时，只需对biz_tag分库分表即可
- `max_id`：当前业务号段的最大值，用于计算下一个号段
- `step`：步长，也就是每次获取ID的数量
- `description`：对于业务的描述，没啥好说的

大致流程就是每个Leaf服务在内存中有两个Segment实例，每个Segement保存一个分段的ID，

- 一个`Segment`是当前用于分配ID，有一个value属性保存这个分段已分配的最大ID，以及一个max属性这个分段最大的ID。
- 另外一个`Segement`是备用的，当一个Segement用完时，会进行切换，使用另一个Segement进行使用。
- 当一个`Segement`的分段ID使用率达到10%时，就会触发另一个Segement去DB获取分段ID，初始化好分段ID供之后使用。

`Leaf`采用`双buffer`的方式，它的服务内部有两个号段缓存区`segment`。当前号段已消耗10%时，还没能拿到下一个号段，则会另启一个更新线程去更新下一个号段。

简而言之就是`Leaf`保证了总是会多缓存两个号段，即便哪一时刻数据库挂了，也会保证发号服务可以正常工作一段时间。

![图片](E:\Development\Typora\images\640.png)

通常推荐号段（`segment`）长度设置为服务高峰期发号QPS的600倍（10分钟），这样即使DB宕机，Leaf仍能持续发号10-20分钟不受影响。

**优点：**

- Leaf服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景。
- 容灾性高：Leaf服务内部有号段缓存，即使DB宕机，短时间内Leaf仍能正常对外提供服务。

**缺点：**

- ID号码不够随机，能够泄露发号数量的信息，不太安全。

- DB宕机会造成整个系统不可用（用到数据库的都有可能）。

  

由于依赖数据库，我们先设计一下表结构：

```sql
CREATE TABLE `leaf_alloc` (
  `biz_tag` varchar(128) NOT NULL DEFAULT '' COMMENT '业务key',
  `max_id` bigint(20) NOT NULL DEFAULT '1' COMMENT '当前已经分配了的最大id',
  `step` int(11) NOT NULL COMMENT '初始步长，也是动态调整的最小步长',
  `description` varchar(256) DEFAULT NULL COMMENT '业务key的描述',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '数据库维护的更新时间',
  PRIMARY KEY (`biz_tag`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

预先插入一条测试的业务数据

```sql
INSERT INTO `leaf_alloc` (`biz_tag`, `max_id`, `step`, `description`, `update_time`) VALUES ('leaf-segment-test', '0', '10', '测试', '2020-02-28 10:41:03');
```



将Leaf项目下载到本地：`https://github.com/Meituan-Dianping/Leaf`

修改一下项目中的`leaf.properties`文件，添加数据库配置

```properties
leaf.name=com.sankuai.leaf.opensource.test
leaf.segment.enable=true
leaf.jdbc.url=jdbc:mysql://127.0.0.1:3306/xin-master?useUnicode=true&characterEncoding=utf8
leaf.jdbc.username=junkang
leaf.jdbc.password=junkang
leaf.snowflake.enable=false
```

**注意**：`leaf.snowflake.enable` 与 `leaf.segment.enable` 是无法同时开启的，否则项目将无法启动。

配置相当的简单，直接启动`LeafServerApplication`后就OK了，接下来测试一下，`leaf`是基于`Http请求`的发号服务， `LeafController` 中只有两个方法，一个号段接口，一个snowflake接口，`key`就是数据库中预先插入的业务`biz_tag`。

```java
@RestController
public class LeafController {
    private Logger logger = LoggerFactory.getLogger(LeafController.class);

    @Autowired
    private SegmentService segmentService;
    @Autowired
    private SnowflakeService snowflakeService;

    /**
     * 号段模式
     * @param key
     * @return
     */
    @RequestMapping(value = "/api/segment/get/{key}")
    public String getSegmentId(@PathVariable("key") String key) {
        return get(key, segmentService.getId(key));
    }

    /**
     * 雪花算法模式
     * @param key
     * @return
     */
    @RequestMapping(value = "/api/snowflake/get/{key}")
    public String getSnowflakeId(@PathVariable("key") String key) {
        return get(key, snowflakeService.getId(key));
    }

    private String get(@PathVariable("key") String key, Result id) {
        Result result;
        if (key == null || key.isEmpty()) {
            throw new NoKeyException();
        }
        result = id;
        if (result.getStatus().equals(Status.EXCEPTION)) {
            throw new LeafServerException(result.toString());
        }
        return String.valueOf(result.getId());
    }
}
```

