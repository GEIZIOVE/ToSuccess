# Canal

[Canal Admin QuickStart · alibaba/canal Wiki (github.com)](https://github.com/alibaba/canal/wiki/Canal-Admin-QuickStart)

[![img](E:\Development\Typora\images\2051340-20210429123231375-821264634.png)](https://img2020.cnblogs.com/blog/2051340/202104/2051340-20210429123231375-821264634.png)

**canal [kə'næl]**，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库**增量日志**解析，提供**增量数据订阅**和**消费**

早期阿里巴巴因为杭州和美国双机房部署，存在跨机房同步的业务需求，实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始，业务逐步尝试数据库日志解析获取增量变更进行同步，由此衍生出了大量的数据库增量订阅和消费业务。

基于日志增量订阅和消费的业务包括

- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护(拆分异构索引、倒排索引等)
- 业务 cache 刷新
- 带业务逻辑的增量数据处理

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

### MySQL工作原理

[![img](E:\Development\Typora\images\2051340-20210429123257853-127378223.png)](https://img2020.cnblogs.com/blog/2051340/202104/2051340-20210429123257853-127378223.png)

- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

### Canal工作原理

[![img](E:\Development\Typora\images\2051340-20210429123316952-183549569.jpg)](https://img2020.cnblogs.com/blog/2051340/202104/2051340-20210429123316952-183549569.jpg)

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)



## Canal使用准备

### MySQL准备

1、对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

```properties
[mysqld]
 # 开启binlog
binlog-format=ROW # 选择ROW模式
server_id=1 # 配置MySQL replaction需要定义，不要和canal的slaveId重复
```

2、授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, **如果已有账户可直接 grant**

```properties
-- 使用命令登录：mysql -u root -p
-- 创建mysql用户 用户名：canal 密码：canal
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

这一步可以直接使用root账户

3、重启数据库，查看配置是否生效

```mysql
mysql> show variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------
```

```mysql
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
```

### Canal使用准备

> Canal下载，这里使用1.1.5版本为例

github链接：https://github.com/alibaba/canal/releases

需要下载(其他可根据自行需要进行下载)：

- canal.deployer-1.1.5.tar.gz
- canal.adapter-1.1.5.tar.gz

> 下载后解压

```bash
tar -zxvf xxxx.tar.gz 
```

1.1.5解压后目录结构如下

```bash
bin	：canal启动、重启、停止文件
conf	：canal配置文件
lib	：canal运行所需的jar包，注意数据库驱动包版本，可手动更换
logs	：canal运行日志
plugin	：一些扩展包，如消息队列
```

> 创建数据库和表

踩坑记录：数据库名不要使用下划线，一开始用canal_test_01的方式命名，发现扫描不到数据库

## Canal-Client使用

### 1、conf\canal.properties配置文件

```properties
# canal-deployer的ip地址，不配置默认是本机
canal.ip =
# canal-deployer的端口号
canal.port = 11111
# 模式
canal.serverMode = tcp
# 加载多个配置文件，需在canal.deployer-1.1.5\conf下创建对应的文件夹，并加上instance.properties配置文件
canal.destinations = example
# 配置多个使用英文逗号隔开
# canal.destinations = example,example2
```

### 2、conf\example\instance.properties配置文件

```properties
# 不能与my.ini下的server_id=1相同
canal.instance.mysql.slaveId=1234

# mysql地址和端口
canal.instance.master.address=127.0.0.1:3306

# 数据库账号密码，可以使用root账号或刚创建的canal账号
canal.instance.dbUsername=root
canal.instance.dbPassword=root
canal.instance.connectionCharset=UTF-8

# 配置监听，支持正则表达式
canal.instance.filter.regex=canal01\\..*
# 配置不监听，支持正则表达式
canal.instance.filter.black.regex=mysql\\.slave_.*
#################################################
## mysql serverId , v1.0.26+ will autoGen
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false
---------------------------------------------------
# position info
canal.instance.master.address=106.53.124.182:3306
canal.instance.master.journal.name=
canal.instance.master.position=154
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=root
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#################################################
```

### 3、检查lib目录下的jar包，一定要有对应的数据库驱动包，且版本要对得上

### 4、在bin目录中启动canal-deployer，启动成功如下输出

- 启动

  ```bash
  sh bin/startup.sh
  ```

- 查看 server 日志

  ```bash
  vi logs/canal/canal.log
  ```

  ```bash
  2013-02-05 22:45:27.967 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.
  2013-02-05 22:45:28.113 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.1.29.120:11111]
  2013-02-05 22:45:28.210 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......
  ```

- 查看 instance 的日志

  ```bash
  vi logs/example/example.log
  ```

  ```bash
  2013-02-05 22:50:45.636 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
  2013-02-05 22:50:45.641 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
  2013-02-05 22:50:45.803 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
  2013-02-05 22:50:45.810 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start successful....
  ```

- 关闭

  ```
  sh bin/stop.sh
  ```

```bash
# 运行后输出
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; 
support was removed in 8.0 Listening for transport dt_socket at address: 9099

# 在logs\canal目录下，会记录canal.log
# 有如下记录表示canal-deployer运行成功
2021-04-28 09:26:57.194 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2021-04-28 09:26:57.244 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2021-04-28 09:26:57.257 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2021-04-28 09:26:57.399 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[192.168.3.51(192.168.3.51):11111]
2021-04-28 09:26:59.440 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......

# 在logs\example目录下，会记录example.log
# 有如下记录表示canal-deployer与数据库连接成功了
2021-04-28 09:30:37.159 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
2021-04-28 09:30:37.175 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^fooddb\..*$
2021-04-28 09:30:37.175 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter : ^mysql\.slave_.*$
2021-04-28 09:30:37.270 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2021-04-28 09:30:37.288 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
2021-04-28 09:30:37.305 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just last position
 {"identity":{"slaveId":-1,"sourceAddress":{"address":"ieonline.microsoft.com","port":3306}},"postion":{"gtid":"","included":false,"journalName":"binlog.000068","position":14636,"serverId":1,"timestamp":1619408047000}}
2021-04-28 09:30:37.744 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=binlog.000068,position=14636,serverId=1,gtid=,timestamp=1619408047000] cost : 466ms , the next step is binlog dump
```

### 5、创建springboot工程监听TestCanal

```java
package com.example.springbootcanal.test;


import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.protocol.CanalEntry;
import com.alibaba.otter.canal.protocol.CanalEntry.Column;
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;
import com.alibaba.otter.canal.protocol.CanalEntry.EventType;
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;
import com.alibaba.otter.canal.protocol.Message;
import com.google.protobuf.InvalidProtocolBufferException;

import java.net.InetSocketAddress;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.ConcurrentLinkedQueue;
/**
 * @Auther: fzl
 * @Date: 2020/4/20 01:21
 * @Description:
 */
public class TestCanal {

    private static Queue<String> SQL_QUEUE = new ConcurrentLinkedQueue<>(); // SQL队列

    public static void main(String[] args) {
        //获取canalServer连接：本机地址,端口号
        CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress("106.53.124.182",
                11111), "example", "", "");
        int batchSize = 1000;
        try {
            //连接canalServer
            connector.connect();
            //订阅Desctinstion
            connector.subscribe();
//            //订阅数据库表,全部表
//            connector.subscribe(".*\\..*");
            connector.rollback();//回滚到未进行ack的地方，下次fetch的时候，可以从最后一个没有ack的地方开始拿
            try {
                while (true) {
                    //尝试从master那边拉去数据batchSize条记录，有多少取多少
                    //轮询拉取数据   上面的where
                    Message message = connector.getWithoutAck(batchSize);
                    long batchId = message.getId(); //获取消息ID
                    int size = message.getEntries().size(); //获取拉取的数据条数
                    if (batchId == -1 || size == 0) {
                        //睡眠
                        Thread.sleep(2000); //每次睡眠2秒
                    } else {
                        dataHandle(message.getEntries()); //如果有数据,处理数据
                    }
                    connector.ack(batchId); //进行 batch id 的确认。确认之后，小于等于此 batchId 的 Message 都会被确认
                    System.out.println("a"+size);
                    //当队列里面堆积的sql大于一定数值的时候就模拟执行
                    if (SQL_QUEUE.size() >= 3) {
                        executeQueueSql();
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (InvalidProtocolBufferException e) {
                e.printStackTrace();
            }
        } finally {
            connector.disconnect();
        }


    }

    /**
     * 模拟执行队列里面的sql语句
     */
    public static void executeQueueSql() {
        int size = SQL_QUEUE.size();
        for (int i = 0; i < size; i++) {
            String sql = SQL_QUEUE.poll(); //取出队列里面的sql
            System.out.println("[sql]----> " + sql);
        }
    }

    /**
     * 数据处理
     *
     * @param entrys
     */
    private static void dataHandle(List<CanalEntry.Entry> entrys) throws InvalidProtocolBufferException {
        for (CanalEntry.Entry entry : entrys) {
            if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                //开启/关闭事务的实体类型，跳过
                continue;
            }
            //RowChange对象，包含了一行数据变化的所有特征
            //比如isDdl 是否是ddl变更操作 sql 具体的ddl sql beforeColumns afterColumns 变更前后的数据字段等等
            RowChange rowChage;
            try {
                rowChage = RowChange.parseFrom(entry.getStoreValue()); //解析存储的数据
            } catch (Exception e) {
                throw new RuntimeException("ERROR ## parser of eromanga-event has an error , data:" + entry.toString(), e);
            }
            //获取操作类型：insert/update/delete类型
            EventType eventType = rowChage.getEventType();
            //打印Header信息
            System.out.println(String.format("================》; binlog[%s:%s] , name[%s,%s] , eventType : %s",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(),
                    eventType));
            //判断是否是DDL语句
            if (rowChage.getIsDdl()) {
                System.out.println("================》;isDdl: true,sql:" + rowChage.getSql());
            }
            //获取RowChange对象里的每一行数据，打印出来
            for (CanalEntry.RowData rowData : rowChage.getRowDatasList()) {
                //如果是删除语句
                if (eventType == EventType.DELETE) {
                    saveDeleteSql(entry);
                    printColumn(rowData.getBeforeColumnsList());
                    //如果是新增语句
                } else if (eventType == EventType.INSERT) {
                    saveInsertSql(entry);
                    printColumn(rowData.getAfterColumnsList());
                    //如果是更新的语句
                } else if (eventType == EventType.UPDATE) {
                    saveUpdateSql(entry);
                }else {
                    //变更前的数据
                    System.out.println("------->; before");
                    printColumn(rowData.getBeforeColumnsList());
                    //变更后的数据
                    System.out.println("------->; after");
                    printColumn(rowData.getAfterColumnsList());
                }
            }
        }

    }

    /**
     * 保存更新语句
     *
     * @param entry
     */
    private static void saveUpdateSql(CanalEntry.Entry entry) {
        try {
            RowChange rowChange = RowChange.parseFrom(entry.getStoreValue()); //解析存储的数据
            List<CanalEntry.RowData> rowDatasList = rowChange.getRowDatasList();  //获取RowChange对象里的每一行数据
            for (CanalEntry.RowData rowData : rowDatasList) {
                List<Column> newColumnList = rowData.getAfterColumnsList();  //获取变更后的数据
                StringBuffer sql = new StringBuffer("update " + entry.getHeader().getSchemaName() + "." + entry.getHeader().getTableName() + " set ");
                for (int i = 0; i < newColumnList.size(); i++) { //遍历变更后的数据
                    sql.append(" " + newColumnList.get(i).getName()
                            + " = '" + newColumnList.get(i).getValue() + "'");
                    if (i != newColumnList.size() - 1) {
                        sql.append(",");
                    }
                }
                sql.append(" where ");
                List<Column> oldColumnList = rowData.getBeforeColumnsList(); //获取变更前的数据
                for (Column column : oldColumnList) {
                    if (column.getIsKey()) {
                        //暂时只支持单一主键
                        sql.append(column.getName() + "=" + column.getValue());
                        break;
                    }
                }
                SQL_QUEUE.add(sql.toString());
            }
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
    }

    /**
     * 保存删除语句
     *
     * @param entry
     */
    private static void saveDeleteSql(CanalEntry.Entry entry) {
        try {
            RowChange rowChange = RowChange.parseFrom(entry.getStoreValue());
            List<CanalEntry.RowData> rowDatasList = rowChange.getRowDatasList();
            for (CanalEntry.RowData rowData : rowDatasList) {
                List<Column> columnList = rowData.getBeforeColumnsList();
                StringBuffer sql = new StringBuffer("delete from " + entry.getHeader().getSchemaName() + "." + entry.getHeader().getTableName() + " where ");
                for (Column column : columnList) {
                    if (column.getIsKey()) {
                        //暂时只支持单一主键
                        sql.append(column.getName() + "=" + column.getValue());
                        break;
                    }
                }
                SQL_QUEUE.add(sql.toString());
            }
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
    }

    /**
     * 保存插入语句
     *
     * @param entry
     */
    private static void saveInsertSql(CanalEntry.Entry entry) {
        try {
            RowChange rowChange = RowChange.parseFrom(entry.getStoreValue());
            List<CanalEntry.RowData> rowDatasList = rowChange.getRowDatasList();
            for (CanalEntry.RowData rowData : rowDatasList) {
                List<Column> columnList = rowData.getAfterColumnsList();
                StringBuffer sql = new StringBuffer("insert into " + entry.getHeader().getSchemaName() + "." + entry.getHeader().getTableName() + " (");
                for (int i = 0; i < columnList.size(); i++) {
                    sql.append(columnList.get(i).getName());
                    if (i != columnList.size() - 1) {
                        sql.append(",");
                    }
                }
                sql.append(") VALUES (");
                for (int i = 0; i < columnList.size(); i++) {
                    sql.append("'" + columnList.get(i).getValue() + "'");
                    if (i != columnList.size() - 1) {
                        sql.append(",");
                    }
                }
                sql.append(")");
                SQL_QUEUE.add(sql.toString());
            }
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
    }

    private static void printColumn(List<Column> columns) {
        for (Column column : columns) {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
        }
    }



}
```

## ClientAPI

### 类设计

在了解具体API之前，需要提前了解下canal client的类设计，这样才可以正确的使用好canal.


![img](E:\Development\Typora\images\687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303039302f363435332f39326233343335302d323566632d333162332d626361362d3865326131653763356532322e6a7067.jpeg)

大致分为几部分：

- ClientIdentity
  canal client和server交互之间的身份标识，目前clientId写死为1001. (目前canal server上的一个instance只能有一个client消费，clientId的设计是为1个instance多client消费模式而预留的，暂时不需要理会)
- CanalConnector
  SimpleCanalConnector/ClusterCanalConnector : 两种connector的实现，simple针对的是简单的ip直连模式，cluster针对多ip的模式，可依赖CanalNodeAccessStrategy进行failover控制
- CanalNodeAccessStrategy
  SimpleNodeAccessStrategy/ClusterNodeAccessStrategy：两种failover的实现，simple针对给定的初始ip列表进行failover选择，cluster基于zookeeper上的cluster节点动态选择正在运行的canal server.
- ClientRunningMonitor/ClientRunningListener/ClientRunningData
  client running相关控制，主要为解决client自身的failover机制。canal client允许同时启动多个canal client，通过running机制，可保证只有一个client在工作，其他client做为冷备. 当运行中的client挂了，running会控制让冷备中的client转为工作模式，这样就可以确保canal client也不会是单点. 保证整个系统的高可用性.

javadoc查看：

- CanalConnector ：http://alibaba.github.io/canal/apidocs/1.0.13/com/alibaba/otter/canal/client/CanalConnector.html

### server/client交互协议

![img](E:\Development\Typora\images\687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303039302f363437302f37646537383537632d653362352d333061352d386337662d3936356536356639616137382e6a7067.jpeg)

具体的网络协议格式，可参见：[CanalProtocol.proto](https://github.com/alibaba/canal/blob/master/protocol/src/main/java/com/alibaba/otter/canal/protocol/CanalProtocol.proto)

get/ack/rollback协议介绍：

- Message getWithoutAck(int batchSize)，允许指定batchSize，一次可以获取多条，每次返回的对象为Message，包含的内容为：
  a. batch id 唯一标识
  b. entries 具体的数据对象，可参见下面的数据介绍
- getWithoutAck(int batchSize, Long timeout, TimeUnit unit)，相比于getWithoutAck(int batchSize)，允许设定获取数据的timeout超时时间
  a. 拿够batchSize条记录或者超过timeout时间
  b. timeout=0，阻塞等到足够的batchSize
- void rollback(long batchId)，顾命思议，回滚上次的get请求，重新获取数据。基于get获取的batchId进行提交，避免误操作
- void ack(long batchId)，顾命思议，确认已经消费成功，通知server删除数据。基于get获取的batchId进行提交，避免误操作

canal的get/ack/rollback协议和常规的jms协议有所不同，允许get/ack异步处理，比如可以连续调用get多次，后续异步按顺序提交ack/rollback，项目中称之为流式api.

流式api设计的好处：

- get/ack异步化，减少因ack带来的网络延迟和操作成本 (99%的状态都是处于正常状态，异常的rollback属于个别情况，没必要为个别的case牺牲整个性能)
- get获取数据后，业务消费存在瓶颈或者需要多进程/多线程消费时，可以不停的轮询get数据，不停的往后发送任务，提高并行化. (作者在实际业务中的一个case：业务数据消费需要跨中美网络，所以一次操作基本在200ms以上，为了减少延迟，所以需要实施并行化)

流式api设计：

![img](E:\Development\Typora\images\687474703a2f2f646c2e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038302f333330302f37663739383665352d643863362d336131372d383739362d3731313636653662633264632e6a7067.jpeg)

- 每次get操作都会在meta中产生一个mark，mark标记会递增，保证运行过程中mark的唯一性
- 每次的get操作，都会在上一次的mark操作记录的cursor继续往后取，如果mark不存在，则在last ack cursor继续往后取
- 进行ack时，需要按照mark的顺序进行数序ack，不能跳跃ack. ack会删除当前的mark标记，并将对应的mark位置更新为last ack cursor
- 一旦出现异常情况，客户端可发起rollback情况，重新置位：删除所有的mark, 清理get请求位置，下次请求会从last ack cursor继续往后取

流式api带来的异步响应模型：
![img](E:\Development\Typora\images\687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303039302f363437392f35323062313532392d366665612d333932372d393635362d3333363661316661333337622e6a7067.jpeg)

### [EntryProtocol.proto](https://github.com/alibaba/canal/blob/master/protocol/src/main/java/com/alibaba/otter/canal/protocol/EntryProtocol.proto)

```
Entry  
    Header  
        logfileName [binlog文件名]  
        logfileOffset [binlog position]  
        executeTime [binlog里记录变更发生的时间戳,精确到秒]  
        schemaName   
        tableName  
        eventType [insert/update/delete类型]  
    entryType   [事务头BEGIN/事务尾END/数据ROWDATA]  
    storeValue  [byte数据,可展开，对应的类型为RowChange]  
RowChange
isDdl       [是否是ddl变更操作，比如create table/drop table]
sql         [具体的ddl sql]
rowDatas    [具体insert/update/delete的变更数据，可为多条，1个binlog event事件可对应多条变更，比如批处理]
beforeColumns [Column类型的数组，变更前的数据字段]
afterColumns [Column类型的数组，变更后的数据字段]
Column
index
sqlType     [jdbc type]
name        [column name]
isKey       [是否为主键]
updated     [是否发生过变更]
isNull      [值是否为null]
value       [具体的内容，注意为string文本]  
```

说明：

- 可以提供数据库变更前和变更后的字段内容，针对binlog中没有的name,isKey等信息进行补全
- 可以提供ddl的变更语句
- insert只有after columns, delete只有before columns，而update则会有before / after columns数据.



## SpringBoot整合Canal +RabbitMQ

### 1.RabbitMQ 队列创建

这一步可以在Java客户端通过`@Bean`创建

- 添加交换机 canal.exchange
- 添加队列 canal.queue
- 队列绑定交换机

### 2.Canal 配置和启动

这里使用V1.1.5版本

#### Canal Server下载

- 官方文档：https://github.com/alibaba/canal/wiki
- 项目地址：https://github.com/alibaba/canal
- 下载地址：https://github.com/alibaba/canal/releases

进入下载地址，选择 [canal.deployer-1.1.5.tar.gz](https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.deployer-1.1.5.tar.gz)

![youlai-mall](E:\Development\Typora\images\5b6f141e606c48d5982bef363bc9aa16.png#pic_center)



#### Canal Server配置

需要配置的东西就两项，一个是监听数据库配置，另一个是 RabbitMQ 连接配置。

改动的两个文件分别是 Canal 配置文件 `canal.properties` 和 实例配置文件 `instance.properties`

㊙️：一个 Server 可以配置多个实例监听 ，Canal 功能默认自带的有个 example 实例，本篇就用 example 实例 。如果增加实例，复制 example 文件夹内容到同级目录下，然后在 `canal.properties` 指定添加实例的名称。



- **canal.properties**

  配置 Canal 服务方式为 RabbitMQ 和连接配置(🏷️ 只列出需要修改的地方)

  ```properties
  # tcp, kafka, rocketMQ, rabbitMQ
  canal.serverMode = rabbitMQ
  ##################################################
  ######### 		    RabbitMQ	     #############
  ##################################################
  rabbitmq.host = 106.53.124.182
  rabbitmq.virtual.host =/
  rabbitmq.exchange =canal.exchange
  rabbitmq.username =guest
  rabbitmq.password =guest
  rabbitmq.deliveryMode =
  ```

- **instance.properties**

  监听数据库配置(🏷️ 只列出需要修改的地方)

  ```properties
  # position info
  canal.instance.master.address=106.53.124.182:3306
  # username/password
  canal.instance.dbUsername=root
  canal.instance.dbPassword=root
  # mq config
  canal.mq.topic=canal.routing.key
  ```

### 3. SpringBoot 整合 Canal + RabbitMQ

🏠 完整源码：https://gitee.com/youlaitech/youlai-mall

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### RabbitMQ连接配置

```yaml
spring:
  rabbitmq:
    host: 106.53.124.182
    port: 5672
    username: root
    password: root
```



#### RabbitMQ 监听同步缓存

```java
/**
 * Canal + RabbitMQ 监听数据库数据变化
 *
 * @author <a href="mailto:xianrui0365@163.com">haoxr</a>
 * @date 2021/11/4 23:14
 */
@Component
@Slf4j
@RequiredArgsConstructor
public class CanalListener {

    private final ISysPermissionService permissionService;
    private final ISysOauthClientService oauthClientService;
    private final ISysMenuService menuService;

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue(value = "canal.queue", durable = "true"),
                    exchange = @Exchange(value = "canal.exchange"),
                    key = "canal.routing.key"
            )
    })
    public void handleDataChange(@Payload CanalMessage message) {
        String tableName = message.getTable();

        log.info("Canal 监听 {} 发生变化；明细：{}", tableName, message);

        if ("sys_oauth_client".equals(tableName)) {
            log.info("======== 清除客户端信息缓存 ========");
            oauthClientService.cleanCache();
        } else if (Arrays.asList("sys_permission", "sys_role", "sys_role_permission").contains(tableName)) {
            log.info("======== 刷新角色权限缓存 ========");
            permissionService.refreshPermRolesRules();
        } else if (Arrays.asList("sys_menu", "sys_role", "sys_role_menu").contains(tableName)) {
            log.info("======== 清理菜单路由缓存 ========");
            menuService.cleanCache();
        }
    }
}

```



### 4. 实战测试



#### 启动 Canal

切换到项目的 `cd /bin` 目录下，输入 `startup` 启动 Canal

### 踩坑记录

妈的转换格式弄了我一年！！！！！

先定义一个DTO

```java
@NoArgsConstructor
@Data
public class CanalDataDTO<T> {

    private String type; // 类型 比如update、insert、delete
    private String table; // 表名
    private List<T> data; //
    private String database; // 数据库名称
    private Long es; // 时间戳
    private Integer id; // id
    private Boolean isDdl; // 是否是DDL操作
    private List<T> old; // 原始数据
    private List<String> pkNames; // 主键名称
    private String sql; // sql语句
    private Long ts; // 时间戳
}
```



然后

```java
/**
 * maxwell监听数据
 *
 * @author yezhiqiu
 * @date 2021/08/02
 */
@Component
@RabbitListener(queues = CANAL_QUEUE)
public class CanalConsumer {

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @RabbitHandler
    public void process(byte[] data) {
        Object obj = JSON.parse(new String(data, StandardCharsets.UTF_8)); //解析消息体
        CanalDataDTO<JSONObject> canalDataDTO = new CanalDataDTO<>();
        BeanUtil.copyProperties(obj, canalDataDTO); //拷贝数据
        List<JSONObject> newData = canalDataDTO.getData();
        List<JSONObject> oldData = canalDataDTO.getOld();
        String type = canalDataDTO.getType();
        if (type.equals("UPDATE")) {
            for (int i = 0; i < newData.size(); i++) {
                Appeal newAppeal = JSONObject.parseObject(newData.get(i).toJSONString(), Appeal.class);
                Appeal oldAppeal = JSONObject.parseObject(oldData.get(i).toJSONString(), Appeal.class);
                if (newAppeal.getAuditStatus() == 1 && oldAppeal.getAuditStatus() == 0) { //审核通过
                    String userId = newAppeal.getUserId();
                    //根据userId查询用户邮箱
                    User user = userMapper.selectOne(Wrappers.lambdaQuery(User.class).eq(User::getUserId, userId));
                    String email = user.getEmail();
                    if (StringUtils.isNotBlank(email)) {
                        // 发送消息
                        EmailDTO emailDTO = new EmailDTO();
                            emailDTO.setEmail(email);
                            emailDTO.setSubject("审核提醒");
                            emailDTO.setContent("您的密码申诉审核已经通过");
                        rabbitTemplate.convertAndSend(EMAIL_EXCHANGE, "*", new Message(JSON.toJSONBytes(emailDTO), new MessageProperties()));
                        }

                    }

                }
            }
        }
    }
```

Q2: I set filters for the table but it doesn't work.

A2:

1. Please step to [AdminGuide](https://github.com/alibaba/canal/wiki/AdminGuide) for the details of canal.instance.filter.regex.

   ```
   MySQL parses the table and Perl supports regex.
   Use comma(,) for multiple regex, double backslash(\\) for escape characters.
   Examples:
   1. All the tables: .* or .*\\..*
   2. All the tables in canal scheme: canal\\..*
   3. Tables starting with canal in canal scheme: canal\\.canal.*
   4. Access a table in canal scheme: canal.test1
   5. A combination of multiple rules: canal\\..*, mysql.test1, mysql.test2 (with comma as delimeter)
   ```



## Spring配置

spring配置的原理是将整个配置抽象为两部分：

- xxxx-instance.xml  (canal组件的配置定义，可以在多个instance配置中共享)
- xxxx.properties  (每个instance通道都有各自一份定义，因为每个mysql的ip，帐号，密码等信息不会相同)

通过spring的PropertyPlaceholderConfigurer通过机制将其融合，生成一份instance实例对象，每个instance对应的组件都是相互独立的，互不影响

 

### properties配置文件

properties配置分为两部分：

- canal.properties  (系统根配置文件)
- instance.properties  (instance级别的配置文件，每个instance一份)

#### **canal.properties介绍：**

 

canal配置主要分为两部分定义：

\1.  instance列表定义 (列出当前server上有多少个instance，每个instance的加载方式是spring/manager等)     

| 参数名字                                                     | 参数说明                                                     | 默认值                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| canal.destinations                                           | 当前server上部署的instance列表                               | 无                                                           |
| canal.conf.dir                                               | conf/目录所在的路径                                          | ../conf                                                      |
| canal.auto.scan                                              | 开启instance自动扫描 如果配置为true，canal.conf.dir目录下的instance配置变化会自动触发： a. instance目录新增： 触发instance配置载入，lazy为true时则自动启动 b. instance目录删除：卸载对应instance配置，如已启动则进行关闭 c. instance.properties文件变化：reload instance配置，如已启动自动进行重启操作 | true                                                         |
| canal.auto.scan.interval                                     | instance自动扫描的间隔时间，单位秒                           | 5                                                            |
| canal.instance.global.mode                                   | 全局配置加载方式                                             | spring                                                       |
| canal.instance.global.lazy                                   | 全局lazy模式                                                 | false                                                        |
| canal.instance.global.manager.address                        | 全局的manager配置方式的链接信息                              | 无                                                           |
| canal.instance.global.spring.xml                             | 全局的spring配置方式的组件文件                               | classpath:spring/memory-instance.xml   (spring目录相对于canal.conf.dir) |
| canal.instance.example.mode canal.instance.example.lazy canal.instance.example.spring.xml ..... | instance级别的配置定义，如有配置，会自动覆盖全局配置定义模式 命名规则：canal.instance.{name}.xxx | 无                                                           |
| canal.instance.tsdb.spring.xml                               | v1.0.25版本新增,全局的tsdb配置方式的组件文件                 | classpath:spring/tsdb/h2-tsdb.xml (spring目录相对于canal.conf.dir) |

 

 

\2.  common参数定义，比如可以将instance.properties的公用参数，抽取放置到这里，这样每个instance启动的时候就可以共享.  【instance.properties配置定义优先级高于canal.properties】

| 参数名字                                   | 参数说明                                                     | 默认值                                                       |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| canal.id                                   | 每个canal server实例的唯一标识，暂无实际意义                 | 1                                                            |
| canal.ip                                   | canal server绑定的本地IP信息，如果不配置，默认选择一个本机IP进行启动服务 | 无                                                           |
| canal.register.ip                          | canal server注册到外部zookeeper、admin的ip信息 (针对docker的外部可见ip) | 无                                                           |
| canal.port                                 | canal server提供socket服务的端口                             | 11111                                                        |
| canal.zkServers                            | canal server链接zookeeper集群的链接信息 例子：10.20.144.22:2181,10.20.144.51:2181 | 无                                                           |
| canal.zookeeper.flush.period               | canal持久化数据到zookeeper上的更新频率，单位毫秒             | 1000                                                         |
| canal.instance.memory.batch.mode           | canal内存store中数据缓存模式 1. ITEMSIZE : 根据buffer.size进行限制，只限制记录的数量 2. MEMSIZE : 根据buffer.size  * buffer.memunit的大小，限制缓存记录的大小 | MEMSIZE                                                      |
| canal.instance.memory.buffer.size          | canal内存store中可缓存buffer记录数，需要为2的指数            | 16384                                                        |
| canal.instance.memory.buffer.memunit       | 内存记录的单位大小，默认1KB，和buffer.size组合决定最终的内存使用大小 | 1024                                                         |
| canal.instance.transactionn.size           | 最大事务完整解析的长度支持 超过该长度后，一个事务可能会被拆分成多次提交到canal store中，无法保证事务的完整可见性 | 1024                                                         |
| canal.instance.fallbackIntervalInSeconds   | canal发生mysql切换时，在新的mysql库上查找binlog时需要往前查找的时间，单位秒 说明：mysql主备库可能存在解析延迟或者时钟不统一，需要回退一段时间，保证数据不丢 | 60                                                           |
| canal.instance.detecting.enable            | 是否开启心跳检查                                             | false                                                        |
| canal.instance.detecting.sql               | 心跳检查sql                                                  | insert into retl.xdual values(1,now()) on duplicate key update x=now() |
| canal.instance.detecting.interval.time     | 心跳检查频率，单位秒                                         | 3                                                            |
| canal.instance.detecting.retry.threshold   | 心跳检查失败重试次数                                         | 3                                                            |
| canal.instance.detecting.heartbeatHaEnable | 心跳检查失败后，是否开启自动mysql自动切换 说明：比如心跳检查失败超过阀值后，如果该配置为true，canal就会自动链到mysql备库获取binlog数据 | false                                                        |
| canal.instance.network.receiveBufferSize   | 网络链接参数，SocketOptions.SO_RCVBUF                        | 16384                                                        |
| canal.instance.network.sendBufferSize      | 网络链接参数，SocketOptions.SO_SNDBUF                        | 16384                                                        |
| canal.instance.network.soTimeout           | 网络链接参数，SocketOptions.SO_TIMEOUT                       | 30                                                           |
| canal.instance.filter.druid.ddl            | 是否使用druid处理所有的ddl解析来获取库和表名                 | true                                                         |
| canal.instance.filter.query.dcl            | 是否忽略dcl语句                                              | false                                                        |
| canal.instance.filter.query.dml            | 是否忽略dml语句 (mysql5.6之后，在row模式下每条DML语句也会记录SQL到binlog中,可参考[MySQL文档](https://dev.mysql.com/doc/refman/5.6/en/replication-options-binary-log.html#sysvar_binlog_rows_query_log_events)) | false                                                        |
| canal.instance.filter.query.ddl            | 是否忽略ddl语句                                              | false                                                        |
| canal.instance.filter.table.error          | 是否忽略binlog表结构获取失败的异常(主要解决回溯binlog时,对应表已被删除或者表结构和binlog不一致的情况) | false                                                        |
| canal.instance.filter.rows                 | 是否dml的数据变更事件(主要针对用户只订阅ddl/dcl的操作)       | false                                                        |
| canal.instance.filter.transaction.entry    | 是否忽略事务头和尾,比如针对写入kakfa的消息时，不需要写入TransactionBegin/Transactionend事件 | false                                                        |
| canal.instance.binlog.format               | 支持的binlog format格式列表 (otter会有支持format格式限制)    | ROW,STATEMENT,MIXED                                          |
| canal.instance.binlog.image                | 支持的binlog image格式列表 (otter会有支持format格式限制)     | FULL,MINIMAL,NOBLOB                                          |
| canal.instance.get.ddl.isolation           | ddl语句是否单独一个batch返回(比如下游dml/ddl如果做batch内无序并发处理,会导致结构不一致) | false                                                        |
| canal.instance.parser.parallel             | 是否开启binlog并行解析模式(串行解析资源占用少,但性能有瓶颈, 并行解析可以提升近2.5倍+) | true                                                         |
| canal.instance.parser.parallelBufferSize   | binlog并行解析的异步ringbuffer队列 (必须为2的指数)           | 256                                                          |
| canal.instance.tsdb.enable                 | 是否开启tablemeta的tsdb能力                                  | true                                                         |
| canal.instance.tsdb.dir                    | 主要针对h2-tsdb.xml时对应h2文件的存放目录,默认为conf/xx/h2.mv.db | ${canal.file.data.dir:../conf}/${canal.instance.destination:} |
| canal.instance.tsdb.url                    | jdbc url的配置(h2的地址为默认值，如果是mysql需要自行定义)    | jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL; |
| canal.instance.tsdb.dbUsername             | jdbc url的配置(h2的地址为默认值，如果是mysql需要自行定义)    | canal                                                        |
| canal.instance.tsdb.dbPassword             | jdbc url的配置(h2的地址为默认值，如果是mysql需要自行定义)    | canal                                                        |
| canal.instance.rds.accesskey               | aliyun账号的ak信息(如果不需要在本地binlog超过18小时被清理后自动下载oss上的binlog，可以忽略该值 | 无                                                           |
| canal.instance.rds.secretkey               | aliyun账号的sk信息(如果不需要在本地binlog超过18小时被清理后自动下载oss上的binlog，可以忽略该值) | 无                                                           |
| canal.admin.manager                        | canal链接canal-admin的地址 (v1.1.4新增)                      | 无                                                           |
| canal.admin.port                           | admin管理指令链接端口 (v1.1.4新增)                           | 11110                                                        |
| canal.admin.user                           | admin管理指令链接的ACL配置 (v1.1.4新增)                      | admin                                                        |
| canal.admin.passwd                         | admin管理指令链接的ACL配置 (v1.1.4新增)                      | 密码默认值为admin的密文                                      |
| canal.user                                 | canal数据端口订阅的ACL配置 (v1.1.4新增)如果为空，代表不开启  | 无                                                           |
| canal.passwd                               | canal数据端口订阅的ACL配置 (v1.1.4新增)如果为空，代表不开启  | 无                                                           |

 

#### instance.properties介绍：

a. 在canal.properties定义了canal.destinations后，需要在canal.conf.dir对应的目录下建立同名的文件

比如：

 

```
canal.destinations = example1,example2
```

 这时需要创建example1和example2两个目录，每个目录里各自有一份instance.properties.

 

 ps. canal自带了一份instance.properties demo，可直接复制conf/example目录进行配置修改

 

```bash
cp -R example example1/
cp -R example example2/
```

 

 

b. 如果canal.properties未定义instance列表，但开启了canal.auto.scan时

- server第一次启动时，会自动扫描conf目录下，将文件名做为instance name，启动对应的instance
- server运行过程中，会根据canal.auto.scan.interval定义的频率，进行扫描
  \1. 发现目录有新增，启动新的instance
  \2. 发现目录有删除，关闭老的instance
  \3. 发现对应目录的instance.properties有变化，重启instance

一个标准的conf目录结果：

 

```bash
jianghang@jianghang-laptop:~/work/canal/deployer/target/canal$ ls -l conf/
总用量 8
-rwxrwxrwx 1 jianghang jianghang 1677 2013-03-19 15:03 canal.properties  ##系统配置
drwxr-xr-x 2 jianghang jianghang   88 2013-03-19 15:03 example  ## instance配置
-rwxrwxrwx 1 jianghang jianghang 1840 2013-03-19 15:03 logback.xml ## 日志文件
drwxr-xr-x 2 jianghang jianghang  168 2013-03-19 17:04 spring  ## spring instance目录
```

 

 

instance.properties参数列表：

| 参数名字                           | 参数说明                                                     | 默认值         |
| ---------------------------------- | ------------------------------------------------------------ | -------------- |
| canal.instance.mysql.slaveId       | mysql集群配置中的serverId概念，需要保证和当前mysql集群中id唯一 (v1.1.x版本之后canal会自动生成，不需要手工指定) | 无             |
| canal.instance.master.address      | mysql主库链接地址                                            | 127.0.0.1:3306 |
| canal.instance.master.journal.name | mysql主库链接时起始的binlog文件                              | 无             |
| canal.instance.master.position     | mysql主库链接时起始的binlog偏移量                            | 无             |
| canal.instance.master.timestamp    | mysql主库链接时起始的binlog的时间戳                          | 无             |
| canal.instance.gtidon              | 是否启用mysql gtid的订阅模式                                 | false          |
| canal.instance.master.gtid         | mysql主库链接时对应的gtid位点                                | 无             |
| canal.instance.dbUsername          | mysql数据库帐号                                              | canal          |
| canal.instance.dbPassword          | mysql数据库密码                                              | canal          |
| canal.instance.defaultDatabaseName | mysql链接时默认schema                                        |                |
| canal.instance.connectionCharset   | mysql 数据解析编码                                           | UTF-8          |
| canal.instance.filter.regex        | mysql 数据解析关注的表，Perl正则表达式.多个正则之间以逗号(,)分隔，转义符需要双斜杠(\\) 常见例子：1.  所有表：.*  or  .*\\..* 2.  canal schema下所有表： canal\\..* 3.  canal下的以canal打头的表：canal\\.canal.* 4.  canal schema下的一张表：canal\\.test15.  多个规则组合使用：canal\\..*,mysql.test1,mysql.test2 (逗号分隔) | .*\\..*        |
| canal.instance.filter.black.regex  | mysql 数据解析表的黑名单，表达式规则见白名单的规则           | 无             |
| canal.instance.rds.instanceId      | aliyun rds对应的实例id信息(如果不需要在本地binlog超过18小时被清理后自动下载oss上的binlog，可以忽略该值) | 无             |

 

几点说明：

> \1.  mysql链接时的起始位置
>
> - canal.instance.master.journal.name +  canal.instance.master.position :  精确指定一个binlog位点，进行启动
> - canal.instance.master.timestamp :  指定一个时间戳，canal会自动遍历mysql binlog，找到对应时间戳的binlog位点后，进行启动
> - 不指定任何信息：默认从当前数据库的位点，进行启动。(show master status)
>
> \2. mysql解析关注表定义
>
> - 标准的Perl正则，注意转义时需要双斜杠：\\
>
> \3. mysql链接的编码
>
> - 目前canal版本仅支持一个数据库只有一种编码，如果一个库存在多个编码，需要通过filter.regex配置，将其拆分为多个canal instance，为每个instance指定不同的编码



### mq相关参数说明 (>=1.1.5版本)

在1.1.5版本开始，引入了[MQ Connector设计](https://github.com/alibaba/canal/pull/2562)，因此参数配置做了部分调整

| 参数名                                      | 参数说明                                                     | 默认值                     |
| :------------------------------------------ | ------------------------------------------------------------ | -------------------------- |
| canal.aliyun.accessKey                      | 阿里云ak                                                     | 无                         |
| canal.aliyun.secretKey                      | 阿里云sk                                                     | 无                         |
| canal.aliyun.uid                            | 阿里云uid                                                    | 无                         |
| canal.mq.flatMessage                        | 是否为json格式 如果设置为false,对应MQ收到的消息为protobuf格式 需要通过CanalMessageDeserializer进行解码 | false                      |
| canal.mq.canalBatchSize                     | 获取canal数据的批次大小                                      | 50                         |
| canal.mq.canalGetTimeout                    | 获取canal数据的超时时间                                      | 100                        |
| canal.mq.accessChannel = local              | 是否为阿里云模式，可选值local/cloud                          | local                      |
| canal.mq.database.hash                      | 是否开启database混淆hash，确保不同库的数据可以均匀分散，如果关闭可以确保只按照业务字段做MQ分区计算 | true                       |
| canal.mq.send.thread.size                   | MQ消息发送并行度                                             | 30                         |
| canal.mq.build.thread.size                  | MQ消息构建并行度                                             | 8                          |
| ------                                      | -----------                                                  | -------                    |
| kafka.bootstrap.servers                     | kafka服务端地址                                              | 127.0.0.1:9092             |
| kafka.acks                                  | kafka为`ProducerConfig.ACKS_CONFIG`                          | all                        |
| kafka.compression.type                      | 压缩类型                                                     | none                       |
| kafka.batch.size                            | kafka为`ProducerConfig.BATCH_SIZE_CONFIG`                    | 16384                      |
| kafka.linger.ms                             | kafka为`ProducerConfig.LINGER_MS_CONFIG` , 如果是flatMessage格式建议将该值调大, 如: 200 | 1                          |
| kafka.max.request.size                      | kafka为`ProducerConfig.MAX_REQUEST_SIZE_CONFIG`              | 1048576                    |
| kafka.buffer.memory                         | kafka为`ProducerConfig.BUFFER_MEMORY_CONFIG`                 | 33554432                   |
| kafka.max.in.flight.requests.per.connection | kafka为`ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION` | 1                          |
| kafka.retries                               | 发送失败重试次数                                             | 0                          |
| kafka.kerberos.enable                       | kerberos认证                                                 | false                      |
| kafka.kerberos.krb5.file                    | kerberos认证                                                 | ../conf/kerberos/krb5.conf |
| kafka.kerberos.jaas.file                    | kerberos认证                                                 | ../conf/kerberos/jaas.conf |
| ------                                      | -----------                                                  | -------                    |
| rocketmq.producer.group                     | rocketMQ为ProducerGroup名                                    | test                       |
| rocketmq.enable.message.trace               | 是否开启message trace                                        | false                      |
| rocketmq.customized.trace.topic             | message trace的topic                                         | 无                         |
| rocketmq.namespace                          | rocketmq的namespace                                          | 无                         |
| rocketmq.namesrv.addr                       | rocketmq的namesrv地址                                        | 127.0.0.1:9876             |
| rocketmq.retry.times.when.send.failed       | 重试次数                                                     | 0                          |
| rocketmq.vip.channel.enabled                | rocketmq是否开启vip channel                                  | false                      |
| rocketmq.tag                                | rocketmq的tag配置                                            | 空值                       |
| ---                                         | ---                                                          | ---                        |
| rabbitmq.host                               | rabbitMQ配置                                                 | 无                         |
| rabbitmq.virtual.host                       | rabbitMQ配置                                                 | 无                         |
| rabbitmq.exchange                           | rabbitMQ配置                                                 | 无                         |
| rabbitmq.username                           | rabbitMQ配置                                                 | 无                         |
| rabbitmq.password                           | rabbitMQ配置                                                 | 无                         |
| rabbitmq.deliveryMode                       | rabbitMQ配置                                                 | 无                         |
| ---                                         | ---                                                          | ---                        |
| pulsarmq.serverUrl                          | pulsarmq配置                                                 | 无                         |
| pulsarmq.roleToken                          | pulsarmq配置                                                 | 无                         |
| pulsarmq.topicTenantPrefix                  | pulsarmq配置                                                 | 无                         |
| ---                                         | ---                                                          | ---                        |
| canal.mq.topic                              | mq里的topic名                                                | 无                         |
| canal.mq.dynamicTopic                       | mq里的动态topic规则, 1.1.3版本支持                           | 无                         |
| canal.mq.partition                          | 单队列模式的分区下标，                                       | 1                          |
| canal.mq.enableDynamicQueuePartition        | 动态获取MQ服务端的分区数,如果设置为true之后会自动根据topic获取分区数替换canal.mq.partitionsNum的定义,目前主要适用于RocketMQ | false                      |
| canal.mq.partitionsNum                      | 散列模式的分区数                                             | 无                         |
| canal.mq.dynamicTopicPartitionNum           | mq里的动态队列分区数,比如针对不同topic配置不同partitionsNum  | 无                         |
| canal.mq.partitionHash                      | 散列规则定义 库名.表名 : 唯一主键，比如mytest.person: id 1.1.3版本支持新语法，见下文 | 无                         |

**以下所有注意转义！！！！！可以看上面Springboot整合实例**

而且尼玛！！！

```properties
# table regex
# table regex .*\\..*表示监听所有表 也可以写具体的表名，用，隔开
canal.instance.filter.regex=Book_Collection_System.bcs_order,Book_Collection_System.book,Book_Collection_System.appeal
# table black regex

canal.instance.filter.black.regex=mysql\\.slave_.*
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
canal.mq.topic=
#canal.routing.key
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.dynamicTopic= bcs_bcs_order:Book_Collection_System\\.bcs_order,bcs_book:Book_Collection_System\\.book,bcs_appeal:Book_Collection_System\\.appeal
```

#### canal.mq.dynamicTopic 表达式说明

canal 1.1.3版本之后, 支持配置格式：schema 或 schema.table，多个配置之间使用逗号或分号分隔

- 例子1：test\\.test 指定匹配的单表，发送到以test_test为名字的topic上
- 例子2：.*\\..* 匹配所有表，则每个表都会发送到各自表名的topic上
- 例子3：test 指定匹配对应的库，一个库的所有表都会发送到库名的topic上
- 例子4：test\\..* 指定匹配的表达式，针对匹配的表会发送到各自表名的topic上
- 例子5：test,test1\\.test1，指定多个表达式，会将test库的表都发送到test的topic上，test1\\.test1的表发送到对应的test1_test1 topic上，其余的表发送到默认的canal.mq.topic值

为满足更大的灵活性，允许对匹配条件的规则指定发送的topic名字，配置格式：topicName:schema 或 topicName:schema.table

- 例子1: test:test\\.test 指定匹配的单表，发送到以test为名字的topic上
- 例子2: test:.*\\..* 匹配所有表，因为有指定topic，则每个表都会发送到test的topic下
- 例子3: test:test 指定匹配对应的库，一个库的所有表都会发送到test的topic下
- 例子4：testA:test\\..* 指定匹配的表达式，针对匹配的表会发送到testA的topic下
- 例子5：test0:test,test1:test1\\.test1，指定多个表达式，会将test库的表都发送到test0的topic下，test1\\.test1的表发送到对应的test1的topic下，其余的表发送到默认的canal.mq.topic值

大家可以结合自己的业务需求，设置匹配规则，建议MQ开启自动创建topic的能力

#### canal.mq.partitionHash 表达式说明

canal 1.1.3版本之后, 支持配置格式：schema.table:pk1^pk2，多个配置之间使用逗号分隔

- 例子1：test\\.test:pk1^pk2 指定匹配的单表，对应的hash字段为pk1 + pk2
- 例子2：.*\\..*:id 正则匹配，指定所有正则匹配的表对应的hash字段为id
- 例子3：.*\\..*:$pk$ 正则匹配，指定所有正则匹配的表对应的hash字段为表主键(自动查找)
- 例子4: 匹配规则啥都不写，则默认发到0这个partition上
- 例子5：.*\\..* ，不指定pk信息的正则匹配，将所有正则匹配的表,对应的hash字段为表名
  - 按表hash: 一张表的所有数据可以发到同一个分区，不同表之间会做散列 (会有热点表分区过大问题)
- 例子6: test\\.test:id,.\\..* , 针对test的表按照id散列,其余的表按照table散列

注意：大家可以结合自己的业务需求，设置匹配规则，多条匹配规则之间是按照顺序进行匹配(命中一条规则就返回)

其他详细参数可参考[Canal AdminGuide](https://github.com/alibaba/canal/wiki/AdminGuide)

##### mq顺序性问题

binlog本身是有序的，写入到mq之后如何保障顺序是很多人会比较关注，在issue里也有非常多人咨询了类似的问题，这里做一个统一的解答

1. canal目前选择支持的kafka/rocketmq，本质上都是基于本地文件的方式来支持了分区级的顺序消息的能力，也就是binlog写入mq是可以有一些顺序性保障，这个取决于用户的一些参数选择
2. canal支持MQ数据的几种路由方式：单topic单分区，单topic多分区、多topic单分区、多topic多分区

- canal.mq.dynamicTopic，主要控制是否是单topic还是多topic，针对命中条件的表可以发到表名对应的topic、库名对应的topic、默认topic name
- canal.mq.partitionsNum、canal.mq.partitionHash，主要控制是否多分区以及分区的partition的路由计算，针对命中条件的可以做到按表级做分区、pk级做分区等

1. canal的消费顺序性，主要取决于描述2中的路由选择，举例说明：

- 单topic单分区，可以严格保证和binlog一样的顺序性，缺点就是性能比较慢，单分区的性能写入大概在2~3k的TPS
- 多topic单分区，可以保证表级别的顺序性，一张表或者一个库的所有数据都写入到一个topic的单分区中，可以保证有序性，针对热点表也存在写入分区的性能问题
- 单topic、多topic的多分区，如果用户选择的是指定table的方式，那和第二部分一样，保障的是表级别的顺序性(存在热点表写入分区的性能问题)，如果用户选择的是指定pk hash的方式，那只能保障的是一个pk的多次binlog顺序性 ** pk hash的方式需要业务权衡，这里性能会最好，但如果业务上有pk变更或者对多pk数据有顺序性依赖，就会产生业务处理错乱的情况. 如果有pk变更，pk变更前和变更后的值会落在不同的分区里，业务消费就会有先后顺序的问题，需要注意



## 遇见的问题

### canal异常 Could not find first log file name in binary log index file

# 问题

最近在使用canal来监测数据库的变化，处理变动的数据。由于有一段时间没有用了，这次启动在日志文件中看到这个异常 Could not find first log file name in binary log index file，详细信息如下：

```log
2020-12-16 19:14:42.053 [destination = tradeAndRefund , address = /192.168.1.6:3306 , EventParser] ERROR c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - dump address /192.168.1.6:3306 has an error, retrying. caused by
java.io.IOException: Received error packet: errno = 1236, sqlstate = HY000 errmsg = Could not find first log file name in binary log index file
        at com.alibaba.otter.canal.parse.inbound.mysql.dbsync.DirectLogFetcher.fetch(DirectLogFetcher.java:102) ~[canal.parse-1.1.4.jar:na]
        at com.alibaba.otter.canal.parse.inbound.mysql.MysqlConnection.dump(MysqlConnection.java:235) ~[canal.parse-1.1.4.jar:na]
        at com.alibaba.otter.canal.parse.inbound.AbstractEventParser$3.run(AbstractEventParser.java:265) ~[canal.parse-1.1.4.jar:na]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_162]
2020-12-16 19:14:42.053 [destination = tradeAndRefund , address = /192.168.1.6:3306 , EventParser] ERROR com.alibaba.otter.canal.common.alarm.LogAlarmHandler - destination:tradeAndRefund[java.io.IOException: Received error packet: errno = 1236, sqlstate = HY000 errmsg = Could not find first log file name in binary log index file
        at com.alibaba.otter.canal.parse.inbound.mysql.dbsync.DirectLogFetcher.fetch(DirectLogFetcher.java:102)
        at com.alibaba.otter.canal.parse.inbound.mysql.MysqlConnection.dump(MysqlConnection.java:235)
        at com.alibaba.otter.canal.parse.inbound.AbstractEventParser$3.run(AbstractEventParser.java:265)
        at java.lang.Thread.run(Thread.java:748)
]
123456789101112
```

# 解决

在配置目录conf里面对应的instance配置目录下有一个meta.dat的文件，里面记录了上次处理的位置。把这个文件直接删掉，或者纠正里面的binlog位置信息，问题就解决了。

# 解决过程

首先我要说下我的环境。我用了canal的admin来进行集群管理，canal.deployer中我使用start.sh local 来启动了server。运行起来之后集群状态正常，查看instance的日志就发现了上面的日志。
然后说一下我的解决过程。一开始看到这个问题，就知道是instance寻找binlog的位置信息时出现了问题，所以翻了一下文档。文档中说可以通过canal.instance.master.journal.name和canal.instance.master.position这两个配置来指定position，也可以通过canal.instance.master.timestamp来指定一个时间。但是这两个配置我都试了，不行！始终有报错。
最后是在没办法，只好一行一行的看启动日志，发现了下面这个信息：

```shell
2020-12-16 19:14:42.045 [destination = tradeAndRefund , address = /192.168.1.6:3306 , EventParser] DEBUG java.sql.ResultSet - {rset-100008} Result: [31, 2020-11-10 17:00:18.504, 2020-11-10 17:00:18.504, tradeAndRefund, mysql-bin.000037, 536485765, 10, 1604921837000, 
1
```

这个是寻找location的日志，我发现这里的文件和位置都不是我设置的，那他是哪里来的呢？最后我在canal.deployer的根目录下执行`grep '000037' ./* -r`命令，最终刚发现在logs目录和conf目录下对应的instance中找到了这些内容。分别删掉这两个东西进行重启，最后发现config下面那个是起作用的，把他给改了就好了。
问题解决~
