# Redis入门 - 数据类型：3种特殊类型详解

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/db/nosql-redis/db-redis-data-type-special.html

## [¶](#hyperloglogs基数统计) HyperLogLogs（基数统计）

> Redis 2.8.9 版本更新了 Hyperloglog 数据结构！

### **什么是基数？**

举个例子，A = {1, 2, 3, 4, 5}， B = {3, 5, 6, 7, 9}；那么基数（**`不重复的元素`**）= 1, 2, 4, 6, 7, 9； （允许容错，即可以接受一定误差）

- **HyperLogLogs 基数统计用来解决什么问题**？

这个结构可以非常省内存的去统计各种计数，比如注册 IP 数、每日访问 IP 数、页面实时UV、在线用户数，共同好友数等。

- **它的优势体现在哪**？

一个大型的网站，每天 IP 比如有 100 万，粗算一个 IP 消耗 15 字节，那么 100 万个 IP 就是 15M。而 HyperLogLog 在 Redis 中每个键占用的内容都是 12K，理论存储近似接近 2^64 个值，不管存储的内容是什么，它一个基于基数估算的算法，只能比较准确的估算出基数，**可以使用少量固定的内存去存储并识别集合中的唯一元素**。而且这个估算的基数并不一定准确，是一个带有 0.81% 标准错误的近似值（对于可以接受一定容错的业务场景，比如IP数统计，UV等，是可以忽略不计的）。

> 总结如下：
>
> - HyperLogLog是一种算法，并非redis独有
> - 目的是做基数统计，故不是集合，不会保存元数据，只记录数量而不是数值。
> - 耗空间极小，支持输入非常体积的数据量
> - 核心是基数估算算法，主要表现为计算时内存的使用和数据合并的处理。最终数值存在一定误差
> - redis中每个hyperloglog key占用了12K的内存用于标记基数（官方文档）
> - pfadd命令并不会一次性分配12k内存，而是随着基数的增加而逐渐增加内存分配；而pfmerge操作则会将sourcekey合并后存储在12k大小的key中，这由hyperloglog合并操作的原理（两个hyperloglog合并时需要单独比较每个桶的值）可以很容易理解。
> - 误差说明：基数估计的结果是一个带有 0.81% 标准错误（standard error）的近似值。是可接受的范围
> - Redis 对 HyperLogLog 的存储进行了优化，在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时才会一次性转变成稠密矩阵，才会占用 12k 的空间

### **相关命令使用**

[pfadd 命令 -- Redis中国用户组（CRUG）](http://www.redis.cn/commands/pfadd.html)

#### `pfadd 添加`

- 影响基数估值则返回1否则返回0.若key不存在则创建
- 时间复杂度O(1)

#### `pfcount 获得基数值`

- 得到基数值，白话就叫做去重值（1，1，2，2，3）的插入pfcount得到的是3
- 可一次统计多个key
- 时间复杂度为O(N)，N为key的个数
- 返回值是一个带有 0.81% 标准错误（standard error）的近似值.

#### `pfmerge 合并多个key`

- 取多个key的并集
- 命令只会返回 OK.
- 时间复杂度为O（N）

```bash
127.0.0.1:6379> pfadd key1 a b c d e f g h i	# 创建第一组元素
(integer) 1
127.0.0.1:6379> pfcount key1					# 统计元素的基数数量
(integer) 9
127.0.0.1:6379> pfadd key2 c j k l m e g a		# 创建第二组元素
(integer) 1
127.0.0.1:6379> pfcount key2
(integer) 8
127.0.0.1:6379> pfmerge key3 key1 key2			# 合并两组：key1 key2 -> key3 并集
OK
127.0.0.1:6379> pfcount key3
(integer) 13
```

### **应用场景**

说明：

- 基数不大，数据量不大就用不上，会有点大材小用浪费空间
- 有局限性，就是只能统计基数数量，而没办法去知道具体的内容是什么
- 和bitmap相比，属于两种特定统计情况，简单来说，HyperLogLog 去重比 bitmap 方便很多
- 一般可以bitmap和hyperloglog配合使用，bitmap标识哪些用户活跃，hyperloglog计数

一般使用：

- 统计注册 IP 数
- 统计每日访问 IP 数
- 统计页面实时 UV 数
- 统计在线用户数
- 统计用户每天搜索不同词条的个数

## [¶](#bitmap-位存储) Bitmap （位存储）

> Bitmap 即位图数据结构，都是操作二进制位来进行记录，`只有0 和 1 两个状态。`

- **用来解决什么问题**？

比如：

​		统计用户信息：活跃，不活跃！ 

​		登录：未登录！ 打卡，不打卡！

 **两个状态的，都可以使用 Bitmaps**！

如果存储一年的打卡状态需要多少内存呢？ 365 天 = 365 bit 1字节 = 8bit 46 个字节左右！

- **相关命令使用**

使用bitmap 来记录 周一到周日的打卡！ 周一：1 周二：0 周三：0 周四：1 ......

```bash
127.0.0.1:6379> setbit sign 0 1
(integer) 0
127.0.0.1:6379> setbit sign 1 1
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 0
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0  
```

查看某一天是否有打卡！

```bash
127.0.0.1:6379> getbit sign 3
(integer) 1
127.0.0.1:6379> getbit sign 5
(integer) 0
```

统计操作，统计 打卡的天数！

```bash
127.0.0.1:6379> bitcount sign # 统计这周的打卡记录，就可以看到是否有全勤！
(integer) 3
```



**`只有0 和 1 两个状态。`**

## [¶](#geospatial-地理位置) geospatial (地理位置)

[Redis GEORADIUS 命令](https://redis.com.cn/commands/georadius.html)

> Redis 的 Geo 在 Redis 3.2 版本就推出了! 这个功能可以推算地理位置的信息: 两地之间的距离, 方圆几里的人

### [¶](#geoadd) geoadd

> 添加地理位置

```bash
127.0.0.1:6379> geoadd china:city 118.76 32.04 nanjing 112.55 37.86 taiyuan 123.43 41.80 shenyang
(integer) 3
127.0.0.1:6379> geoadd china:city 144.05 22.52 shengzhen 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 3
```

**规则**

两级无法直接添加，我们一般会下载城市数据(这个网址可以查询 GEO： http://www.jsons.cn/lngcode)！

- 有效的经度从-180度到180度。
- 有效的纬度从-85.05112878度到85.05112878度。

```bash
# 当坐标位置超出上述指定范围时，该命令将会返回一个错误。
127.0.0.1:6379> geoadd china:city 39.90 116.40 beijin
(error) ERR invalid longitude,latitude pair 39.900000,116.400000
```

### [¶](#geopos) geopos

> 获取指定的成员的经度和纬度

```bash
127.0.0.1:6379> geopos china:city taiyuan nanjing
1) 1) "112.54999905824661255"
   1) "37.86000073876942196"
2) 1) "118.75999957323074341"
   1) "32.03999960287850968"
```

获得当前定位, 一定是一个`坐标值!`

### [¶](#geodist) geodist

> 如果不存在, 返回空

单位如下

- m
- km
- mi 英里
- ft 英尺

```bash
127.0.0.1:6379> geodist china:city taiyuan shenyang m
"1026439.1070"
127.0.0.1:6379> geodist china:city taiyuan shenyang km
"1026.4391"
```

### [¶](#georadius) georadius

> 附近的人 ==> 获得所有附近的人的地址, 定位, 通过半径来查询

获得指定数量的人

```bash
127.0.0.1:6379> georadius china:city 110 30 1000 km	  #	以 100,30 这个坐标为中心, 寻找半径为1000km的城市  这个命令我不知道为啥我允许不了
1) "xian"
2) "hangzhou"
3) "manjing"
4) "taiyuan"
127.0.0.1:6379> georadius china:city 110 30 500 km
1) "xian"
127.0.0.1:6379> georadius china:city 110 30 500 km withdist
1) 1) "xian"
   2) "483.8340"
127.0.0.1:6379> georadius china:city 110 30 1000 km withcoord withdist count 2
1) 1) "xian"
   2) "483.8340"
   3) 1) "108.96000176668167114"
      2) "34.25999964418929977"
2) 1) "manjing"
   2) "864.9816"
   3) 1) "118.75999957323074341"
      2) "32.03999960287850968"
  
        @pdai: 代码已经复制到剪贴板
    
```

**参数 key 经度 纬度 半径 单位 [显示结果的经度和纬度] [显示结果的距离] [显示的结果的数量]**

### [¶](#georadiusbymember) georadiusbymember

> 显示与指定成员一定半径范围内的其他成员

```bash
127.0.0.1:6379> georadiusbymember china:city taiyuan 1000 km #这条命令也运行不了
1) "manjing"
2) "taiyuan"
3) "xian"
127.0.0.1:6379> georadiusbymember china:city taiyuan 1000 km withcoord withdist count 2
1) 1) "taiyuan"
   2) "0.0000"
   3) 1) "112.54999905824661255"
      2) "37.86000073876942196"
2) 1) "xian"
   2) "514.2264"
   3) 1) "108.96000176668167114"
      2) "34.25999964418929977"

```

参数与 georadius 一样

### [¶](#geohash较少使用) geohash(较少使用)

> 该命令返回11个字符的hash字符串

```bash
127.0.0.1:6379> geohash china:city taiyuan shenyang
1) "ww8p3hhqmp0"
2) "wxrvb9qyxk0"
```

将二维的经纬度转换为一维的字符串, 如果两个字符串越接近, 则距离越近

### [¶](#底层) 底层

> geo底层的实现原理实际上就是Zset, 我们可以通过Zset命令来操作geo

```bash
127.0.0.1:6379> type china:city
zset  
```

查看全部元素 删除指定的元素

```bash
127.0.0.1:6379> zrange china:city 0 -1 withscores
 1) "xian"
 2) "4040115445396757"
 3) "hangzhou"
 4) "4054133997236782"
 5) "manjing"
 6) "4066006694128997"
 7) "taiyuan"
 8) "4068216047500484"
 9) "shenyang"
1)  "4072519231994779"
2)  "shengzhen"
3)  "4154606886655324"
127.0.0.1:6379> zrem china:city manjing
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1
1) "xian"
2) "hangzhou"
3) "taiyuan"
4) "shenyang"
5) "shengzhen"
  
```