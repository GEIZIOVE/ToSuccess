[Seata 是什么](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

## 1、注意事项

- 避免直接拉取latest版本镜像，latest版本并不一定是released版本，为避免不必要的问题，请到[docker镜像仓库](https://hub.docker.com/r/seataio/seata-server/tags)确定要拉取的镜像版本。

## 2、下载镜像

```bash
docker pull seataio/seata-server:1.4.2
```

## 3、启动[容器](https://cloud.tencent.com/product/tke?from=10680)

### docker run启动

```bash
docker run -d --name seata-server -p 8091:8091 seataio/seata-server:1.4.2
```

### Docker compose 启动

`docker-compose.yaml` 示例

```yaml
version: "3"
services:
  seata-server:
    image: seataio/seata-server:${latest-release-version}
    hostname: seata-server
    ports:
      - "8091:8091"
      - "7091:7091"
    environment:
      - SEATA_PORT=8091
      - STORE_MODE=file
```

## 4、拷贝文件

```bash
docker cp seata-server:/seata-server  /usr/local/dk/docker/seata/
```

![image-20220919161959696](E:\Development\Typora\images\image-20220919161959696.png)

## 5、修改配置文件

（1）修改配置文件/resources/registry.conf，改为Nacos信息。

```bash
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "106.53.124.182:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }

......

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "106.53.124.182:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
    dataId = "seataServer.properties"
  }
}
```

（2）修改配置文件/resources/file.conf，改为DB信息。

```bash
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "db"
  
  ......

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.cj.jdbc.Driver"
    ## if using mysql to store the data, recommend add rewriteBatchedStatements=true in jdbc connection param
    url = "jdbc:mysql://localhost:3306/seata?rewriteBatchedStatements=true"
    user = "root"
    password = "root"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }
  ......
}
```

### 5.1初始化数据库

① 使用 [`mysql.sql`](https://github.com/seata/seata/blob/develop/script/server/db/mysql.sql) 脚本，初始化 Seata TC Server 的 db 数据库。脚本内容如下：

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```



在 MySQL 中，创建 `seata` 数据库，并在该库下执行该脚本。最终结果如下图：![ 数据库 - MySQL 5.X](E:\Development\Typora\images\22-16635766698623.png)

③ MySQL8 的支持

> 如果胖友使用的 MySQL 是 8.X 版本，则需要看该步骤。否则，可以直接跳过。

首先，需要下载 MySQL 8.X JDBC 驱动，命令行操作如下：

```
$ cd lib
$ wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.19/mysql-connector-java-8.0.19.jar
```



然后，修改 `conf/file` 配置文件，使用该 MySQL 8.X JDBC 驱动。如下图所示：![ 数据库 - MySQL 8.X](E:\Development\Typora\images\24-16635767237966.png)

## 6、停掉旧容器

```bash
docker stop seata-server
docker rm seata-server
```

## 7、启动新容器

```bash
docker run -d \
--restart always \
--name seata-server \
-p 8091:8091 \
-v /usr/local/dk/docker/seata/seata-server/:/seata-server \
-e SEATA_IP=106.53.124.182 \
-e SEATA_PORT=8091 \
seataio/seata-server:1.4.2
```

**注意：** 遇到的坑，如果是部署云[服务器](https://cloud.tencent.com/product/cvm?from=10680)，没有设置SEATA_IP，默认注册的是docker的内网ip，seata启动虽然没有问题，但是微服务项目启动连接时，会报错

```bash
can not register RM,err:can not connect to services-server.
```

## 8、查看Nacos注册情况

![img](E:\Development\Typora\images\1620.png)